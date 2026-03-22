# Camadas de imagem

O menu suspenso «Camadas de imagem» no Visualizador de Imagens Chloros permite alternar rapidamente entre diferentes versões da mesma imagem — desde as capturas originais até aos resultados de refletância processados e às imagens de índice calculadas.

## O que são camadas de imagem?

No Chloros, **camadas** referem-se às diferentes saídas de imagem disponíveis para uma única imagem de origem. Quando processa imagens, o Chloros cria várias versões:

* **Imagens originais** (ficheiros JPG e RAW da sua câmara)
* Saídas **com reflectância calibrada** (se a calibração de reflectância tiver sido ativada)
* **Imagens-alvo** (se a imagem contiver alvos de calibração)
* **Imagens de índice** (NDVI, NDRE, GNDVI, etc., se os índices tiverem sido configurados)

O **menu suspenso Seletor de camadas** no canto superior direito do Visualizador de Imagens permite alternar instantaneamente entre estas versões sem sair do visualizador.***

## Tipos de camadas disponíveis

### JPG

* A imagem de pré-visualização JPG original da sua câmara
* Sempre disponível para todas as imagens
* Não processada, tal como capturada pela câmara
* Mais rápida a carregar e a apresentar

**Quando visualizar:**

* Pré-visualização rápida da captura original
* Verificação da composição e enquadramento da imagem
* Verificação da qualidade da captura antes do processamento

### RAW (Original)

* Os dados RAW originais do sensor da sua câmara
* Desbayered sem pós-processamento aplicado
* Maior profundidade de bits do que o JPG (normalmente dados do sensor de 12 ou 14 bits)

**Quando visualizar:**

* Inspecionar a qualidade dos dados originais do sensor
* Verificar se existem problemas no sensor ou artefactos
* Comparar os resultados antes e depois do processamento

### RAW (Alvo)

* Aparece apenas para imagens identificadas como contendo alvos de calibração
* Mostra a imagem RAW original com o alvo detetado
* Utilizado para verificar se a deteção do alvo foi bem-sucedida

**Quando visualizar:**

* Confirmar se os alvos de calibração foram detetados corretamente
* Verificar a qualidade da imagem do alvo
* Resolver problemas de calibração

{% hint style="info" %}
**Camada do Alvo**: Esta camada só aparece no menu suspenso para imagens que contenham alvos de calibração. As imagens de captura normais não terão esta opção.
{% endhint %}

### RAW (Refletância)

