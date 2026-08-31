# API : Python SDK

{% hint style="info" %}
**Está à procura do API completo?** Esta página é um tutorial prático. Todas as classes públicas, métodos, assinaturas exatas e exemplos que podem ser copiados e colados encontram-se na [Referência do SDK](reference/sdk-reference.md), que está otimizada para assistentes de IA.**Está a trabalhar com um assistente de IA?** Cole este URL no chat para que tenha o Chloros 1.2.0 API completo e atualizado:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Todas as páginas deste manual estão disponíveis em formato Markdown bruto no endereço com o slug em minúsculas + `.md`, e o manual completo está indexado em `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

O **Chloros Python SDK** (`chloros-sdk` no PyPI) controla todas as funcionalidades do aplicativo de ambiente de trabalho a partir de Python: processamento em lote de imagens, controlo em tempo real de câmaras e matrizes LATTICE, sessões de sensores de luz DAQ e automação de projetos guardados. Trata-se de uma camada fina sobre o mesmo backend local que a GUI e o CLI utilizam (HTTP no `127.0.0.1:5000`), pelo que o comportamento é idêntico nas três interfaces.

## Instalação

A instalação é feita em duas etapas: primeiro, o pacote para computador Chloros (que fornece o backend de processamento e os runtimes de hardware) e, em seguida, o pacote Python.

**Passo 1 — Instalar o Chloros.** Windows: execute o instalador para ambiente de trabalho (caminho predefinido `C:\Program Files\MAPIR\Chloros\`) a partir da página [Transferências](download.md). Linux: instale o pacote `.deb` ([Instalação do Linux](linux/linux-installation.md)).**Passo 2 — Instalar o SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Pode nem precisar do pip: todos os instaladores incluem um wheel SDK correspondente. O instalador Windows instala-o automaticamente no seu sistema Python; o Linux `.deb` coloca-o em `/usr/lib/chloros/sdk/` e apresenta o comando exato `pip install --user`. O PyPI é atualizado nas compilações de lançamento, pelo que o `pip install chloros-sdk` corresponde à versão estável mais recente.

**Passo 3 — Inicie sessão uma vez por máquina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache em `~/.chloros/` (em ambas as plataformas). No Windows, pode iniciar sessão de forma equivalente através do separador «<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» do utilizador na aplicação para computador. O SDK requer um plano Chloros+ pago — consulte [Requisitos de licença](#license-requirement) abaixo.

| Requisito | Detalhes |
| --- | --- |
| **Chloros instalado** | Windows: instalador para computador; Linux: pacote `.deb` (fornece o binário de backend) |
| **Python** | 3.7 ou superior (desenvolvido/testado na versão 3.10) |
| **Sistema operativo** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS ou mais recente, ou NVIDIA Jetson (JetPack 6) |
| **Licença** | Início de sessão ativo no Chloros+, qualquer nível pago (Copper ou superior) |

## A vitória em 60 segundos

Uma única chamada cria um projeto, importa uma pasta, configura o processamento e executa o pipeline — iniciando automaticamente o backend, caso este ainda não esteja a funcionar:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(No Linux, utilize os caminhos do Linux: `/home/user/drone_images/flight001`. O SDK funciona de forma idêntica em ambas as plataformas.)

A processar uma pasta de captura do LATTICE? Utilize o wrapper compatível com o LATTICE — este aplica as predefinições corretas (sem deteção do alvo do painel, debayer padrão):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controlo total do pipeline

Para qualquer coisa que vá além de uma linha de comando, utilize o `ChlorosLocal`. Este programa inicia o backend na primeira utilização (`auto_start_backend=True`), cria e configura projetos, monitoriza o progresso e apresenta um resumo após a execução.

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

{% hint style="info" %}
Mantenha o `http://127.0.0.1:5000` predefinido em vez de substituí-lo pelo `localhost` — no Windows, o `localhost` é resolvido primeiro para o `::1` e demora cerca de 2 segundos por pedido no backend apenas IPv4.
{% endhint %}

Utilize-o como gestor de contexto para garantir a limpeza:

```python
import chloros_sdk

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

O `configure()` aceita estas palavras-chave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. Os valores principais:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Os controlos específicos da LATTICE (`input_level`, `radiometric_output`, a família `array_alignment*`) estão documentados com as suas tabelas completas de valores na [Referência do SDK](reference/sdk-reference.md#supported-values).

### Acompanhar o progresso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Ler o resumo pós-execução — e detetar execuções vazias

Após a conclusão, o `process()` anexa o resumo de processamento do backend como `result["summary"]`. Cada entrada no `summary["hints"]` é uma frase completa que explica qualquer aspeto digno de nota — por exemplo, por que razão uma execução produziu zero resultados — e cada indicação é também reemitida como um Python `UserWarning`, pelo que as execuções vazias são autodiagnosticáveis, mesmo que nunca inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` não é gerado quando uma execução não produz imagens.** Este é o único ponto em que o SDK e o CLI diferem deliberadamente: O `chloros-cli process` trata «foram solicitados produtos, nenhum foi gravado» como uma falha e termina com um valor diferente de zero, enquanto o SDK termina normalmente e reporta a condição através do `summary` / dicas. Se o seu pipeline tiver de parar numa execução vazia, verifique isso por si próprio — inspecione o `summary` (ou conte os ficheiros na pasta do projeto) em vez de confiar numa exceção.
{% endhint %}

## Smart Connect — hardware ativo

