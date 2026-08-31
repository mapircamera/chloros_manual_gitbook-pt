# Câmaras monocromáticas e índices de vegetação

## Uma câmara = uma banda

A câmara **M3M**é a versão monocromática da**M3C**da Bayer: um sensor IMX265 monocromático equipado com um único filtro de interferência de banda estreita. A sequência do modelo identifica a banda — `M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685` (exibida em Chloros como `LATT-M3M-L87-F685`). O sensor fornece uma**única banda em escala de cinzentos** sem mosaico de Bayer: não há nada para desmosaicar, nenhum crosstalk entre canais para separar e nenhum equilíbrio de brancos para definir.

Consequências que vale a pena conhecer antes de planear um sistema monocromático:

* **A radiancia e a refletância são totalmente definidas por banda.**Trata-se de mapas radiométricos por banda, pelo que uma câmara M3M produz radiancia calibrada em float32 (W/m²/sr/nm) e refletância em uint16 (`32768` = ρ 1,0) exatamente como o faz uma banda M3C. Os fotogramas monocromáticos contêm uma matriz de resposta do sensor**identitária** — não é necessária nem aplicada qualquer desmistura 3×3.
* **Uma única câmara monocromática não consegue produzir um índice de vegetação.** O NDVI, o NDRE e similares requerem, pelo menos, duas bandas. Para calcular índices a partir de hardware monocromático, é necessário combinar várias câmaras M3M — ver abaixo.
* As câmaras M3M transmitem **Mono12** (12 bits, 2 bytes/pixel na transmissão), o que é importante para o [orçamento da largura de banda da matriz](arrays.md#bandwidth-the-rules-of-thumb).

## O que o Chloros ignora no modo mono — e como o indica

As etapas do pipeline de cor simplesmente não se aplicam a um sensor de banda única. O Chloros **ignora-as com uma mensagem de uma linha** em vez de apresentar um erro, e continua a executá-las normalmente para qualquer câmara M3C (Bayer) na mesma sessão:

| Etapa | Comportamento Mono (M3M) | Comportamento M3C |
| --- | --- | --- |
| Demosaico / debayer | Ignorado — o nível de exportação do `debayered` é uma imagem em escala de cinzentos de 1 canal. | Demosaico de 3 canais. |
| Equilíbrio de brancos (`lattice white-balance`) | Ignorado com uma mensagem de uma linha. | Executa-se normalmente. |
| Perfil de cor (`lattice color-profile`) | Ignorado com uma mensagem de uma linha. | Executa-se normalmente. |
| Saturação/contraste (`lattice color`) | Ignorado com uma mensagem de uma linha. | Funciona normalmente. |
| Desmistura de interferência espectral | Identidade (sem matriz 3×3). | Matriz 3×3 aplicada por câmara. |
| Radiação / refletância | **Executa** — por banda, totalmente calibrado. | Executa por banda. |

A interface gráfica aplica o mesmo filtro: para uma câmara monocromática, o painel de definições por câmara oculta as linhas exclusivas do RGB (Equilíbrio de brancos, Gama, Perfil de cor, Saturação, Contraste, divisões de canais), e o histograma em tempo real fica bloqueado num único traço **MONO**. O discriminador ao longo de toda a pilha é o token `M3M` na cadeia de caracteres do modelo, apresentado na GUI/SDK como `is_mono`.

## Os índices requerem ≥ 2 bandas: alinhar → empilhar → indexar

O fluxo de trabalho do índice mono consiste sempre nas mesmas três etapas:

1. **Alinhamento** — apontar várias câmaras M3M para diferentes comprimentos de onda (por exemplo, uma F650 «Red» e uma F850 «NIR»), ligá-las como um [conjunto de múltiplas câmaras](arrays.md) e deixar que o Chloros calcule a distorção de co-registo entre as câmaras.
2. **Pilha** — os fotogramas alinhados tornam-se uma imagem multibanda (cada câmara contribui com uma banda nomeada).
3. **Índice** — avalia-se uma fórmula de índice sobre as bandas da pilha, renderizando-a opcionalmente através de uma LUT.

Na interface gráfica, toda esta cadeia corresponde ao modo de exibição da matriz **Câmaras Combinadas**: a composição em tempo real já está alinhada, e a Calculadora de Índice da matriz (abaixo) define a fórmula que é renderizada. As exportações capturadas podem ser ajustadas para o mesmo alinhamento com a opção de captura**Alinhada**.

## A Calculadora de Índice

A Calculadora de Índice cria a expressão de índice utilizada pela visualização em tempo real e pelas exportações de índice por câmara. Trata-se de uma superfície partilhada, que pode ser aberta a partir de dois locais na barra lateral do separador «Câmaras»:

* **Por câmara**— Pré-visualização em tempo real → roda dentada**Índice** (apenas câmaras Bayer RGN/OCN/NGB; uma câmara mono isolada não tem controlo de índice, uma vez que uma única banda não permite criar um índice).
* **Por matriz**— definições da matriz → Pré-visualização em tempo real → roda dentada**Índice**. Este é o caminho mono: a lista de bandas abrange**todas as câmaras que integram a matriz**, pelo que um par mono contribui aqui com as suas duas bandas.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Os seus controlos, de cima para baixo:

* **Chips de banda** («Bandas — clicar para adicionar à expressão») — um botão por banda disponível, rotulado com o nome da cor + comprimento de onda em nm (nomes de cores duplicados são diferenciados como, por exemplo, «Cor 850»). Ao clicar, insere-se o token da banda na posição do cursor. As bandas de câmaras que não conseguem produzir radiação por banda (RGB/FRGB) são filtradas.
* **Botões de operadores e funções** — `+ - * / ( ) ^ ,` e `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Área de texto de expressão** — fórmula de digitação livre; o marcador de posição mostra a forma clássica do NDVI, ou seja, `(NIR - Red) / (NIR + Red)`. Uma pré-visualização tokenizada, apenas para leitura, acima dela, apresenta os chips de banda, números e sinalizadores como tokens desconhecidos.
* **Banner de validade**— cinzento «Vazio — não será aplicado nenhum índice»; verde «Expressão válida»; vermelho com o erro de análise específico (banda desconhecida, banda ambígua captada por várias câmaras, parêntese em falta, …); ou âmbar quando a expressão é válida mas**constante** (por exemplo, `X/X`, ou um denominador NDVI digitado com `−` em vez de `+`) — uma constante mapeia todo o fotograma para uma única cor.
* Aparece um aviso âmbar separado se a expressão aplicada estiver correta, mas o **quadro em tempo real for uniforme** (cena plana ou saturada) — a colapsão do histograma é detetada automaticamente.
* **Aplicar LUT**(ativado por predefinição; desativado = alongamento da escala de cinzentos),**Nível**de 2/3/5/7 stops (predefinição: 7 stops) e entradas**Mín. / Máx.**de ambos os lados da barra de gradiente. O valor predefinido para Mín. é**0,2**— amplia a rampa de cores para o intervalo relevante para a vegetação, enquanto valores inferiores são apresentados em escala de cinzentos; defina Mín. para −1 para obter o intervalo completo do índice (o botão**Reiniciar** restaura o intervalo de −1…+1). O valor predefinido para Máx. é 1.
* **Histograma em tempo real** da distribuição do índice — barras com escala de raiz quadrada, linhas de percentis p2/p98 em âmbar, uma linha mediana branca e indicações nas extremidades fora do intervalo («◀ N% &lt; lo» / «hi &lt; N% ▶&quot;) que ficam âmbar acima de 1 % como indicação para alargar a janela Mín./Máx.
* **Aplicar**submete a expressão ao fluxo em tempo real; os ajustes da LUT são aplicados em tempo real sem necessidade de premir «Aplicar». As expressões são deliberadamente**apenas para a sessão** — não são mantidas entre sessões.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## O percurso CLI

A mesma cadeia de alinhamento → pilha → índice, programável de ponta a ponta:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mapeia os símbolos de uma predefinição para os nomes das bandas da pilha. Duas regras evitam que a execução falhe:

* **Os símbolos distinguem maiúsculas de minúsculas** e têm de corresponder exatamente aos nomes dos canais da predefinição — as predefinições utilizam minúsculas (as do NDVI são `red`,`nir`; verifique `--list-presets`). `--channel red=Red_660` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.
* O lado da banda deve indicar o nome de uma banda na pilha alinhada (o `lattice align-info --profile align.json` lista-as). O modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`.

O `lattice index` também funciona totalmente offline com um TIFF multibanda alinhado guardado:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Predefinições de índice

O `lattice index --preset` (e a [Área de teste de índice/LUT](../image-viewer-gui/index-lut-sandbox.md) do separador Imagem, que utiliza o mesmo motor) inclui estas **22 predefinições**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Execute o `chloros-cli lattice index --list-presets` para ver a fórmula e os símbolos de canal de cada predefinição, e o `--list-gradients` para ver os gradientes de cor disponíveis. As fórmulas personalizadas utilizam o `--formula EXPR` com a mesma sintaxe que a Calculadora de Índice. Note que esta lista de predefinições é específica do motor de índices LATTICE — o menu suspenso de processamento em «Definições do Projeto» para imagens importadas apresenta uma lista diferente (consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md)).

O conjunto completo de sinalizadores (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, botões de alinhamento e distorção para `--live`, e outros) está documentado na [Referência do CLI § Índice / Matemática da Vegetação](../reference/cli-reference.md#index--vegetation-maths); os equivalentes do SDK encontram-se na [Referência do SDK](../reference/sdk-reference.md).

## Captura de produtos de índice a partir de uma matriz mono

Com uma matriz ligada e uma expressão de índice aplicada, o `array-capture` (ou a opção **Capturar Tudo** na GUI) guarda os níveis de exportação por câmara *e* a renderização do índice — o `--index`/`--no-index` ativa essa funcionalidade no CLI, e a captura inclui, por predefinição, todos os níveis aplicáveis. A contribuição de uma câmara mono para cada grupo de captura é a sua única banda nos níveis bruto/despixelação (escala de cinzentos)/radiação/refletância, além da composição de índice combinado partilhada quando a matriz funciona em modo combinado. Consulte [Matrizes de múltiplas câmaras § Captura](arrays.md#capturing-monitoring-vs-analysis).
