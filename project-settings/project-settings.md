# Definições do projeto

A barra lateral «Definições do projeto» (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) emChloros

permite-lhe configurar todos os aspetos do processamento de imagens, deteção de alvos de calibração, cálculos de índices multiespectrais e opções de exportação para o seu projeto. Estas definições são guardadas juntamente com o seu projeto e podem ser guardadas como modelos para reutilização em vários projetos.

## Aceder às definições do projeto

Para aceder às definições do projeto:

1. Abra um projeto emChloros


2. Clique no separador **Definições do projeto**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

na barra lateral esquerda
3. O painel de definições apresentará todas as opções de configuração disponíveis, organizadas por categoria



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**As definições que dependem de outras definições aparecem a cinzento.** Quando uma opção principal torna uma definição impossível (por exemplo, desmarcar *Calibração de refletância / equilíbrio de brancos* torna impossível *Exportar refletância*), o controlo dependente é desativado e a sua dica de ferramenta indica a opção que tem de ser alterada.
{% endhint %}

***

## Visualização

### Resolução da miniatura da imagem

* **Tipo**: Seleção por menu suspenso
* **Opções**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Predefinição**: Predefinição (512 px)
* **Descrição**: Resolução (aresta mais longa, em píxeis) na qual as miniaturas da grelha de imagens são renderizadas. Valores mais elevados proporcionam uma imagem mais nítida quando se amplia, mas demoram mais a carregar e consomem mais memória. A resolução total corresponde ao tamanho original da imagem.
* **Nota**: Apenas para visualização — isto nunca afeta o processamento nem os ficheiros exportados.***

## Detecção de alvos

Estas definições controlam a forma como oChloros

deteta e processa os alvos de calibração nas suas imagens. Ambas só estão ativas enquanto a **calibração de refletância / equilíbrio de brancos** estiver ativada (caso contrário, ficam a cinzento, porque a deteção de alvos é totalmente ignorada).

### Área mínima da amostra de calibração (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10 000 píxeis
* **Predefinição**: 25 píxeis
* **Descrição**: Define a área mínima (em píxeis) necessária para que uma região detetada seja considerada uma amostra válida de alvo de calibração. Valores mais baixos detetam alvos mais pequenos, mas podem aumentar os falsos positivos. Valores mais elevados exigem regiões-alvo maiores e mais nítidas para a deteção.
* **Quando ajustar**:
  * Aumente se estiver a obter detecções falsas em pequenos artefactos da imagem
  * Diminua se os seus alvos de calibração parecerem pequenos nas suas imagens e não estiverem a ser detetados

### Agrupamento mínimo de alvos (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Predefinição**: 60
* **Descrição**: Controla o limiar de agrupamento para regões de cores semelhantes durante a deteção de alvos de calibração. Valores mais elevados exigem que cores mais semelhantes sejam agrupadas, resultando numa deteção de alvos mais conservadora. Valores mais baixos permitem maior variação de cor dentro de um grupo de alvos.
* **Quando ajustar**:
  * Aumente se os alvos de calibração estiverem a ser divididos em várias deteções
  * Diminua se os alvos de calibração com variação de cor não estiverem a ser totalmente detetados

***

## Processamento

Estas definições controlam a forma como o «Chloros

» processa e calibra as suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcada)
* **Descrição**: Aplica correção de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é um fenómeno ótico comum em que os cantos e as bordas de uma imagem parecem mais escuros do que o centro, devido às características da lente.
* **Efeito secundário**: Esta opção também seleciona qual o *produto de recurso não calibrado* que uma execução grava (ver abaixo).

### Calibração de refletância / equilíbrio de brancos

* **Tipo**: Caixa de seleção
* **Predefinição**: Ativado (marcado)
* **Descrição**: Ativa a calibração de refletância — a partir de alvos de calibração detetados no enquadramento e/ou dados de luz descendente do sensor de luz do DAQ, dependendo da câmara e do que estiver disponível. Isto normaliza os valores de refletância em todo o seu conjunto de dados e garante medições consistentes, independentemente das condições de iluminação.
* **Quando desativado**: A deteção de alvos é totalmente ignorada e**nenhum produto de refletância pode ser gerado por qualquer câmara** — quer seja oSurvey3

