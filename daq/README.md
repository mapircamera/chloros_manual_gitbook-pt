# Sensores de luz DAQ

> **Está à procura do hardware?**Os próprios sensores — modelos, montagem, tampas, portas, alimentação e a aplicação SCANNER — estão descritos no**[manual do utilizador do DAQ](https://mapir.gitbook.io/daq)**. Este capítulo aborda a sua utilização a partir do Chloros.

Os sensores de luz **DAQ** do MAPIR medem a luz ambiente sob a forma de espectros calibrados radiometricamente. No Chloros, desempenham duas funções:

* **Um instrumento espectral autónomo** — gráficos de espectro em tempo real, dados colorimétricos e gravações do `.daq`, tudo a partir do [separador «Sensores de luz»](gui.md), o [CLI](cli-quick-start.md) ou o Python SDK.
* **Uma fonte de irradiância descendente para a refletância** — durante o processamento, o Chloros interpola as suas leituras do `.daq` para cadae utiliza a luz descendente medida para converter a radiação da câmara em refletância (`--reflectance-source daq`), não sendo necessário um painel na cena para as bandas calibradas.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Três modelos, um formato de dados

| Modelo | Transporte | Descoberta |
| --- | --- | --- |
| **DAQ-U** | USB (serial) | pesquisa por porta serial |
| **DAQ-M** | Bluetooth Low Energy | pesquisa BLE por nome |
| **DAQ-E** | Ethernet (IPv4, alimentação por PoE) | mDNS `_daq-e._tcp` (nome de anfitrião `daq-e-<id>.local`) |

Todos os três utilizam o mesmo protocolo de comunicação e fornecem dados idênticos:

* Um **espectro de 135 pontos, de 340 a 1010 nm, em intervalos de 5 nm**, além dos valores tristimulares CIE XYZ, em cada quadro.
* **Irradiação espectral calibrada radiometricamente em W/m²/nm** — o pacote de calibração de fábrica de cada unidade (juntamente com o seu perfil de correção de limite ativo) é aplicado antes de os dados chegarem até si.
* O mesmo **formato de gravação `.daq`** (um ficheiro SQLite). O processamento posterior é idêntico, independentemente do meio de transporte que tenha gerado o ficheiro.

As pilhas de transporte (serial USB, BLE, mDNS/zeroconf) estão integradas no backend Chloros — não é necessário instalar nada para comunicar com qualquer um dos três modelos a partir da interface gráfica ou dos comandos `pool-*` do CLI.

***

## Intervalo calibrado: 340–1010 nm reportado, ~374–974 nm calibrado

O sensor reporta a grelha completa de 340–1010 nm, mas o ganho radiométrico rastreável pelo NIST abrange aproximadamente **374–974 nm**. O comando Chloros recusa a divisão da refletância absoluta para qualquer banda da câmara com menos de metade do seu peso espectral dentro desse intervalo calibrado; a banda ignorada é indicada com o motivo de omissão `dls-uncalibrated-band-<nm>`.

Entre os SKUs de filtros LATTICE disponíveis, apenas o **F988** é afetado:

A refletância do F988 é calibrada utilizando um painel de refletância na cena: a banda situa-se fora da gama calibrada do sensor de luz do DAQ, pelo que o Chloros aplica a sua captura mais recente do painel e mantém-na entre as observações do painel.

Se uma captura do F988 for processada com apenas dados do DAQ disponíveis, o Chloros rejeita a refletância baseada no DAQ para essa banda com o motivo de omissão `dls-uncalibrated-band-988` — o [fluxo de trabalho do painel de refletância](../calibration-targets.md) é o caminho suportado para o F988.

***

## IDs dos sensores

Cada DAQ reporta um ID de sensor estável. O seu formato difere consoante o modelo:

| Modelo | Formato do ID | Exemplo |
| --- | --- | --- |
| DAQ-U | 5 octetos com hífens | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octetos com hífen | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

O ID do sensor é:

* gravado em todos os ficheiros `.daq` que regista,
* a chave que o Chloros utiliza para obter o pacote de calibração de fábrica dessa unidade,
* o valor que passa ao `--sensor-id` nos comandos CLI e `pool-*`, e
* no caso do DAQ-E, também o seu nome de anfitrião mDNS (`daq-e-def330.local`) — o valor que o `--eth-host` aceita.

***

## Calibração de fábrica e a nuvem

Cada unidade DAQ é calibrada individualmente na fábrica com uma cadeia radiométrica rastreável pelo NIST, e o Chloros carrega o pacote de calibração de cada unidade, identificado pelo ID do seu sensor. O relatório de calibração por unidade (PDF) pode ser descarregado a partir das definições do sensor no [separador «Sensores de luz»](gui.md).

{% hint style="warning" %}
**O DAQ-U e o DAQ-M requerem acesso à nuvem para a calibração.**Nenhum dos modelos armazena nada no próprio dispositivo: os seus pacotes de calibração de fábrica encontram-se na nuvem do MAPIR e são obtidos através do ID do sensor (sendo depois armazenados em cache localmente). O Chloros necessita de uma ligação à Internet para fornecer dados calibrados em W/m²/nm a partir de um DAQ-U ou DAQ-M.**O DAQ-E é a exceção** — este dispositivo armazena a sua calibração no próprio dispositivo.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Onde são guardadas as gravações

| Superfície | Destino predefinido do `.daq` |
| --- | --- |
| GUI — separador «Sensores de luz» | `<project folder>/light_sensor/` (os registos concluídos são adicionados automaticamente ao projeto) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na máquina que executa o backend |

Cada nome de ficheiro `.daq` inclui o ID do sensor e um registo temporal.

***

## Neste capítulo

* [**O separador «DAQ» no Chloros**](gui.md) — o guia completo da interface gráfica: ligação de cada modelo, definições por sensor, gráficos de espectro, dados colorimétricos em tempo real, refletância de sensor duplo e gravação.
* [**Início rápido do CLI (pool-\*)**](cli-quick-start.md) — controlo dos sensores DAQ a partir do `chloros-cli daq pool-*`, o caminho de linha de comandos suportado.
* [**Perfis de Limite e Gama Calibrada**](caps-and-range.md) — quais os limites existentes por modelo, como declará-los e a gama espectral calibrada em pormenor.
* [**Gravação e o formato .daq**](recording.md) — o formato SQLite do `.daq` e os fluxos de trabalho de gravação.
* [**Rede DAQ-E e sincronização de tempo**](ethernet-ptp.md) — modos de transporte DAQ-E e sincronização de tempo PTP.
* [**Fluxos de trabalho de refletância**](reflectance.md) — utilização de dados de radiação descendente do DAQ para calcular a refletância.
* Para obter documentação completa ao nível dos sinalizadores, consulte a [Referência CLI](../reference/cli-reference.md) (secção `chloros-cli daq`) e a [Referência SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ambas redigidas para serem diretamente utilizáveis por assistentes de IA.