* A imagem de saída de refletância calibrada
* Vignette corrigida (se ativada no processamento)
* Refletância calibrada utilizando dados de alvo (se ativada)
* Multibanda TIFF com todos os canais da câmara
* Os valores dos píxeis representam a percentagem de refletância (ao utilizar o modo percentual)
* Pronta para manipulação com o [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizar:**

* Inspecionar resultados calibrados
* Verificar a qualidade da calibração
* Verificar valores de pixel para precisão científica
* Comparar com o original para ver os efeitos da calibração

{% hint style="success" %}
**Recomendado**: Use a camada RAW (Refletância) ao verificar valores de pixels para medições e análises científicas.
{% endhint %}

### RAW (Índice NDVI)... e similares

* Imagem do índice de vegetação calculado (NDVI neste exemplo)
* O nome do índice varia consoante o índice que foi configurado durante o processamento
* Exemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagem em escala de cinzentos de banda única que mostra os resultados do cálculo do índice
* Aparece uma camada para cada índice configurado nas Definições do Projeto

**Nomes de índice possíveis:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* E muitos mais... (ver [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md))

**Quando visualizar:**

* Analisar resultados de cálculo de índices
* Verificar intervalos de valores de índices
* Identificar áreas de interesse
* Verificar imagens de índices antes de utilizar em SIG ou análise

***

## Utilizar o Seletor de Camadas

### Abrir o menu suspenso

1. Abra uma imagem no modo de ecrã inteiro (clique em qualquer miniatura no Visualizador de Imagens)
2. Localize o **menu suspenso de camadas** no canto superior direito do visualizador
3. O menu suspenso mostra a camada atualmente selecionada (por exemplo, «JPG»)
4. Clique no menu suspenso para ver todas as camadas disponíveis

### Alternar entre camadas

1. Clique no menu suspenso de camadas para abrir a lista
2. São apresentadas todas as camadas disponíveis para a imagem atual
3. Clique em qualquer nome de camada para alternar para essa versão
4. A imagem atualiza-se imediatamente para mostrar a camada selecionada

**Mudança rápida:**

* O menu suspenso lembra-se da sua última seleção
* Ao navegar para a imagem seguinte, o Chloros tenta mostrar o mesmo tipo de camada
* Se essa camada não existir na imagem seguinte, o padrão é JPG

### Disponibilidade das camadas

Nem todas as camadas estão disponíveis para todas as imagens:

**Sempre disponível:*** ✅ JPG (todas as imagens têm uma pré-visualização em JPG)

**Disponível condicionalmente:**

* ⚠️ RAW (Original) - Apenas se a imagem tiver sido capturada no modo RAW ou RAW+JPG
* ⚠️ RAW (Alvo) - Apenas se a imagem contiver alvos de calibração detetados
* ⚠️ RAW (Refletância) - Apenas após o processamento com a calibração de refletância ativada
* ⚠️ RAW (\[Índice] Índice) - Apenas após o processamento com índices configurados

***

## Persistência das camadas

### Navegar entre imagens

Quando navega para uma imagem diferente (usando as setas do teclado ou clicando nas miniaturas):**A preferência da camada é mantida:**

* Se estiver a visualizar &quot;RAW (Refletância)&quot;, a imagem seguinte mostra &quot;RAW (Refletância)&quot; (se disponível)
* Se estiver a visualizar &quot;RAW (NDVI Índice)&quot;, a imagem seguinte mostra &quot;RAW (NDVI Índice)&quot; (se disponível)
* Se a mesma camada não existir, o padrão é JPG

**Exemplo de fluxo de trabalho:**

1. Abra a Imagem 1, mude para RAW (Índice NDVI)
2. Pressione → para visualizar a Imagem 2
3. A Imagem 2 exibe automaticamente a camada RAW (Índice NDVI)
4. Continue a navegar - todas as imagens mostram a camada NDVI
5. Muito eficiente para rever resultados de índice em muitas imagens

***

## Fluxos de trabalho comuns

### Fluxo de trabalho 1: Comparação antes/depois

**Objetivo**: Comparar a imagem original com a imagem calibrada

1. Abra a imagem processada no Visualizador de Imagens
2. Selecione **RAW (Original)** no menu suspenso
3. Observe o efeito de vinheta e os valores não calibrados
4. Mude para **RAW (Refletância)** no menu suspenso
5. Compare - efeito de vinheta removido, valores calibrados

### Fluxo de trabalho 2: Revisão do índice

**Objetivo**: Revisar rapidamente os resultados do NDVI em todo o conjunto de dados

1. Abra a primeira imagem processada
2. Selecione **RAW (Índice NDVI)** no menu suspenso
3. Use a tecla de seta → para navegar para a imagem seguinte
4. A camada NDVI persiste automaticamente
5. Continue por todas as imagens, verificando os padrões NDVI
6. Mude para **RAW (Índice NDRE)** para comparar

### Fluxo de trabalho 3: Verificação do alvo

**Objetivo**: Verificar se todas as imagens-alvo foram detetadas corretamente

1. Navegue até uma imagem-alvo
2. Selecione **RAW (Alvo)** no menu suspenso
3. Verifique se os alvos de calibração estão claramente visíveis e foram detetados
4. Navegue até à imagem-alvo seguinte
5. Repita a verificação para todos os alvos

### Fluxo de trabalho 4: Inspeção do valor dos píxeis

**Objetivo**: Verificar os valores de refletância para garantir a precisão científica

1. Abra a imagem processada
2. Selecione a camada **RAW (Refletância)**

3. Ative o modo**Porcentagem de píxeis** (botão na barra de ferramentas superior direita)
4. Mova o cursor sobre as áreas de vegetação
5. Verifique se os valores dos píxeis estão nos intervalos esperados (30-70% para NIR, 5-15% para Red)
6. Verifique se as áreas de solo e água apresentam valores adequados

***

## Compreender os valores dos píxeis por camada

Camadas diferentes apresentam intervalos de valores de píxeis diferentes:

### Camada JPG

* **Intervalo**: 0-255 (8 bits)
* **Significado**: Valores de visualização, com correção gama
* **Utilização**: Apenas para inspeção visual, não para medição científica

### RAW (Original)

* **Intervalo**: 0-65535 (16 bits)
* **Significado**: Números digitais brutos do sensor
* **Utilização**: Verificação do desempenho do sensor, não calibrado

### RAW (Refletância)

* **Intervalo**: 0-65 535 (16 bits TIFF) ou 0,0-1,0 (32 bits em percentagem)
* **Significado**: Refletância percentual calibrada
* **Utilização**: Medições e análises científicas**Para 16 bits TIFF:**Divida por 65.535 para obter a refletância percentual**Para 32 bits Percent:** Os valores representam diretamente a percentagem (0,5 = 50% de refletância)

### RAW (Imagens de Índice)

* **Intervalo**: Varia consoante o índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: Resultado do cálculo do índice
* **Exemplos**:
  * NDVI: -1 a +1 (vegetação normalmente de 0,4 a 0,9)
  * NDRE: -1 a +1 (detecção de stress)
  * EVI: 0 a 1 (vegetação melhorada)

***

## Dicas e melhores práticas

### Alternância eficiente entre camadas

* **Conhecimento dos atalhos de teclado**: Embora não haja atalhos de teclado para camadas, as setas de navegação (←/→) funcionam em todas as camadas
* **Fluxos de trabalho consistentes**: Escolha uma camada (por exemplo, NDVI) e analise todo o conjunto de dados antes de alternar para outra
* **Comparações rápidas**: Alterne entre Original e Refletância para verificar a qualidade do processamento

### Considerações de desempenho

* **O JPG carrega mais rapidamente**: Utilize-o para uma navegação rápida por muitas imagens
* **As camadas RAW carregam mais lentamente**: Maior resolução e profundidade de bits
* **Camadas de índice**: Velocidade semelhante às camadas de refletância
* **O primeiro carregamento é o mais lento**: As visualizações subsequentes da mesma camada são armazenadas em cache e são mais rápidas

### Verificação da qualidade

* **Verifique sempre o RAW (Original)**: Verifique a qualidade dos dados de origem antes de confiar nos resultados processados
* **Compare camadas**: utilize a alternância de camadas para validar se o processamento funcionou corretamente
* **Verifique os intervalos de índice**: utilize o modo Porcentagem de Píxeis com camadas de índice para verificar se os valores são razoáveis***

## Resolução de problemas

### Camada indisponível

**Problema**: a camada esperada não aparece no menu suspenso**Possíveis causas:**

* A imagem não foi processada (apenas JPG e RAW (Original) disponíveis)
* A calibração de refletância foi desativada durante o processamento
* O índice específico não foi configurado nas Definições do Projeto
* A imagem é uma imagem apenas de alvo (não são gerados índices para alvos)

**Soluções:**

1. Verifique se a imagem foi processada (verifique a pasta de saída para ver se existem ficheiros processados)
2. Verifique as Definições do Projeto para confirmar se os índices foram configurados
3. Reprocesse com os índices desejados ativados

### Camada errada apresentada

**Problema**: A imagem abre numa camada inesperada**Causa**: A preferência de camada da imagem anterior foi mantida, mas essa camada não existe na imagem atual**Solução**: O Chloros recorre automaticamente ao JPG quando a camada preferida não está disponível - este é um comportamento normal

### Não é possível ver os alvos de calibração

**Problema**: A camada RAW (Alvo) não mostra a deteção de alvos**Possíveis causas:**

* Os alvos não foram detetados durante o processamento
* A imagem não contém, de facto, alvos
* As definições de deteção de alvos são demasiado restritas

**Soluções:**

1. Verifique o Registo de Depuração para mensagens do tipo «Alvo encontrado»
2. Verifique se a imagem contém efetivamente alvos de calibração visíveis
3. Ajuste as definições de deteção de alvos nas Definições do Projeto
4. Consulte [Escolher Imagens-Alvo](../processing-images-gui/choosing-target-images.md)

***

## Funcionalidades relacionadas

### Ferramentas do Visualizador de Imagens

Ao visualizar qualquer camada, pode utilizar:

* **Controlos de zoom**: Amplie para inspecionar detalhes
* **Panorâmica**: Clique e arraste para se deslocar pela imagem ampliada
* **Inspeção do valor do pixel**: Veja os valores na localização do cursor
* **Setas de navegação**: Mova-se entre imagens mantendo a camada
* **Modo Porcentagem de Pixel**: Alterne entre a exibição em DN e em percentagem

Consulte [Abrir uma imagem em ecrã inteiro](opening-an-image-full-screen.md) para obter a documentação completa do Visualizador de Imagens.

### Caixa de areia de Índice/LUT

Para testes interativos de índice e visualização:

* **Cálculo de índice em tempo real**: Teste diferentes fórmulas de índice
* **Mapeamento de cores LUT**: Aplique gradientes de cor a índices em escala de cinzentos
* **Exportar visualizações**: Guarde imagens de índice coloridas

Consulte [Caixa de areia de Índice/LUT](index-lut-sandbox.md) para obter detalhes.

***

## Próximos passos

Agora que já compreende as camadas de imagem:

* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) - Guia completo do Visualizador de Imagens
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualização interativa de índices
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) - Referência dos índices disponíveis
* [**Concluir o processamento**](../processing-images-gui/finishing-the-processing.md) - Compreender os resultados processados