, orientado por alvos, quer o LATTICE DAQ. As definições dependentes (*Exportar refletância*, *Intervalo mínimo de recalibração* e os limiares de deteção de alvos) ficam desativadas.

### Produtos alternativos não calibrados: Exportar resposta do sensor / Exportar com correção de vinheta

* **Tipo**: Duas caixas de seleção
* **Predefinições**: Ambas ativadas (marcadas)
* **Descrição**: Quando um fotograma não pode ser calibrado em termos de refletância (não foi encontrado nenhum alvo de calibração ou a calibração de refletância está desativada), é gravado como um *produto alternativo não calibrado*. **Existe exatamente um dos dois produtos de recurso por execução, para cada modelo de câmara**, escolhido pelo seletor *Correção de vinheta*:
  * Correção de vinheta **ativada**→ `Vignette_Corrected_Images/` (regida por**Exportar com correção de vinheta**)
  * Correção de vinheta **desativada**→ `Sensor_Response_Images/` (regida pela opção**Exportar resposta do sensor**)
* O produto de reserva que não está ativo é apresentado a cinzento. Desmarcar o que está ativo impede que esse ficheiro seja gravado.

### Produtos de exportação LATTICE

Para projetos que contenham capturas LATTICE, cada fotograma LATTICE importado é distribuído por todos os produtos ativados **e aplicáveis**numa única passagem de processamento. Quatro caixas de seleção controlam essa distribuição (todas**ativadas** por predefinição):

| Configuração | Pasta de saída | O que exporta |
| --- | --- | --- |
| **Exportar sem debayering** | `Debayered_Images/` | A imagem linear sem debayering. Aplica-se a câmarasRGB

e multiespectrais. |
| **Exportar pré-visualização** | `Preview_Images/` | A pré-visualização no ecrã.RGB

= equilíbrio de brancos (iluminante DAQ, quando disponível; caso contrário, «gray-world») + gama; multiespectral = alongamento de cores falsas. |
| **Exportar radiação** | `Radiance_Images/` | Radiação espectral Float32 em W/m²/sr/nm. Apenas multiespectral (M3C/M3M) — não aplicável a arquivos «RGB

». Sempre gravada como «TIFF

» de 32 bits, independentemente da definição do *formato de imagem calibrada*. |
| **Exportar refletância**| `Reflectance_Calibrated_Images/` | Refletância Uint16, dimensionada de modo a que**32768 = refletância 1.0** (marcada como XMP `Chloros:PixelScale`). Apenas multiespectral, gravada quando um registo descendente `.daq` correspondente (ou um alvo no quadro que tenha passado no controlo de qualidade) cobre o quadro. |

* As câmaras principaisRGB

emitem imagens sem debayering + pré-visualização; a radianância/refletância são ignoradas para estas, por não serem aplicáveis.
* A profundidade de bits do debayering/pré-visualização segue a definição *Formato de imagem calibrada*; a radianância é sempre float32.
* O processamentoSurvey3

não é afetado por estas quatro opções.

As mesmas quatro opções de ativação/desativação existem no modo «headless» como `chloros-cli process --debayered / --preview / --radiance / --reflectance` e como os parâmetros correspondentes do «SDK

». Substituíram o antigo sinalizador `--radiometric-output`, que já não existe.

{% hint style="warning" %}
**Desativar todos os produtos aplicáveis faz com que a execução falhe.** A partir da versão 1.2.0, uma execução de processamento que tenha solicitado produtos mas não tenha gravado nenhum produto de imagem reporta uma falha e o `CLI

` termina com um valor diferente de zero, em vez de reportar um sucesso silencioso. O registo indica o produto que não foi possível gravar e o motivo. Uma execução deliberadamente apenas com metadados (sem nada solicitado) continua a ser considerada bem-sucedida.
{% endhint %}

### Fonte de refletância (configuração do projeto, definida através deCLI

/SDK

)

O projeto também armazena qual a **referência de refletância** que o produto de refletância LATTICE utiliza. Não existe um controlo específico no painel de configurações; o valor é armazenado na configuração do projeto como `Processing → "Target reflectance source"` e é definido com `chloros-cli process --reflectance-source {auto,target,daq}` ou através do parâmetro `reflectance_source` doSDK

