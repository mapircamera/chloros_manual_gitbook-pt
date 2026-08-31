# Adaptação Dinâmica de Computação

O Chloros 1.2.0 utiliza a deteção de hardware e a seleção automática da estratégia de processamento. O motor de processamento adapta-se ao seu hardware — desde um Jetson Orin Nano até uma estação de trabalho com várias GPUs — sem qualquer configuração manual.

***

## Como funciona

Quando o Chloros é iniciado, faz um perfil do seu sistema:

1. **Deteta o sistema operativo** — Windows ou Linux
2. **Identifica os núcleos da CPU e a RAM total**

3.**Deteta a presença de GPU** — Capacidade NVIDIA CUDA, VRAM, modelo
4. **Identifica o modelo Jetson** (se aplicável) — através do `/proc/device-tree/model`
5. **Verifica os sensores térmicos** (Jetson) — para um processamento sensível à temperatura
6. **Seleciona a estratégia de computação** — com base em todo o hardware detetado
7. **Configura automaticamente o número de trabalhadores, o tipo de pipeline e a alocação de memória**

O perfil detetado é armazenado em cache para a sessão, tanto na memória como no disco, para que as execuções posteriores iniciem mais rapidamente:

| Plataforma | Perfil em cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (respeita o `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Elimine esse ficheiro para forçar uma nova deteção — útil após adicionar uma GPU ou mais memória RAM. O Chloros também volta a detetar automaticamente quando o cache foi gravado por uma versão anterior incompatível.

***

## Estratégias de computação

O Chloros seleciona uma de três estratégias de computação com base no seu hardware:

| Estratégia | Selecionada quando | Trabalhadores | Executor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA com**12 GB ou mais de VRAM**(na memória unificada do Jetson, requer também 12 GB ou mais de RAM partilhada total) | `min(4, VRAM ÷ 4GB)`, mínimo de 2 —**limitado a 2 no Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA com**2 a 12 GB de VRAM**| 3 (sobreposição de E/S; acesso à GPU serializado por um semáforo).**1 (sequencial) em Jetsons com menos de 12 GB de RAM** | `ProcessPoolExecutor` (spawn); sequencial no próprio processo em Jetsons com pouca RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Sem GPU CUDA ou com menos de 2 GB de VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exemplos práticos da fórmula de trabalhadores do `GPU_PARALLEL`: 12 GB de VRAM → 3 trabalhadores, 16 GB ou mais → 4 trabalhadores, qualquer Jetson → 2 trabalhadores.

O paralelismo é implementado com o `concurrent.futures` padrão do Python: As estratégias de GPU utilizam um `ProcessPoolExecutor` com o método de início **spawn** (cada worker é um processo separado com o seu próprio contexto CUDA — o `fork` copiaria um estado CUDA já inicializado e corromperia os processos filhos), e a estratégia de CPU utiliza um `ThreadPoolExecutor`. O Chloros não utiliza qualquer framework distribuído de terceiros (como o Ray).

### Tipos de pipeline

* **`fused_gpu`** — Percurso de processamento totalmente na GPU. As operações de debayer, correção e indexação são executadas na GPU numa única passagem fusionada. Apresenta o maior débito, mas requer mais VRAM.
* **`tiled_gpu`** — Percurso de GPU com eficiência de memória. Processa imagens em blocos para caberem na memória limitada da GPU. Tem um rendimento inferior, mas funciona em dispositivos com restrições de memória.
* **`cpu_fallback`** — Processamento exclusivamente na CPU utilizando paralelismo multithread. Utilizado quando não está disponível nenhuma GPU NVIDIA e como último recurso quando ambas as vias da GPU falham.

A cadeia de alternativas em tempo de execução é sempre `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Substituição manual da estratégia

Defina a variável de ambiente `CHLOROS_STRATEGY` para forçar uma estratégia específica — uma opção avançada para quando a deteção automática escolher algo inadequado para a sua situação (por exemplo, manter a GPU livre para outras tarefas):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

A correspondência da variável não distingue maiúsculas de minúsculas; tudo o que não for um dos três nomes é ignorado e a deteção automática prossegue normalmente. Mesmo com uma substituição, o Chloros continua a selecionar o número de trabalhadores por si:

| Substituição | Número de trabalhadores utilizado |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

É preferível definir isto por comando, em vez de de forma permanente, para que as execuções normais continuem a adaptar-se automaticamente.

***

## Comportamento específico da plataforma

| Plataforma | Estratégia | Trabalhadores | Pipeline | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequencial) | Modo eficiente em termos de memória, uma imagem de cada vez |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequencial) | A memória RAM partilhada inferior a 12 GB obriga ao processamento sequencial |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Dispositivo de borda recomendado — limite do Jetson em 2 trabalhadores |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Desempenho máximo na periferia (também limitado pelo Jetson a 2 trabalhadores) |
| **Computador de secretária com GPU de 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 trabalhadores sobrepõem E/S enquanto um semáforo serializa o acesso à GPU |
| **Computador de secretária com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (concorrente) | Desempenho ideal no computador de secretária: 12 GB → 3 trabalhadores, 16 GB ou mais → 4 |
| **Sistema apenas com CPU** | `CPU_PARALLEL` | núcleos físicos − 1 (mín. 2) | `cpu_fallback` | Não requer GPU, utiliza um conjunto de threads |

