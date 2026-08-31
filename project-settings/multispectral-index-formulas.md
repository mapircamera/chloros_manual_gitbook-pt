---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Fórmulas de índices multiespectrais

As fórmulas de índices abaixo utilizam uma combinação dos intervalos de transmissão média do filtro Survey3:

<table><thead><tr><th align="center">Cor do filtro Survey3</th><th width="196.199951171875" align="center">Survey3 Nome do filtro</th><th width="159.800048828125" align="center">Intervalo de transmissão (FWHM)</th><th align="center">Transmissão média</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Quando estas fórmulas são utilizadas, o nome pode terminar em «\_1» ou «\_2», o que corresponde ao facto de ter sido utilizado o filtro NIR, o NIR1 ou o NIR2.

Para as câmaras LATTICE M3C (passa-banda triplo da Bayer), o mesmo motor de indexação utiliza as bandas do filtro M3C:

| Filtro M3C | Banda 1 (centro/FWHM) | Banda 2 (centro/FWHM) | Banda 3 (centro/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

As câmaras LATTICE M3M são de banda única (um filtro de banda estreita por câmara), pelo que os índices multibanda não são calculados para uma imagem M3M isolada. Para calcular índices com o M3M, combine duas ou mais câmaras numa pilha multibanda alinhada e utilize o motor de índices LATTICE (`chloros-cli lattice index` ou a Calculadora de Índices em tempo real da interface gráfica).

***

## Onde cada nome de índice funciona

O Chloros tem **três** superfícies de índice, e as suas listas predefinidas não são idênticas. Utilize esta secção para verificar se um nome funcionará onde pretende utilizá-lo.

| Onde se encontra | Qual a lista aplicável | Contagem |
| --- | --- | --- |
| Definições do projeto → Índice → Adicionar índice (GUI) | Superfície 1 | 27 |
| Visualizador de Imagens [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Superfície 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Superfície 2 | 22 |
| SDK `process_folder(indices=[...])` | Superfície 2 | 22 |
| `chloros-cli lattice index --preset` | Superfície 3 | 22 (um 22 diferente) |
| Separador «Câmaras» – Calculadora de Índice em Tempo Real | Superfície 3 | 22 (um 22 diferente) |

As Superfícies 1 e 2 funcionam com **uma imagem de cada vez, proveniente de uma câmara**, utilizando os slots de símbolos `x`/`y`/`z`(/`a`) associados aos canais de filtro dessa câmara. A superfície 3 funciona com uma**pilha multibanda alinhada** — várias câmaras LATTICE co-registadas num único cubo — e refere-se aos canais pelo nome em minúsculas.

### 1. Definições do projeto na GUI / Menu suspenso da área de teste do Visualizador de Imagens — 27 fórmulas

O menu suspenso apresenta-as nesta ordem (é a ordem de inserção, não alfabética):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

Na GUI, arrasta os canais de filtro da tua câmara para as ranhuras de banda da fórmula, pelo que qualquer fórmula pode ser utilizada com qualquer atribuição de banda que a tua câmara suporte. As fórmulas personalizadas que guardaste são acrescentadas abaixo desta lista.

As **cinco fórmulas exclusivas da GUI** — aquelas que a lista CLI/SDK `--indices` não aceita — são implementadas da seguinte forma:

| Predefinição exclusiva da GUI | Fórmula (conforme implementada) | Espaços |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (quatro slots) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

O mapeamento pretendido para cada um é apresentado na respetiva secção mais abaixo nesta página (por exemplo, o GARI espera x=Green, y=NIR, z=Blue, a=Red). GARI é a única fórmula em Chloros que utiliza um quarto espaço.

### 2. Expansão de nomes de CLI / SDK `--indices` — 22 predefinições

A opção `chloros-cli process --indices` (e o parâmetro SDK `indices`) aceita estes nomes de predefinições:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Os nomes de índices desconhecidos são ignorados silenciosamente.** Um nome fora desta lista (incluindo as cinco fórmulas exclusivas da GUIexclusivas da GUI: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` e qualquer fórmula personalizada que tenha guardado na GUI) é omitido com apenas um aviso no registo — a execução continua sem esse índice e a própria execução continua a ser considerada bem-sucedida. O aviso é apresentado da seguinte forma:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Os nomes são comparados sem distinção entre maiúsculas e minúsculas após a remoção de espaços em branco; assim, `ndvi`, `NDVI` e ` NDVI ` correspondem à mesma predefinição. Uma predefinição também é ignorada se necessitar de uma banda que o filtro da sua câmara não forneça.
{% endhint %}

As fórmulas exatas tal como implementadas (os símbolos `x`/`y`/`z` são slots de banda; o mapeamento predefinido é apresentado por predefinição):

| Predefinição | Fórmula (conforme implementada) | Filtro predefinido | Espaços (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Como um nome de predefinição se transforma em posições de banda

Quando se passa um nome simples, como `NDVI`, o Chloros tem de decidir qual o canal de qual ficheiro que cada símbolo lê. Utiliza esta tabela, que mapeia um código de filtro para a posição na matriz de cada canal:

| Código do filtro | Canal → índice da matriz |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` é aceite como um alias para Orange, também 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

O **filtro predefinido** da predefinição (a coluna «Filtro predefinido» acima) é utilizado quando o projeto contém imagens com esse filtro. Caso contrário, o Chloros analisa os filtros efetivamente presentes no projeto, na ordem `RGN, OCN, NGB, RGB, RE, NIR`, e seleciona o primeiro que consiga fornecer todos os canais de que a predefinição necessita. Se nenhum o conseguir, a predefinição é descartada para essa execução. É por isso que o `NDVI` solicitado num conjunto de dados composto apenas por OCN ainda produz um resultado razoável — ele vincula-se às posições do OCNposições Orange e NIR.

As cadeias de caracteres do modelo LATTICE M3C transportam o filtro com um prefixo `F` (`LATT-M3C-L41-FRGN`), mas o prefixo é omitido quando o código do filtro é lido a partir da imagem; por isso, uma câmara FRGN resolve através da linha `RGN` acima e não necessita de tratamento especial.

### 3. Motor de indexação LATTICE (`lattice index --preset`, Calculadora de Índice em tempo real) — 22 predefinições

O motor LATTICE funciona com pilhas multibanda alinhadas (matrizes em tempo real ou ficheiros TIFF multibanda exportados) e utiliza nomes de canais em minúsculas (`red`, `green`, `blue`, `red_edge`, `nir`). A sua lista de predefinições difere das duas anteriores:

| Predefinição | Fórmula | Canais |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | vermelho, NIR |
| GNDVI | `(nir - green) / (nir + green)` | verde, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | azul, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | vermelho\_borda, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | azul, verde, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | vermelho, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | vermelho, infravermelho |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | vermelho, infravermelho |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | azul, vermelho, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | vermelho, NIR |
| CVI | `(nir / green) - (red / green)` | vermelho, verde, infravermelho |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | vermelho, infravermelho |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | vermelho, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | vermelho, verde, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | vermelho, verde, azul |
| NGRDI | `(green - red) / (green + red)` | vermelho, verde |
| VARI | `(green - red) / (green + red - blue)` | vermelho, verde, azul |
| TGI | `green - 0.39*red - 0.61*blue` | vermelho, verde, azul |
| EXG | `2*green - red - blue` | vermelho, verde, azul |
| CIRE | `(nir / red_edge) - 1` | vermelho\_borda, infravermelho |
| CIGREEN | `(nir / green) - 1` | verde, nir |
| NDWI | `(green - nir) / (green + nir)` | verde, nir |

Execute o comando `chloros-cli lattice index --list-presets` para imprimir esta tabela a partir da sua compilação instalada e o comando `--list-gradients` para ver os gradientes de cor disponíveis. Os símbolos dos canais distinguem maiúsculas de minúsculas e devem corresponder aos nomes em minúsculas da predefinição (por exemplo, `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Tal como implementado na GUI e na lista de predefinições CLI/SDK, o CVI é a fórmula da «razão das razões»:

$$
CVI = {(z / y) \over (x / y)}
$$

com o mapeamento de canais predefinido RGB x=Red, y=Green e z=Blue. Na interface gráfica, pode arrastar qualquer um dos canais da sua câmara para as ranhuras x/y/z. Note que a predefinição `CVI` do motor de índices LATTICE utiliza uma fórmula diferente, `(NIR / Green) - (Red / Green)` — consulte as tabelas acima para a superfície que está a utilizar.

***

## ENDVI - Índice de Vegetação por Diferença Normalizada Melhorado

Este índice utiliza o canal azul, além do NIR e do verde, e é popular em câmaras com filtro NGB, nas quais a banda azul substitui a vermelha.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

A implementação é a fórmula de símbolos `((x+y)-(2*z))/((x+y)+(2*z))` — atribua os canais NIR e Green da sua câmara aos slots x/y e Blue ao z (para uma câmara NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Índice de Vegetação Melhorado

Este índice foi originalmente desenvolvido para ser utilizado com dados MODIS como uma melhoria em relação ao NDVI, através da otimização do sinal de vegetação em áreas com elevado índice de área foliar (LAI). É mais útil em regiões com valores elevados de LAI, onde o NDVI pode saturar. Utiliza a região de refletância azul para corrigir os sinais de fundo do solo e para reduzir as influências atmosféricas, incluindo a dispersão de aerossóis.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Os valores de EVI devem variar entre 0 e 1 para os píxeis de vegetação. Elementos brilhantes, como nuvens e edifícios brancos, juntamente com elementos escuros, como a água, podem resultar em valores de píxeis anómalos numa imagem EVI. Antes de criar uma imagem EVI, deve mascarar as nuvens e os elementos brilhantes da imagem de refletância, e, opcionalmente, aplicar um limiar aos valores dos píxeis entre 0 e 1.

_Referência: Huete, A., et al. «Visão geral do desempenho radiométrico e biofísico dos índices de vegetação MODIS.» Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Índice de Cobertura Florestal 1

_Apenas na GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice distingue as copas das árvores de outros tipos de vegetação utilizando imagens de refletância multiespectral que incluem uma banda de «borda vermelha».

$$
FCI1 = Red * RedEdge
$$

As áreas florestais apresentarão valores mais baixos de FCI1 devido à menor refletância das árvores e à presença de sombras no interior da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. «Índices robustos de cobertura florestal para imagens multiespectrais.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Índice de Cobertura Florestal 2

_Apenas na GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice distingue as copas das árvores de outros tipos de vegetação utilizando imagens de refletância multiespectral que não incluem uma banda de «red edge».

$$
FCI2 = Red * NIR
$$

As áreas florestais apresentarão valores mais baixos de FCI2 devido à menor refletância das árvores e à presença de sombras no interior do dossel.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. «Índices robustos de cobertura florestal para imagens multiespectrais.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Índice de Monitorização Ambiental Global

_Apenas GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice de vegetação não linear é utilizado para a monitorização ambiental global a partir de imagens de satélite e procura corrigir os efeitos atmosféricos. É semelhante ao NDVI, mas é menos sensível aos efeitos atmosféricos. É afetado pelo solo nu; por conseguinte, não é recomendado para utilização em áreas com vegetação esparsa ou moderadamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Onde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referência: Pinty, B., e M. Verstraete. GEMI: um índice não linear para monitorizar a vegetação global a partir de satélites. Vegetation 101 (1992): 15-20._

***

## GARI - Green Índice resistente a efeitos atmosféricos

_Apenas na GUI — não está disponível como predefinição CLI/SDK `--indices`._

Este índice é mais sensível a uma ampla gama de concentrações de clorofila e menos sensível aos efeitos atmosféricos do que o NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A constante gama é uma função de ponderação que depende das condições dos aerossóis na atmosfera. O ENVI utiliza um valor de 1,7, que é o valor recomendado por Gitelson, Kaufman e Merzylak (1996, página 296).

_Referência: Gitelson, A., Y. Kaufman e M. Merzylak. «Utilização de um canal Green na teledeteção da vegetação global a partir do EOS-MODIS.» Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Índice de clorofila Green

Este índice é utilizado para estimar o teor de clorofila nas folhas numa vasta gama de espécies vegetais.

$$
GCI = {NIR \over Green} - 1
$$

A disponibilidade de comprimentos de onda largos no NIR e no verde proporciona uma melhor previsão do teor de clorofila, ao mesmo tempo que permite maior sensibilidade e uma relação sinal-ruído mais elevada.

_Referência: Gitelson, A., Y. Gritz e M. Merzlyak. «Relações entre o teor de clorofila foliar e a refletância espectral e algoritmos para a avaliação não destrutiva da clorofila em folhas de plantas superiores.» Journal of Plant Physiology 160 (2003): 271-282._

***

## Índice foliar GLI - Green

Este índice foi originalmente concebido para ser utilizado com uma câmara digital RGB para medir a cobertura de trigo, em que os valores digitais (DNs) do vermelho, verde e azul variam entre 0 e 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Os valores do GLI variam entre -1 e +1. Os valores negativos representam o solo e elementos não vivos, enquanto os valores positivos representam folhas verdes e caules.

_Referência: Louhaichi, M., M. Borman e D. Johnson. «Plataforma com localização espacial e fotografia aérea para documentação dos impactos do pastoreio no trigo.» Geocarto International 16, n.º 1 (2001): 65-70._

***

## GNDVI - Green Índice de Vegetação por Diferença Normalizada

Este índice é semelhante ao NDVI, exceto que mede o espectro verde de 540 a 570 nm em vez do espectro vermelho. Este índice é mais sensível à concentração de clorofila do que o NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referência: Gitelson, A., e M. Merzlyak. «Detecção Remota da Concentração de Clorofila nas Folhas de Plantas Superiores.» Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Índice de Vegetação Ajustado ao Solo Otimizado

Este índice foi originalmente concebido com fotografia a cores no infravermelho para prever as necessidades de azoto do milho. É semelhante ao OSAVI, mas substitui a banda verde pela vermelha.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referência: Sripada, R., et al. «Determinação das necessidades de azoto do milho durante a época de cultivo utilizando fotografia aérea a cores no infravermelho». Tese de doutoramento, Universidade Estadual da Carolina do Norte, 2005._

***

## Índice de Vegetação da Relação GRVI - Green

Este índice é sensível às taxas de fotossíntese nas copas das árvores, uma vez que as refletâncias do verde e do vermelho são fortemente influenciadas pelas alterações nos pigmentos foliares.

$$
GRVI = {NIR \over Green }
$$

_Referência: Sripada, R., et al. «Fotografia aérea a cores no infravermelho para determinar as necessidades de azoto no início da época no milho.» Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Índice de Vegetação Ajustado ao Solo

Este índice foi originalmente concebido com fotografia a cores no infravermelho para prever as necessidades de azoto do milho. É semelhante ao SAVI, mas substitui a banda verde pela vermelha.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referência: Sripada, R., et al. «Determinação das necessidades de azoto durante a época de cultivo do milho utilizando fotografia aérea a infravermelho colorido.» Tese de doutoramento, Universidade Estadual da Carolina do Norte, 2005._

***

## LAI - Índice de Área Folhosa

Este índice é utilizado para estimar a cobertura foliar e para prever o crescimento e o rendimento das culturas. O ENVI calcula o LAI verde utilizando a seguinte fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Onde EVI é:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Os valores elevados de LAI variam normalmente entre aproximadamente 0 e 3,5. No entanto, quando a cena contém nuvens e outros elementos brilhantes que produzem píxeis saturados, os valores de LAI podem exceder 3,5. O ideal é mascarar as nuvens e os elementos brilhantes da sua cena antes de criar uma imagem LAI.

_Referência: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. «Dados multiespectrais aéreos para quantificar o índice de área foliar, a concentração de azoto e a eficiência fotossintética na agricultura.» Remote Sensing of Environment 81, n.º 2-3 (2002): 179-193._

***

## LCI - Índice de clorofila foliar

_Apenas na GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice é utilizado para estimar o teor de clorofila em plantas superiores, sendo sensível à variação da refletância causada pela absorção da clorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referência: Datt, B. «Remote Sensing of Water Content in Eucalyptus Leaves.» Journal of Plant Physiology 154, n.º 1 (1999): 30-36._

***

## MNLI - Índice Não-Linear Modificado

Este índice é um aperfeiçoamento do Índice Não Linear (NLI) que incorpora o Índice de Vegetação Ajustado ao Solo (SAVI) para ter em conta o fundo do solo. O ENVI utiliza um valor de 0,5 para o fator de ajuste do fundo da copa (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referência: Yang, Z., P. Willis e R. Mueller. «Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.» Atas do Simpósio de Teledeteção Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Índice de Vegetação Ajustado ao Solo Modificado 2

Este índice é uma versão mais simples do índice MSAVI proposto por Qi et al. (1994), que representa uma melhoria em relação ao Índice de Vegetação Ajustado ao Solo (SAVI). Reduz o ruído do solo e aumenta a gama dinâmica do sinal da vegetação. O MSAVI2 baseia-se num método indutivo que não utiliza um valor constante de _L_ (como no SAVI) para destacar a vegetação saudável.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referência: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. «A Modified Soil Adjusted Vegetation Index.» Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Rácio Simples Modificado

Este índice é uma modificação da razão simples NIR/Red, concebida para linearizar a sua relação com parâmetros biofísicos, e é mais sensível do que o NDVI em densidades de vegetação mais elevadas.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referência: Chen, J. «Avaliação de índices de vegetação e de uma razão simples modificada para aplicações boreais». Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE — Diferença Normalizada RedEdge

Este índice é semelhante ao NDVI, mas compara o contraste entre o NIR e o RedEdge em vez de Red, o que muitas vezes permite detetar o stress da vegetação mais cedo.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Índice de Vegetação por Diferença Normalizada

Este índice é uma medida de vegetação verde e saudável. A combinação da sua formulação de diferença normalizada com a utilização das regiões de maior absorção e refletância da clorofila torna-o robusto numa ampla gama de condições. No entanto, pode saturar-se em condições de vegetação densa quando o valor do LAI se torna elevado.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

O valor deste índice varia entre -1 e 1. O intervalo habitual para a vegetação verde situa-se entre 0,2 e 0,8.

_Referência: Rouse, J., R. Haas, J. Schell e D. Deering. Monitorização de sistemas de vegetação nas Grandes Planícies com o ERTS. Terceiro Simpósio ERTS, NASA (1973): 309-317._

***

## NLI - Índice Não Linear

Este índice parte do princípio de que a relação entre muitos índices de vegetação e os parâmetros biofísicos da superfície é não linear. Ele lineariza as relações com parâmetros da superfície que tendem a ser não lineares.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referência: Goel, N. e W. Qin. «Influências da arquitetura do dossel nas relações entre vários índices de vegetação e o LAI e o Fpar: uma simulação computacional.» Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Índice de Vegetação Ajustado ao Solo Otimizado

Este índice baseia-se no Índice de Vegetação Ajustado ao Solo (SAVI). Utiliza um valor padrão de 0,16 para o fator de ajuste de fundo da copa. Rondeaux (1996) determinou que este valor proporciona uma maior variação do solo do que o SAVI para coberturas vegetais baixas, ao mesmo tempo que demonstra uma maior sensibilidade a coberturas vegetais superiores a 50%. Este índice é mais adequado para utilização em áreas com vegetação relativamente escassa, onde o solo é visível através da copa das árvores.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referência: Rondeaux, G., M. Steven e F. Baret. «Optimization of Soil-Adjusted Vegetation Indices.» Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Índice de Vegetação por Diferença Renormalizada

Este índice utiliza a diferença entre os comprimentos de onda do infravermelho próximo e do vermelho, juntamente com o NDVI, para destacar a vegetação saudável. É insensível aos efeitos do solo e da geometria de observação solar.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referência: Roujean, J. e F. Breon. «Estimativa da PAR absorvida pela vegetação a partir de medições de refletância bidirecional.» Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Índice de Vegetação Ajustado ao Solo

Este índice é semelhante ao NDVI, mas suprime os efeitos dos píxeis de solo. Utiliza um fator de ajuste de fundo da copa, _L_, que é uma função da densidade da vegetação e requer frequentemente conhecimento prévio das quantidades de vegetação. Huete (1988) sugere um valor ótimo de _L_=0,5 para ter em conta as variações de fundo do solo de primeira ordem. Este índice é mais adequado para áreas com vegetação relativamente esparsa, onde o solo é visível através da copa das árvores.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referência: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI)». Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Índice de Vegetação por Diferença Transformada

Este índice é útil para monitorizar a cobertura vegetal em ambientes urbanos. Não sofre saturação, ao contrário do NDVI e do SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referência: Bannari, A., H. Asalhi e P. Teillet. «Índice de Vegetação por Diferença Transformada (TDVI) para o Mapeamento da Cobertura Vegetal» em Atas do Simpósio de Geociências e Teledeteção, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Índice Visível Resistente a Efeitos Atmosféricos

Este índice baseia-se no ARVI e é utilizado para estimar a fração de vegetação numa cena com baixa sensibilidade aos efeitos atmosféricos.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referência: Gitelson, A., et al. «Linhas de vegetação e solo no espaço espectral visível: um conceito e uma técnica para a estimativa remota da fração de vegetação». International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Índice de vegetação de ampla gama dinâmica

Este índice é semelhante ao NDVI, mas utiliza um coeficiente de ponderação (_a_) para reduzir a disparidade entre as contribuições dos sinais do infravermelho próximo e do vermelho para o NDVI. O WDRVI é particularmente eficaz em cenas com densidade de vegetação moderada aalta, quando o NDVI excede 0,6. O NDVI tende a estabilizar-se à medida que a fração de vegetação e o índice de área foliar (LAI) aumentam, enquanto o WDRVI é mais sensível a uma gama mais ampla de frações de vegetação e a variações no LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

O coeficiente de ponderação (_a_) pode variar entre 0,1 e 0,2. Henebry, Viña e Gitelson (2004) recomendam um valor de 0,2.

_Referências_

_Gitelson, A. «Índice de Vegetação de Ampla Gama Dinâmica para a Quantificação Remota das Características Biofísicas da Vegetação». Journal of Plant Physiology 161, n.º 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. «O Índice de Vegetação de Ampla Gama Dinâmica e a sua Utilidade Potencial para a Análise de Lacunas.» Gap Analysis Bulletin 12: 50-56._
