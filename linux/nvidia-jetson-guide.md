# Guia do NVIDIA Jetson O

Chloros

no NVIDIA Jetson permite o processamento de imagens multiespectrais na periferia — no terreno, em UAVs e em instalações remotas. OChloros

1.2.0 deteta o seu modelo Jetson no arranque e otimiza a sua estratégia de processamento para o hardware que encontrar. **Não é necessário qualquer ajuste manual.**

***

## Modelos Jetson suportados

| Modelo                | RAM            | Estratégia de processamento                                     | Utilização recomendada                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB partilhados | `GPU_PARALLEL` (2 workers)                              | Desempenho máximo, conjuntos de dados de grande dimensão                      |
| **Jetson Orin NX**   | 8–16 GB partilhados  | `GPU_PARALLEL` (2 workers, 16 GB) / `GPU_SINGLE` (8 GB)   | Recomendação principal para implementação em aeronaves e no terreno |
| **Jetson Orin Nano** | 8 GB partilhados     | `GPU_SINGLE` (1 trabalhador, sequencial)                     | Computação de ponta de nível básico                                 |

{% hint style="info" %}
O pacote arm64 doLinux

requer o **JetPack 6**, que está disponível na família Jetson Orin. Os modelos mais antigos (Nano, TX2, Xavier NX) não podem executar o JetPack 6 e não são suportados pelo pacote atual.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (recomenda-se a versão mais recente)
* **NVIDIA CUDA** (incluída no JetPack)
* **Plano pagoChloros

+** — nível Copper ou superior (necessário para todo o acesso aCLI

/SDK

; aplicado do lado do servidor)

## Instalação

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Para obter detalhes gerais sobre a instalação doLinux

, localizações de ficheiros e resolução de problemas, consulte [Instalação doLinux

](linux-installation.md).

{% hint style="info" %}
**Coloque o diretório de extração num dispositivo de armazenamento rápido.** Os binários compilados descompactam-se automaticamente para um diretório temporário a cada inicialização — o que é extremamente lento a partir de um cartão SD.Chloros

utiliza automaticamente o `/mnt/ssd/tmp`, caso este exista; caso contrário, defina o `TMPDIR` para um caminho no seu NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptação Dinâmica de Computação no Jetson

### Como Funciona

Ao iniciar, oChloros

analisa o seu sistema:

1. **Deteta o modelo Jetson** através de `/proc/device-tree/model`
2. **Lê a memória partilhada disponível da GPU/CPU** (o Jetson utiliza memória unificada)
3. **Seleciona uma estratégia de processamento** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Define automaticamente o número de trabalhadores, o tipo de pipeline e a alocação de memória**A decisão é determinada pela**memória RAM partilhada total**, e não pelo nome do modelo:

* **Com menos de 12 GB de RAM total**(todos os Jetsons de 8 GB): `GPU_SINGLE` com**1 trabalhador — processamento sequencial deliberado**. A memória é insuficiente para trabalhadores simultâneos, pelo que as imagens são processadas uma de cada vez. Nos Jetsons com**8 GB ou menos**, o Thread 3 ignora completamente o conjunto de trabalhadores e executa o seu trabalho por imagem durante o próprio processo.
* **12 GB ou mais**(Orin NX de 16 GB, AGX Orin): a memória unificada cumpre os requisitos para o `GPU_PARALLEL`, mas o número de trabalhadores está**limitado a 2 no Jetson** — a GPU, a RAM dos processos de trabalho e os contextos CUDA por trabalhador recorrem todos ao mesmo conjunto partilhado, pelo que um maior número de trabalhadores aumenta o risco de falhas por falta de memória.

Pode substituir a escolha automática com a variável de ambiente `CHLOROS_STRATEGY` — consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamento por modelo

| Modelo Jetson                | Estratégia       | Trabalhadores | Execução                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Ciclo sequencial no próprio processo (`tiled_gpu` sob pressão de memória) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Ciclo sequencial no próprio processo                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Processos de trabalho simultâneos, `fused_gpu` caminho  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Processos de trabalho simultâneos, caminho `fused_gpu`  |

A principal diferença entre as plataformas é a **memória**. Um Jetson de 8 GB tem de processar imagens uma de cada vez, utilizando uma abordagem em mosaico eficiente em termos de memória quando a carga é elevada, enquanto um Orin de 16 GB ou mais pode processar 2 imagens simultaneamente através da GPU, utilizando o pipeline fundido de maior rendimento.

