# CLI : Linha de comandos

> **Referência completa:**[CLI Referência](reference/cli-reference.md) documenta**todos os parâmetros de todos os subcomandos** e está otimizada para assistentes de IA — cole o seu URL no seu assistente e peça um comando que funcione: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Dica para ferramentas de IA:** qualquer página deste manual está disponível em formato Markdown bruto, bastando acrescentar `.md` ao seu URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), e `https://mapir.gitbook.io/chloros/llms.txt` indexa todo o manual para consumo por LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## O que é oCLI


O `chloros-cli` é a interface de linha de comandos do mesmo motor de processamento que a aplicação de ambiente de trabalhoChloros
utiliza. Trata-se de um cliente «HTTP
» leve sobre o backendChloros
(um servidor local no `127.0.0.1:5000`) — a maioria dos comandos inicia o backend automaticamente, pelo que basta uma única chamada ao `chloros-cli process …` num script.

Funciona no **Windows
10/11 (x64)**e no**Linux
(x86_64 e NVIDIA Jetson arm64 no JetPack 6)**, em qualquer terminal, sem necessidade de interface gráfica. Verifique a sua instalação com:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

As famílias de comandos, num relance:

* **Processamento e conta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — consulte [Idiomas suportados](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (apenasLinux
/Jetson)
* **Hardware ativo** — `lattice` (controlo de câmara LATTICE, mais de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automação** — `project` (executar um projeto guardado noChloros
sem interface gráfica, incluindo receitas de captura em YAML)

Opções globais que vale a pena conhecer: `--port N` (porta do backend, predefinição `5000`), `-v/--verbose`, `--restart` (reiniciar à força o backend), `--backend-exe PATH`. Consulte a [Referência doCLI
](reference/cli-reference.md) para obter a lista completa.

***

## Instalação

OCLI
**vem incluído no instalador doChloros** em todas as plataformas — não existe um download separado doCLI
. Obtenha o instalador na página [Download](download.md).

###Windows


O instalador coloca oCLI
em:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e adiciona essa pasta ao seu sistema `PATH` — **abra um novo terminal**após a instalação para que o `PATH` atualizado seja detetado. O instalador também coloca scripts de lançamento (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) na raiz da instalação, além de um**Chloros
CLI
** no menu Iniciar, cada um dos quais abre um terminal com o `chloros-cli` pronto a utilizar.

###Linux


Instale o `.deb` para a sua arquitetura:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Isto instala o `chloros-cli` até ao `/usr/bin/chloros-cli` (já na versão `PATH`) e o backend para a versão `/usr/lib/chloros/chloros-backend`, juntamente com o runtime ArenaSDK
necessário para as câmaras LATTICE. Consulte [Instalação doLinux
](linux/linux-installation.md) para obter mais detalhes.

### Verificar

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Início de sessão e licenciamento

CLI
(ePython
SDK
) requer um **plano pagoChloros
+**— qualquer nível pago inclui esta funcionalidade; o nível gratuito não. A restrição é aplicada**do lado do servidor** pelo backend, não pelo binárioCLI
: uma chamada sem sessão é rejeitada com o código de erro `401 AUTH_REQUIRED`, e uma chamada com sessão iniciada no plano gratuito com o código de erro `403 PLAN_UPGRADE_REQUIRED`, quer provenha do `chloros-cli`, doSDK
ou de um clienteHTTP
desenvolvido manualmente. Atualize em [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Inicie sessão **uma vez por máquina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Palavras-passe com caracteres especiais**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` é corrompida pelo shell (oCLI
deteta isto num erro 401 e tenta novamente automaticamente, mas as aspas simples evitam completamente o problema).
{% endhint %}

A sessão é armazenada em cache em `~/.chloros/user_session.json` e continua a funcionar offline durante o período de carência do plano (30 dias para planos mensais, até ao vencimento para planos anuais). O `chloros-cli status` funciona mesmo sem um plano pago, pelo que o motivo da recusa está sempre visível.

{% hint style="danger" %}
**A agendar tarefas sem interface gráfica? Inicie sessão primeiro.**Um comando de criação de processos de backend (`process`, `status`, `export-status`, …) executado**sem sessão em cache**não falha rapidamente — entra num prompt interativo `Email:` / `Password:` na stdin. Uma tarefa cron ou etapa de CI não supervisionada ficará, portanto,**bloqueada à espera de entrada**. Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez na máquina antes de agendar qualquer coisa.
{% endhint %}

***

## A sua primeira execução de processamento

Aponte o `process` para uma pasta de capturas — ele deteta automaticamente oSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ou uma combinação destas:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

O progresso é transmitido em tempo real por cada thread do pipeline (Detecção, Análise, Processamento, Exportação), e uma execução bem-sucedida termina com a indicação do número de produtos de imagem que foram gravados (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Onde são guardados os resultados

O `process` grava numa **pasta de projeto**, e não na sua pasta de entrada:

* Sem o `-o`: o projeto é criado na sua pasta de projetos predefinida (partilhada com a GUI; gere-a com o `get-project-folder` / `set-project-folder`, opção alternativa `~/Chloros Projects`), com o nome `-n/--project-name` ou um carimbo de data/hora (`YYYYMMDD_HHMMSS`) quando omitido.
* Com `-o PATH`: essa pasta **é** a pasta do projeto. Se já contiver um `project.json`, é criado um ficheiro irmão com o sufixo `_1`/`_2`… em vez de se sobrescrever.

Dentro do projeto, os produtos são agrupados **por câmara e, em seguida, por formato de ficheiro**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE (correspondendo ao EXIF da captura `Model`) e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) paraSurvey3
. A pasta de formato segue a sequência `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Todos os produtos exportados mantêm o nome do ficheiro ORIGINAL.**Uma exportação do Radiance do `capture_..._raw.tif` continua a chamar-se `capture_..._raw.tif` — apenas se encontra na pasta `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do ficheiro**, por isso utilize um padrão de pesquisa para o diretório, não para o sufixo `*radiance*`.
{% endhint %}

### As opções que irá realmente utilizar

| Opção | Padrão | O que faz |
| --- | --- | --- |
| `-o, --output PATH` | pasta de projeto padrão | Localização da pasta do projeto (ver acima). |
| `-n, --project-name NAME` | carimbo de data/hora | Nome do projeto. |
| `--format FMT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação a exportar (ver [Índices de vegetação](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neural, mais lento, qualidade máxima (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | ativado | Correção de vinheta. |
| `--reflectance / --no-reflectance` | ativado | Calibração de refletância; para o LATTICE, esta opção também ativa ou desativa o produto de refletância. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para ficheiros TIFF do LATTICE. |

Para tudo o resto — ajuste da deteção de alvos, PPK, pinos de exposição, sinalizadores de alinhamento de matrizes — consulte a [secção `process` da Referência doCLI
](reference/cli-reference.md).

***

## Escolher o que exportar (produtos LATTICE)

O processamento LATTICE ramifica-se para **todos os produtos aplicáveis numa única passagem**. Quatro opções por produto estão todas**ativadas por predefinição**; utilize o formulário `--no-` para desativar uma:

| Opção | Produto |
| --- | --- |
| `--debayered` | Demosaico linear → `Debayered_Images/` |
| `--preview` | Pré-visualização no ecrã (equilíbrio de brancos + gama; expansão de cores falsas para multiespectral) → `Preview_Images/` |
| `--radiance` | radiação em float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 refletância, compatível com Pix4D → `Reflectance_Calibrated_Images/` |

RGB
as câmaras principais emitem sempre apenas dados pós-debayering + pré-visualização — a radiação/refletância por banda não tem significado para um sensor de banda larga, pelo que essas opções não têm qualquer efeito para elas.Survey3
`.raw` ignora as opções e segue o caminho padrão de refletância/alvo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (padrão `auto`) seleciona a referência de refletância: `auto` cria um [alvo de calibração](calibration-targets.md) dentro do quadro que cumpre os requisitos de controlo de qualidade como referência absoluta e recorre à divisão de luz descendente do sensor de luz do DAQ (ρ = π·L/E) quando não há alvo presente; o `target` é rigoroso (sem substituição do DAQ); `daq` tem autoridade do DAQ. Podem ser fornecidas varreduras de alvos medidos por unidade com `--target-reflectance-dir`.

{% hint style="info" %}
**Leitura de píxeis de refletância:**o DN que significa ρ = 1,0 é**por fonte** — Os ficheiros LATTICE incluem a marcação `Chloros:PixelScale=32768` no XMP; os ficheirosSurvey3
utilizam 65535 (e não contêm etiquetas `Chloros:*`). Leia a etiqueta e divida por esse valor, em vez de assumir uma constante. Os detalhes e o único caso-limite deliberado sem escala encontram-se na [Referência doCLI
](reference/cli-reference.md).
{% endhint %}

**O processamento começa sempre a partir de `raw`.** Os produtos derivados (exportações de debayering, radiação e refletância) nunca são reenviados para o pipeline — reimportá-los e processá-los aplicaria duas vezes os cálculos de calibração, pelo queChloros
os ignora e indica isso mesmo. O `--input-level` é a via de escape deliberada para quando for realmente necessário forçar um ponto de entrada.

***

## Quando uma execução falha

A partir da versão 1.2.0, o `process` falha de forma evidente, em vez de «ter sucesso» sem apresentar qualquer resultado:

* Uma execução que **solicitou produtos mas não gravou nenhum**— apenas o `project.json` e o `calibration_data.json` — apresenta o `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, pelo que os scripts conseguem detetá-lo. As causas habituais: a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou todos os produtos solicitados eram inaplicáveis para essas câmaras (por exemplo, solicitar radianância/refletância a partir de câmaras que apenas capturam o canalRGB
).
* Uma **execução deliberada apenas com metadados** (todos os produtos desativados, sem `--indices`) continua a ser considerada bem-sucedida — uma saída de imagem vazia é o resultado correto neste caso.
* Repita a execução com o `--verbose` e verifique o registo do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explicam as omissões por câmara.

Códigos de saída: `0` sucesso · `1` falha genérica · `2` erro de argumento · `130` interrompido por Ctrl+C.

***

## Índices de vegetação

Execute o `--indices` com um ou mais nomes de predefinições; cada índice é guardado na sua própria pasta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Os 22 nomes predefinidos que o `process --indices` aceita:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Existem três listas de índices — não as confunda.**O menu suspenso «Definições do projeto» da GUI tem 27 fórmulas (acrescenta `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — estas cinco são exclusivas da GUI e**não** são válidas para `--indices`). O comando «live/offline» `lattice index --preset` utiliza a sua própria lista separada de 22 predefinições. As fórmulas e os cálculos de bandas estão documentados em [Fórmulas de Índices Multiespectrais](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: uma breve introdução

A família `daq pool-*` controla os sensores espectrais DAQ dMAPIR
(DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) através do conjunto persistente do backend — a GUI,CLI
eSDK
partilham todos um único identificador ativo. **`pool-*` é o caminho DAQ suportado na versão fornecida doCLI
**; outros subcomandos `daq` que possa ver referenciados são uma superfície interna exclusiva de origemMAPIR
e terminam com um erro explícito que o remete para o `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

O `pool-record`, sem o `--duration`, executa-se até ao `pool-record --stop`; o diretório de saída predefinido é o `~/Documents/DAQ Live View/` **na máquina do backend**. O perfil de correção de limite é escolhido no momento da ligação (`--cap-id`, predefinição do backend `sunshine_cosine`) e pode ser alterado em tempo real com `pool-set-cap` — os perfis de cap e a gama calibrada do sensor são abordados nos capítulos sobre DAQ deste manual.

{% hint style="warning" %}
**DAQ-E num anfitrião com várias placas de rede:** a primeira deteção automática de `pool-connect --eth` após o arranque pode falhar, mesmo com um sensor em bom estado. `--eth-host <ip-or-hostname>` é a forma fiável — recorra a ela sempre que a deteção não produzir resultados.
{% endhint %}

***

## Câmaras LATTICE, PTP e automatização de projetos

A família `lattice` (mais de 45 subcomandos) abrange todo o processo de trabalho com câmaras LATTICE: deteção, capturas individuais, matrizes sincronizadas persistentes com o fluxo de ligação «smart-prep» da GUI, pré-visualização ao vivo no navegador, alinhamento, cálculos de índice e diagnóstico da placa de rede do anfitrião. Uma amostra:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

A par disso: o `chloros-cli time-sync` gera relatórios sobre o «grandmaster» PTP que o anfitriãoChloros
executa (as câmaras LATTICE e os sensores DAQ-E funcionam como escravos deste para carimbos de data/hora entre dispositivos), e o `chloros-cli project` abre um projeto «Chloros
» guardado e controla as suas câmaras, matrizes e sensores sem interface gráfica — incluindo receitas de captura YAML programadas.

Estas três famílias (`lattice`, `project`, `daq pool-*`) são também as únicas que suportam o comando `CHLOROS_BACKEND_URL` para controlar um backend **remoto**; os comandos principais destinam-se sempre à máquina local.

Os tutoriais completos encontram-se nos capítulos sobre o LATTICE deste manual; todas as opções estão na [Referência doCLI
](reference/cli-reference.md).

***

## Resolução de problemas: Top 5

| Sintoma | Solução |
| --- | --- |
| O `Login required`, ou uma tarefa agendada, fica bloqueado num prompt do `Email:` | Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez nesta máquina — os comandos sem uma sessão em cache são executados de forma interativa, em vez de falharem imediatamente. |
| `backend unreachable` | Inicie a aplicação de ambiente de trabalho «Chloros
» ou execute diretamente o binário do backend (`chloros-backend`). Se apontar o `lattice`/`project`/`daq pool-*` para um backend remoto, verifique o `CHLOROS_BACKEND_URL`. |
| Ligação da matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | O anel de receção da placa de rede do anfitrião foi reposto para os valores predefinidos — a principal causa de um equipamento que antes funcionava se recusar a ligar-se, normalmente após uma atualização do controlador da placa de rede. Execute o `chloros-cli lattice network --fix` a partir de um terminal com **privilégios elevados** (ou defina o `ReceiveBufferLen=256` e o `PendingReceives=64`); consulte a secção *Configuração e Ajuste da Placa de Rede do Anfitrião* da documentação de referência. |
| O subcomando `daq` termina com a mensagem: «requer o pacote DAQ completo…» | Esperado nas compilações fornecidas — oCLI
compilado fornece apenas a família `daq pool-*`, que abrange a ligação, o fluxo, o registo e a seleção de cap. Utilize o `pool-*` (ou o `chloros_sdk.connect_daq_sensor()` disponível emPython
). |
| O Jetson apresenta um aviso de swap antes de pastas de grande dimensão | Adicione swap baseado em ficheiros — oCLI
apresenta os comandos exatos `fallocate`/`swapon` a executar. |

***

## Obter ajuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todos os sinalizadores, todos os subcomandos:** [CLI
Referência](reference/cli-reference.md)
* **Equivalente noPython
:** [Python
SDK
](api-python-sdk.md) e a [SDK
Referência](reference/sdk-reference.md)
* **Suporte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
