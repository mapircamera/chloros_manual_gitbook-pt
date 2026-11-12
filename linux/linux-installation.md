# Instalação no Linux

O Chloros é distribuído para o Linux sob a forma de pacotes `.deb`, que instalam a CLI, o servidor backend e o SDK do Python (para o sistema `python3`, além de um wheel compatível com a versão para virtualenvs).

Os nomes dos ficheiros dos pacotes incluem a versão e a arquitetura: `chloros_1.3.0_amd64.deb` para x86_64, e `chloros_1.3.0_arm64_jp6.deb` para arm64 (NVIDIA Jetson, JetPack 6). Substitua o ficheiro que descarregou efetivamente nos comandos abaixo.

{% hint style="info" %}
**Novidade na versão 1.3.0: os programas são fornecidos descompactados.** O backend e a CLI são instalados como pastas, `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`, em vez de ficheiros auto-extraíveis individuais. Já não se descompactam automaticamente numa pasta temporária a cada arranque, pelo que iniciam muito mais rapidamente (no x86_64, o backend demorava cerca de 100 segundos a descompactar-se a cada arranque) e já não necessitam de vários GB de espaço livre na pasta `/tmp`. A versão amd64, `.deb`, também tem cerca de metade do tamanho anterior.
{% endhint %}

***

## Linux amd64 (x86_64)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribuição** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processador** | x86_64 (Intel/AMD) | Intel Core i7 ou superior |
| **Memória (RAM)** | 8 GB | 16 GB ou mais |
| **Placa gráfica** | Nenhuma (processamento pela CPU) | GPU NVIDIA com 4 GB ou mais de VRAM (12 GB ou mais desbloqueia o `GPU_PARALLEL`; 7 GB ou mais mantém o Texture Aware fora do percurso de imagem única) |
| **Armazenamento** | 2 GB de espaço livre | SSD com 10 GB ou mais de espaço livre |
| **Python** | Python 3.7 ou superior (para o SDK) | Python 3.10 ou superior |

> **O Ubuntu 20.04 e o Debian 11 não são suportados.** A lista de dependências do `.deb` é
> derivada daquilo a que o backend Chloros está efetivamente ligado, e isso inclui um
> `libc6` mais recente do que a glibc 2.31 fornecida pelas versões «focal» e «bullseye», pelo que o `apt` recusa a
> instalação de imediato, em vez de permitir que esta falhe mais tarde, em tempo de execução.

### Instalação

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Utilize o `apt install ./…`, e não o `dpkg -i`.** O `apt` resolve as dependências do pacote e também instala o que recomenda: os pré-requisitos do Python SDK, `python3-numpy` e `python3-requests`. O `dpkg -i` não instala nenhum dos dois. A CLI e o backend são autónomos e funcionam sem eles, mas o `import chloros_sdk` precisa deles; a saída da instalação indica quaisquer que estejam em falta. Mantenha o `./`, para que o `apt` instale o ficheiro em vez de procurar um pacote com esse nome. O pacote também declara o `libgl1`, que o OpenCV incluído carrega e que pode faltar numa imagem mínima do servidor.
{% endhint %}

Verifique a instalação:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson com JetPack 6 | Jetson Orin NX de 16 GB ou AGX Orin |
| **JetPack** | JetPack 6.x | Versão mais recente do JetPack 6 |
| **Memória (RAM)** | 8 GB (partilhada entre GPU e CPU) | 16 GB+ partilhados (12 GB+ é o limite mínimo para processamento paralelo) |
| **Armazenamento** | 2 GB de espaço livre | SSD NVMe com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para o SDK) | Python 3.10+ |

### Instalação

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

