# Escolhendo imagens alvo

Marcar quais imagens contêm alvos de atenção é uma etapa crucial que acelera significativamente o pipeline de processamento do Cloros. Ao pré-selecionar as imagens alvo, você elimina a necessidade do Chloros digitalizar todas as imagens do seu conjunto de dados em busca de alvos específicos.

## Por que marcar imagens alvo?

### Velocidade de processamento

Sem marcar as imagens alvo, o Cloros deve:

* Digitalize cada imagem do seu projeto
* Execute algoritmos de detecção de alvo em cada imagem
* Verifique centenas ou milhares de imagens desnecessariamente

**Resultado**: o processamento pode demorar significativamente mais, especialmente para grandes conjuntos de dados.

### Com imagens alvo marcado

Ao verificar a coluna Destino para imagens específicas:

* O Cloros apenas verifica as imagens verificadas em busca de alvos
* A detecção do alvo é concluída muito mais rapidamente
* O tempo geral de processamento é bastante limitado

{% dica estilo = "sucesso" %}
**Melhoria de velocidade**: marcar de 2 a 3 imagens alvo em um conjunto de dados de 500 imagens pode reduzir o tempo de detecção do alvo de mais de 30 minutos para menos de 1 minuto.
{% endhint %}

***

## Como marcar imagens alvo

### Etapa 1: Identifique suas imagens alvo

Examine as imagens importadas no Navegador de arquivos e identifique quais imagens contêm alvos específicos.

**Cenários comuns:**

* **Alvo pré-captura**: Capturado antes de iniciar a sessão
* **Alvo pós-captura**: Capturado após a conclusão da sessão
* **Alvos em campo**: Alvos colocados dentro da área de captura
* **Múltiplos alvos**: 2 a 3 imagens alvo por sessão (recomendado)

### Etapa 2: verifique a coluna de destino

Para cada imagem contendo um alvo específico:

1. Localize uma imagem na tabela do navegador de arquivos
2. Encontre a coluna **Destino** (coluna mais à direita)
3. Clique na caixa de seleção na coluna Destino dessa imagem
4. Repita para todas as imagens contendo alvos

### Etapa 3: verifique sua seleção

Antes de processar, verifique novamente:

* [] Todas as imagens com alvos de descoberta são verificadas
* [] Nenhuma imagem não alvo é verificada acidentalmente
* [] Os alvos são claramente visíveis nas imagens verificadas

***

## Melhores práticas para imagens alvo

### Diretrizes de captura de alvo

**Tempo:**

* Capturar imagens alvo imediatamente antes e durante a sessão de captura
* Nas mesmas condições de iluminação do seu sensor de luz DAQ
* O ideal é capturar imagens alvo com a maior frequência possível para obter os melhores resultados. Caso contrário, os dados do sensor de luz serão usados ​​para ajustar a privacidade ao longo do tempo.

**Posição da câmera:**

* Segure a câmera acima do alvo de forma que fique centralizada e preencha cerca de 40-60% do centro da imagem.
* Mantenha a câmera paralela/nadir à superfície alvo

**Iluminação:**

* Mesma iluminação ambiente do seu sensor de luz DAQ
* Evite sombras nas superfícies alvo
* Não bloqueie sua fonte de luz com seu corpo, veículo ou vegetação
* Condições nubladas fornecem resultados mais consistentes

**Condição alvo:**

* Mantenha os painéis alvo limpos e secos
* Todos os 4 painéis devem estar claramente visíveis e desobstruídos
* Alvos perpendiculares/nadir à fonte de luz, se possível

### Quantas imagens são alvo?

**Mínimo:** 1 imagem alvo por sessão. **Recomendado:** 3 a 5 imagens alvo por sessão.

**Cronograma de práticas recomendadas:**

* 3-5 imagens capturadas logo após a gravação do sensor de luz
* Gire uma câmera entre as capturas para obter os melhores resultados
* Opcional: periodicamente no meio da sessão se as condições de iluminação mudarem constantemente

***

## Trabalhando com múltiplas câmeras

### Configurações da câmera dupla