{% hint style="info" %}
**Memória unificada do Jetson**: os dispositivos Jetson partilham a memória da GPU e da CPU. Um Jetson Orin NX de 16 GB apresenta cerca de 15,3 GB de VRAM, mas trata-se da mesma RAM física utilizada pelo SO e pelos processos da CPU. É por isso que os Jetsons com 16 GB ou mais se qualificam para o `GPU_PARALLEL` tal como uma GPU de computador de secretária com 12 GB ou mais, mas estão limitados a 2 trabalhadores — a GPU, os processos de trabalho e os seus contextos CUDA por trabalhador recorrem todos ao mesmo conjunto partilhado.
{% endhint %}

### Orçamento da GPU por VRAM (GPUs discretas)

Em anfitriões x86_64 com uma GPU NVIDIA discreta, a VRAM detetada também define a quantidade de recursos que o processamento da placa pode reclamar e até que tamanho os lotes podem atingir:

| VRAM detetada | Limite máximo do orçamento da GPU | Multiplicador do tamanho do lote |
| --- | --- | --- |
| **8 GB ou mais** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5-6 GB** | 80% | ×1,5 |
| **2-3,5 GB** | 75% | ×1,25 |
| **Menos de 2 GB** | 70% | ×1,0 |

As GPUs discretas reservam apenas 0,5 GB para o sistema, uma vez que não partilham a memória RAM do sistema. Os perfis Jetson reservam muito mais e têm um limite inferior — consulte o [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Atribuição dinâmica de memória da GPU

O Chloros utiliza um [pipeline de processamento de 4 threads](processing-pipeline.md):

* **Thread 1** (Detecção) — Carregamento da imagem, análise de EXIF, deteção do alvo
* **Thread 2** (Calibração) — Cálculo da calibração de refletância
* **Thread 3** (Processamento) — Debayer da GPU, correção de vinheta, cálculo do índice
* **Thread 4** (Exportação) — Gravação de ficheiros, incorporação de metadados

As threads 1, 2 e 4 consomem poucos recursos da GPU; a thread 3 é a que mais consome. À medida que as threads anteriores do pipeline terminam, a sua quota de GPU é **redistribuída pelas restantes threads ativas**, pelo que a thread 3 recebe progressivamente mais memória à medida que a execução avança.

### Fases de alocação

| Fase | Threads ativos | Distribuição da memória da GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Dividida por todos os threads, a maior parte para o Thread 3 |
| **Início-meio** | 2, 3, 4 | A quota do Thread 1 é redistribuída |
| **Meio-fim** | 3, 4 | As quotas dos Threads 1 e 2 vão para os Threads 3 e 4 |
| **Tardia** | 3 ou 4 | A última thread ativa recebe a sua alocação máxima |

Duas regras regem estes valores:

* Uma thread que seja a **única** ativa recebe a alocação máxima do seu perfil.
* Quando mais do que uma tarefa *pesada* da GPU está ativa, a alocação base de cada tarefa pesada é dividida entre elas (nunca ficando abaixo do mínimo configurado).

O valor efetivamente utilizado em tempo de execução é o **mais baixo** entre a alocação do perfil da plataforma e a recomendação em tempo real do monitor de memória da GPU, pelo que uma placa ocupada prevalece sempre sobre um perfil otimista.***

## Processamento sensível à textura

O debayer sensível à textura (**apenas Chloros+** — `--debayer texture-aware`) executa um modelo de redução de ruído baseado em IA/ML que necessita de cerca de 1,75 GB de VRAM em FP16 por cópia, pelo que utiliza muito mais memória da GPU do que o método padrão:

* Os sistemas com **menos de 7 GB de VRAM**processam o «Texture Aware» num**ciclo síncrono, uma imagem de cada vez** — não é possível acomodar várias cópias do modelo, e um conjunto de trabalhadores apenas aumentaria a contenção
* Os sistemas com **7 GB ou mais de VRAM** podem processar o Texture Aware em simultâneo, embora com um número reduzido de trabalhadores em comparação com o método Standard
* No **Jetson**, o Texture Aware é sempre atribuído a um único trabalhador e, em modelos de baixo consumo (Nano, Orin Nano), também aplica automaticamente um limite de frequência da GPU — consulte o [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Gestão térmica (Jetson)

Os dispositivos Jetson têm restrições térmicas, especialmente em implementações em espaços fechados ou a bordo de aeronaves. O Chloros monitoriza os sensores de temperatura integrados no Jetson e ajusta automaticamente o tamanho dos lotes:

| Temperatura | Resposta |
| --- | --- |
| **&lt; 70 °C** | Funcionamento normal — velocidade máxima |
| **70 °C** (Aviso) | O tamanho do lote é reduzido progressivamente (100% → 50% entre 70 °C e 80 °C) |
| **80 °C** (Crítico) | Limitação agressiva (50% → 0% entre 80 °C e 90 °C) |
| **90 °C** (Desligamento) | Interrompe totalmente o processamento da GPU |

Em sistemas de secretária com refrigeração adequada, a limitação térmica raramente é acionada.

***

## Gestão da pressão de memória

O Chloros monitoriza continuamente a memória da GPU durante o processamento e reage em três níveis.

**Dimensionamento de lotes.** Um lote começa com 8 imagens multiplicadas pelo multiplicador da plataforma, conforme as tabelas acima. Chloros verifica então a VRAM livre, reserva 20% da mesma para a sobrecarga própria do PyTorch e assume cerca de 100 MB de memória da GPU por imagem de 12 MP — o lote é o menor entre o limite derivado da memória e a base da plataforma. Nunca desce abaixo de 1.**Redução preventiva.**Acima de**85% de utilização da VRAM**, os tamanhos dos lotes são reduzidos antes que ocorra qualquer falha.**Redução da alocação por thread.** À medida que a utilização em tempo real aumenta, o orçamento de GPU de cada thread é reduzido: ×0,75 acima de 80% de utilização, ×0,5 acima de 90%. Os intervalos de monitorização são 70% (conservador), 85% (limite normal de funcionamento) e 95% (risco de OOM).**Retrocesso e recuperação em caso de OOM.** Se, mesmo assim, ocorrer um evento de falta de memória:

* o tamanho do lote é **reduzido para metade** e, novamente, para metade em cada OOM consecutivo — cada lote subsequente bem-sucedido recua essa penalização um passo
* as alocações das threads ativas são reduzidas para 70% do seu valor atual e o alocador muda para a sua estratégia conservadora, relaxando novamente após uma série de alocações bem-sucedidas
* sob pressão severa, o pipeline recua de `fused_gpu` para `tiled_gpu` e, como último recurso, para `cpu_fallback`

**RAM do anfitrião (Jetson).** Antes do processamento, o CLI estima o pico de memória do anfitrião com base na contagem de imagens e no modo de debayer e avisa se a RAM, juntamente com o espaço de troca suportado por ficheiros, for provavelmente insuficiente, apresentando os comandos exatos para adicionar espaço de troca — consulte o [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Monitorização da Adaptação de Computação

### Diagnósticos do Sistema

O `chloros-cli selftest` é a forma mais rápida de confirmar o que a camada de computação deteta:

```bash
chloros-cli selftest
```

As suas 7 verificações abrangem a versão, a disponibilidade das portas, o arranque do backend, o `/api/test`, as informações do sistema, a presença do modelo de redução de ruído e a prontidão do CUDA + redução de ruído. A verificação 5 apresenta diretamente a linha de hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

A verificação 7 apresenta o resultado `CUDA: <bool>, Denoiser: <bool>` — ambas as condições têm de ser verdadeiras para que o Texture Aware possa ser utilizado.

### Registos do backend

A estratégia e o número de trabalhadores são escolhidos no backend no início de cada execução — não há nenhum banner CLI que os anuncie. Quando algo se comporta de forma inesperada (um caminho de GPU a recuar, um OOM, um denoiser que não carrega), é no registo do backend dessa sessão que isso aparece:

| Plataforma | Localização do registo |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (um ficheiro por execução) |
| **Linux, backend CLI iniciado** | também `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progresso em tempo real

Durante uma execução, o CLI apresenta o progresso em tempo real por thread (Detecção, Análise, Processamento, Exportação) transmitido através de Server-Sent Events — a forma prática de verificar se a Thread 3 é o gargalo. Ver [Pipeline de Processamento](processing-pipeline.md).

***

## Próximos passos

* [Pipeline de processamento](processing-pipeline.md) — Compreender a arquitetura do pipeline de 4 threads
* [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementação e otimização específicas para o Jetson
* [CLI : Linha de comandos](../CLI.md) — O guia do CLI
* [Referência do CLI](../reference/cli-reference.md) — Lista exaustiva de comandos para a versão 1.2.0
