---
description: Esta página lista alguns índices multiespectrais que o Cloros usa
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Fórmulas de índice multiespectral

As fórmulas de índice abaixo usam uma combinação de faixas médias de transmissão do filtro Survey3:

 <table><thead><tr><th align="center">Cor do filtro Survey3</th><th width="196.199951171875" align="center">Nome do filtro Survey3</th><th width="159.800048828125" align="center">Faixa de transmissão (FWHM)</th><th align="center">Média Transmissão</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483nm</td><td align="center">475nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512nm</td><td align="center">494nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558nm</td><td align="center">547nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640nm</td><td align="center">619nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735nm</td><td align="center">724nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848nm</td><td alinhar="center">823nm</td></tr><tr><td alinhar="center">NIR2</td><td alinhar="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865nm</td><td align="center">850nm</td></tr></tbody></table> 

Quando estas fórmulas são utilizadas o nome pode terminar em "\_1" ou "\_2", que corresponde a qual filtro NIR, NIR1 ou NIR2 foi utilizado.

***

## EVI - Índice de Vegetação Aprimorado

Este índice foi desenvolvido originalmente para uso com dados MODIS como uma melhoria em relação ao NDVI, otimizando o sinal de crescimento em áreas de alto índice de área foliar (IAF). É mais útil em regiões com alto IAF onde o NDVI pode saturar. Ele usa uma região de refletância azul para corrigir sinais de fundo do solo e reduzir influências atmosféricas, incluindo dispersão de aerossóis.

$$
EVI = 2,5 * {(NIR - Vermelho) \over (NIR + 6 * Vermelho - 7,5 * Azul + 1)}
$$

Os valores de EVI devem variar de 0 a 1 para pixels de vegetação. Características características, como nuvens e edifícios brancos, juntamente com características escuras, como água, podem resultar em valores de pixel anômalos em uma imagem EVI. Antes de criar uma imagem EVI, você deve mascarar nuvens e recursos estendidos de imagem de refletância e, opcionalmente, limitar os valores de pixel de 0 a 1.

_Referência: Huete, A., et al. "Visão geral do desempenho radiométrico e biofísico dos índices de urbanização MODIS." Sensoriamento Remoto do Meio Ambiente 83 (2002):195–213._

***

## FCI1 - Índice de Cobertura Florestal 1

Este índice distingue copas de florestas de outros tipos de vegetação usando imagens de refletância multiespectral que incluem uma faixa vermelha na borda.

$$
FCI1 = Vermelho * Borda Vermelha
$$

As áreas florestadas apresentam valores mais baixos de FCI1 devido à menor refletância das árvores e à presença de sombras dentro da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. "Índices robustos de cobertura florestal para imagens multiespectrais." Engenharia Fotogramétrica e Sensoriamento Remoto 84.8 (2018): 505-512._

***

## FCI2 - Índice de Cobertura Florestal 2

Este índice distingue copas de florestas de outros tipos de vegetação usando imagens de refletância multiespectral que não incluem uma faixa vermelha na borda.

$$
FCI2 = Vermelho * NIR
$$

As áreas florestadas apresentam valores mais baixos de FCI2 devido à menor refletância das árvores e à presença de sombras dentro da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. "Índices robustos de cobertura florestal para imagens multiespectrais." Engenharia Fotogramétrica e Sensoriamento Remoto 84.8 (2018): 505-512._

***

## GEMI - Índice Global de Monitoramento Ambiental

Este índice de crescimento não linear é usado para monitoramento ambiental global a partir de imagens de satélite e tenta corrigir os efeitos atmosféricos. É semelhante ao NDVI, mas é menos sensível aos efeitos atmosféricos. É afetado por solo descoberto; Portanto, não é recomendado para uso em áreas de vegetação esparsa ou moderadamente densa.

$$
GEMI = eta (1 - 0,25 * eta) - {Vermelho - 0,125 \over 1 - Vermelho}
$$

Onde:

$$
eta = {2(NIR^{2}-Vermelho^{2}) + 1,5 * NIR + 0,5 * Vermelho \over NIR + Vermelho + 0,5}
$$

_Referência: Pinty, B. e M. Verstraete. GEMI: um índice não linear para monitorar a vegetação global a partir de satélites. Vegetação 101 (1992): 15-20._

***

## GARI - Índice Verde de Resistência Atmosférica

Este índice é mais sensível a uma ampla gama de concentrações de clorofila e menos sensível aos efeitos atmosféricos do que o NDVI.

