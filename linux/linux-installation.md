# Instalação do Linux

O Chloros é distribuído para o Linux como pacotes `.deb` que instalam o CLI e o servidor backend. O Python SDK é um pacote pip separado (também incluído no `.deb` como um wheel com versão compatível).

Os nomes dos ficheiros dos pacotes incluem a versão e a arquitetura: `chloros_1.2.0_amd64.deb` para x86_64 e `chloros_1.2.0_arm64_jp6.deb` para as compilações do JetPack 6 para Jetson. Substitua o ficheiro que descarregou efetivamente nos comandos abaixo.

***

## Linux amd64 (x86_64)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribuição** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processador** | x86_64 (Intel/AMD) | Intel Core i7 ou superior |
| **Memória (RAM)** | 8 GB | 16 GB ou mais |
| **Placa gráfica** | Nenhuma (processamento pela CPU) | GPU NVIDIA com 4 GB ou mais de VRAM (12 GB ou mais desbloqueia o `GPU_PARALLEL`; 7 GB ou mais mantém o Texture Aware desativado no percurso de imagem única) |
| **Armazenamento** | 2 GB de espaço livre | SSD com 10 GB ou mais de espaço livre |
| **Python** | Python 3.7 ou superior (para o SDK) | Python 3.10+ |

> **O Ubuntu 20.04 e o Debian 11 não são suportados.** A lista de dependências do `.deb` é
> derivada do que o backend Chloros efetivamente liga, e isso inclui
> o `libc6 (>= 2.34)`. Tanto o Focal como o Bullseye vêm com a glibc 2.31, pelo que o `apt` recusa a
> instalação de imediato, em vez de permitir que esta falhe mais tarde, em tempo de execução.

### Instalação

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
O `dpkg -i` não resolve as dependências. Se indicar que faltam pacotes, o `sudo apt-get install -f` (ou o `sudo apt --fix-broken install`) conclui a instalação — este é o procedimento normal, não um erro.
{% endhint %}

Verifique a instalação:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson com JetPack 6 | Jetson Orin NX de 16 GB ou AGX Orin |
| **JetPack** | JetPack 6.x | Versão mais recente do JetPack 6 |
| **Memória (RAM)** | 8 GB (partilhada entre GPU e CPU) | 16 GB ou mais partilhados (12 GB ou mais é o limite mínimo para trabalhadores paralelos da GPU) |
| **Armazenamento** | 2 GB de espaço livre | SSD NVMe com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para o SDK) | Python 3.10+ |

### Instalação

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

