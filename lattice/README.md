# Câmaras LATTICE

A LATTICE é o sistema modular de câmaras multiespectrais da MAPIR destinado à imagem agrícola e científica. Todas as câmaras LATTICE são construídas com base no sensor de obturador global Sony IMX265 (**3,1 MP, píxeis de 3,45 µm**) e ligam-se através de Ethernet como um dispositivo**GigE Vision**.

O Chloros 1.2.0 controla as câmaras LATTICE em tempo real — deteção, pré-visualização em tempo real, captura e matrizes sincronizadas de várias câmaras — a partir de três interfaces:

| Interface    | Onde                                                          | Plataformas                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Separador **Câmaras** na barra lateral do Chloros                         | Windows 10/11 x64                                        |
| CLI        | Família de comandos `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |

> **À procura do hardware?**Os módulos de câmara, lentes, filtros e bandas, estruturas e suportes de montagem, cabos, PoE e cablagem de disparo estão documentados no [**manual do utilizador da LATTICE**](https://mapir.gitbook.io/lattice-camera). Este capítulo aborda o controlo das câmaras a partir do Chloros.

As capturas do LATTICE são ficheiros padrão `.tif`/`.tiff`, e o Chloros processa-as sempre a partir da captura em bruto. Consulte a [Referência do CLI](../reference/cli-reference.md) e a [Referência do SDK](../reference/sdk-reference.md) para obter o comando completo e a superfície API.

## Duas configurações de sensores

| Configuração | Sensor       | Filtro                                | O que uma câmara fornece                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Cor Bayer | filtro de passagem de banda tripla                |**Três bandas calibradas a partir de uma única exposição**                 |
| **M3M**| Monocromático   | filtro de interferência de banda estreita única |**Uma banda calibrada**; combine várias câmaras M3M para obter índices |

Como uma câmara M3M é monocromática por trás de um único filtro, cada banda recebe a sua própria exposição. Uma câmara M3C cobre todas as suas três bandas com uma única exposição do sensor.

## Cadeias de caracteres de modelo e nomenclatura

Cada câmara armazena a sua identidade no GenICam `DeviceUserID` como uma cadeia de caracteres de modelo:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros exibe-a com o prefixo `LATT-` (por exemplo, `LATT-M3M-L87-F450`). A mesma cadeia «`LATT-…`» é gravada na etiqueta EXIF «`Model`» de cada exportação e é utilizada como nome da pasta de saída da câmara nos projetos processados.

| Componente | Valores                                                   | Significado                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Cor Bayer / monocromático                                                                          |
| Lente      | `L41` / `L87`                                            | O número corresponde ao **campo de visão horizontal em graus**: L41 = estreito (41°), L87 = amplo (87°)    |
| Filtro    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ou `F<nm>` (M3M) | Consulte [Filtros e bandas espectrais](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

A cadeia de caracteres do modelo determina tudo o que se segue: Chloros determina o perfil do sensor, a disposição das bandas e a calibração de fábrica a partir de `DeviceUserID` + `DeviceSerialNumber`. Não há nada a configurar por câmara — consulte [Ligar câmaras](connecting.md).

## Filtros e bandas

Os centros das bandas, as bordas do FWHM e o catálogo completo de 23 SKUs da M3M são especificações do produto, pelo que constam no manual do hardware: [**Filtros e bandas espectrais**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

O que importa do ponto de vista do software: o código do filtro na cadeia de caracteres do modelo determina quais os produtos que o Chloros pode criar. As câmaras com filtro RGB (`FRGB`) emitem apenas produtos sem debayering e de pré-visualização — a radianância e a refletância por banda não são significativas para um sensor de banda larga, pelo que o Chloros as ignora e indica isso mesmo. Todos os outros filtros produzem a cadeia completa radiação → refletância → índice.

## Calibração radiométrica em resumo

Todas as câmaras LATTICE são calibradas individualmente na fábrica de acordo com uma cadeia rastreável pelo NIST e são fornecidas com um certificado específico para cada câmara. O que isso abrange, como é medido e a precisão que pode indicar encontram-se no manual do hardware: [**Calibração Radiométrica de Fábrica**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

No que diz respeito ao software, o importante é que o Chloros determina a calibração correta quando uma câmara é ligada e fixa os coeficientes aplicados em todas as exportações — consulte [Ligar câmaras](connecting.md).

## Neste capítulo

* [Ligar câmaras](connecting.md) — deteção automática, a caixa de diálogo de ligação da GUI, equivalentes a CLI/SDK e como a calibração de fábrica é determinada (pacote na própria câmara vs. nuvem) quando uma câmara se liga.

Outros tópicos do LATTICE — definições da câmara e controlo em tempo real, modos de captura, matrizes multicâmara e processamento mono (M3M) e índices — são abordados nas respetivas secções deste manual, e a lista completa de comandos encontra-se na [Referência CLI](../reference/cli-reference.md) e na [Referência do SDK](../reference/sdk-reference.md).