:

* **`auto`** (predefinição): um alvo de calibração no quadro que tenha passado no controlo de qualidade (QA) torna-se a referência absoluta, recorrendo à divisão de radiação descendente do DAQ (ρ = πL/E) quando não existe nenhum alvo ou o controlo de qualidade falha.
* **`target`**: refletância estritamente orientada pelo alvo — sem substituição pelo DAQ.
* **`daq`**: refletância determinada pelo DAQ; os alvos no quadro não são utilizados como referência.

O valor armazenado é comparado sem distinção entre maiúsculas e minúsculas e algumas grafias são aceites como aliases: `target`, `target_image`, `empirical` e `empirical_line` significam todos **alvo**; `daq`, `dls`, `light_sensor` e `sensor` significam todos**daq**. Qualquer outra coisa — incluindo uma chave ausente — resulta em**auto**.

As digitalizações de alvo **medidas** por unidade são pesquisadas pelo número de série/QR da unidade de destino, como `<serial>.csv`, em três locais: o diretório indicado com `--target-reflectance-dir` (armazenado como `Processing → "Target reflectance dir"`), na própria pasta `target_reflectance/` do projeto e no caminho da variável de ambiente `CHLOROS_TARGET_REFLECTANCE_DIR`. Quando não existe nenhuma varredura medida para essa unidade, é utilizada, em vez disso, a curva nominal publicada para o modelo alvo.

### Método de demosaicing

* **Tipo**: Seleção por menu suspenso
* **Opções**:
  * Padrão (Rápido, Qualidade Média)
  * Sensível à Textura (Lento, Qualidade Máxima) \[Chloros

+]
* **Predefinição**: Padrão (Rápido, Qualidade Média)
* **Descrição**: Seleciona o algoritmo de demosaicing utilizado para converter os dados brutos do sensor com padrão Bayer em imagens a cores. O método «Padrão (Rápido, Qualidade Média)» proporciona um equilíbrio ideal entre a velocidade de processamento e a qualidade da imagem. O «Sensível à textura (Lento, Qualidade máxima)» \[Chloros

+] utiliza um algoritmo de demosaicing sensível às arestas de alta qualidade, combinado com um modelo de redução de ruído baseado em IA/ML que elimina quase todo o ruído do demosaicing. O modelo «Texture Aware» requer memória da GPU (VRAM) para funcionar. Recomendamos a sua utilização quando tiver mais de 4 GB de VRAM disponível, para um processamento mais rápido.
* **Sempre que a linha for um menu suspenso**: o menu suspenso com duas opções só aparece quando**ambas** as condições forem verdadeiras — está com uma subscriçãoChloros

+ válida, **e** o projeto não contém capturas LATTICE. Caso contrário, a linha é apresentada como texto simples com a indicação «`Standard (Fast, Medium Quality)`», sem qualquer opção para selecionar.
* **Nota sobre o LATTICE**: Não existe nenhum modelo «Texture Aware» treinado para o LATTICE, e o pipeline impõe o demosaico padrão para os fotogramas LATTICE, independentemente do valor armazenado. Se adicionar uma pasta LATTICE a um projeto que já tivesse a opção «Texture Aware» selecionada, oChloros

reescreve a configuração para «Standard», em vez de deixar um valor desatualizado em `project.json`.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3 600 segundos
* **Predefinição**: 0 segundos
* **Descrição**: Define o intervalo de tempo mínimo (em segundos) entre a utilização de alvos de calibração. Quando definido como 0, oChloros

utilizará todos os alvos de calibração detetados. Quando definido com um valor superior, oChloros

utilizará apenas alvos de calibração separados por, pelo menos, esse número de segundos, reduzindo o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de calibração.
* **Quando ajustar**:
  * Defina como 0 para obter a máxima precisão de calibração quando as condições de iluminação variam
  * Aumente (por exemplo, para 60-300 segundos) para um processamento mais rápido quando a iluminação for consistente e houver imagens frequentes de alvos de calibração