O mesmo layout que o amd64 `.deb` (Python SDK, buffers de receção e serviços de backend incluídos), para Jetson Orin / Orin NX / Orin Nano no **JetPack 6** (L4T R36). O seu PyTorch é a compilação CUDA do JetPack 6 para o Orin, pelo que o processamento na GPU, incluindo o debayer Texture Aware, é executado na GPU do Jetson. O pacote não inclui o CUDA propriamente dito: utiliza as bibliotecas CUDA e cuDNN que o JetPack 6 instala (`sudo apt install nvidia-jetpack` numa imagem que não as possua). Quando não é possível utilizar uma GPU CUDA, o processamento é executado na CPU e o `--debayer texture-aware` utiliza o debayer padrão (o backend regista uma linha `[DEBAYER] … needs a CUDA GPU …` a explicar o motivo). Para obter informações sobre o comportamento da Jetson em termos de memória, temperatura e implementação no terreno, consulte o [Guia NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalação do Python SDK (Todos os sistemas Linux)**O `.deb` instala o SDK automaticamente.** Ambos os pacotes (amd64 e Jetson) instalam-no no sistema `python3`, pelo que o `import chloros_sdk` funciona logo após a instalação, sem necessidade de pip nem de ligação à rede. Os seus pacotes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e os carregadores de projetos) são colocados no `/usr/lib/python3/dist-packages`, pertencente ao pacote: uma atualização substitui-os e a remoção do pacote remove-os. A versão corresponde sempre à CLI e ao backend que instalou.

