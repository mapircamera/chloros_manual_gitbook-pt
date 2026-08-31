---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Câmaras compatíveis

O Chloros processa imagens de duas famílias de câmaras MAPIR em **todas as plataformas** (Windows, Linux amd64 e Linux arm64/Jetson):

* **Survey3** — câmaras Survey3W (grande angular) e Survey3N (angular estreito). Entrada: `RAW+JPG`.
* **LATTICE**— Módulos de câmara multiespectral M3C e M3M. Entrada: capturas `.tif`/`.tiff`. As câmaras LATTICE também podem ser**controladas em tempo real** a partir do Chloros — através do separador «Câmaras» da interface gráfica (Windows) ou do `chloros-cli lattice` / o Python SDK (Windows e Linux) — incluindo conjuntos sincronizados de várias câmaras. Consulte o [guia da LATTICE](lattice/).

O pipeline de processamento também aceita ficheiros de entrada `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Fabricante</th><th width="250">Modelo da câmara</th><th width="138">Modelo do filtro</th><th width="187">Tipo de imagem</th></tr></thead><tbody><tr><td><strong>Fabricante</strong></td><td><strong>Modelo da câmara</strong></td><td><strong>Modelo do filtro</strong></td><td><strong>Tipo de imagem</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

A linha LATTICE é um sistema modular de câmaras multiespectrais baseado no sensor de obturador global Sony IMX265 (3,1 MP, píxeis de 3,45 µm). Cada câmara armazena a sua identidade como uma cadeia de caracteres do modelo:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros apresenta-a com o prefixo `LATT-` (por exemplo, `LATT-M3M-L41-F550`), e a cadeia de caracteres do modelo controla tudo a jusante — o perfil do sensor, a disposição das bandas e a calibração são definidos automaticamente; não há nada a configurar por câmara. O número da lente corresponde ao **campo de visão horizontal em graus**: `L41` = estreito 41°, `L87` = amplo 87°.

Existem duas configurações de sensor:

| Configuração | Sensor      | Tipo de filtro                           | Bandas por câmara                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Cor Bayer | Triplo passa-banda                       | 3 bandas espectrais a partir de uma única exposição                                 |
| **M3M**       | Monocromático  | Filtro de interferência de banda estreita única | 1 banda calibrada — combine várias câmaras M3M para índices de vegetação |

### Opções de filtro M3C (Bayer)

| Filtro | Bandas (nome @ centro nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Catálogo de filtros M3M (mono) — 23 SKUs

O número F corresponde à referência do SKU; a banda medida (gravada em cada produto calibrado para exportação) é a análise do filtro por lote:

| SKU    | Centro (nm, medido) | Limites do FWHM (nm) | Largura (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_«Os limites da banda são medidos como valores de largura total ao meio do máximo a partir das digitalizações de filtro por lote do MAPIR — os mesmos valores que o Chloros inclui em cada exportação calibrada.»_ “— (nominal)” = ainda não foi realizada nenhuma varredura do lote; para esses SKUs, o centro indicado é o número do SKU e a largura é o valor fornecido pelo fabricante.

† «A refletância F988 é calibrada utilizando um painel de refletância na cena: a banda situa-se fora da gama calibrada do sensor de luz DAQ, pelo que o Chloros aplica a sua captura mais recente do painel e mantém-na entre as observações do painel.» Consulte [Alvos de calibração](calibration-targets.md).

Para controlo da câmara em tempo real, matrizes, configuração de rede e a cadeia de processamento radiométrico, consulte o [guia LATTICE](lattice/).