### Desvio do fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: Especifica o desfasamento de fuso horário (em horas em relação ao UTC) para os carimbos de data e hora dos dados do sensor de luz, utilizado ao fazer a correspondência entre os registos do sensor de luz e as horas de captura das imagens. As gravações mais recentes do `.daq` incluem a sua própria proveniência de fuso horário, pelo que isto é necessário principalmente para registos mais antigos gravados na hora local.

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Predefinição**: Desativado (desmarcado)
* **Descrição**: Ativa a utilização de correções cinemáticas pós-processadas (PPK) provenientes de gravadores DAQMAPIR

que incluam um GPS (GNSS). Quando ativada, a funcionalidade «Chloros

» utilizará quaisquer ficheiros de registo .daq que contenham dados de pinos de exposição no diretório do seu projeto e aplicará correções precisas de geolocalização às suas imagens.
* **Requisito**: Deve existir um ficheiro de registo .daq com entradas de pino de exposição no diretório do seu projeto
* **Quando ativar**: Recomenda-se ativar sempre a correção PPK se tiver entradas de feedback de exposição no seu ficheiro de registo .daq.

### Pino de exposição 1

* **Tipo**: Seleção por menu suspenso
* **Visibilidade**: Apenas visível quando a opção «Aplicar correções PPK» estiver ativada E houver dados de exposição disponíveis para o Pino 1
* **Opções**:
  * Nomes de modelos de câmara detetados no projeto
  * «Não utilizar» - Ignorar este pino de exposição
* **Predefinição**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao Pino de exposição 1 para sincronização temporal PPK. O pino de exposição regista o momento exato em que o obturador da câmara é acionado, o que é fundamental para uma geolocalização PPK precisa.
* **Comportamento da seleção automática**:
  * Uma câmara + um pino: Seleciona automaticamente a câmara
  * Uma câmara + dois pinos: O pino 1 é automaticamente atribuído à câmara
  * Várias câmaras: É necessária uma seleção manual

### Pino de exposição 2

* **Tipo**: Seleção por menu suspenso
* **Visibilidade**: Apenas visível quando a opção «Aplicar correções PPK» está ativada E existem dados de exposição disponíveis para o pino 2
* **Opções**:
  * Nomes de modelos de câmara detetados no projeto
  * «Não utilizar» — Ignora este pino de exposição
* **Predefinição**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao pino de exposição 2 para sincronização temporal PPK ao utilizar uma configuração de duas câmaras.
* **Comportamento da seleção automática**:
  * Câmara única + pino único: o Pino 2 é automaticamente definido como «Não utilizar»
  * Câmara única + dois pinos: o Pino 2 é automaticamente definido como «Não utilizar»
  * Várias câmaras: é necessária uma seleção manual
* **Nota**: A mesma câmara não pode ser atribuída simultaneamente ao Pino 1 e ao Pino 2.***

## Sensor de Luz DAQ

Esta secção aparece nas Definições do Projeto e lista todos os ficheiros de luz descendente DAQ do projeto — gravações `.daq` e registos de luz descendente DAQ-M `.csv`. As gravações efetuadas no separador «Sensores de Luz» são adicionadas automaticamente ao projeto aberto.



<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Cada linha apresenta o ficheiro, o modelo do sensor e a correção da tampa difusora efetivamente em vigor para esse ficheiro. Acima das linhas encontra-se um único controlo válido para todo o projeto:

### Substituição da tampa (todos os ficheiros)

* **Tipo**: Seleção por menu suspenso
* **Opções**: `Auto`, além dos perfis de correção da tampa válidos para os tipos de sensores presentes no projeto
* **Predefinição**: Automático
* **Guardado como**: `Processing → "DAQ cap id"` (predefinição `auto`)
* **Descrição**: O `Auto` utiliza o cap registado em cada ficheiro (presume-se o cap «Sunshine» quando nada foi registado — todos os sistemas de aquisição de dados (DAQ) daMAPIR

são fornecidos com o corretor «Sunshine»). A seleção de um cap específico substitui **todos os** ficheiro de radiação descendente do projeto: as gravações em bruto são corrigidas com ele e as gravações que já incluem um cap são re-referenciadas (a correção registada é anulada e a selecionada é aplicada).
* **Importante**: O cap selecionado deve corresponder ao cap que foi fisicamente colocado durante a gravação. Nem o sensor nem o software conseguem detetar a tampa física — um ID de tampa incorreto corrige os espectros de forma errada.