$$
GARI = {NIR - [Verde - \gamma(Azul - Vermelho)] \over NIR + [Verde - \gamma(Azul - Vermelho)] }
$$

A gama constante é uma função de ponderação que depende das condições do aerossol na atmosfera. ENVI usa um valor de 1,7, que é o valor recomendado por Gitelson, Kaufman e Merzylak (1996, página 296).

_Referência: Gitelson, A., Y. Kaufman e M. Merzylak. "Uso de um Canal Verde em Sensoriamento Remoto de Vegetação Global do EOS-MODIS." Sensoriamento Remoto do Meio Ambiente 58 (1996): 289-298._

***

## GCI - Índice de Clorofila Verde

Este índice é usado para estimar o conteúdo de clorofila nas folhas em uma ampla variedade de espécies de plantas.

$$
GCI = {NIR \sobre Verde} - 1
$$

Ter amplos comprimentos de onda NIR e verdes fornecem uma melhor previsão do conteúdo de clorofila, ao mesmo tempo que permite mais sensibilidade e uma relação sinal-ruído mais alto.

_Referência: Gitelson, A., Y. Gritz e M. Merzlyak. "Relações entre o conteúdo de clorofila foliar e a refletância espectral e algoritmos para avaliação não destrutiva de clorofila nas folhas superiores das plantas." Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Índice de Folhas Verdes

Este índice foi projetado originalmente para uso com uma câmera digital RGB para medir a cobertura do trigo, onde os números digitais (DNs) vermelhos, verdes e azuis variam de 0 a 255.

$$
GLI = {(Verde - Vermelho) + (Verde - Azul) \over (2 * Verde) + Vermelho + Azul }
$$

Os valores de GLI variam de -1 a +1. Valores negativos representam características do solo e não vivas, enquanto valores positivos representam folhas e caules verdes.

_Referência: Louhaichi, M., M. Borman e D. Johnson. "Plataforma espacialmente localizada e fotografia aérea para documentação dos impactos do pastoreio no trigo." Geocarto Internacional 16, No. 1 (2001): 65-70._

***

## GNDVI - Índice de Vegetação por Diferença Normalizada Verde

Este índice é semelhante ao NDVI, exceto que mede o espectro verde de 540 a 570 nm em vez do espectro vermelho. Este índice é mais sensível à concentração de clorofila do que o NDVI.

$$
GNDVI = {(NIR - Verde) \over (NIR + Verde) }
$$

_Referência: Gitelson, A. e M. Merzlyak. "Detecção remota da concentração de clorofila nas folhas superiores das plantas." Avanços na Pesquisa Espacial 22 (1998): 689-692._

***

## GOSAVI - Índice de Vegetação Ajustado ao Solo Otimizado Verde

Este índice foi projetado originalmente com fotografia infravermelha colorida para prever as necessidades de nitrogênio para o milho. É semelhante ao OSAVI, mas substitui a faixa verde pela vermelha.

$$
GOSAVI = {NIR - Verde \sobre NIR + Verde + 0,16) }
$$

_Referência: Sripada, R., et al. "Determinando as necessidades de nitrogênio na estação para milho usando fotografia aérea infravermelha colorida." Ph.D. dissertação, Universidade Estadual da Carolina do Norte, 2005._

***

## GRVI - Índice de Vegetação Razão Verde

Este índice é sensível às taxas fotossintéticas nas copas das florestas, uma vez que as refletâncias verdes e vermelhas são fortemente influenciadas pelas mudanças nos pigmentos das folhas.

$$
GRVI = {NIR \sobre Verde }
$$

_Referência: Sripada, R., et al. "Fotografia aérea infravermelha colorida para determinar as necessidades de nitrogênio no milho no início da temporada." Revista de Agronomia 98 (2006): 968-977._

***

## GSAVI - Índice de Vegetação Ajustado ao Solo Verde

Este índice foi projetado originalmente com fotografia infravermelha colorida para prever as necessidades de nitrogren para o milho. É semelhante ao SAVI, mas substitui a faixa verde pela vermelha.

$$
GSAVI = 1,5 * {(NIR - Verde) \over (NIR + Verde + 0,5) }
$$

_Referência: Sripada, R., et al. "Determinando as necessidades de nitrogênio na estação para milho usando fotografia aérea infravermelha colorida." Ph.D. dissertação, Universidade Estadual da Carolina do Norte, 2005._

***

## IAF - Índice de Área Foliar

Este índice é usado para estimar a cobertura vegetal e prever o crescimento e o rendimento das culturas. A ENVI calcula o LAI verde utilizando a seguinte fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0,118
$$

