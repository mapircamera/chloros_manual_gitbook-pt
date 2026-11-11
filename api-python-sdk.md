# API: Python SDK

{% hint style="info" %}
**Está à procura da API completa?** Esta página é um tutorial prático. Todas as classes públicas, métodos, assinaturas exatas e exemplos que podem ser copiados e colados encontram-se na [Referência do SDK](reference/sdk-reference.md), que está otimizada para assistentes de IA.**Está a trabalhar com um assistente de IA?** Cole este URL no chat para que tenha acesso à API completa e atualizada do Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Cada página deste manual está disponível como markdown bruto no seu slug em minúsculas + `.md`, e o manual completo está indexado em `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

O **Chloros Python SDK** (`chloros-sdk` no PyPI) controla tudo o que a aplicação de ambiente de trabalho pode fazer a partir de Python: processamento de imagens em lote, controlo em tempo real da câmara LATTICE e da matriz, sessões de aquisição de dados (DAQ) com sensores de luz e automatização de projetos guardados. Trata-se de uma camada fina sobre o mesmo backend local que a GUI e a CLI utilizam (HTTP em `127.0.0.1:5000`), pelo que o comportamento é idêntico nas três interfaces.

## Instalação

O SDK vem incluído no pacote de ambiente de trabalho do Chloros, que também fornece o backend de processamento e os runtimes de hardware, e a sua versão corresponde sempre à do pacote.

**Passo 1 — Instalar o Chloros.** Windows: execute o instalador para ambiente de trabalho (caminho predefinido `C:\Program Files\Chloros\`) a partir da página [Download](download.md). Linux: instale o pacote `.deb` com o `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalação no Linux](linux/linux-installation.md)).**Passo 2 — Verificar o SDK** (Python 3.7+). Normalmente, não é necessário instalar nada:

* **Windows:** o instalador instala o wheel do SDK incluído no Python do seu sistema (se não estiver instalado nenhum Python, este passo é ignorado).
* **Linux:** o `.deb` instala o SDK para o sistema `python3` no `/usr/lib/python3/dist-packages`. Os seus pré-requisitos, `python3-numpy` e `python3-requests`, são recomendados pelo pacote: o `apt install` inclui-os, enquanto o `dpkg -i` não.
* **Um virtualenv** não reconhece os pacotes do sistema; instale o ficheiro «wheel» incluído no pacote nele: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` no Linux, ou o `chloros_sdk-*.whl` na pasta `sdk` da pasta de instalação do Windows (`C:\Program Files\Chloros\sdk\` por predefinição).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Numa máquina sem o pacote Chloros, cujo SDK controla um backend noutra máquina (ver [O que uma instalação apenas com o pip obtém](#what-a-pip-only-install-gets)), instale-o a partir do PyPI: `pip install chloros-sdk`. O PyPI é atualizado nas compilações de lançamento, pelo que corresponde à versão estável mais recente.

**Passo 3 — Inicie sessão uma vez por máquina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache no `~/.chloros/` (em ambas as plataformas). No Windows, pode efetuar o início de sessão de forma equivalente através do separador «Utilizador» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> da aplicação para computador. O SDK requer um plano Chloros+ pago — consulte [Requisitos de licença](#license-requirement) abaixo.

| Requisito | Detalhes |
| --- | --- |
| **Chloros instalado** | Windows: instalador para computador; Linux: pacote `.deb` (fornece o binário do backend e o SDK) |
| **Python** | 3.7 ou superior (desenvolvido/testado na versão 3.10) |
| **Sistema operativo** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS ou mais recente, ou NVIDIA Jetson (JetPack 6) |
| **Licença** | Login ativo no Chloros+, qualquer nível pago (Copper ou superior) |

## A vitória em 60 segundos

Uma única chamada cria um projeto, importa uma pasta, configura o processamento e executa o pipeline — iniciando automaticamente o backend, caso ainda não esteja a funcionar:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(No Linux, utilize caminhos do Linux: `/home/user/drone_images/flight001`. O SDK funciona de forma idêntica em ambas as plataformas.)

A processar uma pasta de capturas LATTICE? Utilize o wrapper compatível com o LATTICE — este aplica as predefinições corretas (sem deteção do alvo do painel, debayer padrão):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controlo total do pipeline

Para qualquer coisa que vá além de uma linha de comando, utilize o `ChlorosLocal`. Este utiliza um backend que já esteja em execução (o da aplicação para computador) ou inicia o que está instalado na primeira utilização (`auto_start_backend=True`; no Windows, a partir da pasta `backend` da pasta de instalação do Chloros; `C:\Program Files\Chloros\backend\` por predefinição), cria e configura projetos, monitoriza o progresso e apresenta um resumo após a execução.

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
Mantenha o `http://127.0.0.1:5000` predefinido em vez de substituir por `localhost` — no Windows, o `localhost` é resolvido primeiro como `::1` e demora cerca de 2 segundos por pedido no backend apenas para IPv4.
{% endhint %}