Existe deliberadamente **um** controlo para todo o projeto, em vez de menus suspensos por ficheiro: a configuração aplica-se a todas as fontes de luz descendente no projeto.***

## Alinhamento de matrizes

Esta secção aparece **apenas** quando pelo menos uma imagem no projeto contém a transformação de alinhamento módulo a módulo que as matrizes LATTICE aplicam no momento da captura (etiquetas XMP `Chloros:Alignment*`). Mostra quantas imagens contêm etiquetas de alinhamento, qual a câmara que serve de referência (identificador `REF`) e uma tabela por câmara com o número de imagens.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Aplicar alinhamento de matrizes

* **Tipo**: Caixa de seleção
* **Predefinição**: Ativado (marcado)
* **Armazenado como**: `Processing → "Array alignment"`
* **Descrição**: Distorce todos os produtos processados (debayering / pré-visualização / radiação / refletância / índice) para a geometria de referência partilhada da matriz, utilizando a transformação aplicada no momento da captura. Desativado = exportação na geometria nativa de cada sensor.

### Recortar para sobreposição comum

* **Tipo**: Caixa de seleção (apenas ativa enquanto *Aplicar alinhamento da matriz* estiver ativado)
* **Predefinição**: Ativado (marcado)
* **Armazenado como**: `Processing → "Array alignment crop"`
* **Descrição**: Recorta as exportações alinhadas para a região partilhada por todos os módulos da câmara, de modo a que todas as bandas tenham a mesma área de cobertura. Desativado mantém a área total do sensor (preenchimento a preto fora da fonte).

### Reamostragem

* **Tipo**: Menu suspenso (apenas ativo enquanto a opção *Aplicar alinhamento da matriz* estiver ativada)
* **Opções**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Predefinição**: Bilinear
* **Armazenado como**: `Processing → "Array alignment interpolation"`
* **Descrição**: Interpolação utilizada pela distorção de alinhamento. A opção «Mais próximo» preserva os valores exatos da fonte (sem mistura entre píxeis) para uma análise radiométrica rigorosa; a opção «Bilinear» é a mais adequada para mapeamento e utilização visual.

