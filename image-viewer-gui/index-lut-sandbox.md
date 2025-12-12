# Sandbox de índice/LUT

O Index/LUT Sandbox é um espaço de trabalho interativo dentro do Chloros Image Viewer que permite experimentar cálculos de índice multiespectral e visualizações de núcleos em tempo real. Essa ferramenta poderosa ajuda você a testar diferentes índices, refinar valores intermitentes e criar visualizações prontas para publicação sem reprocessar todo o conjunto de dados.

## O que é o índice/LUT Sandbox?

### Propósito

Um Sandbox fornece:

* **Cálculo de índice em tempo real** - Aplique qualquer índice de recorte instantaneamente
* **Ajuste LUT interativo** - Ajuste fino de gradientes e faixas de cores
* **Otimização do fluxo de trabalho** - Determine as melhores configurações antes do processamento em lote

### Sandbox vs. Processamento de Projeto

**Sandbox de índice/LUT (interativo):**

* Uma imagem por vez
* Comentários instantâneos
* Experimental e iterativo
* Nenhuma alteração permanente nos arquivos
* Perfeito para explorar e testar

**Processamento do projeto (lote):**

* Conjunto de dados inteiro de uma só vez
* Configurações pré-configuradas
* Arquivos de saída permanente
* Demora muito
* Melhor quando as configurações são finalizadas

{% dica estilo = "sucesso" %}
**Melhor fluxo de trabalho**: use o Sandbox para experimentar e encontrar as configurações ideais de índice e LUT e, em seguida, aplicar essas configurações durante o processamento do projeto para todo o seu conjunto de dados.
{% endhint %}

***

## Trabalhando com o Sandbox Index/LUT

### Compreendendo os índices pré-calculados

Sem cloro, os índices podem ser aplicados durante o processamento do projeto. Para determinar quais configurações de índice e LUT você deseja aplicar às exportações, é mais fácil usar um sandbox do visualizador de imagens.

A caixa de areia permite que você:

* **Aplique novos índices e gradientes de núcleos (LUTs)** para visualizar os dados
* **Ajuste as configurações de visualização** interativamente
* **Ver** imagens de índice já calculadas
* **Inspeção** valores de pixels em todos os níveis de zoom

### Abrindo a caixa de areia

O Index/LUT Sandbox é acessado no **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebar tab:

1. Clique em uma imagem na categoria de imagens do navegador de arquivos. Ela será aberta na guia **Visualizador de imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Clique na guia **Visualizador de imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir a barra lateral pop-out esquerda, caso ainda não esteja aberto

### Selecionando uma imagem para aplicar um índice/LUT

Para trabalhar com um índice no Image Viewer<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandbox:

1. **Abra uma imagem** na nota de imagens principal clicando nela
2. O guia **Visualizador de imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> será aberto
3. Clique no **menu suspenso Camada** (canto superior direito do visualizador)
4. Selecione a camada no menu suspenso:
   * RAW (Refletância)

### Aplicando um índice a uma imagem

Assim que a imagem estiver em tela cheia e o **Visualizador de imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab sidebar is open:

1. Marque a caixa Índice na parte superior da barra lateral
2. Escolha o filtro da sua câmera no menu suspenso à esquerda
3. Escolha a fórmula de índice desejada no menu suspenso à direita
4. Arraste os círculos coloridos do canal de filtro para os locais na fórmula de índice abaixo
5. Assim que a fórmula for válida, a imagem será atualizada e mostrará os valores do índice
6. Mova o cursor do mouse para ver os valores da localização do cursor
7. Aumente o zoom para ver pixels individuais e seus valores associados

Cada índice tem uma faixa de valores e significado específico:

#### Exemplo de NDVI

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

Para obter a documentação completa da fórmula de índice, consulte [Fórmulas de índice multiespectral](../project-settings/multispectral-index-formulas.md).

***

## Trabalhando com LUTs (tabelas de consulta)

### O que é uma LUT?

