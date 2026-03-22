# Abrir uma imagem em ecrã inteiro

O Visualizador de Imagens Chloros oferece uma interface dedicada em ecrã inteiro para visualizar, analisar e manipular as suas imagens multiespectrais. Quer esteja a visualizar imagens originais ou resultados processados, o Visualizador de Imagens oferece ferramentas poderosas para inspeção e análise.

## Aceder ao Visualizador de Imagens

### A partir do Navegador de Ficheiros

A forma mais comum de abrir uma imagem no Visualizador de Imagens:

1. Certifique-se de que está no separador **Navegador de Ficheiros** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Clique em qualquer **miniatura de imagem** na grelha de imagens
3. A imagem abre-se na **área de pré-visualização principal** (centro do ecrã)
4. A imagem está agora carregada e pronta para visualização em ecrã inteiro

### Abrir a separador do Visualizador de Imagens

Assim que uma imagem estiver carregada na área de pré-visualização:

1. Clique no ícone **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
2. A guia Visualizador de Imagens abre-se, exibindo a imagem selecionada em ecrã inteiro
3. Ferramentas avançadas de visualização e análise ficam disponíveis na barra lateral esquerda

***

## Visão geral da interface do Visualizador de Imagens

### Área de exibição principal

A maior parte do ecrã mostra a sua imagem:

* **Resolução total**: Imagens exibidas na resolução nativa
* **Zoom**: Use os controlos ou a roda do rato para ampliar
* **Deslocamento**: Clique e arraste para se deslocar quando estiver ampliado
* **Proporção mantida**: As imagens são redimensionadas proporcionalmente***

## Opções de visualização

### Navegação básica nas imagens

#### Navegar pelas imagens

Navegue pelo seu conjunto de imagens usando atalhos de teclado ou botões:

* **Imagem seguinte**: Clique no botão → ou prima a tecla**→** (Seta para a direita)
* **Imagem anterior**: Clique no botão ← ou prima a tecla**←** (Seta para a esquerda)
* **Ir para uma imagem específica**: Volte ao Navegador de ficheiros e clique na miniatura desejada

#### Controlos de zoom

Ajuste a ampliação para inspecionar os detalhes da imagem:

**Aumentar:*** Clique no botão **+** (Mais)
* Prima a tecla **+**ou**=*** Rode a roda do rato **para cima**

**Diminuir:*** Clique no botão **−** (Menos)
* Prima a tecla **−** (Menos)
* Rode a roda do rato **para baixo**

#### Deslocamento ao ampliar

Quando ampliado para além do tamanho do ecrã:

1. Mova o cursor do rato sobre a imagem
2. Clique e **mantenha premido o botão esquerdo do rato**

3.**Arraste** para deslocar a imagem
4. Solte para parar o deslocamento

**Alternativa**: Use as teclas de seta para deslocar em pequenos incrementos***

## Inspeção do valor dos píxeis

### Visualização dos valores dos píxeis no cursor

À medida que move o cursor do rato sobre a imagem, os valores dos píxeis são apresentados em tempo real:**Localização da apresentação dos valores:*** **Número flutuante e linha vermelha na legenda do gradiente LUT do índice do lado direito*** **Quando se amplia ainda mais, valor flutuante perto do cursor e do pixel destacado*** Mostra os valores do pixel **sob o cursor ou destacado*** Atualiza-se à medida que move o rato

***

## Tipos de imagem que pode visualizar

### JPG

**Imagens JPG da câmara:**

* Apresenta os dados JPG tal como visualizados
* Mostra valores originais, não corrigidos
* Útil para verificar a qualidade da imagem antes do processamento

### RAW (Original)

### RAW (Refletância)

**Após o processamento:**

* Vinheta corrigida
* Refletância calibrada
* Multibanda TIFF (Red, Green, NIR, etc.)
* Dados científicos prontos para análise

### RAW (Índice)

**NDVI, NDRE, GNDVI, etc. (ficheiros \_NDVI.tif):**

