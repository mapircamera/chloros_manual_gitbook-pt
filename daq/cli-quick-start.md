# Guia de Início Rápido do CLI (pool-*)

As unidades `chloros-cli` fornecidas controlam os sensores DAQ através da família de comandos **`daq pool-*`** — clientes HTTP «leves» que operam o sensor através do conjunto persistente de sensores do backend Chloros. O backend é responsável pelo transporte, pelo que a GUI, a CLI e os scripts do SDK partilham todos um único identificador ativo, em vez de disputarem a porta. Tudo o que um cliente precisa está acessível através do `pool-*`: ligar, transmitir, gravar ficheiros `.daq` calibrados e trocar perfis de cap.

O `pool-*` é a interface DAQ suportada nas versões lançadas. O `chloros-cli daq --help` lista os subcomandos do `pool-*`, e o `daq discover` / `daq list` executam a mesma varredura que o `pool-discover`. Quando um subcomando de DAQ direto para o hardware não está disponível, o programa termina com um erro explícito que indica o pacote em falta e remete o utilizador de volta para o `pool-*` — nada falha silenciosamente. (O `pip install chloros-sdk` também não fornece os comandos de acesso direto ao hardware.)

***

## Pré-requisitos

* **O backend do Chloros deve estar a funcionar** — os comandos `pool-*` são clientes HTTP, não controladores de hardware. No Windows, inicie a aplicação de ambiente de trabalho do Chloros (ela inicia o backend). Num Linux/Jetson sem interface gráfica, ative o serviço como o seu utilizador: `sudo systemctl enable --now chloros-backend@$USER.service` (consulte [Instalação no Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Um login no Chloros+ (plano pago)**: execute primeiro o comando `chloros-cli login`. A aplicação é do lado do servidor — sem um login, os comandos falham com o `401 AUTH_REQUIRED`; no plano gratuito (Iron), falham com o `403 PLAN_UPGRADE_REQUIRED`.
* Por predefinição, os comandos têm como alvo o `http://127.0.0.1:5000`; a família `daq pool-*` respeita a variável de ambiente `CHLOROS_BACKEND_URL` se o seu backend estiver a ser executado noutro local.

***

## Uma sessão de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — abrir um sensor no conjunto

| Variante | Significado |
| --- | --- |
| `daq pool-connect` | Detecção inteligente: encontrar qualquer DAQ nesta máquina. |
| `daq pool-connect --port PORT` | DAQ-U numa porta série específica (por exemplo, `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC detetado automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M num MAC BLE conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E com um nome de anfitrião ou IP conhecido — **o caminho fiável**. |
| `daq pool-connect --eth` | DAQ-E com deteção automática (mDNS, com recurso a ARP). Consulte a advertência abaixo. |

Sinalizadores de ajuste, todos opcionais:

| Sinalizador | Significado |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tempo de integração manual em milissegundos. |
| `--frame-avg N` / `-f N` | Média de fotogramas por espectro reportado. |
| `--no-ae` | Desativar a exposição automática (a AE está ativada por predefinição). |
| `--no-stream` | Ligar sem iniciar a transmissão (retomar mais tarde com `pool-stream --start`). |
| `--cap-id CAP` | Perfil de correção de cap; o valor predefinido do backend é `sunshine_cosine`. Ver [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

A ligação de um DAQ-E que já se encontre no conjunto sob outro dos seus endereços — o seu nome de anfitrião `daq-e-<id>.local` ou o seu IP — devolve a entrada existente no conjunto, em vez de abrir a unidade uma segunda vez.

{% hint style="warning" %}
**Advertência sobre a deteção automática do `--eth`.** Num anfitrião com múltiplas ligações (mais do que uma interface de rede ativa), o *primeiro* `pool-connect --eth` após o arranque pode não encontrar nada, mesmo com o sensor em bom estado — a pesquisa de deteção pode não detetar a interface do sensor enquanto a cache ARP estiver vazia. Se o `--eth` não encontrar nada, tente novamente ou ignore completamente a descoberta com o `--eth-host <ip-or-hostname>`, que é o caminho fiável em máquinas com múltiplas interfaces de rede. O nome de anfitrião do DAQ-E é `daq-e-<id>.local` (por exemplo, `daq-e-def330.local`); o seu endereço IP simples também funciona.
{% endhint %}

## `pool-discover` — fazer uma varredura sem estabelecer ligação

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Procura sensores aos quais se pode ligar e apresenta o endereço para ligação, sem estabelecer a ligação. O `--only` aceita um subconjunto separado por vírgulas do `usb,ble,eth`; O `--timeout SEC` define o tempo de varredura por transporte (padrão: 5 s, limitado a 1–20 s). Os sensores já abertos no conjunto não são listados (um DAQ-M ligado deixa de anunciar a sua presença) — utilize o `pool-list` para esses casos. Numa versão comercializada, o `chloros-cli daq discover` e o `daq list` executam esta mesma verificação.

## `pool-list` — ver o que está ligado

Mostra todos os sensores no conjunto de back-end, incluindo o `sensor_id` de que todos os outros comandos necessitam:

| Modelo | Formato do `sensor_id` | Exemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos com hífen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Um DAQ-E cujo PTP tenha sido desativado (a configuração é guardada na unidade) é assinalado em `pool-list` com `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` e o comando que o reativa — ver [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — ler quadros de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Devolve o quadro mais recente ou os quadros mais recentes `--recent N`; o `--json` emite uma saída legível por máquina para a criação de scripts. Os quadros representam a irradiação espectral (W/m²/nm) calibrada radiometricamente numa grelha de 135 pontos, entre 340 e 1010 nm, com o perfil da tampa do sensor já aplicado. Para obter valores quantitativos de irradiância, calcule a média de, pelo menos, 15 segundos de fotogramas — trata-se de uma característica do instrumento, não de um defeito.

## `pool-stream` — pausar ou retomar a transmissão

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — gravar um ficheiro `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Indicador | Padrão | Significado |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Duração da gravação em segundos; `0` significa que a gravação deve decorrer até ser emitida a ordem `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Diretório de saída. Com o backend neste computador, um caminho relativo é resolvido em relação à pasta onde o comando foi executado; com um backend remoto, o caminho é resolvido **na máquina que executa o backend**. |
| `--device-name NAME` | — | Etiqueta armazenada com a gravação. |
| `--stop` | — | Interromper uma gravação em curso. |
| `--confirm-after SEC` | `20` | Após o início, aguarde até este tempo para que a primeira linha seja gravada e emita um aviso sonoro (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) se nenhuma chegar. `0` desativa a verificação. |
| `--heartbeat SEC` | `30` | Com `--duration`, imprima a contagem de linhas a cada SEC segundos e avise se deixar de aumentar. `0` desativa essa funcionalidade. |

{% hint style="info" %}
O registo ocorre no backend, pelo que o ficheiro `.daq` é guardado no sistema de ficheiros da **máquina do backend** — por predefinição, em `~/Documents/DAQ Live View/`, não necessariamente no local onde executou a CLI. Os nomes dos ficheiros incluem o ID do sensor e um carimbo de data/hora.
{% endhint %}

## `pool-set-cap` — declarar a tampa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

O ID da tampa seleciona o perfil de correção medido na fábrica aplicado a cada espectro e **tem de corresponder à tampa fisicamente instalada no sensor** — nem o sensor nem o software conseguem detetar a tampa por si próprios, e a seleção é registada em cada ficheiro `.daq`. O valor predefinido em todos os casos é `sunshine_cosine` (todos os DAQ são fornecidos com a tampa corretora de cosseno Sunshine instalada, com uma atenuação de cerca de 12× por design — uma alteração não declarada da tampa corrige incorretamente os espectros aproximadamente por esse fator).

| `--cap-id` | Disponível em |
| --- | --- |
| `sunshine_cosine` (padrão) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Apenas DAQ-U |
| `none` | Apenas DAQ-E — ver nota |

Um ID de tampa fora do conjunto do sensor é rejeitado na ligação, com um erro evidente. `none` (DAQ-E) significa que a tampa foi fisicamente removida — continua a aplicar um perfil geométrico de fábrica para o difusor de vidro recuado do DAQ-E, pelo que não é uma operação nula, e um DAQ-E sem tampa é uma configuração de bancada, não uma configuração de campo suportada. (Um DAQ-U sem tampa é verdadeiramente «nu» e não necessita de qualquer perfil de correção; o DAQ-M é utilizado com a sua tampa Sunshine.)

## `pool-disconnect` — libertar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

O comando `pool-disconnect` liberta um sensor independentemente de quem o tenha aberto, pelo que também liberta um sensor que um script que tenha falhado tenha deixado na piscina (só depois de a gravação nesse sensor ter terminado). Um sensor que não se encontre no conjunto não tem nada para libertar: o comando apresenta `<sensor-id>: already released (not in the pool)` e sai com `0`. Uma libertação que o backend recusou ou não confirmou sai com `1`.

## `pool-enable-ptp` — reativar o PTP de um DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Um DAQ-E mantém a sua configuração de ativação/desativação do PTP na sua própria memória; por isso, uma unidade cujo PTP tenha sido desativado permanece desativada — e os seus carimbos de data/hora não estão sincronizados com o PTP — até ser reativada. O comando `pool-list` assinala essa unidade e apresenta este comando para a mesma. O Chloros nunca reativa uma unidade por si próprio, uma vez que o estado «desativado» pode ser intencional. Novidade na versão 1.3.0.

## Configurações do dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` mostra o firmware do sensor (e se esta versão do Chloros inclui um mais recente), o seu endereço e como o obteve, a configuração de rede guardada caso difira da que está em execução, a transmissão ao ligar e o PTP. O `pool-set-network` reinicia o sensor para aplicar a alteração e volta a ligá-lo com as definições que tinha; o `--no-reboot` apenas as guarda. O endereço estático deve estar na rede deste computador; caso contrário, o Chloros rejeita-o; o `--force` anula essa restrição, mas apenas para um computador que venha a ser reendereçado ou encaminhado para ele. As reinicializações são recusadas enquanto o sensor estiver a registar. `pool-update-firmware` é o equivalente à opção **«Atualizar para \<version\>»** da interface gráfica para a linha de comandos e a forma de atualizar um DAQ-E no Linux, que não possui interface gráfica. Novidade na versão 1.3.0.

***

## Resumo dos comandos

| Comando | Finalidade |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir um sensor no conjunto de backends. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Procura sensores sem estabelecer ligação (localiza o endereço MAC BLE de um DAQ-M). |
| `daq pool-list` | Mostra todos os sensores no conjunto com o respetivo `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Os N quadros de espectro calibrados mais recentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Iniciar/parar uma gravação `.daq` (do lado do backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de limite durante a execução. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar um sensor ou todos eles. |
| `daq pool-enable-ptp --sensor-id ID` | Reativar o PTP num DAQ-E cujo PTP esteja desativado. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, endereço, configuração de rede guardada, transmissão ao ligar e PTP de um DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP ou um endereço estático; reinicia e reconecta o sensor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Transmissão ao ligar. |
| `daq pool-reboot --sensor-id ID` | Reinicia um DAQ-E e volta a ligá-lo. |
| `daq pool-update-firmware --sensor-id ID` | Atualiza o firmware do DAQ-E desta versão num sensor mais antigo. |

***

## Resolução de problemas na primeira ligação do DAQ-E

1. O DAQ-E não tem LED de estado — confirme a alimentação através do indicador PoE/ligação no switch ou na porta do injetor e aguarde alguns segundos após o arranque para que o dispositivo inicie e se junte à rede.
2. O computador de backend deve estar no **mesmo domínio de difusão** que o sensor — o mDNS não atravessa routers.
3. A firewall deve permitir a passagem do mDNS (UDP 5353), dos dados do DAQ-E (UDP 5002) e do PTP (UDP 319/320). O instalador do Windows adiciona uma regra para isso, e o `.deb` do Linux abre essas portas num `ufw` / `firewalld` que estava ativo no momento da instalação; um firewall de terceiros, ou um ativado posteriormente, precisa de permitir o acesso a estas portas.
4. Ainda nada do `--eth`? Utilize o `--eth-host` com o nome de host da unidade (`daq-e-<id>.local`) ou o IP, o que ignora a descoberta.

***
{% hint style="info" %}
**Dica para assistentes de IA.** Todas as páginas deste manual são apresentadas em Markdown bruto — acrescente `.md` ao slug da URL em minúsculas de uma página (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); o índice legível por máquina é `https://mapir.gitbook.io/chloros/llms.txt`. Para obter a documentação completa ao nível das opções de `chloros-cli daq` e de todas as outras famílias de comandos, consulte a [Referência da CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); o caminho Python é `chloros_sdk.connect_daq_sensor()` na [Referência do SDK](../reference/sdk-reference.md).
{% endhint %}