Utilize-o como um gestor de contexto. Ao sair do bloco, termina-se a utilização do backend por esta instância, não o próprio backend: este permanece em execução para o próximo script e desliga-se automaticamente após 30 minutos sem utilização. O `shutdown_backend()` encerra uma instância já iniciada.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

O `configure()` aceita estas palavras-chave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. Os valores principais:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Os parâmetros específicos do LATTICE (`input_level`, `radiometric_output` e a família `array_alignment*`) estão documentados com as suas tabelas completas de valores na [Referência do SDK](reference/sdk-reference.md#supported-values).

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

Após a conclusão, o `process()` anexa o resumo de processamento do backend como `result["summary"]`. Cada entrada no `summary["hints"]` é uma frase completa que explica qualquer aspeto digno de nota — por exemplo, por que razão uma execução não produziu qualquer resultado — e cada indicação é também reemitida como um `UserWarning` em Python, pelo que as execuções vazias são autodiagnosticáveis, mesmo que nunca se inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**O `process()` não é acionado quando uma execução não produz imagens.** Este é o único ponto em que o SDK e a CLI diferem deliberadamente: O `chloros-cli process` trata «foram solicitados produtos, nenhum foi gravado» como uma falha e termina com um valor diferente de zero, enquanto o SDK retorna normalmente e reporta a condição através do `summary` / dicas. Se o seu pipeline tiver de parar numa execução vazia, verifique isso por si próprio — inspecione o `summary` (ou conte os ficheiros na pasta do projeto) em vez de confiar numa exceção.
{% endhint %}

## Smart Connect — hardware ativo

Três auxiliares abrem sessões persistentes no conjunto de hardware do backend — o mesmo conjunto que a CLI, outros scripts e os separadores de câmara da GUI utilizam, para que os scripts do SDK coexistam com eles sem disputarem portas seriais ou largura de banda de rede. Todos os três iniciam automaticamente um backend local se nenhum estiver a ser executado. O painel DAQ da GUI não faz parte do conjunto: feche um sensor de luz na GUI antes de o abrir a partir de um script.

### Câmara LATTICE única — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Os caminhos relativos apontam para o local onde o seu script é executado (alterado na versão 1.3.0).** Com um backend local, um caminho relativo `output_dir` (incluindo o caminho padrão `"output"`) — e os outros caminhos que o backend abre (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — são resolvidos em relação ao diretório de trabalho do seu script, pelo que o `cam.capture("output/")` grava no `<current folder>/output/`. Na versão 1.2.0, um backend instalado resolvia-os na sua própria pasta de instalação e apresentava a falha «Acesso negado». No caso de um backend remoto, os caminhos são enviados tal como foram digitados.
{% endhint %}

### Matriz sincronizada — `connect_array`

O `connect_array` é o ponto de entrada recomendado para configurações com várias câmaras. Executa o mesmo fluxo de preparação inteligente que a GUI: análise de rede, seleção automática do nível de sincronização, sincronização de tempo PTP, seleção do formato de píxeis por câmara, inicialização do AE e ativação do disparador GPIO. A **primeira porta série é a mestre** (é ela que dispara o impulso de acionamento do hardware); as restantes são escravas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adicione `smart=True` a qualquer captura em matriz para aguardar que a exposição automática se estabilize em todas as câmaras antes do disparo.

Novidade na versão 1.3.0: um `ArraySession` também fornece-lhe fotogramas sincronizados diretamente, sem gravar ficheiros:

* O `arr.grab()` devolve um fotograma por câmara, todos a partir de um único disparo, como matrizes numpy (`{serial: array}`); `.group_id` é o disparador partilhado e `.errors` identifica qualquer câmara que tenha ficado de fora.
* `arr.frame_stream(fps=5.0)` produz um conjunto deste tipo por tick, para processamento em tempo real no seu próprio pipeline.
* `arr.continuous("output/")` executa a captura contínua própria do backend (o modo «Contínuo» da GUI) e devolve um `ContinuousHandle` com `status()`, `wait()` e `stop()` — utilize-o em vez de um ciclo de captura quando precisar que cada disparo seja guardado a uma taxa elevada.

Para os modos de captura (Único / Contínuo / Intervalo / Mais rápido), gravadores, burst-to-video e alinhamento de matrizes, consulte a [Referência do SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sem argumentos, o `connect_daq_sensor()` deteta automaticamente o protocolo de transporte (ordem de prioridade: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada quadro transporta o `spectrum` de 135 pontos (W/m²/nm quando calibrado), um sinalizador `is_saturated` e CIE `x`, `y` e CIE `x`. Para selecionar um sensor ou protocolo de transporte específico, passe uma indicação explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Note que os perfis de correção de teto (`cap_id`) **não** são um parâmetro do SDK — selecione-os através de `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projetos guardados — `open_project`

Um projeto Chloros guardado mantém o hardware ligado (`cameras.json` + `sensors.json` juntamente com `project.json`), e o `chloros_sdk.open_project(path)` pode voltar a ligar as suas câmaras, matrizes e sensores de luz de uma só vez e iniciar capturas pelo nome do dispositivo. Os sensores de luz voltam a ligar-se através de um backend do Chloros em execução, com as suas definições e capacidade guardadas. Consulte [Automatização de Projetos](reference/sdk-reference.md#project-automation--chlorosproject) na referência.

## O que uma instalação apenas com o pip oferece

Verifique os indicadores de disponibilidade ao nível do módulo antes de utilizar superfícies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Num anfitrião com **apenas** o `pip install chloros-sdk` e sem o pacote de ambiente de trabalho Chloros:

* `ChlorosLocal`, `process_folder` e `process_lattice_capture` **não** funcionam — necessitam do binário de backend incluído no instalador do ambiente de trabalho.
* Os auxiliares smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) são clientes HTTP puros, pelo que funcionam com um backend noutro computador — mas os backends fornecidos ligam-se apenas ao loopback, pelo que deve reencaminhar a porta manualmente (por exemplo, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passar o `backend_url="http://127.0.0.1:5000"` juntamente com o `auto_start_backend=False`. Consulte [Modo de Backend Remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* As classes LATTICE de hardware direto (`LatticeCamera`, `CameraPool`, …) são importadas, mas requerem o tempo de execução do SDK da Arena, proveniente do pacote para computador — sem ele, o `CAMERA_AVAILABLE` é o `False`.
* Os sensores DAQ são controlados apenas através de um backend (tunelado) — `connect_daq_sensor()`, ou o `proj.sensors` de um projeto; o backend abre o sensor na sua própria máquina.

## Assistentes de IA — `chloros-mcp`

O SDK inclui o `chloros-mcp`, um servidor MCP que permite ao Claude Desktop, ao Claude Code, ao Cursor e a outros clientes MCP controlar o Chloros: processar imagens e operar câmaras LATTICE, matrizes e sensores de luz DAQ. As suas ferramentas abrangem as funcionalidades deste SDK. É instalado juntamente com o SDK, pelo que, para o Claude Code, a configuração resume-se a uma única linha:

```bash
claude mcp add chloros -- chloros-mcp
```

Outros clientes: [Utilizar o Chloros com assistentes de IA](ai-assistants.md). Opções e todas as ferramentas: [Servidor MCP (assistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Requisitos de licença

O acesso ao SDK requer um login ativo no Chloros+ em qualquer nível pago — **Copper ou superior**(Copper / Bronze / Silver / Gold); o plano gratuito Iron não tem acesso ao SDK/CLI. A aplicação desta regra é**do lado do servidor**: cada pedido de SDK deve incluir tanto uma sessão ativa como um plano pago; caso contrário, o backend devolve `403` / `PLAN_UPGRADE_REQUIRED` (gerado como `ChlorosLicenseError` pelo `ChlorosLocal` e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um utilizador que tenha terminado a sessão recebe `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — a reexecução do `chloros-cli login` corrige o primeiro caso, mas não o segundo.

A utilização offline funciona dentro do período de carência do plano: o nível é lido a partir da cache de validação do servidor (5 minutos) ou da cache de licenças assinadas e vinculadas ao equipamento (30 dias para planos mensais; até ao termo da subscrição para planos anuais). Quando o período de carência expira, o plano passa a ser gratuito e o acesso ao SDK é suspenso até que o computador se ligue ao servidor pelo menos uma vez. O `chloros-cli status` permanece acessível no nível gratuito, pelo que o motivo está sempre visível. Consulte [Início de sessão no Chloros+](chloros+-login.md).

## Exceções

Intercepte a classe base para tratar «qualquer problema no Chloros»:

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

Todas as exceções do pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivam de `ChlorosError`. Uma observação importante: `ChlorosConnectError` — gerado apenas por `connect_camera` / `connect_array` / `connect_daq_sensor` — deriva diretamente de `Exception`, **não** de `ChlorosError`, pelo que `except ChlorosError` não o detetará. A hierarquia completa encontra-se na [Referência do SDK](reference/sdk-reference.md#exceptions).

## Ver também

* [Referência do SDK](reference/sdk-reference.md) — a superfície completa da API, otimizada para assistentes de IA.
* [Referência da CLI](reference/cli-reference.md) — cada subcomando da CLI corresponde a uma chamada ao SDK.
* [Utilizar o Chloros com assistentes de IA](ai-assistants.md) — receitas de configuração e prompts do `chloros-mcp`.
* [Descarregar](download.md) — instaladores para Windows e Linux.
