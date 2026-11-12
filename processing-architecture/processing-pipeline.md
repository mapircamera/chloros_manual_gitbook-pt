# Pipeline de processamento O

Chloros 1.3.0 utiliza um pipeline de processamento de 4 threads que funciona como uma linha de montagem por etapas. Cada thread trata de uma fase distinta do fluxo de trabalho, pelo que várias imagens podem estar em processamento em diferentes etapas ao mesmo tempo.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arquitetura do pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Cada imagem passa por todas as quatro threads por ordem. Com o processamento multithread do Chloros+, várias imagens ocupam diferentes threads simultaneamente — enquanto a Thread 3 processa uma imagem, a Thread 1 pode estar a detetar a seguinte, a Thread 2 a calibrar outra e a Thread 4 a gravar uma imagem concluída no disco.

O progresso é relatado por thread e transmitido através de Server-Sent Events (o backend publica-os no `/api/events`). Na visualização do progresso em tempo real da CLI, as quatro fases são identificadas como **Detecção, Análise, Processamento, Exportação**.

***

## Detalhes das threads

### Thread 1: Detecção

**Objetivo**: Carregar imagens e detetar alvos de calibração.

* Lê ficheiros de imagem do disco — pares Survey3 `.raw`+`.jpg`, capturas LATTICE `.tif`/`.tiff` e `.dng`
* Extrai metadados EXIF (GPS, modelo da câmara, registos de data e hora, exposição)
* Deteta alvos de calibração: geometrias de alvos marcados com ArUco para capturas LATTICE e o detetor de painel clássico para fotografias de alvos de calibração do Survey3
* Resultados: dados de imagem + metadados + resultados da deteção de alvos

Trata-se principalmente de um segmento limitado pela E/S e pela CPU.

### Thread 2: Calibração

**Objetivo**: Calcular os parâmetros de calibração a partir dos alvos detetados.

* Calcula os coeficientes de calibração de refletância a partir das imagens dos alvos
* Calcula os parâmetros de correção de vinheta
* Determina as curvas de calibração por banda
* Resultados: parâmetros de calibração para cada imagem

Um segmento de computação limitado pela CPU. Numa execução do Survey3 ou mista, o Segmento 3 aguarda este segmento quando a calibração de refletância está ativada, para que os seus coeficientes estejam prontos antes de qualquer imagem ser processada. Numa execução apenas do «LATTICE», o Segmento 3 é executado durante a deteção: cada fotograma inicia assim que a sua âncora de alvo estiver finalizada. Um fotograma considera o alvo mais próximo no tempo, em qualquer dos lados, pelo que aguarda até que todos os fotogramas da sua câmara mais próximos no tempo do que o seu alvo mais próximo atual tenham sido digitalizados. Os produtos são os mesmos, apenas mais cedo.

### Thread 3: Processamento (GPU)

**Objetivo**: Aplicar correções e calcular índices de vegetação.

**Esta é a thread que exige maior capacidade de computação.**

* **Debayering**: converte dados RAW Bayer em imagens multicanal
  * Padrão (Rápido, Qualidade Média) — predefinição, `--debayer standard`
  * Sensível à Textura (Lento, Qualidade Máxima) — apenas Chloros+, `--debayer texture-aware`, utiliza um modelo de redução de ruído baseado em IA/ML
  * As capturas LATTICE mono (M3M) são de banda única: as etapas de demosaico e equilíbrio de brancos são ignoradas para estas (com uma mensagem de registo de uma linha), enquanto quaisquer imagens M3C/Bayer na mesma execução continuam a ser submetidas a essas etapas
* **Correção de vinheta**: aplica a correção de vinheta da lente em toda a imagem
* **Calibração de refletância**: aplica coeficientes de calibração para converter para valores de refletância
* **Cálculo de índices**: calcula índices de vegetação (NDVI, NDRE, GNDVI, …)
* Resultados: dados de imagem processados, prontos para exportação

Esta thread é a que mais beneficia da aceleração por GPU, e é a thread que a [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) ajusta.

### Thread 4: Exportação

**Objetivo**: Gravar as imagens processadas no disco.

