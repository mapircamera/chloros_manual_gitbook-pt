# Ambiente de Teste de Índices/LUT

O Ambiente de Teste de Índices/LUT é um espaço de trabalho interativo no Visualizador de Imagens Chloros que permite experimentar cálculos de índices multiespectrais e visualizações a cores em tempo real. Esta poderosa ferramenta ajuda-o a testar diferentes índices, refinar intervalos de valores e criar visualizações prontas para publicação sem ter de reprocessar todo o seu conjunto de dados.

## O que é a Caixa de Areia de Índice/LUT?

### Objetivo

A Caixa de Areia oferece:

* **Cálculo de índice em tempo real** - Aplique qualquer índice de vegetação instantaneamente
* **Ajuste interativo de LUT** - Ajuste com precisão os gradientes e intervalos de cor
* **Otimização do fluxo de trabalho** - Determine as melhores configurações antes do processamento em lote

### Sandbox vs. Processamento de projeto

**Sandbox de Índices/LUT (Interativa):**

* Uma imagem de cada vez
* Feedback instantâneo
* Experimental e iterativa
* Sem alterações permanentes nos ficheiros
* Perfeita para explorar e testar

**Processamento de projeto (em lote):**

* Conjunto de dados completo de uma só vez
* Configurações pré-definidas
* Ficheiros de saída permanentes
* Demorado
* Ideal quando as configurações estão finalizadas

{% hint style="success" %}
**Melhor fluxo de trabalho**: Utilize a Sandbox para experimentar e encontrar as configurações ideais de índice e LUT e, em seguida, aplique essas configurações durante o Processamento de projeto para todo o seu conjunto de dados.
{% endhint %}

***

## Trabalhar com a Sandbox de Índice/LUT

### Compreender os Índices Pré-calculados

No Chloros, os índices podem ser aplicados durante o processamento do projeto. Para determinar quais definições de índice e LUT pretende aplicar às exportações, o mais fácil é utilizar a sandbox do visualizador de imagens.

A sandbox permite-lhe:

* **Aplicar novos índices e gradientes de cor (LUTs)** para visualizar os dados
* **Ajustar as definições de visualização** de forma interativa
* **Visualizar** imagens de índice já calculadas
* **Inspecionar** valores de píxeis em todos os níveis de zoom

### Abrir a área de teste

A área de teste de Índice/LUT é acedida no separador da barra lateral do **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Clique numa imagem na grelha de imagens do navegador de ficheiros; esta abre-se no separador **Visualizador de Imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Clique na guia**Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir a barra lateral pop-out à esquerda, caso ainda não esteja aberta

### Selecionar uma imagem para aplicar um Índice/LUT

Para trabalhar com um índice na <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Abra uma imagem** da grelha de imagens principal clicando nela
2. A guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> será então aberta
3. Clique no **menu suspenso Camadas** (canto superior direito do visualizador)
4. Selecione a camada no menu suspenso:
   * RAW (Refletância)

### Aplicar um Índice a uma Imagem

Assim que a imagem estiver em ecrã inteiro e a barra lateral do **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Marque a caixa Índice na parte superior da barra lateral
2. Escolha o filtro da sua câmara no menu suspenso à esquerda
3. Escolha a fórmula de índice desejada no menu suspenso à direita
4. Arraste os círculos de cor do canal do filtro para as posições na fórmula de índice abaixo
5. Assim que a fórmula for válida, a imagem será atualizada e mostrará os valores do índice
6. Mova o cursor do rato para ver os valores na localização do cursor
7. Aumente o zoom para ver os píxeis individuais e os seus valores associados

Cada índice tem um intervalo de valores e um significado específicos:

#### Exemplo NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Para obter a documentação completa sobre fórmulas de índice, consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

***

## Trabalhar com LUTs (Tabelas de Consulta)

### O que é uma LUT?

Uma **Tabela de Consulta (LUT)** mapeia valores numéricos de índice para cores para visualização:

