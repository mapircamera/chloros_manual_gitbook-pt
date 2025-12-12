# Camadas de imagem

O menu suspenso Camadas de imagem no Visualizador de imagens do Chloros permite alternar rapidamente entre diferentes versões da mesma imagem - desde as capturas originais até saídas de refletância processadas e imagens de índices calculados.

## O que são camadas de imagem?

No Chloros, **camadas** consulte as diferentes saídas de imagem disponíveis para uma única imagem de origem. Ao processar imagens, o Chloros cria múltiplas versões:

* **Imagens originais** (arquivos JPG e RAW da sua câmera)
* **Refletância calibrada** saídas (se a concentração de refletância estiver habilitada)
* **Imagens alvo** (veja a imagem contém alvos específicos)
* **Imagens de índice** (NDVI, NDRE, GNDVI, etc. se os índices foram configurados)

O **menu suspenso Seletor de camada** no canto superior direito do Visualizador de imagens permite alternar instantes entre essas versões sem sair do visualizador.

***

## Tipos de camadas disponíveis

### JPG

* A imagem de visualização JPG original da sua câmera
* Sempre disponível para todas as imagens
* Não processado, conforme capturado pela câmera
* Mais rápido para carregar e exibir

**Quando visualizar:**

* Visualização rápida da captura original
* Verificando a composição e enquadramento da imagem
* Verificando a qualidade da captura antes do processamento

### CRU (Original)

* Os dados originais do sensor RAW da sua câmera
* Debayered sem pós-processamento aplicado
* Profundidade de bits maior que JPG (normalmente dados de sensor de 12 ou 14 bits)

**Quando visualizar:**

* Inspecionando a qualidade dos dados do sensor original
* Verificando problemas ou artefatos no sensor
* Comparando os resultados do processamento antes/depois

### RAW (alvo)

* Aparece apenas para imagens identificadas como contendo alvos de descoberta
* Mostrar a imagem RAW original com alvo detectado
* Usado para verificar se a detecção do alvo foi bem-sucedida

**Quando visualizar:**

* Confirmando que os alvos encontrados foram detectados corretamente
* Verificando a qualidade da imagem alvo
* Solução para problemas de deficiência

{% dica estilo = "info" %}
**Camada de destino**: esta camada só aparece no menu suspenso para imagens que contêm alvos de ocorrência. Imagens de captura regulares não terão essa opção.
{% endhint %}

### RAW (Refletância)

* A imagem de saída de refletância calibrada
* Vinheta corrigida (se habilitada no processamento)
* Refletância calibrada usando dados do alvo (habilitada)
* TIFF multibanda com todos os canais de câmera
* Os valores de pixel representam a refletância percentual (ao usar o modo percentual)
* Pronto para manipular com o [Sandbox de índice/LUT](index-lut-sandbox.md)

**Quando visualizar:**

* Inspecionando resultados calibrados
* Verificando a qualidade da experiência
* Verificando valores de pixels para resultados científicos
* Comparando com o original para ver os efeitos de encontrar

{% dica estilo = "sucesso" %}
**Recomendado**: Utilize uma camada RAW (Refletância) para verificar valores de pixels para análise científica e análise científica.
{% endhint %}

### RAW (Índice NDVI)... e similares

* Imagem calculada do índice de vegetação (NDVI neste exemplo)
* O nome do índice muda com base em qual índice foi configurado durante o processamento
* Exemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagem em escala de cinza de banda única mostrando resultados de cálculo de índice
* Uma camada aparece para cada índice configurado nas configurações do projeto

**Nomes de índices possíveis:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* E muito mais... (consulte [Fórmulas de índice multiespectral](../project-settings/multispectral-index-formulas.md))

**Quando visualizar:**

* Examinando os resultados do cálculo do índice
* Verificando intervalos de valores de índice
* Identificando áreas de interesse
* Verificando imagens de índice antes de usar em GIS ou análise

***

## Usando o seletor de camadas

### Abrindo o menu suspenso

1. Abra uma imagem no modo de tela cheia (clique em qualquer miniatura no Visualizador de imagens)
2. Localize o **menu suspenso de camadas** no canto superior direito do visualizador
3. O menu suspenso mostra a camada atualmente selecionada (por exemplo, "JPG")
4. Clique no menu suspenso para ver todas as camadas disponíveis

### Alternando camadas

1. Clique no menu suspenso da camada para abrir a lista
2. Todas as camadas disponíveis para a imagem atual são mostradas
3. Clique em qualquer nome de camada para mudar para essa versão
4. A imagem é atualizada imediatamente para exibir a camada removida

**Troca rápida:**