Se você estiver usando duas câmeras MAPIR simultaneamente (por exemplo, Survey3W RGN + Survey3N OCN):

1. Capture imagens alvo com **ambas as câmeras** ao mesmo tempo
2. Use o **mesmo alvo físico** para ambas as câmeras
3. Marque imagens de destino para **ambos os tipos de câmera** no Navegador de arquivos
4. O cloro usará alvos protetores para a segurança de cada câmera

### Coluna Modelo de Câmera

A coluna **Modelo da câmera** ajuda a identificar quais imagens vieram de qual câmera:

* Pesquisa3W\_RGN
* Pesquisa3N\_OCN
* Pesquisa3W\_RGB
* etc.

Use esta coluna para verificar se você marcou alvos para cada tipo de câmera em seu projeto.

***

## Configurações de detecção de alvo

### Ajustando a sensibilidade de detecção

Se o Chloros não estiver detectando seus alvos corretamente, ajuste essas configurações em [Configurações do projeto](adjusting-project-settings.md):

**Área mínima de amostra de ocorrência:**

* **Padrão**: 25 pixels
* **Aumente** se obteve detecções falsas em pequenos artistas
* **Diminua** se os alvos não estiverem sendo detectados

**Clustering de destino mínimo:**

* **Padrão**: 60
* **Aumente** se os alvos estivessem sendo divididos em múltiplas detecções
* **Diminua** se os alvos com variação de cor não foram totalmente detectados

***

## Problemas comuns de imagem alvo

### Problema: Nenhum alvo detectado

**Causas possíveis:**

* Imagens de destino não marcadas no navegador de arquivos
* Alvo muito pequeno no quadro (<30% da imagem)
* Má iluminação (sombras, brilho)
* Configurações de detecção de alvo muito rigorosas

**Soluções:**

1. Verifique se a coluna Target está marcada para imagens corretas
2. Revisar a qualidade da imagem desejada na visualização
3. Recapturar alvos se a qualidade for ruim
4. Ajuste as configurações de detecção de alvo, se necessário

### Problema: Detecções de alvos falsos

**Causas possíveis:**

* Prédios, veículos ou cobertura do solo branco confundidos com alvos
* Manchas manchadas na vegetação
* Sensibilidade de detecção muito baixa

**Soluções:**

1. Marque apenas imagens de destino reais para limitar o escopo da detecção
2. Aumentar a área mínima de amostra encontrada
3. Aumentar o valor mínimo de clustering de destino
4. -se de que as imagens de destino mostram apenas o alvo (mínima confusão de fundo)

***

## Lista de verificação de verificação

Antes de iniciar o processamento, verifique a seleção da imagem alvo:

* [] Pelo menos 1 imagem alvo marcada por sessão
* [] As caixas de seleção da coluna de destino são marcadas para todas as imagens de destino
* [] Imagens alvo capturadas no mesmo período da pesquisa
* [] Todos claramente visíveis na visualização quando clicados
* [] Todos os 4 painéis de blindagem ocultos em cada imagem alvo
* [] Sem sombras ou obstruções nos alvos
* [] Para câmera dupla: alvos marcados para ambos os tipos de câmera

***

## Processamento sem alvo

### Processamento sem alvos específicos

Embora não seja recomendado para trabalhos científicos, você pode analisar sem metas:

1. Deixe todas as caixas de seleção da coluna de destino demarcadas
2. **Desativar** "Calibração de refletância" nas configurações do projeto
3. A correção da vinha ainda será aplicada
4. A saída não será calibrada para refletância absoluta

{% dica estilo = "aviso" %}
**Não recomendado**: sem isolamento de refletância, os valores de pixel representam apenas o brilho relativo, e não específicos científicos de refletância. Use alvos específicos para obter resultados precisos e repetíveis.
{% endhint %}

***

## Próximas etapas

Depois de marcar suas imagens de destino:

1. **Revisar suas configurações** - Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
2. **Iniciar processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)

Para obter mais informações sobre os próprios alvos de calibração, consulte [Alvos de calibração](../calibration-targets.md).