Três auxiliares abrem sessões persistentes no conjunto de hardware do backend — o mesmo conjunto que a GUI utiliza, pelo que os scripts SDK coexistem com a aplicação de ambiente de trabalho sem disputarem portas seriais ou largura de banda de rede. Todos os três iniciam automaticamente um backend local, caso nenhum esteja a ser executado.

### Câmara LATTICE única — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Matriz sincronizada — `connect_array`

O `connect_array` é o ponto de entrada recomendado para configurações com várias câmaras. Executa o mesmo fluxo de preparação inteligente que a interface gráfica: análise de rede, seleção automática do nível de sincronização, sincronização de tempo PTP, seleção do formato de píxeis por câmara, inicialização do AE e ativação do gatilho GPIO. A **primeira porta série é a mestre** (é ela que dispara o impulso de acionamento do hardware); as restantes são escravas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adicione o `smart=True` a qualquer captura em matriz para aguardar que a exposição automática se estabilize em todas as câmaras antes do disparo. Para os modos de captura (Única / Contínua / Intervalo / Mais rápida), gravadores, burst-to-video e alinhamento de matriz, consulte a [Referência do SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sem argumentos, o `connect_daq_sensor()` deteta automaticamente o protocolo de transporte (ordem de prioridade: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada quadro transporta o valor de 135 pontos `spectrum` (W/m²/nm quando calibrado), um sinalizador `is_saturated` e CIE `x`, `y`, `z`. Para especificar um sensor ou protocolo de transporte específico — a escolha mais fiável em hosts com várias interfaces de rede, onde a deteção automática por Ethernet pode não identificar um DAQ-E em bom estado na primeira tentativa —, passe uma indicação explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Note que os perfis de correção de maiúsculas (`cap_id`) **não** são um controlador SDK — selecione-os através de `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projetos guardados — `open_project`

Um projeto Chloros guardado mantém o hardware ligado (`cameras.json` + `sensors.json` juntamente com `project.json`), e o `chloros_sdk.open_project(path)` pode voltar a ligar tudo de uma só vez e efetuar capturas por nome do dispositivo. Consulte [Automatização de Projetos](reference/sdk-reference.md#project-automation--chlorosproject) na referência.

## O que uma instalação apenas via pip oferece

Verifique os indicadores de disponibilidade ao nível do módulo antes de utilizar superfícies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Num anfitrião com **apenas** o `pip install chloros-sdk` e sem o pacote de ambiente de trabalho Chloros:

* O `ChlorosLocal`, o `process_folder` e o `process_lattice_capture` **não** funcionam — necessitam do binário de backend incluído no instalador do ambiente de trabalho.
* Os auxiliares de ligação inteligente (`connect_camera`, `connect_array`, `connect_daq_sensor`) são clientes puros do HTTP, pelo que funcionam com um backend noutro computador — mas os backends fornecidos ligam-se apenas ao loopback, pelo que terá de reencaminhar a porta manualmente (por exemplo, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passar o `backend_url="http://127.0.0.1:5000"` com o `auto_start_backend=False`. Consulte [Modo de Backend Remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* As classes LATTICE de hardware direto (`LatticeCamera`, `CameraPool`, …) podem ser importadas, mas necessitam do tempo de execução Arena SDK do pacote para computador — sem ele, `CAMERA_AVAILABLE` é `False`.
* O `daq_sdk` (as classes DAQ diretas) é fornecido com a instalação para computador, e não com o pacote PyPI, pelo que o `DAQ_AVAILABLE` é o `False` num host que utilize apenas o pip — em vez disso, controle os sensores DAQ através do `connect_daq_sensor()` contra um backend (tunelado).

## Requisitos de licença

O acesso ao SDK requer um login ativo no Chloros+ em qualquer nível pago — **Copper ou superior**(Copper / Bronze / Silver / Gold); o plano gratuito Iron não tem acesso ao SDK/CLI. A aplicação é**do lado do servidor**: cada pedido SDK deve incluir tanto uma sessão ativa como um plano pago; caso contrário, o backend devolve `403` / `PLAN_UPGRADE_REQUIRED` (gerado como `ChlorosLicenseError` pelo `ChlorosLocal` e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um utilizador que tenha saído da sessão recebe os códigos de erro `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — a reexecução do `chloros-cli login` corrige o primeiro caso, mas não o segundo.

A utilização offline funciona dentro do período de carência do plano: o nível é lido a partir da cache de validação do servidor (5 minutos) ou da cache de licenças assinadas e vinculadas ao equipamento (30 dias para planos mensais; até ao termo da subscrição para planos anuais). Quando o período de carência expira, o plano passa a ser gratuito e o acesso ao SDK é interrompido até que o computador se ligue ao servidor uma vez. O `chloros-cli status` permanece acessível no nível gratuito, pelo que o motivo está sempre visível. Consulte [Chloros+ Início de sessão](chloros+-login.md).

## Exceções

Intercepte a classe base para lidar com «qualquer coisa que tenha corrido mal»:

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

Todas as exceções do pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivam de `ChlorosError`. Uma observação importante: `ChlorosConnectError` — gerado apenas por `connect_camera` / `connect_array` / `connect_daq_sensor` — deriva do simples `Exception`, **não** do `ChlorosError`, pelo que o `except ChlorosError` não o detetará. A hierarquia completa encontra-se na [Referência do SDK](reference/sdk-reference.md#exceptions).

## Ver também

* [Referência do SDK](reference/sdk-reference.md) — a superfície completa do API, otimizada para assistentes de IA.
* [Referência do CLI](reference/cli-reference.md) — cada subcomando do CLI corresponde a uma chamada do SDK.
* [Descarregar](download.md) — programas de instalação para o Windows e o Linux.
