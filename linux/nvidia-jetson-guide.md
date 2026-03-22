# Guia do NVIDIA Jetson

XPROTX000047 O XPROTX no NVIDIA Jetson permite o processamento de imagens multiespectrais na periferia — no terreno, em UAVs e em instalações remotas. XPROTX000048 O XPROTX deteta automaticamente o seu modelo Jetson e otimiza a estratégia de processamento para o seu hardware.

***

## Modelos Jetson suportados

| Modelo                | RAM            | Estratégia de processamento                                   | Utilização recomendada                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB partilhados | `GPU_PARALLEL` (4 trabalhadores)                            | Desempenho máximo, grandes conjuntos de dados                      |
| **Jetson Orin NX**   | 8-16 GB partilhada  | `GPU_PARALLEL` (3 trabalhadores, 16 GB) / `GPU_SINGLE` (8 GB) | Recomendação principal para implantação aérea e no terreno |
| **Jetson Orin Nano** | 8 GB partilhados     | `GPU_SINGLE` (1 trabalhador)                               | Computação de ponta de nível básico                                 |
| **Jetson Nano**      | 4-8 GB partilhados   | `GPU_SINGLE` (1 trabalhador)                               | Nível básico, com memória limitada                          |

{% hint style="info" %}
**Modelos Jetson antigos** (TX2, TX1, Xavier NX) podem não ser suportados. O desempenho irá variar consoante a memória GPU disponível e as capacidades CUDA.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (recomenda-se a versão mais recente)
* **NVIDIA CUDA** (incluída no JetPack)
* **Licença Chloros+** (necessária para acesso a CLI/SDK)

## Instalação

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Para obter detalhes gerais sobre a instalação do Linux, consulte [Instalação do Linux](linux-installation.md).

***

## Adaptação dinâmica de computação no Jetson

O Chloros deteta automaticamente o seu modelo Jetson e seleciona a estratégia de processamento ideal. **Não é necessário qualquer ajuste manual.**

### Como funciona

No arranque, o Chloros cria um perfil do seu sistema:

1. **Deteta o modelo Jetson** através do `/proc/device-tree/model`
2. **Lê a GPU disponível/memória partilhada**

3.**Seleciona uma estratégia de processamento** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Define automaticamente o número de trabalhadores, o tipo de pipeline e a alocação de memória**

### Comportamento por modelo

| Modelo Jetson                | Estratégia       | Trabalhadores | Pipeline                       | Concorrência |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (eficiência de memória) | Serializado  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (caminho completo da GPU)    | Concorrente  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concorrente  |

{% hint style="success" %}
O **Jetson Orin NX 16 GB** é a escolha ideal para implementação na periferia — recebe a estratégia `GPU_PARALLEL` com 3 trabalhadores simultâneos, proporcionando processamento paralelo real da GPU num formato compacto.
{% endhint %}

A principal diferença entre as plataformas é a **memória**. Um Jetson Nano com 8 GB de memória partilhada tem de processar imagens uma de cada vez utilizando uma abordagem em mosaico eficiente em termos de memória, enquanto um Orin NX com 16 GB pode processar 3 imagens através da GPU simultaneamente utilizando o pipeline fundido de maior rendimento.

Para a referência completa sobre adaptação computacional, consulte [Adaptação Computacional Dinâmica](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gestão térmica

Os dispositivos Jetson têm uma margem térmica limitada, especialmente em implementações fechadas ou aéreas. O Chloros inclui monitorização térmica automática e limitação de desempenho:

| Temperatura         | Ação                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcionamento normal — velocidade de processamento total          |
| **70 °C** (Aviso)  | Reduzir automaticamente o tamanho do lote                   |
| **80 °C** (Crítico) | Limitação agressiva — menor simultaneidade         |
| **90 °C** (Desligamento) | Interromper totalmente o processamento da GPU — é necessário arrefecimento |

{% hint style="warning" %}
**Assegure ventilação e dissipação de calor adequadas** para um processamento sustentado, especialmente em caixas de campo fechadas ou sistemas aéreos. A limitação térmica reduzirá o rendimento do processamento para proteger o hardware.
{% endhint %}

***

## Gestão de memória

Os dispositivos Jetson utilizam **memória unificada** — a GPU e a CPU partilham a mesma RAM física. Isto significa que a VRAM indicada (por exemplo, 15,3 GB no Orin NX de 16 GB) não é memória dedicada à GPU; é partilhada com o sistema operativo e outros processos.

### Recomendações de swap

Para conjuntos de dados grandes ou processamento de debayer com reconhecimento de textura, o Chloros pode recomendar a criação de espaço de swap:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Estimativas de memória por imagem:**

* Debayer padrão: ~10 MB por imagem
* Debayer com reconhecimento de textura: ~15 MB por imagem

O Chloros calcula automaticamente a memória necessária com base no tamanho do seu conjunto de dados e avisa-o se for recomendado o uso de swap.

### Fallback OOM (Out of Memory)

Se for detetada uma condição de falta de memória durante o processamento:

1. O Chloros reduz automaticamente o número de trabalhadores da GPU
2. Recorre do pipeline `fused_gpu` para o `tiled_gpu` (mais eficiente em termos de memória)
3. Continua o processamento com um rendimento reduzido em vez de entrar em falha

***

## Implementação no terreno

### Considerações sobre alimentação

| Modelo Jetson     | Consumo típico de energia | Notas                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C ou conector cilíndrico    |
| Jetson Orin Nano | 7-15 W              | Conector cilíndrico CC          |
| Jetson Orin NX   | 10-25 W             | Conector cilíndrico CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ou conector cilíndrico |

Planeie o seu orçamento de energia para processamento sustentado — o consumo máximo de energia ocorre durante o Thread 3 (Processamento), que exige uso intensivo da GPU.

### Recomendações de armazenamento

* **SSD NVMe** fortemente recomendado para implementações arm64
* Os cartões SD são demasiado lentos para processamento — use-os apenas como mídia de arranque
* Planeie 2 a 3 vezes o tamanho dos seus dados de imagem brutos para a saída processada

### Operação sem monitor através do SSH

O Chloros e o CLI são ideais para implementações Jetson sem monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Processamento automatizado com systemd

Crie um serviço systemd para processamento automatizado:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Combine com um temporizador systemd para processamento agendado:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Exemplos de fluxos de trabalho

### Processamento básico do Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK no Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Processamento em lote de vários voos

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para utilização no terreno

Para implementações no terreno e aéreas, considere estas opções de placas de suporte Jetson Orin NX de 16 GB:

* **Aéreo/drone**: Sistemas com classificação de resistência à vibração (MIL-STD), leves (menos de 300 g), arrefecimento passivo
* **Campo robusto**: Caixas à prova de água IP67/IP69K com conectividade para câmaras PoE GigE
* **Mínimo/orçamento**: Kits de desenvolvimento com caixas adicionais

Contacte o [Suporte MAPIR](https://www.mapir.camera/community/contact) para obter recomendações específicas de hardware para o seu cenário de implementação.

***

## Próximos passos

* [Linux Instalação](linux-installation.md) — Detalhes gerais da instalação do Linux
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Referência completa da estratégia de computação
* [Pipeline de processamento](../processing-architecture/processing-pipeline.md) — Compreender o pipeline de 4 threads
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
* [API : Python SDK](../api-python-sdk.md) — Referência completa do SDK