* **Entrada**: Valor do pixel do índice (por exemplo, NDVI 0,65)
* **Saída**: Cor RGB (por exemplo, verde brilhante)
* **Objetivo**: Tornar os padrões mais fáceis de ver e interpretar**LUT em escala de cinzentos vs. LUT a cores:**

* Escala de cinzentos: Científica e neutra, mostra dados brutos
* LUT a cores: Intuitiva e impactante, destaca padrões e diferenças

{% hint style="success" %}
**Capacidade de visualização**: A aplicação de uma LUT a cores a uma imagem de índice em escala de cinzentos torna significativamente mais fácil identificar padrões, anomalias e áreas de interesse num relance.
{% endhint %}

### Aplicar uma LUT a uma imagem de índice

Depois de ter uma imagem de índice a mostrar

1. Clique no <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> botão &quot;+Adicionar LUT&quot;
2. Selecione o gradiente de cor
3. Ajuste os pontos finais mínimo/máximo do recorte
4. Ajuste o Modo de Recorte
5. Marque a caixa Índice na barra lateral do **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral da guia para aplicar a LUT

### Escolher um gradiente de cor

**Selecionar um gradiente:**

1. No painel LUT, localize a**barra de gradiente colorida**

2. Passe o rato por cima dela para ver as predefinições de gradiente disponíveis
3. Selecione o gradiente desejado
4. A imagem **atualiza-se imediatamente** com novas cores quando a caixa Índice está marcada

{% hint style="success" %}
**Melhores práticas**: Para índices de vegetação como o NDVI, o gradiente Red-Amarelo-Green é o mais intuitivo, pois está alinhado com associações de cores naturais (verde = saudável, amarelo = moderado, vermelho = estressado).
{% endhint %}

### Ajustar classes de cor

O **controlo Classes**determina quantos níveis de cor distintos aparecem no seu gradiente:**Opções de contagem de classes:*** **2-5 classes**: Categorias muito amplas, zonas distintas
* **6-10 classes**: Equilibrado, bom para classificação
* **11-20 classes**: Gradientes suaves, aparência contínua
* **Mais de 20 classes**: Quase contínuo, suavidade máxima**Como ajustar:**

1. No painel LUT, localize os**quadrados de amostras de cor abaixo da barra de gradiente**

2. Ajuste o número de classes adicionando com o botão +
3. Remova o número de classes clicando duas vezes numa amostra de cor
4. O gradiente atualiza-se **em tempo real** na imagem**Efeito na visualização:*** **Menos classes** (3-5): Cria zonas distintas, classificação simplificada, categorias mais fáceis de distinguir
* **Número médio de classes** (6-10): Abordagem equilibrada, adequada para a maioria das aplicações
* **Mais classes** (15-20): Transições suaves, variação detalhada, aparência fotográfica**Quando utilizar:*** **Poucas classes (3-5)**: Slides de apresentação, mapas de classificação, relatórios simples
* **Número médio de classes (6-10)**: Análise geral, detalhe equilibrado, relatórios padrão
* **Muitas classes (15-20)**: Análise científica, inspeção detalhada, resultados com qualidade de publicação

### Ajuste fino dos intervalos de valores

Os **controlos de intervalo de valores**determinam quais os valores de índice que correspondem a quais cores no seu gradiente:**Controlos de intervalo no painel LUT:*** **Valor mínimo**: Limite inferior da escala de cores
* **Valor máximo**: Limite superior da escala de cores
* **Valores intermédios**: Distribuídos automaticamente entre o mínimo e o máximo (com base na contagem de classes)

#### Ajustar valores mínimo/máximo

**Para ajustar intervalos de valores:**

1. No painel LUT, localize os campos de entrada**Valor mínimo**e**Valor máximo**

2. Clique no campo**Valor mínimo**

