# Camadas de imagem

O **menu suspenso de camadas** no canto superior direito do Visualizador de Imagens permite alternar entre todas as versões da imagem que está a visualizar — desde a captura original, passando por cada produto processado, até às imagens de índice calculadas — sem sair do visualizador.

## O que são camadas de imagem?

Uma «camada» no Chloros é um **ficheiro de produto**registado em relação a uma imagem de origem. A importação fornece-lhe os ficheiros de origem; o processamento adiciona uma camada para cada produto gerado pela execução. Os ficheiros exportados mantêm o nome do ficheiro de origem — é a**pasta** que identifica o produto, e o nome da camada é a etiqueta atribuída pelo Chloros a essa pasta.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## A lista de camadas

### Sempre presente

| Camada | O que é |
| --- | --- |
| **JPG**(ou**PNG**/**TIFF**) | O ficheiro base que veio com a captura. O Survey3 importa um `.JPG` ao lado de cada `.RAW`; As capturas LATTICE incluem uma pré-visualização de ecrã PNG ou TIFF. Identificada de acordo com o que foi efetivamente importado |
| **RAW (Original)** | O fotograma RAW de origem, submetido a processamento de remoção de bayering para visualização, sem correções aplicadas. Disponível desde o momento da importação — não necessita de processamento |

Uma captura do LATTICE cujo ficheiro base **é** o seu fotograma em bruto não tem uma entrada de base separada: o `RAW (Original)` já a abrange.

### Produtos de processamento Survey3

| Camada | Gravada em | Existe quando |
| --- | --- | --- |
| **RAW (Alvo)** | — | O fotograma foi identificado como contendo um alvo de calibração |
| **RAW (Refletância)** | `Reflectance_Calibrated_Images/` | A calibração de refletância foi executada com sucesso neste quadro |
| **Correção de vinheta**| `Vignette_Corrected_Images/` | Não foi possível efetuar a calibração de refletância do quadro**e** a *correção de vinheta* estava ativada |
| **Resposta do sensor**| `Sensor_Response_Images/` | Não foi possível calibrar a refletância do fotograma**e** a *correção de vinheta* estava desativada |
| **Equilíbrio de brancos** | `White_Balanced_Images/` | Foi gerado um produto com equilíbrio de brancos |

{% hint style="info" %}
**A correção de vinheta e a resposta do sensor são alternativas, nunca ambas.** Existe exatamente um produto de recurso não calibrado por execução, para cada modelo de câmara, e a opção *correção de vinheta* seleciona qual deles. Consulte [Definições do projeto](../project-settings/project-settings.md).
{% endhint %}

### Níveis LATTICE

O LATTICE captura o fan-out nestes níveis numa única passagem de processamento. Quais existem depende das opções de exportação por produto nas Configurações do Projeto e do que se aplica à câmara.

| Camada | Gravada em | Aplica-se a |
| --- | --- | --- |
| **RAW (sem bayering)** | `Debayered_Images/` | RGB e multiespectral |
| **RAW (Pré-visualização)** | `Preview_Images/` | Multiespectral (estiramento de cores falsas) |
| **Equilíbrio de brancos** | `Preview_Images/` | Câmaras principais RGB — a pré-visualização RGB está registada com este nome para que coincida com a camada Survey3 com o mesmo nome |
| **RAW (Radiação)** | `Radiance_Images/` | Apenas multiespectral |
| **RAW (Radiação)** | `Reflectance_Calibrated_Images/` | Apenas multiespectral, e apenas quando um registo descendente `.daq` correspondente ou um alvo no quadro aprovado no controlo de qualidade (QA) cobrir o quadro |

As câmaras principais RGB não têm radiometria por banda, pelo que a radiação e a refletância são ignoradas para estas como **não aplicáveis** — o registo indica isso em vez de falhar silenciosamente.

### Camadas de índice, LUT e sandbox

| Padrão da camada | Exemplo | De onde provém |
| --- | --- | --- |
| **RAW (`<INDEX>` Índice)** | `RAW (NDVI Index)` | Um por índice configurado nas Definições do Projeto, calculado durante o processamento |
| **`<INDEX>` LUT** | `NDVI LUT` | A versão com mapeamento de cor de um índice |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Um por execução de exportação [Índice/LUT Sandbox](index-lut-sandbox.md) |

Se o mesmo nome de índice for configurado mais do que uma vez com definições diferentes, o segundo e os seguintes recebem um número no nome (`RAW (NDVI2 Index)`), para que as camadas permaneçam distinguíveis.

***

## Utilizar o seletor de camadas

1. Abra uma imagem em ecrã inteiro clicando numa miniatura na grelha
2. Clique no **menu suspenso das camadas** no canto superior direito do visualizador
3. Escolha uma camada — a imagem atualiza-se imediatamente

O menu suspenso apresenta **JPG, RAW (Original), RAW (Destino), RAW (Refletância)** em primeiro lugar, por essa ordem, e lista tudo o resto a seguir, na ordem em que os produtos foram registados.

### Preferência de camada ao navegar

Premir **←**/**→** avança para a imagem seguinte e tenta manter-te na mesma camada:

1. **Correspondência exata primeiro** — se a imagem seguinte tiver uma camada com o mesmo nome, é essa que aparece. É isto que o mantém na camada `RAW (NDVI Index)` enquanto percorre um conjunto completo
2. **Em seguida, uma correspondência por tipo** — uma camada de índice procura qualquer camada de índice, uma LUT procura qualquer LUT, «refletância» procura «refletância», «alvo» procura «alvo», «original» procura «original» e «base» procura «base»
3. **Em seguida, apenas para camadas de exportação** — o nome é mantido mesmo que a lista de camadas ainda não tenha sido atualizada, porque o ficheiro já existe no disco. É isto que permite rever os produtos enquanto uma execução ainda os está a gravar
4. **Caso contrário** — a primeira camada disponível, que normalmente é a imagem base

Os ficheiros sidecar `.daq` e `.csv` no projeto são ignorados pela navegação com as setas, pelo que percorrer as imagens nunca leva a um registo do sensor de luz.

O zoom e a panorâmica também se mantêm ao longo das imagens, o que facilita a comparação «antes e depois» da mesma posição no campo.

***

## Compreender os valores dos píxeis por camada

O [painel «Valores do cursor»](opening-an-image-full-screen.md#cursor-values) apresenta o valor real por canal sob o cursor, na unidade em que a camada está armazenada. As suas colunas variam consoante a camada:

| Camada | Unidade apresentada | Notas |
| --- | --- | --- |
| Base (JPG / PNG / pré-visualização TIFF) | DN, 0–255 | Valores de visualização, com correção de gama no RGB. Apenas para inspeção visual |
| RAW (Original) | DN | Valores digitais brutos do sensor. O eixo do histograma indica a profundidade: 255 (8 bits), 4095 (12 bits) ou 65535 (16 bits) |
| RAW (sem debayering) | DN | Linear, sem alongamento da imagem |
| RAW (Pré-visualização) / Equilíbrio de brancos | DN | Produto de visualização — esticado ou com correção de gama. Não se destina a medição |
| RAW (Radiação) | **W/m²/sr/nm** | Radiação física em Float32. Sem coluna DN |
| RAW (Refletância) | DN **e %** | Percentagem calculada com a escala própria desse ficheiro — ver abaixo |
| Exportações de índice / LUT / sandbox | Valor de índice, ou componentes RGB | Um ficheiro de índice de canal único apresenta o valor de índice; um ficheiro LUT com mapeamento de cores apresenta os componentes Red/Green/Blue |

### Refletância: a escala é por ficheiro

{% hint style="warning" %}
**«Dividir por 65 535» só está correto para Survey3.** A refletância LATTICE é armazenada numa escala diferente, e misturar os dois divisores é a forma mais comum de obter valores de refletância que são exatamente metade do que deveriam ser.
{% endhint %}

| Fonte | DN que equivale a uma refletância de 1,0 | Identificado por |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | A etiqueta XMP `Chloros:PixelScale=32768` incluída em todas as exportações de refletância LATTICE. A margem de 2× significa que valores de ρ superiores a 1,0 são representáveis, em vez de serem cortados |
| **Survey3**|**65535** | Sem etiqueta de escala XMP Chloros — a calibração Survey3 grava ρ × dtype-max e corta no valor de 1,0 |

Para SIG e scripts: leia o valor `Chloros:PixelScale` do ficheiro e divida por ele. Se a etiqueta estiver ausente, o ficheiro está na escala Survey3 (65535). O visualizador, a área de teste do índice/LUT e a exportação do índice calculam todos a escala desta mesma forma, pelo que o número que se vê no cursor é o número utilizado nos cálculos do índice.

Armazenamento específico do formato com base nessa escala:

* **TIFF (32 bits, por cento)** armazena DN / 65535 como um número de ponto flutuante
* **PNG (8 bits)**e**JPG (8 bits)** armazenam DN × 255 / 65535
* Uma **exportação de 8 bits TIFF de uma captura com fonte de 8 bits** é limitada ao intervalo 0–255 em vez de ser redimensionada e, deliberadamente, não inclui qualquer etiqueta de escala. O painel apresenta apenas o DN para esses ficheiros, sem coluna de percentagem

### Intervalos de valores de índice

| Família de índices | Intervalo típico | Leitura |
| --- | --- | --- |
| Diferença normalizada (NDVI, GNDVI, NDRE, ENDVI…) | −1 a +1 | Vegetação saudável geralmente entre 0,4 e 0,9; solo nu próximo de 0; água com valores negativos |
| Ajustada ao solo (SAVI, OSAVI, MSAVI2…) | aproximadamente de −1 a +1,5 | Valor semelhante ao de NDVI com o fundo do solo suprimido |
| Rácio (GRVI, GCI, MSR, CIRE…) | sem limite superior | Os rácios aumentam sem limite à medida que a faixa do denominador tende para zero |
| EVI / LAI | 0 a ~1, 0 a ~3,5 | As nuvens e outros píxeis saturados fazem com que ambos saiam do intervalo — máscara-os primeiro |

Consulte [Fórmulas de índices multiespectrais](../project-settings/multispectral-index-formulas.md) para conhecer a fórmula exata por trás de cada predefinição.

***

## Fluxos de trabalho comuns

### Comparação antes/depois

1. Selecione **RAW (Original)** e observe a vinheta e os valores não calibrados
2. Mude para **RAW (Refletância)**

3. Compare — vinheta removida, valores calibrados. O zoom e a panorâmica mantêm-se, para que esteja a observar a mesma área

### Analise um índice ao longo de um conjunto completo

1. Abra a primeira imagem processada e selecione a camada de índice
2. Prima **→** repetidamente — a camada de índice acompanha-o de imagem em imagem
3. Observe o histograma na barra lateral à medida que avança: um fotograma cuja distribuição apresente picos merece uma análise mais detalhada

### Verifique os alvos de calibração

1. Selecione **RAW (Alvo)** num fotograma-alvo
2. Confirme se o alvo está claramente visível e foi detetado
3. Passe para o próximo fotograma-alvo — a camada de alvos acompanha-o

### Verifique a precisão dos valores de refletância

1. Selecione **RAW (Refletância)**

2. Leia a coluna**%** no painel «Valores do cursor» — já está corretamente dimensionada para esse ficheiro
3. Verifique a coerência em relação a materiais conhecidos no fotograma: a vegetação saudável apresenta valores elevados em NIR e baixos no vermelho; um alvo de calibração deve apresentar valores próximos da sua refletância publicada

***

## Resolução de problemas

### Uma camada que eu esperava não consta no menu suspenso

**Possíveis causas**

* A imagem nunca foi processada — apenas a camada base e a `RAW (Original)` existem
* A opção de exportação do produto está desmarcada nas «Definições do projeto»
* O produto não se aplica a essa câmara (radianância e refletância num mestre RGB; qualquer índice numa câmara M3M monocromática de banda única)
* A calibração de refletância não tinha dados com que trabalhar — sem cobertura descendente do `.daq` e sem alvo no quadro aprovado no controlo de qualidade — pelo que o quadro recorreu à opção «Vignette Corrected» (Correção de vinheta) ou «Sensor Response» (Resposta do sensor)

**O que fazer**

1. Verifique o registo da execução: o Chloros indica quando um produto de exportação solicitado foi impossível de gerar e porquê
2. Verifique as opções de exportação por produto em [Definições do Projeto](../project-settings/project-settings.md)
3. Confirme se a pasta do produto existe na árvore de saída do projeto
4. Reprocesse com o produto ativado

### A lista de camadas parece desatualizada

O Chloros volta a analisar as pastas de produtos do projeto enquanto uma execução está em curso e corrige registos de camadas em falta com base no que está efetivamente no disco; assim, uma camada cuja exportação tenha terminado normalmente aparece por si só numa sondagem. Sair da imagem e voltar a ela força uma nova resolução.

### Os valores de refletância parecem ser metade do que deveriam ser

É quase certo que está a dividir um ficheiro LATTICE por 65535. Utilize o `Chloros:PixelScale` (32768) ou consulte a coluna **%**, que já aplicou esse valor.

### A camada de índice existe, mas a imagem está em branco

O índice necessita de bandas que a sua camada não possui — por exemplo, um índice que lê um terceiro canal aplicado a um ficheiro de um ou dois canais. Mude para uma camada multibanda (refletância ou sem Bayer), ou escolha um índice que se adapte ao filtro da câmara.

***

## Próximos passos

* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) — leitura do cursor, histograma e controlo do GSD
* [**Área de teste de índices/LUT**](index-lut-sandbox.md) — visualização interativa de índices e exportação
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — a referência do índice
* [**Conclusão do processamento**](../processing-images-gui/finishing-the-processing.md) — a árvore de pastas de saída para a qual estas camadas apontam