As mesmas três opções existem sem prefixo como `chloros-cli process --array-alignment`, `--array-alignment-crop` e `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Índice

Estas definições permitem-lhe configurar índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel de configuração de índices especiais
* **Descrição**: Abre um painel interativo onde pode selecionar e configurar índices multiespectrais de vegetação (NDVI

,NDRE

,EVI

, etc.) para calcular durante o processamento da imagem. Pode adicionar vários índices, cada um com as suas próprias definições de visualização.
* **Índices disponíveis**: O menu suspenso da interface gráfica inclui**27** fórmulas de índices multiespectrais predefinidas (consulte [Fórmulas de Índices Multiespectrais](multispectral-index-formulas.md) para consultar a lista completa, incluindo os nomes que também são aceites pela opçãoCLI

/SDK

`--indices`).
* **Funcionalidades**:
  * Selecione entre fórmulas de índices predefinidas
  * Arraste os canais de filtro da sua câmara para as ranhuras de banda da fórmula
  * Configure gradientes de cor para visualização (LUT - Tabelas de Consulta)
  * Defina valores-limiar e modos de recorte
  * Crie fórmulas de índices personalizadas
* **Nota**: Os índices não são calculados para câmaras monocromáticas LATTICE M3M de banda única — os índices multibanda não são definidos numa única banda. As câmarasSurvey3

e LATTICE M3C não são afetadas.



<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Cada índice que adicionar apresenta a sua fórmula como expressão matemática, com um círculo colorido por slot de banda: vermelho =Red

, verde =Green

, azul =Blue

, laranja =Orange

, ciano =Cyan

, roxo =NIR

, magenta = RE. Arraste um círculo da linha acima da fórmula para um espaço para o associar; clique duas vezes num espaço associado para o limpar. O índice só é calculado uma vez, desde que todos os espaços utilizados pela fórmula tenham um canal.

### Fórmulas personalizadas (FuncionalidadeChloros

+)

* **Tipo**: Matriz de definições de fórmulas personalizadas
* **Disponibilidade**: Requer o início de sessão com uma subscriçãoChloros

+ elegível.
* **Descrição**: Permite criar e guardar fórmulas de índices multiespectrais personalizadas utilizando cálculos de bandas. As fórmulas personalizadas são guardadas com as definições do seu projeto e podem ser utilizadas tal como os índices integrados.
* **Como criar**:
  1. No painel de configuração do Índice, abra a calculadora de fórmulas personalizadas
  2. Escreva a fórmula utilizando os **símbolos de banda**, não os nomes das bandas
  3. Guarde a fórmula com um nome descritivo — esta aparecerá então na parte inferior do menu suspenso de fórmulas, e poderá arrastar os círculos dos canais da sua câmara para os respetivos slots, exatamente como uma predefinição integrada
* **Sintaxe da fórmula**:
  * Espaços de banda: `x`, `y`, `z`, `a`, `b`, `c` — seis posições que pode mapear para canais reais arrastando-as
  * Operadores: `+`, `-`, `*`, `/`, `^` e `()` para agrupamento
  * Funções: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Porquê símbolos e não nomes de bandas**: uma fórmula escrita como `(y-x)/(y+x)` funciona em qualquer câmara, porque o mapeamento por arrastar e largar determina se `y` corresponde aoNIR

de 850 nm de um filtroRGN

ou aoNIR

de 808 nm de um filtroOCN

. As predefinições integradas são armazenadas da mesma forma — consulte [Fórmulas de Índices Multiespectrais](multispectral-index-formulas.md) para conhecer a forma exata dos símbolos de todas as 27.
* **Onde funcionam**: as fórmulas personalizadas são guardadas com as definições do projeto e podem ser utilizadas na [Área de Testes de Índices/LUT](../image-viewer-gui/index-lut-sandbox.md), bem como no processamento.**Não** são aceites pela lista de nomesCLI

/SDK

`--indices`, que apenas expande os 22 nomes de predefinições integradas.

***

## Exportação

Estas definições controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrada

* **Tipo**: Seleção no menu suspenso
* **Opções**:
  * **TIFF

(16 bits)** - FormatoTIFF

de 16 bits não comprimido
  * **TIFF

(32 bits, Porcentagem)** -TIFF

de 32 bits em ponto flutuante com valores de refletância expressos em percentagens
  * **PNG

(8 bits)** - Formato «PNG

» comprimido de 8 bits
  * **JPG (8 bits)** - Formato «JPEG

» comprimido de 8 bits
* **Padrão**: «TIFF

» (16 bits)
* **Descrição**: Seleciona o formato de ficheiro para guardar imagens processadas e calibradas. As imagens exportadas são guardadas numa subpasta específica para cada formato, dentro da pasta de cada câmara (`tiff16`, `tiff32`, `png8`, `jpg8`), com uma pasta `<Product>_Images/` por produto. Os ficheiros exportados mantêm o nome do ficheiro de origem — é a pasta, e não um sufixo do nome do ficheiro, que identifica o produto.
* **Recomendações de formato**:
  * **TIFF

(16 bits)**: Recomendado para análises científicas e fluxos de trabalho profissionais. Preserva a máxima qualidade dos dados sem artefactos de compressão. Ideal para análise multiespectral e processamento posterior em software SIG.
  * **TIFF

(32 bits, Porcentagem)**: Ideal para fluxos de trabalho que requerem valores de refletância expressos em percentagem (0-100%). Oferece a máxima precisão para medições radiométricas.
  * **PNG

(8 bits)**: Adequado para visualização na Web e visualização geral. Ficheiros de menor tamanho com compressão sem perdas, mas com gama dinâmica reduzida.
  * **JPG (8 bits)**: Ficheiros de menor tamanho, ideal apenas para pré-visualizações e exibição na Web. Utiliza compressão com perdas, o que não é adequado para análises científicas.
* **Nota**: A radiancia do LATTICE é sempre exportada comoTIFF

de 32 bits com precisão flutuante, independentemente desta configuração.

***

## Guardar modelo de projeto

Esta funcionalidade permite-lhe guardar as configurações atuais do seu projeto como um modelo reutilizável.

* **Tipo**: Campo de texto + botão «Guardar»
* **Descrição**: Introduza um nome descritivo para o seu modelo de definições e clique no ícone «Guardar». O modelo irá armazenar todas as definições atuais do seu projeto (detecção de alvos, opções de processamento, índices, e formato de exportação) para facilitar a reutilização em projetos futuros. Os modelos são armazenados na pasta `Project Templates/`, dentro da pasta de gravação do seu projeto, e também podem ser selecionados ou exportados a partir do menu principal (*Selecionar modelo* / *Guardar modelo* / *Exportar modelo*).
* **Casos de utilização**:
  * Criar modelos para diferentes sistemas de câmaras (RGB

, multiespectral,NIR

)
  * Guardar configurações padrão para tipos específicos de culturas ou fluxos de trabalho de análise
  * Partilhar definições consistentes entre toda a equipa
* **Como utilizar**:
  1. Configure todas as definições desejadas para o projeto
  2. Introduza um nome para o modelo (por exemplo, «RedEdge

Survey3

NDVI

Standard»)
  3. Clique no ícone «Guardar»
  4. O modelo pode agora ser carregado ao criar novos projetos

***

## Guardar pasta do projeto

Esta definição especifica onde os novos projetos são guardados por predefinição.

* **Tipo**: Exibição do caminho do diretório + botão «Editar»
* **Predefinição (Windows

)**: `C:\Users\[Username]\Chloros Projects`
* **Predefinição (Linux

)**: `~/Chloros Projects`
* **Descrição**: Mostra o diretório predefinido atual onde são criados novos projetos «Chloros

». Clique no ícone de edição para selecionar um diretório diferente. A alteração é armazenada como uma única linha de texto no ficheiro `~/.chloros/working_directory.txt` — num «Windows

», ou seja, o ficheiro `C:\Users\<Username>\.chloros\working_directory.txt`. Se esse ficheiro estiver em falta ou indicar um caminho que já não existe, oChloros

recorre ao valor predefinido acima referido. OCLI

lê e grava o mesmo ficheiro, pelo que o `chloros-cli` e a interface gráfica (GUI) estão sempre em sintonia quanto à localização dos projetos.
* **Os modelos de projeto** encontram-se numa subpasta `Project Templates/` deste diretório.
* **Quando alterar**:
  * Defina para uma unidade de rede para colaboração em equipa
  * Mude para uma unidade com mais espaço de armazenamento para conjuntos de dados de grande dimensão
  * Organize os projetos por ano, cliente ou tipo de projeto em pastas diferentes
* **Nota**: A alteração desta definição afeta apenas os NOVOS projetos. Os projetos existentes permanecem nas suas localizações originais.***

## Persistência das definições

Um projeto «Chloros

» é uma **pasta**. Todas as definições do projeto são guardadas no ficheiro `project.json` dentro dessa pasta; o hardware ligado é memorizado juntamente com elas nos ficheiros `cameras.json` e `sensors.json`, pelo que reabrir um projeto também restabelece a ligação às suas câmaras e sensores de luz. Ao reabrir um projeto, todas as definições são restauradas exatamente como as deixou. Os projetos guardados também podem ser controlados sem interface gráfica através de `chloros-cli project` ou do `open_project` doSDK

.

### Hierarquia das definições

As definições são aplicadas na seguinte ordem:

1. **Predefinições do sistema** - Predefinições integradas definidas peloChloros

2. **Definições do modelo** - Se carregar um modelo ao criar um projeto
3. **Definições do projeto guardado** - Definições guardadas com o ficheiro do projeto
4. **Ajustes manuais** - Quaisquer alterações que efetue durante a sessão atual

### Definições e processamento de imagens

As definições de processamento são lidas quando se inicia uma execução de processamento. Alterar uma definição não altera retroativamente os produtos que já se encontram no disco — execute o processamento novamente para aplicar as novas definições. Algumas definições nunca afetam o processamento:

* Resolução da miniatura da imagem (apenas para visualização)
* Guardar modelo do projeto
* Guardar pasta do projeto

***

## Referência das chaves de configuração

Para automatização (CLI

`--config`,SDK

`configure` ou leitura direta de `project.json`), estas são as chaves exatas em `Project Settings`:

| Caminho da chave | Tipo | Padrão |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | número entre 0 e 10 000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | número entre 0 e 100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | booleano | `true` |
| `Processing → Export sensor response` | booleano | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Padrão |
| `Processing → Minimum recalibration interval` | número 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | número -12..12 | `0` |
| `Processing → Apply PPK corrections` | booleano | `false` |
| `Processing → DAQ cap id` | ID do perfil de limite ou `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista de configurações de índice | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

