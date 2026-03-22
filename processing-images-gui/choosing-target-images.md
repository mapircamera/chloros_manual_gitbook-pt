# Escolha das imagens-alvo

Marcar as imagens que contêm alvos de calibração é um passo crucial que acelera significativamente o fluxo de processamento do Chloros. Ao pré-selecionar as imagens-alvo, elimina-se a necessidade de o Chloros analisar todas as imagens do seu conjunto de dados em busca de alvos de calibração.

## Porquê marcar as imagens-alvo?

### Velocidade de processamento

Sem marcar as imagens-alvo, o Chloros tem de:

* Analisar todas as imagens do seu projeto
* Executar algoritmos de deteção de alvos em cada imagem
* Verificar centenas ou milhares de imagens desnecessariamente

**Resultado**: O processamento pode demorar significativamente mais tempo, especialmente em conjuntos de dados de grande dimensão.

### Com imagens-alvo marcadas

Quando marca a coluna «Alvo» para imagens específicas:

* O Chloros apenas analisa as imagens marcadas em busca de alvos
* A deteção de alvos é concluída muito mais rapidamente
* O tempo total de processamento é consideravelmente reduzido

{% hint style="success" %}
**Melhoria de velocidade**: Marcar 2-3 imagens-alvo num conjunto de dados de 500 imagens pode reduzir o tempo de deteção de alvos de mais de 30 minutos para menos de 1 minuto.
{% endhint %}

***

## Como marcar imagens-alvo

### Passo 1: Identifique as suas imagens-alvo

Analise as imagens importadas no Navegador de Ficheiros e identifique quais as imagens que contêm alvos de calibração.

**Cenários comuns:*** **Alvo pré-captura**: Capturado antes de iniciar a sessão
* **Alvo pós-captura**: Capturado após concluir a sessão
* **Alvos no campo**: Alvos colocados dentro da área de captura
* **Vários alvos**: 2-3 imagens-alvo por sessão (recomendado)

### Passo 2: Verifique a coluna Alvo

Para cada imagem que contenha um alvo de calibração:

1. Localize a imagem na tabela do Navegador de Ficheiros
2. Encontre a coluna **Alvo** (coluna mais à direita)
3. Clique na caixa de seleção na coluna Alvo para essa imagem
4. Repita para todas as imagens que contenham alvos

### Passo 3: Verifique a sua seleção

Antes do processamento, verifique novamente:

* [ ] Todas as imagens com alvos de calibração estão marcadas
* [ ] Nenhuma imagem que não seja um alvo está marcada acidentalmente
* [ ] Os alvos estão claramente visíveis nas imagens marcadas

***

## Melhores práticas para imagens de alvo

### Diretrizes para a captura de alvos

**Tempo:**

* Capture imagens de alvo imediatamente antes e durante toda a sua sessão de captura
* Nas mesmas condições de iluminação do seu sensor de luz DAQ
* Idealmente, capture imagens de alvo com a maior frequência possível para obter os melhores resultados. Caso contrário, os dados do sensor de luz serão utilizados para ajustar a calibração ao longo do tempo.

**Posição da câmara:**

* Segure a câmara acima do alvo de forma a que este fique centrado e ocupe cerca de 40-60% do centro da imagem.
* Mantenha a câmara paralela/nadir à superfície do alvo

**Iluminação:**

* A mesma iluminação ambiente do seu sensor de luz DAQ
* Evite sombras nas superfícies dos alvos
* Não bloqueie a sua fonte de luz com o seu corpo, veículo ou vegetação
* Condições de céu nublado proporcionam resultados mais consistentes

**Condição do alvo:**

* Mantenha os painéis dos alvos limpos e secos
* Todos os 4 painéis devem estar claramente visíveis e sem obstruções
* Alvos perpendiculares/nadir à fonte de luz, se possível

### Quantas imagens do alvo?

**Mínimo:**1 imagem do alvo por sessão.**Recomendado:** 3-5 imagens do alvo por sessão.**Cronograma de melhores práticas:**

* 3-5 imagens capturadas logo após o sensor de luz iniciar a gravação
* Gire a câmara entre as capturas para obter os melhores resultados
* Opcional: periodicamente a meio da sessão, se as condições de iluminação mudarem constantemente

***

## Trabalhar com várias câmaras

### Configurações com duas câmaras

