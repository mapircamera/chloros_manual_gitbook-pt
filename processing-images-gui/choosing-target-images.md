# Escolha das imagens-alvo

Assinalar quais as imagens que contêm alvos de calibração indica ao Chloros exatamente onde procurá-los. Quando pelo menos uma imagem estiver assinalada na coluna «Alvo», o Chloros analisa **apenas as imagens assinaladas** — por isso, assinalar os alvos é a forma de acelerar o processamento e de evitar que as imagens de levantamento sejam confundidas com um alvo.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Porquê marcar imagens-alvo?

### A marcação controla a análise

Quando marca a coluna «Alvo» para imagens específicas:

* O Chloros analisa apenas as imagens marcadas em busca de alvos
* A deteção de alvos é concluída muito mais rapidamente
* As imagens de levantamento não podem dar origem a detecções falsas de alvos

Se **nenhuma** imagem estiver marcada, o Chloros volta a analisar todas as imagens do projeto:

* Os algoritmos de deteção de alvos são executados em cada imagem
* Centenas ou milhares de imagens são analisadas desnecessariamente
* O processamento demora significativamente mais tempo, especialmente no caso de grandes conjuntos de dados

{% hint style="success" %}
**Melhoria de velocidade**: Marcar 2-3 imagens-alvo num conjunto de dados de 500 imagens pode reduzir o tempo de deteção de alvos de mais de 30 minutos para menos de 1 minuto.
{% endhint %}

***

## Como marcar imagens-alvo

### Passo 1: Identifique as suas imagens-alvo

Analise as imagens importadas no Navegador de Ficheiros e identifique quais as imagens que contêm alvos de calibração.

**Cenários comuns:*** **Alvo pré-captura**: Capturado antes do início da sessão
* **Alvo pós-captura**: capturado após a conclusão da sessão
* **Alvos no terreno**: alvos colocados dentro da área de captura
* **Vários alvos**: 2 a 3 imagens de alvo por sessão (recomendado)

### Passo 2: Verifique a coluna «Alvo» <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Para cada imagem que contenha um alvo de calibração:

1. Localize a imagem na tabela do Navegador de Ficheiros
2. Encontre a coluna **«Alvo»** (coluna mais à direita)
3. Clique na caixa de seleção na coluna «Alvo» para essa imagem
4. Repita o procedimento para todas as imagens que contenham alvos

### Passo 3: Verifique a sua seleção

Antes do processamento, verifique novamente:

* [ ] Todas as imagens com alvos de calibração estão marcadas
* [ ] Nenhuma imagem que não seja um alvo foi marcada acidentalmente
* [ ] Os alvos estão claramente visíveis nas imagens marcadas

***

## LATTICE: Os alvos são opcionais quando um DAQ está a gravar

Para as câmaras multiespectrais LATTICE, um alvo de calibração no enquadramento é **uma das duas** referências de refletância possíveis:

* **Alvo no enquadramento**: quando uma imagem de alvo marcada passa pelos critérios de qualidade (QA) dChloros, o alvo torna-se a**referência de refletância absoluta** para as imagens à sua volta.
* **Irradiação descendente do DAQ**: quando não existe nenhum alvo (ou a QA falha), o Chloros calcula a refletância a partir da irradiação descendente do sensor de luz do DAQ (ρ = π·L/E). Se uma gravação do `.daq` ou do DAQ-M `.csv` abranger as suas capturas, obtém-se uma refletância calibrada**sem qualquer imagem-alvo**.

Este comportamento automático é o predefinido. No CLI / SDK, corresponde a `--reflectance-source auto`; também pode forçar `target` (estrito — sem substituição do DAQ) ou `daq` (autoridade do DAQ). Consulte a [Referência do CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrias de alvos LATTICE**: além da deteção clássica de painéis utilizada para Survey3, o processamento LATTICE suporta**alvos marcados com ArUco**,**alvos com ROI fixa**e**alvos em faixa**, configurados por projeto. Podem ser fornecidas varreduras de refletância**medida** do alvo por unidade através do número de série (CLI: `--target-reflectance-dir`, um `<serial>.csv` por unidade-alvo), com os espectros nominais T3/T4P como alternativa.

{% hint style="info" %}
**Módulo F988**: a refletância do F988 é calibrada utilizando um painel de refletância no local da cena: a banda situa-se fora da gama calibrada do sensor de luz do DAQ, pelo que o Chloros aplica a sua captura mais recente do painel e mantém-na entre as observações do painel. Se um módulo F988 for processado apenas com o DAQ, o Chloros rejeita a refletância baseada no DAQ para essa banda (motivo de omissão `dls-uncalibrated-band-988`) — o fluxo de trabalho com o painel é o método suportado.
{% endhint %}

***

## Melhores práticas para imagens-alvo

### Orientações para a captura de imagens-alvo

**Tempo:**

* Capture imagens-alvo imediatamente antes e ao longo da sua sessão de captura
* Nas mesmas condições de iluminação que o seu sensor de luz DAQ
* Idealmente, capte imagens do alvo com a maior frequência possível para obter os melhores resultados. Caso contrário, os dados do sensor de luz serão utilizados para ajustar a calibração ao longo do tempo.

**Posição da câmara:**

* Segure a câmara acima do alvo de forma a que este fique centrado e ocupe cerca de 40-60% do centro da imagem.
* Mantenha a câmara paralela ou na posição nadir em relação à superfície do alvo

**Iluminação:**

* A mesma iluminação ambiente que o seu sensor de luz DAQ
* Evite sombras nas superfícies do alvo
* Não bloqueie a sua fonte de luz com o seu corpo, veículo ou vegetação
* Condições de céu nublado proporcionam os resultados mais consistentes

**Condição do alvo:**

* Mantenha os painéis do alvo limpos e secos
* Todos os painéis do seu alvo (por exemplo, os 4 num T4) devem estar claramente visíveis e desobstruídos
* Se possível, posicione os alvos perpendicularmente ou na direção nadir em relação à fonte de luz

### Quantas imagens do alvo?

**Mínimo:**1 imagem do alvo por sessão.**Recomendado:** 3 a 5 imagens do alvo por sessão.**Cronograma recomendado:**

* 3 a 5 imagens captadas logo após o sensor de luz iniciar a gravação
* Alterne a câmara entre as capturas para obter os melhores resultados
* Opcional: periodicamente a meio da sessão, se as condições de iluminação mudarem constantemente

***

## Trabalhar com várias câmaras

### Configurações com duas câmaras

Se utilizar duas câmaras MAPIR em simultâneo (por exemplo, Survey3W RGN + Survey3N OCN):

1. Capture imagens-alvo com **ambas as câmaras** ao mesmo tempo
2. Utilize o **mesmo alvo físico** para ambas as câmaras
3. Marque as imagens dos alvos para **ambos os tipos de câmara** no Navegador de Ficheiros
4. O Chloros utilizará os alvos adequados para a calibração de cada câmara

### Coluna «Modelo da Câmara»

A coluna **«Modelo da Câmara»** ajuda a identificar quais as imagens provenientes de cada câmara:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* etc.

Utilize esta coluna para verificar se marcou alvos para cada tipo de câmara no seu projeto.

***

## Definições de deteção de alvos

### Ajustar a sensibilidade de deteção

Se o Chloros não estiver a detetar os seus alvos corretamente, ajuste estas definições em [Definições do projeto](adjusting-project-settings.md):**Área mínima da amostra de calibração (px):*** **Predefinição**: 25 píxeis
* **Aumente** se estiver a obter detecções falsas em pequenos artefactos
* **Diminua** se os alvos não estiverem a ser detetados**Agrupamento mínimo de alvos (0-100):*** **Padrão**: 60
* **Aumente** se os alvos estiverem a ser divididos em várias deteções
* **Diminua** se os alvos com variação de cor não estiverem a ser totalmente detetados

{% hint style="info" %}
**Dica do CLI**: O `chloros-cli process` aceita os mesmos parâmetros (`--min-target-size`, `--target-clustering`), e o seu sinalizador `--target`/`--targets` marca toda uma pasta de entrada como «apenas painel de alvos». Consulte a [Referência do CLI](../reference/cli-reference.md).
{% endhint %}

***

## Problemas comuns com imagens-alvo

### Problema: Não foram detetados alvos

**Possíveis causas:**

* Imagens-alvo não marcadas no Navegador de Ficheiros
* Alvo demasiado pequeno no enquadramento (&lt; 30% da imagem)
* Iluminação inadequada (sombras, reflexos)
* Definições de deteção de alvos demasiado restritivas

**Soluções:**

1. Verifique se a coluna «Alvo» está marcada para as imagens corretas
2. Verifique a qualidade da imagem do alvo na pré-visualização
3. Volte a capturar os alvos se a qualidade for fraca
4. Ajuste as definições de deteção de alvos, se necessário

### Problema: Deteções falsas de alvos

**Possíveis causas:**

* Edifícios brancos, veículos ou cobertura do solo confundidos com alvos
* Manchas brilhantes na vegetação
* Sensibilidade de deteção demasiado baixa

**Soluções:**

1. Marque apenas as imagens de alvo reais — apenas as imagens marcadas são analisadas
2. Aumente a área mínima da amostra de calibração
3. Aumente o valor mínimo de agrupamento de alvos
4. Certifique-se de que as imagens de alvo mostram apenas o alvo (o mínimo de ruído de fundo)

***

## Lista de verificação

Antes de iniciar o processamento, verifique a sua seleção de imagens-alvo:

* [ ] Pelo menos 1 imagem-alvo marcada por sessão (ou, para o LATTICE, uma gravação `.daq`/`.csv` que abranja a sessão)
* [ ] As caixas de seleção da coluna «Alvo» estão marcadas para todas as imagens-alvo
* [ ] Imagens-alvo captadas no mesmo intervalo de tempo que o levantamento
* [ ] Alvos claramente visíveis na pré-visualização quando clicados
* [ ] Todos os painéis de calibração visíveis em cada imagem-alvo
* [ ] Sem sombras nem obstruções nos alvos
* [ ] Para sistema de duas câmaras: os alvos estão marcados em ambos os tipos de câmara

***

## Processamento sem alvos

### LATTICE: Com uma gravação DAQ

Se um sensor de luz DAQ tiver registado a irradiância descendente durante as suas capturas LATTICE, não é necessário qualquer alvo:

1. Importe o ficheiro `.daq` (ou DAQ-M `.csv`) com as imagens
2. Deixe a coluna «Alvo» desmarcada
3. A refletância é calculada automaticamente a partir da referência de irradiância descendente do DAQ
4. A radiancia nunca necessita de um alvo nem de um DAQ — provém exclusivamente da calibração radiométrica de fábrica da câmara

### Processamento sem qualquer referência

Também é possível processar sem alvos e sem um DAQ:

1. Deixe todas as caixas de seleção da coluna «Alvo» desmarcadas
2. **Desative** «Calibração de refletância / equilíbrio de brancos» nas Definições do Projeto — a deteção de alvos é então totalmente ignorada
3. A correção de vinheta continuará a ser aplicada
4. A saída não será calibrada para refletância absoluta (o LATTICE multiespectral continua a exportar produtos sem debayering, de pré-visualização e de radiancia)

{% hint style="warning" %}
**Não recomendado para trabalho científico dSurvey3**: sem calibração de refletância, os valores de píxeSurvey3es representam apenas brilho relativo, não medições científicas de refletância. Utilize alvos de calibração (ou, no caso do LATTICE, um sensor de luz DAQ) para obter resultados precisos e repetíveis.
{% endhint %}

***

## Próximos passos

Depois de ter marcado as suas imagens-alvo:

1. **Verifique as suas definições** - Consulte [Ajustar as definições do projeto](adjusting-project-settings.md)
2. **Inicie o processamento** — Consulte [Iniciar o processamento](starting-the-processing.md)
3. **Acompanhe o progresso** — Consulte [Acompanhamento do processamento](monitoring-the-processing.md)

Para mais informações sobre os alvos de calibração propriamente ditos, consulte [Alvos de calibração](../calibration-targets.md).
