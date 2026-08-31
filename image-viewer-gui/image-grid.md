# Grelha de imagens

Depois de importar imagens para um projeto, irá vê-las organizadas numa grelha na área principal. É na grelha que escolhe **qual a versão de cada imagem que está a visualizar** — os botões acima dela alternam todas as miniaturas de uma só vez entre os ficheiros de origem e cada produto processado.

## Tamanho das miniaturas

Utilize o controlo deslizante de zoom no canto superior direito para ajustar o tamanho das miniaturas das imagens. O controlo deslizante varia entre **64 px e 1200 px**.

* **Ctrl + roda do rato** também permite redimensionar as miniaturas.
* **Ctrl + `+`**/**Ctrl + `=`**e**Ctrl + `−`** aumentam o tamanho em 4 px por pressão. O intervalo de teclas termina em 64 px no tamanho mínimo e, no tamanho máximo, no tamanho que couber exatamente duas miniaturas por linha na janela atual.
* O tamanho que definir é guardado com o projeto (`UI → Grid thumbnail size` em `project.json`, predefinição `160`), pelo que, ao reabrir o projeto, este é restaurado.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>A *resolução* das miniaturas é uma definição distinta do *tamanho* das miniaturas: consulte **Exibição → Resolução das miniaturas de imagem** em [Definições do projeto](../project-settings/project-settings.md) (padrão: 512 px na aresta mais longa). O tamanho refere-se à dimensão com que o mosaico é desenhado; a resolução refere-se à quantidade de detalhes que são obtidos para o preencher.***

## A barra de ferramentas da grelha

A linha de botões acima da grelha tem até três grupos, da esquerda para a direita:

1. **Por Disparador / Por Câmara** — modo de agrupamento. Aparece apenas em projetos que contenham capturas LATTICE.
2. **Botões de filtro de câmara** — um por câmara LATTICE. Aparecem apenas no modo Por Câmara.
3. **Botões de modo de exportação/visualização** — que produto cada miniatura mostra.

Quando a janela é demasiado estreita para os exibir todos, os grupos recolhem-se da direita para a esquerda, transformando-se em menus suspensos ao passar o cursor: os botões de exportação/visualização recolhem-se primeiro, seguidos dos botões de câmara. O grupo recolhido deixa um único botão com a opção atualmente ativa e, ao passar o cursor sobre ele, o conjunto completo desliza para baixo. **Os modos «Por disparador» e «Por câmara» nunca se recolhem.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Botões de exportação e visualização

Estes botões alternam as miniaturas da grelha entre tipos de imagem. **Um botão aparece assim que o produto a que se refere existir** — o que, no caso dos ficheiros de origem, significa imediatamente após a importação, e não após o processamento. O Chloros volta a digitalizar os produtos do projeto enquanto uma execução está em curso, pelo que os botões aparecem durante o processamento à medida que cada produto começa a ser gravado no disco.

### O botão base

O botão de exportação mais à esquerda está identificado com **o que importou efetivamente**:

| O que importou | Rótulo do botão |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Capturas LATTICE com uma pré-visualização no ecrã ao lado do fotograma RAW | `PNG` ou `TIFF`, consoante as pré-visualizações |
| Capturas do LATTICE em que o ficheiro base **é** o fotograma RAW | *sem botão* — o `RAW (Original)` já mostra esse ficheiro |

Num projeto misto, a etiqueta segue a extensão utilizada pela maioria das imagens.

### Botões do produto

| Botão | Mostra | Quando aparece |
| --- | --- | --- |
| **Alvos** | Imagens com um alvo de calibração detetado | Após uma execução que tenha detetado alvos |
| **Refletância** | Imagens de refletância calibradas | Apenas em projetos Survey3 — os projetos LATTICE utilizam `RAW (Reflectance)` em vez disso, pelo que a grelha nunca apresenta dois botões de refletância |
| **Equilíbrio de brancos** | O produto com equilíbrio de brancos (câmaras RGB) | Após o processamento |
| **Correção de vinheta** | A opção alternativa não calibrada com correção de vinheta | Após uma execução em que não foi possível aplicar a calibração de refletância e a *correção de vinheta* estava ativada |
| **Resposta do sensor** | A opção alternativa de resposta do sensor não calibrada | O mesmo, mas com a *correção de vinheta* desativada |
| **`RAW (<INDEX> Index)`** | Um botão por índice calculado | Após uma execução com índices configurados |
| **`<INDEX> LUT`** | Um botão por índice mapeado por cor | Após uma execução com uma LUT configurada |
| **`<Index> <Index\|LUT> <NNN>`** | Um botão por execução de exportação da [Área de teste de índices/LUT](index-lut-sandbox.md) | No momento em que uma exportação da área de teste termina |

### Botões de nível LATTICE

Os projetos que contêm capturas LATTICE incluem estes botões, rotulados com o nome do nível em vez do nome do produto:

| Botão | Nível |
| --- | --- |
| **RAW (Original)** | O fotograma RAW de origem, tal como foi importado |
| **RAW (Radiação)** | Radiação espectral em Float32, W/m²/sr/nm |
| **RAW (Refletância)** | Refletância em uint16, 32768 = ρ 1,0 |

O `RAW (Original)` está disponível desde o momento da importação — não necessita de qualquer processamento. Quando uma importação LATTICE não tem qualquer botão de base (o ficheiro de base de cada captura é o seu quadro bruto), a grelha desloca-se automaticamente para o primeiro botão de nível disponível, para que o destaque na barra de ferramentas corresponda ao que está a ver.

As exportações de dois níveis Chloros **não têm um botão de grelha próprio**:

* **Sem debayering** — a visualização do `RAW (Original)` já é apresentada sem debayering, pelo que um segundo botão numa imagem visualmente idêntica seria um elemento desnecessário. O produto `RAW (Debayered)` continua a ser gravado no disco e continua a poder ser selecionado a partir do menu suspenso da camada de ecrã inteiro.
* **Pré-visualização** — nas câmaras RGB, a pré-visualização é registada como a camada `White Balanced`, que possui um botão. Nas câmaras multiespectrais, é registada como `RAW (Preview)` e é acessível a partir do menu suspenso de camadas em ecrã inteiro.

{% hint style="info" %}
Estes botões de nível só são apresentados em projetos que contenham efetivamente fotogramas LATTICE. Os projetos Survey3 registam alguns dos mesmos nomes de camadas internas, e os botões são filtrados para esses projetos, pelo que uma grelha Survey3 mantém o seu conjunto familiar `JPG / Targets / Reflectance`.
{% endhint %}

Ao clicar numa miniatura da grelha, abre-se o [Visualizador de Imagens](opening-an-image-full-screen.md) em ecrã inteiro no **mesmo produto que a grelha está a mostrar** — se a grelha estiver definida para `Targets`, a miniatura abre a imagem de destino exportada.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Agrupamento de um projeto LATTICE: Por Gatilho vs. Por Câmara

As capturas em matriz produzem várias imagens do mesmo instante a partir de diferentes módulos de câmara. O agrupamento determina como a grelha as empilha. Ambos os modos apresentam barras de cabeçalho recolhíveis em largura total; **cada grupo começa expandido**, e o Chloros memoriza aqueles que fechar. O estado de recolhimento é monitorizado separadamente por modo, pelo que fechar um grupo no modo «Por Câmara» não fecha nada no modo «Por Disparador».

### Por Câmara (predefinição)

Um grupo por módulo de câmara. O cabeçalho mostra o modelo e o número de série da câmara (`LATT-M3M — <serial>`) e o número de fotografias. Os mosaicos dentro de um grupo são ordenados cronologicamente por evento de captura.

Neste modo, a barra de ferramentas também apresenta um **botão de filtro de câmara por câmara**, identificado como `MODEL (SERIAL)`. Todas as câmaras começam por estar selecionadas; clicar num botão desmarca essa câmara e remove o seu grupo da grelha. Esta é a forma mais rápida de rever uma faixa ao longo de todo um voo.

### Por Acionamento

Um grupo por evento de captura — o conjunto de fotogramas que todos os módulos captaram no mesmo acionamento. O cabeçalho mostra a hora da captura, o número de câmaras que contribuíram e um ícone por modelo de câmara no grupo. Os mosaicos dentro de um grupo são ordenados por número de série da câmara, pelo que a mesma banda fica na mesma coluna para cada disparo.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
As imagens não LATTICE num projeto misto não são agrupadas — são apresentadas como mosaicos simples após os grupos.

***

## As miniaturas da grelha seguem o tamanho do bloco GSD

Se tiver definido um tamanho de bloco **GSD (px)** na barra lateral do separador «Imagem», as miniaturas em grelha são apresentadas com essa mesma resolução no solo — e não apenas na visualização em ecrã inteiro. Um tamanho de bloco de 8 significa que cada pixel exibido é a média de um bloco de 8 × 8 pixels de origem, em qualquer parte da aplicação que mostre a imagem.

Como um mosaico tem, para começar, apenas algumas centenas de píxeis de largura, os tamanhos de bloco grossos deixam de fazer uma diferença visível na grelha muito antes de o fazerem na visualização em ecrã inteiro: um quadro de 4000 px desenhado num mosaico de 160 px já tem cerca de 25 píxeis de origem por pixel exibido. Consulte [Abrir uma imagem em ecrã inteiro](opening-an-image-full-screen.md#gsd-block-size) para obter informações sobre o próprio controlo.

***

## Páginas relacionadas

* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) — o visualizador em ecrã inteiro, valores do cursor e histograma
* [**Camadas de imagem**](image-layers.md) — o menu suspenso de camadas dentro do visualizador em ecrã inteiro
* [**Área de testes de índice/LUT**](index-lut-sandbox.md) — criação e exportação de visualizações de índice
* [**Definições do projeto**](../project-settings/project-settings.md) — os botões de exportação que determinam quais os produtos que existem
