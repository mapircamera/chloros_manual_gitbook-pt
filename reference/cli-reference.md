# Referência do Chloros CLI

**Versão:**

1.2.0**Gerado:**29/07/2026 19:19 ·**Revisado:** 30/08/2026**Público-alvo:** Otimizado para utilização por LLM; legível por humanos.**Âmbito:** Todos os subcomandos do `chloros-cli` destinados ao utilizador, com opções e exemplos que podem ser copiados e colados.

Este documento constitui a referência completa para a ferramenta de linha de comandos `chloros-cli`, fornecida com o MAPIR Chloros. É intencionalmente exaustivo, para que um LLM (ou um ser humano) possa compor qualquer fluxo de trabalho suportado a partir das listas abaixo, sem necessidade de inspecionar o código-fonte.

Se precisar apenas dos pontos principais, avance para:
- [Guia de Início Rápido de Cinco Minutos](#five-minute-quickstart)
- [Fluxo de trabalho de primeira ligação da câmara LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxo de trabalho de primeira ligação do sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess)

---

## Convenções

- Todos os comandos têm o prefixo `chloros-cli`. Em Windows, o ficheiro binário é `chloros-cli.exe`; em Linux /Jetson é `chloros-cli`.
- Os argumentos opcionais são apresentados como `--flag`. Os argumentos posicionais obrigatórios são apresentados sem parênteses.
- Quando é fornecido um valor por predefinição, a omissão do sinalizador utiliza esse valor.
- O CLI é um cliente «thin» HTTP sobre o backend Chloros (servidor Flask em `127.0.0.1:5000`). O backend é iniciado automaticamente pela maioria dos comandos. `CHLOROS_BACKEND_URL=<url>` aponta para o **`lattice`**,**`project`**e**`daq pool-*`** para um backend remoto — os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixam deliberadamente o `http://127.0.0.1:<port>` e ignoram-no (o literal IPv4 evita Windows&#x27; `localhost`→`::1` ~2 spor pedido). Consulte [Variáveis de Ambiente](#environment-variables).
- É necessário um login com uma conta Chloros+ para todas as chamadas SDK / CLI (execute o `chloros-cli login` uma vez por máquina; armazenado em cache no `~/.chloros/`).
- Os exemplos utilizam os caminhos Linux; em Windows, substitua `/home/user/...` por `C:/Users/.../...`.

---

## Sinopse de nível superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opções globais

| Sinalizador | Descrição |
| --- | --- |
| `--backend-exe PATH` | Substituir o executável do backend detetado automaticamente. |
| `--port N` | Porta do backend HTTP (predefinição: `5000`). |
| `-v, --verbose` | Ativar saída detalhada. |
| `--restart` | Reiniciar forçadamente o backend (encerra qualquer `backend_server.py` em execução). |
| `--version` | Exibir a versão (`Chloros CLI 1.2.0`). |
| `--help` | Mostrar a ajuda de nível superior. |

### Índice de comandos

| Comando | Finalidade |
| --- | --- |
| [`process`](#chloros-cli-process) | Processar uma pasta de capturas «Survey3» ou «LATTICE» de ponta a ponta. |
| [`login`](#chloros-cli-login) | Autenticar esta máquina com uma conta Chloros+. |
| [`logout`](#chloros-cli-logout) | Limpar as credenciais armazenadas em cache. |
| [`status`](#chloros-cli-status) | Mostrar o estado atual da licença/autenticação. |
| [`export-status`](#chloros-cli-export-status) | Progresso em tempo real da exportação do Thread-4 durante uma execução do `process`. |
| [`language`](#chloros-cli-language) | Definir ou listar a língua de visualização dCLI (38 suportadas). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#comandos-da-pasta-do-projeto) | Pasta padrão do projeto (partilhada com a GUI). |
| [`update`](#chloros-cli-update) | Verificar e instalar atualizações do CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnósticos do sistema + testes de funcionamento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Estado/controlo do «grandmaster» PTP. |
| [`lattice`](#chloros-cli-lattice) | Controlo e captura da câmara LATTICE (mais de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Controlo do sensor espectral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir e executar um projeto guardado do Chloros (câmaras + DAQs). |

---

## Instalação

O `chloros-cli` é fornecido no instalador para ambiente de trabalho Chloros em todas as plataformas suportadas — não existe um download separado CLI. A instalação do pacote da plataforma adiciona o `chloros-cli` ao seu `PATH`, juntamente com a aplicação para computador e o binário de backend que esta controla.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> O instalador também inclui scripts de lançamento práticos (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abrem um shell CLI pronto a utilizar; estes são abordados no [Guia do Utilizador do CLI](../CLI.md) e não são repetidos aqui.

### Windows (.exe)

1. Descarregue o instalador do Windows a partir da página de downloads.
2. Execute o `Chloros-Setup-x.y.z.exe` e siga as instruções do assistente. O caminho de instalação predefinido é o `C:\Program Files\Chloros\` (o ficheiro «CLI» é guardado em `C:\Program Files\Chloros\cli\`, que o instalador adiciona à variável PATH).
3. Abra um novo terminal (`cmd.exe`, PowerShell ou Terminal dWindows) para que o `PATH` atualizado seja detetado.

```powershell
chloros-cli --version
```

O instalador adiciona automaticamente o `chloros-cli.exe` ao seu sistema `PATH` e inclui o runtime do Arena SDK necessário para as câmaras LATTICE.

### Linux amd64 (.deb)

Para o Ubuntu 22.04 LTS ou mais recente / estações de trabalho x86_64 baseadas no Debian.

> **O Ubuntu 20.04 não é suportado.** A lista de dependências do pacote deriva do
> que o backend efetivamente vincula, e isso inclui o `libc6 (>= 2.34)`;
> a versão «focal» inclui a glibc 2.31. O `apt` recusa a instalação em vez de permitir que esta falhe em
> tempo de execução.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

O ficheiro .deb instala:
- O `chloros-cli` para o `/usr/bin/chloros-cli`
- O backend compilado para o `/usr/lib/chloros/chloros-backend`
- O ambiente de execução do Arena SDK (para câmaras LATTICE)
- Modelos de redução de ruído, pacotes de calibração e configuração do canal de atualização

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

O mesmo layout que o .deb para amd64, com uma compilação CUDA otimizada para o Jetson Orin / Orin NX / Orin Nano.

### Autenticação única por máquina

Todas as plataformas requerem um login único em Chloros+ antes de as chamadas SDK / CLI funcionarem:

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache em `~/.chloros/user_session.json`.

### Verificar a instalação

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **É necessária uma subscrição do Chloros+.**O CLI requer um plano ativo do Chloros+.**Copper**é o nível de entrada Chloros+ — todos os níveis pagos do Chloros+ têm acesso a CLI / SDK; apenas o nível gratuito**Iron** não tem. (Correspondência de IDs de plano: `0`=Iron/gratuito, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Atualize em [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Este limite mínimo é imposto pelo backend, e não apenas pelo CLI: um pedido com os sinalizadores SDK / CLI sem um plano pago é rejeitado com o código `403 PLAN_UPGRADE_REQUIRED`, quer provenha do `chloros-cli`, do Python SDK ou de um cliente HTTP desenvolvido manualmente. Um utilizador que tenha terminado a sessão recebe, em vez disso, o código `401 AUTH_REQUIRED`. O acesso funciona offline durante o período de carência do plano (30 dias para planos mensais e até ao termo da validade para planos anuais) e cessa quando esse período expira; o código `chloros-cli status` continua a funcionar para que o motivo seja visível (é a rota SDK / CLI isenta do controlo de níveis — `GET /api/license-status`).

---

## Guia de Início Rápido de Cinco Minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Processar uma pasta de imagens através do pipeline completo Chloros (detecção de alvo → calibração → vinheta → refletância → exportação do índice).

### Sinopse

```
chloros-cli process INPUT [OPTIONS]
```

### Argumentos posicionais

| Argumento | Descrição |
| --- | --- |
| `INPUT` | Caminho para a pasta de entrada que contém os ficheiros `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou `.dng`. |

### Opções comuns

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `-o, --output PATH` | uma nova pasta com data e hora no caminho padrão do projeto (`~/Chloros Projects`, a menos que configurado) | Pasta do projeto a criar ou reutilizar. Se a pasta já contiver um ficheiro `project.json`, é criada uma pasta irmã `_1`/`_2` em vez de se sobrescrever. |
| `-n, --project-name NAME` | automático (carimbo de data/hora) | Nome do projeto. |
| `--debayer {standard,texture-aware}` | `standard` | O `texture-aware` utiliza um débayer neural do tipo Chloros+; mais lento, mas de maior qualidade. |
| `--vignette / --no-vignette` | `--vignette` | Correção de vinheta. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibração de refletância (utiliza alvo do painel, se encontrado, e calibração NIST por série para o LATTICE). Para o LATTICE multiespectral, isto funciona também como o botão de alternância do **produto** de refletância — consulte [Botões de alternância de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desativado | Aplicar correções PPK GNSS a partir de ficheiros sidecar. |
| `--exposure-pin-1 MODEL` | desativado | Fixar o modelo «Survey3» de um equipamento com duas câmaras modelo «pin-1». |
| `--exposure-pin-2 MODEL` | desativado | Fixar o modelo «pin-2». |
| `--recal-interval SECONDS` | 0 | Forçar a repetição dos cálculos de calibração a cada N segundos de tempo de captura. |
| `--timezone-offset HOURS` | local | Substituir o desfasamento do fuso horário incorporado nos metadados de saída. |
| `--format FORMAT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para ficheiros TIFF do LATTICE (o Survey3 .raw não é afetado). Além disso, a «saída de emergência» que permite que uma captura **sem raw** seja processada na totalidade — ver [Como é uma pasta de capturas](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ativado | Emite o produto linear pós-debayering (`Debayered_Images`). Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ativado | Emite a pré-visualização de ecrã (`Preview_Images`): RGB = equilíbrio de brancos (iluminante DAQ, quando disponível; caso contrário, «gray-world») + gama; multispec = alongamento de cores falsas. |
| `--radiance / --no-radiance` | ativado | Emite radianância float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referência para o produto de refletância LATTICE: `auto` = o alvo de- o alvo que passa no quadro é a referência absoluta, recurso de fallback DAQ-downwelling (ρ = π·L/E); `target` = estrito (sem substituição DAQ); `daq` = DAQ-autoritário. Ver [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nenhum | Diretório de varreduras de refletância do alvo **medida** (`<serial>.csv`); recorre aos espectros nominais T3/T4P em caso de falha. |
| `--array-alignment / --no-array-alignment` | ativado | Matrizes LATTICE: aplica o alinhamento módulo a módulo registado no ficheiro XMP de cada captura (`Chloros:Alignment*`) a todos os produtos processados (desbayering / pré-visualização / radianância / refletância / índice). Não realiza nenhuma operação em imagens sem as etiquetas. |
| `--array-alignment-crop / --no-array-alignment-crop` | recorte | Recorta as exportações alinhadas à região de sobreposição comum da matriz, para que todos os módulos partilhem uma única área de cobertura; o `--no-…` mantém a área total do sensor (preenchimento a preto fora da fonte). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reamostragem para a distorção de alinhamento. O `nearest` preserva os DNs exatos da fonte (sem mistura de valores radiométricos entre píxeis). |

### Opções de deteção de alvos

| Indicador | Descrição |
| --- | --- |
| `--min-target-size PIXELS` | Tamanho mínimo do painel-alvo (px) para o detetor. |
| `--target-clustering 0-100` | Sensibilidade de agrupamento. |
| `--target / --targets` | Tratar a pasta de entrada como sendo apenas de painéis-alvo (ignorar a deteção de levantamentos). |

### Exemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Porproduto (LATTICE multiespectral)

O processamento LATTICE ramifica-se em **todos os produtos aplicáveis numa única passagem**. Os quatro interruptores por tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — estão todos**ativados por predefinição**; utilize o formulário `--no-<type>` para desativar um. As câmaras principais RGB emitem apenas imagens sem debayering + pré-visualização (sem radiação/refletância por banda), pelo que `--radiance`/`--reflectance` são sem efeito para elas. Os interruptores são ignorados para Survey3 `.raw` (que segue o caminho padrão de refletância/alvo). *(O antigo sinalizador `--radiometric-output {reflectance,radiance,sensor-response}` foi **removido** e substituído por estes comutadores; já não existe o nível `sensor-response`.)*

| Produto | Saída | É necessária a descida de dados DAQ? |
| --- | --- | --- |
| `--debayered` | Desmosaico linear (`Debayered_Images`). | Não. |
| `--preview` | Pré-visualização do ecrã (`Preview_Images`): RGB = WB + gama; multispec = falso-estiramento de cor. | Não. |
| `--radiance` | float32 W/m²/sr/nm proveniente da cadeia radiométrica completa (`Radiance_Images`). | Não. |
| `--reflectance` | uint16 refletância ρ (`32768` = 1,0), compatível com Pix4D. | **Sim**, a menos que seja ancorado por um alvo no quadro que tenha passado no controlo de qualidade (ver abaixo). |

`--reflectance-source` seleciona a referência de refletância:**`auto`**(predefinição) torna um alvo dentro do quadro que tenha passado no controlo de qualidade (QA) a**referência absoluta**— as cadeias de linhas empíricas ancoradas no alvo são cruzadas em painéis de validação e o resultado vencedor medido é aplicado — recorrendo à divisória descendente do DAQ (ρ = π·L/E) quando não há nenhum alvo presente ou o controlo de qualidade falha;**`target`**é rigoroso (sem substituição do DAQ);**`daq`**opta pelo comportamento em que o DAQ tem autoridade. A geometria do alvo (ArUco / ROI fixo / faixa) provém da configuração de alvos do projeto; `--target-reflectance-dir DIR` mantém as varreduras**medidas** por unidade (`<serial>.csv`) consultadas através do número de série/QR da unidade-alvo, com os espectros T3/T4P nominais como alternativa.

O caminho de refletância do DAQ resolve automaticamente a **radiação descendente com carimbo temporal correspondente**a partir de um**`.daq`**gravado (DAQ-U/M/E)**ou um `.csv` nativo do DAQ-M**encontrado juntamente com as imagens. Se um pacote de calibração por câmara ou DAQ não estiver armazenado em cache localmente, o pipeline**obtém-no automaticamente da AWS** na primeira utilização (requer ligação à Internet uma única vez; armazenado em cache sob o nome `~/.chloros/`).

#### Leitura de píxeis de refletância (Pix4D / Metashape / os seus próprios scripts)

A refletância é armazenada como um valor inteiro DN, e **o valor DN que corresponde a ρ = 1,0 depende da câmara de origem**:

| Origem | ρ = 1,0 corresponde a | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem até ρ 2,0) | O XMP `Chloros:PixelScale=32768` está gravado no ficheiro. |
| Survey3 | `65535` (cortado em ρ 1,0) | Não há etiquetas XMP `Chloros:*` — essa ausência *é* o sinal. |

**Leia `Chloros:PixelScale` e divida por esse valor** em vez de assumir uma constante. A etiqueta está definida no domínio uint16, pelo que se mantém como `32768` em todos os formatos de saída que redimensionam — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` são todas autodescritivas (normalizam primeiro o tipo de dados armazenado de volta para uint16 primeiro: ×257 a partir de 8 bits, ×65535 a partir de float).

> **Um caso não tem escala, por definição.** Quando uma captura deé gravada como «TIFF» de 8 bits, o pipeline *limita* o valor a 0..255 em vez de reescalar, pelo que todos os valores acima de ρ≈0,008 são nivelados para 255 e nenhuma escala descreve o ficheiro. Chloros omite deliberadamente tanto o `Chloros:PixelScale` como o tuplo `MicaSense:RadiometricCalibration`, e regista o motivo. **Se a etiqueta estiver ausente num ficheiro de refletância LATTICE, não presuma que existe uma escala — reexporte a 16 bits ou 32 bits** em vez de dividir píxeis que nunca foram divisíveis.

#### EXIF transferido para a exportação

O `process` copia o **bloco GPS e o seu ExifIFD** da captura de origem para cada produto, pelo que uma
exportação inclua `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` juntamente com a georreferência.

**O `FocalLength` não é opcional para a fotogrametria.** O Pix4D calcula a distância entre amostras no solo a partir da
distância focal mais a altitude; sem a etiqueta, recorre a uma escala extremamente errada. Num
voo de 49 capturas sobre um pomar de laranjeiras, a falta da etiqueta transformou um local de 411 m × 160 m numa reconstrução de
47,8 km × 13 km — uma ortofotografia de 455 MP composta principalmente por «nodata», o que foi então interpretado como um problema de mosaico e
um problema com o BigTIFF antes de alguém verificasse a GSD. Se a sua ortofotografia apresentar uma escala implausível,
execute primeiro o `exiftool -FocalLength` no produto exportado.

A cópia **não** é deliberadamente o `-all:all`: as etiquetas estruturais do IFD0 corrompem a saída do LATTICE quando
copiadas, e o `ExifImageWidth` / `ExifImageHeight` são excluídos porque descrevem a
*captura de origem* — uma exportação que já tenha sido redimensionada acabaria por conter dimensões
contraditórias em relação à sua própria imagem rasterizada. O XMP é gravado diretamente em vez de ser copiado, porque o ExifTool
descartou as etiquetas XMP da mesma invocação quando o bloco XMP é copiado (o que eliminaria as etiquetas de calibração MAPIR
).

### Onde são guardados os resultados

Os produtos são gravados **na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para LATTICE (correspondendo ao EXIF da captura
`Model`) e `<model>_<filter>` para a Survey3 — duas câmaras que partilham um sensor e um filtro, mas que diferem
na lente mantêm árvores separadas, porque a vinheta, o campo de visão e a distorção diferem. A pasta de
formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para
`TIFF (32-bit, Percent)`.

> **Cada produto exportado mantém o nome do ficheiro ORIGINAL.** Uma exportação do Radiance de
> `capture_…_raw.tif` continua a chamar-se `capture_…_raw.tif` — apenas se encontra em
> `tiff32/Radiance_Images/`. **É a pasta que identifica o produto, não o nome do ficheiro**, por isso, a pesquisa global
> por `*radiance*.tif` não encontra nada; em vez disso, procure no diretório.

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

O `process` também processa as gravações `.daq` na sua pasta de entrada e **não**
precisa de imagens para o fazer: um DAQ-U / DAQ-M / DAQ-E utilizado isoladamente constitui uma
sistema de captura, e uma pasta que contenha apenas ficheiros `.daq` é uma entrada válida.

Um DAQ pode ser gravado **sem** a sua calibração — é isso que os gravadores públicos
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fazem por predefinição: gravam as contagens brutas do sensor e marcam o ficheiro de forma a que
Chloros recupere a calibração de fábrica desse sensor **por porta série** (primeiro na cache local,
depois na nuvem MAPIR) e a aplique. O `process` grava o resultado de volta:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

O `.csv` contém uma linha por leitura: carimbo de data/hora UTC, tempo de integração, potência total,
lux fotópico/escotópico em lux, PPFD (e a sua divisão em azul/verde/vermelho), comprimento de onda de pico e, por fim, o
espectro completo na grelha de comprimentos de onda do próprio sensor. O `.daq` é reimportado sem ser
calibrado uma segunda vez.

Em caso de sucesso, a execução reporta `Light-sensor products written: N (calibrated .daq + .csv)`.
O texto entre parênteses descreve o que foi efetivamente gravado, pelo que se lê
`(RAW COUNTS — this sensor has no calibration bundle)` para um sensor sem pacote e
`(N calibrated, M raw counts)` para uma pasta que contém ambos. Os títulos do próprio backend
`[DAQ-EXPORT]` e `[RUN-SUMMARY]` derivam a sua formulação da mesma forma — nenhum dos
três pode considerar uma exportação em bruto como calibrada.

Uma gravação DAQ-U / DAQ-M / DAQ-E cujo pacote de calibração não possa ser obtido — porque está
offline ou porque esse sensor não tem nenhuma calibração registada — é **ignorada com indicação do motivo** numa
linha `[DAQ-EXPORT]`, nunca sendo gravada como um ficheiro «calibrado» contendo contagens brutas.
Ligue-se à Internet e volte a executar. O motivo é aquele que o leitor efetivamente
estabeleceu para esse ficheiro (esquema ilegível, ausência de pacote, um erro de gravação), e o resumo da execução
lista motivos **distintos** — vinte ficheiros ignorados por uma única causa são apresentados como uma
única causa, e não como vinte repetições da mesma.

#### As gravações DAQ-A são exportadas como contagens brutas

A família **DAQ-A** é anterior ao sistema de pacotes por número de série e não tem nenhum pacote de calibração
para obter — em vez disso, é calibrada no terreno em relação a um alvo de refletância, razão pela qual
nunca precisou de um. A recusa dessas gravações deixou-as sem qualquer forma de obter os seus
números, pelo que são exportados com um **nome diferente**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Um nome de ficheiro diferente, em vez de um indicador dentro do ficheiro, porque a informação tem de sobreviver
ao envio por e-mail apenas como um nome simples. O cabeçalho `.csv` indica
`raw spectral sensor counts (NOT irradiance)` e avisa que os valores são comparáveis
**dentro** do ficheiro — que é exatamente para isso que a calibração baseada em alvo os utiliza — e
não entre sensores. As colunas fotométricas dependentes da potência (potência total, lux fotópico e
escotópico, PPFD) são registadas como **NULL** em vez de serem integradas a partir de contagens, e o resumo
da execução indica `RAW COUNTS`, pelo que os dados «exportados» num registo não pode ser interpretado como irradiância.

As gravações antigas **v1.01 / v1.02** (gravadas por um DAQ-A-SD) não incluem uma época por leitura,
apenas a. O comparador de imagens ↔ de luz descendente continua a rejeitá-las — comparar um
quadro com uma hora de gravação seria um erro imperceptível — mas o exportador lê-as, e
o CSV imprime `clock=daq_created_on`, pelo que o produto indica em que relógio se encontra.

### Notas

- O `process` deteta automaticamente se a sua pasta é do tipo «Survey3», «LATTICE» ou mista.
- O progresso é transmitido através de «Server-Sent Events»; o CLI mostra o progresso em tempo real por thread («Detecting», «Analyzing», «Processing», «Exporting»).
- Para o Linux /Jetson, o CLI verifica a memória de troca e pode emitir um aviso antes de processar pastas de grande dimensão. O «debayer» sensível à textura também aplica automaticamente um limite de frequência da GPU em Jetsons de baixo consumo (Nano, Orin Nano).
- Se for bem-sucedida, a execução indica quantos produtos de imagem foram gravados (`Image products written: N`).

#### Uma execução que não grava imagens falha

Se solicitou produtos e a execução não gravou **nenhum** — apenas `project.json` e
`calibration_data.json` — o `process` trata isso como uma falha: apresenta
`Processing finished but wrote no image products.` e **termina com um valor diferente de zero**, para que um script possa
detetá-lo. A mensagem indica o nome da pasta do projeto e as causas habituais:

- a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou
- todos os produtos solicitados foram ignorados por serem inaplicáveis a essas câmaras (por exemplo, solicitar
  radianância/refletância de câmaras do tipo «RGB ).

Execute novamente com o `--verbose` e verifique o registo do backend nas linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explicam as omissões por câmara que, de outra forma, não chegariam à saída do CLI.

Uma execução deliberada apenas com metadados — todos os produtos desativados e sem `--indices` — continua a ser um
**sucesso**, porque uma saída de imagem vazia é o resultado correto nesse caso.

O mesmo se aplica a uma **execução apenas com o sensor de luz**: uma pasta de gravações `.daq` não tem imagens para exportar
por definição, e a execução é avaliada com base no `.daq` / `.csv` calibrado que escreveu em vez disso.

---

## `chloros-cli login`

Autentique esta máquina com uma conta na nuvem Chloros+. As credenciais são armazenadas em cache de forma segura em `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (removendo partes da palavra-passe ou duplicando-as). Em caso de erro 401, o CLI tenta automaticamente novamente, acrescentando `$$` e, em seguida, com metade da palavra-passe sem repetições; se a nova tentativa for bem-sucedida, o sistema inicia a sessão e apresenta a sintaxe correta com aspas simples a utilizar na próxima vez.

> **Utilização sem interface gráfica/por script: a ausência de uma sessão em cache significa um prompt interativo, e não uma falha imediata.** Qualquer comando que inicie um backend (`process`, `status`, `export-status`, `time-sync`, …) executado sem uma licença/sessão em cache entra num prompt interativo `Email:` / `Password:` na stdin antes de prosseguir. Uma tarefa não supervisionada sem sessão em cache ficará, portanto, bloqueada à espera de entrada — execute o `chloros-cli login EMAIL PASSWORD` uma vez por máquina antes de agendar trabalho sem interface gráfica.

---

## `chloros-cli logout`

Limpa a sessão em cache e força um novo início de sessão na próxima chamada.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Mostra o nível de licença atual (Iron/Copper/Bronze/Silver/Gold), o utilizador autenticado e o número de ligações de dispositivos.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifica o progresso em tempo real da exportação do Thread-4. É seguro chamar **durante** a execução de um `process` a partir de outro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Define o idioma de visualização do «CLI» (38 idiomas suportados, incluindo CJK, RTL e Indic). Recorre ao inglês de forma harmoniosa em consolas antigas que não conseguem renderizar o script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemplos

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandos da pasta do projeto

Estes comandos gerem a localização predefinida da pasta do projeto (partilhada com a GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Apenas para Jetson. Verifica o `version_url` a partir do `/etc/chloros/update.conf` e oferece a possibilidade de descarregar e instalar o `.deb` correspondente.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

No Linux /Jetson, o CLI também executa uma **verificação automática de atualizações em cada arranque** (sem bloqueio, nunca atrasa o comando): lê o `/etc/chloros/update.conf`, armazena o resultado em cache durante 1 hora no `~/.chloros/update_cache.json` e apresenta o `Update available: vX.Y.Z / Run: chloros-cli update` quando existe uma versão mais recente. É ignorado silenciosamente em caso de erro e em Windows.

---

## `chloros-cli selftest`

Executa um teste de funcionamento de 7 etapas: versão, disponibilidade de portas, arranque do backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presença do modelo de redução de ruído, disponibilidade de CUDA + redução de ruído.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Estado e controlo do «grandmaster» PTP. O anfitrião «Chloros» executa o «grandmaster» PTP; as câmaras LATTICE e as unidades DAQ-E funcionam como escravas deste para a obtenção de carimbos de data/hora entre dispositivos.

| Subcomando | Descrição |
| --- | --- |
| `status` | Mostrar o estado do grandmaster, as prioridades BMCA e a identidade do relógio. |
| `peers` | Listar os escravos detetados através do Delay_Req (câmaras + sensores DAQ-E). |
| `cameras` | Estado do PTP por câmara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar o processo «grandmaster». |
| `set-priority --priority1 N --priority2 N` | Anular as prioridades do BMCA. |

### Exemplos

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controlo da câmara LATTICE. Todos os subcomandos passam pelo backend Chloros; o backend é o proprietário do conjunto de câmaras, pelo que as chamadas subsequentes a CLI reutilizam o mesmo identificador aberto.

### Opções comuns (partilhadas pela maioria dos subcomandos)

| Sinalizador | Descrição |
| --- | --- |
| `-d, --device N` | Índice da câmara (predefinição: 0). |
| `-s, --serial SN` | Número de série específico; substitui `--device`. |
| `--serials SN1,SN2,…` | Números de série separados por vírgulas para operação com várias câmaras. |
| `--all` | Operar em todas as câmaras detetadas. |
| `--exposure US` | Tempo de exposição em microssegundos. |
| `--gain DB` | Ganho em dB. |
| `--pixel-format FMT` | por exemplo, `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensões da imagem. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplicar uma predefinição de configurações. Todos funcionam em modo livre, exceto o `triggered`, que arma a câmara para uma flutuação de hardware na Linha 2 — sem qualquer sinal a acionar essa linha, a câmara ficará à espera indefinidamente em vez de capturar. |
| `-o, --output DIR` | Diretório de saída (padrão: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamanho do pacote GVSP. O `auto` executa sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxo de trabalho da primeira ligação da câmara LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referência de subcomandos

#### Detecção e informação

| Subcomando | Finalidade |
| --- | --- |
| `lattice info` | Listar câmaras ligadas (fabricante, modelo, número de série, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analisa o sistema anfitrião para obter a configuração ideal da câmara. O `--no-discover` ignora a deteção de câmaras (mais rápido, análise apenas pela placa de rede). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Verificar/corrigir as definições da placa de rede; estimar largura de banda/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Capacidade de rede do backend com esquema estável + recomendação de matriz (retorna `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). O `auto_capped_fps` mantém a resolução solicitada, mas limita os FPS alvo — leia o `recommended.recommended_target_fps` e passe-o como destino de ligação; trate-o como sucesso, não como um erro. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Análise de cenários hipotéticos sem abrir as câmaras. **`--n-active` é o número total de câmaras na rede, não apenas as desta matriz**— aumente-o quando câmaras autónomas transmitirem em simultâneo, ou se o orçamento da rede for calculado com base numa procura que as subestime (predefinição: `len(--models)`). Imprime sempre as linhas agregadas `Wire budget:` (MB/s exigidos vs. limite máximo seguro contra colisões) e `Max cameras:`, e sinaliza `** OVER-SUBSCRIBED**` quando o conjunto sobrecarrega a ligação — consulte [Modelo de fps e burst do conjunto](#array-fps--burst-model). |
| `lattice gpu` | Mostrar o estado da GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verificar ou atualizar o firmware da câmara. A seleção local de `.fwa` está fixada: o ficheiro em `firmware/<MODEL_PREFIX>/` que corresponde ao 5 da compilação5 da compilação é gravado quando presente (a versão mais recente apenas como alternativa), pelo que uma imagem do fornecedor mais recente armazenada no disco permanece inativa até que esse pino seja alterado — as versões deliberadamente mais recentes chegam às unidades através do manifesto AWS assinado, que é preferido quando mais recente. |
| `lattice presets [--apply NAME]` | Listar ou aplicar predefinições da câmara. |
| `lattice status` | Estado da câmara em tempo real. |

#### Captura

| Subcomando | Finalidade |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Fotograma único. **Guarda todos os tipos de exportação por predefinição** (`--processing all`); consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). `--levels` guarda um subconjunto explícito (substitui `--processing`); `--force-daq` grava a leitura DAQ atribuída como um ficheiro `.daq`, mesmo numa captura apenas em formato bruto. `--jpeg-quality` = JPEG qualidade 1–100 (padrão 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Transmite para o disco até Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Pré-visualização MJPEG em tempo real no navegador. `--ae-damping` define o amortecimento da exposição automática (0,4–100). |

#### Ajuste do sensor

| Subcomando | Finalidade |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Ler/gravar qualquer nó GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Exposição e AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Ganho e ganho automático. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | ROI do sensor e binning. |
| `lattice format [--set FMT] [--list]` | Formato de pixel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Disparador de hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sem sinalizadores = WB de disparo único) | Operações de WB. Apenas câmarasRGB/Bayer; uma operação nula (ignorada) em M3M monocromático. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Pipeline de cor de exibição «RGB». `natural` (predefinição) é o acabamento em tempo real mais económico; `enhanced` adiciona remoção de franjas + vibrância + contraste local CLAHE para o aspeto completo de paridade de hub a cerca de 2× o custo do acabamento por fotograma, pelo que resulta numa taxa de fotogramas **em direto** mais baixa — as capturas guardadas obtêm sempre o acabamento completo, independentemente do caso. RGB Apenas para câmaras /Bayer; ignorado na M3M monocromática. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Exibir saturação/contraste (câmaras com filtro RGB). Ignorado em M3M monocromáticas. |
| `lattice filter [--set NAME] [--list]` | Definir o modelo de filtro da câmara (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Verifica os nós de alimentação/térmicos; alterna para o modo de inatividade de baixo consumo. |

#### Calibração e sensores

| Subcomando | Finalidade |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de um alvo de refletância. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Comandos integrados de luz descendente. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplicar correção de vinheta a imagens existentes. |

#### Multicâmara (Sessões Transitórias)

| Subcomando | Finalidade |
| --- | --- |
| `lattice multi-info` | Listar todas as câmaras com funções de sincronização. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Um fotograma sincronizado de cada câmara. Guarda **todos os tipos de exportação por predefinição**quando está ligada uma matriz persistente; o recurso de fallback transitório sem matriz é**apenas desbayered** (execute primeiro o `array-connect` para o resto). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Transmite fotogramas sincronizados (transitório). |
| `lattice multi-test [--count N]` | Teste de temporização de sincronização GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Detecção automática da ligação mestre/escravo GPIO. |

#### Alinhamento

| Subcomando | Finalidade |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — além dos parâmetros do detetor/comparador `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, dos parâmetros RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, da combinação de múltiplos fotogramas `[--averaging mean\|median\|inlier_weighted]`, restrições geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrição espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` e substituições por escravo `[--per-cam-override SN:KEY=VALUE]` (repetível) | Calcular o perfil de alinhamento a partir de câmaras em tempo real. O `--prefilter` assume por predefinição o valor do `gradient` (mapa de arestas; corresponde ao alinhador da GUI/matriz — as arestas mantêm-se entre bandas espectrais). O `--matcher flann` compensa a partir de cerca de 5000 características; o `--averaging median` é robusto face a uma captura defeituosa, o `inlier_weighted` pondera com base no número de correspondências; `--lock-scale` projeta para a rotação mais próxima (sem escala), `--lock-axis` zera uma componente de translação; `--mask` aplica-se a todas as câmaras (utilize `--per-cam-override` para definições por câmara, por exemplo, `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` recusa-se a guardar uma calibração cujo RMS de reprojeção exceda o limite. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Captura um fotograma multibanda alinhado. O `--bit-depth`, por predefinição, ajusta-se à câmara; o `--no-crop` mantém o fotograma completo (preenchendo com preto); o `--interpolation` (padrão `linear`) e `--border-mode`/`--border-value` (padrão `constant`/0) controlam a distorção da CPU — o caminho da GPU é bilinear independentemente disso. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Quadros multibanda alinhados ao fluxo (os mesmos parâmetros de warp que `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Apresentar detalhes do perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Alterar a ordem das camadas. |

#### Índice / Cálculos de vegetação

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Conjunto completo de sinalizadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (repetível), `--capture-level raw|debayered|radiance|reflectance|unknown` (substitui o nível de captura registado no TIFF de origem; predefinição: lido a partir dos metadados de TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Com o `--live`, os botões de distorção de alinhamento também se aplicam: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Os símbolos `--channel` distinguem maiúsculas de minúsculas.** O lado do símbolo deve corresponder exatamente aos nomes dos canais da predefinição (as predefinições utilizam minúsculas, por exemplo, NDVI = `red`, `nir` — verifique `--list-presets`), e a parte da banda deve corresponder a um nome de banda na pilha alinhada (ou ser um índice de banda com base em 0 no modo offline). `--channel red=Red_660 --channel nir=NIR_850` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.

#### Ligações persistentes (Smart-Prep, fluxo equivalente à GUI)

Estes comandos mantêm as câmaras abertas no conjunto de backends ao longo de várias invocações dCLI.

| Subcomando | Finalidade |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adicionar uma câmara ao conjunto (câmara única, sem matriz). |
| `lattice cam-disconnect [--serial SN] [--all]` | Liberar. |
| `lattice cam-list` | Listar câmaras no conjunto. |
| **`lattice array-connect`**|**Ligar um conjunto sincronizado persistente (O ponto de entrada recomendado).** Executa o fluxo completo de preparação inteligente da GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Liberar um conjunto. |
| `lattice array-list` | Listar matrizes ligadas. |
| `lattice array-status [--array-id ID]` | FPS em tempo real, PTP, último erro. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Uma captura sincronizada da matriz em tempo real — Única / Contínua / Por intervalo / Mais rápida. **O valor predefinido é `all`** (um ficheiro por tipo de exportação aplicável por câmara). As câmaras ignoradas (p.RGB, excluídas da radianância/refletância) são indicadas com `Skipped: SN:<serial> (<reason>)`; a leitura DAQ utilizada para a refletância é guardada em simultâneo e indicada com `DAQ: <path>`. Consulte [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Gravar a visualização do índice combinado em tempo real em vídeo/GIF (nível de monitorização; requer que o fluxo combinado esteja aberto). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Série de imagens Bayer brutas com alta taxa de fotogramas por segundo (nível de análise; reprocessar offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocessar uma sequência «raw» guardada em vídeo(s) calibrado(s). |

##### Opções do `array-connect`

| Indicador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detetar automaticamente todas as câmaras LATTICE (necessita de ≥2) | O primeiro número de série é o MASTER. Quando omitido, a deteção filtra para os modelos LATTICE (`TRI032*`) e liga todos eles. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | automático | Taxa de disparo do gatilho do mestre. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituir o seletor de nível. |
| `--wire-ceiling-mbps MB_PER_S` | detetado automaticamente | **O orçamento de banda sustentado do anfitrião, em MB/s — o valor do qual depende toda a alocação da matriz.** Reduza-o quando a matriz reportar quadros com GVSP corrompidos: o valor automático é derivado da, que sobrestima adaptadores USB, pistas PCIe de baixa largura de banda e estruturas partilhadas ocupadas. Persiste no bloco de captura da matriz do projeto, pelo que uma reabertura / CLI / SDK ou uma reconexão o restaura. Consulte [Integridade da matriz](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Agrupamento de hardware. |
| `--no-recommend` | desativado | Ignorar a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativar PTP (os carimbos de data/hora entre câmaras **não** são, então, comparáveis). |

### Smart-AE / Smart-Capture

Os arranjos LATTICE executam AE contínuo em segundo plano assim que são ligados, mas uma cena recém-focalizada demora um momento a convergir. O `array-capture --smart` é a **opção de conveniência integrada**: aguarda que o AE se estabilize em todas as câmaras da matriz e, em seguida, aciona a captura. Utilize-a quando mudar de cena a meio de uma sessão.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

A política de estabilização é conservadora por predefinição: tempo limite de 5 s, janela de estabilidade de 1,5 s, tolerância de variação de exposição de ±5 %. Ajuste através do «SDK» (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se precisar de um comportamento diferente da automação.

### Níveis de exportação de captura (o valor predefinido de `all`)

A partir desta versão, `lattice capture`, `lattice multi-capture`, e `lattice array-capture` **por predefinição para `--processing all`** — um ficheiro guardado por tipo de exportação que se aplica a cada câmara, correspondendo ao comportamento «Capturar Tudo» da GUIcomportamento «Capturar tudo» da interface gráfica do utilizador. Os níveis são:

| Nível | Saída | Aplica-se a |
| --- | --- | --- |
| `raw` | Bayer de canal único (câmaras monocromáticas: a banda única) diretamente do sensor. | Todas as câmaras. |
| `debayered` | Demosaico BGR de 3 canais (câmaras monocromáticas: escala de cinzentos de 1 canal). | Todas as câmaras. |
| `radiance` | float32 W/m²/sr/nm através da cadeia radiométrica completa. | Apenas multiespectral (M3C/M3M) apenas — **ignorado para câmaras com filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatível com Pix4D. | Multiespectralespectral apenas, e **apenas quando um DAQ estiver associado + a câmara estiver calibrada**; caso contrário, ignorado. |
| `preview` / `display` | Cadeia completa de pré-visualização na GUI (CCM + WB + gama de acordo com o perfil da câmara). `lattice capture` denomina isto como `preview`; `array-capture`/`multi-capture` utilizam o `display`. | Todas as câmaras. |

Passe um único nível para guardar apenas esse (`--processing debayered`). Quando se solicita o `all`, os níveis que não se aplicam a uma determinada came são ignorados (e indicados), não gerando erros — uma câmara não ligada ou não calibrada continua a receber `raw` / `debayered` / `preview`.

Para qualquer fotograma de refletância, a leitura de luz descendente do DAQ efetivamente utilizada é gravada num ficheiro **`.daq`** anexo ao lado das imagens (para que a captura possa ser reprocessada posteriormente) e indicada numa linha `DAQ:`.

### Como se apresenta uma pasta de capturas

Cada tipo de exportação é colocado na sua **própria subpasta** em `-o`, pelo que uma captura de vários níveis nunca mistura tipos:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` é a data e hora da captura e `<serial>` o número de série da câmara, pelo que um grupo sincronizado partilha uma
data e hora entre as câmaras. **Repare na única assimetria:** o nível `display` é armazenado numa pasta
denominada `preview/`, enquanto os próprios ficheiros mantêm `_display` no nome — a pasta e o sufixo diferem
apenas esse nível. Os níveis desconhecidos recorrem a uma pasta com o seu próprio nome e, se a subpasta
não puder ser criada, o ficheiro é gravado na raiz de saída, em vez de se perder.

**Reprocessar uma pasta de capturas:**aponte o `chloros-cli process` para a**raiz das capturas**
(`output/`). O `process` normalmente importa apenas a pasta que indicar, mas quando essa pasta não contém
imagens e possui subpastas, percorre-as automaticamente — assim, as subpastas do nível raiz e a
raiz `.daq` são todas selecionadas de uma só vez. Cada nível de uma captura é importado como uma única imagem, com
os outros níveis disponíveis como modos, em vez de uma imagem por nível.

Nomear diretamente uma **subpasta de nível** (por exemplo, `output/raw/`) também funciona. Ao fazê-lo, a raiz
`.daq` para trás; por isso, copie ou indique a leitura do DAQ juntamente com ela quando voltar a derivar um produto radiométrico
a partir de `raw/` — caso contrário, a correspondência do carimbo de data/hora não terá nada com que se relacionar.

**O processamento começa sempre a partir de `raw`.** Em cada captura, o fotograma bruto é a fonte do pipeline;
`debayered`, `radiance`, `reflectance` e `preview` surgem como modos de visualização, mas nunca são reenviados
através do pipeline. O reprocessamento de um produto derivado reaplicaria a vinheta, o CCM e
os cálculos de radiação que já estão incorporados nos seus píxeis, pelo que o Chloros recusa essa operação em vez de
efectuar um duplo processamento. Duas consequências que vale a pena conhecer:

- As renderizações `index/` e `composite/` **nunca** são processadas. São saídas, não capturas —
  uma renderização LUT do tipo «NDVI» não tem qualquer interpretação significativa de radiação.
- Uma pasta de capturas exportada **sem** `raw` (por exemplo, `array-capture --processing reflectance`) não tem
  uma fonte legítima no pipeline. Essas capturas são importadas e exibidas normalmente, mas o `process` ignora-as
  e indica isso:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se precisar realmente de fazer passar um produto derivado — uma sessão de hub capturada com
  o `demosaic` ativado, ou uma pasta legada — o `--input-level {raw,debayered,processed}` força o ponto de entrada
  e anula a omissão. Esse sinalizador é a via de escape deliberada; `auto` (o valor predefinido)
  nunca processa uma captura que não tenha dados brutos.

### Capturas ignoradas em matrizes com filtros mistos

Quando se misturam câmaras «RGB» e multiespectrais numa única matriz, o `array-capture --processing radiance` (ou o `reflectance`) guarda os fotogramas multiespectrais e **ignora** as câmaras «RGB» — a radianância por Bayer não é significativa para um sensor de banda larga. O CLI apresenta explicitamente cada ficheiro guardado (com o seu nível de exportação), cada `.daq` gravado e cada omissão, pelo que o número de ficheiros não é surpreendente:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Ignorar— os tokens de motivo de omissão seguem o padrão `<level>-not-applicable-to-rgb-cam`. A refletância também pode ser omitida com `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` e com `dls-uncalibrated-band-<nm>` quando a banda se situa maioritariamente fora da(~374–974 nm) — entre os SKUs disponíveis, apenas o F988, cujo percurso suportado é o fluxo de trabalho do painel de refletância.

Utilize `--processing debayered` (ou `display`) para incluir todas as câmaras, independentemente do tipo de filtro, ou o `all` predefinido para obter todos os níveis aplicáveis por câmara de uma só vez.

---

## Modos de captura, gravadores e reprocessamento offline

Todos estes funcionam numa **matriz persistente** (execute primeiro o `array-connect`). Espelham o painel de captura da GUI.

### Modos do `array-capture`

O `array-capture` é um comando único com quatro modos de obturador, além de um conjunto de opções de exportação:

| Modo | Indicador | Comportamento |
| --- | --- | --- |
| **Único** *(predefinição)* | (nenhum) | Um grupo de captura sincronizado, depois sai. |
| **Contínuo** | `--continuous` | Passagens consecutivas até `Ctrl+C`, `--count N` ou `--duration S`. |
| **Intervalo** | `--interval S` | Uma passagem a cada `S` segundos (medidos a partir do início de cada passagem), com os mesmos limites. |
| **Mais rápido** | `--fastest` | Apenas dados brutos + a leitura DAQ atribuída + o composto de índice combinado; ignora os cálculos de radiação/refletância/visualização para que o fotograma seja processado rapidamente. Implica `--processing raw --force-daq`. Reprocesse posteriormente o `.daq` guardado em produtos calibrados. |

Opções de exportação (combináveis com qualquer modo; todas partilham a interface gráfica/ponto final SDK):

| Indicador | Efeito |
| --- | --- |
| `--processing LEVEL` | Nível único de exportação, ou `all` (padrão). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportação (por exemplo, `raw,radiance,reflectance`); **substitui `--processing`**. |
| `--aligned` / `--no-aligned` | Alinhar a exportação não bruta de cada membro ao [perfil de alinhamento](#) (co-registado). Os dados não processados permanecem sem transformação, mas transportam a transformação nos metadados. Recorre ao alinhamento não alinhado (com um aviso) se a matriz não tiver perfil. |
| `--index` / `--no-index` | Guardar / ignorar a sobreposição do índice de vegetação, caso esteja configurada. Por predefinição: renderiza-a. |
| `--force-daq` | Guarda a leitura DAQ/DLS atribuída como um ficheiro sidecar `.daq`, mesmo quando nenhum nível selecionado a necessite (por exemplo, uma), para que os fotogramas possam ser reprocessados em refletância/índice offline. |
| `--smart` | Aguardar que o AE se estabilize em todas as câmaras antes de acionar o disparo (ver [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compressão de pixelsTIFF. `deflate` (predefinição) = zlib L1 sem perdas + preditor horizontal, ~4,1 MB por fotograma em resolução total; `none` = não comprimido, ~5× mais rápido na gravação a ~6,3 MB por fotograma — utilizar para obter a taxa sustentada máxima quando o disco o permitir. Ambos são sem perdas e são lidos de forma idêntica na importação. |

> **Gravação única TIFF + o modelo de taxa sustentada.**As capturas são gravadas numa**única**passagem do ficheiro TIFF, transportando píxeis + XMP + IFD0 Marca/Modelo (medido em Mono12 de resolução total: 36 ms comprimido / 6,5 ms não comprimido, contra ~148 ms para o antigo método de «gravar e depois reescrever com o ExifTool»); o único trabalho restante do ExifTool (ajustes no sub-IFD EXIF) é executado num processo de fundo assíncrono, e um fotograma fica completo e pronto para importaçãopronta para importação mesmo que esse processo nunca seja executado. Note-se que a compressão DEFLATE mantém o GIL do Python, pelo que as gravações comprimidas**não**são paralelizadas entre os threads de gravação de cada câmara — uma captura sustentada a resolução total com 8 câmaras à taxa do sensor (~10,4 fps) requer o `--compression none`**e** um disco de classe NVMe (~500 MB/s de gravações sustentadas). O mesmo parâmetro está disponível como `compression` no `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — vídeo/GIF com índice combinado (nível de monitorização)

Grava tudo o que a **visualização combinada em tempo real** estiver a exibir num `.avi` (e, opcionalmente, num `.gif`). Como utiliza o sinal composto em tempo real, o fluxo combinado deve estar aberto (por exemplo, a matriz está a ser pré-visualizada na GUI) para que os fotogramas sejam recebidos. Verifica o progresso a cada 2 s e pára no `--duration`, no `Ctrl+C` ou quando o gravador termina automaticamente.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino (omitir se apenas uma estiver ligada). |
| `-o, --output DIR` | `output` | Diretório de saída (local no backend). |
| `--fps F` | `10` | Taxa de fotogramas de gravação. |
| `--duration S` | até Ctrl+C | Paragem automática após `S` segundos. |
| `--gif` | desativado | Gravar também um GIF animado. |
| `--gif-only` | desativado | Gravar apenas um GIF (sem `.avi`). |

### `array-burst` — sequência de imagens raw-Bayer com alta taxa de fotogramas (nível de análise)

Lê diretamente o buffer do grupo sincronizado do ciclo de captura — **sem cadeia de calibração, sem exiftool, sem necessidade de visualização em tempo real** — pelo que funciona à taxa de captura máxima da câmara. Grava fotogramas raw + um manifesto por fotograma + um `.daq` por leitura DLS em `<output>/bursts/<base>/`. Reprocesse offline (próximo comando)ou passe o `--build` para o fazer imediatamente ao parar.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Diretório de saída (a sequência de dados é guardada em `<DIR>/bursts/<base>/`). |
| `--duration S` | até Ctrl+C | Paragem automática após `S` segundos. |
| `--max-frames N` | ilimitado | Paragem automática após `N` fotogramas brutos. |
| `--build` | desativado | Após a paragem, reprocessar imediatamente a sequência (tal como em `array-build-video`). |
| `--products …` | `combined:index` | Com `--build`: que vídeo(s) criar (ver abaixo). |
| `--fps F` | `10` | Com `--build`: fps do vídeo de saída. |
| `--save-tiffs` | desativado | Com `--build`: também guardar ficheiros TIFF calibrados por fotograma. |
| `--gif` | desativado | Com `--build`: também gravar GIF(s) animados. |

### `array-build-video` — reprocessar offline uma sequência guardada

Associa temporalmente cada fotograma bruto à leitura `.daq` guardada mais próxima e envia-o através da **mesma cadeia de radiação / refletância / índice que o pipeline de importação**, gerando um ou mais vídeos.

O `--products` é uma lista separada por vírgulas de itens `kind:level`, em que `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Um `level` sem valor (sem `kind:`) assume por predefinição o valor `per_cam`. O valor predefinido é `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicador | Valor por predefinição | Descrição |
| --- | --- | --- |
| `--burst-dir DIR` | (obrigatório) | Caminho para a pasta de burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, como acima. |
| `--fps F` | `10` | fps do vídeo de saída. |
| `--save-tiffs` | desativado | Guardar também ficheiros TIFF calibrados por fotograma juntamente com o(s) vídeo(s). |
| `--gif` | desativado | Grava também GIFs animados. |

> **Escolha o gravador certo.** O `array-record` é de *nível de monitorização* — captura o composto em direto tal como é apresentado e requer que o fluxo esteja aberto. `array-burst` → `array-build-video` é de *nível de análise* — guarda dados brutos do sensor à taxa máxima e reconstrói posteriormente vídeos calibrados de radiação/refletância/índice, sem necessidade de visualização em direto.

### Câmaras monocromáticas (M3M) de banda única

A linha **M3M**é a versão monocromática da**M3C**da Bayer: um filtro de interferência de banda estreita por câmara (`M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685`), pelo que o sensor fornece uma**única banda em escala de cinzentos**, sem mosaico Bayer. Não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de brancos para definir — todo o processo de cor «RGB-display» simplesmente não se aplica.

O que isso significa na CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detetam uma câmara monocromática e**ignoram-na com uma mensagem de uma linha**, em vez de aplicarem definições sem sentido. Continuam a funcionar normalmente com uma câmara RGB /Bayer M3C na mesma sessão.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** continuam a funcionar — a radianância e a refletância são *por banda* e estão perfeitamente bem definidas para uma banda. Os fotogramas monocromáticos contêm uma matriz de resposta do sensor **identitária** (sem separação 3×3), pelo que o plano passa pela matemática de calibração inalterado.
- **Uma única câmara monocromática não consegue produzir um índice de vegetação.**NDVI / NDRE /etc. requerem pelo menos duas bandas (por exemplo, Red + NIR). Para obter um índice a partir de hardware monocromático, aponte**várias** câmaras M3M para diferentes comprimentos de onda, alinhe-as numa pilha multibanda e calcule o índice *dessa* pilha:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`; os símbolos têm de corresponder **exatamente** aos nomes dos canais da predefinição (distingue maiúsculas de minúsculas; os de NDVI são em minúsculas: `red`, `nir` — ver `--list-presets`), e o nome do lado da banda deve corresponder a uma banda na pilha alinhada (o modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`).

O discriminador em toda a pilha é o token `M3M` na cadeia de caracteres do modelo (que nunca aparece numa cadeia de caracteres `M3C`), apresentado na GUI/SDKe como `is_mono`.

---

## Configuração e ajuste da placa de rede do anfitrião (matrizes LATTICE)

As câmaras LATTICE transmitem GVSP através do adaptador Ethernet do anfitrião; por isso, em matrizes com várias câmaras, o **driver**do adaptador e o**tamanho do anel de receção** são tão importantes quanto a velocidade da ligação. Configurações incorretas manifestam-se como um erro `FRAMES WILL DROP` / `Reduce ROI to enable` no painel «Configurações do Conjunto» (e como `lattice network-analysis` / `analyze_array_network()` do «SDK»), mesmo quando as próprias câmaras estão a funcionar corretamente.

### Adaptadores USB 10GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

| Item | Valor necessário | Por que é importante |
| --- | --- | --- |
| **Versão do controlador**|**≥ v10.67 (janeiro de 2026)**, INF `rtump64x64sta.inf` | O controlador antigo de**2016**(v10.65, `rtump64x64.inf`) não gere corretamente oe verificações de erros com**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ao desligar, reiniciar ou entrar em modo de suspensão. A transição fica bloqueada (tempo de espera de ~5 min), o utilizador forçao desligamento à força, e os desligamentos incorretos repetidos**corrompem o repositório WMI**(o PowerShell e as ferramentas começam a falhar com o código de erro `Invalid class`) e**bloqueiam a pilha USB** na próxima inicialização (a placa de rede não é ativada; as unidades USB deixam de ser reconhecidas). Atualize a partir de realtek.com (ou do fornecedor do dongle) antes de confiar em reinícios corretos. |
| **Buffers de receção**— palavra-chave `ReceiveBufferLen` |**256**(máximo do controlador) | O anel RX da placa de rede. O valor predefinido do controlador de**32**deixa apenas cerca de 0,26 MB de anel utilizável — demasiado pequeno para uma rajada multicâmara — pelo que o painel da matriz reporta `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e bloqueia as ligações. Com**256**, o anel é grande (**cerca de 13,5 MB medidos no anfitrião de 10 GbE do laboratório**), proporcionando ao pipeline de receção margem real para rajadas GVSP com várias câmaras. (Se uma determinada configuração *estabelece ligação* é decidido por duas verificações — a verificação de admissão **sensível ao esgotamento**e a verificação de**sobre-subscrição agregada** — e não por uma simples comparação entre a rajada e o anel; ver [Modelo de fps e rajadas do array](#array-fps--burst-model).) |
| **URBs de receção**— palavra-chave `PendingReceives` |**64** (máx.) | Blocos de pedidos USB em trânsito; aumentar em conjunto com os buffers de receção para absorção de rajadas. |
| **Jumbo Frame** — palavra-chave `*JumboPacket` | **9014** | Necessário para pacotes GVSP de 9000 bytes (6 vezes menos pacotes por quadro do que com 1500). |

> ⚠️ **Uma atualização do controlador da placa de rede REINICIA estas propriedades avançadas para os valores predefinidos.**Após atualizar ou substituir o controlador do adaptador,**reaplique** `ReceiveBufferLen=256` e `PendingReceives=64`, ou o painel do array voltará a bloquear, mesmo que «nada tenha mudado no hardware». Esta é a #1 causa de um equipamento que antes funcionava se recusar subitamente a estabelecer ligação.

Aplique a partir de um PowerShell com **privilégios elevados** (substitua pelo nome do seu adaptador, por exemplo, `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` abrange adaptadores USB 10GbE.** Agora deteta o tipo de adaptador e ajusta a palavra-chave correta para o anel de receção: `*ReceiveBuffers`→2048 para placas de rede PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 para o controlador Realtek **USB** 10GbE (que não expõe o `*ReceiveBuffers`). Os valores-alvo são limitados ao máximo reportado por cada controlador (`NumericParameterMaxValue`), pelo que nunca é gravado um valor fora do intervalo. Execute-o a partir de um **terminal com privilégios elevados** ; tal como qualquer ajuste baseado no registo, a alteração entra em vigor após o reinício do adaptador ou o reinício do sistema. Os comandos manuais `Set-NetAdapterAdvancedProperty` acima referidos continuam a ser uma boa alternativa — aplicam-se em tempo real (reassociam o adaptador) sem necessidade de reinício.

### Noções básicas de rede (todas as ligações LATTICE)

- **Endereçamento:** endereço local de ligação `169.254.0.0/16` (GigE Vision LLA). O anfitrião assume um endereço estático `169.254.x.x/16`; as câmaras e o DAQ-E autoatribuem-se no mesmo intervalo. Não é necessário DHCP nem gateway.
- **Tamanho do pacote:**prefira jumbo (9000), mas deixe que o auto-sonda determinar o valor — esta remedeia a cada ligação e já ultrapassa o limite de 1500 bytes da câmara para o ICMP através de uma sonda GVSP, pelo que opta pelo jumbo sempre que a ligação o suportar efetivamente. Defina com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` apenas quando tiver a certeza de do que o teste, e prefira por comando em vez de permanente: um pin ignora o teste, pelo que, se o caminho não conseguir realmente transportar 9000,**todas as** capturas expiram com `SC_ERR_TIMEOUT -1011` (ver [Variáveis de Ambiente](#environment-variables)).
- **O anel RX dimensiona-se com `ReceiveBufferLen`:**no valor predefinido `32`, o anel utilizável é de ~0,26 MB (demasiado pequeno para qualquer rajada com várias câmaras); no valor máximo `256`, é grande (~13,5 MB medidos no anfitrião de 10 GbE do laboratório), proporcionando margem real. A possibilidade de uma configuração ser estabelecida é então decidida pela verificação de admissão sensível ao consumo**e** pela verificação de sobre-subscrição agregada abaixo — não por uma comparação bruta entre a sequência de dados e o anel.

### fps do array e modelo de burst

Como interpretar o painel «Array Settings» (e o `lattice analyze-array` / o `analyze_array_network` do «SDK»):

- **A burst é somada por câmara no formato de píxeis real de cada câmara.**As câmaras**M3M**mono transmitem**Mono12 (2 B/px)**; as câmaras**M3C**Bayer transmitem 8 ou 12 bits (a TRI032S emite silenciosamente BayerRG12 mesmo quando é solicitado BayerRG8). Assim, um fotograma de resolução total com 4 câmaras tem**cerca de 12,6 MB se forem todas de 8 bits, mas cerca de 25 MB com três câmaras mono de 12 bits**. A projeção determina o formato de cada câmara a partir do seu modelo (cache de identidade), pelo que a sequência corresponde ao que a ligação transmite efetivamente — e não a uma suposição genérica de BayerRG8.
- **Um adaptador Ethernet USB está limitado a 200 MB/s, independentemente das especificações indicadas.** A tabela de eficiência que converte uma velocidade de ligação num valor sustentado é derivada da PCIe; uma placa de rede USB anuncia a sua velocidade de ligação *Ethernet*, mas está limitada pelo barramento USB e pelo seu controlador. Um dongle USB 10GbE costumava atingir cerca de 1063 MB/s «sustentados» — um valor que nunca foi comprovado — e o ritmo resultante corrompia 6–18 % dos quadros, apesar de continuar a reportar uma taxa de fotogramas por segundo (fps) alvo normal. As placas de rede ligadas por USB estão agora limitadas a **200 MB/s** como valor absoluto (o limite é o barramento, pelo que não varia em função das especificações nominais; um adaptador USB de 1 GbE atinge cerca de 80 MB/s e não é afetado). O `wire_ceiling_source` no registo de capacidades indica isso por escrito, e o `nic_is_usb` assinala-o. É possível anular qualquer uma das opções com o `--wire-ceiling-mbps`.
- **A admissão tem em conta o fluxo de saída, não compara o burst total com o anel.** Um burst simultâneo só tem de caber no *backlog transitório* = `max(0, Σ per-cam arrival − host drain) × emit_window`, não no burst completo. Numa estrutura de anel de câmara lenta com anfitrião rápido / fabric de câmaras lentas (um host **PCIe**de 10 G + 4 câmaras de 1 GbE: chegada ≈ 320 MB/s, drenagem ≈ 1063 MB/s), o host esvazia mais rapidamente do que as câmaras enchem, o backlog ≈ 0, pelo que a simulação de emissão em resolução total**é permitida**, apesar de a rajada de 25 MB exceder o anel de 13,5 MB. Coloque as mesmas quatro câmaras atrás de um adaptador**USB**de 10 GbE e a drenagem é de 200 MB/s, e não 1063 — a chegada ultrapassa-a, e a perda manifesta-se como fotogramas corrompidos em vez de uma taxa de fotogramas mais baixa. Num anfitrião de 1 GbE, o limite mínimo de 31,25 MB/s de DLThr das câmaras faz com que a chegada ultrapasse a taxa de escoamento →**bloqueia** corretamente (para *esta* classe de bloqueio, reduza a ROI ou utilize binning ≥ 2). A admissão é uma das **duas** portas de ligação — a outra é a verificação de sobre-subscrição agregada abaixo.
- **Os fps projetados representam um limite máximo conservador para a recuperação em série.**O ciclo de captura do anfitrião atualmente retira o buffer de cada câmara**em série**(~uma janela de emissão por câmara cada), pelo que o ciclo é limitado por `max(readout+emit, N × emit)`, com a emissão por câmara limitada ao**ligação de acesso**da câmara (1 GbE ≈ 80 MB/s), e não do uplink do anfitrião. Para um conjunto de 4 câmaras a resolução total de 12 bits, isso corresponde a**~2,8 fps**, o que corresponde aos ~2,7–3,0 fps medidos. O valor é deliberadamente**dependente da exposição****, pelo que, em cenas pouco iluminadas, o valor real pode descer ligeiramente abaixo do limite máximo à medida que a exposição se prolonga. A recuperação em série é o verdadeiro limitador de fps; a sua paralelização elevaria o limite máximo para a taxa de emissão única.
- **A sobre-subscrição agregada constitui um bloqueio de ligação rígido.**A alocação de largura de banda por câmara tem um limite mínimo de**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), pelo que, assim que esse limite mínimo for atingido, a procura agregada (`per_cam × N`) pode exceder o**limite máximo da ligação à prova de colisões**(`sustained × sim_emit_factor`). Limites práticos de resolução total em 1 GbE:**6 câmaras a 1500 MTU, 9 com jumbo**. Este limite é uma propriedade exclusiva da ligação e do limite mínimo — é**independente do tamanho do quadro**, pelo que**o agrupamento e uma ROI mais pequena NÃO ajudam** (reduzem os bytes por *quadro*, não os bytes por *segundo* regulados pelo GevSCPD*); as únicas soluções são menos câmaras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. O sintoma seria a perda de pacotes do GVSP, e não uma redução gradual dos fps; por isso, o `analyze-array` zera os valoresde fps alcançáveis e apresenta o `**OVER-SUBSCRIBED**`, e o `array-connect` com uma resolução fixa **recusa a ligação** (caso contrário, o «walk-down» agruparia os quadros, o que também não elimina este tipo de bloqueio). O `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` reclassifica a recusa para um aviso sonoro para trabalho de laboratório — ver [Variáveis de Ambiente](#environment-variables).

### Estado do array — qual o subsistema que está a perder fotogramas

O `GET /api/camera/array/<array_id>/capability` de um array ligado apresenta um bloco
`health` ativo, reavaliado numa janela contínua de **10 segundos**. Divide a perda de quadros
nas duas causas que requerem correções opostas, em vez de reportar uma taxa «incompleta»
que não identifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por porta série) | O quadro **chegou e estava estruturalmente incorreto**— perda de pacotes GVSP. |**Rede**: largura de banda, ritmo, anel de receção da placa de rede, MTU |
| `never_arrived_rate_pct` (por série) | O quadro **nunca chegou**— a câmara não disparou ou nada saiu dela. |**Disparador / sincronização**: cabo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A pior taxa de cada câmara. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmara (ambas as causas em conjunto). | — |
| `stable_for_seconds` | Durante quanto tempo cada câmara permaneceu abaixo de 0,01 %. | — |

Acima de 5 %, o backend regista uma linha `[array-health <id>] WARN` indicando a divisão — na
primeira violação, numa mudança de faixa de gravidade, uma vez por minuto enquanto persistir e uma vez quando
for resolvida. A metade corrompida imprime `[gvsp-corrupt <SN>]` na primeira ocorrência por câmara e
motivo, seguindo-se um resumo a cada 60 s. Todas as avaliações continuam a ser registadas no ficheiro de registo do backend;
os contadores avançam em cada buffer, independentemente do que for impresso.

O mesmo registo indica o número a que toda a alocação está associada:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | Oem vigor, em MB/s. |
| `wire_ceiling_source` | De onde veio esse número, por extenso — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `--wire-ceiling-mbps` (ou o campo **Orçamento de banda** da GUI) o define. |
| `nic_is_usb` | `true` para um adaptador Ethernet USB — ver o limite de 200 MB/acima. |

**Interpretação:** um valor diferente de zero para `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` em 0
significa que o disparo e a sincronização do cabo estão perfeitos e que 100 % da perda ocorre no caminho de rede
— reduza o valor de `--wire-ceiling-mbps` e volte a ligar. O padrão inverso aponta, em vez disso, para o
cabo de sincronização ou para a linha de disparo.

> **O `--target-fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido
> uma vez na ligação, pelo que reduzir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na procura não produziu qualquer melhoria;
> baixar o limite máximo da ligação de 240 para 200 MB/s fez com que o mesmo equipamento passasse de 10,4 %
> de quadros corrompidos para 0,00 %.

> **A redução automática a meio do fluxo não está disponível no firmware TRI032S.** Um conjunto em funcionamento
> não consegue resolver isto sozinho; desligue e volte a ligar para que o seletor de tempo de ligação possa
> replanejar com o novo limite máximo.

### Sintoma → correção

| Sintoma (Definições do Conjunto / ligação / `analyze_array_network`) | Causa | Correção |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` redefiniu para 32 (normalmente após uma atualização do controlador) | Defina `ReceiveBufferLen`→256, `PendingReceives`→64; reabra o painel (reinicie o backend se este tiver armazenado em cache o tamanho antigo do anel) |
| Reinício/desligamento bloqueia; posteriormente, erros WMI `Invalid class`, a placa de rede não ativa, faltam unidades USB | Driver antigo Realtek USB 10GbE de 2016 → BSOD `0x9F` → desligamentos forçados-desligamentos | Atualize o controlador do adaptador para ≥ v10.67 (2026) e, em seguida, reaplique as definições do anel de receção acima |
| A ligação é bem-sucedida, mas devolve uma resolução inferior à nativa | O Smart-prep reduziu automaticamente o quadro para se ajustar à ligação | Atualize a ligação / aceite a redução / `--force-tier slip-emit-and-capture` |
| O conjunto de ecrãs indica um fps alvo válido, mas apresenta apenas uma fração desse valor; `health.gvsp_corrupt_rate_pct` diferente de zero, `never_arrived_rate_pct` 0 | O orçamento de largura de banda inferido do anfitrião exagera a capacidade real (típico num adaptador USB Ethernet, numa via PCIe estreita ou numa malha partilhada) | Volte a ligar com um valor mais baixo de `--wire-ceiling-mbps` e verifique novamente o bloco de integridade. **Não** `--target-fps` — O ritmo do GevSCPD é fixo na ligação |
| Câmaras em falta nos grupos publicados; `health.never_arrived_rate_pct` diferente de zero, `gvsp_corrupt_rate_pct` 0 | Caminho de disparo/sincronização — as câmaras não estão a disparar, não se trata de um problema de rede | Verifique o cabo de sincronização M8 e o `--line`; confirme se todos os membros estão armados (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` excedido em `analyze-array`, ou recusa de ligação com resolução fixa (`array over-subscribes the wire`) | A procura agregada por câmara (limite mínimo de 8 MB/s × N câmaras) excede o limite máximo de largura de banda seguro contra colisões — 6 câmaras em resolução total numa ligação de 1 GbE a 1500 MTU, 9 com quadros jumbo | Menos câmaras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. **ROI/binning NÃO ajudará** (o limite máximo é independente do tamanho do quadro). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` substitui no banco de ensaio (aceita a perda de pacotes) |

---

## `chloros-cli daq`

Comandos do sensor espectral. Duas classes:
- **`pool-*`**— clientes «HTTP» ligeiros que controlam o sensor através do conjunto persistente do backend.**Este é o caminho suportado e o único presente no CLI fornecido.** O backend é responsável pelo transporte, pelo que a GUI, os scripts CLI e SDK partilham todos um único identificador ativo, em vez de disputarem a porta série.
- **Tudo o resto**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — acesso direto ao hardware, documentado abaixo por uma questão de exaustividade. Estes requerem o pacote `daq` Python, que**não está incluído em nenhum artefacto fornecido**: o CLI compilado exclui-o (`scripts/Build-CLI.ps1` define `--nofollow-import-to=daq`, e os transporta o `pyserial` / `bleak` / `zeroconf` com ele), e o pacote PyPI SDK também não o contém. Funcionam apenas a partir de um checkout de código-fonte, pelo que devem ser tratados como um caminho de desenvolvimento interno do MAPIR, em vez de algo a que se deva recorrer.
- **`discover` / `list`** situam-se entre os dois: são comandos diretos de hardware a partir de um checkout do código-fonte, mas numa compilação distribuída recorrem ao `pool-discover` e o backend executa a análise. Assim, a análise funciona em qualquer lugar — o que é importante, pois é a única forma de obter o MAC BLE de um DAQ-M.

> **`chloros-cli daq --help`** (e `-h` / `help`) lista os subcomandos `pool-*` — a ajuda é deliberadamente encaminhada para o cliente do pool, de modo a refletir os comandos que são efetivamente executados. Se invocar um subcomando de acesso direto ao hardware numa compilação fornecida, o programa encerra com um erro explícito indicando o pacote em falta e remetendo-o de volta para o `pool-*`; nada falha silenciosamente. (O `discover` / `list` são a exceção — redirecionam para o `pool-discover` e funcionam normalmente.)
>
> **Tudo o que um cliente precisa está acessível através de `pool-*`** — ligar, transmitir, gravar ficheiros `.daq` calibrados e trocar perfis de condensadores. O DAQ também pode ser controlado a partir de Python com `chloros_sdk.connect_daq_sensor()`, que utiliza o mesmo caminho partilhado.

### Fluxo de trabalho da primeira ligação do sensor DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referência do `pool-*`

| Subcomando | Objetivo |
| --- | --- |
| `daq pool-connect` (detecção inteligente) | Abrir um sensor no conjunto de recursos do backend. |
| `daq pool-connect --port PORT` | DAQ-U numa porta série específica. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC detetado automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE num MAC conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E através de Ethernet num anfitrião conhecido. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, anfitrião detetado automaticamente (mDNS + recurso de fallback ARP; funciona a partir de uma cache ARP vazia em Windows e Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Ajustar janela de integração / estado do AE. |
| `daq pool-connect --no-stream` | Ligar, mas não iniciar ainda a transmissão (retomar com `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de correção de Cap. O valor predefinido no backend é `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Analisa todos os transportes à procura de sensores aos quais se possa ligar, sem estabelecer ligação. **É assim que se encontra o MAC BLE de um DAQ-M.** `daq discover` / `daq list` encaminham-se automaticamente para aqui nas compilações fornecidas. Os sensores já abertos no conjunto não são listados — um DAQ-M ligado deixa de anunciar — por isso, utilize o `pool-list` para esses casos. |
| `daq pool-list` | Mostrar todos os sensores no conjunto de sensores do backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Quadros do espectro N mais recentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Iniciar/parar uma gravação .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de cap em tempo de execução. |

### Subcomandos diretos de hardware (apenas na versão de código-fonte — não incluídos nas compilações fornecidas)

> Listados por uma questão de exaustividade. Estes requerem o pacote `daq` Python, além de `pyserial` / `bleak` / `zeroconf`, nenhum dos quais é fornecido na versão compilada CLI nem no PyPI SDK — funcionam apenas a partir de uma cópia do código-fonte MAPIR. **Se estiver a utilizar uma compilação lançada Chloros, utilize em vez disso os comandos `pool-*` acima**; estes abrangem a ligação, a transmissão, o registo e a seleção de cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Abrir, ligar a, e executar um projeto «Chloros» guardado (uma pasta com `cameras.json` + `sensors.json` + `project.json`). Tudo passa pelo backend, pelo que a GUI e o CLI produzem um estado de hardware idêntico.

### Referência de subcomandos

| Subcomando | Finalidade |
| --- | --- |
| `project open PATH` | Imprimir o manifesto de dispositivos do projeto (câmaras, matrizes, sensores). |
| `project devices PATH [--reconnect]` | Listar ou repetir a deteção. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Ligar todas as câmaras / matrizes / sensores guardados. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Captura única de uma câmara ou matriz especificada. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Série de N fotogramas de uma câmara ou matriz especificada (`-n/--count` valor predefinido 5; `-i/--interval` intervalo entre fotogramas em segundos, valor predefinido 0). As sequências de matrizes eliminam a duplicação de grupos sincronizados repetidos (vigilância de obsolescência), para que uma matriz em ciclo parcial não possa devolver N cópias de um fotograma; apresenta os resultados por iteração. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Transmissão paradisco através de uma tarefa de backend. `--poll-interval` = segundos entre as sondagens `/stats` (padrão 2,0). |
| `project sensor read PATH NAME [--json]` | Quadro de espectro mais recente. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Gravar .daq. |
| `project run PATH RECIPE.yaml` | Executar uma receita de captura YAML/JSON. `--dry-run` valida sem executar. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcular o alinhamento para uma matriz — consulte [a tabela de opções abaixo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Imprimir o perfil de alinhamento atual. |
| `project align clear PATH NAME` | Eliminar o perfil armazenado em cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Deslocar ligeiramente a transformação de um «escravo». |
| `project align export PATH NAME --to FILE` | Guardar o perfil em JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carregar um perfil guardado. |

#### Opções do `project align calibrate`

| Opção | Padrão | Descrição |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alinhamento. **Estas grafias diferem das de `lattice align-calibrate`**, que aceita as formas abreviadas `orb` / `akaze` / `phase`; os dois comandos não são intercambiáveis nesta opção. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar o modelo para se ajustar. |
| `--frames N` | `1` | Instantâneos de fotogramas sincronizados para calcular a média. |
| `--reference SN` | o mestre | Número de série da câmara de referência; todos os outros membros são distorcidos para se alinharem com ela. |
| `--max-features N` | `5000` | Limite de contagem de características ORB. |
| `--ratio-threshold F` | `0.75` | Teste da razão . |
| `--ransac-threshold-px F` | `3.0` | Limiar de pontos internos do RANSAC. |
| `--min-matches N` | `15` | **Limiar de qualidade** — rejeitar a resolução se o número de correspondências de pontos internos for inferior a este valor. |
| `--max-reproj-err-px F` | `4.0` | **Limite de qualidade** — rejeita a resolução acima deste erro de reprojeção RMS. |
| `--checkerboard RxC` | — | Geometria da placa para `--method checkerboard`, por exemplo, `9x6`. |
| `--name PROFILE` | vazio | Nome do perfil incorporado no ficheiro «JSON» guardado. **Não é o nome da matriz** — esse é o `NAME` posicional. |

As duas barreiras de qualidade são a razão pela qual uma calibração pode ser bem-sucedida na resolução e, mesmo assim,
recusar-se a guardar: um perfil que falhe em qualquer uma delas causaria, silenciosamente, um registo incorreto em todas as
capturas posteriores, pelo que é recusado em vez de ser guardado.

### Exemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL de receita

O `project run RECIPE.yaml` aceita um ficheiro YAML ou JSON que descreva uma sequência de ações:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ações suportadas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. A ação `burst` requer o `name` (obrigatório), o `count` (valor predefinido 5), `interval` (segundos, valor predefinido 0), `output`, `format` e `settings` (configurações por câmara idênticas às de `apply`); as sequências de fotos em série utilizam o mesmo watchdog de grupo recém-sincronizado que `project burst`.

Execute:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variáveis de ambiente

| Variável | Efeito |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Substitui o backend URL (padrão: `http://127.0.0.1:5000`) — **respeitada apenas pelas famílias de comandos `lattice`, `project`, e `daq pool-*`.** Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ligam o pino `http://127.0.0.1:<port>` e ignoram esta variável (ov4 contorna a penalização de «Windows» `localhost`→`::1` de ~2 s porpedido), pelo que visam sempre a máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` reclassifica a recusa de ligação por sobre-subscrição da matriz (procura agregada por cam &gt; limite de ligação seguro contra colisões com `pin_resolution`) para um «aviso sonoro e prossiga», aceitando a perda de pacotes GVSP. Apenas para uso em bancos de ensaio — ver [Modelo de fps e rajada da matriz](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Definido pelo próprio «CLI»; indica ao backend para ativar o processamento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignora a sonda de fallback do GVSP (apenas resultados ICMP). **Isto desativa o jumbo, não se limita apenas a silenciar o registo** — a câmara responde a pings DF apenas até 1500 em cada caminho, pelo que esta sonda é a única forma de detetar o jumbo. Poupa cerca de 1 s por câmara por ligação; custa cerca de 1,45× o limite da ligação se a rede *pudesse* ter transportado pacotes jumbo. O SDK avisa quando o configura. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixa o tamanho do pacote GVSP em N bytes; ignora completamente o teste. É preferível usar a configuração por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) em vez de a definir permanentemente: um tamanho fixado deixa de se adaptar à rede à sua frente, e fixar 9000 num caminho que não suporta pacotes jumbo faz com que **todas** as capturas entrem em timeout com o código de erro `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Substituir o diretório de extração do Nuitka onefile. O CLI utiliza automaticamente o `/mnt/ssd/tmp`, se estiver presente. |

---

## Códigos de saída

| Código | Significado |
| --- | --- |
| `0` | Sucesso. |
| `1` | Falha genérica (a maioria dos erros de subcomandos). |
| `2` | Erro de argumento. |
| `130` | Interrompido por Ctrl+C. |

---

## Dicas para a resolução de problemas

- **«É necessário iniciar sessão»** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez neste computador.
- **«backend inacessível»** → Inicie a aplicação de ambiente de trabalho Chloros ou execute diretamente o binário do backend (`chloros-backend`), ou verifique o `CHLOROS_BACKEND_URL` se for remoto.
- **Os comandos do `lattice` falham com a mensagem «Controladores da câmara LATTICE não encontrados»** → O runtime Arena SDK não está instalado; o CLI é fornecido com o `win32api` incluído em Windows, mas o runtime C faz parte do instalador da GUI.
- **A ligação da matriz / Definições da matriz apresenta «FRAMES WILL DROP» ou «Reduza a ROI para ativar»** → O anel de receção da placa de rede do anfitrião é demasiado pequeno (normalmente é reposto para 32 após uma atualização do controlador da placa de rede). Consulte [Configuração e Ajuste da NIC do Anfitrião](#host-nic-setup--tuning-lattice-arrays) — defina o `ReceiveBufferLen=256` e o `PendingReceives=64`.
- **A máquina bloqueia ao reiniciar/desligar, depois o WMI `Invalid class` / a placa de rede não ativa / faltam unidades USB** → O controlador desatualizado do adaptador USB 10 GbE está a causar o erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o controlador do adaptador — consulte [Configuração e ajuste da NIC do anfitrião](#host-nic-setup--tuning-lattice-arrays).
- **Aviso de swap do Jetson** → Adicione um swap baseado em ficheiro; o comando `CLI` apresenta os comandos exatos `fallocate` / `swapon`.
- **Faltam comandos diretos do DAQ** → Esperado: o `chloros-cli` fornecido exclui deliberadamente o pacote `daq`, pelo que apenas o `pool-*` está presente (o PyPI SDK também não o inclui). Utilize o `pool-*`, que controla o mesmo sensor através do backend, ou o `chloros_sdk.connect_daq_sensor()` disponível em Python.

---

## Ver também

- [Python Referência do SDK](sdk-reference.md) — equivalente programático de todos os comandos CLI.
- [Guia de sensores DAQ](../daq/README.md) — ligações específicas dos sensores + calibração.
- Documentação online: `https://mapir.gitbook.io/chloros/cli`
