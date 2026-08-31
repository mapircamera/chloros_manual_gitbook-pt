# Gravação e o formato .daq

Um ficheiro `.daq` é o formato de gravação do sensor de luz do Chloros: uma **base de dados SQLite** de fotogramas espectrais calibrados provenientes de um sensor DAQ. Basta gravar um durante uma sessão de captura e o pipeline de refletância poderá, posteriormente, dividir cada imagem pela irradiância descendente medida nesse exato momento.

## O que contém um ficheiro .daq

| Propriedade | Valor |
| --- | --- |
| Contentor | Base de dados SQLite, um ficheiro por sensor por gravação |
| Nome do ficheiro | Inclui o **ID do sensor**e um**carimbo temporal**, por exemplo, `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Espectro por fotograma | 135 pontos, 340–1010 nm em intervalos de 5 nm, mais tristímulo CIE XYZ |
| Unidades | Irradiação espectral calibrada, **W/m²/nm** (pacote de calibração de fábrica + perfil da tampa aplicado) |
| Metadados gravados | ID do sensor (a chave para obter a calibração de fábrica dessa unidade) e o perfil de tampa em vigor — ver [Perfis de tampa e intervalo calibrado](caps-and-range.md) |

O formato é idêntico no DAQ-U, no DAQ-M e no DAQ-E, pelo que o processamento a jusante não se preocupa com o dispositivo de transporte que o registou.

O registo calibrado requer o pacote de calibração de fábrica do sensor. Para o DAQ-U e o DAQ-M, o backend obtém o pacote da nuvem do MAPIR através do ID do sensor (a gravação é recusada se tal não for possível); as unidades DAQ-E estão isentas, uma vez que mantêm a sua calibração no próprio dispositivo.

## Gravação a partir da GUI

A gravação na GUI requer um **projeto aberto** (caso contrário, os botões «Gravar» ficam desativados):

* **Gravar tudo / Parar tudo** — na parte superior da barra lateral «Sensores de luz»; inicia ou interrompe uma gravação do `.daq` em todos os sensores ligados de uma só vez.
* **Gravar / Parar Gravação** — por sensor, na janela modal de definições (ícone da roda dentada). Um indicador vermelho «REC» aparece nas linhas de informação em tempo real do sensor durante a gravação.

Os ficheiros são gravados no `<project>/light_sensor/` e, quando uma gravação termina — seja através de «Parar», «Parar tudo» ou da desconexão de um sensor de gravação —, o `.daq` concluído é **automaticamente adicionado ao projeto aberto**. Aparece na lista de ficheiros do projeto sem necessidade de adição manual, já pronto para o processamento de refletância.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Gravação a partir do CLI

O CLI grava através do conjunto de sensores do backend (o backend tem de estar em execução — estes comandos são clientes HTTP «leves»):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Obter o valor do `--sensor-id` a partir do `chloros-cli daq pool-list`. Dois valores predefinidos que vale a pena conhecer:

| Opção | Padrão |
| --- | --- |
| `--duration` | `0` — registar até `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` no sistema de ficheiros do **backend**, e não no do CLI |

A distinção do diretório de saída é importante quando o CLI tem como destino um backend noutro computador: o ficheiro é guardado no local onde o backend é executado.

## Gravação a partir do Python

O `DAQSensorSession` (retornado pelo `chloros_sdk.connect_daq_sensor()`) expõe a mesma gravação agrupada: O `record_start(output_dir=None, device_name=None)` devolve o caminho do ficheiro, enquanto o `record_stop()` devolve o `{path, rows}`. Consulte a [Referência do SDK](../reference/sdk-reference.md) para a sessão completa API. As classes de hardware direto do SDK (apenas para instalações em computadores de secretária) gravam as gravações no `~/Documents/DAQ/` por predefinição; para versões lançadas, o caminho agrupado acima é a rota suportada.

## Utilização de um ficheiro .daq na fase de processamento

Para calcular a refletância a partir de imagens, o Chloros necessita da irradiância descendente correspondente a cada exposição:

* **Mantenha o `.daq` juntamente com as imagens.**Na fase de processamento, o pipeline determina automaticamente a**irradiação descendente com data e hora correspondentes** a partir de um `.daq` gravado (qualquer modelo de DAQ) — ou a partir de um `.csv` nativo do DAQ-M — encontrado juntamente com as imagens. As gravações da GUI cumprem este requisito automaticamente, uma vez que são adicionadas ao projeto no momento em que terminam.
* **A calibração é obtida mediante solicitação.** Se um pacote de calibração de fábrica por câmara ou por DAQ ainda não estiver armazenado em cache localmente, o Chloros obtém-no automaticamente da nuvem do MAPIR na primeira utilização (é necessária ligação à Internet uma vez; armazenado em cache no `~/.chloros/`).
* **As capturas em tempo real gravam o seu próprio ficheiro sidecar.** Para qualquer fotograma de refletância capturado em tempo real, a leitura do DAQ efetivamente utilizada é guardada como um ficheiro sidecar `.daq` ao lado das imagens, para que a captura possa ser reprocessada posteriormente sem a gravação original.

## Recuperar a irradiância

O processamento de um projeto também exporta todos os registos do sensor de luz que contém para uma
pasta `Light Sensor/` ao lado dos produtos de imagem. Isto **não** requer imagens: um
sensor de luz utilizado isoladamente constitui uma captura completa, e uma pasta que contenha apenas ficheiros `.daq`
é uma entrada válida. A execução indica quantos produtos do sensor de luz foram gravados.

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | Um arquivo reprocessável com o mesmo esquema de um registo em tempo real, que agora declara o pacote de calibração que o produziu. A sua reimportação **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grelha de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas: potência total, lux fotópico e escotópico, PPFD com a sua divisão em azul/verde/vermelho e comprimento de onda de pico. |

Um DAQ-U ou DAQ-M cujo pacote de calibração não possa ser obtido — está offline ou
esse sensor não tem nenhuma calibração registada — é **ignorada com indicação do motivo**, nunca sendo gravada
como um ficheiro «calibrado» contendo contagens brutas. Ligue-se à Internet e execute novamente. Um DAQ-E
possui a sua própria calibração, pelo que só precisa disto quando a unidade não está ligada e
nada está armazenado em cache localmente.

### DAQ-A: contagens brutas e por que razão essa é a resposta certa

O **DAQ-A** é anterior ao sistema de pacotes de calibração por porta série e não tem nenhum pacote para
obter. Isso não é um lapso: um DAQ-A é calibrado no terreno em relação a um
alvo de refletância, e a calibração baseada no alvo requer apenas a resposta *relativa*
do sensor — que é exatamente o que são as suas contagens brutas. O Chloros é calibrado com elas atualmente.

Assim, um registo do DAQ-A é exportado, mas com um nome diferente:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, e não `_calibrated` — um nome de ficheiro diferente, em vez de um indicador no interior do ficheiro,
porque a informação tem de se manter intacta quando o ficheiro for enviado por e-mail apenas com o nome. O cabeçalho `.csv`
indica `raw spectral sensor counts (NOT irradiance)` e avisa que os valores são
comparáveis **dentro** do ficheiro e não entre sensores. As colunas que só têm
significado para a irradiação real — potência total, lux, PPFD — são deixadas em branco, em vez de
serem calculadas a partir de contagens.

As gravações mais antigas do DAQ-A-SD (esquema v1.01 / v1.02) registam apenas a hora de gravação do ficheiro, e não um
carimbo de data/hora por leitura. O Chloros não irá fazer a correspondência das imagens com essas — emparelhar um fotograma com uma
hora de gravação seria errado, mesmo que não parecesse errado — mas a exportação lê-as corretamente e
o CSV indica em que relógio se encontra.

Para obter informações completas sobre a refletância — sensor único com câmara e sensor duplo (ambiente/objeto) — consulte [Fluxos de Trabalho de Refletância](reflectance.md).