Onde está o EVI:

$$
EVI = 2,5 * {(NIR - Vermelho) \over (NIR + 6 * Vermelho - 7,5 * Azul + 1)}
$$

Valores altos de LAI normalmente variam de aproximadamente 0 a 3,5. No entanto, quando a cena contém nuvens e outras características extraordinárias que produzem pixels saturados, os valores de LAI podem ultrapassar 3,5. Idealmente, você deve mascarar nuvens e recursos aprimorados de sua cena antes de criar uma imagem LAI.

_Referência: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. "Dados multiespectrais aerotransportados para quantificação do índice de área foliar, concentração de nitrogênio e eficiência fotossintética na agricultura." Sensoriamento Remoto do Meio Ambiente 81, não. 2-3 (2002): 179-193._

***

## LCI - Índice de Clorofila Foliar

Este índice é utilizado para estimar o conteúdo de clorofila em plantas superiores, sensível à variação na refletância causada pela absorção de clorofila.

$$
LCI = {NIR2 - RedEdge \sobre NIR2 + Vermelho}
$$

_Referência: Datt, B. "Sensorização Remota do Conteúdo de Água em Folhas de Eucalipto." Jornal de Fisiologia Vegetal 154, não. 1 (1999): 30-36._

***

## MNLI - Índice Não Linear Modificado

Este índice é um aprimoramento do Índice Não Linear (NLI) que incorpora o Índice de Vegetação Ajustado ao Solo (SAVI) para levar em conta o contexto do solo. ENVI usa um valor de fator de ajuste de fundo do dossel (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Vermelho) * (1 + L) \over (NIR^{2} + Vermelho + L) }
$$

_Referência: Yang, Z., P. Willis e R. Mueller. "Impacto da imagem AWIFS aprimorado com proporção de banda na precisão da classificação do corte." Anais do Simpósio de Sensoriamento Remoto Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Índice de Vegetação 2 Ajustado ao Solo Modificado

Este índice é uma versão mais simples do índice MSAVI proposto por Qi, et al (1994), que melhora o Índice de Vegetação Ajustado ao Solo (SAVI). Reduz o ruído do solo e aumenta a faixa dinâmica do sinal da vegetação. O MSAVI2 é baseado em um método indutivo que não utiliza um valor _L_ constante (como no SAVI) para destacar as florestas densas.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Vermelho)} \mais de 2}
$$

_Referência: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. "Um Índice de Vegetação Modificado e Ajustado ao Solo." Sensoriamento Remoto do Meio Ambiente 48 (1994): 119-126._

***

## NDRE- Diferença Normalizada RedEdge

Este índice é semelhante ao NDVI, mas compara o contraste entre o NIR com o RedEdge em vez do Red, que muitas vezes detecta o estresse do vegetação mais cedo.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge }
$$

***

## NDVI - Índice de Vegetação por Diferença Normalizada

Este índice é uma medida de vegetação verde e saudável. A combinação de sua formulação de diferença normalizada e o uso das regiões de maior absorção e refletância da clorofila o tornam robusto em uma ampla gama de condições. Pode, no entanto, saturar as condições de vegetação densa quando o IAF se torna elevado.

$$
NDVI = {NIR - Vermelho \sobre NIR + Vermelho }
$$

O valor deste índice varia de -1 a 1. O intervalo comum para vegetação verde é de 0,2 a 0,8.

_Referência: Rouse, J., R. Haas, J. Schell e D. Deering. Monitoramento de Sistemas Vegetais nas Grandes Planícies com ERTS. Terceiro Simpósio ERTS, NASA (1973): 309-317._

***

## NLI - Índice Não Linear

Este índice assume que a relação entre muitos índices de vegetação e parâmetros biofísicos de superfície não é linear. Ele lineariza relações com interferências de superfícies que tendem a não ser lineares.

$$
NLI = {NIR^{2} - Vermelho \sobre NIR^{2} + Vermelho }
$$

_Referência: Goel, N. e W. Qin. "Influências da arquitetura do dossel nas relações entre vários índices de vegetação e LAI e Fpar: uma simulação computacional." Revisões de Sensoriamento Remoto 10 (1994): 309-347._

***

## OSAVI - Índice de Vegetação Ajustado ao Solo Otimizado