3. Digite o valor mínimo desejado (por exemplo, `0.2`)
4. Prima **Enter** ou clique fora do campo
5. Repita para o campo **Valor Máx.** (por exemplo, `0.9`)
6. A visualização **atualiza-se imediatamente**{% hint style="info" %}**Escalonamento automático**: Quando aplica uma LUT pela primeira vez, o Chloros define automaticamente o mínimo/máximo para o intervalo de dados real na imagem. Pode então restringir este intervalo para se concentrar em intervalos de valores específicos de interesse.
{% endhint %}

**Exemplo de ajustes de intervalo do NDVI:*** **Intervalo completo**: `-1.0` a `1.0` (mostrar todos os valores possíveis)
* **Focado na vegetação**: `0.2` a `0.9` (excluir solo nu e água)
* **Apenas vegetação saudável**: `0.5` a `0.9` (destacar apenas plantas vigorosas)
* **Detecção de stress**: `0.2` a `0.5` (enfatizar áreas problemáticas)
* **Intervalo personalizado**: Ajuste com base nos valores de pixel observados**Porquê ajustar os intervalos?*** **Aumentar o contraste** na sua área de interesse
* **Excluir valores irrelevantes** (por exemplo, massas de água, solo nu)
* **Padronizar a visualização** em várias imagens ou datas
* **Enfatizar diferenças subtis** dentro de um intervalo de valores estreito

### Recortar valores fora do intervalo

Quando os valores dos pixels ficam fora do intervalo mínimo/máximo definido, pode controlar a forma como são apresentados utilizando **modos de recorte**.

#### **Opções de modos de recorte disponíveis:**

#### 1. Mínimo e Máximo

* Pixels **abaixo do mínimo**→ apresentar utilizando a**primeira cor** do gradiente (por exemplo, vermelho)
* Pixéis **acima do máximo**→ exibidos utilizando a**última cor** do gradiente (por exemplo, verde)
* **Caso de utilização**: Enfatizar extremos, mostrar o intervalo completo de dados com cores saturadas nos limites
* **Exemplo**: Os valores NDVI abaixo de 0,2 aparecem todos a vermelho, os valores acima de 0,9 aparecem todos a verde

#### 2. Fundo transparente

* Os píxeis **fora do intervalo**tornam-se**totalmente transparentes*** Apenas os píxeis **dentro do intervalo** mostram o gradiente de cores
* **Caso de utilização**: sobreposição GIS, isolar intervalos de valores específicos, destacar apenas áreas de interesse
* **Exemplo**: Mostrar apenas os valores de NDVI entre 0,4 e 0,7 a cores, tudo o resto transparente

{% hint style="warning" %}
**Limitação de transparência**: Os píxeis transparentes aparecerão com a cor de fundo no visualizador. Quando exportados durante o processamento, a transparência é preservada no formato PNG, mas não em JPG.
{% endhint %}

#### 3. Fundo de Índice

* Os píxeis **fora do intervalo**são apresentados em**escala de cinzentos** (mostrando valores de índice brutos)
* Os píxeis **dentro do intervalo**apresentam**gradiente de cor*** **Caso de utilização**: Destaque subtil, mantém o contexto enquanto enfatiza áreas de interesse
* **Exemplo**: Destaque a vegetação sob stress com cores (NDVI 0,3-0,5) enquanto mostra as áreas saudáveis a cinzento

#### 4. Fundo original

* Os píxeis **fora do intervalo**exibem a**imagem multiespectral original*** Os píxeis **dentro do intervalo**mostram um**gradiente de cor*** **Caso de uso**: O mais intuitivo - combina o contexto natural da imagem com uma sobreposição analítica de cores
* **Exemplo**: Veja a aparência real do campo/cultura com áreas de stress codificadas por cores sobrepostas

### Escolhendo o modo de recorte certo