Se estiver a utilizar duas câmaras MAPIR simultaneamente (por exemplo, Survey3W RGN + Survey3N OCN):

1. Capture imagens de alvo com **ambas as câmaras** ao mesmo tempo
2. Utilize o **mesmo alvo físico** para ambas as câmaras
3. Marque as imagens de alvo para **ambos os tipos de câmara** no Navegador de Ficheiros
4. O Chloros utilizará os alvos apropriados para a calibração de cada câmara

### Coluna Modelo da Câmara

A coluna **Modelo da Câmara** ajuda a identificar quais as imagens que provêm de cada câmara:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Utilize esta coluna para verificar se marcou alvos para cada tipo de câmara no seu projeto.

***

## Definições de deteção de alvos

### Ajustar a sensibilidade de deteção

Se o Chloros não estiver a detetar os seus alvos corretamente, ajuste estas definições em [Definições do projeto](adjusting-project-settings.md):**Área mínima de amostra de calibração:*** **Padrão**: 25 píxeis
* **Aumente** se estiver a obter detecções falsas em pequenos artefactos
* **Diminua** se os alvos não estiverem a ser detetados**Agrupamento mínimo de alvos:*** **Padrão**: 60
* **Aumente** se os alvos estiverem a ser divididos em várias detecções
* **Diminua** se os alvos com variação de cor não forem totalmente detetados***

## Problemas comuns com imagens de alvo

### Problema: Nenhum alvo detetado

**Possíveis causas:**

* Imagens de alvo não marcadas no Navegador de ficheiros
* Alvo demasiado pequeno no enquadramento (&lt; 30% da imagem)
* Iluminação deficiente (sombras, reflexos)
* Definições de deteção de alvo demasiado restritivas

**Soluções:**

1. Verifique se a coluna «Alvo» está marcada para as imagens corretas
2. Verifique a qualidade da imagem do alvo na pré-visualização
3. Recapture os alvos se a qualidade for fraca
4. Ajuste as definições de deteção de alvos, se necessário

### Problema: Deteções falsas de alvos

**Possíveis causas:**

* Edifícios brancos, veículos ou cobertura do solo confundidos com alvos
* Manchas brilhantes na vegetação
* Sensibilidade de deteção demasiado baixa

**Soluções:**

1. Marque apenas as imagens de alvo reais para limitar o âmbito da deteção
2. Aumente a área mínima da amostra de calibração
3. Aumente o valor mínimo de agrupamento de alvos
4. Certifique-se de que as imagens de alvo mostram apenas o alvo (mínimo de ruído de fundo)

***

## Lista de verificação

Antes de iniciar o processamento, verifique a sua seleção de imagens de alvo:

* [ ] Pelo menos 1 imagem de alvo marcada por sessão
* [ ] As caixas de seleção da coluna «Alvo» estão marcadas para todas as imagens de alvo
* [ ] Imagens de alvo capturadas no mesmo período da pesquisa
* [ ] Alvos claramente visíveis na pré-visualização quando clicados
* [ ] Todos os 4 painéis de calibração visíveis em cada imagem de alvo
* [ ] Sem sombras ou obstruções nos alvos
* [ ] Para câmaras duplas: Alvos marcados para ambos os tipos de câmara

***

## Processamento sem alvos

### Processamento sem alvos de calibração

Embora não seja recomendado para trabalhos científicos, pode processar sem alvos:

1. Deixe todas as caixas de seleção da coluna Alvo desmarcadas
2. **Desative** a &quot;Calibração de refletância&quot; nas Configurações do projeto
3. A correção de vinheta continuará a ser aplicada
4. A saída não será calibrada para refletância absoluta

{% hint style="warning" %}
**Não recomendado**: Sem calibração de refletância, os valores dos píxeis representam apenas brilho relativo, não medições científicas de refletância. Utilize alvos de calibração para obter resultados precisos e repetíveis.
{% endhint %}

***

## Próximos passos

Depois de marcar as suas imagens-alvo:

1. **Revise as suas definições** - Consulte [Ajustar as definições do projeto](adjusting-project-settings.md)
2. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)
3. **Acompanhe o progresso** - Consulte [Monitorização do processamento](monitoring-the-processing.md)

Para mais informações sobre os alvos de calibração propriamente ditos, consulte [Alvos de calibração](../calibration-targets.md).
