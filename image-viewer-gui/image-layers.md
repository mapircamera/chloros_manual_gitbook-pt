# Camadas de imagem

O menu suspenso Camadas de imagem no Visualizador de imagens Chloros permite alternar rapidamente entre diferentes versões da mesma imagem - desde as capturas originais até saídas de refletância processadas e imagens de índice calculadas.

## O que são camadas de imagem?

No Chloros, **camadas** referem-se às diferentes saídas de imagem disponíveis para uma única imagem de origem. Quando processa imagens, o Chloros cria várias versões:

* **Imagens originais** (ficheiros JPG e RAW da sua câmara)
* Saídas **calibradas por refletância** (se a calibração de refletância estiver ativada)
* **Imagens-alvo** (se a imagem contiver alvos de calibração)
* **Imagens de índice** (NDVI, NDRE, GNDVI, etc., se os índices estiverem configurados)

O **menu suspenso Seletor de camadas** no canto superior direito do Visualizador de imagens permite alternar instantaneamente entre essas versões sem sair do visualizador.

***

## Tipos de camadas disponíveis

### JPG

* A imagem de pré-visualização JPG original da sua câmara
* Sempre disponível para todas as imagens
* Não processada, tal como capturada pela câmara
* Mais rápida a carregar e exibir

**Quando visualizar:**

* Pré-visualização rápida da captura original
* Verificação da composição e enquadramento da imagem
* Verificação da qualidade da captura antes do processamento

### RAW (Original)

* Os dados originais do sensor RAW da sua câmara
* Desembarrado sem pós-processamento aplicado
* Maior profundidade de bits do que JPG (normalmente dados do sensor de 12 ou 14 bits)

**Quando visualizar:**

* Inspeção da qualidade dos dados originais do sensor
* Verificar se há problemas ou artefactos no sensor
* Comparar os resultados antes e depois do processamento

### RAW (alvo)

* Só aparece para imagens identificadas como contendo alvos de calibração
* Mostra a imagem RAW original com o alvo detetado
* Usado para verificar se a deteção do alvo foi bem-sucedida

**Quando visualizar:**

* Confirmar se os alvos de calibração foram detetados corretamente
* Verificar a qualidade da imagem do alvo
* Resolver problemas de calibração

{% hint style=&quot;info&quot; %}
**Camada de alvo**: esta camada só aparece no menu suspenso para imagens que contêm alvos de calibração. Imagens de captura regulares não terão essa opção.
{% endhint %}

### RAW (Refletância)

* A imagem de saída de refletância calibrada
* Vinheta corrigida (se ativada no processamento)
* Refletância calibrada usando dados de alvo (se ativada)
* Multibanda TIFF com todos os canais da câmara
* Os valores de pixel representam a refletância percentual (ao usar o modo percentual)
* Pronto para manipulação com o [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizar:**

* Inspecionar resultados calibrados
* Verificar a qualidade da calibração
* Verificar os valores dos pixels para precisão científica
* Comparar com o original para ver os efeitos da calibração

{% hint style=&quot;success&quot; %}
**Recomendado**: Use a camada RAW (Refletância) ao verificar os valores dos pixels para medições e análises científicas.
{% endhint %}

### RAW (NDVI Índice)... e similares

* Imagem do índice de vegetação calculado (NDVI neste exemplo)
* O nome do índice muda com base no índice que foi configurado durante o processamento
* Exemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagem em escala de cinza de banda única mostrando os resultados do cálculo do índice
* Uma camada aparece para cada índice configurado nas Definições do projeto

**Nomes de índices possíveis:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* E muitos mais... (consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md))

**Quando visualizar:**

* Examinar os resultados do cálculo do índice
* Verificar os intervalos de valores do índice
* Identificar áreas de interesse
* Verificar as imagens do índice antes de usar em GIS ou análise

***

## Usando o seletor de camadas

### Abrindo o menu suspenso

1. Abra uma imagem no modo de tela cheia (clique em qualquer miniatura no Visualizador de imagens)
2. Localize o **menu suspenso de camadas** no canto superior direito do visualizador
3. O menu suspenso mostra a camada selecionada no momento (por exemplo, «JPG»)
4. Clique no menu suspenso para ver todas as camadas disponíveis

### Alternando camadas

1. Clique no menu suspenso de camadas para abrir a lista
2. Todas as camadas disponíveis para a imagem atual são mostradas
3. Clique em qualquer nome de camada para alternar para essa versão
4. A imagem é atualizada imediatamente para mostrar a camada selecionada

**Alternância rápida:**