| Modo de recorte              | Ideal para                                   | Estilo de visualização          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Mínimo e Máximo**    | Exibição completa dos dados, análise científica     | Todos os pixels coloridos           |
| **Fundo Transparente** | Sobreposições GIS, isolando intervalos específicos    | Cor no intervalo, em branco fora dele |
| **Fundo de índice**       | Ênfase subtil, mantendo o contexto dos dados  | Cor no intervalo, cinzento fora dele  |
| **Fundo original**    | Relatórios, apresentações, análise intuitiva | Cor no intervalo, foto fora dele |

### Criar cores LUT personalizadas

Para um controlo total sobre a sua visualização, pode criar **gradientes de cor personalizados** editando pontos de cor individuais.**Para criar um gradiente personalizado:**

1. No painel LUT, localize a**barra de pré-visualização do gradiente**

2. Procure os**quadrados de amostras de cor** abaixo do gradiente
3. **Clique num ponto de cor** para o selecionar
4. Abre-se um **seletor de cores**

5. Escolha uma nova cor utilizando:
   * **Roda de cores**: Seleção visual de cores
   * **Controles deslizantes RGB/HSV**: Controlo preciso da cor
   * **Introdução de código hexadecimal**: Especificação exata da cor (por exemplo, `#FF0000` para vermelho)
6. Clique fora do seletor de cores **para aplicar a nova cor**

7. O gradiente**atualiza-se imediatamente** na imagem**Adicionar ou remover pontos de cor:*** **Adicionar um ponto**: Clique no ícone + para adicionar uma nova amostra no final
* **Remover um ponto**: Clique duas vezes no quadrado de cor para remover a amostra**Estratégias de personalização:*** **Inverter gradiente**: Inverta a ordem das cores para inverter o significado (por exemplo, verde = baixo, vermelho = alto)
* **Cores da marca**: Combine com a paleta de cores da sua organização para relatórios
* **Adequado para daltónicos**: Use combinações de laranja-azul ou roxo-amarelo
* **Otimização de impressão**: Escolha cores que funcionem tanto na impressão a cores como em tons de cinzento
* **Limiares múltiplos**: Utilize cores distintas em limiares de valores específicos para classificação

{% hint style="info" %}
**Guardar gradientes personalizados**: Os gradientes personalizados podem ser guardados e reutilizados. Clique no ícone de guardar no painel LUT para preservar os seus esquemas de cores personalizados para utilização futura.
{% endhint %}

***

## Fluxo de trabalho interativo

### Atualizações em tempo real

Todos os ajustes de LUT na área de teste atualizam a imagem **instantaneamente e de forma interativa**:

* **Mudar camada** → A imagem muda imediatamente
* **Selecionar gradiente** → As cores atualizam-se instantaneamente
* **Ajustar intervalo de valores** → O contraste muda em tempo real
* **Alterar classes** → A suavidade do gradiente atualiza-se imediatamente
* **Modificar recorte** → A exibição do fundo muda instantaneamente
* **Editar cores** → O gradiente personalizado aplica-se imediatamente**Não é necessário o botão «Aplicar»** - todas as alterações são em tempo real e interativas!

{% hint style="success" %}
**Feedback em tempo real**: O feedback visual instantâneo permite-lhe experimentar rapidamente diferentes configurações até encontrar a visualização ideal para as suas necessidades de análise.
{% endhint %}

### Fluxo de trabalho de refinamento iterativo

**Fluxo de trabalho típico de otimização de LUT:**

1.**Selecione a camada de índice** (por exemplo, RAW (Refletância))
2. **Aplique o índice** - Escolha o filtro da câmara e a fórmula do índice, arraste os círculos coloridos para a localização apropriada na fórmula do índice
3. **Aplique o gradiente LUT** - Comece com a predefinição Red-Yellow-Green
4. **Inspecione os valores dos píxeis** - Mova o cursor, observe os intervalos de valores
5. **Ajustar mínimo/máximo** - Restrinja para se concentrar na vegetação (por exemplo, 0,2 a 0,9)
6. **Escolher recorte** - Experimente «Original Background» para contextualização
7. **Aperfeiçoar cores** - Personalize o gradiente, se necessário, para ênfase específica
8. **Finalizar configurações**- Registe as configurações e copie para as Configurações do Projeto para processamento de exportação