Os requisitos próprios do SDK, `python3-numpy` e `python3-requests`, são **recomendados, não obrigatórios** pelo pacote: o `sudo apt install ./chloros_<version>_<arch>.deb` instala-os, o `dpkg -i` não. A saída da instalação indica a versão do SDK que o `python3` importa, quaisquer pré-requisitos em falta, e qualquer `chloros_sdk` noutro local no caminho do `python3` que substitua a cópia do pacote (como, por exemplo, uma versão instalada anteriormente com o `pip install --user`).**Num virtualenv**, que não tem acesso aos pacotes do sistema, instale o wheel incluído nesse ambiente (totalmente offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**A partir do PyPI** — apenas para uma máquina sem o pacote Chloros instalado, cujo SDK controla um backend noutra máquina (ver [Modo de Backend Remoto](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Extras opcionais, para uma instalação via pip (num virtualenv ou no PyPI):

| Extra | Comando | Adiciona |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para transmissão de progresso em tempo real |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Verifique o SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
O SDK do Python comunica com o backend que o `.deb` instala através de uma HTTP API local (`http://127.0.0.1:5000`). O `ChlorosLocal` e os auxiliares de ligação iniciam-no automaticamente quando necessário; o `connect_all()` de um projeto guardado nunca o faz, por isso, numa máquina sem monitor, execute o backend como um [serviço](#always-on-ptp-for-headless-hosts). Utilize sempre o endereço IPv4 literal em vez de `localhost` — o `localhost` pode ser resolvido para o `::1` e demorar cerca de dois segundos por pedido.
{% endhint %}

***

## Configuração inicial

### 1. Iniciar sessão

O acesso à CLI e ao SDK requer um plano pago do Chloros+ (**Copper** ou superior), aplicado do lado do servidor: um utilizador que tenha saído da sessão recebe `401 AUTH_REQUIRED`, e um utilizador do plano gratuito (Iron) recebe `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

As credenciais são armazenadas em cache em `~/.chloros/user_session.json`.

{% hint style="info" %}
**As atualizações mantêm o seu início de sessão; a remoção apaga-o.** A instalação de uma versão mais recente do `.deb` sobre uma instalação existente (incluindo através do `chloros-cli update`) mantém o `~/.chloros/user_session.json` e a licença armazenada em cache, pelo que as tarefas CLI/SDK sem interface gráfica continuam a funcionar. A remoção do pacote (`sudo apt remove chloros`) apaga o login guardado e a cache da licença para todos os utilizadores na máquina; por isso, uma nova instalação necessita novamente do `chloros-cli login`.
{% endhint %}

### 2. Verifique o estado da sua licença

```bash
chloros-cli status
```

O `chloros-cli status` funciona em qualquer nível (incluindo o gratuito), pelo que pode sempre verificar por que motivo o acesso está ou não disponível.

### 3. Executar o diagnóstico do sistema

```bash
chloros-cli selftest
```

São executadas sete verificações por ordem e o comando termina com um valor diferente de zero se alguma delas falhar:

| # | Verificação | O que verifica |
| --- | --- | --- |
| 1 | **Versão** | A CLI indica a sua versão (`v1.3.0`). |
| 2 | **Porta disponível** | A porta 5000 está livre, *ou* já foi respondida por um backend do Chloros em bom estado (o que conta como aprovado). |
| 3 | **Arranque do backend** | O binário do backend é iniciado. |
| 4 | **Teste da API (`/api/test`)** | O backend responde com `status: ok`. |
| 5 | **Informações do sistema** | Imprime `GPU: <name>, CUDA: <bool>, PyTorch: <version>` a partir de `/api/system-info`. |
| 6 | **Modelos do denoiser** | Encontra modelos `*.pth.enc` (no Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Redutor de ruído**| O Texture Aware está efetivamente utilizável — requer CUDA**e**, pelo menos, um ficheiro de modelo. |

A execução termina com o `N/7 checks passed`, listando quaisquer falhas por nome.

### 4. Processar o seu primeiro conjunto de dados

```bash
chloros-cli process ~/datasets/flight001
```

***

## Ficheiros e diretórios

### Por utilizador O

Chloros mantém as suas credenciais e a configuração da CLI num único diretório multiplataforma, **`~/.chloros/`** (no Windows, `%USERPROFILE%\.chloros\`). Por outro lado, dois caches específicos do Linux seguem as convenções XDG — estes respeitam os diretórios `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando definidos.

| Caminho | Finalidade |
| --- | --- |
| `~/.chloros/user_session.json` | Cache da sessão de início de sessão gravado por `chloros-cli login` (mantido após atualizações do pacote; limpo quando o pacote é removido) |
| `~/.chloros/working_directory.txt` | Substituição da pasta de projeto predefinida (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferência de idioma da CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuração de idioma partilhada com a interface gráfica do Windows — um valor `language` aqui tem prioridade sobre o `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de uma hora para a verificação de atualizações de arranque do Linux/Jetson |
| `~/.chloros/backend.log` | Registo do backend quando este foi iniciado pela CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacotes de calibração LATTICE por câmara armazenados em cache, indexados pelo número de série e pelo hash do pacote |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Substituições opcionais do utilizador para perfis de correção de limite do DAQ |
| `~/.config/chloros/system_config.json` | Perfil de hardware armazenado em cache a partir da Adaptação Dinâmica de Computação — elimine-o para forçar uma nova deteção de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Registos do servidor backend, um ficheiro por início |
| `~/Documents/MAPIR/Chloros Projects/` | Pasta de projeto predefinida quando não está definida qualquer substituição |

### A nível do sistema

| Caminho | Finalidade |
| --- | --- |
| `/usr/bin/chloros-cli` | Script de envoltório — define `LD_LIBRARY_PATH` para as bibliotecas nativas incluídas (e `TMPDIR` para `/mnt/ssd/tmp` quando essa pasta existe), executando em seguida o binário real |
| `/usr/bin/chloros-backend` | Script de envoltório — o mesmo para o backend, que marca como uma compilação de produção instalada |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | A CLI e o backend compilados, instalados descompactados (novidade na versão 1.3.0); os programas são o `cli/chloros-cli` e o `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Scripts de lançamento de duas linhas que iniciam os programas reais acima — mantidos nestes caminhos porque o SDK, o CLI e os scripts mais antigos os procuram aqui |
| `/usr/lib/chloros/arena_runtime/` | Ambiente de execução do SDK da Arena exigido pelas câmaras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos de redução de ruído encriptados utilizados pelo debayer Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (e `lattice_sdk`, `lattice_cam`, `arena_api`, os carregadores do projeto) | O SDK Python, instalado para o sistema `python3` e pertencente ao pacote |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Wheel do Python SDK correspondente a esta compilação exata, para um virtualenv |
| `/usr/lib/chloros/exiftool` | Lançador do ExifTool incluído (com ligação simbólica para o `/usr/local/bin/exiftool` apenas se não existir nenhum exiftool do sistema). A partir da versão 1.3.0, o script ExifTool é fornecido com a sua própria biblioteca Perl em `/usr/lib/chloros/exiftool-dist/`, pelo que funciona sem o pacote do sistema `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configuração do canal de atualização lida pelo `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Define o `net.ipv4.ip_unprivileged_port_start = 319` para que o backend possa estabelecer ligação às portas PTP sem direitos de root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Aumenta os buffers de receção de sockets do kernel (`net.core.rmem_max`, `net.core.rmem_default`) para 32 MB para fluxos de câmara LATTICE e aplica-os na instalação (um valor já definido como superior é mantido). Com a configuração padrão do kernel, os pacotes da câmara são descartados no kernel e um conjunto de câmaras perde fotogramas e taxa de fotogramas; o `chloros-cli lattice network` notifica quando os buffers estão baixos. Ver [Hosts Linux — buffers de receção de sockets](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Aponta o carregador dinâmico para `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede ao utilizador com sessão iniciada acesso à ponte serial USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Serviço de backend sempre ativo (opt-in) que executa o backend como o utilizador que indicar (instalado, **não ativado**) — este é o que deve ativar; consulte [PTP sempre ativo](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | O mesmo serviço que executa o backend como **root**(instalado,**não ativado**), mantido para que uma máquina que o tenha ativado continue a funcionar |
| `/var/log/chloros/` | O registo detalhado do serviço root, legível pelo grupo `adm` (eliminado quando o pacote é removido) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada do menu de aplicações «Chloros CLI» que abre um terminal |

**Firewall (novidade na versão 1.3.0).** Se o `ufw` ou o `firewalld` estiverem ativos no momento da instalação, o pacote abre as portas UDP `5002` (dados DAQ-E), `5353` (descoberta mDNS), `319` e `320` (PTP), e remove essas regras quando o pacote é desinstalado. Se estiver instalado um firewall, mas este não estiver ativo, o pacote não adiciona nada e apresenta as portas a permitir, caso o ative posteriormente. Não abre nenhuma porta TCP: uma atualização do firmware do DAQ-E, na qual o sensor se liga novamente a este computador através de TCP para recolher a imagem, falha com o código de erro `… device never connected back on TCP <port> …` enquanto uma firewall ativa bloqueia essa ligação.

## Localização do executável do backend

A CLI e o SDK detetam automaticamente o backend:

| Componente | Caminho |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (um lançador para o `/usr/lib/chloros/backend/chloros-backend`) |

Substitua o caminho do backend com o sinalizador da CLI `--backend-exe` ou o parâmetro do construtor do SDK `backend_exe`, e a porta com `--port` (padrão: `5000`).

{% hint style="info" %}
O `CHLOROS_BACKEND_URL` aponta para as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para um backend remoto. Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoram-no deliberadamente e têm sempre como alvo o `http://127.0.0.1:<port>`.
{% endhint %}

***

## Câmaras LATTICE e sensores de luz DAQ no Linux

Todas as famílias de comandos «live-hardware» funcionam no Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — deteta, liga, configura e captura imagens a partir de câmaras LATTICE e matrizes sincronizadas. O `.deb` inclui o ambiente de execução do SDK da Arena necessário e regista-o no carregador dinâmico.
* **`chloros-cli daq pool-*`** — liga sensores de luz DAQ-U/M/E através do conjunto de backends, transmite espectros calibrados, grava ficheiros `.daq` e troca perfis de cap, e, no caso de um DAQ-E, altera as suas definições de rede, reinicia-o e atualiza o seu firmware. O `chloros-cli daq discover` procura sensores que possa ligar, e o `chloros-cli daq --help` lista os subcomandos do `pool-*`.
* **`chloros-cli project`** — executa um projeto guardado (as suas câmaras, sensores e definições de processamento) sem interface gráfica.
* **`chloros-cli time-sync`** — inspecionar o «grandmaster» PTP que o backend Chloros executa para câmaras LATTICE e sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

O comando `--sensor-id` é pré-requisito para os comandos `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; o `pool-list` apresenta os IDs atualmente no conjunto.

{% hint style="info" %}
**O `--eth-host` ignora a descoberta.** O `pool-connect --eth` encontra um DAQ-E através do mDNS, com um recurso de fallback ARP que funciona a partir de uma cache ARP vazia; ao passar o IP ou o nome de host do sensor com o `--eth-host`, estabelece-se uma ligação direta ao mesmo.
{% endhint %}

**As permissões seriais do DAQ-U** são geridas pela regra udev instalada (`uaccess` + grupo `dialout`). Se um sensor que já estava ligado permanecer inacessível, recarregue as regras ou volte a ligá-lo:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulte a [referência da CLI](../CLI.md) para ver o conjunto completo de comandos.

### PTP sempre ativo para anfitriões sem monitor

O pacote instala dois serviços systemd para o backend, nenhum deles ativado. Num Jetson sem monitor ou num servidor que deva manter a sincronização de tempo PTP a funcionar continuamente para sensores DAQ-E e câmaras LATTICE, ou onde scripts do SDK liguem projetos guardados (o `connect_all()` nunca inicia um backend), ative o serviço **como o seu utilizador**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

O `chloros-backend@<user>.service` executa o backend como esse utilizador: as capturas que grava são dele, o seu registo detalhado encontra-se no `~/.cache/chloros/logs` e utiliza o `chloros-cli login` (`~/.chloros`), por isso inicie sessão primeiro uma vez com esse utilizador. A instalação do pacote apresenta este comando com o seu nome de utilizador preenchido.

Sem um serviço, o backend (e, com ele, o grandmaster PTP) só funciona enquanto algo o tiver iniciado — um comando `chloros-cli` ou um script SDK.

A unidade associa o backend às definições de ambiente `127.0.0.1:5000` (configurações de ambiente `CHLOROS_HOST` / `CHLOROS_PORT` dentro da unidade; substitua por `sudo systemctl edit chloros-backend@.service`) e reinicia-o em caso de falha após 5 segundos.

**O serviço root.**O `chloros-backend.service` executa o backend como**root**. Continua instalado para que uma máquina que o tenha ativado continue a funcionar, mas tudo o que um backend root escreve pertence ao root, pelo que um utilizador não pode eliminar as suas próprias capturas; o seu registo encontra-se em `/var/log/chloros`, legível pelo grupo `adm`. Para mudar para o seu utilizador:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**O backend de um serviço é partilhado e nunca é interrompido por um cliente.** O `chloros-cli` e os scripts SDK utilizam-no tal como o encontram: o `process` é executado nele, carregando o seu próprio projeto no lugar do último. Nos casos em que a CLI, de outra forma, reiniciaria o backend (`--restart`, ou um `process` cuja importação não encontrou nada), ela identifica a unidade, exibe «`sudo systemctl restart <unit>`» e sai do «`1`», porque um backend parado dessa forma permaneceria inativo. Reinicie-o manualmente com «`sudo systemctl restart chloros-backend@$USER.service`». O backend de um serviço nunca se interrompe por inatividade, e o `chloros-cli backend stop` recusa essa ação, indicando o comando `systemctl stop`.**Como o PTP obtém as suas portas.** O PTP utiliza as portas UDP 319/320, ambas abaixo do limite normal de 1024 para portas privilegiadas. O comando `postinst` do pacote escreve `/etc/sysctl.d/60-chloros-ptp.conf` com `net.ipv4.ip_unprivileged_port_start = 319`, o que permite que o backend as vincule enquanto é executado como o seu utilizador. A partir da versão 1.3.0, o pacote não define capacidades de ficheiro (`setcap`) no backend, e o `libcap2-bin` deixou de ser uma dependência: no backend descompactado, uma capacidade de ficheiro impediria o seu arranque, por isso não adicione nenhuma por conta própria.

***

## Exemplos de scripts em Bash

{% hint style="info" %}
**Códigos de saída compatíveis com scripts.**O `chloros-cli process` encerra o `0` em caso de sucesso e**com um valor diferente de zero em caso de falha — incluindo uma execução que solicitou produtos de imagem mas não gravou nenhum** (exibe `Processing finished but wrote no image products.` e indica o nome da pasta do projeto e as causas habituais). As execuções bem-sucedidas indicam quantos produtos de imagem foram gravados (`Image products written: N`, mais `(plus M clamp masks)` quando foram gravadas máscaras). Códigos de saída: `0` sucesso, `1` falha, `2` erro de argumento, `130` interrompido.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Os valores válidos para o `--format` são exatamente quatro e contêm espaços — coloque-os sempre entre aspas:

| Valor do `--format` | Pasta de saída |
| --- | --- |
| `TIFF (16-bit)` *(padrão)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` aceita `standard` (padrão) ou `texture-aware` (Chloros+).

### Processamento automatizado com o Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplo de SDK do Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Resolução de problemas

### CLI não encontrada após a instalação

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### O backend não inicia após a adição de capacidades de ficheiro

Não execute o `setcap` no backend. A partir da versão 1.3.0, o pacote não define quaisquer capacidades de ficheiro, e as portas 319/320 do PTP provêm da configuração `sysctl` descrita em [Como o PTP obtém as suas portas](#always-on-ptp-for-headless-hosts). Uma capacidade de ficheiro coloca o backend descompactado num modo de execução segura, no qual não consegue localizar as suas próprias bibliotecas, pelo que não arranca. Se tiver adicionado uma, remova-a:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### O PTP não inicia / Não consegue ligar à porta 319

Confirme se o limite mínimo de portas sem privilégios foi reduzido e, caso não tenha sido, reaplique-o para a inicialização atual:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Em seguida, verifique o «grandmaster»:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Os comandos `lattice` falham com a mensagem «LATTICE camera drivers not found»

O backend não conseguiu carregar o runtime da câmara LATTICE com o qual está instalado. Todos os comandos `lattice` são executados no backend; a própria CLI não contém controladores de câmara. Reinstale o Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): o `.deb` instala o ambiente de execução juntamente com o backend, no `/usr/lib/chloros/arena_runtime`, e regista-o no carregador dinâmico. Para verificar esse registo:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Falha ao iniciar o backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Os registos do backend relativos ao arranque falhado encontram-se em `~/.cache/chloros/logs/`. Se a porta 5000 pertencer a um serviço de backend (`systemctl status chloros-backend@$USER.service`), reinicie o serviço em vez de encerrar o seu processo.

### CUDA não detetada

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

O ficheiro `chloros-cli selftest` apresenta a mesma informação numa única linha: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Bibliotecas partilhadas em falta

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Arranque lento em sistemas com cartão SD

A partir da versão 1.3.0, o backend e a CLI são instalados descompactados, pelo que já não se extraem para um diretório temporário a cada arranque. Os ficheiros temporários são transferidos automaticamente para o `/mnt/ssd/tmp`, desde que essa pasta exista; caso contrário, pode direcioná-los para um sistema de ficheiros rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Atualização do Chloros no Linux

O comando `update` é exclusivo para Linux/Jetson. Verifica a versão publicada no canal de atualização configurado em `/etc/chloros/update.conf` e oferece a possibilidade de descarregar e instalar o `.deb` correspondente:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

No Linux/Jetson, a CLI também executa uma verificação de atualizações não bloqueante em cada arranque (resultado armazenado em cache durante uma hora em `~/.chloros/update_cache.json`) e apresenta `Update available: vX.Y.Z` quando existe uma versão mais recente. As suas definições, projetos e dados de início de sessão permanecem inalterados após uma atualização.

A atualização do pacote interrompe todos os serviços de backend durante a instalação, pelo que um script em execução perde as suas câmaras e sensores e, em seguida, reinicia cada um dos que estiverem ativados (um script iniciado manualmente, mas nunca ativado, permanece parado, tal como após um reinício).

## Desinstalação

```bash
sudo apt remove chloros
```

A remoção interrompe e desativa todos os serviços de backend, restaura o limite padrão de portas sem privilégios (1024), remove o ficheiro de configuração do buffer de receção (os valores em execução permanecem até ao reinício), o link simbólico do exiftool incluído, a configuração do carregador Arena, as regras de firewall adicionadas pelo pacote e o PythonSDK instalado, e limpa as credenciais em cache. Os seus projetos e ficheiros de dados `~/.chloros/` não são afetados. O `sudo apt purge chloros` também elimina o `/var/log/chloros`.

***

## Próximos passos

* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — Otimização e implementação específicas para o Jetson
* [CLI: Linha de comandos](../CLI.md) — o guia da CLI
* [API: Python SDK](../api-python-sdk.md) — o guia do SDK
* [Referência da CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas exaustivas de comandos/API para a versão 1.3.0
* [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware
