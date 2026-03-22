# Adaptação dinâmica de computação

O Chloros 1.1.0 introduz a deteção inteligente de hardware e a seleção automática da estratégia de processamento. O motor de processamento adapta-se ao seu hardware — desde um Jetson Nano até uma estação de trabalho com várias GPUs — sem qualquer configuração manual.

***

## Como funciona

Quando o Chloros é iniciado, ele analisa automaticamente o seu sistema:

1. **Deteta o sistema operativo** — Windows ou Linux
2. **Identifica os núcleos da CPU e a RAM total**

3.**Deteta a presença de GPU** — capacidade NVIDIA CUDA, VRAM, modelo
4. **Identifica o modelo Jetson** (se aplicável) — através do `/proc/device-tree/model`
5. **Verifica os sensores térmicos** (Jetson) — para um processamento sensível à temperatura
6. **Seleciona a estratégia de computação ideal** — com base em todo o hardware detetado
7. **Configura automaticamente o número de trabalhadores, o tipo de pipeline e a alocação de memória**O resultado é armazenado em cache para que as execuções subsequentes iniciem mais rapidamente. Se o hardware for alterado (por exemplo, se for adicionada uma GPU), o Chloros volta a criar o perfil na próxima inicialização.***

## Estratégias de Computação

O Chloros seleciona uma de três estratégias de computação com base no seu hardware:

| Estratégia | GPU Necessária | Trabalhadores | Pipeline | Ideal para |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Sim (12 GB+ de VRAM ou 16 GB+ partilhada) | 3-4 | `fused_gpu` | GPUs de desktop com 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Sim (&lt; 12 GB de VRAM) | 1-3 | `tiled_gpu` | GPUs de nível básico, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Não | núcleos - 1 | `cpu_fallback` | Sistemas sem GPU NVIDIA |

### Tipos de pipeline

* **`fused_gpu`** — Percurso completo de processamento da GPU. Todas as operações de debayer, correção e indexação são executadas na GPU numa única passagem fusionada. Oferece o maior rendimento, mas requer mais VRAM.
* **`tiled_gpu`** — Percurso de GPU com eficiência de memória. Processa imagens em blocos para caberem na memória limitada da GPU. Oferece menor rendimento, mas funciona em dispositivos com restrições de memória.
* **`cpu_fallback`** — Processamento apenas na CPU utilizando paralelismo multithread. Utilizado quando não há nenhuma GPU NVIDIA disponível.***

## Comportamento específico da plataforma

| Plataforma | Estratégia | Trabalhadores | Pipeline | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serializado) | Modo eficiente em termos de memória, processa uma imagem de cada vez |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (concorrente) | Dispositivo de borda recomendado — processamento paralelo real da GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (concorrente) | Máximo desempenho de borda |
| **Desktop com GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bom desempenho de desktop com blocos eficientes em termos de memória |
| **Desktop com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Desempenho ideal no desktop |
| **Sistema apenas com CPU** | `CPU_PARALLEL` | núcleos - 1 | `cpu_fallback` | Não requer GPU, utiliza ThreadPool |

{% hint style="info" %}
**Memória unificada Jetson**: Os dispositivos Jetson partilham a memória da GPU e da CPU. Um Jetson Orin NX de 16 GB apresenta cerca de 15,3 GB de VRAM, mas trata-se da mesma RAM física utilizada pelo sistema operativo e pelos processos da CPU. O Chloros tem isto em conta ao definir os limites de alocação de memória.
{% endhint %}

***

## Alocação dinâmica de memória da GPU

O Chloros utiliza um [pipeline de processamento de 4 threads](processing-pipeline.md):

* **Thread 1** (Detecção) — Carregamento de imagens, análise de EXIF, deteção de alvos
* **Thread 2** (Calibração) — Cálculo da calibração de refletância
* **Thread 3** (Processamento) — Debayer da GPU, correção de vinheta, cálculo de índice
* **Thread 4** (Exportação) — Gravação de ficheiros, incorporação de metadados

À medida que as threads anteriores do pipeline concluem o seu trabalho (por exemplo, todas as imagens foram detetadas), a sua alocação de memória da GPU é libertada e **redistribuída pelas restantes threads ativas**. Isto significa que a Thread 3 (a fase que exige maior uso da GPU) obtém progressivamente mais memória à medida que o pipeline avança, melhorando o rendimento para o trabalho que exige maior capacidade de computação.

### Fases de alocação

| Fase | Threads ativas | Distribuição da memória da GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Dividida por todas as threads |
| **Início-meio** | 2, 3, 4 | Memória da thread 1 redistribuída |
| **Meio-fim** | 3, 4 | Memória das threads 1+2 vai para 3+4 |
| **Fim** | 3 ou 4 | Memória máxima para a thread restante |***

## Processamento Texture Aware

O método de debayer Texture Aware (apenas Chloros+) utiliza significativamente mais memória da GPU do que o método Standard devido ao modelo de redução de ruído de IA/ML:

* Os sistemas com **&lt; 7 GB de VRAM** são forçados a um ciclo de processamento síncrono para o modo Texture Aware (uma imagem de cada vez)
* Os sistemas com **7 GB ou mais de VRAM** podem processar o modo Texture Aware em simultâneo, embora com um número reduzido de trabalhadores em comparação com o modo Standard***

## Gestão térmica (Jetson)

Os dispositivos Jetson têm restrições térmicas, especialmente em implementações fechadas ou aéreas. O Chloros monitoriza as temperaturas da GPU e da CPU e ajusta automaticamente o processamento:

| Temperatura | Resposta |
| --- | --- |
| **&lt; 70 °C** | Funcionamento normal — velocidade máxima |
| **70 °C** (Aviso) | Reduzir tamanho do lote |
| **80 °C** (Crítico) | Limitação agressiva — diminuir a simultaneidade e o número de trabalhadores |
| **90 °C** (Desligamento) | Interromper totalmente o processamento da GPU |

A monitorização da temperatura utiliza o `tegrastats` em plataformas Jetson. Em sistemas de secretária com refrigeração adequada, a limitação térmica raramente é acionada.

***

## Gestão da pressão de memória

O Chloros monitoriza a pressão da memória do sistema durante o processamento:

* **Limite de memória**: uma utilização de 85% aciona um comportamento conservador
* **Redução por falta de memória**: se ocorrer um evento de falta de memória, a alocação é reduzida em 25% (multiplicador de 0,75x)
* **Recurso de fallback do pipeline**: Sob pressão de memória severa, o pipeline recua automaticamente de `fused_gpu` para `tiled_gpu`
* **Recomendações de swap**: No Jetson, o Chloros avisa se o espaço de swap for insuficiente para o tamanho do seu conjunto de dados***

## Monitorização da adaptação de computação

### Saída de estado do CLI

Quando o processamento começa, o CLI apresenta o perfil de hardware detetado:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnóstico do sistema

Execute o `chloros-cli selftest` para ver um perfil de hardware completo e verificar as capacidades de computação:

```bash
chloros-cli selftest
```

Isto verifica a disponibilidade da CUDA, a memória da GPU, os modelos de redução de ruído e a conectividade do backend.

***

## Próximos passos

* [Pipeline de processamento](processing-pipeline.md) — Compreender a arquitetura do pipeline de 4 threads
* [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementação e otimização específicas para Jetson
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