As chaves `Array alignment` são gravadas na primeira vez que a secção «Alinhamento de Matriz» é renderizada ou quando uma chamada de automação as define. Enquanto estiverem ausentes, o pipeline utiliza os mesmos valores apresentados acima (`true`, `true`, bilinear), pelo que um projeto.json sem elas se comporta de forma idêntica a um que as inclua.

### Chaves armazenadas em `project.json` sem controlo no painel de definições

Estas encontram-se na mesma árvore `Project Settings` e são lidas pelo processamento, mas não encontrará um widget para elas na barra lateral:

| Caminho da chave | Tipo | Padrão | Definido por |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`,SDK

`input_level=`. Substitui a forma como os ficheiros TIFF de entrada do LATTICE são interpretados; o `auto` deduz a partir da etiqueta XMP de cada ficheiro, juntamente com a contagem de canais. Ignorado para capturas do tipo «Survey3

» `.raw`. Não é deliberadamente uma configuração da GUI — a opção «auto» é a correta em todos os casos normais. |
| `Processing → Target reflectance dir` | cadeia de caracteres do caminho | `""` | `chloros-cli process --target-reflectance-dir`, ou o alvo do projetoAPI

|
| `Processing → Target reflectance config` | dicionário indexado pelo número de série da câmara | `{}` | Registo de um alvo no quadro (modo `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | cadeia de caracteres do caminho | `""` |SDK