* Imagens em escala de cinzentos de banda única
* Os valores dos píxeis representam os resultados do cálculo do índice
* Intervalo normalmente de -1 a +1 para índices normalizados
* É possível aplicar tabelas de conversão de cores (LUTs) para visualização

***

## Aplicação de Índices e LUTs

Aplique índices multiespectrais e tabelas de conversão de cores (LUTs):

1. Localize a **Área de Teste de Índices/LUTs**no**Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra lateral
2. Selecione o índice de vegetação (NDVI, NDRE, etc.)
3. Selecione uma fórmula multiespectral ou crie a sua própria fórmula personalizada (apenas Chloros+)
4. Aplique um gradiente de LUT de cor para visualização
5. Ajuste os intervalos de valores e os limiares

Consulte [Index/LUT Sandbox](index-lut-sandbox.md) para obter instruções detalhadas.

***

## Atalhos de teclado

### Navegação

* **→** (Seta para a direita): Imagem seguinte
* **←** (Seta para a esquerda): Imagem anterior
* **Home**: Primeira imagem da lista
* **End**: Última imagem da lista

### Zoom

* **+**ou**=**: Aumentar zoom
* **−**: Diminuir zoom
* **Roda do rato**: Aumentar/diminuir zoom***

### Verificar cálculos de índices

Verifique se os índices foram calculados corretamente:

1. Abra NDVI ou outra imagem de índice
2. Verifique as áreas de vegetação:
   * **NDVI**: Deve apresentar valores entre 0,4 e 0,9 para plantas saudáveis
   * **NDRE**: Valores mais elevados para crescimento vigoroso
   * **GNDVI**: Semelhante a NDVI, mas sensível à clorofila
3. Verifique a não vegetação:
   * **Solo**: Perto de 0 ou ligeiramente negativo
   * **Água**: Valores negativos (-0,5 a 0)***

## Resolução de problemas de visualização

### A imagem não abre

**Possíveis causas:**

* Ficheiro corrompido durante o processamento
* Formato de ficheiro não suportado
* Memória insuficiente para imagens de grande dimensão

**Soluções:**

1. Tente abrir num visualizador externo para verificar a integridade do ficheiro
2. Verifique se o formato do ficheiro corresponde ao tipo esperado
3. Feche outras aplicações para libertar memória
4. Tente uma imagem mais pequena/diferente

### Exibição de imagem a preto ou branco

**Possíveis causas:**

* Intervalo de valores fora da capacidade de exibição
* Imagem de 32 bits com valores invulgares
* Erro no cálculo do índice

**Soluções:**

1. Verifique os valores dos píxeis - se todos forem muito baixos ou muito altos, ajuste o intervalo de exibição
2. Tente abrir no QGIS ou similar com ajuste automático do intervalo
3. Verifique o Registo de Depuração do processamento para erros

### Os valores dos píxeis parecem errados

**Possíveis causas:**

* Visualização da imagem errada (original vs. processada)
* A calibração não foi aplicada corretamente
* Os dados do sensor de luz não foram incluídos na entrada
* O modo percentual foi alternado incorretamente

**Soluções:**

1. Verifique se está a visualizar o resultado processado (verifique a extensão do nome do ficheiro)
2. Verifique o estado do botão do modo percentual
3. Compare com imagens conhecidas como válidas do mesmo conjunto de dados

***

## Próximos passos

Agora que já consegue visualizar imagens em ecrã inteiro:

* [**Camadas de imagem**](image-layers.md) - Saiba mais sobre a visualização multibanda
* [**Área de teste de índices/LUT**](index-lut-sandbox.md) - Aplique índices personalizados e mapeamento de cores
* [**Fórmulas de Índices Multiespectrais**](../project-settings/multispectral-index-formulas.md) - Compreenda os índices disponíveis

Para o fluxo de trabalho de processamento, consulte:

* [**Processamento de Imagens (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guia completo de processamento