### Inspeção dos valores dos pixels

Compreender os valores reais dos píxeis é crucial para definir intervalos de LUT eficazes:**Como inspecionar valores:**

1. Os valores dos píxeis são exibidos quando a imagem tem a caixa**Índice**, ou ambas as caixas**Índice**e**LUT**,**marcadas**.
2. **Mova o cursor** sobre diferentes áreas da imagem
3. **Observe os valores dos píxeis** exibidos na legenda ao passar o cursor
4. Amplie a imagem para ver píxeis individuais destacados com um valor flutuante
5. **Tome notas** dos intervalos de valores para diferentes características:
   * **Vegetação saudável**: por exemplo, NDVI 0,55-0,85
   * **Vegetação em estado de stress**: por exemplo, NDVI 0,30-0,50
   * **Solo nu**: por exemplo, NDVI 0,05-0,25
   * **Água** (se presente): por exemplo, NDVI -0,05 a 0,10**Utilização dos valores dos píxeis para definir intervalos de LUT:**Após inspecionar os valores dos pixels, ajuste os valores mínimo/máximo da LUT em conformidade:**Exemplo de cenário:*** **Observação**: Valores do solo = 0,05-0,25, Em estresse = 0,25-0,50, Saudável = 0,50-0,85
* **Objetivo**: Visualizar apenas a saúde das plantas (excluir o solo)
* **Configurações da LUT**: Mín. = `0.25`, Máx. = `0.85`
* **Recorte**: «Fundo original» para ver o solo na cor natural
* **Resultado**: O gradiente de cor aplica-se apenas à vegetação; o solo é apresentado como na imagem original

{% hint style="info" %}
**Gama dinâmica**: Diferentes culturas, estações do ano e fases de crescimento terão gamas de valores diferentes. Verifique sempre os valores dos píxeis no seu conjunto de dados específico antes de definir as gamas LUT.
{% endhint %}

***

## Índices personalizados (Chloros+)

### Criação de fórmulas de índices personalizados

{% hint style="info" %}
**Onde criar**: Os índices personalizados podem ser configurados nas**Definições do projeto** antes do processamento, bem como na barra lateral da área de teste do Visualizador de imagens.
{% endhint %}

**Para criar um índice personalizado:**

1.**Abra as **Configurações do projeto** (antes do processamento) ou a barra lateral da área de teste do Visualizador de imagens
2. Navegue até ao **menu suspenso Fórmula do índice**

3. Procure a opção**&quot;Personalizado&quot;** (é necessário estar com sessão iniciada com uma licença Chloros+)
4. **Defina a sua fórmula** utilizando variáveis de banda:
   * Nomes das bandas: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operadores: `+`, `-`, `*`, `/`, `^` (exponente)
   * Funções: `sqrt()`, `abs()`, etc. (se suportadas)
   * Parênteses: `()` para a ordem das operações
5. **Dê um nome ao seu índice** (por exemplo, «MyIndex» ou «CustomNDVI»)
6. **Guarde a configuração**

**Exemplos de fórmulas personalizadas:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Validação da fórmula**: Certifique-se de que a sua fórmula utiliza bandas disponíveis na sua câmara. Por exemplo, RedEdge só está disponível em câmaras com um filtro RedEdge.
{% endhint %}

***

## Próximos passos

Agora que já compreende o Index/LUT Sandbox:

* **Aplicar ao processamento**: Utilize as definições descobertas em [Definições do projeto](../project-settings/project-settings.md)
* **Processamento em lote**: Aplique índices otimizados a conjuntos de dados completos
* **Saiba mais**: Leia [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md)

Documentação relacionada:

* [**Camadas de Imagem**](image-layers.md) - Gestão e visualização de camadas
* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) - Noções básicas do Visualizador de Imagens
* [**Processamento de imagens (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Fluxo de trabalho completo de processamento