`process_folder(daq_log_path=…)`. Aponta para uma gravação `.daq` ou para uma pasta com várias gravações |
| `Target Detection → Minimum calibration target squares` | número | `4` | Predefinição antiga; sem controlo e sem o sinalizador «CLI

» |
| `UI → Grid thumbnail size` | número | `160` | O próprio controlo deslizante de zoom das miniaturas da grelha de imagens |

Duas preferências do visualizador são armazenadas **ao nível superior em `project.json`**, totalmente fora de `Project Settings`, porque se tratam de estados de visualização e não de definições de processamento:

| Caminho-chave | Tipo | Padrão | Definido por |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | número inteiro 1–256 | `1` | O controlo GSD (px) do separador de imagem — ver [Abrir uma imagem em ecrã inteiro](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Melhores práticas

1. **Comece com as predefinições**: As predefinições funcionam bem para a maioria dos sistemas de câmarasMAPIR

e fluxos de trabalho típicos.
2. **Crie modelos**: Depois de otimizar as definições para um fluxo de trabalho ou câmara específicos, guarde-as como um modelo para garantir a consistência entre projetos.
3. **Teste antes do processamento completo**: Ao experimentar novas definições, teste-as num pequeno subconjunto de imagens antes de processar todo o seu conjunto de dados.
4. **Documente as suas definições**: Utilize nomes de modelos descritivos que indiquem o sistema de câmara, o tipo de processamento e a utilização pretendida (por exemplo, «Survey3

_RGB_NDVI_Agricultura»).
5. **Seleção do formato de exportação**: Escolha o formato de exportação com base na utilização final:
   * Análise científica →TIFF

(16 bits ou 32 bits)
   * Processamento SIG →TIFF

(16 bits)
   * Visualização rápida →PNG

(8 bits)
   * Partilha na Web → JPG (8 bits)

***

Para mais informações sobre índices multiespectrais no «Chloros

», consulte a página [Fórmulas de índices multiespectrais](multispectral-index-formulas.md).
