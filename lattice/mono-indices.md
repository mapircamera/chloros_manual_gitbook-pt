# Câmaras monocromáticas e índices de vegetação

## Uma câmara = uma banda

Uma câmara **M3M**é a versão monocromática da**M3C** da Bayer: um sensor IMX265 monocromático equipado com um único filtro de interferência de banda estreita. A sequência do modelo identifica a banda — `M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685` (exibida no Chloros como `LATT-M3M-L87-F685`). O sensor fornece uma **única banda em escala de cinzentos** sem mosaico Bayer: não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de brancos para definir.

Consequências que vale a pena conhecer antes de planear um sistema monocromático:

* **A radianância e a refletância são totalmente definidas por banda.**Trata-se de mapas radiométricos por banda, pelo que uma câmara M3M produz radianância calibrada em float32 (W/m²/sr/nm) e refletância em uint16 (`32768` = ρ 1,0) exatamente como o faz uma banda M3C. Os fotogramas monocromáticos contêm uma matriz de resposta do sensor**identitária** — não é necessária nem aplicada qualquer desmistura 3×3.
* **Uma única câmara monocromática não consegue produzir um índice de vegetação.** O NDVI, o NDRE e outros índices semelhantes requerem, pelo menos, duas bandas. Para calcular índices a partir de hardware monocromático, é necessário combinar várias câmaras M3M — ver abaixo.
* As câmaras M3M transmitem **Mono12** (12 bits: 2 bytes/pixel na transmissão, ou 1,5 bytes/pixel quando um conjunto de câmaras utiliza [transporte de 12 bits compactado](arrays.md#packed-12-bit-transport)), o que é relevante para [o planeamento da largura de banda do conjunto de câmaras](arrays.md#bandwidth-how-many-cameras-fit).

## O que o Chloros ignora no modo mono — e como o indica

As etapas do pipeline de cor simplesmente não se aplicam a um sensor de banda única. O Chloros **ignora-as com uma mensagem de uma linha** em vez de apresentar um erro, e continua a executá-las normalmente para qualquer câmara M3C (Bayer) na mesma sessão:

| Etapa | Comportamento Mono (M3M) | Comportamento M3C |
| --- | --- | --- |
| Demosaico / debayer | Ignorado — o nível de exportação `debayered` é uma imagem em escala de cinzentos de 1 canal. | Demosaico de 3 canais. |
| Equilíbrio de brancos (`lattice white-balance`) | Ignorado com uma mensagem de uma linha. | Executa-se normalmente. |
| Perfil de cor (`lattice color-profile`) | Ignorado com uma mensagem de uma linha. | Executa-se normalmente. |
| Saturação/contraste (`lattice color`) | Ignorado com uma mensagem de uma linha. | Executa-se normalmente. |
| Separação de interferência espectral | Identidade (sem matriz 3×3). | Matriz 3×3 aplicada por câmara. |
| Radiação / refletância | **Executa** — por banda, totalmente calibrado. | Executa por banda. |

A GUI aplica o mesmo filtro: para uma câmara monocromática, o painel de definições por câmara oculta as linhas exclusivas de RGB (Equilíbrio de brancos, Gama, Perfil de cor, Saturação, Contraste, divisões de canais), e o histograma em tempo real fica bloqueado num único traço **MONO**. O discriminador em toda a pilha é o token `M3M` na cadeia de caracteres do modelo, apresentado na GUI/no SDK como `is_mono`.

## Os índices requerem ≥ 2 bandas: alinhar → empilhar → indexar

O fluxo de trabalho do índice mono consiste sempre nas mesmas três etapas:

1. **Alinhar** — apontar várias câmaras M3M para diferentes comprimentos de onda (por exemplo, uma F650 «Red» e uma F850 «NIR»), ligá-las como um [conjunto de múltiplas câmaras](arrays.md) e deixe o Chloros calcular a distorção de co-registo entre as câmaras.
2. **Empilhar** — os fotogramas alinhados tornam-se uma imagem multibanda (cada câmara contribui com uma banda nomeada).
3. **Índice** — avalia uma fórmula de índice sobre as bandas da pilha, renderizando-a opcionalmente através de uma LUT.

Na interface gráfica, toda esta cadeia corresponde ao modo de visualização de matriz **Câmaras Combinadas**: a composição em tempo real já está alinhada e a Calculadora de Índice da matriz (abaixo) define a fórmula que é renderizada. As exportações capturadas podem ser distorcidas para o mesmo alinhamento com a opção de captura**Alinhado**.

## A Calculadora de Índice

A Calculadora de Índice cria a expressão de índice utilizada pela visualização em tempo real e pelas exportações de índice por câmara. Trata-se de uma superfície partilhada, acedida a partir de dois locais na barra lateral do separador «Câmaras»:

* **Por câmara**— Pré-visualização em tempo real → ícone de engrenagem**Índice** (apenas câmaras Bayer RGN/OCN/NGB; uma câmara mono isolada não tem controlo de índice, pois uma única banda não permite criar um índice).
* **Por matriz**— definições da matriz → Pré-visualização em tempo real → engrenagem**Índice**. Este é o caminho para o modo monocromático: a lista de bandas abrange**todas as câmaras do conjunto**, pelo que um par monocromático contribui com as suas duas bandas aqui.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Os seus controlos, de cima para baixo:

* **Chips de banda** («Bandas — clicar para adicionar à expressão») — um botão por banda disponível, rotulado com o nome da cor + comprimento de onda em nm (nomes de cores duplicados são diferenciados como, por exemplo, «Cor 850»). Ao clicar, insere-se o token da banda na posição do cursor. As bandas de câmaras que não conseguem produzir radiação por banda (RGB/FRGB) são filtradas.
* **Botões de operadores e funções** — `+ - * / ( ) ^ ,` e `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Área de texto de expressão** — fórmula de introdução livre; o espaço reservado mostra a forma clássica do NDVI: `(NIR - Red) / (NIR + Red)`. Uma pré-visualização tokenizada, apenas para leitura, acima desta, apresenta os chips de banda, números e sinalizadores como tokens desconhecidos.
* **Banner de validade**— cinzento «Vazio — não será aplicado nenhum índice»; verde «Expressão válida»; vermelho com o erro de análise específico (banda desconhecida, banda ambígua captada por várias câmaras, parênteses em falta, …); ou âmbar quando a expressão é válida, mas a**constante** (por exemplo, `X/X`, ou um denominador NDVI digitado como `−` em vez de `+`) — uma constante mapeia todo o fotograma para uma única cor.
* Aparece um aviso âmbar separado se a expressão aplicada estiver correta, mas o **quadro em tempo real for uniforme** (cena plana ou saturada) — a colapsação do histograma é detetada automaticamente.
* **Aplicar LUT**(ativado por predefinição; desativado = alongamento da escala de cinzentos),**Nível**de 2/3/5/7 stops (predefinição: 7 stops) e entradas**Mín. / Máx.**que ladeiam a barra de gradiente. O valor predefinido para Mín. é**0,2**— amplia a rampa de cores para a gama relevante para a vegetação, enquanto valores inferiores são apresentados em escala de cinzentos; defina Mín. para −1 para obter a gama completa do índice (o botão**Reiniciar** restaura o intervalo de −1…+1). O valor predefinido para Máx. é 1.
* **Histograma em tempo real** da distribuição do índice — barras com escala de raiz quadrada, linhas âmbar dos percentis p2/p98, uma linha mediana branca e indicações das extremidades fora do intervalo («◀ N% < lo» / «hi < N% ▶") que ficam âmbar acima de 1 %, servindo de indicação para alargar a janela Mín./Máx.
* **Aplicar** aplica a expressão ao fluxo em tempo real; os ajustes da LUT são aplicados em tempo real sem necessidade de premir «Aplicar». Com um projeto aberto, a expressão e as definições da LUT são guardadas com o projeto e restauradas quando o reabrir; sem projeto, permanecem apenas durante a sessão.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## O caminho da CLI



A mesma cadeia alinhar → empilhar → indexar, programável de ponta a ponta:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` utiliza uma matriz conectada que contém as suas câmaras, ou abre-as ele próprio para a execução única. O `index --live` abre sempre as câmaras por si próprio (uma [matriz transitória](../reference/cli-reference.md#multi-camera-transient-sessions)) e não possui o `--serials`: abre todas as câmaras na rede. Por isso, liberte todas as câmaras que o Chloros mantém antes disso — a GUI, o `cam-connect`, o `array-connect` (`chloros-cli lattice array-disconnect --all`): abrir uma câmara retida uma segunda vez pode interromper o fluxo do seu detentor ou causar uma falha no backend. Uma câmara que não transmita nenhum fotograma faz com que a execução falhe, e nenhuma pilha ou índice é gravado.

O `--channel` mapeia os símbolos de uma predefinição para os nomes das bandas da pilha. Duas regras evitam que a execução falhe:

* **Os símbolos distinguem maiúsculas de minúsculas** e devem corresponder exatamente aos nomes dos canais da predefinição — as predefinições utilizam minúsculas (as do NDVI são `red`,`nir`; verifique `--list-presets`). `--channel red=Red_660` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.
* O lado da banda deve indicar o nome de uma banda na pilha alinhada (`lattice align-info --profile align.json` lista-as). O modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`.

O `lattice index` também funciona offline com um ficheiro TIFF multibanda alinhado guardado. Cada formulário é executado no backend, pelo que este deve estar em execução e com sessão iniciada num plano pago do Chloros+; um `--input` ou `--output` relativo é resolvido em relação à pasta a partir da qual executa o comando:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Predefinições de índice

O `lattice index --preset` (e a opção [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) do separador Imagem, que utiliza o mesmo motor) inclui estas **22 predefinições**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Execute o comando `chloros-cli lattice index --list-presets` para ver a fórmula e os símbolos dos canais de cada predefinição, e o comando `--list-gradients` para ver os gradientes de cor disponíveis. As fórmulas personalizadas utilizam o comando `--formula EXPR` com a mesma sintaxe que a Calculadora de Índices. Note que esta lista de predefinições é específica do motor de índices LATTICE — o menu suspenso de processamento das «Definições do Projeto» para imagens importadas apresenta uma lista diferente (consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md)).

O conjunto completo de sinalizadores (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, os controlos de alinhamento e distorção para `--live` e muito mais) está documentado na [Referência da CLI § Matemática de Índices/Vegetação](../reference/cli-reference.md#index--vegetation-maths); os equivalentes do SDK encontram-se na [Referência do SDK](../reference/sdk-reference.md).

## Captura de produtos de índice a partir de uma matriz mono

Com uma matriz ligada e uma expressão de índice aplicada, o `array-capture` (ou a opção da GUI **Capturar Tudo**) guarda os níveis de exportação por câmara *e* a renderização do índice — o `--index`/`--no-index` alterna esta opção na CLI e, por predefinição, captura todos os níveis aplicáveis. A contribuição de uma câmara mono para cada grupo de captura é a sua única banda nos níveis bruto/despixelação (escala de cinzentos)/radiação/refletância, além da composição de índice combinado partilhada quando a matriz funciona no modo combinado. Consulte [Matrizes Multicâmara § Captura](arrays.md#capturing-monitoring-vs-analysis).