* O menu suspenso lembra a sua última seleção
* Ao navegar para a próxima imagem, o Chloros tenta mostrar o mesmo tipo de camada
* Se essa camada não existir na próxima imagem, o padrão será JPG

### Disponibilidade de camadas

Nem todas as camadas estão disponíveis para todas as imagens:

**Sempre disponível:**

* ✅ JPG (todas as imagens têm uma pré-visualização JPG)

**Disponível condicionalmente:**

* ⚠️ RAW (Original) - Apenas se a imagem foi capturada no modo RAW ou RAW+JPG
* ⚠️ RAW (Alvo) - Apenas se a imagem contiver alvos de calibração detetados
* ⚠️ RAW (Refletância) - Apenas após processamento com calibração de refletância ativada
* ⚠️ RAW (\[Índice] Índice) - Apenas após processamento com índices configurados

***

## Persistência da camada

### Navegando entre imagens

Quando você navega para uma imagem diferente (usando as setas do teclado ou clicando nas miniaturas):

**A preferência de camada é preservada:**

* Se estiver a visualizar «RAW (Refletância)», a próxima imagem mostra «RAW (Refletância)» (se disponível)
* Se estiver a visualizar «RAW (NDVI Índice)», a próxima imagem mostra «RAW (NDVI Índice)» (se disponível)
* Se a mesma camada não existir, o padrão será JPG

**Exemplo de fluxo de trabalho:**

1. Abra a Imagem 1, mude para RAW (NDVI Index)
2. Pressione → para visualizar a Imagem 2
3. A Imagem 2 exibe automaticamente a camada RAW (NDVI Index)
4. Continue a navegar - todas as imagens mostram a camada NDVI
5. Muito eficiente para rever os resultados do índice em muitas imagens

***

## Fluxos de trabalho comuns

### Fluxo de trabalho 1: Comparação antes/depois

**Objetivo**: Comparar a imagem original com a imagem calibrada

1. Abra a imagem processada no Visualizador de Imagens
2. Selecione **RAW (Original)** no menu suspenso
3. Observe a vinheta e os valores não calibrados
4. Mude para **RAW (Refletância)** no menu suspenso
5. Compare - vinheta removida, valores calibrados

### Fluxo de trabalho 2: Revisão do índice

**Objetivo**: Revisar rapidamente os resultados NDVI em todo o conjunto de dados

1. Abra a primeira imagem processada
2. Selecione **RAW (Índice NDVI)** no menu suspenso
3. Use a tecla de seta → para navegar até a próxima imagem
4. A camada NDVI persiste automaticamente
5. Continue por todas as imagens, verificando os padrões NDVI
6. Mude para **RAW (Índice NDRE)** para comparar

### Fluxo de trabalho 3: Verificação do alvo

**Objetivo**: Verificar se todas as imagens-alvo foram detetadas corretamente

1. Navegue até uma imagem-alvo
2. Selecione **RAW (Alvo)** no menu suspenso
3. Verifique se os alvos de calibração estão claramente visíveis e foram detetados
4. Navegue até a próxima imagem alvo
5. Repita a verificação para todos os alvos

### Fluxo de trabalho 4: Inspeção do valor do pixel

**Objetivo**: Verificar os valores de refletância para precisão científica

1. Abra a imagem processada
2. Selecione a camada **RAW (Refletância)**
3. Ative o modo **Porcentagem de pixel** (botão na barra de ferramentas superior direita)
4. Mova o cursor sobre as áreas de vegetação
5. Verifique se os valores dos pixels estão dentro dos intervalos esperados (30-70% para NIR, 5-15% para Red)
6. Verifique se as áreas de solo e água têm valores adequados

***

## Compreender os valores dos pixels por camada

Camadas diferentes mostram intervalos de valores de pixels diferentes:

### Camada JPG

* **Intervalo**: 0-255 (8 bits)
* **Significado**: Valores de exibição, com correção gama
* **Utilização**: Apenas para inspeção visual, não para medições científicas

### RAW (Original)

* **Intervalo**: 0-65535 (16 bits)
* **Significado**: Números digitais brutos do sensor
* **Utilização**: Verificação do desempenho do sensor, não calibrado

### RAW (Refletância)

* **Intervalo**: 0-65.535 (16 bits TIFF) ou 0,0-1,0 (32 bits Porcentagem)
* **Significado**: Percentagem de refletância calibrada
* **Utilização**: Medições e análises científicas

**Para 16 bits TIFF:** Divida por 65.535 para obter a percentagem de refletância **Para 32 bits Percent:** Os valores representam diretamente a percentagem (0,5 = 50% de refletância)