Uma **Tabela de consulta (LUT)** mapa de valores de índice numérico em núcleos para visualização:

* **Entrada**: valor do pixel do índice (por exemplo, NDVI 0,65)
* **Saída**: cor RGB (por exemplo, verde brilhante)
* **Objetivo**: tornar os padrões mais simples de ver e interpretar

**LUT em tons de cinza x núcleos:**

* Tons de cinza: Científico e neutro, mostra dados brutos
* Color LUT: Intuitivo e impactante, destaca padrões e diferenças

{% dica estilo = "sucesso" %}
**Poder de visualização**: A aplicação de uma LUT colorida a uma imagem de índice em escala de cinza facilita muito a identificação rápida de padrões, anomalias e áreas de interesse.
{% endhint %}

### Aplicando uma LUT a uma imagem de índice

Depois de ter uma imagem de índice

1. Clique no botão<img src="../.gitbook/assets/image.png" alt="" data-size="line"> "+Adicionar LUT"
2. Selecione o gradiente de cor
3. Ajuste dos pontos finais mínimo/máximo de recorte
4. Ajuste o modo de recorte
5. Marque a caixa Índice na barra lateral da guia **Visualizador de imagens**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para aplicar uma LUT

### Escolhendo um gradiente de cor

**Selecionando um gradiente:**

1. No painel LUT, localize a **barra de gradiente colorida**
2. Passe o mouse sobre ele para ver as predefinições de gradiente disponíveis
3. Selecione o gradiente desejado
4. A imagem **atualiza imediatamente** com novos núcleos quando a caixa Índice é marcada

{% dica estilo = "sucesso" %}
**Melhores Práticas**: Para índices de vegetação como o NDVI, o gradiente Vermelho-Amarelo-Verde é mais intuitivo porque se alinha com associações naturais de núcleos (verde=saudável, amarelo=moderado, vermelho=estressado).
{% endhint %}

### Ajustando classes de núcleos

O **controle Classes** determina quantas etapas de núcleos discretos aparecem no seu gradiente:

**Opções de contagem de turmas:**

* **2-5 turmas**: categorias muito amplas, zonas separadas
* **6-10 aulas**: Equilibrado, bom para classificação
* **Aulas 11-20**: Gradientes suaves, aparência contínua
* **Mais de 20 aulas**: necessários máximos quase contínuos

**Como ajustar:**

1. No painel LUT, localize os **quadrados de amostra de núcleos abaixo da barra de gradiente**
2. Ajuste o número de aulas adicionando o botão +
3. Remova o número de classes clicando duas vezes em uma amostra de cor
4. O gradiente é atualizado **em tempo real** na imagem

**Efeito na visualização:**

* **Menos classes** (3-5): Cria zonas separadas, classificação simplificada, categorias mais simples de distinguir
* **Aulas médias** (6-10): Abordagem equilibrada, boa para a maioria das aplicações
* **Mais aulas** (15-20): Transições suaves, variação panorâmica, aparência fotográfica

**Quando usar:**

* **Poucas aulas (3-5)**: Slides de apresentação, mapas de classificação, relatórios simples
* **Classes médias (6-10)**: Análise geral, detalhes equilibrados, relatórios padrão
* **Muitas aulas (15-20)**: Análise científica, inspeção detalhada, resultados com qualidade de publicação

### Faixas de valores de ajuste fino

Os **controles de intervalo de valores** determinam quais valores de índice são mapeados para quais núcleos em seu gradiente:

**Controles de alcance no painel LUT:**

* **Valor mínimo**: Limite inferior da escala de núcleos
* **Valor* máximo*: Limite superior da escala de núcleos
* **Valores intermediários**: distribuídos automaticamente entre mínimo e máximo (com base na contagem de turmas)

#### Ajustando Valores Mín/Máx.

**Para ajustar intervalos de valores:**