* O menu suspenso lembra sua última seleção
* Ao navegar para a próxima imagem, o Chloros tenta mostrar o mesmo tipo de camada
* Se essa camada não existir na próxima imagem, o padrão é JPG

### Disponibilidade de camadas

Nem todas as camadas estão disponíveis para todas as imagens:

**Sempre disponível:**

* ✅ JPG (cada imagem tem uma visualização em JPG)

**Disponível condicionalmente:**

* ⚠️ RAW (Original) - Apenas uma imagem foi capturada no modo RAW ou RAW+JPG
* ⚠️ RAW (Alvo) - Apenas a imagem contém alvos de descobertas descobertas
* ⚠️ RAW (Refletância) - Somente após processamento com ocorrência de refletância habilitada
* ⚠️ RAW (\[Índice] Índice) - Somente após processamento com índices configurados

***

## Persistência de Camada

### Navegando entre imagens

Ao navegar para uma imagem diferente (usando as teclas de seta ou clicando nas miniaturas):

**A preferência de camada é preservada:**

* Se você estiver visualizando "RAW (Refletância)", a próxima imagem mostra "RAW (Refletância)" (se disponível)
* Se você estiver visualizando "RAW (Índice NDVI)", a próxima imagem mostra "RAW (Índice NDVI)" (se disponível)
* Se a mesma camada não existir, o padrão é JPG

**Exemplo de fluxo de trabalho:**

1. Abra a imagem 1, mude para RAW (índice NDVI)
2. Pressione → para visualizar a Imagem 2
3. A imagem 2 exibe automaticamente a camada RAW (índice NDVI)
4. Continue navegando - todas as imagens mostram a camada NDVI
5. Muito eficiente para verificar resultados de índice em muitas imagens

***

## Fluxos de trabalho comuns

### Fluxo de trabalho 1: comparação antes/depois

**Objetivo**: comparar a imagem original com a calibrada

1. Abra a imagem processada no Image Viewer
2. Selecione **RAW (Original)** no menu suspenso
3. Observe os valores de vinheta e não calibrados
4. Mude para **RAW (Refletância)** no menu suspenso
5. Comparar - vinheta removida, valores calibrados

### Fluxo de trabalho 2: revisão do índice

**Objetivo**: analisar rapidamente os resultados do NDVI em todo o conjunto de dados

1. Abra a primeira imagem processada
2. Selecione **RAW (índice NDVI)** no menu suspenso
3. Use → tecla de seta para navegar para a próxima imagem
4. A camada NDVI persiste automaticamente
5. Continue através de todas as imagens, verificando os padrões NDVI
6. Mude para **RAW (índice NDRE)** para comparar

### Fluxo de trabalho 3: seleção de destino

**Objetivo**: verificar se todas as imagens alvo foram detectadas corretamente

1. Navegue até uma imagem de destino
2. Selecione **RAW (destino)** no menu suspenso
3. Verifique se os alvos de suspeitas estão claramente visíveis e detectados
4. Navegue para a próxima imagem de destino
5. Repita a verificação para todos os alvos

### Fluxo de trabalho 4: inspeção de valor de pixel

**Objetivo**: verificar os valores de refletância para obter resultados científicos

1. Abrir imagem processada
2. Selecione a camada **RAW (Refletância)**
3. Ativo o modo **Porcentagem de pixels** (botão na barra de ferramentas superior direita)
4. Mova o cursor sobre as áreas de vegetação
5. Verifique se os valores dos pixels estão nos intervalos esperados (30-70% para NIR, 5-15% para Vermelho)
6. Verifique as áreas de solo e água para valores protegidos

***

## Compreendendo os valores de pixel por camada

As camadas mostram diferentes intervalos de diferentes valores de pixel:

### Camada JPG

* **Intervalo**: 0-255 (8 bits)
* **Significado**: Valores de exibição, corrigidos por gama
* **Uso**: Somente inspeção visual, não para medição científica

### CRU (Original)

* **Intervalo**: 0-65535 (16 bits)
* **Significado**: Números digitais brutos do sensor
* **Uso**: Verificando o desempenho do sensor, não calibrado

### RAW (Refletância)

* **Intervalo**: 0-65.535 (TIFF de 16 bits) ou 0,0-1,0 (porcentagem de 32 bits)
* **Significado**: Refletância percentual calibrada
* **Uso**: Medições e análises científicas

**Para TIFF de 16 bits:** Divida por 65.535 para obter a porcentagem de refletância **Para porcentagem de 32 bits:** Os valores representam diretamente a porcentagem (0,5 = 50% de refletância)

### RAW (imagens de índice)

