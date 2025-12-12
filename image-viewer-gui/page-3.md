# Abrindo uma imagem em tela cheia

O Chloros Image Viewer fornece uma interface dedicada em tela cheia para visualizar, analisar e manipular suas imagens multiespectrais. Seja visualizando imagens originais ou saídas processadas, o Image Viewer oferece ferramentas poderosas para inspeção e análise.

## Acessando o Visualizador de Imagens

### Do navegador de arquivos

A maneira mais comum de abrir uma imagem no Image Viewer:

1. Verifique se você está na guia **Navegador de arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Clique em qualquer **miniatura de imagem** na grade de imagens
3. A imagem abre na **área principal de visualização** (centro da tela)
4. A imagem agora está carregada e pronta para visualização em tela cheia

### Abrindo a guia Visualizador de imagens

Depois que uma imagem for carregada na área de visualização:

1. Clique no ícone **Visualizador de imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
2. A guia Visualizador de imagens é aberta, exibindo a imagem selecionada em tela inteira
3. Ferramentas avançadas de visualização e análise ficam disponíveis na barra lateral esquerda

***

## Visão geral da interface do visualizador de imagens

### Área de exibição principal

A maior parte da tela mostra sua imagem:

* **Resolução total**: imagens exibidas em resolução nativa
* **Zoomável**: use os controles ou a roda do mouse para ampliar
* **Movimentável**: clique e arraste para mover quando ampliado
* **Proporção mantida**: as imagens são dimensionadas proporcionalmente

***

## Opções de visualização

### Navegação básica de imagens

#### Navegue pelas imagens

Navegue pelo seu conjunto de imagens usando atalhos de teclado ou botões:

* **Próxima imagem**: Clique no botão → ou pressione a tecla **→** (Seta para a direita)
* **Imagem anterior**: Clique no botão ← ou pressione a tecla **←** (Seta para a esquerda)
* **Pular para a imagem específica**: Retorne ao Navegador de Arquivos e clique na miniatura desejada

#### Controles de zoom

Ajuste a ampliação para inspecionar os detalhes da imagem:

**Ampliar:**

* Clique no botão **+** (Mais)
* Pressione a tecla **+** ou **=**
* Role a roda do mouse **para cima**

**Diminuir zoom:**

* Clique no botão **−** (Menos)
* Pressione a tecla **−** (menos)
* Role a roda do mouse **para baixo**

**Ajustar à tela:**

* Clique no botão **↔** (Ajustar)
* Pressione a tecla **0** (Zero)
* Clique duas vezes na imagem

#### Pan quando ampliado

Quando ampliado além do tamanho da tela:

1. Mova o cursor do mouse sobre a imagem
2. Clique e **mantenha pressionado o botão esquerdo do mouse**
3. **Arraste** para mover a imagem
4. Solte para parar de panoramizar

**Alternativa**: use as teclas de seta para deslocar em pequenos incrementos

***

## Inspeção de valor de pixel

### Visualizando Valores de Pixel no Cursor

À medida que você move o cursor do mouse sobre a imagem, os valores dos pixels são exibidos em tempo real:

**Local de exibição do valor:**

* **Número flutuante e linha vermelha na legenda do gradiente LUT do índice do lado direito**
* **Quando ampliado ainda mais, valor flutuante próximo ao cursor e pixel destacado**
* Mostra valores para pixel **sob o cursor ou realçado**
* Atualizações conforme você move o mouse

***

## Tipos de imagens que você pode visualizar

### Imagens originais (pré-processamento)

**Imagens RAW + JPG da câmera:**

* Exibir dados RAW conforme visualizados
* Mostrar valores originais e não corrigidos
* Útil para verificar a qualidade da imagem antes do processamento

### Imagens de refletância calibrada

**Após processamento:**

* Vinheta corrigida
* Refletância calibrada
* TIFF multibanda (vermelho, verde, NIR, etc.)
* Dados científicos prontos para análise

### Imagens de índice

**NDVI, NDRE, GNDVI, etc. (\_NDVI.tif arquivos):**