1. No painel LUT, localize os campos de entrada **Min Value** e **Max Value**
2. Clique no campo **Valor mínimo**
3. Digite o valor mínimo desejado (por exemplo,`0.2`)
4. Pressione **Enter** ou clique fora do campo
5. Repita para o campo **Valor máximo** (por exemplo,`0.9`)
6. A visualização **atualiza imediatamente**

{% dica estilo = "info" %}
**Escalonamento automático**: Quando você aplica uma LUT pela primeira vez, o Chloros define automaticamente o mínimo/máximo para o intervalo de dados reais na imagem. Você pode então restringir esse intervalo para se concentrar em intervalos de valores específicos de interesse.
{% endhint %}

**Exemplos de ajustes de faixa de NDVI:**

* **Gama completa**:`-1.0` to `1.0`(mostrar todos os valores possíveis)
* **Foco na vegetação**:`0.2` to `0.9`(excluir solo descoberto e água)
* **Apenas vegetação saudável**:`0.5` to `0.9`(destaque apenas plantas vigorosas)
* **Detecção de estresse**:`0.2` to `0.5`(enfatize as áreas problemáticas)
* **Intervalo personalizado**: ajuste com base nos valores de pixels coletados

**Por que ajustar os intervalos?**

* **Aumente o contraste** na sua área de interesse
* **Excluir valores irrelevantes** (por exemplo, corpos d'água, solo descoberto)
* **Padronize a visualização** em diversas imagens ou dados
* **Enfatize diferenças sutis** dentro de uma faixa estreita de valores

### Registrando valores fora do intervalo

Quando os valores de pixel estão fora do intervalo mínimo/máximo definido, você pode controlar como eles são exibidos usando **modos de recorte**.

#### **Opções de modo de recorte disponíveis:**

#### 1. Mínimo e Máximo

* Pixels **abaixo do mínimo** → exibição usando a **primeira cor** em gradiente (por exemplo, vermelho)
* Pixels **acima do máximo** → exibir usando a **última cor** em gradiente (por exemplo, verde)
* **Caso de uso**: Enfatize os extremos, mostre o intervalo completo de dados com núcleos saturados nos limites
* **Exemplo**: valores de NDVI abaixo de 0,2 aparecem todos em vermelho, valores acima de 0,9 aparecem todos em verde

#### 2. Fundo Transparente

* Pixels **fora do intervalo** tornam-se **totalmente transparentes**
* Apenas pixels **dentro do intervalo** mostram gradiente de cor
* **Caso de uso**: sobreposição de GIS, isolando intervalos de valores específicos, destacando apenas áreas de interesse
* **Exemplo**: Mostrar apenas NDVI 0,4-0,7 em núcleos, todo o resto transparente

{% dica estilo = "aviso" %}
**Limitação de transparência**: pixels transparentes aparecem como cor de fundo no visualizador. Quando exportado durante o processamento, a transparência é preservada no formato PNG, mas não no JPG.
{% endhint %}

#### 3. Histórico do índice

* Pixels **fora do intervalo** são baratos em **tons de cinza** (mostrando valores de índice bruto)
* Pixels **dentro do intervalo** mostram **gradiente de cor**
* **Caso de uso**: Destaque sutil, mantenha o contexto enquanto enfatiza áreas de interesse
* **Exemplo**: Vegetação estressada com destaque colorido (NDVI 0,3-0,5) enquanto mostra áreas altas em cinza

#### 4. Plano de fundo original

* Pixels **fora do intervalo** exibem uma **imagem multiespectral original**
* Pixels **dentro do intervalo** mostram **gradiente de cor**
* **Caso de uso**: Mais intuitivo: combina o contexto natural da imagem com sobreposição analítica de núcleos
* **Exemplo**: Veja a aparência real do campo/colheita com áreas de tensão codificadas por núcleos sobrepostas

### Escolhendo o modo de recorte correto

| Modo de recorte | Melhor para | Estilo de visualização |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Mínimo e Máximo** | Exposição completa de dados, análise científica | Todos os pixels coloridos |
| **Fundo Transparente** | Sobreposições GIS, isolando especificidades específicas | Cor na faixa, em branco além |
| **Antecedentes do índice** | Ênfase sutil, mantendo o contexto dos dados | Cor na faixa, cinza além |
| **Fundo original** | Relatórios, apresentações, análises intuitivas | Cor na faixa, foto além |

### Criando núcleos LUT personalizados

Para ter controle total sobre sua visualização, você pode criar **gradientes de núcleos personalizados** editando intervalos de núcleos individuais.

**Para criar um gradiente personalizado:**

1. No painel LUT, localize a **barra de visualização de gradiente**
2. Procure **quadrados de amostra de núcleos** abaixo do gradiente
3. **Clique em uma parada de cor** para selecioná-la
4. Um **seletor de núcleos** é aberto
5. Escolha uma nova cor usando:
   * **Roda de núcleos**: seleção visual de núcleos
   * **Controles deslizantes RGB/HSV**: controle preciso de núcleos
   * **Entrada de código hexadecimal**: Especificação exata da cor (por exemplo,`#FF0000`para vermelho)
6. Clique fora do seletor de cores **para aplicar a nova cor**
7. O **atualiza imediatamente** na imagem

**Adicionando ou removendo interrupções de cor:**

* **Adicionar uma parada**: clique no ícone + para adicionar uma nova amostra no final
* **Remover uma parada**: clique duas vezes no quadrado colorido para remover uma amostra

**Estratégias de personalização:**

* **Inverter gradiente**: inverta a ordem das cores para inverter o significado (por exemplo, verde = baixo, vermelho = alto)
* **Núcleos da marca**: combine uma paleta de núcleos da sua organização para relatórios
* **Para daltônicos**: use combinações laranja-azul ou roxo-amarelo
* **Otimização de impressão**: escolha de cores que funcionarão tanto na impressão colorida quanto em escala de cinza
* **Vários limites**: usar núcleos distintos em limites de valores específicos para classificação

{% dica estilo = "info" %}
**Salvar gradientes personalizados**: gradientes personalizados podem ser salvos e reutilizados. Clique no ícone salvar no painel LUT para preservar seus esquemas de núcleos personalizados para uso futuro.
{% endhint %}

***

## Fluxo de trabalho interativo

### Atualizações em tempo real

Todas as configurações de LUT no sandbox atualizam a imagem **intencionalmente e interativamente**:

* **Alterar camada** → A imagem muda imediatamente
* **Seleção gradiente** → As cores são atualizadas instantaneamente
* **Ajuste a faixa de valores** → Mudanças de contraste em tempo real
* **Alterar aulas** → A especialização do gradiente é atualizada imediatamente
* **Modificar recorte** → A exibição do plano de fundo muda instantaneamente
* **Editar núcleos** → O gradiente personalizado é aplicado imediatamente

**Não é necessário o botão "Aplicar"** - todas as alterações são ao vivo e interativas!

{% dica estilo = "sucesso" %}
**Feedback ao vivo**: o feedback visual instantâneo permite que você experimente rapidamente diferentes configurações até encontrar a visualização ideal para suas necessidades de análise.
{% endhint %}

### Fluxo de trabalho de refinamento iterativo

**Fluxo de trabalho típico de otimização de LUT:**

1. **Selecione a camada de índice** (por exemplo, RAW (Refletância))
2. **Aplicar índice** - Escolha o filtro da câmera e a fórmula do índice, atraindo os círculos coloridos para o local protegido na fórmula do índice
3. **Aplicar gradiente LUT** - Comece com a predefinição Vermelho-Amarelo-Verde
4. **Inspecione os valores dos pixels** - Mova o cursor e observe os intervalos de valores
5. **Ajuste mín/máx** - Estreite para focar no planejamento (por exemplo, 0,2 a 0,9)
6. **Escolha o recorte** - Experimente "Plano de fundo original" para contextualizar
7. **Refinar cores** - Personalize o gradiente, se necessário, para dar ênfase específica
8. **Finalizar configurações** - Configurações do documento e copiar para Configurações do projeto para processamento de exportação

### Inspeção de valor de pixel

Compreender os valores reais dos pixels é crucial para definir intervalos LUT eficazes:

**Como funcionar valores:**

1. Os valores de pixel são mostrados quando a imagem tem o Índice ou ambas as caixas Índice e LUT **marcadas**.
2. **Mova o cursor** sobre diferentes áreas da imagem
3. **Observe os valores de pixel** exibidos na legenda enquanto você passa o mouse
4. Aumente o zoom para ver pixels individuais destacados com um valor flutuante
5. **Tome notas** das variações de valores para diferentes recursos:
   * **Vegetação saudável**: por exemplo, NDVI 0,55-0,85
   * **Vegetação estressada**: por exemplo, NDVI 0,30-0,50
   * **Solo descoberto**: por exemplo, NDVI 0,05-0,25
   * **Água** (se presente): por exemplo, NDVI -0,05 a 0,10

**Usando valores de pixel para definir intervalos de LUT:**

Depois de executar os valores de pixel, ajuste seu LUT mínimo/máximo de acordo:

**Cenário de exemplo:**

* **Observação**: Valores do solo = 0,05-0,25, Estressado = 0,25-0,50, Saudável = 0,50-0,85
* **Objetivo**: visualizar apenas a saúde das plantas (excluindo apenas)
* **Configurações LUT**: Mín =`0.25`, Máx =`0.85`
* **Recorte**: "Fundo original" para ver o solo na cor natural
* **Resultado**: O gradiente de cor se aplica apenas à vegetação, o solo é exibido como imagem original

{% dica estilo = "info" %}
**Faixa Dinâmica**: Diferentes culturas, estações e estágios de crescimento possuem diferentes faixas de valores. Sempre verifique os valores de pixel em seu conjunto de dados específicos antes de definir intervalos de LUT.
{% endhint %}

***

## Índices personalizados (Cloros+)

### Criação de fórmulas de índice personalizadas

{% dica estilo = "info" %}
**Onde criar**: índices personalizados podem ser configurados em **Configurações do projeto** antes do processamento, bem como na barra lateral da sandbox do Image Viewer.
{% endhint %}

**Para criar um índice personalizado:**

1. **Abra as configurações do projeto** (antes do processamento) ou a barra lateral da sandbox do Image Viewer
2. Navegue até a lista suspensa **Fórmula do índice**
3. Adquira a opção **"Custom"** (deve estar logado com licença Chloros+)
4. **Definir sua fórmula** usando variáveis ​​de banda:
   * Nomes de bandas:`NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operadores:`+`, `-`, `*`, `/`, `^`(expoente)
   * Funções:`sqrt()`, `abs()`, etc. (se compatível)
   *Parênteses:`()`para ordem de operações
5. **Nomeie seu índice** (por exemplo, "MyIndex" ou "CustomNDVI")
6. **Salve uma configuração**

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

{% dica estilo = "aviso" %}
**Validação de fórmula**: comprove-se que sua fórmula usa faixas disponíveis em sua câmera. Por exemplo, RedEdge está disponível apenas em câmeras com filtro RedEdge.
{% endhint %}

***

## Próximas etapas

Agora que você entende o Sandbox Index/LUT:

* **Aplicar ao processamento**: use as configurações descobertas em [Configurações do projeto](../project-settings/page-2.md)
* **Processo em lote**: aplique índices otimizados a conjuntos de dados completos
* **Saiba mais**: Leia [Fórmulas de índice multiespectral](../project-settings/multispectral-index-formulas.md)

Documentação relacionada:

* [**Camadas de imagem**](image-layers.md) - Gerenciamento e visualização de camadas
* [**Abrindo uma imagem em tela cheia**](page-3.md) - Noções básicas do visualizador de imagens
* [**Processamento de imagens (GUI)**](../processing-images-gui/page-1.md) - Fluxo de trabalho de processamento completo