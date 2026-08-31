# Área de teste de Índice/LUT

A Área de teste de Índice/LUT é o espaço de trabalho interativo na barra lateral do Visualizador de Imagens Chloros. Basta escolher uma fórmula, associar os canais da sua câmara à mesma, colorir com um gradiente e ajustar o intervalo de valores — e a imagem é atualizada em tempo real enquanto o faz. Desde a versão 1.2.0, também pode **guardar o que criou**, quer para uma imagem, quer para todo o projeto, sem necessidade de reprocessamento.

## Para que serve a Sandbox

| Sandbox de Índice/LUT (interativa)        | Processamento do projeto (em lote)       |
| -------------------------------------- | -------------------------------- |
| Uma imagem de cada vez, feedback instantâneo  | Todo o conjunto de dados numa única execução     |
| Experimental e iterativo             | Configurações pré-definidas          |
| Rende em tempo real; guarda apenas quando o utilizador solicitar  | Grava sempre os ficheiros finais      |
| Perfeito para encontrar as definições certas | Ideal quando as definições já estão definidas |

{% hint style="success" %}
**O fluxo de trabalho habitual**: ajuste na Sandbox até que a visualização corresponda ao que pretende; depois, exporte diretamente da Sandbox ou copie as mesmas definições de índice e LUT para [Definições do Projeto](../project-settings/project-settings.md), para que a próxima execução do processamento as aplique a todas as imagens.
{% endhint %}

***

## Abrir o Sandbox

1. Clique numa imagem na grelha — esta abre-se em ecrã inteiro no separador **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Clique no ícone do **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir a barra lateral esquerda, caso ainda não esteja aberta
3. Escolha uma camada multibanda no menu suspenso de camadas no canto superior direito — **RAW (Refletância)** é a escolha habitual, porque os valores de índice calculados com base na refletância calibrada são comparáveis entre imagens

A barra lateral mostra, de cima para baixo:

* o nome da imagem e o modelo da câmara
* o botão **Exportar/Guardar Imagem(ns)** — aparece assim que a opção Índice ou LUT for marcada
* as caixas de seleção **Índice**e**LUT**
* o painel de configuração do índice
* o painel **Valores do Cursor** com a leitura, o histograma e o controlo GSD

{% hint style="warning" %}
**Não disponível para câmaras monocromáticas.** Numa imagem LATTICE M3M de banda única, ambas as caixas de seleção estão desativadas, com a dica de ferramenta _«Não disponível para sensores monocromáticos (M3M)»_ — um índice multibanda não está definido numa única banda. Para calcular índices a partir de câmaras M3M, combine duas ou mais numa pilha multibanda alinhada e utilize o motor de índices LATTICE.
{% endhint %}

***

## Aplicar um índice

