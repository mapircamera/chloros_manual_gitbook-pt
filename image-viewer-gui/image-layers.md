# Camadas de imagem

O **menu suspenso de camadas** no canto superior direito do Visualizador de Imagens permite alternar entre todas as versões da imagem que está a visualizar — desde a captura original, passando por cada produto processado, até às imagens de índice calculadas — sem sair do visualizador.

## O que são camadas de imagem?

Uma «camada» no Chloros é um **ficheiro de produto**registado em relação a uma imagem de origem. A importação fornece-lhe os ficheiros de origem; o processamento adiciona uma camada para cada produto gerado pela execução. Os ficheiros exportados mantêm o nome do ficheiro de origem — é a**pasta** que identifica o produto, e o nome da camada é a etiqueta que o Chloros atribui a essa pasta.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## A lista de camadas

### Sempre presentes

| Camada | O que é |
| --- | --- |
| **JPG**(ou**PNG**/**TIFF**) | O ficheiro base que veio com a captura. O Survey3 importa um `.JPG` ao lado de cada `.RAW`; as capturas do LATTICE trazem uma pré-visualização de exibição em PNG ou TIFF. Rotulado de acordo com o que foi efetivamente importado |
| **RAW (Original)** | O fotograma bruto de origem, submetido a débayering para visualização, sem correções aplicadas. Disponível desde o momento da importação — não necessita de processamento |

Uma captura do LATTICE cujo ficheiro base **é** o seu quadro bruto não tem uma entrada de base separada: o `RAW (Original)` já o abrange.

### Produtos de processamento do Survey3



| Camada | Gravada em | Existe quando |
| --- | --- | --- |
| **RAW (Alvo)** | — | O quadro foi identificado como contendo um alvo de calibração |
| **RAW (Refletância)** | `Reflectance_Calibrated_Images/` | A calibração de refletância foi executada com sucesso neste quadro |
| **Corrigido de vinheta**| `Vignette_Corrected_Images/` | Não foi possível efetuar a calibração de refletância do quadro**e** a *correção de vinheta* estava ativada |
| **Resposta do sensor**| `Sensor_Response_Images/` | Não foi possível calibrar o quadro em termos de refletância**e** a *correção de vinheta* estava desativada |
| **Equilíbrio de brancos** | `White_Balanced_Images/` | Foi gravado um produto com equilíbrio de brancos |

{% hint style="info" %}
**A correção de vinheta e a resposta do sensor são alternativas, nunca ambas.** Existe exatamente um produto de recurso não calibrado por execução, para cada modelo de câmara, e o seletor *Correção de vinheta* escolhe qual. Consulte [Definições do projeto](../project-settings/project-settings.md).
{% endhint %}

### Níveis LATTICE

O LATTICE captura o fan-out nestes níveis numa única passagem de processamento. Quais existem depende das opções de exportação por produto nas *Definições do Projeto* e do que se aplica à câmara.

| Camada | Gravada em | Aplica-se a |
| --- | --- | --- |
| **RAW (sem debayering)** | `Debayered_Images/` | RGB e multiespectral |
| **RAW (Pré-visualização)** | `Preview_Images/` | Multiespectral (estiramento de cores falsas) |
| **Equilíbrio de brancos** | `Preview_Images/` | Câmaras principais RGB — a pré-visualização RGB é registada com este nome para que corresponda à camada do Survey3 com o mesmo nome |
| **RAW (Radiação)** | `Radiance_Images/` | Apenas multiespectral |
| **RAW (Refletância)** | `Reflectance_Calibrated_Images/` | Apenas multiespectral, e apenas quando um registo descendente `.daq` correspondente ou um alvo no quadro aprovado no controlo de qualidade (QA) cobrir o quadro |

As câmaras principais RGB não têm radiometria por banda, pelo que a radiação e a refletância são ignoradas para elas como **não aplicáveis** — o registo indica isso em vez de falhar silenciosamente.

{% hint style="info" %}
**As camadas brutas e sem debayering alinham-se com os produtos (novidade na versão 1.3.0).**Os ficheiros brutos e sem debayering do LATTICE são guardados na orientação nativa do sensor e registam no XMP a orientação que lhes pertence; os outros produtos são guardados com essa orientação já aplicada. O visualizador e a grelha de imagens alternam entre**RAW (Original)**e**RAW (Sem Bayes)** de acordo com esse registo para efeitos de visualização, pelo que o fotograma já não se inverte quando se alterna entre uma camada de produto e uma camada bruta. Trata-se apenas de uma alteração de visualização — os ficheiros nunca são reescritos. Os ficheiros sem um registo de orientação (capturados antes da versão 1.3.0 ou criados por outro software) são apresentados tal como estão armazenados. Consulte [Formatos de imagem de saída](../output-image-formats.md#orientation).
{% endhint %}

As máscaras de limitação (`*_clampmask.tif` numa pasta `clamp_masks/`) não são camadas: o Chloros ignora-as durante a importação. Consulte [Formatos de imagem de saída](../output-image-formats.md#clamp-masks).

### Camadas de índice, LUT e sandbox

| Padrão de camada | Exemplo | De onde provém |
| --- | --- | --- |
| **RAW (Índice `<INDEX>`)** | `RAW (NDVI Index)` | Uma por índice configurado nas Definições do Projeto, calculada durante o processamento |
| **`<INDEX>` LUT** | `NDVI LUT` | A versão com mapeamento de cores de um índice |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Um por execução de exportação [Índice/LUT Sandbox](index-lut-sandbox.md) execução de exportação |

Se o mesmo nome de índice for configurado mais do que uma vez com definições diferentes, o segundo e os seguintes recebem um número no nome (`RAW (NDVI2 Index)`), para que as camadas permaneçam distinguíveis.

***

## Utilizar o seletor de camadas

1. Abra uma imagem em ecrã inteiro clicando numa miniatura na grelha
2. Clique no **menu suspenso de camadas** no canto superior direito do visualizador
3. Escolha uma camada — a imagem atualiza-se imediatamente

O menu suspenso apresenta **JPG, RAW (Original), RAW (Alvo), RAW (Refletância)** em primeiro lugar, por essa ordem, e lista tudo o resto a seguir, na ordem em que os produtos foram registados.

### Preferência de camada durante a navegação

Premir **←**/**→** avança para a imagem seguinte e tenta manter-te na mesma camada:

1. **Correspondência exata primeiro** — se a imagem seguinte tiver uma camada com o mesmo nome, é essa que aparece. É isto que o mantém na camada `RAW (NDVI Index)` enquanto percorre um conjunto completo
2. **Em seguida, uma correspondência por tipo** — uma camada de índice procura qualquer camada de índice, uma LUT procura qualquer LUT, «refletância» procura «refletância», «alvo» procura «alvo», «original» procura «original» e «base» procura «base»
3. **Em seguida, apenas para camadas de exportação** — o nome é mantido mesmo que a lista de camadas ainda não esteja atualizada, porque o ficheiro já existe no disco. É isto que permite rever os produtos enquanto uma execução ainda os está a gravar
4. **Caso contrário** — a primeira camada disponível, que normalmente é a imagem base

Os ficheiros sidecar `.daq` e `.csv` no projeto são ignorados pela navegação com as setas, pelo que a navegação pelas imagens nunca atinge um registo do sensor de luz.

O zoom e a panorâmica também se mantêm ao longo das imagens, o que facilita a comparação «antes e depois» da mesma posição no campo.

***

## Compreender os valores dos píxeis por camada

O [painel «Valores do cursor»](opening-an-image-full-screen.md#cursor-values) apresenta o valor real por canal sob o cursor, na unidade em que a camada está armazenada. As suas colunas variam consoante a camada:

| Camada | Unidade apresentada | Notas |
| --- | --- | --- |
| Base (pré-visualização JPG / PNG / TIFF) | DN, 0–255 | Valores de visualização, com correção gama em RGB. Apenas para inspeção visual |
| RAW (Original) | DN | Valores digitais brutos do sensor. O eixo do histograma indica a profundidade: 255 (8 bits), 4095 (12 bits) ou 65535 (16 bits) |
| RAW (sem debayering) | DN | Linear, sem ampliação na exibição |
| RAW (Pré-visualização) / Equilíbrio de brancos | DN | Produto de exibição — ampliado ou com correção de gama. Não se destina a medição |
| RAW (Radiação) | **W/m²/sr/nm** | Radiação física em Float32. Sem coluna DN |
| RAW (Refletância) | DN **e %** | Percentagem calculada com a escala própria desse ficheiro — ver abaixo |
| Exportações de índice / LUT / sandbox | Valor de índice ou componentes RGB | Um ficheiro de índice de canal único apresenta o valor de índice; um ficheiro LUT com mapeamento de cores apresenta os componentes Red /Green /Blue |

### Refletância: a escala é específica de cada ficheiro

{% hint style="warning" %}
**«Dividir por 65 535» só está correto para o Survey3.** A refletância do LATTICE é armazenada numa escala diferente, e misturar os dois divisores é a forma mais comum de obter valores de refletância que são exatamente metade do que deveriam ser.
{% endhint %}

| Fonte | DN que equivale à refletância 1,0 | Identificado por |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | A etiqueta XMP `Chloros:PixelScale=32768` incluída em todas as exportações de refletância do LATTICE. A margem de 2× significa que valores de ρ superiores a 1,0 são representáveis, em vez de serem cortados |
| **Survey3**| **65535** | Sem etiqueta de escala XMP do Chloros — a calibração do Survey3 grava ρ × dtype-max e corta no valor 1,0 |

Para SIG e scripts: leia `Chloros:PixelScale` do ficheiro e divida por esse valor. Se a etiqueta estiver ausente, o ficheiro está na escala dSurvey3

(65535). O visualizador, a área de teste do índice/LUT e a exportação do índice determinam a escala da mesma forma; por isso, o número que se vê no cursor é o número utilizado nos cálculos do índice.

Armazenamento específico do formato, além dessa escala:

* **TIFF (32 bits, Porcentagem)** armazena DN / 65535 como um número flutuante
* **PNG (8 bits)**e**JPG (8 bits)** armazenam DN × 255 / 65535
* Um **produto DN de 8 bits de uma captura de 8 bits**, ou um ficheiro de refletância gravado antes da versão 1.3.0, é limitado ao intervalo 0–255 e não contém qualquer etiqueta de escala. O painel apresenta apenas o DN para esses ficheiros, sem coluna de percentagem

### Intervalos de valores do índice

| Família de índices | Intervalo típico | Leitura |
| --- | --- | --- |
| Diferença normalizada (NDVI, GNDVI, NDRE, ENDVI…) | −1 a +1 | Vegetação saudável normalmente entre 0,4 e 0,9; solo nu próximo de 0; água com valores negativos |
| Ajustado ao solo (SAVI, OSAVI, MSAVI2…) | aproximadamente de −1 a +1,5 | Leitura semelhante à do NDVI, com o fundo do solo suprimido |
| Rácio (GRVI, GCI, MSR, CIRE…) | ilimitado para valores superiores | Os rácios aumentam sem limite à medida que a banda do denominador se aproxima de zero |
| EVI / LAI | 0 a ~1, 0 a ~3,5 | As nuvens e outros píxeis saturados fazem com que ambos fiquem fora do intervalo — é necessário máscar esses píxeis primeiro |

Consulte [Fórmulas dos Índices Multiespectrais](../project-settings/multispectral-index-formulas.md) para conhecer a fórmula exata por trás de cada predefinição.

***

## Fluxos de trabalho comuns

### Comparação antes/depois

1. Selecione **RAW (Original)** e observe a vinheta e os valores não calibrados
2. Mude para **RAW (Refletância)**

3. Compare — vinheta removida, valores calibrados. O zoom e a panorâmica mantêm-se, para que esteja a observar a mesma área

### Analise um índice em todo o conjunto

1. Abra a primeira imagem processada e selecione a camada de índice
2. Prima **→** repetidamente — a camada de índice acompanha-o de imagem em imagem
3. Observe o histograma na barra lateral à medida que avança: um fotograma cuja distribuição apresente picos merece uma análise mais detalhada

### Verificar alvos de calibração

1. Selecione **RAW (Alvo)** num fotograma-alvo
2. Confirme se o alvo está claramente visível e foi detetado
3. Passe para o próximo fotograma-alvo — a camada de alvos acompanha-o

### Verifique a precisão dos valores de refletância

1. Selecione **RAW (Refletância)**

2. Leia a coluna**%** no painel «Valores do cursor» — já está devidamente escalada para esse ficheiro
3. Faça uma verificação de coerência em relação a materiais conhecidos no fotograma: a vegetação saudável apresenta valores elevados no NIR e baixos no vermelho; um alvo de calibração deve apresentar valores próximos da sua refletância publicada

***

## Resolução de problemas

### Uma camada que eu esperava não consta no menu suspenso

**Possíveis causas**

* A imagem nunca foi processada — existem apenas a base e a `RAW (Original)`
* A opção de exportação do produto está desmarcada nas Definições do Projeto
* O produto não se aplica a essa câmara (radianância e refletância num master RGB; qualquer índice numa câmara monocromática M3M de banda única)
* A calibração de refletância não teve dados com que trabalhar — sem cobertura descendente do `.daq` e sem alvo no quadro aprovado no controlo de qualidade — pelo que o quadro recorreu à correção de vinheta ou à resposta do sensor

**O que fazer**

1. Verifique o registo da execução: o Chloros indica quando um produto de exportação solicitado foi impossível de gerar e porquê
2. Verifique as opções de exportação por produto em [Definições do projeto](../project-settings/project-settings.md)
3. Confirme se a pasta do produto existe na árvore de saída do projeto
4. Reprocesse com o produto ativado

### A lista de camadas parece desatualizada

O Chloros volta a analisar as pastas de produtos do projeto enquanto uma execução está em curso e corrige registos de camadas em falta com base no que está efetivamente no disco; por isso, uma camada cuja exportação tenha terminado normalmente aparece por si só numa sondagem. Sair da imagem e voltar a ela força uma nova resolução.

### Os valores de refletância parecem ser metade do que deveriam ser

É quase certo que está a dividir um ficheiro LATTICE por 65535. Utilize o `Chloros:PixelScale` (32768) ou consulte a coluna **%**, que já aplicou esse valor.

### A camada de índice existe, mas a imagem está em branco

O índice necessita de bandas que a sua camada não possui — por exemplo, um índice que lê um terceiro canal aplicado a um ficheiro de um ou dois canais. Mude para uma camada multibanda (refletância ou sem bayering) ou escolha um índice que se adapte ao filtro da câmara.

***

## Próximos passos

* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) — leitura do cursor, histograma e controlo do GSD
* [**Área de testes de índices/LUT**](index-lut-sandbox.md) — visualização interativa de índices e exportação
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — a referência de índices
* [**Concluir o processamento**](../processing-images-gui/finishing-the-processing.md) — a árvore de pastas de saída para a qual estas camadas apontam
