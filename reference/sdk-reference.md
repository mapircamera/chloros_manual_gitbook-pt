# Chloros Python SDK Referência

**Versão:**

1.2.0**Gerado:**29/07/2026 19:19 ·**Revisado:** 30/08/2026**Pacote:** `chloros-sdk` (PyPI)**Público-alvo:** Otimizado para utilização por LLM; legível por humanos.**Âmbito:** Todas as classes, funções e auxiliares públicas expostas pelo `import chloros_sdk`, com exemplos que podem ser copiados e colados, abrangendo processamento de imagem, controlo de câmara única, matrizes sincronizadas, sensores DAQ e automatização de projetos.

Se precisar apenas dos pontos principais, avance para:
- [Instalação e Início Rápido](#instalação)
- [Smart-Connect para Matrizes LATTICE](#smart-connect-para-câmaras-lattice)
- [Sessões de Sensores DAQ](#sessões-de-sensores-daq)
- [Automatização de Projetos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arquitetura em 60 Segundos

O SDK é uma camada fina de Python sobre o backend Chloros (o mesmo servidor Flask que a GUI de ambiente de trabalho e o CLI utilizam). Para automatização, basta importar o `chloros_sdk` e chamar métodos de alto nível; nos bastidores, cada chamada transforma-se num pedido HTTP para o backend local na porta 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente não `localhost`, que é resolvido primeiro como `::1` em Windows e demora cerca de 2 s por pedido quando o backend é apenas IPv4). O backend é responsável pelo conjunto de hardware — câmaras, sensores DAQ, perfis de alinhamento, buffers de fotogramas — pelo que os scripts SDK podem coexistir com a GUI sem disputarem portas seriais ou largura de banda da placa de rede.

Existem três interfaces que irá utilizar:

1. **`ChlorosLocal` + funções livres** (`process_folder`, `process_lattice_capture`) — Pipeline de processamento de imagens. Execute uma pasta inteira através da calibração / remoção de pixel de cor / exportação de índice a partir de uma única chamada Python.
2. **Maneuças Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abra uma sessão de backend persistente para hardware em tempo real. O mesmo fluxo «smart-prep» que a GUI: sonda de rede, seleção automática de camada, PTP, inicialização de AE, configuração do gatilho GPIO.
3. **`ChlorosProject` / `open_project`** — Carregar um projeto guardado (pasta com `cameras.json` + `sensors.json` + `project.json`), ligar tudo de uma só vez e efetuar capturas de disco com identificadores nomeados.

As superfícies 1 e 2 **inicie automaticamente um backend local**, caso ainda não haja nenhum a escutar (o mesmo binário incluído que a GUI/CLI executam) — para que um script simples funcione a partir de um shell novo, sem que seja necessário iniciar primeiro um backend. Passe `auto_start_backend=False` para desativar esta opção (por exemplo, ao apontar para um backend remoto, que nunca é iniciado). Consulte [Início Automático do Backend](#backend-auto-start). A superfície 3 comporta-se de forma diferente: o `open_project()` não aceita o parâmetro `auto_start_backend`, e o `connect_all()` nunca inicia um backend — ele testa o `http://127.0.0.1:5000` uma vez e, se nada responder, recorre silenciosamente ao controlo direto (sem backend) do dispositivo `lattice_sdk`. Apenas o `proj.process()` e o `stream(..., overlays=True)` criam de forma diferida um `ChlorosLocal()` (que executa o autoinicialização automática).

Todos os três estão sujeitos a autenticação: execute o `chloros-cli login` uma vez na máquina ou inicie sessão através da GUI do ambiente de trabalho. As chamadas a SDK sem uma sessão válida geram o erro `ChlorosAuthenticationError`.

Requisitos:
- Python 3.7+ (conforme declarado pelo pacote; desenvolvido/testado na versão 3.10)
- Chloros Desktop instalado localmente (o binário do backend vem incluído no instalador)
- Conta ativa no Chloros+. O nível mínimo para SDK / CLI é o nível **Copper**ou superior (Copper / Bronze / Silver / Gold); o nível gratuito**Iron**não tem acesso a SDK / CLI. Isto é aplicado**do lado do servidor**: todos os pedidos com a flag SDK / CLI devem incluir tanto uma sessão ativa como um plano pago; caso contrário, o backend devolve `403` com `error_code: PLAN_UPGRADE_REQUIRED` (apresentado como `ChlorosLicenseError` pelo `ChlorosLocal` e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um utilizador que tenha saído da sessão recebe `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) em vez disso — os dois são distintos porque a reexecução de `chloros-cli login` corrige o primeiro, mas não consegue corrigir o segundo.
- A utilização offline é suportada durante o período de carência do plano: o nível é lido a partir da cache de validação do servidor (5 min) ou da cache de licenças assinadas e vinculadas ao equipamento (30 dias para planos mensais, até ao termo da subscrição para planos anuais). Quando esse período de carência expira, o plano passa a ser gratuito e o acesso a SDK / CLI é interrompido até que o computador consiga estabelecer ligação ao servidor. O `chloros-cli status` (`GET /api/license-status`) permanece acessível no nível gratuito, pelo que a razão é evidente — é a única rota SDK / CLI isenta da restrição do plano.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS ou mais recente**, ou Jetson (JetPack 6). O Ubuntu 20.04**não** é suportado: as dependências do `.deb` derivam do que o backend liga, incluindo o `libc6 (>= 2.34)`, e o Focal inclui a glibc 2.31.

---

## Instalação

O Python SDK é uma camada fina Python sobre o backend Chloros. Para tudo o que vá além de alguns fluxos de trabalho exclusivamente de aquisição de dados (DAQ), é necessário ter o **pacote de ambiente de trabalho Chloros instalado localmente** (instalador Windows ou Linux `.deb`) — é isso que fornece o binário do backend, o ambiente de execução Arena SDK para câmaras LATTICE e os pacotes de calibração.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Passo 1 — Instalar o pacote da plataforma «Chloros»

#### Windows (.exe)

1. Descarregue o `Chloros-Setup-x.y.z.exe` a partir da página de downloads.
2. Execute o instalador e siga as instruções do assistente. O caminho de instalação predefinido é `C:\Program Files\MAPIR\Chloros\`.
3. Inicie o Chloros pelo menos uma vez e inicie sessão com a sua conta Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Passo 2 — Instalar o Python SDK

**O instalador Chloros inclui um wheel SDK correspondente.** Cada instalador Windows e cada ficheiro .deb Linux coloca no disco um `chloros_sdk-X.Y.Z-py3-none-any.whl` que corresponde exatamente à versão da GUI / CLI / backend. Não é necessário acompanhar o PyPI para se manter em sincronia.

#### Windows

O instalador executa automaticamente o `pip install` com o ficheiro wheel incluído, utilizando o seu Python do sistema (é preferível o lançador `py.exe`; em caso de falha, recorre-se ao `python -m pip`). Não é necessária qualquer ação — o `import chloros_sdk` funciona no seu ambiente Python após uma instalação bem-sucedida. Se não existir nenhum Python na máquina, o instalador ignora silenciosamente este passo e a GUI + CLI continuam a funcionar.

#### Linux (.deb)

O ficheiro .deb coloca o wheel em `/usr/lib/chloros/sdk/`. O `postinst` apresenta o comando exato — as distribuições PEP 668 recusam gravações globais do pip por predefinição, pelo que não fazemos a instalação automática:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implementações do Jetson em ambiente isolado (air-gapped), este processo é totalmente offline — o wheel já se encontra no disco.

#### PyPI Público

Para anfitriões que utilizam apenas o pip (sem pacote de ambiente de trabalho Chloros instalado; fluxos de trabalho com backend remoto ou apenas DAQ):

```bash
pip install chloros-sdk
```

O PyPI é atualizado nas compilações do instalador da versão de lançamento, pelo que o wheel publicado corresponde à versão estável mais recente. As compilações de desenvolvimento (por exemplo, `1.1.4.dev1`) só são fornecidas através do «wheel» do instalador incluído.

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **É necessária uma subscrição em Chloros+.** Todas as chamadas a SDK requerem um login ativo em Chloros+. Execute o `chloros-cli login user@example.com 'YourPassword'` uma vez por máquina; as credenciais são armazenadas em cache no `~/.chloros/`.

### Preciso do pacote para computador?

O pacote pip, por si só, **não** é suficiente para a maioria dos fluxos de trabalho. Eis o que cada superfície SDK necessita:

| Superfície SDK | Necessita do Pacote Desktop? | Porquê |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sim** | Inicia automaticamente o binário do backend em `/usr/lib/chloros/chloros-backend` (Linux) ou `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sim**(local)**/ Não**(remoto) | Clientes Pure HTTP através do backend. Backend local → é necessário o pacote para o ambiente de trabalho. Backend remoto → `backend_url=`**através de um túnel** (ver Modo de Backend Remoto — os backends fornecidos ligam-se apenas ao loopback). |
| `ChlorosProject` / `open_project` | **Sim** | Executa projetos guardados através do backend. |
| Classes LATTICE diretas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sim** | Requer o ambiente de execução nativo Arena SDK incluído no pacote para computador. Caso contrário, o `CAMERA_AVAILABLE` corresponde ao `False` na importação. |
| Classes Direct DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Não** | Python puro através de pyserial/bleak/zeroconf. Um ambiente apenas com o pip pode controlar os DAQs de ponta a ponta. |

### Modo de backend remoto (anfitrião apenas com o pip, através de túnel)

> **O backend fornecido não está acessível através da LAN.** As compilações
> de produção ligam-se apenas ao loopback (ambas as famílias de loopback) e recusam categoricamente o
> único modo não-loopback (`CHLOROS_CLOUD_MODE`), pelo que
> o `backend_url="http://<lan-ip>:5000"` **não funciona com um
> Chloros instalado** — esse padrão só funcionava com um backend source/dev.
> Para controlar um backend noutro computador, reencaminhe a sua porta de loopback
> manualmente e aponte o SDK para o túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Os anfitriões sem interface gráfica / CI / robótica podem manter uma máquina com a instalação completa do ambiente de trabalho como o «servidor Chloros» e `pip install chloros-sdk` em todos os outros locais — mas o transporte entre eles é o túnel configurado pelo utilizador acima, e não um URL direto na LAN.

> **Limitação conhecida — o `ChlorosLocal` não suporta apenas o pip.** Atualmente, o `ChlorosLocal(backend_url=BACKEND)` resolve um binário de backend local no seu construtor *antes* de sondar o URL e gera o erro `ChlorosBackendError` («Backend Chloros não encontrado…») quando não está instalado nenhum pacote de ambiente de trabalho — mesmo com um backend remoto acessível. Apenas a superfície de ligação inteligente acima (`connect_camera` / `connect_array` / `connect_daq_sensor`, além de `analyze_array_network` e os auxiliares `list_*` / `discover_*`) funciona a partir de um anfitrião que utilize apenas o pip.

### Fluxo de trabalho apenas para DAQ (anfitrião que utilize apenas o pip)

Se precisar apenas de sensores DAQ e não utilizar câmaras LATTICE nem processamento de imagem, o pacote pip é autônomo:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Sem backend, sem .deb, sem necessidade de iniciar sessão em Chloros+ para trabalho de DAQ diretamente no hardware.

---

## Início rápido

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Índice de nível superior do API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Processamento de Imagens — `ChlorosLocal`

A classe principal do pipeline. Inicia o backend na primeira utilização, cria/configura projetos, monitoriza o progresso e devolve resumos após a execução.

### Construtor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Métodos

| Método | Descrição |
| --- | --- |
| `create_project(project_name, camera=None)` | Cria um novo projeto (opcionalmente com um modelo de câmara como o `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importa imagens RAW/TIF/JPG/DNG **e gravações do sensor de luz `.daq`**. Devolve `count` (imagens) e `scan_count` (gravações). Apenas emite um aviso se a pasta não contiver nenhum dos dois. |
| `export_light_sensor(daq=True, csv=True)` | Grava os ficheiros calibrados `.daq` + `.csv` para cada registo do sensor de luz do projeto, na pasta `<project>/Light Sensor/`. Consulte [Gravações do sensor de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Definir parâmetros de processamento. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executar o pipeline. Retorna `{"status": "complete", "async": False}`, além de uma chave `summary` quando o backend a fornecer — consulte [Resumo pós-execução e sugestões](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Inspecionar o estado do backend. |
| `logout()` | Limpar as credenciais em cache. |
| `shutdown_backend()` | Encerrar o backend (se tiver sido iniciado com `SDK`). |
| `discover_cameras()` | Detectar câmaras LATTICE **através do backend desta instância** (`/api/camera/discover`). Retorna uma lista de dicionários (`serial`, `model`, `ip`, …) — com a mesma estrutura que a GUI/ CLI apresenta. Lista vazia se não for encontrada nenhuma ou se o backend estiver inacessível. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura um único fotograma**através do backend**(iniciado automaticamente por este identificador) para que receba a mesma preparação que a GUI/ CLI (padrão de 12 bits, reutilização do conjunto, metadados de calibração incorporados). Resolva o alvo com `serial=` ou `device_index=`; passe `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Devolve o dicionário de metadados antigo (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Fornece fotogramas de pré-visualização com sobreposição composta a partir de um cliente MJPEG «thin» agrupado — cliente MJPEG leve através da rota `/api/camera/<serial>/stream-annotated` do backend (zebra / grelha / mira / histograma / picos / ponto desenhado do lado do servidor). `decode=True` gera matrizes BGR; `False` gera bytes brutos de «JPEG». Também acessível por projeto como `ChlorosProject.stream(overlays=True)`. |

Utilizar como gestor de contexto para uma limpeza garantida:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Gravações do sensor de luz — calibrado `.daq` + `.csv`

Um DAQ-U / DAQ-M / DAQ-E pode ser gravado **sem** o seu pacote de calibração. É isso
que os gravadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): gravam as contagens brutas do sensor e marcam o
ficheiro de forma a que Chloros recupere a calibração de fábrica desse sensor **por número de série** — cache local
primeiro, depois na nuvem MAPIR — e aplica-a na importação.

O Chloros reescreve o resultado como dois produtos por gravação, sob
`<project>/Light Sensor/`:

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | O arquivo reprocessável — com o mesmo esquema de uma gravação em tempo real, declarando agora o pacote que o produziu. A reimportação **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grelha de comprimentos de onda do próprio sensor, uma linha por leitura, mais colunas fotométricas (potência total, lux fotópico/escotópico, PPFD e a sua divisão em azul/verde/vermelho, comprimento de onda de pico). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Apenas sensores sem pacote (DAQ-A).** Contagens espectrais brutas do sensor — *não* irradiância. Ver abaixo. |

O `process()` realiza esta exportação como uma das suas etapas. **Não** requer imagens:
um sensor de luz utilizado de forma independente constitui um fluxo de trabalho de primeira classe, e um projeto deste tipo tem zero
imagens por definição.

**As gravações do DAQ-A são exportadas como contagens brutas.** A família DAQ-A é anterior ao sistema de pacotes por número de série
e não tem nenhum pacote para recuperar — em vez disso, é calibrada no terreno em relação a um
alvo de refletância, razão pela qual nunca precisou de um. Essas gravações são exportadas
com a raiz `_raw` em vez de `_calibrated`: um nome de ficheiro diferente em vez de um indicador
dentro do ficheiro, porque a informação tem de se manter intacta ao ser enviada por e-mail apenas com o nome. O
cabeçalho `.csv` indica `raw spectral sensor counts (NOT irradiance)` e avisa que os
valores são comparáveis **dentro** do ficheiro — exatamente para isso que serve a calibração baseada em alvo
para esse fim — e não entre sensores. As colunas fotométricas dependentes da potência (potência total,
lux fotópico/escutópico, PPFD) apresentam o valor **NULL** em vez de serem integradas a partir das contagens.

Um DAQ-U / DAQ-M / DAQ-E cujo pacote simplesmente não tenha sido possível obter continua a ser **ignorada**,
em vez de ser gravada em formato bruto: nesse caso, o pacote existe e «reconectar e reprocessar» é um conselho válido.

As gravações antigas **v1.01 / v1.02** (um DAQ-A-SD grava estas) não contêm uma época por leitura,
apenas a hora de gravação do ficheiro. O comparador de imagem↔luz descendente continua a rejeitá-las — comparar um
fotograma com uma hora de gravação seria um erro impercetível — mas o exportador lê-as, e o
CSV imprime `clock=daq_created_on` para que o produto indique em que relógio se encontra.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Uma gravação cujo pacote de calibração não possa ser obtido (offline ou um sensor sem
calibração registada) é reportada sob o código `skipped` **com o motivo**. Nunca é
gravado como um ficheiro «calibrado» contendo contagens brutas — ligue-se à Internet e
execute novamente, e a exportação será concluída.

### Chamadas de retorno de progresso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumo pós-execução e sugestões

Após a conclusão, o `process()` recupera o `GET /api/processing-summary` e anexa o corpo como `result["summary"]`. A recuperação é feita com o melhor esforço possível e nunca bloqueia um retorno bem-sucedido — se o resumo não estiver disponível, o `process()` recorre ao formato simples do `{"status": "complete", "async": False}`. Cada entrada no `summary["hints"]` — frases completas com a correção sugerida, p.por exemplo, a razão pela qual uma execução produziu zero resultados — é também reemitida como um `UserWarning` do tipo «Python», pelo que as execuções com zero resultados são autodiagnósticas, mesmo que nunca se inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` é a partelegível pela máquina:

| Chave | O que conta |
| --- | --- |
| `models` | Grupos de câmaras na execução. |
| `images_in_groups` | Imagens de origem nesses grupos. |
| `targets_found` | Alvos de refletância detetados. |
| `images_calibrated` | Imagens que a execução calibrou. |
| `exported_files` | **Ficheiros de produtos de imagem gerados pela execução.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registos do sensor de luz, contados separadamente de forma intencional — provêm de uma fase diferente e existem para execuções sem qualquer imagem, pelo que incluí-los faria com que uma execução apenas de aquisição de dados (DAQ) parecesse ter exportado imagens. |

Juntamente com estes: `summary["output_dirs"]` (todos os diretórios em que foi gravado),
`summary["light_sensor_export"]`, `summary["stopped"]` (verdadeiro quando o utilizador interrompeu a
execução, para que as contagens parciais não sejam interpretadas como uma execução concluída com produção insuficiente), e
`summary["groups"]` (a repartição por grupo).

O `exported_files` é registado pelo pipeline **à medida que escreve**, não sendo extraído posteriormente dos
objetos de imagem do projeto. As estratégias paralelas e de GPU criam os seus próprios objetos de imagem (em subprocessos de trabalho para os percursos de GPU), pelo que a verificação antiga reportava
`0 file(s) written` para cada uma dessas execuções e, em seguida, emitia a indicação de exportações nulas — em execuções
em que tudo tinha funcionado. Se criar um script com base neste número, uma execução paralela em bom estado agora
apresenta uma contagem diferente de zero.

Os saltos do sensor de luz indicam o motivo que o leitor efetivamente identificou para cada ficheiro — um
esquema ilegível, um pacote em falta, um erro de gravação — **deduplicados**, de modo que vinte ficheiros
ignorados por uma única causa são interpretados como uma única causa, em vez de vinte repetições da mesma.

> **`process()` não é gerado quando uma execução não produz imagens.** Este é o único ponto em que o SDK e
> o CLI diferem deliberadamente: o `chloros-cli process` trata «foram solicitados produtos, nenhum foi
> gravado» como uma falha e termina com um valor diferente de zero, enquanto o SDK termina normalmente e reporta a
> condição através do `summary` / dicas. Se o seu pipeline parar numa execução vazia, verifique-o
> você mesmo — inspecione o `summary` (ou conte os ficheiros na pasta do projeto) em vez de confiar na
> ausência de uma exceção. As causas habituais são uma pasta de entrada que não foi reconhecida como uma
> captura e produtos ignorados por serem inaplicáveis às câmaras presentes (por exemplo, radiação de câmaras «RGB» apenas
>).

### Funções de conveniência

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valores suportados

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Saída radiométrica (pipeline multiespectral LATTICE)

O nível de exportação multiespectral LATTICE (M3C/M3M) do pipeline `process` — `reflectance` (padrão), `radiance`, `sensor-response` ou `all` (cada modo aplicável por imagem) — corresponde à configuração de processamento **«Saída radiométrica»»** do projeto. O `configure()` tem uma palavra-chave dedicada para isso:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A saída de emergência avançada — escrever a chave `"Radiometric output"` do projeto através de `custom_settings` — continua a funcionar, mas lembre-se de que substitui todo o bloco de definições (ver o aviso abaixo):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (o valor predefinido) divide a radiação da câmara pelo **timestampcorrespondente ao DAQ descendente**, resolvido automaticamente a partir de um `.daq` (DAQ-U/M/E) gravado**ou de um `.csv` nativo do DAQ-M**encontrado juntamente com as imagens; qualquer pacote de calibração por câmara ou DAQ que falte localmente é**obtido automaticamente da AWS** na primeira utilização. O CLI apresenta isto como opções de ativação/desativação por tipo de produto no `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **substitui** todo o bloco de definições calculadas (por definição, ignora as outras palavras-chave e a validação do `configure()`). Ao utilizá-lo, inclua todas as chaves `Project Settings` que lhe interessem, tal como no exemplo acima.

---

## Smart-Connect para câmaras LATTICE

Sessões de backend persistentes para hardware em tempo real. Utiliza os mesmos pontos de extremidade que a GUI, pelo que o comportamento é idêntico em SDK / CLI / GUI.

### Câmara única — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Assinatura do `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### Métodos do `CameraSession`

| Método | Descrição |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lê os nós GenICam; devolve `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Escreve os nós pelo nome amigável (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Captura um **único** fotograma. Devolve uma lista com um elemento contendo dicionários de metadados do fotograma. (A captura em rajada/múltiplos fotogramas foi removida — chame `capture()` num ciclo se precisar de uma série.) |
| `disconnect()` | Liberta da reserva. Não tem efeito se estivermos ligados a uma sessão já aberta. |

Controlos de exportação do `capture()` (mesmo modelo que a matriz + GUI):

- `processing` / `levels` — `processing="all"` guarda todos os tipos de exportação aplicáveis; `levels=["raw","radiance"]` guarda apenas aqueles (substitui o `processing`). Omita ambos para utilizar a predefinição do backend.
- `force_daq=True` — guarda a leitura atribuída do DAQ/DLS como um ficheiro «sidecar» `.daq`, mesmo numa captura apenas em formato bruto, para que o fotograma possa ser reprocessado para refletância/índice posteriormente. Não tem efeito se nenhum DAQ estiver ligado.

### Matriz Sincronizada — `ArraySession` (Smart-Prep)

O `connect_array` é **o ponto de entrada recomendado** para configurações com várias câmaras. Executa internamente todo o fluxo de preparação inteligente da GUI:

1. **Análise de rede** (`/api/camera/array/recommend`) — determina o maior tamanho de fotograma que cabe no nível sim-emit sem perda de fotogramas.
2. ****Seleção automática de nível** — `sim-capture-sim-emit` se a ligação suportar; caso contrário, `sim-capture-ftd-stagger` ou `slip-emit-and-capture`.
3. **Redução automática**— reduz silenciosamente o tamanho do quadro / aumenta o agrupamento quando a ligação não consegue sustentar a resolução solicitada.**Esta rede de segurança não cobre a sobre-subscrição agregada**: o excesso de câmaras para a ligação não pode ser resolvido através da redução dos fotogramas — consulte [Sobresubscrição](#over-subscription-the-per-cam-floor).
4. **PTP ativado** por predefinição — os carimbos de data/hora entre câmaras são comparáveis com uma precisão de microssegundos.
5. **Seleção automática do formato de píxeis por câmara** — câmaras «RGB» → `BayerRG8`, multiespectral → `BayerRG12`.
6. **Inicialização do AE** — captura instantânea do estado atual do AE de cada câmara para que a ligação não reinicie a exposição a meio da operação.
7. **Configuração do disparador GPIO** — a `connect_array` arma todas as câmaras (`TriggerMode=On`, `TriggerSource=Line2`) para que o pulso do mestre acione as escravas através do cabo M8. Este é um passo exclusivo para matrizes: uma única câmara aberta com `LatticeCamera` executafunciona de forma autónoma.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Assinatura `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valores `force_tier`:
- `"sim-capture-sim-emit"` — verdadeiramente simultâneo (todas as câmaras disparam na mesma flanco do relógio).
- `"sim-capture-ftd-stagger"` — escalonamento temporal flexível(as câmaras emitem em momentos ligeiramente desfasados para que os pacotes sejam serializados na linha).
- `"slip-emit-and-capture"` — captura sequencial por câmara (sem sincronização temporal; única opção quando nenhum tamanho de quadro se adequa à sincronização simultânea).

`wire_ceiling_mbps` substitui o **orçamento sustentado de largura de banda do anfitrião** em MB/s — o único
valor do qual depende toda a alocação da matriz. Deixe-o como `None` para utilizar o valor detetado automaticamente
. Reduz-o quando a matriz reportar quadros corrompidos pelo GVSP: o valor automático é derivado
da taxa de ligação anunciada pela NIC, que sobrestima adaptadores USB, pistas PCIe de largura reduzida e
estruturas partilhadas ocupadas — e essa sobreestimativa manifesta-se como quadros corrompidos, em vez de como uma
ligação visivelmente lenta. O valor é guardado no bloco de captura da matriz do projeto, pelo que uma
reabertura ou um `connect_array` posterior o restaura como qualquer outra configuração da matriz.
Consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames).

#### Sobresubscrição (o limite mínimo por câmara)

O ritmo de emissão simulada atribui a cada câmara uma quota do orçamento de banda seguro contra colisões, com um limite mínimo de **8 MB/s por câmara**(`per_cam_floor_bps`). Assim que o `N × floor` excede o limite máximo de segurança contra colisões, o array**sobresubscreve a ligação**— o modo de falha é a perda de pacotes GVSP, não uma taxa de fotogramas mais baixa — e não existe qualquer solução relacionada com o tamanho dos fotogramas:**o agrupamento e a área de interesse (ROI) reduzem os bytes por fotograma, e não os bytes regulados por segundo**que a verificação agregada compara. Limites práticos de resolução total num anfitrião de 1 GbE:**6 câmaras a 1500 MTU, 9 com quadros jumbo** (o `max_cams_collision_safe` na resposta da análise indica o limite máximo para a sua ligação). Soluções: menos câmaras, quadros jumbo de ponta aou uma placa de rede mais rápida.

- As respostas `analyze_array_network()` e `/api/camera/array/connect` incluem `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` é verdadeiro, a projeção **zera os campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) em vez de indicar uma taxa enganadora de «lenta, mas a funcionar».
- `POST /api/camera/array/connect` aceita um parâmetro de corpo `pin_resolution` (**apenas HTTP — não um kwarg SDK**; `connect_array` não o expõe). O «pinning» remove a rede de segurança de redução de binning- para baixo, pelo que uma ligação com excesso de subscrições e com o `pin_resolution` definido é**rejeitada categoricamente** com um erro que indica todas as soluções possíveis. Sem o «pinning», a ligação prossegue com o «walk-down», mas avisa que a redução não consegue limpar o agregado.
- Saída de emergência em ambiente de teste: defina `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` no ambiente do backend para rebaixar a recusa para um aviso sonoro — liga-se na mesma e aceita a perda de pacotes.

#### Integridade da matriz — qual o subsistema que está a perder quadros

O `GET /api/camera/array/<array_id>/capability` contém um bloco `health` ativo numa
matriz ligada, reavaliado numa janela contínua de **10 segundos**. Divide a perda de quadros
nas duas causas que requerem correções opostas, em vez de uma taxa «incompleta» que
não identifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por número de série) | O quadro **chegou e estava estruturalmente incorreto**— perda de pacotes GVSP. |**Rede**: largura de banda disponível, ritmo de transmissão, anel de receção da placa de rede, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nunca chegou**— a câmara não disparou ou nada saiu dela. |**Acionamento / sincronização**: cabo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Pior taxa de cada câmara. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmara (ambas as causas juntas). | — |
| `stable_for_seconds` | Durante quanto tempo cada câmara permaneceu abaixo de 0,01 %. | — |

Juntamente com o `health`, o mesmo registo indica o número em que toda a alocação fica pendente:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de largura de banda sustentado do anfitrião em vigor, MB/s. |
| `wire_ceiling_source` | De onde provém esse valor, por palavras — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `wire_ceiling_mbps=` o definiu. |
| `nic_is_usb` | `true` para um adaptador USB Ethernet. |

Não existe um wrapper SDK para este ponto de extremidade — leia-o diretamente:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Leitura:** um valor diferente de zero em `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` igual a 0 significa
que o disparo e a sincronização do cabo estão perfeitos e que 100 % da perda ocorre no caminho da rede — reduza
o valor de `wire_ceiling_mbps` e volte a ligar. O padrão inverso aponta, em vez disso, para o cabo de sincronização ou para a
linha de disparo.

> **O `target_fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido uma vez na
> ligação, pelo que diminuir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na procura não produziu qualquer melhoria, enquanto
> a redução do limite máximo da ligação de 240 para 200 MB/s fez com que o mesmo equipamento passasse de 10,4 % de quadros corrompidos para
> 0,00 %.

> **A redução automática a meio do fluxo não está disponível no firmware TRI032S.** Um array em funcionamento não pode
> resolver isto por si próprio; desligue e volte a ligar para que o selecionador de tempo de ligação volte a planear com base
> no novo limite máximo.

Um **adaptador Ethernet USB está limitado a 200 MB/s** pelo sondador, independentemente da sua
placa de identificação: a tabela de eficiência que transforma uma taxa de ligação num valor sustentado é
derivada da PCIe, e uma placa de rede USB anuncia a sua velocidade de ligação Ethernet, embora esteja limitada pelo
barro USB e pelo seu controlador. O limite é absoluto, não uma fração — um adaptador USB de 1 GbE
atinge cerca de 80 MB/s e não é afetado.

#### Métodos `ArraySession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Ativo `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Um grupo de captura sincronizado. Devolve um `CaptureResult` (lista de dicionários de fotogramas + `.skipped`). Controlos de exportação abaixo. |
| `capture(..., smart=True)` | **Captura inteligente** — aguarda que o AE se estabilize em todas as câmaras e, em seguida, aciona a captura. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Captura mais rápida: apenas dados brutos+ a leitura DAQ atribuída (+ o índice combinado livre). Espelha o botão «Captura mais rápida» da GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Contínua / Intervalo num único ciclo delimitado. Devolve `list[CaptureResult]`.**Requer `count` e/ou `duration_s`** para que termine (o «SDK» não tem Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Iniciar a gravação da visualização ao vivo do índice combinado para vídeo/GIF → `RecorderHandle`. Um gravador composto por matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Iniciar uma sequência de imagens Bayer em bruto com alta taxa de fotogramas → `RecorderHandle`. Reprocessar offline com `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocessar offline uma sequência RAW guardada em vídeo calibrado(s). Bloqueia até estar concluído (`wait=True`) e devolve `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Verifica um trabalho de compilação offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Liberta toda a matriz. |

`capture()`Controlos de exportação  (mesmo ponto de extremidade que a GUI/ CLI utiliza):

- `processing` / `levels` — O `processing="all"` (ou `levels=["raw","radiance",…]`) guarda todos os tipos de exportação aplicáveis por câmara; um único valor `processing` guarda apenas esse nível.
- `aligned=True` — alinha todas as exportações não brutas de cada membro ao [perfil de alinhamento](#alinhamento da matriz) (co-registado); os dados em bruto permanecem sem distorção, mas transportam a transformação nos metadados. Recorre à opção «não alinhado» (com um aviso exibido no `alignment` do resultado) se a matriz não tiver perfil.
- `render_index=False` — ignora a sobreposição do índice de vegetação por câmara; por predefinição, apresenta-a onde estiver configurada.
- `force_daq=True` — guarda a leitura DAQ/DLS atribuída como um `.daq` sidecar, mesmo que nenhum nível selecionado o necessite.

**CompressãoTIFF (botão «HTTP»):**O `ArraySession.capture()` não envia a chave `compression`, pelo que se aplica a predefinição do backend — o `POST /api/camera/array/capture` lê um parâmetro de corpo do `compression`, o `"deflate"`, por predefinição (zlib L1 sem perdas + preditor horizontal, ~4,1 MB por fotograma em resolução total). O `"none"` grava sem compressão (~6,3 MB/fotograma) com uma**gravação ~5× mais rápida** — ambos são sem perdas e são lidos de forma idêntica na importação. O SDK não expõe nenhum kwarg para isso; a saída de emergência é o `chloros-cli lattice array-capture --compression none` ou o HTTP em formato bruto. O DEFLATE também detém o GIL do Python, pelo que as gravações comprimidas não se paralelizam entre os threads de gravação por câmara — a captura sustentada a resolução total com 8 câmaras à taxa do sensor requer o `compression: "none"`. Detalhes: [Referência do CLI → array-capture](cli-reference.md).**Substituições de exportação por membro (apenas HTTP):**o mesmo ponto final também aceita `exclude_serials` (lista — exclui membros do conjunto guardado; a matriz continua a ser acionada como um único grupo sincronizado e os membros excluídos são devolvidos em `excluded`), `serial_levels` (substituições ao nível de câmara do `{serial: [level tokens]}`) e `serial_index` (substituições de sobreposição de índice de câmara do `{serial: bool}`sobreposições de índice por câmara). Trata-se de parâmetros de corpo com paridade com a GUI e**ainda não são argumentos de parâmetro dSDK**; os membros ausentes dos mapas recorrem aos valores globais da matriz `levels` / `render_index`.

##### Inspecção de câmaras ignoradas — `CaptureResult.skipped`

O `ArraySession.capture()` devolve um `CaptureResult`, que é uma subclasse do `list`: itere-o, indexe-o, aplique-lhe o `len()` — todos os padrões existentes continuam a funcionar. O novo código pode inspecionar o atributo `.skipped` para verificar quais as câmaras que foram excluídas e porquê. O caso mais comum é o das câmarRGBes numa matriz de filtros mistos quando se solicita `processing="radiance"` ou `"reflectance"` — a radiação por pixel Bayer não faz sentido para um sensor de banda larga, pelo que o backend ignora essas câmaras em vez de produzir resultados sem sentido.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Os tokens de motivo seguem o padrão `<level>-not-applicable-to-rgb-cam` (uma entrada por nível ignorado, cada uma com `level`). As omissões específicas de refletância são `reflectance-skipped-no-fresh-dls` (não há leitura recente da radiação descendente disponível), `reflectance-skipped-bound-daq-unavailable (…)` (não foi possível aceder ao DAQ associado) e `dls-uncalibrated-band-<nm>` — a banda situa-se maioritariamente fora da gama radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm), pelo que a divisão absoluta da refletância baseada no DAQ é recusada e o fotograma reverte-se claramente para a resposta do sensor. Entre os SKUs disponíveis, apenas o F988 a ativa; o percurso suportado por essa câmara é o fluxo de trabalho com painel de refletância.

Níveis do `processing`:

| Nível | Saída |
| --- | --- |
| `"raw"` | Bayer monocanal (câmaras monocromáticas: a banda única) diretamente do sensor. |
| `"debayered"` *(predefinição do SDK)* | BGR de 3 canais via demosaico bilinear (câmaras monocromáticas: escala de cinzentos de 1 canal). |
| `"radiance"` | float32 W/m²/sr/nm através da cadeia radiométrica completa. Apenas multiespectral — as câmaras «RGB» são ignoradas. |
| `"reflectance"` | uint16 0..32768 (compatível com Pix4D); requer um emparelhamento DAQ em tempo real para referência absoluta. Apenas multiespectral. |
| `"display"` | Cadeia completa correspondente à pré-visualização da GUI (CCM + WB + gama de acordo com o perfil da câmara). |
| `"all"` | **Um ficheiro por nível aplicável** para cada câmara (correspondente à opção da GUI «Capturar Tudo» / CLI). O `CaptureResult` devolvido contém então um dicionário de fotogramas por `(cam, level)`, com o nível em cada dicionário; os níveis inaplicáveis aparecem em `.skipped`. A leitura do DAQ utilizada para qualquer quadro de refletância é guardada como um ficheiro «sidecar» `.daq`. |

> **Nota — o valor predefinido difere do CLI.** O `ArraySession.capture()` tem como valor predefinido o `processing="debayered"`; o comando `chloros-cli lattice array-capture` tem como valor predefinido o `processing="all"`. Passe o `processing="all"` explicitamente a partir do SDK para espelhar o CLI /GUI com gravação em vários níveis.

### Modos de captura e gravadores

A superfície da matriz espelha o painel de captura da GUI: modos de obturação Único / Contínuo / Intervalo / Mais rápido, além de dois gravadores (vídeo composto em direto e rajada em formato bruto → reprocessamento offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**é o ciclo contínuo/por intervalos do SDK. Como não existe um `Ctrl+C` para o interromper a partir de um script,**deve** passar o `count` e/ou o `duration_s` (ele pára quando qualquer um deles for atingido). O `interval_s` é medido a partir do início de cada passagem (em consonância com a GUI). Os restantes kwargs passam diretamente para o `capture()`.
- **`record`** é de *nível de monitorização*: captura o composto de índice combinado em tempo real tal como é apresentado, pelo que o fluxo combinado tem de estar aberto para que os fotogramas sejam recebidos. Um gravador de composto por matriz (lança uma exceção se já houver um em execução).
- **`burst` → `build_video`** é de *nível de análise*: O `burst` grava fotogramas em bruto + um manifesto por fotograma + um `.daq` por cada leitura DLS distinta sob o `<output>/bursts/<base>/` à taxa máxima do ciclo de captura (sem cadeia, sem exiftool, sem visualização em tempo real). O `build_video` sincroniza temporalmente cada fotograma com o `.daq` mais próximo e reexecuta a cadeia de radianância/refletância/índice do pipeline de importação. O `products` é uma lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (padrão: o índice combinado). `burst().stop()` também inicia automaticamente uma construção do índice combinado com o melhor esforço possível, devolvida como `build_job` no resultado da paragem.

#### `RecorderHandle`

Devolvido por `ArraySession.record()` e `ArraySession.burst()`. Utilize-o como gestor de contexto para parar automaticamente ao sair do âmbito, ou execute-o manualmente.

| Membro | Descrição |
| --- | --- |
| `job_id` | ID da tarefa de backend (str). |
| `kind` | `"composite"` (de `record`) ou `"raw"` (de `burst`). |
| `start_stats` | O dicionário devolvido pela chamada `start`. |
| `result` | `None` durante a execução; o dicionário final do resultado da paragem, uma vez parado. |
| `stats(timeout=10.0)` | Estatísticas do trabalho em tempo real (fotogramas gravados, fps efetivos, tempo decorrido). |
| `stop(timeout=60.0)` | Interrompe o gravador; devolve e armazena em cache o resultado final. Idempotente (uma segunda chamada devolve o resultado armazenado em cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Ligar-se a uma matriz já ligada — `attach_array`

Se a matriz já estiver ativa (a GUI abriu-a ou uma sessão anterior do SDK chamou `connect_array`), utilize `attach_array` para obter um identificador do mesmo, em vez de voltar a ligar-se. O `connect_array` apresenta sempre o erro «A câmara <sn>já está na matriz<id>» nessa situação, porque a chamada POST do `/array/connect` para um membro do conjunto não é idempotente; o `attach_array` lê o `/api/camera/array/list` e faz a correspondência com base no array_id ou nos números de série.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Padrão: SDK Os scripts que partilham o ambiente com a GUI do ambiente de trabalho devem tentar primeiro o `attach_array` e recorrer ao `connect_array` se ainda não houver nenhum array no pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Importante — o encerramento do gestor de contexto DESLIGA a ligação.**O `ArraySession.disconnect()` envia sempre um POST para o `/array/disconnect`; não existe um guard «attached-notcomo acontece com o `CameraSession` / `DAQSensorSession`. Se estiver a partilhar espaço com a GUI e não quiser desmontar a matriz ao sair do âmbito,**não utilize o bloco `with`** — guarde o identificador numa variável normal e ignore o `disconnect()` explícito:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Auxiliar de Análise de Rede

Útil antes de abrir o array — permite prever se as configurações propostas serão adequadas:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` é um dos seguintes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (caso contrário, `error`). `auto_capped_fps` significa que a resolução solicitada se adequa ao anel RX apenas a uma taxa de disparo limitada — mantenha a resolução e passe de `target_fps=result["recommended"]["recommended_target_fps"]` para `connect_array` (ver [Exemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Como interpretar a projeção** (mesmo modelo que o painel «Configurações da Matriz» da GUI):

- **A sequência (`frame_bytes_total`) é somada por câmara no formato de píxeis real de cada câmara.**As câmaras Mono**M3M**transmitem Mono12 (2 B/px), independentemente do valor de `pixel_format` que se passar; assim, um fotograma de resolução total com 4 câmaras tem**~25 MB** com três câmaras mono, e não os ~12,6 MB que a hipótese de 8 bits em todas as câmaras daria. O backend determina o formato de cada câmara a partir do seu modelo.
- **A admissão (`burst_fits_nic_ring`) tem em conta a drenagem**, e não a comparação entre o burst completo e o anel: o sim-emit adapta-se quando o anfitrião esvazia o anel RX mais rapidamente do que as câmaras o enchem. Um anfitrião de 10 G + câmaras de 1 GbE**admite** resolução total mesmo quando o burst excede o anel; um anfitrião de 1 GbE bloqueia (`needs_force_slip` / `auto_shrunk`).
- **O `achievable_fps_max` é um limite máximo conservador para a recuperação em série** — o `max(readout+emit, N×emit)` com a emissão por câmara limitada à ligação de câmara de 1 GbE, independentemente da exposição. Por exemplo, ~2,8 fps para um conjunto de 4 câmaras a resolução total de 12 bits (corresponde aos ~2,7–3,0 medidos no tempo de execução). Modelo completo: [CLI Referência → Modelo de fps e rajadas do conjunto](cli-reference.md#array-fps--burst-model).
- **A sobre-subscrição (`oversubscribed: true`) significa que o limite mínimo de N × por câmara excede o limite máximo seguro contra colisões** — os campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) apresentam o valor 0, e a redução automática/agrupamento não conseguem resolver o problema (reduzem os bytes por fotograma, não o ritmo de bytes por segundo). As soluções consistem em utilizar menos câmaras, quadros jumbo ou uma placa de rede mais rápida; o `max_cams_collision_safe` indica o limite máximo (6 câmaras em resolução total numa ligação de 1 GbE com MTU de 1500, 9 com quadros jumbo). A resposta também inclui os códigos `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps` (8 MB/s). Consulte [Sobresubscrição](#over-subscription-the-per-cam-floor).

### Descoberta e listagem

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Os conjuntos LATTICE executam o AE contínuo em segundo plano assim que são ligados, mas uma cena recém-focada demora um pouco a convergir. O **Smart-Capture** é a solução prática integrada: verifica a exposição de cada câmara, aguarda até que o conjunto esteja estável numa janela e, em seguida, aciona a captura. É equivalente à interface gráfica: o botão «smart» chama o mesmo ponto final do backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Ao utilizar o `ChlorosProject` (próxima secção), tem à sua disposição mais opções de configuração:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

A política de AE inteligente é conservadora por predefinição. Torne o `exposure_tolerance_pct` mais restritivo para trabalhos radiométricos exigentes; torne-o mais flexível para cenas em rápida mudança, nas quais apenas se pretende um resultado «suficientemente próximo».

---

## Sessões do sensor DAQ

Conjunto de backends persistente para sensores espectrais (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Espelha a superfície da câmara: deteção inteligente, reutilização do conjunto, ligação idempotente.

### Detecção Inteligente (Configuração Zero)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioridade: Ethernet → BLE → USB. Passe qualquer indicação explícita para fixar o transporte.

### Transporte Fixado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Métodos `DAQSensorSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Resumo da entrada no conjunto (estado de transmissão/gravação, intervalo de comprimento de onda, SHA de calibração, tempo de integração, frame_avg, estado AE). |
| `latest(n=1, timeout=10.0)` | Devolve até N quadros de espectro mais recentes. |
| `stream_start()` / `stream_stop()` | Retomar / pausar a transmissão (o identificador permanece aberto). |
| `record_start(output_dir=None, device_name=None)` | Inicia a gravação de um ficheiro .daq. Devolve o caminho do ficheiro. Não é permitido para DAQ-U/M sem um pacote de calibração AWS (o DAQ-E está isento). |
| `record_stop()` | Parar a gravação. Devolve `{path, rows}`. |
| `disconnect()` | Libertação da piscina. Não tem efeito paranão pertencentes ao utilizador. |

> **Os perfis de correção de cap (`cap_id`) não são um controlo do «SDK».** `connect_daq_sensor()` / `DAQSensorSession` não expõem nenhum parâmetro `cap_id` nem método `set_cap`. Selecione um perfil de correção de limite da frota através do «CLI» (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ou das rotas «HTTP» do backend (`/api/daq`) (`/api/daq/connect` e `/api/daq/<id>/cap-id` aceitam `cap_id`).

### Descoberta — encontrar um endereço para estabelecer ligação

O `discover_daq_sensors()` procura em USB / BLE / ETH por sensores que *possa* abrir. É a contraparte DAQ do `discover_lattice_cameras()` e a única forma de obter o **MAC BLE de um DAQ-M** — um DAQ-E tem um nome de anfitrião e um DAQ-U uma porta COM, mas o MAC não está impresso no dispositivo nem é listado pelo sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrição |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome do anfitrião — passar para `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Rótulo legível por humanos. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, ou `None` para uma porta que a análise não consegue identificar (os adaptadores seriais USB são indistinguíveis sem uma sonda, pelo que os desconhecidos são exibidos em vez de ocultados). |
| `extra` | Detalhes por transporte (nome anunciado do BLE, fabricante USB, IP/FW do DAQ-E/…). Os valores vazios são omitidos. |

| Parâmetro | Padrão | Descrição |
| --- | --- | --- |
| `transports` | os três | Sequência (ou cadeia CSV) que limita a verificação. Vale a pena especificar quando se sabe o que se pretende — o BLE é a parte mais lenta. |
| `scan_timeout` | 5 | Janela de varredura por transporte em segundos; o backend limita o valor a 1–20. |
| `timeout` | 60,0 | Limite máximo de «HTTP» para toda a chamada (tal como noutras partes do «SDK»). |
| `auto_start_backend` | `True` | Inicia um backend local se nenhum estiver em execução. Nunca é iniciado para um `backend_url` remoto. |

> **Os sensores já abertos na piscina não aparecem.** Um periférico BLE ligado deixa de anunciar e uma porta COM aberta não pode ser detetada, pelo que a descoberta lista o que está *disponível para ligação*. É normal obter um resultado vazio logo após ter ligado algo — utilize o `list_daq_sensors()` para o que já possui. Os transportes cuja verificação não podeser executada (sem o bleak / zeroconf instalado) são ignorados em vez de gerarem erros, pelo que um computador sem Bluetooth continua a obter as respostas relativas a USB e ETH.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Co-tenancy com GUI / CLI

Se a GUI já tiver um sensor aberto, chamar o `connect_daq_sensor(port="COM3")` a partir de Python devolve um identificador marcado como `already_connected=True`. O `disconnect()` da sessão passa então a ser uma operação nula, pelo que o seu script SDK nãoretirar o sensor da GUI ao sair do Scope.

### Classes de hardware direto (sem backend)

O `daq_sdk` é reexportado pelo `chloros_sdk`, pelo que também pode controlar sensores de ponta a ponta noprocesso sem o backend:

> **Disponibilidade:**O `daq_sdk` é fornecido com a instalação do desktop Chloros,**não** com o pacote PyPI — o `pip install chloros-sdk` fornece-lhe o `lattice_sdk`, mas deixa de fora o `chloros_sdk.DAQ_AVAILABLE == False`. Verifique essa opção antes de utilizar estas classes; num host que utilize apenas o pip, acione o sensor através [`connect_daq_sensor()`](#daq-sensor-sessions), que não necessita de bibliotecas de transporte locais.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Dê preferência ao caminho de ligação inteligente (`connect_daq_sensor`) quando pretender partilhar a propriedade com a GUI; utilize as classes diretas para scripts sem interface gráfica que detêm a propriedade exclusiva do sensor.

---

## Automatização de Projetos — `ChlorosProject`

Um projeto «Chloros» guardado é uma pasta que contém `cameras.json` + `sensors.json` + `project.json`. O `open_project` carrega o manifesto, e o `connect_all` coloca todos os dispositivos guardados em modo ativo com as suas definições guardadas — o mesmo estado de hardware que a GUI produziria.

### Exemplo mínimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Ou como gestor de contexto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Métodos do `ChlorosProject`

| Método | Descrição |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Deteta e liga todos os dispositivos guardados. Devolve um relatório de ligação por classe. Utiliza um backend em execução quando este estiver a escutar no `127.0.0.1:5000`; caso contrário, recorre silenciosamente ao controlo direto (sem backend) — nunca cria um backend. |
| `disconnect_all()` | Desliga tudo. |
| `capture_all(output_dir=".")` | Um quadro de cada câmara + matriz + espectro de cada sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Gerador que produz fotogramas BGR `numpy` a partir de uma câmara (ou matriz) nomeada. O `overlays=False` é um `lattice_sdk`(as matrizes produzem dicionários `{serial: frame}`). O `overlays=True` é encaminhado através do `ChlorosLocal.camera_stream()` → o, com o bloco `ui.overlay` guardado pela câmara a ser transmitido como parâmetros de consulta. Requer o modo backend e uma **câmara autónoma**: uma câmara em modo direto gera o erro `RuntimeError` (o backend não consegue aceder a uma câmara pertencente a este processo) e um conjunto gera o erro `NotImplementedError` (sobreposições compostas por câmara — transmitir um membro pelo nome). Umequivalente de um único comando: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Executar o alinhamento em todas as matrizes atualmente ligadas. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executar o pipeline de calibração/indexação nas (envolve o `ChlorosLocal.process`; estes quatro são os **únicos** argumentos-chave aceites — `indices=`, etc., geram o erro `TypeError`; define índices através de `ChlorosLocal.configure()`). Constrói de forma preguiçosa um `ChlorosLocal()`, que inicia automaticamente um backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` indexado por nome E número de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexado por nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexado por nome E slot_id.
- `proj.config` — `project.json["config"]` dicionário.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Níveis de processamento.** `capture()`, `grab()` e `frame_stream()` recebem todos o mesmo token `processing`
, e a cadeia é cumulativa — cada nível executa tudo o que está acima dele:

| Nível | Saída | Notas |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo do sensor | Sem demosaico. As sobreposições não estão disponíveis neste nível. |
| `debayered` | BGR de 3 canais (**padrão**) | Desmosaico bilinear. O único nível que funciona sem o modo «backend». |
| `radiance` | float32, W/m²/sr/nm | Cadeia radiométrica completa: desmosaico + desmistura 3×3 (multiespectral) + DSNU + campo plano+ escala NIST, com a exposição × ganho eliminados para que os valores sejam absolutos. |
| `reflectance` | uint16, 32768 = 1,0 | Radianância dividida pela irradiância descendente (ρ = π·L/E). Requer uma leitura DLS/DAQ — ver a nota abaixo. |
| `display` | 8 bits, semelhante a sRGB | Renderização equivalente à GUI: CCM + equilíbrio de brancos + gama através do perfil de cor ativo da câmara. |

Qualquer valor que não seja `debayered` requer o modo backend; uma câmara em modo direto gera
`NotImplementedError`. O `reflectance` requer uma leitura de luz descendente utilizável — o ponto final do fotograma puxa
o DAQ agrupado para a ranhura DLS da câmara automaticamente, mas sem um DAQ associado, a cadeia recusa a
saída de refletância e regista honestamente a desclassificação nos metadados devolvidos, em vez de devolver silenciosamente
um produto de qualidade inferior.

> **Escala DN de refletância — não a codifique de forma rígida.** A refletância LATTICE utiliza `32768` = ρ 1,0 e regista
> XMP `Chloros:PixelScale=32768`; a refletância «Survey3» utiliza `65535` = ρ 1,0 e não contém
> etiquetas `Chloros:*`. Leia a etiqueta e divida por ela. Está definida no domínio uint16, pelo que se mantém
> `32768` para todos os formatos que redimensionam (TIFF de 16 bits, 8-bits PNG /JPG, 32-bits percent) — normalize
> primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8-bits, ×65535 a partir de float). A única exceção:
> uma captura de origem de 8-bits gravada como 8-bits TIFF é *cortada*, não é redimensionada, pelo que nenhuma escala a descreve
> — Chloros omite totalmente o `PixelScale` e a tupla MicaSense nesse caso. Trate a ausência de uma
> etiqueta num ficheiro de refletância LATTICE como «sem escala válida», e não como um valor por defeito.

> **EXIF transferido para a exportação.** O `process()` copia o bloco GPS da captura de origem
> **e o seu ExifIFD** para cada produto, pelo que as exportações incluem `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, bem como a
> georreferenciação. O `FocalLength` é a distância de amostragem do solo calculada pelo Pix4D a partir de — sem ela,
> a reconstrução recai numa escala totalmente errada (num caso medido, um local de 411 m
> transformou-se num de 47,8 km). A cópia não é deliberadamente `-all:all`: as etiquetas estruturais do IFD0 quebram
> a saída do LATTICE, e os ficheiros `ExifImageWidth`/`Height` são excluídos porque descrevem a captura da fonte
> em vez da imagem raster exportada.

Sub-sinalizadores da fase de captura (aplicáveis aos níveis radiométricos — `radiance`, `reflectance`, `display`):

| Sinalizador | Padrão | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + separação de 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de WB. Compatível com DLS quando um DAQ está associado à câmara. |
| `apply_index` | `False` | Avaliação do índice de vegetação. |
| `index_expression` | `None` | Fórmula de substituição. Se não estiver vazio →-ativa o índice. |
| `annotated` | `False` | Sobrepor decorações da GUI (zebra/grelha/picos). Não disponível para `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **O tipo de retorno é `CapturePathMap`, não `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` é `Dict[str, Union[str, List[str]]]`: um de nível único
> `processing` atribui a cada número de série um caminho, enquanto um de nível múltiplo (`"all"`, ou uma
> lista explícita `levels`) fornece-lhe a **lista ordenada** de todos os produtos guardados para essa
> câmara. Uma composição combinada em direto, caso esteja a ser transmitida, aparece sob a chave extra
> chave `"combined"`, em vez de sob um número de série. O código que assume o `str` falha na
> forma de lista sem que nenhum verificador de tipos se oponha — a anotação indicava o `Dict[str, str]`
> durante algum tempo após o lançamento do formato de lista, razão pela qual o alias existe. Normalize
> quando pretender o formato simples:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Alinhamento de matrizes

O `ArrayHandle` expõe toda a superfície de alinhamento. Por predefinição, os perfis são válidos apenas para a sessão — chame explicitamente o `export_alignment()` para os tornar persistentes.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alinhamento no momento da ligação

O `connect_all(align=...)` pode alinhar automaticamente todas as matrizes no momento da ligação:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Recorre ao `project.json["config"]["auto_align_on_connect"]` quando não especificado.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Hardware direto (sem backend)

Quando pretender zerar a dependência do backend (CI, robôs headless, incorporado), importe diretamente o `lattice_sdk` e o `daq_sdk` — ambos são reexportados pelo `chloros_sdk`. Atenção ao `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: o `lattice_sdk` está incluído no pacote PyPI (mas requer a presença do runtime Arena SDK), enquanto o `daq_sdk` é fornecido apenas com a instalação para computador.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Predefinições e o gatilho

Três das quatro predefinições são de **funcionamento livre**: a câmara expõe continuamente e o
`capture()` devolve o quadro seguinte. O `triggered` é a exceção — prepara a
câmara para uma mudança de estado de hardware na Linha 2, pelo que não capta nada até que essa ocorra.

| Predefinição | Acionador | Utilizar quando |
| --- | --- | --- |
| `default` | funcionamento contínuo | utilização geral |
| `high_speed` | funcionamento contínuo | 8 bits, limite de 60 fps, exposição curta |
| `high_quality` | funcionamento livre | 12 bits, sem limite de fps — a escolha habitual para fotografias |
| `triggered` | **armada, Linha 2** | a câmara está ligada a um cabo de sincronização M8 e é acionada por outro dispositivo |

Se escolher `triggered` (ou definir o `trigger_mode="On"` manualmente) sem que nada
comande a Linha 2, todos os `capture()` entrarão em timeout — corretamente, uma vez que pediu
à câmara para aguardar. O SDK explica isto quando acontece; consulte
[SC_ERR_TIMEOUT durante a captura](#direct-hardware-backend-free).

> **Nota — As mensagens «GVSP probe» / `SC_ERR_TIMEOUT -1011` na ligação não são erros.**&gt; Na ligação, o SDK tenta negociar**quadros jumbo** (pacotes GVSP de 9000 bytes) para obter um maior débito. Numa ligação direta ponto a ponto da placa de rede (por exemplo, um endereço `169.254.x.x` local da ligação), a rede normalmente não consegue transportar quadros jumbo, pelo que esta sondagem atinge o tempo limite e regista mensagens como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Este é o **plano de contingência previsto**: o SDK reverte automaticamente para pacotes padrão de 1500 bytes e a câmara continua a ligar-se normalmente (as linhas `[chunk-enable …]` que se seguem fazem parte da sequência normal de ligação). A captura continua a funcionar.
>
> Pode ignorar esta sonda, mas **não se trata apenas de um silenciador de registos — ela desativa os quadros jumbo.** A câmara responde «Não-Fragment apenas até 1500 bytes, independentemente da qualidade da sua rede, pelo que o teste de ping, por si só, nunca consegue detetar quadros jumbo; esta sonda é a única que o consegue fazer. Desative-a e a câmara utilizará pacotes padrão de 1500 bytes para sempre, em qualquer rede:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Só vale a pena numa rede que *saiba* que não suporta jumbo, onde poupa cerca de um segundo no tempo de ligação por câmara. Como se trata de uma alteração real e não apenas cosmética, o «SDK» agora indica isso quando a utiliza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Não altere esta configuração, a menos que tenha um motivo para o fazer.** Se permanecer ativada, cada ligação volta a medir a rede de que dispõe efetivamente: ligue-se a um switch compatível com jumbo e a próxima ligação deteta o jumbo automaticamente, sem necessidade de configuração nem reinício.
>
> Se *quiser* a taxa de transferência do jumbo, ative o jumbo de ponta a ponta (MTU da placa de rede 9000 + um switch que os transmita), ou fixe-o com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando souber que a ligação o suporta — embora seja preferível utilizar o comando `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando em vez de o definir permanentemente, uma vez que um tamanho fixo ignora a sondagem e deixa de se adaptar à rede à sua frente. **Todos os** dispositivos no caminho têm de transmitir pacotes jumbo — incluindo qualquer divisor ou injetor PoE, que é a razão habitual pela qual uma configuração que, de outra forma, seria compatível com pacotes jumbo, não consegue transmiti-los.

> **O `SC_ERR_TIMEOUT -1011` durante o `capture()` / `grab*()` é um problema diferente — esse é um erro real.**&gt; A nota acima refere-se apenas ao `-1011` registado pela**sonda de tempo de ligação**. O mesmo erro gerado a partir de uma**captura** significa que a câmara se ligou corretamente, mas não está a enviar quaisquer imagens:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> O indício revelador é uma câmara cujo canal de *controlo* está em bom estado — a deteção funciona, as definições e as gravações `[chunk-enable …]` são todas bem-sucedidas — enquanto *todos* os fotogramas atingem o tempo limite.
>
> **A causa habitual é a câmara estar configurada para um disparador de hardware.** Com o `trigger_mode="On"` e o `trigger_source="Line2"`, a câmara não emite absolutamente nada até que chegue uma flutuação elétrica ao cabo de sincronização M8. Se não houver nenhum cabo a alimentar essa linha, todas as capturas ficam à espera indefinidamente. A câmara não está avariada e a rede está a funcionar bem — está a fazer exatamente o que lhe foi pedido.
>
> O `CameraSettings()` e as predefinições `default` / `high_speed` / `high_quality` funcionam, e uma captura que atinge o tempo limite enquanto está armada explica a situação em vez de apresentar apenas um `-1011`. O `PRESETS["triggered"]` arma a Linha 2, conforme previsto.
>
> Para forçar qualquer câmara a funcionar em modo livre:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se continuar a expirar com o `trigger_mode="Off"`, a câmara não está realmente a enviar dados — envie-nos o registo e o `ip link show`.

#### Perfis de cor (Pré-visualização em direto do RGB) — `set_color_profile`

O `LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleciona o perfil de cor de visualização para a **pré-visualização em direto** nas câmaras RGB (as câmaras multiespectrais ignoram esta configuração):

| Perfil | Significado |
| --- | --- |
| `raw` | Ignora completamente a cadeia radiométrica. |
| `linear` | DSNU + flat + WB, sem CCM, sem gama. |
| `natural` | Linear + CCM medido + gama sRGB, apenas com o acabamento básico (suavização de croma + dessaturação de realces) — a predefinição realista. |
| `enhanced` | `natural` mais o acabamento completo com paridade de hub (remoção de franjas, vibração, contraste local CLAHE). Aspecto mais rico com aproximadamente **o dobro do custo de acabamento por fotograma**, o que resulta numa taxa de fotogramas LIVE mais baixa. |
| `custom_temp` | `natural`, mas com o equilíbrio de brancos fixado no valor de Kelvin do `custom_cct_k` (DLS ignorado; limitado ao intervalo de 2000–10000 K no backend-). |

O perfil é um **botão de velocidade/aspecto**apenas em pré-visualização ao vivo**: as capturas guardadas obtêm sempre o acabamento rico e completo, independentemente do perfil selecionado; por isso, escolher o `natural` para recuperar tempo de fotograma não reduz a qualidade do que é gravado no disco. Um perfil desconhecido aumenta o `ValueError`; quando um backend chloros estiver acessível, a alteração também é enviada por POST para ele, para que o próximo fotograma de pré-visualização a reflita (os utilizadores do direct-SDK sem um backend continuam a receber a alteração das definições).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Câmaras monocromáticas (M3M) e `Calibration`

Uma câmara monocromática **M3M** (`M3M-<lens>-F<wavelength>`) é de banda única: um plano em escala de cinzentos, sem mosaico de Bayer, sem matriz de interferência espectral 3×3. O `Calibration` reconhece-a e expõe um sinalizador `is_mono`. A refletância continua a aplicar-se como um mapa radiométrico por banda (a separação de cores é a matriz identidade), mas os cálculos multibanda numa única câmara geram resultados válidos em vez de devolvem valores sem sentido:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Para construir um índice de vegetação a partir de hardware monocromático, combine várias câmaras M3M em diferentes comprimentos de onda numa pilha multibanda alinhada (ver [Alinhamento de Matriz](#array-alignment)) e calcule o índice nessa pilha, em vez de o fazer numa única câmara.

Modo direto do DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Chaves aceites para `apply_sensor_settings`**— exatamente `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; obsoleta em favor de `cap_id`), `filter_model` (DAQ-M) e `cap_id` (todos os tipos de DAQ; `None`/`""`/`"none"` = sensor simples, sem correção de limite). As chaves desconhecidas são**ignoradas silenciosamente** — por exemplo, `{"integration_time": 64}` não faz nada (deve ser `integration_time_ms`). Retorna `{"applied": [...], "errors": {...}}` e nunca gera uma exceção.

`chloros_sdk` re-exporta apenas a superfície principal utilizada acima. O API público completo `daq_sdk` (22 nomes) adiciona o seguinte — importe-os diretamente de `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Exceções

Intercepte a classe base para tratar «qualquer coisa que tenha corrido mal Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` são exportados ao nível superior juntamente com os restantes; também podem ser importadas a partir de `chloros_sdk.exceptions`, conforme ilustrado.

Hierarquia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemplos completos

### 1. Processar uma pasta com uma barra de progresso personalizada

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matriz LATTICE em tempo real → Refletância + Referência DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanha de captura orientada por projeto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Fluxo de fotogramas de várias câmaras → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captura direta por hardware sem interface gráfica (sem backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verificação de capacidades antes de ligar uma matriz de 4 câmaras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente à receita de captura (Python puro)

A DSL de receitas do CLI tem um equivalente direto no Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Arranque automático do backend

Os pontos de entrada do smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, e `discover_lattice_cameras` — são clientes «thin» HTTP que pressupõem que um backend está a escutar no `127.0.0.1:5000` (o endereço URL predefinido da superfície do Smart-Connect). Quando a GUI ou o CLI já estão a ser executados, um deles está. A partir de um script simples, pode não haver nenhum — por isso, estas funções **iniciam automaticamente o binário do backend incluído** (sem janela, da mesma forma que o `ChlorosLocal` faz) antes da sua primeira chamada e, em seguida, aguardam até `backend_startup_timeout` para que este seja iniciado.

Regras:

- **Apenas um URL local é alguma vez iniciado.** Um `backend_url` que aponte para `localhost` / `127.0.0.1` / `[::1]` é elegível; qualquer outro anfitrião é considerado como sendo a máquina de outra pessoa e nunca é criado.
- **O backend é deixado em execução para reutilização** (tal como no CLI) — não há um encerramento implícito quando o seu script termina. Ao executar novamente o script, o backend ativo é reutilizado.
- **Desative esta funcionalidade com `auto_start_backend=False`** em qualquer uma dessas chamadas (por exemplo, quando apontar para um backend remoto ou se gerir o ciclo de vida do backend por conta própria).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se o binário incluído não puder ser localizado ou iniciado, a chamada subsequente HTTP gera um `ChlorosConnectError` **sensível à plataforma** e passível de ação, em vez de um simples registo de ligação recusada — no Windows, isto remete-o para a aplicação de ambiente de trabalho ou um comando `chloros-cli`; no Linux (sem GUI), aponta para um comando `chloros-cli` ou para o `.deb`.

---

## Ambiente e cabeçalhos

O SDK marca todas as chamadas de backend HTTP com `X-Chloros-Client: sdk`. O backend aplica as regras de licenciamento SDK / CLI (é necessário efetuar o início de sessão **e** ter um plano pago Chloros+), em vez do caminho da camada gratuita da GUI. Isto é definido automaticamente no momento da importação — não é necessário fazer nada.

`http://localhost` e `http://127.0.0.1` são detetados como o backend local. As chamadas para outros anfitriões (por exemplo, o seu próprio serviço de análise) permanecem inalteradas.

Substitua o backend URL passando `backend_url=` (ou `api_url=` em `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Um `backend_url` que não seja de loopback só alcança um backend de origem/backend de dispositivo — os backends fornecidos limitam-se ao loopback; consulte o Modo de Backend Remoto para o padrão de túnel.)

---

## Versões e Compatibilidade

- A versão SDK é exposta como `chloros_sdk.__version__`.
- O SDK vincula o comportamento à versão do backend incluído. Combinar um SDK mais antigo com um backend mais recente geralmente funciona (pontos finais compatíveis com), mas a combinação de um backend mais recente SDK com um backend mais antigo pode provocar erros `404` em novos pontos de extremidade — atualize a aplicação para computador para que corresponda.
- A interface «smart-connect» (`connect_camera` / `connect_array` / `connect_daq_sensor`) e o endpoint de análise de rede devolvem esquemas JSON estáveis; os novos campos são adicionais.

---

## Dicas para resolução de problemas

- **`ChlorosAuthenticationError: Login required`** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina ou inicie sessão através da aplicação de ambiente de trabalho Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → As chamadas Smart-Connect iniciam automaticamente um backend local, pelo que isto só aparece quando o binário incluído não consegue ser encontrado/iniciado (por exemplo, um hostsem pacote de ambiente de trabalho). A mensagem adapta-se à plataforma: em Windows, abra a aplicação para ambiente de trabalho ou execute qualquer comando `chloros-cli`; em Linux, execute um comando `chloros-cli` (não existe GUI) ou instale o `.deb`. Para um backend remoto, passe `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** na importação → Falha ao carregar o `lattice_sdk` (normalmente, as DLLs de tempo de execução do Arena SDK não estão instaladas). A superfície que não é da câmara continua a funcionar.
- **A ligação da matriz devolve uma resolução inferior à nativa**→ O backendfunção «smart-prep» reduz automaticamente o tamanho do fotograma para se adaptar ao canal de transmissão. Utilize o `analyze_array_network()` para perceber o motivo e, em seguida, atualize a ligação, aceite a redução ou passe o `force_tier="slip-emit-and-capture"` para captura sequencial. A segurança de redução**não** abrange a sobre-subscrição agregada (`oversubscribed: true`, campos fps 0): o excesso de câmaras para a ligação não pode ser corrigido através de agrupamento/ROI — reduza o número de câmaras, ative quadros jumbo ou mude para uma placa de rede mais rápida (consulte [Sobresubscrição](#over-subscription-the-per-cam-floor)).
- **O `analyze_array_network()` indica que o anel de receção da placa de rede está muito pequeno (~0,26 MB) / portas de ligação com «FRAMES WILL DROP&quot;** → O anel de receção da placa de rede do anfitrião está na sua configuração predefinida (muitas vezes redefinição para 32 após uma atualização do controlador da placa de rede). Num adaptador Realtek USB 10 GbE, defina `ReceiveBufferLen=256` e `PendingReceives=64` (elevado) e, em seguida, reinicie o backend para que este volte a ler o anel. Procedimento completo: [CLI Referência → Configuração e ajuste da NIC do anfitrião](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **O anfitrião bloqueia ao reiniciar/desligar, seguindo-se erros WMI `Invalid class` / a NIC nãoativar** → Driver USB 10GbE desatualizado a causar o erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o driver do adaptador para uma versão atual (≥ 2026) e volte a aplicar as definições do anel de receção. Consulte [Referência do CLI → Configuração e afinação da NIC do anfitrião](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Refletância recusada** → É necessário associar um DAQ ativo à câmara (ou matriz) para obter a refletância em escala absoluta. Efetue a associação através da GUI ou utilize o `processing="radiance"` (W/m²/sr/nm), que não requer um sensor emparelhado.
- **A captura do `smart=True` demora mais do que o esperado** → A convergência do AE depende da dinâmica da cena; aumente o valor de `exposure_tolerance_pct` ou reduza o de `stability_window_s` se pretender um disparador mais rápido (menos estável).

---

## Ver também

- [Referência CLI](cli-reference.md) — cada subcomando CLI corresponde a uma chamada SDK.
- [Guia de Sensores DAQ](../daq/README.md) — regras específicas de ligação, calibração e registo dos sensores.
- Documentação online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