* Imagens em escala de cinza de banda única
* Os valores de pixel representam resultados de cálculo de índice
* Faixa normalmente de -1 a +1 para índices normalizados
* Pode aplicar LUTs coloridos para visualização

***

## Aplicação de índice e LUT

Aplique índices multiespectrais e tabelas de consulta de cores:

1. Localize **Index/LUT Sandbox** no **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> barra lateral
2. Selecione o índice de vegetação (NDVI, NDRE, etc.)
3. Selecione a fórmula multiespectral ou crie a sua própria fórmula personalizada (somente Cloros+)
4. Aplicar gradiente de cor LUT para visualização
5. Ajustar intervalos de valores e limites

See [Index/LUT Sandbox](index-lut-sandbox.md) for detailed instructions.

***

## Atalhos de teclado

### Navegação

* **→** (Seta para a direita): Próxima imagem
* **←** (Seta para a esquerda): imagem anterior
* **Home**: Primeira imagem da lista
* **Fim**: Última imagem da lista

### Zoom

* **+** ou **=**: Ampliar
* **−**: Diminuir zoom
* **0** (Zero): Ajustar à tela
* **Roda do mouse**: Aumentar/diminuir zoom

### Ver controles

* **P**: Alternar modo de porcentagem de pixels
* **L**: Alternar painel de camadas
* **Esc**: fecha a tela inteira ou retorna ao navegador de arquivos

### Outro

* **Ctrl+S**: Salva a imagem atual
* **F**: modo de tela inteira (se disponível)

***

### Verificando cálculos de índice

Verifique se os índices foram calculados corretamente:

1. Abra NDVI ou outra imagem de índice
2. Verifique as áreas de vegetação:
   * **NDVI**: Deve mostrar 0,4-0,9 para plantas saudáveis
   * **NDRE**: Valores mais altos para crescimento vigoroso
   * **GNDVI**: semelhante ao NDVI, mas sensível à clorofila
3. Verifique a não vegetação:
   * **Solo**: Perto de 0 ou ligeiramente negativo
   * **Água**: Valores negativos (-0,5 a 0)

***

## Solução de problemas de visualização

### A imagem não abre

**Possíveis causas:**

* Arquivo corrompido durante o processamento
* Formato de arquivo não suportado
* Memória insuficiente para imagens grandes

**Soluções:**

1. Tente abrir no visualizador externo para verificar a integridade do arquivo
2. Verifique se o formato do arquivo corresponde ao tipo esperado
3. Feche outros aplicativos para liberar memória
4. Experimente uma imagem menor/diferente

### Exibição de imagem em preto ou branco

**Possíveis causas:**

* Faixa de valores fora da capacidade de exibição
* Imagem flutuante de 32 bits com valores incomuns
* Erro de cálculo do índice

**Soluções:**

1. Verifique os valores dos pixels - se estiverem todos muito baixos ou muito altos, ajuste o intervalo de exibição
2. Tente abrir no QGIS ou similar com ajuste de intervalo automático
3. Verifique se há erros no log de depuração do processamento

### Os valores de pixel parecem errados

**Possíveis causas:**

* Visualizando imagem errada (original vs processada)
* A calibração não foi aplicada corretamente
* Os dados do sensor de luz não foram incluídos na entrada
* Modo percentual alternado incorretamente

**Soluções:**

1. Verifique se você está visualizando a saída processada (verifique o sufixo do nome do arquivo)
2. Verifique o estado do botão do modo percentual
3. Compare com imagens em bom estado do mesmo conjunto de dados

***

## Próximas etapas

Agora que você pode visualizar as imagens em tela cheia:

* [**Image Layers**](image-layers.md) - Aprenda sobre visualização multibanda
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Aplicar índices personalizados e mapeamento de cores
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Entenda os índices disponíveis

Para fluxo de trabalho de processamento, consulte:

* [**Processamento de imagens (GUI)**](../processing-images-gui/page-1.md) - Guia completo de processamento