### Orçamento de GPU por modelo

Cada modelo Jetson possui também um perfil de hardware que limita a quantidade de memória que o processamento do conjunto partilhado pode ocupar e dimensiona os tamanhos dos lotes:

| Modelo | Limite máximo do orçamento de GPU | Multiplicador do tamanho do lote | Reservado para o sistema/ecrã |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

A RAM detetada ajusta o perfil: um Jetson que indique **16 GB ou mais** tem o seu multiplicador de lote aumentado para ×1,2. O tamanho base do lote antes dos multiplicadores é de 8 imagens.

Para consultar a referência completa sobre a adaptação computacional, consulte [Adaptação Computacional Dinâmica](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limite de frequência da GPU para o Texture Aware no Nano e no Orin Nano

O debayer do Texture Aware executa inferência de redes neurais na GPU, o que pode desencadear **avisos de sobrecorrente**em modelos Jetson de baixo consumo (classe de 10-15 W) quando a GPU funciona à velocidade máxima do relógio. Antes do processamento do Texture Aware num**Jetson Nano ou Orin Nano**,Chloros

verifica a frequência máxima da GPU e limita-a a **510 MHz** (510000000) caso esteja atualmente superior:

* Se o comando `CLI

` conseguir escrever no nó sysfs da frequência da GPU, o limite é **aplicado automaticamente** e é exibida uma confirmação.
* Caso contrário (requer direitos de root), oCLI

apresenta o comando exato `sudo` para aplicar o limite manualmente, aguarda um momento para que o possa ler e, em seguida, continua — o processamento continua a decorrer, mas poderá apresentar avisos de sobrecorrente.

Para aplicar o limite manualmente antes do processamento:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Os modelos de maior potência (Orin NX 25 W, AGX Orin 60 W) funcionam à velocidade máxima da GPU; não é aplicado qualquer limite. O «Standard debayer» nunca aciona o limite em nenhum modelo.

{% hint style="info" %}
**O Texture Aware no Jetson processa sempre uma imagem de cada vez.** Cada worker precisaria do seu próprio contexto CUDA (~1 GB), além da sua própria cópia do modelo de redução de ruído, o que a memória unificada não consegue suportar — por isso, no Jetson, o caminho do Texture Aware é fixado a um único worker com acesso à GPU serializado. É de esperar que o Texture Aware seja significativamente mais lento do que o Standard em qualquer Jetson.
{% endhint %}

***

## Gestão Térmica

Os dispositivos Jetson têm uma margem térmica limitada, especialmente em implementações fechadas ou a bordo de aeronaves. O «Chloros

» monitoriza a temperatura do SoC e limita automaticamente o tamanho dos lotes:

| Temperatura         | Ação                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Funcionamento normal — velocidade de processamento total          |
| **70 °C** (Aviso)  | O tamanho do lote é reduzido progressivamente (100% → 50% entre 70 °C e 80 °C) |
| **80 °C** (Crítico) | Redução agressiva (50 % → 0 % entre 80 °C e 90 °C) |
| **90 °C** (Desligamento) | Paragem total do processamento da GPU — é necessário arrefecimento |

{% hint style="warning" %}
**Assegure uma ventilação e dissipação de calor adequadas** para um processamento sustentado, especialmente em caixas de campo fechadas ou em sistemas aéreos. A limitação térmica reduzirá o rendimento do processamento para proteger o hardware.
{% endhint %}

***

## Gestão da memória

Os dispositivos Jetson utilizam **memória unificada** — a GPU e a CPU partilham a mesma RAM física. A VRAM indicada (por exemplo, ~15,3 GB num Orin NX de 16 GB) não é memória dedicada à GPU; é a mesma RAM que o sistema operativo e todos os outros processos estão a utilizar.

### Aviso e recomendações sobre a área de troca

Antes do processamento no Jetson, oCLI

conta as imagens RAW na sua pasta de entrada (`.tif`, `.tiff`, `.raw`, `.dng` — as pré-visualizações em JPG não são contabilizadas), estima o pico de memória necessário para a execução e **avisa antes de iniciar** se for provável que a RAM + swap sejam insuficientes. O aviso tem como título `LOW MEMORY WARNING - Jetson Detected`, apresenta o número de imagens, a RAM, o espaço de troca atual e o pico estimado e, em seguida, fornece os comandos exatos `fallocate` / `chmod` / `mkswap` / `swapon` dimensionados para o seu projeto (nunca inferiores a 8 GB). Faz uma pausa de alguns segundos para que a mensagem não se perca no histórico de rolagem e, em seguida, o processamento continua.**Estimativas de memória utilizadas pelo aviso:**

| Modo de debayer | Base | Por imagem |
| --- | --- | --- |
| Padrão | ~1,5 GB | ~10 MB |
| Sensível à textura | ~2,5 GB (modelo + tempo de execução dePython

) | ~15 MB |

O aviso é acionado quando o pico estimado excede a RAM + swap menos uma margem de segurança de 1 GB, e conta apenas com swap **baseado em ficheiros** — uma configuração apenas com zram continuará a ser sinalizada.

Para adicionar swap manualmente (exemplo: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

### Gestão de OOM (Out of Memory)

Durante o processamento, oChloros

monitoriza a memória da GPU e reduz a performance de forma controlada, em vez de entrar em falha:

1. Quando a utilização da memória da GPU excede **85%**, os tamanhos dos lotes são reduzidos de forma preventiva
2. Se ainda assim ocorrer um evento de falta de memória, o tamanho do lote é **reduzido para metade** e, novamente, para metade em cada OOM consecutivo; cada lote subsequente bem-sucedido recua essa penalização um passo
3. Sob pressão sustentada, o pipeline recorre do `fused_gpu` para o caminho `tiled_gpu`, mais eficiente em termos de memória, e, como último recurso, para o processamento na CPU

***

## Implementação no terreno

### Considerações sobre o consumo de energia

| Modelo Jetson     | Consumo de energia típico | Notas                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Conector cilíndrico CC          |
| Jetson Orin NX   | 10–25 W             | Conector cilíndrico CC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD ou conector cilíndrico |

Planeie o seu orçamento de energia para um processamento sustentado — o consumo máximo de energia ocorre durante o Thread 3 (Processamento), que exige um uso intensivo da GPU.

### Recomendações de armazenamento

* **SSD NVMe** fortemente recomendado para implementações em arm64
* Os cartões SD são demasiado lentos para o processamento — utilize-os apenas como suporte de arranque
* Preveja 2 a 3 vezes o tamanho dos seus dados de imagem brutos para a saída processada

### Operação sem monitor através doSSH



Chloros

CLI

é ideal para implementações do Jetson sem monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend sempre ativo para sincronização temporal do LATTICE / DAQ-E

Se o seu Jetson controlar câmaras LATTICE ou sensores de luz DAQ-E sem monitor, ative o serviço systemd do backend para que o «grandmaster» PTP funcione continuamente (a unidade está instalada, mas não está ativada por predefinição):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Consulte [Instalação doLinux

](linux-installation.md#always-on-ptp-for-headless-hosts) para obter detalhes, incluindo como o pacote torna as portas PTP 319/320 ligáveis sem direitos de root.

### Processamento automatizado com o systemd

Crie um serviço systemd para o processamento automatizado:

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

O `chloros-cli process` termina com um valor diferente de zero quando uma execução que solicitou produtos não grava imagens, pelo que o estado de falha do systemd é relevante para a monitorização.

Combine com um temporizador do systemd para processamento agendado:

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

### Processamento básico no Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

###Python

SDK

no Jetson

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para utilização no terreno

Para implementações no terreno e aéreas, considere estas opções de placas de suporte Jetson Orin NX de 16 GB:

* **Aéreo/drone**: Sistemas com classificação de resistência à vibração (MIL-STD), leves (menos de 300 g) e com refrigeração passiva
* **Condições de campo exigentes**: caixas à prova de água IP67/IP69K com conectividade para câmaras GigE PoE
* **Mínimo/orçamento reduzido**: kits de desenvolvimento com caixas adicionais

Contacte o [Suporte doMAPIR

](https://www.mapir.camera/community/contact) para obter recomendações específicas de hardware para o seu cenário de implementação.

***

## Próximos passos

* [Instalação doLinux

](linux-installation.md) — Detalhes gerais sobre a instalação doLinux


* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Referência completa sobre estratégias de computação
* [Pipeline de processamento](../processing-architecture/processing-pipeline.md) — Compreender o pipeline de 4 threads
* [CLI

: Linha de comandos](../CLI.md) — O guia «CLI

»
* [API

:Python

SDK

](../api-python-sdk.md) — O guia «SDK

»
* [Referência doCLI

](../reference/cli-reference.md) e [Referência doSDK

](../reference/sdk-reference.md) — Listas exaustivas de comandos/API

os para a versão 1.2.0