1. Assinale a caixa **Índice** na parte superior da barra lateral
2. Escolha o filtro da sua câmara no menu suspenso à esquerda (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Selecione uma fórmula de índice no menu suspenso à direita — 27 fórmulas integradas, além de quaisquer fórmulas personalizadas que tenha guardado
4. A fórmula é apresentada como expressão matemática abaixo, com um círculo vazio em cada espaço de banda. **Arraste um círculo de canal colorido para um espaço** para o associar
5. Assim que todos os espaços utilizados pela fórmula estiverem associados, a imagem atualiza-se e mostra os valores do índice
6. Passe o cursor sobre a imagem para ler os valores; o painel **Valores do Cursor** adiciona uma linha de índice com o valor sob o cursor

Clique duas vezes num espaço associado para o limpar. Uma fórmula incompleta é um estado normal durante o arrastar, não um erro — a imagem simplesmente não é atualizada até que a fórmula esteja completa.

Os círculos de canal estão codificados por cores: vermelho = Red, verde = Green, azul = Blue, laranja = Orange, ciano = Cyan, roxo = NIR, magenta = RE. As mesmas cores são utilizadas para os pontos dos canais e as curvas do histograma no painel «Valores do cursor».

### Exemplo do NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Para consultar a referência completa às fórmulas — as três listas de predefinições e quais os nomes que funcionam em cada caso — consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

### Com a opção «Índice» marcada, mas sem LUT

A imagem é representada em **escala de cinzentos**, esticada entre os dois valores de limiar. Isto é intencional: a imagem de índice é constituída por dados escalares, e a escala de cinzentos é a sua representação fiel. Adicione uma LUT quando pretender cor.***

## Trabalhar com LUTs (Tabelas de Consulta)

Uma **Tabela de Consulta** mapeia valores de índice para cores: introduza NDVI 0,65 e obtém-se um verde específico. Não altera os dados — altera a forma como estes são interpretados.

### Adicionar uma LUT

1. Clique no botão **«+ Adicionar LUT»**, <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">, abaixo da fórmula
2. Escolha um gradiente de cor
3. Defina o mínimo e o máximo de recorte
4. Escolha um Modo de Recorte
5. Assinale a caixa **LUT** na barra lateral para a renderizar

A caixa de seleção **LUT** permanece desativada até que uma LUT tenha sido efetivamente configurada no índice.

### Escolher um gradiente de cor

Passe o cursor sobre a **barra de gradiente**para abrir a lista de predefinições — o Chloros inclui**sete** predefinições de gradiente:

| # | Gradiente                            | Forma                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Amarelo → Green (**predefinição**)  | Divergente — corresponde à intuição habitual sobre a vegetação: verde = saudável |
| 2 | Roxo → Amarelo → Green             | Divergente, com uma extremidade inferior distinta                                  |
| 3 | Castanho → Branco → Blue                | Divergente em torno de um ponto médio claro                                   |
| 4 | Preto → Roxo → Rosa → Amarelo pálido | Sequencial, do escuro para o claro                                           |
| 5 | Red → Amarelo → Blue                 | Divergente em torno de um ponto médio claro                                   |
| 6 | Roxo → Blue → Green → Amarelo      | Sequencial, do escuro para o claro                                           |
| 7 | Orange → Branco → Roxo             | Divergente em torno de um ponto médio claro                                   |

Um gradiente **divergente**coloca uma cor neutra no meio da janela, o que funciona bem quando o ponto médio tem um significado específico (um limiar, uma data de referência). Um gradiente**sequencial** varia monotonamente do escuro para o claro, o que funciona bem para uma quantidade que apenas tem «mais» e «menos».

Cada predefinição tem sete pontos de cor. Clique numa predefinição e a imagem atualiza-se imediatamente (quando a caixa LUT estiver marcada).

### Editar os pontos de cor

Por baixo da barra de gradiente encontra-se uma linha de amostras de cor, uma por ponto:

* **Alterar uma cor**: clica numa amostra para abrir o seletor de cores (roda de cores, controles deslizantes RGB/HSV ou um código hexadecimal, como `#FF0000`)
* **Adicionar um ponto**: clique no botão**+** no final da linha — é adicionado um ponto branco
* **Remover um ponto**:**clique duas vezes** na amostra
* **Manter um gradiente editado**: clique no ícone de guardar ao lado da barra de gradiente para adicionar o seu gradiente editado à lista de predefinições, para que possa selecioná-lo novamente

O gradiente que configurou num índice é guardado juntamente com esse índice nas definições do projeto, pelo que permanece mesmo após fechar e reabrir o projeto.

**Menos pontos**produzem zonas distintas que se interpretam como uma classificação;**mais pontos** produzem transições suaves, quase fotográficas. Três a cinco pontos são adequados para slides de apresentação e mapas de classificação; seis a dez são adequados para análise geral; quinze ou mais são adequados para inspeção detalhada e figuras de publicação.

### Definir o intervalo de valores

O controlo de limiar é um **deslizador com duas alças**que varia entre −1 e +1, com uma caixa de texto editável em cada extremidade para valores exatos e um botão**AUTO**.

* Arraste qualquer uma das alças ou digite um número na respetiva caixa e prima Enter
* **AUTO**define o intervalo para o**

2.º e 98.º percentis** dos valores de índice válidos da imagem — um bom ponto de partida que ignora valores atípicos. O Chloros arredonda o resultado de forma adaptativa, para 4 casas decimais num intervalo muito estreito, 3 para um intervalo estreito e 2 nos restantes casos
* Qualquer ajuste manual tem precedência sobre a opção **AUTO**até premir**AUTO** novamente

Exemplo das janelas do NDVI:

| Objetivo                                    | Mín.  | Máx. |
| --------------------------------------- | ---- | --- |
| Mostrar tudo                            | −1,0 | 1,0 |
| Apenas vegetação, excluir solo e água | 0,2  | 0,9 |
| Apenas vegetação saudável                 | 0,5  | 0,9 |
| Enfatizar o stress                        | 0,2  | 0,5 |

Restringir a janela aumenta o contraste dentro da sua área de interesse e empurra tudo o resto para fora do intervalo — onde o **Modo de Recorte** decide o que acontece a esses dados.***

## Modos de recorte

Quando o valor do índice de um píxel fica fora da janela mínimo/máximo, o Modo de Recorte decide como este é representado.

| Rótulo do menu suspenso                  | Valor armazenado      | Os píxeis fora do intervalo são desenhados como                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Mínimo e Máximo** (predefinição) | `clip`            | A cor mais próxima da extremidade do gradiente — os valores abaixo do mínimo assumem a primeira cor, os valores acima do máximo assumem a última |
| **Fundo transparente**      | `transparent`     | Totalmente transparente (alfa real)                                                                                                  |
| **Fundo indexado**| `indexColor`      | Escala de cinzentos, estendida por toda a**plena** gama de índices da imagem, pelo que a estrutura fora da gama continua visível a cinzento                |
| **Fundo original**         | `backgroundColor` | A própria imagem subjacente, de modo que a sobreposição de cor fica por cima da cena real                                                |

| Modo                       | Ideal para                               | Aspecto                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Mínimo e Máximo**      | Apresentação completa dos dados, análise científica | Todos os píxeis coloridos                      |
| **Fundo Transparente** | Sobreposições SIG, isolando uma faixa de valores   | Cor no interior da janela, nada no exterior |
| **Fundo indexado**       | Ênfase mantendo o contexto dos dados    | Cor no interior, cinzento no exterior               |
| **Fundo original**    | Relatórios e apresentações              | Cor no interior, fotografia no exterior         |

{% hint style="info" %}
**Os píxeis sem dados são sempre transparentes, em todos os modos.** Um pixel cujo índice não seja finito (uma divisão por 0) ou seja exatamente −1,0 ou +1,0 (sentinelas de saturação, resultantes de uma banda apresentar valor zero enquanto a outra não) é tratado como «sem dados» em vez de como um valor extremo. Isto mantém as luzes sobre-expostas e as sombras perdidas fora da sua escala de cores, em vez de as representar como a leitura mais extrema no quadro. A mesma regra define quais os píxeis que alimentam os limiares AUTO e o histograma de índices, para que os três estejam em concordância.
{% endhint %}

A transparência é preservada quando a exportação é gravada como PNG. Não pode ser representada em JPG.

***

## Ler valores enquanto ajusta

O painel **Valores do cursor**, abaixo do painel de configuração, é o instrumento de medição do Sandbox:

* Mova o cursor sobre a imagem e leia os valores de origem por canal, bem como o valor do índice na sua própria linha
* Ative o botão **ÍNDICE** acima do histograma para ver a distribuição dos valores de índice no fotograma, com os seus dois limiares de recorte representados como linhas tracejadas laranja e o valor do cursor como uma linha branca — esta é a forma mais rápida de escolher uma janela que contenha efetivamente os seus dados
* Ative **CURSOR** para ver linhas de marcação nos valores sob o ponteiro
* Aumente o zoom para além de 60× (menos se estiver definido um tamanho de bloco GSD) para destacar os píxeis individuais exibidos com um valor flutuante

Uma rotina prática:

1. Tome nota dos valores sobre vegetação saudável, vegetação sob stress, solo nu e água
2. Observe onde esses agrupamentos se situam no histograma do índice
3. Defina os valores mínimo e máximo para delimitar o agrupamento que lhe interessa
4. Escolha um modo de recorte — _Original Background_ mantém a cena visível à sua volta

***

## Exportar a partir da Sandbox

Tudo o que foi descrito acima é uma pré-visualização em tempo real até que o guarde. O botão **Exportar/Guardar Imagem(ns)** na parte superior da barra lateral abre um painel que desliza sobre a barra lateral (em vez de cobrir a imagem, para que possa continuar a ver o que está a avaliar).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opções

| Opção                          | Efeito                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aplicar à imagem atual**      | Guarda exatamente a imagem apresentada, com estas definições                                                                                                |
| **Aplicar a todas as imagens do projeto** | Reexecuta a configuração idêntica em todas as imagens do projeto. As imagens sem as bandas necessárias para este índice são ignoradas, não sendo consideradas erros |
| **Barra de gradiente de índice/LUT**      | Também grava uma imagem de legenda separada por exportação, com o intervalo de valores identificado                                                                     |
| **Histograma de índice**             | Também grava uma imagem de histograma separada por exportação, mostrando os valores mínimo e máximo dos dados e os limiares de recorte                                               |

Se o **tamanho do bloco GSD** no separador da imagem for superior a 1, o painel avisa-o antes de confirmar: a exportação guarda o que está a ver, incluindo a média por bloco. Defina primeiro o controlo GSD de volta para 1 se quiser a resolução total.

### Para onde vão os ficheiros

Cada clique em **Exportar**aloca uma**nova pasta, que nunca é reutilizada**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Exemplos: `Sandbox_Exports/NDVI_LUT_001/`, seguido de `Sandbox_Exports/NDVI_LUT_002/` para a próxima execução. A numeração é derivada da análise do que já se encontra no disco, pelo que permanece mesmo após reinicializações e pastas que elimine manualmente. Nada é nunca sobrescrito — o objetivo da Sandbox é comparar uma tentativa com a anterior.

Dentro da pasta, por imagem:

| Ficheiro                                                   | Conteúdo                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | A imagem renderizada, pixel a pixel tal como o visualizador a apresentou |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | O ficheiro auxiliar da barra de gradiente, se solicitado                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | O ficheiro auxiliar do histograma de índices, se solicitado                  |

Os dois ficheiros complementares são sempre gravados em **resolução total**, mesmo quando a imagem principal é calculada com média por bloco: o tamanho do bloco corresponde à resolução do ecrã, e ambos os ficheiros complementares apresentam os valores reais do índice por pixel. Além disso, apresentam mais informação do que as versões no ecrã — ambas indicam a janela de alongamento _e_ os valores mínimos e máximos reais dos dados, pelo que uma legenda guardada continua a ser legível meses mais tarde, mesmo sem o projeto aberto.

### Progresso e resultados

A exportação de todo o projeto demora alguns minutos, pelo que a execução apresenta relatórios através de um canal de progresso em tempo real, em vez de bloquear o sistema:

* Uma barra de progresso mostra `current / total` e o ficheiro que está a ser gravado
* Quando termina, o painel indica quantas imagens foram exportadas, quantas foram ignoradas e o caminho da pasta de saída
* As imagens ignoradas são listadas com o motivo (são mostradas até cinco, seguidas de uma linha «+N mais»). O motivo habitual é uma camada que não possui os canais de que este índice necessita
* Se **nenhuma** imagem do projeto puder utilizar o índice, a execução reporta uma falha, em vez de deixar uma pasta vazia

Apenas uma exportação em ambiente de teste pode ser executada de cada vez. Iniciar uma segunda execução enquanto outra está em curso é recusado com uma mensagem clara, em vez de permitir que duas execuções disputem o mesmo ficheiro de projeto.

### A grelha apresenta a execução

Cada execução concluída aparece como um botão próprio na barra de ferramentas [grelha de imagens](image-grid.md), com a etiqueta `<IndexName> <Index|LUT> <NNN>`. É assim que se comparam as execuções: faça duas exportações com gradientes ou limiares diferentes e, em seguida, alterne entre os dois botões na grelha.

***

## Fórmulas de índice personalizadas (Chloros+)

{% hint style="info" %}
**Onde criá-las**: na barra lateral do Sandbox ou em**Definições do projeto** antes do processamento. Ambas são gravadas na mesma lista ao nível do projeto.
{% endhint %}

1. Abra a calculadora de fórmulas personalizadas a partir do menu suspenso de fórmulas de índice (requer o início de sessão com uma subscrição Chloros+ elegível)
2. Escreva a fórmula utilizando os **símbolos de faixa e intervalo** `x`, `y`, `z`, `a`, `b`, `c` — não são nomes de bandas
3. Operadores disponíveis: `+`, `-`, `*`, `/`, `^` e `()` para agrupamento
4. Funções disponíveis: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Dê-lhe um nome e guarde-a — ela aparece na parte inferior do menu suspenso de fórmulas e pode associar-lhe os canais arrastando os círculos dos canais, exatamente como uma predefinição integrada

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**As fórmulas personalizadas estão disponíveis apenas na interface gráfica.** A opção CLI/SDK `--indices` expande os 22 nomes de predefinições integradas e ignora silenciosamente tudo o resto, incluindo as suas fórmulas personalizadas. Para processar em lote uma fórmula personalizada, configure-a nas «Definições do projeto» e execute o processamento, ou utilize a exportação «Aplicar a todas as imagens do projeto» da Sandbox.
{% endhint %}

***

## Resolução de problemas

### «Esta camada não tem os canais de que este índice necessita»

A fórmula lê uma posição de canal que a camada atual não possui — por exemplo, um índice de três slots num ficheiro de um ou dois canais. Mude para uma camada multibanda (refletância ou sem bayering) ou escolha um índice que se adapte ao filtro da sua câmara.

### «Não foi possível aceder ao backend de processamento de imagens»

O backend não está a responder. Verifique o separador «Registo»; se o backend estiver a reiniciar, o Sandbox recupera-se automaticamente assim que este estiver de volta.

### A imagem não alterou quando arrastei um círculo

A fórmula ainda não está completa. Uma fórmula incompleta é tratada como um estado normal de arrastamento — nada é renderizado e nada é reportado como erro. Preencha todos os campos utilizados pela fórmula.

### A imagem inteira está com uma única cor

A sua janela de recorte está provavelmente muito fora dos dados. Prima **AUTO**para a alinhar aos percentis 2.º/98.º, ou ative o histograma**ÍNDICE** para ver onde os dados se encontram realmente.

### As cores exportadas não correspondem ao que vi

Devem corresponder — o caminho de exportação é um espelho deliberado da pré-visualização em tempo real, incluindo o alfa do modo de recorte, e a média por bloco é aplicada _após_ a colorização, exatamente como o visualizador o faz. Se houver diferenças, verifique se o tamanho do bloco GSD não se alterou entre a visualização e a exportação.

***

## Próximos passos

* [**Camadas de imagem**](image-layers.md) — em que camada aplicar um índice e o que significam os seus valores
* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) — a leitura do cursor, o histograma e o controlo do GSD em pormenor
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — todas as predefinições, em todas as superfícies
* [**Definições do projeto**](../project-settings/project-settings.md) — integrar as definições que definiu numa execução de processamento