### RAW (imagens de índice)

* **Intervalo**: varia de acordo com o índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: resultado do cálculo do índice
* **Exemplos**:
  * NDVI: -1 a +1 (vegetação normalmente de 0,4 a 0,9)
  * NDRE: -1 a +1 (detecção de stress)
  * EVI: 0 a 1 (vegetação melhorada)

***

## Dicas e melhores práticas

### Troca eficiente de camadas

* **Conhecimento dos atalhos do teclado**: embora não haja atalhos do teclado para camadas, as setas de navegação (←/→) funcionam em todas as camadas
* **Fluxos de trabalho consistentes**: escolha uma camada (por exemplo, NDVI) e analise todo o conjunto de dados antes de mudar para outra
* **Comparações rápidas**: alterne entre Original e Reflectância para verificar a qualidade do processamento

### Considerações sobre desempenho

* **JPG carrega mais rápido**: use para navegação rápida por muitas imagens
* **Camadas RAW carregam mais lentamente**: resolução e profundidade de bits mais altas
* **Camadas de índice**: velocidade semelhante às camadas de refletância
* **O primeiro carregamento é o mais lento**: as visualizações subsequentes da mesma camada são armazenadas em cache e são mais rápidas

### Verificação de qualidade

* **Verifique sempre o RAW (Original)**: verifique a qualidade dos dados de origem antes de confiar nos resultados processados
* **Compare camadas**: use a alternância de camadas para validar se o processamento funcionou corretamente
* **Verifique os intervalos de índice**: use o modo Porcentagem de pixels com camadas de índice para verificar se os valores são razoáveis

***

## Resolução de problemas

### Camada indisponível

**Problema**: A camada esperada não aparece no menu suspenso

**Possíveis causas:**

* A imagem não foi processada (apenas JPG e RAW (Original) disponíveis)
* A calibração da refletância foi desativada durante o processamento
* O índice específico não foi configurado nas Definições do projeto
* A imagem é uma imagem apenas de destino (sem índices gerados para destinos)

**Soluções:**

1. Verifique se a imagem foi processada (verifique a pasta de saída para arquivos processados)
2. Verifique as configurações do projeto para confirmar se os índices foram configurados
3. Reprocesse com os índices desejados ativados

### Camada errada exibida

**Problema**: A imagem abre em uma camada inesperada

**Causa**: A preferência de camada da imagem anterior foi transferida, mas essa camada não existe na imagem atual

**Solução**: O Chloros recorre automaticamente ao JPG quando a camada preferida não está disponível - este é um comportamento normal

### Não consigo ver os alvos de calibração

**Problema**: A camada RAW (Alvo) não mostra a deteção do alvo

**Possíveis causas:**

* Os alvos não foram detetados durante o processamento
* A imagem não contém realmente alvos
* As definições de deteção de alvos são demasiado rigorosas

**Soluções:**

1. Verifique o registo de depuração para ver se há mensagens «Alvo encontrado»
2. Verifique se a imagem contém realmente alvos de calibração visíveis
3. Ajuste as definições de deteção de alvos nas Definições do projeto
4. Consulte [Escolher imagens-alvo](../processing-images-gui/choosing-target-images.md)

***

## Funcionalidades relacionadas

### Ferramentas do visualizador de imagens

Ao visualizar qualquer camada, pode utilizar:

* **Controlos de zoom**: Amplie para inspecionar detalhes
* **Panorâmica**: Clique e arraste para mover a imagem ampliada
* **Inspeção do valor dos pixels**: Veja os valores na localização do cursor
* **Setas de navegação**: Mova-se entre imagens mantendo a camada
* **Modo Porcentagem de pixels**: Alterne entre a exibição DN e porcentagem

Consulte [Abrir uma imagem em ecrã inteiro](opening-an-image-full-screen.md) para obter a documentação completa do Visualizador de imagens.

### Sandbox de índice/LUT

Para testes e visualização interativos de índices:

* **Cálculo de índice em tempo real**: teste diferentes fórmulas de índice
* **Mapeamento de cores LUT**: aplique gradientes de cor a índices em escala de cinza
* **Exportar visualizações**: salve imagens de índice coloridas

Consulte [Sandbox de índice/LUT](index-lut-sandbox.md) para obter detalhes.

***

## Próximos passos

Agora que você entende as camadas de imagem:

* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) - Guia completo do Visualizador de Imagens
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualização interativa do índice
* [**Fórmulas do índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Referência dos índices disponíveis
* [**Concluindo o processamento**](../processing-images-gui/finishing-the-processing.md) - Compreendendo os resultados processados