Este índice é baseado no Índice de Vegetação Ajustado ao Solo (SAVI). Ele usa um valor padrão de 0,16 para o fator de ajuste de fundo do dossel. Rondeaux (1996) determinou que este valor proporciona maior variação do que o SAVI para baixa cobertura vegetal, ao mesmo tempo que demonstra maior sensibilidade à cobertura vegetal superior a 50%. Este índice é melhor utilizado em áreas com vegetação relativamente esparsa, onde o solo é visível através da copa.

$$
OSAVI = {(NIR - Vermelho) \over (NIR + Vermelho + 0,16) }
$$

_Referência: Rondeaux, G., M. Steven e F. Baret. "Otimização de Índices de Vegetação Ajustados ao Solo." Sensoriamento Remoto do Meio Ambiente 55 (1996): 95-107._

***

## RDVI - Índice de Vegetação por Diferença Renormalizada

Este índice utiliza a diferença entre os comprimentos de onda do infravermelho próximo e do vermelho, juntamente com o NDVI, para destacar a vegetação saudável. É insensível aos efeitos da geometria de visualização do solo e do sol.

$$
RDVI = {(NIR- Vermelho) \over \sqrt{(NIR + Vermelho)} }
$$

_Referência: Roujean, J. e F. Breon. "Estimando PAR absorvido pela vegetação a partir de profundidade de refletância bidirecional." Sensoriamento Remoto do Meio Ambiente 51 (1995): 375-384._

***

## SAVI - Índice de Vegetação Ajustado ao Solo

Este índice é semelhante ao NDVI, mas suprime os efeitos dos pixels do solo. Ele utiliza um fator de ajuste de fundo da copa, _L_, que é uma função da densidade da vegetação e muitas vezes requer conhecimento prévio da quantidade de vegetação. Huete (1988) sugere um valor ótimo de _L_=0,5 para levar em conta as variações de fundo do solo de primeira ordem. Este índice é melhor utilizado em áreas com vegetação relativamente esparsa, onde o solo é visível através da copa.

$$
SAVI = {1,5 * (NIR- Vermelho) \over (NIR + Vermelho + 0,5) }
$$

_Referência: Huete, A. "Um Índice de Vegetação Ajustado ao Solo (SAVI)." Sensoriamento Remoto do Meio Ambiente 25 (1988): 295-309._

***

## TDVI - Índice de Vegetação por Diferença Transformada

Este índice é útil para monitorar cobertura vegetal em ambientes urbanos. Não saturado como NDVI e SAVI.

$$
TDVI = 1,5 * {(NIR- Vermelho) \over \sqrt{NIR^{2} + Vermelho + 0,5} }
$$

_Referência: Bannari, A., H. Asalhi e P. Teillet. "Índice de Vegetação por Diferença Transformada (TDVI) para Mapeamento de Cobertura Vegetal" Em Anais do Simpósio de Geociências e Sensoriamento Remoto, IGARSS '02, IEEE International, Volume 5 (2002)._

***

## VARI - Índice Visível de Resistência Atmosférica

Este índice é baseado no ARVI e é utilizado para estimar uma fração de vegetação em uma cena com baixa sensibilidade aos efeitos atmosféricos.

$$
VARI = {Verde - Vermelho \sobre Verde + Vermelho - Azul }
$$

_Referência: Gitelson, A., et al. "Linhas de vegetação e solo no espaço espectral visível: um conceito e técnica para estimativa remota da fração de vegetação. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Índice de Vegetação da Ampla Faixa Dinâmica

Este índice é semelhante ao NDVI, mas utiliza um coeficiente de ponderação (_a_) para reduzir a disparidade entre as contribuições dos sinais infravermelhos próximos e vermelhos para o NDVI. O WDRVI é particularmente eficaz em cenas com densidade de vegetação moderada a alta quando o NDVI excede 0,6. O NDVI tende a se estabilizar quando a fração de vegetação e o índice de área foliar (IAF) aumentam, enquanto o WDRVI é mais sensível a uma gama mais ampla de frações de vegetação e as alterações no IAF.

$$
WDRVI = {(\alpha * NIR- Vermelho) \over (\alpha * NIR + Vermelho)}
$$

O coeficiente de ponderação (_a_) pode variar de 0,1 a 0,2. Um valor de 0,2 é recomendado por Henebry, Viña e Gitelson (2004).

_Referências_

_Gitelson, A. "Índice de Vegetação de Ampla Faixa Dinâmica para Quantificação Remota de Características Biofísicas da Vegetação." Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. "O Índice de Vegetação de Ampla Faixa Dinâmica e sua Utilidade Potencial para Análise de Lacunas." Boletim de análise de lacunas 12: 50-56._