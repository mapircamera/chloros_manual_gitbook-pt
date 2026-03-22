# Pipeline de processamento

O Chloros 1.1.0 utiliza um pipeline de processamento de 4 threads que funciona como uma linha de montagem por etapas. Cada thread lida com uma fase distinta do fluxo de trabalho de processamento, permitindo que várias imagens sejam processadas em simultâneo em diferentes etapas.

***

## Arquitetura do pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Cada imagem passa por todas as quatro threads em ordem. Com o processamento multithread do Chloros+, várias imagens podem estar em threads diferentes simultaneamente — enquanto a Thread 3 processa uma imagem, a Thread 1 pode estar a detetar a seguinte, a Thread 2 pode estar a calibrar outra e a Thread 4 pode estar a gravar uma imagem processada anteriormente no disco.

***

## Detalhes das Threads

### Thread 1: Detecção

**Objetivo**: Carregar imagens e detetar alvos de calibração.

* Lê ficheiros de imagem do disco (RAW, JPG)
* Extraia metadados EXIF (GPS, modelo da câmara, carimbos de data/hora, exposição)
* Detecte alvos de calibração ArUco em imagens de alvo marcadas
* Resultados: dados de imagem + metadados + resultados da deteção de alvos

Esta é principalmente uma thread limitada por E/S e CPU.

### Thread 2: Calibração

**Objetivo**: Calcular parâmetros de calibração a partir dos alvos detetados.

* Calcula coeficientes de calibração de refletância a partir de imagens de alvo
* Calcula parâmetros de correção de vinheta
* Determina curvas de calibração por banda
* Resultados: parâmetros de calibração para cada imagem

Esta é uma thread de computação dependente da CPU.

### Thread 3: Processamento (GPU)

**Objetivo**: Aplicar correções e calcular índices de vegetação.**Esta é a thread que mais exige recursos de computação.*** **Debayering**: Converte dados RAW no padrão Bayer em imagens multicanal
  * Padrão (Rápido, Qualidade Média) — padrão
  * Sensível à Textura (Lento, Qualidade Máxima) — apenas Chloros+, utiliza redução de ruído por IA/ML
* **Correção de vinheta**: Aplica correção de vinheta da lente em toda a imagem
* **Calibração de refletância**: Aplica coeficientes de calibração para converter em valores de refletância
* **Cálculo de índices**: Calcula índices de vegetação (NDVI, NDRE, GNDVI, etc.)
* Resultados: dados de imagem processados prontos para exportação

Este segmento beneficia ao máximo da aceleração por GPU. O sistema [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) otimiza principalmente o comportamento deste segmento.

### Thread 4: Exportação

**Objetivo**: Gravar imagens processadas no disco.

* Grava ficheiros de saída no formato selecionado (TIFF 16 bits, TIFF 32 bits %, PNG, JPG)
* Incorpora metadados EXIF nos ficheiros de saída (GPS, carimbos de data/hora, parâmetros de processamento)
* Organiza a saída em subpastas por modelo de câmara
* Resultados: ficheiros finais no disco

Esta é principalmente uma thread limitada pela E/S. O armazenamento em SSD melhora significativamente o desempenho da Thread 4.

***

## Processamento sequencial vs. em pipeline

### Modo Gratuito (Sequencial)

Na versão gratuita do Chloros, as imagens são processadas **uma de cada vez**, sequencialmente através das quatro etapas:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A barra de progresso da GUI mostra 2 etapas: Detecção de Alvo e Processamento.

### Modo Chloros+ (em pipeline)

Com uma licença Chloros+, todas as quatro threads operam **simultaneamente** em imagens diferentes:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A barra de progresso da GUI mostra 4 etapas: Detecção, Análise, Calibração, Exportação. Passe o cursor sobre a barra de progresso para ver o progresso por thread.

{% hint style="success" %}
**O processamento em pipeline com o Chloros+** pode ser 3 a 5 vezes mais rápido do que o processamento sequencial, dependendo do seu hardware e do tamanho do conjunto de dados. O aumento de velocidade é maior em sistemas com GPUs e SSDs rápidos.
{% endhint %}

***

## Progresso da exportação da Thread 4

No Chloros 1.1.0, a thread de exportação (Thread 4) tem o seu próprio acompanhamento de progresso dedicado. Pode monitorizar o progresso da exportação separadamente:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

O processamento está concluído quando a Thread 4 atinge 100%.

***

## Relação com a Adaptação Dinâmica de Computação

O sistema de [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) afeta principalmente a **Thread 3 (Processamento)**:

* Estratégia **`GPU_PARALLEL`**: a Thread 3 processa várias imagens através da GPU simultaneamente utilizando o pipeline `fused_gpu`
* Estratégia **`GPU_SINGLE`**: O Thread 3 processa uma imagem de cada vez utilizando o pipeline `tiled_gpu`, eficiente em termos de memória
* Estratégia **`CPU_PARALLEL`**: O Thread 3 utiliza processamento baseado na CPU com paralelismo multithread

A alocação de memória da GPU do Thread 3 também muda dinamicamente à medida que os Threads 1 e 2 são concluídos — consulte [Alocação Dinâmica de Memória da GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Próximos Passos

* [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) — Como o Chloros seleciona a estratégia ideal para o seu hardware
* [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamento do pipeline específico da plataforma no Jetson
* [Monitorização do processamento](../processing-images-gui/monitoring-the-processing.md) — Monitorização do progresso na GUI
