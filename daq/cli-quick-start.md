# CLI Guia de Início Rápido (pool-*)

As unidades `chloros-cli` fornecidas controlam os sensores DAQ através da família de comandos **`daq pool-*`** — clientes HTTP «light» que operam o sensor através do conjunto de sensores persistente do backend Chloros. O backend é responsável pelo transporte, pelo que a GUI, o CLI e os scripts SDK partilham todos um único identificador ativo, em vez de disputarem a porta. Tudo o que um cliente precisa está acessível através do `pool-*`: ligar, transmitir, gravar ficheiros `.daq` calibrados e trocar perfis de cap.

O `pool-*` é também a **única** superfície DAQ nas compilações lançadas. O `chloros-cli daq --help` lista os subcomandos do `pool-*`, e a invocação de um subcomando DAQ de acesso direto ao hardware numa versão lançada termina com um erro explícito que indica o pacote em falta e remete-o de volta para o `pool-*` — nada falha silenciosamente. (Os comandos de DAQ direto de hardware só funcionam a partir de um checkout da fonte MAPIR; o `pip install chloros-sdk` também não os disponibiliza.)

***

## Pré-requisitos

* **O backend Chloros tem de estar em execução** — os comandos `pool-*` são clientes do HTTP, não controladores de hardware. No Windows, inicie a aplicação de ambiente de trabalho Chloros (esta inicia o backend). No Linux/Jetson sem ecrã, ative o serviço: `sudo systemctl enable --now chloros-backend.service`.
* **Um início de sessão no Chloros+ (nível pago)**: execute primeiro o `chloros-cli login`. A aplicação da política é do lado do servidor — sem um login, os comandos falham com o `401 AUTH_REQUIRED`; no plano gratuito (Iron), falham com o `403 PLAN_UPGRADE_REQUIRED`.
* Por predefinição, os comandos têm como alvo o `http://127.0.0.1:5000`; a família `daq pool-*` respeita a variável de ambiente `CHLOROS_BACKEND_URL` se o seu backend estiver a ser executado noutro local.

***

## Uma sessão de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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
| `--cap-id CAP` | Perfil de correção de cap; o valor predefinido do backend é `sunshine_cosine`. Consulte [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Advertência sobre a descoberta automática do `--eth`.** Num anfitrião com múltiplas ligações (mais do que uma interface de rede ativa), o *primeiro* `pool-connect --eth` após o arranque pode não apresentar resultados, mesmo com o sensor em bom estado — a pesquisa de descoberta pode não detetar a interface do sensor enquanto a cache ARP estiver inativa. Se o `--eth` não encontrar nada, tente novamente ou ignore completamente a deteção com o `--eth-host <ip-or-hostname>`, que é o caminho fiável em máquinas com várias interfaces de rede. O nome de host do DAQ-E é `daq-e-<id>.local` (por exemplo, `daq-e-def330.local`); o seu endereço IP simples também funciona.
{% endhint %}

## `pool-list` — ver o que está ligado

Mostra todos os sensores no conjunto de backends, incluindo o `sensor_id` de que todos os outros comandos necessitam:

| Modelo | Formato do `sensor_id` | Exemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos com hífen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — ler quadros de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Devolve o quadro mais recente ou os quadros mais recentes; o `--recent N` emite uma saída legível por máquina para a criação de scripts. Os quadros representam a irradiância espectral (W/m²/nm) calibrada radiometricamente numa grelha de 135 pontos, entre 340 e 1010 nm, com o perfil da tampa do sensor já aplicado. Para obter valores quantitativos de irradiância, calcule a média de, pelo menos, 15 segundos de quadros — trata-se de uma característica do instrumento, não de um defeito.

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
| `--duration SEC` / `-d SEC` | `0` | Duração da gravação em segundos; `0` significa que a execução deve continuar até que se emita o comando `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Diretório de saída, resolvido **na máquina que executa o backend**. |
| `--device-name NAME` | — | Etiqueta armazenada com a gravação. |
| `--stop` | — | Interrompe uma gravação em curso. |

{% hint style="info" %}
A gravação ocorre no backend, pelo que o ficheiro `.daq` fica no sistema de ficheiros da **máquina do backend** — por predefinição, no `~/Documents/DAQ Live View/`, e não necessariamente no local onde executou o CLI. Os nomes dos ficheiros incluem o ID do sensor e um carimbo de data/hora.
{% endhint %}

## `pool-set-cap` — declarar a tampa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

O ID da tampa seleciona o perfil de correção medido de fábrica aplicado a cada espectro e **tem de corresponder à tampa fisicamente instalada no sensor** — nem o sensor nem o software conseguem detetar a tampa por si próprios, e a seleção é registada em cada ficheiro `.daq`. O valor predefinido em todos os casos é `sunshine_cosine` (todos os DAQ são fornecidos com a tampa corretora de cosseno Sunshine instalada, com uma atenuação de cerca de 12× por definição — uma alteração não declarada da tampa corrige incorretamente os espectros aproximadamente por esse fator).

| `--cap-id` | Disponível em |
| --- | --- |
| `sunshine_cosine` (padrão) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Apenas DAQ-U |
| `none` | Apenas DAQ-E — ver nota |

Um ID de tampa fora do conjunto do sensor é rejeitado na ligação, com um erro explícito. `none` (DAQ-E) significa que a tampa foi fisicamente removida — continua a aplicar um perfil geométrico de fábrica para o difusor de vidro recuado do DAQ-E, pelo que não se trata de uma operação nula, e um DAQ-E sem tampa é uma configuração de bancada, não uma configuração de campo suportada. (Um DAQ-U sem tampa é verdadeiramente «nu» e não necessita de qualquer perfil de correção; o DAQ-M é utilizado com a sua tampa Sunshine.)

## `pool-disconnect` — libertar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Resumo dos comandos

| Comando | Finalidade |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir um sensor no conjunto de sensores do backend. |
| `daq pool-list` | Mostrar todos os sensores no conjunto com o seu `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Os N quadros de espectro calibrados mais recentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Iniciar/parar uma gravação `.daq` (no backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de limite durante a execução. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar um sensor ou todos eles. |

***

## Resolução de problemas na primeira ligação do DAQ-E

1. O DAQ-E não tem LED de estado — confirme a alimentação através do indicador PoE/ligação no comutador ou na porta do injetor e aguarde alguns segundos após o arranque para que o dispositivo inicie e se junte à rede.
2. O computador backend deve estar no **mesmo domínio de difusão** que o sensor — o mDNS não atravessa routers.
3. No Windows, aceite o aviso da firewall Defender na primeira execução (mDNS UDP 5353, dados do DAQ-E UDP 5002, PTP UDP 319/320).
4. Ainda não há resposta do `--eth`? Utilize o `--eth-host` com o nome de anfitrião da unidade (`daq-e-<id>.local`) ou o endereço IP — este é o caminho mais fiável, especialmente em anfitriões com múltiplas ligações.

***{% hint style="info" %}**Dica para assistentes de IA.** Todas as páginas deste manual são apresentadas em Markdown bruto — acrescente `.md` ao slug em minúsculas de uma página, URL (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); o índice legível por máquina é `https://mapir.gitbook.io/chloros/llms.txt`. Para obter a documentação completa ao nível dos sinalizadores do `chloros-cli daq` e de todas as outras famílias de comandos, consulte a [Referência do CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); o caminho do Python é o `chloros_sdk.connect_daq_sensor()` na [Referência do SDK](../reference/sdk-reference.md).
{% endhint %}
