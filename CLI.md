# CLI: Linha de comandos

> **Referência completa:**[Referência da CLI](reference/cli-reference.md) documenta**todos os parâmetros de todos os subcomandos** e está otimizada para assistentes de IA — cole o seu URL no seu assistente e peça um comando funcional: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Dica para ferramentas de IA:** qualquer página deste manual está disponível em formato Markdown bruto, bastando acrescentar `.md` ao seu URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), e `https://mapir.gitbook.io/chloros/llms.txt` indexa todo o manual para consumo por LLM.

> **Quer que o seu assistente execute o próprio Chloros?** O `chloros-mcp`, um servidor MCP instalado com o Python SDK, funciona com o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP: consulte [Utilizar o Chloros com assistentes de IA](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## O que é a CLI


O `chloros-cli` é a interface de linha de comandos do mesmo motor de processamento que a aplicação de ambiente de trabalho Chloros
utiliza. Trata-se de um cliente HTTP
leve sobre o backend do Chloros
(um servidor local no `127.0.0.1:5000`) — a maioria dos comandos inicia o backend automaticamente, pelo que basta uma única chamada ao `chloros-cli process …` num script. O backend permanece então em execução para o comando seguinte: parando no comando `chloros-cli logout` (a menos que a aplicação esteja aberta ou que uma execução ou compilação esteja em curso), no comando `chloros-cli backend stop`, ou por si próprio após 30 minutos sem nenhum cliente, execução ou dispositivo ligado. O comando `chloros-cli backend status` mostra isso.

Funciona no **Windows
10/11 (x64)**e no**Linux
(x86_64 e NVIDIA Jetson arm64 no JetPack 6)**, em qualquer terminal, sem necessidade de GUI. Verifique a sua instalação com:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

As famílias de comandos, num relance:

* **Processamento e conta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — consulte [Idiomas suportados](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (apenas Linux
/Jetson)
* **Hardware ativo** — `lattice` (controlo de câmara LATTICE, mais de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automação** — `project` (executar um projeto Chloros
guardado sem interface gráfica, incluindo receitas de captura YAML)

Opções globais que vale a pena conhecer: `--port N` (porta do backend, predefinição `5000`), `-v/--verbose`, `--restart` (reiniciar à força o backend), `--backend-exe PATH`. Consulte a [Referência da CLI
](reference/cli-reference.md) para obter a lista completa.

***

## Instalação

A CLI
**vem incluída no instalador do Chloros** em todas as plataformas — não existe um download separado da CLI
. Obtenha o instalador na página [Download](download.md).

### Windows


O instalador coloca a CLI
em:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e adiciona essa pasta ao seu sistema `PATH` uma única vez (uma atualização também remove entradas duplicadas deixadas por instaladores anteriores) — **abra um novo terminal**após a instalação para que o `PATH` atualizado seja detetado. O instalador também coloca scripts de lançamento (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) na raiz da instalação, além de um**Chloros
CLI
** no menu Iniciar, cada um dos quais abre um terminal com o `chloros-cli` pronto a utilizar.

### Linux


Instale o `.deb` para a sua arquitetura com o `apt` (não o `dpkg -i`, que ignora os requisitos recomendados do SDK
do Python
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Isto instala o `chloros-cli` até ao `/usr/bin/chloros-cli` (já presente no `PATH`), o backend e os programas de CLI
descompactados em `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/` (o `/usr/lib/chloros/chloros-backend` é um lançador que inicia o backend), o tempo de execução do SDK
da Arena de que o backend necessita para as câmaras LATTICE e o Python
SDK
para o sistema `python3`. Consulte [Instalação no Linux
](linux/linux-installation.md) para obter mais detalhes.

### Verificar

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Início de sessão e licenciamento

O acesso à CLI
(e ao Python
SDK
) requer um **plano Chloros
+ pago**— qualquer nível pago inclui-o; o nível gratuito não. O limite mínimo é aplicado**do lado do servidor** pelo backend, não pelo binário da CLI
: uma chamada sem sessão ativa é rejeitada com o código `401 AUTH_REQUIRED`, e uma chamada com sessão iniciada no plano gratuito com o código `403 PLAN_UPGRADE_REQUIRED`, quer provenha do `chloros-cli`, do SDK
ou de um cliente HTTP
personalizado. Atualize em [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Inicie sessão **uma vez por máquina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Palavras-passe com caracteres especiais**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` é corrompida pelo shell (a CLI
deteta isto num erro 401 e tenta novamente automaticamente, mas as aspas simples evitam completamente o problema).
{% endhint %}

A sessão é armazenada em cache em `~/.chloros/user_session.json` e continua a funcionar offline durante o período de carência do plano (30 dias para planos mensais, até ao vencimento para planos anuais). O `chloros-cli status` funciona mesmo sem um plano pago, pelo que o motivo da recusa está sempre visível.

{% hint style="danger" %}
**A agendar tarefas sem interface gráfica? Inicie sessão primeiro.**Um comando de criação de processos de backend (`process`, `status`, `export-status`, …) executado**sem sessão em cache**não falha rapidamente — entra num prompt interativo `Email:` / `Password:` na stdin. Uma tarefa cron ou etapa de CI não supervisionada ficará, portanto,**bloqueada à espera de entrada**. Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez na máquina antes de agendar qualquer coisa.
{% endhint %}

***

## A sua primeira execução de processamento

Aponte o `process` para uma pasta de capturas — ele deteta automaticamente o Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ou uma combinação destes:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Os fluxos de progresso são transmitidos em tempo real por cada thread do pipeline (Detecção, Análise, Processamento, Exportação), e uma execução bem-sucedida termina com a indicação do número de produtos de imagem que foram gravados: `Image products written: N`, seguido de `(plus M clamp masks)` quando a execução também gravou [máscaras de limitação](output-image-formats.md#clamp-masks) (estas não são contabilizadas como produtos).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Se um backend do Chloros
já estiver a funcionar (por exemplo, o da aplicação para computador), o `process` utiliza-o e abre o seu projeto nesse backend. Recusa-se enquanto a aplicação estiver aberta num projeto que a própria aplicação tenha aberto: processe a partir da aplicação ou feche esse projeto. Se uma execução anterior ainda estiver a terminar, aguarda até 15 segundos antes de recusar. O `--restart` força a utilização de um backend novo. No Linux
, um backend executado como serviço do systemd nunca é reiniciado: o `process` utiliza-o tal como está e, nos casos em que, de outra forma, o reiniciaria (`--restart` ou uma importação que não encontrou nada), exibe o código `sudo systemctl restart <unit>` e encerra com o código `1`.

### Onde as saídas são guardadas

O `process` grava numa **pasta de projeto**, e não na sua pasta de entrada:

* Sem `-o`: o projeto é criado na sua pasta de projetos predefinida (partilhada com a GUI; gere-a com `get-project-folder` / `set-project-folder`; quando nenhuma estiver definida, o `Documents/MAPIR/Chloros Projects` na sua pasta de utilizador), com o nome `-n/--project-name` ou um carimbo de data/hora (`YYYYMMDD_HHMMSS`) quando omitido.
* Com o `-o PATH`: essa pasta **é** a pasta do projeto. Se já contiver um `project.json`, é criado um ficheiro irmão com o sufixo `_1`/`_2`… com sufixo, em vez de se sobrescrever. Um `PATH` relativo é considerado em relação à pasta a partir da qual executa o `chloros-cli`.

{% hint style="info" %}
**Caminhos relativos (alterados na versão 1.3.0).** Todos os caminhos que o backend abre — `-o`/`--output` em qualquer comando (incluindo o `output` predefinido dos comandos de captura), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask` — é resolvido em relação à pasta a partir da qual o `chloros-cli` é executado, pelo que o `-o captures/` grava no `<current folder>/captures/`. Na versão 1.2.0, o backend resolvia esses ficheiros na sua própria pasta de instalação, o que falhava com a mensagem «Acesso negado». No caso de um backend remoto (`CHLOROS_BACKEND_URL`), os caminhos são enviados tal como digitados e resolvidos nessa máquina.
{% endhint %}

{% hint style="warning" %}
**Windows
: mantenha a pasta do projeto curta, a menos que os caminhos longos estejam ativados.** Com os caminhos longos do Windows
desativados (a predefinição do Windows
), nenhum caminho de ficheiro pode ter mais de 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. O `process` verifica isto antes de iniciar: uma pasta de projeto demasiado profunda é rejeitada com a mensagem **«A pasta de projeto é demasiado profunda para este computador»**, a pasta de projeto mais longa que cabe e como ativar os caminhos longos — nada é processado. Utilize um `-o` (ou `set-project-folder`) mais curto, ou ative os caminhos longos do Windows
(a opção **Ativar caminhos longos do Windows** do instalador, marcada por predefinição) e reinicie o Chloros
.
{% endhint %}

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

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE (correspondendo ao EXIF da captura `Model`) e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para o Survey3
. A pasta de formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para `TIFF (32-bit, Percent)`.

No caso de uma câmara multiespectral M3C, também pode aparecer uma pasta `clamp_masks/` numa pasta de formato, ao lado das pastas de produtos (por exemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/` e `tiff32/clamp_masks/Radiance_Images/`). Esta pasta contém as máscaras por pixel dos produtos de radiação e refletância em que a separação de bandas limitou um valor negativo a zero — consulte [Máscaras de limitação](output-image-formats.md#clamp-masks). Nenhuma pasta de produto contém, em caso algum, uma máscara.

{% hint style="info" %}
**Todos os produtos exportados mantêm o nome do ficheiro ORIGINAL.**Uma exportação de radiação de `capture_..._raw.tif` continua a chamar-se `capture_..._raw.tif` — apenas se encontra em `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do ficheiro**, por isso utilize um padrão de pesquisa para o diretório, não para o sufixo `*radiance*`.
{% endhint %}

### As opções que irá realmente utilizar

| Opção | Padrão | O que faz |
| --- | --- | --- |
| `-o, --output PATH` | pasta de projeto padrão | Localização da pasta do projeto (ver acima). |
| `-n, --project-name NAME` | carimbo de data/hora | Nome do projeto. |
| `--format FMT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. O Chloros
não aumenta a resolução: em todas as imagens de 8 bits, um formato com maior profundidade é gravado como `TIFF (8-bit)`, e isso é indicado uma vez. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação a exportar (ver [Índices de vegetação](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neural, mais lento, a mais alta qualidade (Chloros
+, requer uma GPU CUDA: sem uma, a execução utiliza o `standard`; recusado para capturas LATTICE, que utilizam sempre o `standard`). |
| `--vignette / --no-vignette` | ativado | Correção de vinheta. |
| `--reflectance / --no-reflectance` | ativado | Calibração de refletância; para o LATTICE, esta opção também ativa ou desativa o produto de refletância. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para ficheiros TIFF do LATTICE. |

Para tudo o resto — ajuste da deteção de alvos, PPK, pinos de exposição, sinalizadores de alinhamento de matrizes — consulte a [secção `process` da Referência da CLI
](reference/cli-reference.md).

***

## Escolher o que exportar (produtos LATTICE)

O processamento LATTICE ramifica-se para **todos os produtos aplicáveis numa única passagem**. Quatro opções por produto estão todas**ativadas por predefinição**; utilize o formulário `--no-` para desativar uma:

| Opção | Produto |
| --- | --- |
| `--debayered` | Demosaico linear → `Debayered_Images/`: o desvio de escuro do pacote de calibração é removido, seguido de um demosaico sensível às arestas, sem correção de linearidade, campo plano ou separação de cores — o mesmo ficheiro que uma captura guarda ao nível do `debayered`. Mantém a orientação nativa do sensor (registada em XMP). |
| `--preview` | Pré-visualização no ecrã (equilíbrio de brancos + gama; alongamento em cores falsas para multiespectral) → `Preview_Images/` |
| `--radiance` | radianância float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 refletância, compatível com Pix4D → `Reflectance_Calibrated_Images/` |

As câmaras principais RGB
emitem sempre apenas dados pós-debayering + pré-visualização — a radianância/refletância por banda não é significativa para um sensor de banda larga, pelo que essas opções não têm efeito para elas. O Survey3
`.raw` ignora as opções e segue o caminho padrão de refletância/alvo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (padrão `auto`) seleciona a referência de refletância: O `auto` define um [alvo de calibração](calibration-targets.md) dentro do quadro, que tenha passado no controlo de qualidade (QA), como referência absoluta e recorre à divisão de luz descendente do sensor de luz do DAQ (ρ = π·L/E) quando não há nenhum alvo presente (um alvo que passe no controlo de qualidade com um sinalizador compete com o DAQ na pontuação medida); o `target` é rigoroso (sem substituição pelo DAQ); `daq` tem autoridade sobre o DAQ. Podem ser fornecidas digitalizações de alvos medidos por unidade com `--target-reflectance-dir`.

{% hint style="info" %}
**Leitura de píxeis de refletância:**o DN que significa ρ = 1,0 é**por fonte** — os ficheiros LATTICE inserem a tag `Chloros:PixelScale=32768` no XMP; os ficheiros Survey3
utilizam 65535 e não contêm a etiqueta `Chloros:PixelScale` (contudo, contêm outras etiquetas de processamento `Chloros:*`). Leia a etiqueta e divida por ela, em vez de assumir um valor constante. Os detalhes e o único caso-limite deliberado sem escala encontram-se na [Referência da CLI
](reference/cli-reference.md).
{% endhint %}

**O processamento começa sempre a partir de `raw`.** Os produtos derivados (exportações de debayering/radiação/refletância) nunca são reenviados através do pipeline — reimportá-los e processá-los aplicaria duas vezes os cálculos de calibração, pelo que o Chloros
os ignora e indica isso mesmo. O `--input-level` é a via de escape deliberada para quando for realmente necessário forçar um ponto de entrada.

***

## Quando uma execução falha

A partir da versão 1.2.0, o `process` falha de forma evidente, em vez de «ter sucesso» sem apresentar qualquer resultado:

* Uma execução que **solicitou produtos mas não gravou nenhum**— apenas `project.json` e `calibration_data.json` — apresenta `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, pelo que os scripts conseguem detetá-lo. As causas habituais: a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou todos os produtos solicitados eram inaplicáveis para essas câmaras (por exemplo, solicitar radianância/refletância a partir de câmaras apenas RGB
).
* Uma **execução deliberada apenas com metadados** (todos os produtos desativados, sem `--indices`) continua a ser considerada bem-sucedida — uma saída de imagem vazia é o resultado correto nesse caso.
* Volte a executar com o `--verbose` e verifique o registo do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explicam as omissões por câmara.
* No Windows
, com os caminhos longos desativados, uma pasta de projeto demasiado profunda para o limite de 260 caracteres no caminho é rejeitada **antes** do início do processamento, sendo utilizada a pasta de projeto mais longa que couber (ver [Onde são guardados os ficheiros de saída](#where-the-outputs-land)).
* Os comandos de captura (`lattice capture`, `lattice array-capture` e as outras famílias de captura) indicam todos os ficheiros que não conseguiram gravar e saem com o código `1`; as capturas repetidas param quando a unidade fica cheia.

Códigos de saída: `0` sucesso · `1` falha genérica · `2` erro de argumento · `130` interrompido por Ctrl+C.

***

## Índices de vegetação

Execute o `--indices` com um ou mais nomes predefinidos; cada índice é guardado na sua própria pasta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Os 22 nomes predefinidos que o `process --indices` aceita:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

É adicionado ao projeto um nome que ainda não constava na lista. Um nome que não seja uma predefinição, ou que as bandas das câmaras não consigam formar, é ignorado, sendo emitido um aviso a indicar o mesmo. Se a CLI
não conseguir ler a lista de índices atual do projeto, mantém a lista inalterada e emite um aviso, em vez de a substituir.

{% hint style="warning" %}
**Existem três listas de índices — não as confunda.**O menu suspenso «Definições do Projeto» da GUI tem 27 fórmulas (adiciona `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — estas cinco são exclusivas da GUI e**não** são válidas para `--indices`). O comando «live/offline» `lattice index --preset` utiliza a sua própria lista separada de 22 predefinições. As fórmulas e os cálculos de bandas estão documentados em [Fórmulas de Índices Multiespectrais](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: uma introdução rápida

A família `daq pool-*` controla os sensores espectrais DAQ da MAPIR
(DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) através do conjunto persistente do backend — a GUI, a CLI
e o SDK
partilham todos um único identificador ativo. **O `pool-*` é o caminho DAQ suportado na CLI
fornecida**; outros subcomandos `daq` que possa ver referenciados são uma superfície interna da MAPIR
apenas de origem e saem com um erro explícito que o direciona para o `pool-*`.

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

`pool-record` sem `--duration` é executado até `pool-record --stop`; o diretório de saída predefinido é o `~/Documents/DAQ Live View/` **na máquina do backend**. O perfil de correção de limite é escolhido no momento da ligação (`--cap-id`, predefinição do backend `sunshine_cosine`) e pode ser alterado em tempo real com `pool-set-cap` — os perfis de limite e a gama calibrada do sensor são abordados nos capítulos sobre DAQ deste manual.

{% hint style="warning" %}
**DAQ-E num anfitrião com várias placas de rede:** a primeira deteção automática do `pool-connect --eth` após o arranque pode falhar, mesmo com um sensor em bom estado. O `--eth-host <ip-or-hostname>` é a opção fiável — recorra a ela sempre que a deteção não produzir resultados.
{% endhint %}

***

## Câmaras LATTICE, PTP e automatização de projetos

A família `lattice` (mais de 45 subcomandos) abrange todo o processo de trabalho com câmaras LATTICE: deteção, capturas únicas, matrizes sincronizadas persistentes com o fluxo de ligação «smart-prep» da GUI, pré-visualização ao vivo no navegador, alinhamento, cálculos de índice e diagnóstico da placa de rede do anfitrião. Uma amostra:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

A par disso: o `chloros-cli time-sync` apresenta um relatório sobre o «grandmaster» PTP que o anfitrião Chloros
executa (as câmaras LATTICE e os sensores DAQ-E funcionam como escravos deste para carimbos de data/hora entre dispositivos), e o `chloros-cli project` abre um projeto Chloros
guardado e controla as suas câmaras, matrizes e sensores sem interface gráfica — incluindo receitas de captura YAML programadas.

Estas três famílias (`lattice`, `project`, `daq pool-*`) são também as únicas que suportam o `CHLOROS_BACKEND_URL` para controlar um backend **remoto**; os comandos principais destinam-se sempre à máquina local.

Os tutoriais completos encontram-se nos capítulos sobre o LATTICE deste manual; todas as opções estão na [Referência da CLI
](reference/cli-reference.md).

***

## Resolução de problemas: as 5 principais

| Sintoma | Solução |
| --- | --- |
| O `Login required` ou uma tarefa agendada fica bloqueada num prompt do `Email:` | Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez nesta máquina — os comandos sem uma sessão em cache são executados de forma interativa, em vez de falharem imediatamente. |
| `backend unreachable` | Inicie a aplicação de ambiente de trabalho Chloros
ou inicie o backend manualmente: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` no Windows
(na pasta `backend` da sua pasta de instalação; não se encontra em `PATH`), `chloros-backend` no Linux
. `chloros-cli backend status` indica se algum está a ser executado; `lattice`/`project`/`daq pool-*` nunca iniciam nenhum. Se os configurar para um backend remoto, verifique o `CHLOROS_BACKEND_URL`. |
| Ligação à matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | O anel de receção da placa de rede do anfitrião foi reposto para os valores predefinidos — a principal causa de um equipamento que antes funcionava se recusar a ligar, normalmente após uma atualização do controlador da placa de rede. Execute o comando `chloros-cli lattice network --fix` a partir de um terminal com **privilégios elevados** (ou defina os parâmetros `ReceiveBufferLen=256` e `PendingReceives=64`); consulte a secção *Configuração e Ajuste da Placa de Rede do Anfitrião* da documentação de referência. |
| O subcomando `daq` termina com a mensagem: «requer o pacote DAQ completo…» | Utilize os comandos `daq pool-*`, que controlam o mesmo sensor através do backend: ligação, transmissão, gravação, seleção de cap e, para um DAQ-E, as suas definições de rede, um reinício e atualizações de firmware (ou `chloros_sdk.connect_daq_sensor()` a partir de Python
). |
| O Jetson apresenta um aviso de swap antes de pastas de grande dimensão | Adicione swap baseado em ficheiros — a CLI
apresenta os comandos exatos `fallocate`/`swapon` a executar. |

***

## Obter ajuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todos os sinalizadores, todos os subcomandos:** [Referência da CLI
](reference/cli-reference.md)
* **Equivalente em Python
:** [Python
SDK
](api-python-sdk.md) e a [Referência do SDK
](reference/sdk-reference.md)
* **Suporte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