* **Intervalo**: variação de acordo com o índice (normalmente -1,0 a +1,0 para índices normalizados)
* **Significado**: resultado do cálculo do índice
* **Exemplos**:
  * NDVI: -1 a +1 (vegetação normalmente 0,4 a 0,9)
  * NDRE: -1 a +1 (detecção de estresse)
  * EVI: 0 a 1 (vegetação melhorada)

***

## Dicas e práticas recomendadas

### Troca de camada eficiente

* **Reconhecimento de atalhos de teclado**: embora não haja atalho de teclado para camadas, as setas de navegação (←/→) funcionam em todas as camadas
* **Fluxos de trabalho consistentes**: escolha uma camada (por exemplo, NDVI) e revise todo o conjunto de dados antes de mudar para outro
* **Comparações rápidas**: alterne entre Original e Refletância para verificar a qualidade do processamento

### Considerações de desempenho

* **JPG carrega mais rápido**: use para navegação rápida por muitas imagens
* **As camadas RAW carregam mais lentamente**: Maior resolução e profundidade de bits
* **Camadas de índice**: velocidade semelhante às camadas de refletância
* **O primeiro carregamento é mais lento**: as visualizações subsequentes da mesma camada são armazenadas em cache e mais rápidas

### Verificação de Qualidade

* **Sempre verifique RAW (Original)**: verifique a qualidade dos dados de origem antes de confiar nas saídas processadas
* **Comparar camadas**: use uma alternância de camadas para validar que o processamento funcionou corretamente
* **Verifique os intervalos de índice**: use o modo Pixel Percent com camadas de índice para verificar se os valores são razoáveis

***

## Solução de problemas

### Camada não disponível

**Problema**: a camada esperada não aparece no menu suspenso

**Causas possíveis:**

* A imagem não foi processada (apenas JPG e RAW (Original) disponíveis)
* A configuração de refletância foi desativada durante o processamento
* O índice específico não foi configurado nas configurações do projeto
* A imagem é apenas de destino (nenhum índice gerado para destinos)

**Soluções:**

1. Verifique se a imagem foi processada (verifique a pasta de saída dos arquivos processados)
2. Verifique as configurações do projeto para confirmar se os índices foram configurados
3. Reprocessar com os índices desejados habilitados

### Camada errada mostrada

**Problema**: a imagem abre em uma camada inesperada

**Causa**: A preferência de camada da imagem anterior foi transportada, mas essa camada não existe na imagem atual

**Solução**: O Cloros volta automaticamente para JPG quando a camada preferencial não está disponível - este é um comportamento normal

### Não consigo ver os alvos de ocorrência

**Problema**: a camada RAW (Destino) não mostra detecção de alvo

**Causas possíveis:**

* Os alvos não foram detectados durante o processamento
* Na verdade, a imagem não contém alvos
* Configurações de detecção de alvo muito rigorosas

**Soluções:**

1. Verifique o log de depuração para mensagens "Destino encontrado"
2. Verifique se a imagem realmente contém alvos visíveis
3. Ajustar as configurações de detecção de alvo nas configurações do projeto
4. Consulte [Escolher imagens de destino](../processing-images-gui/choosing-target-images.md)

***

## Recursos relacionados

### Ferramentas de visualização de imagens

Ao visualizar qualquer camada, você pode usar:

* **Controles de zoom**: amplo para detalhes do funcionamento
* **Pan**: clique e arraste para mover a imagem ampliada
* **Inspeção de valor de pixel**: veja os valores na localização do cursor
* **Setas de navegação**: mova-se entre imagens enquanto mantém a camada
* **Modo Pixel Percent**: Alternar entre exibição de DN e porcentagem

Consulte [Abrindo uma imagem em tela cheia](page-3.md) para obter a documentação completa do Image Viewer.

### Sandbox de índice/LUT

Para testar e visualizar o índice interativo:

* **Cálculo de índice em tempo real**: teste diferentes fórmulas de índice
* **Mapeamento de cores LUT**: aplique gradientes de cores a índices de escala de cinza
* **Exportar visualizações**: salve imagens de índice colorido

Consulte [Sandbox de índice/LUT](index-lut-sandbox.md) para obter detalhes.

***

## Próximas etapas

Agora que você entende as camadas da imagem:

* [**Abrindo uma imagem em tela inteira**](page-3.md) - Guia completo do visualizador de imagens
* [**Sandbox de índice/LUT**](index-lut-sandbox.md) - Visualização interativa do índice
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Referência de índices disponíveis
* [**Finalizando o processamento**](../processing-images-gui/finishing-the-processing.md) - Compreendendo as saídas processadas