* Grava os ficheiros de saída no formato selecionado — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Incorpora metadados nos ficheiros de saída (GPS, carimbos de data/hora, parâmetros de processamento) — incluindo as etiquetas de processamento `Chloros:*`, que, a partir da versão 1.3.0, são gravadas em todos os modos de processamento, tanto para o Survey3 como para o LATTICE
* Organiza os ficheiros de saída na pasta do projeto como `<camera>/<format>/<Product>_Images/` — por exemplo, `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Os ficheiros exportados mantêm o nome do ficheiro de origem; a pasta identifica o produto.**
* Para capturas do LATTICE, um fotograma de origem pode dar origem a vários produtos (Debayered, Preview, Radiance, Reflectance, Index), cada um na sua própria pasta de produtos. Os produtos Index são calculados a partir do produto de refletância gerado pela execução (quando não existe nenhum, a partir de outro produto gerado pela execução, como o de radiação). Os produtos de radiação e refletância das câmaras multiespectrais M3C também podem obter uma máscara de limitação por pixel em `<format>/clamp_masks/` (ver [Formatos de Imagem de Saída](../output-image-formats.md#clamp-masks))
* Saídas: ficheiros finais no disco

Trata-se principalmente de um thread limitado pela E/S — o armazenamento em SSD melhora significativamente o seu desempenho.

***

## Por trás dos bastidores: Executores

No Thread 3, o trabalho por imagem é paralelizado da seguinte forma:

* **Estratégias de GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilizam um conjunto de**processos de trabalho**, cada um deles um processo separado do sistema operativo com o seu próprio contexto CUDA. O Chloros inicia cada trabalhador lançando novamente o seu próprio programa backend instalado — e não através do `multiprocessing` do Python, cujo arranque do trabalhador não funciona dentro da aplicação empacotada
* **`CPU_PARALLEL`** utiliza um conjunto de threads (o `ThreadPoolExecutor` do Python) — o NumPy e o OpenCV libertam o GIL, pelo que as threads são suficientes
* Os dispositivos Jetson com 8 GB ou menos de RAM partilhada ignoram completamente o executor e processam no próprio processo, de forma sequencial
* O Texture Aware numa GPU com menos de 7 GB de VRAM também é executado sequencialmente — o modelo de redução de ruído não cabe mais do que uma vez O

Chloros não utiliza nenhuma estrutura distribuída de terceiros (como o Ray). Consulte [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) para saber como a estratégia e o número de trabalhadores são escolhidos.

***

## Processamento sequencial vs. em pipeline

### Modo Livre (Sequencial)

Na versão gratuita do Chloros, as imagens são processadas **uma de cada vez**, sequencialmente ao longo das quatro fases:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A interface gráfica apresenta uma barra de progresso simplificada no modo gratuito; as suas fases sequenciais são indicadas como **Detecção de Alvos**e, em seguida,**Processamento**.

### Modo Chloros+ (em pipeline)

Com uma licença Chloros+, as quatro threads operam **simultaneamente** em imagens diferentes:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A barra de progresso da GUI mostra as quatro etapas; passe o cursor por cima dela para ver o progresso por thread. Na CLI, as mesmas quatro etapas são apresentadas em tempo real como **Detecção, Análise, Processamento, Exportação**.

{% hint style="info" %}
**Um rótulo, dois nomes.** A CLI denomina a fase 3 de _Processamento_. O feed de progresso do modo premium do backend — aquele que a barra de progresso da GUI apresenta — rotula a mesma fase como _Calibração_. Trata-se da mesma thread a realizar o mesmo trabalho (Thread 3: debayer, correções, índices).
{% endhint %}

{% hint style="success" %}
**O processamento em pipeline com o Chloros+** pode ser 3 a 5 vezes mais rápido do que o processamento sequencial, dependendo do seu hardware e do tamanho do conjunto de dados. O aumento de velocidade é maior em sistemas com GPUs e SSDs rápidos.
{% endhint %}

***

## Progresso da exportação da Thread 4

A thread de exportação tem o seu próprio acompanhamento de progresso, que pode consultar separadamente:**Interface de CLI**:

```bash
chloros-cli export-status
```

**SDK**:

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

O processamento está concluído quando a Thread 4 atinge 100%.

{% hint style="info" %}
**Uma execução que não grava imagens é considerada um falhanço.**Em caso de sucesso, o `chloros-cli process` indica quantos produtos de imagem foram gravados (`Image products written: N`). Se foram solicitados produtos e**nenhum**foi gravado — apenas o `project.json` e o `calibration_data.json` — a CLI apresenta o `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, indicando o nome da pasta do projeto e as causas habituais (a pasta de entrada não foi reconhecida como uma captura — verifique o layout e o `--input-level` — ou todos os produtos solicitados eram inaplicáveis a essas câmaras). Os scripts podem basear-se no código de saída.
{% endhint %}

***

## Relação com a Adaptação Dinâmica de Computação

A [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) afeta principalmente a **Thread 3 (Processamento)**:

* **`GPU_PARALLEL`**: A Thread 3 processa várias imagens através da GPU simultaneamente, utilizando o pipeline `fused_gpu`
* **`GPU_SINGLE`**: A Thread 3 serializa o acesso à GPU com um semáforo enquanto os processos de trabalho sobrepõem as operações de E/S, utilizando o `fused_gpu` ou o pipeline `tiled_gpu`, eficiente em termos de memória
* **`CPU_PARALLEL`**: O thread 3 utiliza processamento baseado na CPU com paralelismo multithread

A alocação de memória da GPU da Thread 3 também aumenta à medida que as Threads 1 e 2 terminam — consulte [Alocação Dinâmica de Memória da GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Próximos passos

* [Adaptação dinâmica de computação](dynamic-compute-adaptation.md) — Como o Chloros seleciona a estratégia ideal para o seu hardware
* [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamento do pipeline específico da plataforma no Jetson
* [Monitorização do processamento](../processing-images-gui/monitoring-the-processing.md) — Monitorização do progresso na GUI
* [Referência da CLI](../reference/cli-reference.md) — `process`, `export-status`, códigos de saída e formato da saída