O mesmo layout que o amd64 `.deb`, com uma compilação CUDA otimizada para o Jetson Orin / Orin NX / Orin Nano. Para obter informações sobre o comportamento da memória, térmico e de implementação em campo do Jetson, consulte o [Guia NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalação do Python e do SDK (Todos os Linux)

O SDK é um cliente puro Python HTTP para o backend, pelo que o mesmo pacote funciona em amd64 e arm64. Duas fontes:**Do PyPI** — a versão estável publicada:

```bash
pip install chloros-sdk
```

**Do ficheiro wheel incluído** — garantidamente compatível com o CLI/backend que acabou de instalar (utilize esta opção quando o seu `.deb` for mais recente do que o do PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**As distribuições PEP 668** (Ubuntu 23.10+, Debian 12+) recusam instalações do pip a nível do sistema. Utilize o `pip install --user …`, um ambiente virtual ou o `sudo pip install --break-system-packages …`. O instalador de pacotes nunca instala automaticamente o SDK no seu sistema Python — essa escolha fica ao seu critério.
{% endhint %}

Extras opcionais:

| Extra | Comando | Adiciona |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para transmissão em tempo real do progresso |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Verifique o SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
O `.deb` instala o Chloros, o CLI e o backend. O Python e o SDK comunicam com esse backend através de uma rede local HTTP e API (`http://127.0.0.1:5000`) e inicia-o automaticamente quando necessário. Utilize sempre o endereço IPv4 literal em vez de `localhost` — o `localhost` pode ser resolvido para o `::1` e demorar cerca de dois segundos por pedido.
{% endhint %}

***

## Configuração inicial

### 1. Iniciar sessão

O acesso a CLI e SDK requer um plano pago Chloros+ (**Copper** ou superior), aplicada do lado do servidor: um utilizador que tenha saído da sessão recebe o `401 AUTH_REQUIRED`, e um utilizador do plano gratuito (Iron) recebe o `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

As credenciais são armazenadas em cache em `~/.chloros/user_session.json`.

{% hint style="warning" %}
**É necessário iniciar sessão novamente após cada instalação ou atualização.** O script `prerm` do pacote limpa deliberadamente o `~/.chloros/user_session.json` e a licença armazenada em cache para todos os utilizadores do computador, para que uma nova compilação revalide sempre a licença, em vez de confiar num cache desatualizado.
{% endhint %}

### 2. Verifique o estado da sua licença

```bash
chloros-cli status
```

O `chloros-cli status` funciona em qualquer nível (incluindo o gratuito), pelo que pode sempre verificar por que motivo o acesso está ou não disponível.

### 3. Execute o diagnóstico do sistema

```bash
chloros-cli selftest
```

São executadas sete verificações em sequência, e o comando termina com um valor diferente de zero se alguma delas falhar:

| # | Verificação | O que comprova |
| --- | --- | --- |
| 1 | **Versão** | O CLI indica a sua versão (`v1.2.0`). |
| 2 | **Porta disponível** | A porta 5000 está livre, *ou* já foi respondida por um backend Chloros em bom estado (o que conta como aprovado). |
| 3 | **Arranque do backend** | O binário do backend é iniciado. |
| 4 | **Teste do API (`/api/test`)** | O backend responde com `status: ok`. |
| 5 | **Informações do sistema** | Exibe `GPU: <name>, CUDA: <bool>, PyTorch: <version>` a partir de `/api/system-info`. |
| 6 | **Modelos do Denoiser** | Encontra modelos `*.pth.enc` (em Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Redutor de ruído**| O Texture Aware é efetivamente utilizável — requer CUDA**e**, pelo menos, um ficheiro de modelo. |

A execução termina com o `N/7 checks passed`, listando quaisquer falhas por nome.

### 4. Processar o seu primeiro conjunto de dados

```bash
chloros-cli process ~/datasets/flight001
```

***

## Ficheiros e diretórios

### Por utilizador

O Chloros guarda as suas credenciais e a configuração do CLI num único diretório multiplataforma, **`~/.chloros/`** (no Windows, `%USERPROFILE%\.chloros\`). Dois caches específicos do Linux seguem, em vez disso, as convenções XDG — estes respeitam as definições do `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando definidas.

| Caminho | Finalidade |
| --- | --- |
| `~/.chloros/user_session.json` | Cache da sessão de início de sessão gravada pelo `chloros-cli login` (limpada a cada instalação/atualização de pacote) |
| `~/.chloros/working_directory.txt` | Substituição da pasta de projeto predefinida (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferência de idioma do CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuração de idioma partilhada com a interface gráfica do Windows — um `language` aqui tem prioridade sobre o `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de uma hora para a verificação de atualizações de arranque do Linux/Jetson |
| `~/.chloros/backend.log` | Registo do backend quando este foi iniciado pelo CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacotes de calibração LATTICE por câmara armazenados em cache, indexados pelo número de série e pelo hash do pacote |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Substituições opcionais do utilizador para perfis de correção de limite de DAQ |
| `~/.config/chloros/system_config.json` | Perfil de hardware armazenado em cache proveniente da Adaptação Dinâmica de Computação — elimine-o para forçar uma nova deteção de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Registos do servidor backend, um ficheiro por início |
| `~/Chloros Projects/` | Pasta de projeto predefinida quando não está definida qualquer substituição |

### A nível do sistema

| Caminho | Finalidade |
| --- | --- |
| `/usr/bin/chloros-cli` | Script de envoltório — define `LD_LIBRARY_PATH` para as bibliotecas nativas incluídas no pacote e, em seguida, executa o binário real |
| `/usr/bin/chloros-backend` | Script de envoltório — o mesmo, mais `CHLOROS_PRODUCTION=1` para que o mecanismo de autenticação do backend nunca se desative silenciosamente |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Os binários compilados |
| `/usr/lib/chloros/arena_runtime/` | Ambiente de execução Arena SDK necessário para as câmaras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos de redução de ruído encriptados utilizados pelo debayer Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK roda correspondente a esta compilação exata |
| `/usr/lib/chloros/exiftool` | exiftool incluído (com ligação simbólica para o `/usr/local/bin/exiftool` apenas se não existir um exiftool no sistema) |
| `/etc/chloros/update.conf` | Configuração do canal de atualização lida pelo `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Define o `net.ipv4.ip_unprivileged_port_start = 319` para que o backend possa estabelecer ligação às portas PTP sem direitos de root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Aponta o carregador dinâmico para o `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede ao utilizador autenticado acesso à ponte série USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Serviço de backend sempre ativo (instalado, **não ativado**) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada no menu da aplicação «Chloros CLI» que abre um terminal |

## Localização do executável do backend

O CLI e o SDK detetam automaticamente o backend:

| Componente | Caminho |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Substitua o caminho do backend com o sinalizador `--backend-exe` CLI ou o parâmetro do construtor `backend_exe` SDK, e a porta com `--port` (por predefinição, `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` aponta para as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** num backend remoto. Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoram-no deliberadamente e têm sempre como alvo o `http://127.0.0.1:<port>`.
{% endhint %}

***

## Câmaras LATTICE e sensores de luz DAQ no Linux

Todas as famílias de comandos de hardware em tempo real funcionam no Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — detetar, ligar, configurar e capturar imagens a partir de câmaras LATTICE e matrizes sincronizadas. O `.deb` inclui o runtime Arena SDK necessário e regista-o no carregador dinâmico.
* **`chloros-cli daq pool-*`** — ligar sensores de luz DAQ-U/M/E através do conjunto de backends, transmitir espectros calibrados e gravar ficheiros `.daq`. O CLI compilado inclui apenas a família `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — executa um projeto guardado (as suas câmaras, sensores e definições de processamento) sem interface gráfica.
* **`chloros-cli time-sync`** — inspecionar o «grandmaster» PTP em que o backend Chloros é executado para câmaras LATTICE e sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

O `--sensor-id` é pré-requisito para o `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; o `pool-list` mostra os IDs atualmente no conjunto.

{% hint style="info" %}
**Dê preferência ao `--eth-host` para a primeira ligação do DAQ-E numa máquina com múltiplas ligações de rede.** A deteção automática pesquisa o mDNS e pode não encontrar a interface do sensor devido a uma cache ARP vazia; por isso, o primeiro `pool-connect --eth` após o arranque pode falhar, mesmo que o sensor esteja em perfeitas condições. Ao passar o IP ou o nome de anfitrião do sensor, a deteção é totalmente ignorada.
{% endhint %}

**As permissões seriais do DAQ-U** são geridas pela regra udev instalada (`uaccess` + grupo `dialout`). Se um sensor que já estava ligado permanecer inacessível, recarregue as regras ou volte a ligá-lo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulte a [referência CLI](../CLI.md) para obter o conjunto completo de comandos.

### PTP sempre ativo para anfitriões sem monitor

Na primeira instalação, a unidade systemd `chloros-backend.service` é gerada, mas **não está ativada**. Num Jetson sem monitor ou num servidor que deva manter a sincronização de tempo PTP a funcionar continuamente para sensores DAQ-E e câmaras LATTICE, ative-a:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Sem isso, o PTP só funciona enquanto o backend Chloros estiver em execução — ou seja, durante uma sessão ativa de CLI/SDK.

A unidade associa o backend a `127.0.0.1:5000` (configurações de ambiente `CHLOROS_HOST` / `CHLOROS_PORT` no interior da unidade; substituir por `sudo systemctl edit chloros-backend.service`) e reinicia-o em caso de falha após 5 segundos.

**Como o PTP obtém as suas portas.** O PTP utiliza as portas UDP 319/320, ambas abaixo do limite normal de 1024 para portas privilegiadas. O `postinst` do pacote grava o `/etc/sysctl.d/60-chloros-ptp.conf` com o `net.ipv4.ip_unprivileged_port_start = 319`, o que permite que o backend as vincule enquanto é executado como o seu utilizador. Além disso, aplica o `setcap cap_net_bind_service,cap_net_raw=+ep` ao binário do backend como medida de segurança — é por isso que o `libcap2-bin` é uma dependência declarada do pacote.***

## Exemplos de scripts em Bash

{% hint style="info" %}
**Códigos de saída compatíveis com scripts.**O `chloros-cli process` sai com o código `0` em caso de sucesso e**com um valor diferente de zero em caso de falha — incluindo uma execução que solicitou produtos de imagem mas não gravou nenhum** (imprime o código `Processing finished but wrote no image products.` e indica o nome da pasta do projeto e as causas habituais). As execuções bem-sucedidas indicam quantos produtos de imagem foram gravados (`Image products written: N`). Códigos de saída: `0` sucesso, `1` falha, `2` erro de argumento, `130` interrompido.
{% endhint %}

### Processar vários conjuntos de dados

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Processar com definições personalizadas

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Os valores válidos para o `--format` são exatamente quatro e contêm espaços — coloque-os sempre entre aspas:

| Valor do `--format` | Pasta de saída |
| --- | --- |
| `TIFF (16-bit)` *(predefinição)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

O `--debayer` aceita o `standard` (padrão) ou o `texture-aware` (Chloros+).

### Processamento automatizado com o Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplo de Python e SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Resolução de problemas

### CLI não encontrado após a instalação

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Permissão negada

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «setcap falhou» durante a instalação

O `.deb` aplica o `cap_net_bind_service` ao `/usr/lib/chloros/chloros-backend` para que este possa ligar-se às portas PTP 319/320 sem direitos de root. Se o `libcap2-bin` não estiver presente no momento da instalação, a chamada é ignorada. Instale-o e reinstale o pacote:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### O PTP não inicia / Não é possível ligar à porta 319

Confirme se o limite mínimo de portas sem privilégios foi reduzido e, caso não tenha sido, reaplique-o para o arranque atual:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Em seguida, verifique o grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «Não foram encontrados controladores de câmara LATTICE»

O tempo de execução do Arena SDK não está a ser resolvido. Confirme se a configuração do carregador que o pacote escreve está presente e atualizada:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Falha no arranque do backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Os registos do backend relativos ao arranque falhado encontram-se em `~/.cache/chloros/logs/`.

### CUDA não detetada

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

O ficheiro `chloros-cli selftest` relata o mesmo num único registo: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Falta de bibliotecas partilhadas

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Arranque lento em sistemas com cartão SD

Os binários compilados extraem-se para um diretório temporário a cada inicialização. Se o `/mnt/ssd/tmp` existir, o Chloros utiliza-o automaticamente; caso contrário, defina o `TMPDIR` para um sistema de ficheiros rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Atualização do Chloros no Linux

O comando `update` é exclusivo do Linux/Jetson. Verifica a versão publicada no canal de atualização configurado em `/etc/chloros/update.conf` e oferece a possibilidade de descarregar e instalar o `.deb` correspondente:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

No Linux/Jetson, o CLI também executa uma verificação de atualizações não bloqueante em cada arranque (resultado armazenado em cache durante uma hora no `~/.chloros/update_cache.json`) e apresenta o `Update available: vX.Y.Z` quando existe uma versão mais recente. As suas definições e projetos são mantidos após a atualização; terá de iniciar sessão novamente a seguir.

## Desinstalação

```bash
sudo apt remove chloros
```

A remoção interrompe o `chloros-backend.service`, restaura o limite mínimo predefinido para portas sem privilégios (1024), remove o link simbólico do exiftool incluído e a configuração do carregador Arena, e limpa as credenciais armazenadas em cache. Os seus projetos e ficheiros de dados `~/.chloros/` não são afetados.

***

## Próximos passos

* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — otimização e implementação específicas para o Jetson
* [CLI: Linha de comandos](../CLI.md) — o guia CLI
* [API: Python SDK](../api-python-sdk.md) — o guia SDK
* [Referência do CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas exaustivas de comandos/API para a versão 1.2.0
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware
