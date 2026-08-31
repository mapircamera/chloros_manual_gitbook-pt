# Abrir uma imagem em ecrã inteiro

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Uma imagem aberta em ecrã inteiro, com o seletor de camadas no canto superior direito</p></figcaption></figure>

O Visualizador de Imagens Chloros é a interface em ecrã inteiro para visualizar, inspecionar e medir as suas imagens. É aqui que pode ler os **valores reais dos píxeis** — DN por canal, percentagem de refletância ou radianância em W/m²/sr/nm — em vez da pré-visualização distorcida que o ecrã apresenta.

## Aceder ao Visualizador de Imagens

### A partir do Navegador de Ficheiros

1. Abra o separador **Navegador de ficheiros** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Clique em qualquer **miniatura** na [grelha de imagens](image-grid.md)
3. A imagem abre em ecrã inteiro no separador **Visualizador de imagens**

A imagem abre no produto que a grelha estava a apresentar. Se a grelha estiver definida para `RAW (Reflectance)`, essa será a camada em que ficará.

### Abrir a barra lateral do **Visualizador de Imagens**Clique no ícone do**Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda para abrir o painel de análise. Este contém, de cima para baixo:

* o nome da imagem e o modelo da câmara
* o botão **Exportar/Guardar Imagem(ns)** (apenas quando um Índice ou LUT estiver ativo)
* as caixas de seleção **Índice**e**LUT** e o painel de configuração do índice — consulte [Área de teste de Índice/LUT](index-lut-sandbox.md)
* o painel **Valores do cursor**: leitura por canal, histograma da camada e o controlo GSD***

## Navegação e zoom

### Navegar pelas imagens

* **Imagem seguinte**: o botão → ou a tecla**→** (seta para a direita)
* **Imagem anterior**: o botão ← ou a tecla**←** (seta para a esquerda)
* **Ir para uma imagem específica**: volte à grelha e clique na respetiva miniatura

O zoom e a panorâmica mantêm-se à medida que se desloca entre imagens, pelo que pode percorrer um conjunto mantendo-se na mesma parte do quadro.

### Zoom

O zoom é controlado pela **roda do rato**, em incrementos de 15%, ancorado no cursor — o ponto sob o ponteiro permanece sob o ponteiro. O intervalo é limitado pelo tamanho da imagem e da janela: não é possível diminuir o zoom para além do ajuste à janela, e o limite superior é definido pela resolução nativa da imagem.

Não existem teclas dedicadas ao zoom no visualizador em ecrã inteiro. (Na grelha, **Ctrl + `+` / `−`** redimensiona as miniaturas — um controlo diferente.)

### Deslocamento com zoom

Clique e mantenha premido o botão esquerdo do rato sobre a imagem e arraste. O deslocamento é limitado, pelo que a imagem não pode ser arrastada para fora do ecrã.

### Inspeção por pixel com zoom elevado

Quando a ampliação efetiva ultrapassa **60×**, o Chloros desenha uma caixa de destaque em torno do pixel individual exibido sob o cursor e apresenta um valor flutuante ao lado do mesmo.

A ampliação «efetiva» tem em conta o tamanho do bloco GSD: com um tamanho de bloco de 8, a caixa de destaque aparece com um zoom de 7,5× em vez de 60×, porque um pixel exibido corresponde já a 8 × 8 píxeis de origem. Se reduzir o zoom para um valor inferior ao limiar, a caixa de destaque desaparece.

### Atalhos de teclado

| Tecla                             | Onde       | Ação                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Ecrã inteiro | Imagem seguinte                          |
| **←**                           | Ecrã inteiro | Imagem anterior                      |
| **Ctrl + R**                    | Ecrã inteiro | Reiniciar a área de teste do índice/LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Grelha        | Miniaturas maiores (4 px por pressão)  |
| **Ctrl + `−`**                  | Grelha        | Miniaturas mais pequenas (4 px por pressão) |***

## Valores do cursor

Passe o cursor sobre a imagem e o painel **Valores do cursor** indica o valor de cada canal abaixo dela.

{% hint style="success" %}
**Estes são os valores reais do ficheiro.** A tela no ecrã é uma pré-visualização esticada de 8 bits e não pode fornecê-los; por isso, o Chloros recolhe amostras do ficheiro do produto real para a leitura. É por isso que um fotograma bruto de 12 bits apresenta valores acima de 255 e que uma camada de radiação float32 apresenta unidades físicas.
{% endhint %}

### O que significam as colunas

O painel adapta-se à camada que está a visualizar:

| Camada que está a visualizar              | Colunas apresentadas    | Notas                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Refletância                        | **DN**e**%** | A percentagem é calculada com base na escala própria desse ficheiro — ver abaixo                                      |
| Radiação                           | **W/m²/sr/nm**   | Valores físicos de tipo float; não há coluna DN, porque um DN não faz sentido aqui                           |
| Bruto / Desbayizado / pré-visualização / JPG    | **DN**           | Números digitais inteiros                                                                         |
| Exportações de refletância em percentagem de 32 bits | Apenas **%**       | O valor de tipo float armazenado não é um DN, pelo que arredondá-lo para um número inteiro resultaria na impressão de valores sem sentido como `0` ou `1` |

Cada linha está identificada com o nome do canal do filtro da sua câmara — `Red / Green / NIR` para RGN, `Orange / Cyan / NIR` para OCN, `NIR / Green / Blue` para NGB, `Red / Green / Blue` para RGB e o nome da banda única para as câmaras RE, NIR e mono M3M. Cada etiqueta contém um ponto colorido que corresponde aos círculos de canal utilizados no editor de fórmulas de índice.

As imagens **de índice e LUT** guardadas constituem um caso especial: contêm componentes do mapa de cores em vez de bandas espectrais, pelo que as suas linhas são identificadas como `Red / Green / Blue` (ou `Index` para um ficheiro de índice de canal único) em vez de com os nomes dos filtros da câmara.

Quando um índice está ativo na área de teste, surge uma linha adicional abaixo dos canais, mostrando o **valor do índice** na posição do cursor, com o nome do índice e um ponto branco que corresponde ao seu marcador no histograma.

### A percentagem de refletância utiliza a escala própria de cada ficheiro

{% hint style="warning" %}
**Não se deve assumir que 65535 = 100%.** O Chloros armazena a refletância em escalas diferentes, dependendo da câmara que a produziu, e o visualizador determina a escala correta para cada ficheiro.
{% endhint %}

| Origem                  | DN que equivale à refletância 1,0 | Como é identificado                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Etiqueta XMP `Chloros:PixelScale=32768` gravada em todas as exportações de refletância do LATTICE. A margem de 2× permite que o ficheiro contenha ρ superior a 1,0 sem distorção |
| **Survey3**|**65535**                      | Sem etiqueta de escala XMP Chloros — a calibração Survey3 grava ρ × dtype-max e corta no valor 1,0                                                               |

O visualizador, a área de teste do índice/LUT e a exportação do índice resolvem todos a escala através da mesma implementação única; por isso, um valor que se lê no cursor é o mesmo valor utilizado nos cálculos do índice.

Duas consequências que vale a pena conhecer:

* Um TIFF de **32 bits em percentagem**armazena DN/65535 como um float, e uma exportação**de 8 bits** PNG/JPG armazena DN × 255/65535 — o visualizador converte ambos de volta antes de exibir uma percentagem.
* Há um caso que não pode ser recuperado: uma **exportação TIFF de 8 bits de uma captura com origem de 8 bits** é limitada ao intervalo 0–255 em vez de ser redimensionada e, deliberadamente, não inclui qualquer etiqueta de escala. Para esses ficheiros, o painel apresenta apenas o DN, sem a coluna de percentagem. Esta é a resposta sincera, não se trata de um erro.***

## O histograma da camada

Abaixo das linhas do cursor encontra-se um histograma em tempo real da camada que está a visualizar, em **256 intervalos**. Por predefinição, desenha uma curva combinada, ponderada**`(R + 2G + B) / 4`**— o mesmo espaço de medição que os histogramas da câmara LATTICE utilizam. Ao ativar**RGB**, este é substituído por curvas por canal nas cores dos canais, misturadas de forma aditiva para que as sobreposições se mantenham legíveis. As camadas monocromáticas desenham sempre a curva única.

O eixo horizontal está na unidade própria da camada:

| Camada       | Unidade do eixo  | Máximo do eixo                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Refletância | percentagem    | 125% — a margem do produto permite ρ superior a 1,0           |
| Radianância    | W/m²/sr/nm | O pico próprio do fotograma, arredondado por excesso para dois algarismos significativos |
| Dados de 8 bits  | DN         | 255                                                        |
| Dados de 12 bits | DN         | 4095                                                       |
| Dados de 16 bits | DN         | 65535                                                      |

Quando o eixo está em DN e atinge um desses três limites máximos, o Chloros também reconhece a profundidade de bits do que está a visualizar.

Existem três botões acima do histograma:

| Botão     | Predefinição | Efeito                                                                                                                                                                                                                                                                                                           |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Ativado      | Desenha linhas marcadoras no histograma nos valores exatos apresentados nas linhas acima, para que possa ver onde o pixel sob o cursor se situa na distribuição do fotograma. No modo RGB, existe um marcador por canal na sua própria cor; caso contrário, aparece um único marcador branco no valor combinado |
| **ÍNDICE**| Ativado      | Aparece apenas enquanto um índice estiver ativo. Alterna o histograma das bandas de origem para a**distribuição do valor do índice**, com os dois limiares de recorte representados por linhas tracejadas laranja e o valor do índice do cursor por uma linha branca                                                          |
| **RGB**| Desligado    | Alterna da curva combinada para curvas por canal. Num sensor mono, este botão exibe**MONO** e fica desativado — existe apenas um canal para mostrar                                                                                                                                  |

O histograma é calculado com base nos **blocos que se vêem**, e não nos píxeis de origem por trás deles: ao alterar o tamanho do bloco GSD, a distribuição é recalculada, de modo a que o histograma, o marcador do cursor e a imagem apresentada estejam sempre em concordância.***

## Tamanho do bloco GSD

Na parte inferior do painel encontra-se o controlo **GSD (px)**: uma caixa de número, um controlo deslizante de**1 a 256**e um botão**RESET**.

Este controlo torna a imagem _exibida_ menos detalhada, calculando a média de um bloco N × N de píxeis de origem num único pixel exibido. `1` é a resolução nativa.

* Afeta **a visualização em ecrã inteiro, as miniaturas da grelha, a indicação do cursor e ambos os histogramas** — tudo o que mostra a imagem está de acordo com a mesma resolução no terreno.
* É **apenas para visualização**. O processamento e a exportação não são afetados. A única exceção é deliberada: uma exportação para a [Index/LUT Sandbox](index-lut-sandbox.md) guarda o que está a ver, pelo que mantém o tamanho de bloco atual, e o painel de exportação avisa-o quando o tamanho de bloco for superior a 1.
* O valor é armazenado **por projeto** como `viewer_display.gsd_bin` no `project.json`, pelo que se mantém mesmo após o encerramento e a reabertura.
* A indicação do cursor refere o bloco, e não o pixel de origem, sempre que o tamanho do bloco for superior a 1 — o valor apresentado é a média do bloco sob o cursor.

{% hint style="info" %}
**Porquê «tamanho do bloco» e não centímetros por pixel?** Um valor em cm/px requer uma altura acima do solo. Os dados EXIF de um único fotograma contêm a altitude GPS acima do nível médio do mar, e não acima do terreno para o qual a câmara estava apontada; por isso, o Chloros não indicará uma distância ao solo que não possa calcular com precisão. O tamanho do bloco em píxeis de origem é o mesmo recurso alternativo que as ferramentas de nuvem do MAPIR utilizam quando a distância de amostragem ao solo é desconhecida.
{% endhint %}

***

## Tipos de imagem que pode visualizar

O menu suspenso de camadas no canto superior direito do visualizador lista todas as versões da imagem atual. As entradas que aparecem dependem da câmara e do que foi processado — consulte [Camadas de Imagem](image-layers.md) para ver a lista completa e saber como o menu suspenso funciona.

### Survey3

* **JPG** — o ficheiro de pré-visualização da própria câmara
* **RAW (Original)** — o ficheiro `.RAW` de origem, submetido a débayering para visualização, sem correções
* **RAW (Alvo)** — um fotograma identificado como contendo um alvo de calibração
* **RAW (Refletância)** — o produto de refletância calibrado (65535 = ρ 1,0)
* **Corrigido de vinheta**/**Resposta do sensor** — o produto de recurso não calibrado
* **Equilíbrio de brancos** — o produto com equilíbrio de brancos
* **RAW (Índice `<INDEX>`)**e**LUT `<INDEX>`** — imagens de índice calculadas

### LATTICE

As capturas do LATTICE utilizam o mesmo menu suspenso, com os nomes dos níveis do pipeline:

| Camada                 | O que contém                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | O fotograma RAW de origem tal como foi capturado                                     |
| **RAW (Sem efeito de Bayer)**   | A imagem linear sem efeito de Bayer                                           |
| **RAW (Pré-visualização)**     | A pré-visualização no ecrã — alongamento em cores falsas para câmaras multiespectrais |
| **Equilíbrio de brancos**    | A pré-visualização no ecrã para as câmaras principais RGB (equilíbrio de brancos + gama)   |
| **RAW (Radiação)**    | Radiação espectral em Float32, em W/m²/sr/nm                              |
| **RAW (Refletância)** | Refletância em uint16, 32768 = ρ 1,0                                    |

A radianância e a refletância são exclusivas do modo multiespectral: uma câmara principal RGB não possui radiometria por banda, pelo que essas camadas não são geradas para ela.

***

## Aplicação de índices e LUT

Aplique índices multiespectrais e tabelas de consulta (LUT) de cor a partir da barra lateral:

1. Abra o **Visualizador de Imagens** na barra lateral <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Assinale **Índice**

3. Escolha o filtro da sua câmara e uma fórmula de índice; em seguida, arraste os círculos dos canais para as ranhuras da fórmula
4. Adicione uma LUT e selecione um gradiente, limiares e um modo de recorte
5. Verifique os valores na posição do cursor e guarde o resultado com **Exportar/Guardar Imagem(ns)**Consulte [Área de Testes de Índices/LUT](index-lut-sandbox.md) para o guia passo a passo completo.***

## Resolução de problemas

### A imagem não abre

**Possíveis causas**: o ficheiro foi movido ou eliminado após a importação; o produto nunca foi gravado; memória insuficiente para uma imagem muito grande.**O que fazer**:

1. Verifique se o ficheiro da camada ainda existe na árvore de saída do projeto
2. Abra o ficheiro num visualizador externo para confirmar se está intacto
3. Feche outras aplicações para libertar memória

### A imagem está preta, branca ou com cores distorcidas

**Possíveis causas**: o alongamento do ecrã não tem com que trabalhar (um fotograma quase constante); uma camada float32 com valores invulgares; um índice que não produziu dados válidos.**O que fazer**:

1. Leia os valores do cursor — se todos os canais estiverem em zero ou próximos de zero, o problema está nos dados, não na visualização
2. Verifique o histograma: um pico isolado numa das extremidades indica que o fotograma está cortado ou vazio
3. Verifique o registo de processamento da execução que produziu a camada

### Os valores parecem errados

**Possíveis causas**: está numa camada diferente daquela que pensa; está a comparar uma percentagem com um DN bruto; está a comparar um ficheiro LATTICE com um ficheiro Survey3 utilizando o mesmo divisor.**O que fazer**:

1. Confirme a camada selecionada no menu suspenso — as unidades do painel seguem a camada
2. Para a refletância, utilize a coluna **%** em vez de dividir o DN manualmente; se tiver de dividir, utilize o valor `Chloros:PixelScale` desse ficheiro (32768 para o LATTICE; se estiver ausente, significa 65535 para o Survey3)
3. Defina o tamanho do bloco GSD de volta para 1 — acima de 1, está a ler uma média do bloco, não um pixel
4. Verifique se a calibração da refletância foi efetivamente executada para esse fotograma; um produto de recurso não calibrado (Resposta do Sensor / Corrigido para Vignette) não é refletância

***

## Próximos passos

* [**Camadas de imagem**](image-layers.md) — o nome de cada camada, quando existe, e o que significam os seus valores
* [**Área de teste de índices/LUT**](index-lut-sandbox.md) — criar, ajustar e exportar visualizações de índices
* [**Marcadores de mapa**](map-markers.md) — o mesmo conjunto de imagens num mapa
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — a referência dos índices

Para o fluxo de trabalho de processamento, consulte [Processamento de imagens (GUI)](../processing-images-gui/adding-files-to-a-project.md).
