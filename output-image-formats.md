---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagem de saída O

Chloros exporta produtos processados em quatro formatos de ficheiro. Selecione o formato nas Definições do Projeto (GUI), com `--format` (CLI) ou com `export_format` (SDK). A CLI e o SDK aceitam as cadeias de caracteres exatas abaixo.

| Cadeia de caracteres do formato | Extensão | Tipo de pixel | Intervalo de pixels | Notas |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(padrão)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametria / SIG. |
| `TIFF (8-bit)` | `.tif` | número digital uint8 | 0 – 255 | Para todas as capturas de 8 bits, que oferecem apenas os formatos de 8 bits: o Chloros não aumenta a profundidade de bits. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% de refletância. Algumas aplicações não conseguem ler ficheiros TIFF de ponto flutuante; os ficheiros são maiores. |
| `PNG (8-bit)` | `.png` | número digital uint8 | 0 – 255 | Compressão sem perdas, adequada para visualização na Web e em ecrã. |
| `JPG (8-bit)` | `.jpg` | número digital uint8 | 0 – 255 | Compressão com perdas, ficheiros mais pequenos. |

## Onde são guardados os ficheiros de saída

Os ficheiros são guardados na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para a Survey3. **Cada produto exportado mantém o nome do ficheiro de origem — é a pasta que identifica o produto, e não um sufixo do nome do ficheiro.** Consulte [Onde são guardados os ficheiros de saída](reference/cli-reference.md) na Referência da CLI para conhecer todas as regras.

## Produtos LATTICE (níveis de captura e exportação)

Um fotograma bruto do LATTICE é dividido em todos os produtos solicitados numa única passagem. Cada tipo de produto tem a sua própria opção de ativação/desativação (caixas de seleção na GUI ou, na CLI, `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos ativados por predefinição):

| Nível | Conteúdo | Tipo de dados |
| --- | --- | --- |
| `raw` | Dados Bayer diretamente do sensor (câmaras monocromáticas: a banda única). O processamento começa sempre a partir dos dados brutos. | Tal como capturados |
| `debayered` | Desmosaico linear — 3 canais para M3C, 1 canal em escala de cinzentos para M3M: o desvio de escuro do pacote de calibração é removido, seguido de um desmosaico sensível a contornos, sem correção de linearidade, campo plano ou separação de canais. Uma exportação `process` grava o mesmo ficheiro que uma captura guarda neste nível, em todos os modos de processamento. | DN linear |
| `radiance` | Radiação espectral absoluta proveniente da cadeia radiométrica completa, em **W/m²/sr/nm**. Sempre gravado como TIFF de 32 bits (`tiff32/Radiance_Images/`), independentemente do formato de exportação selecionado. | float32 |
| `reflectance` | Refletância ρ, em que **DN 32768 = ρ 1,0 (100%)** com margem até ρ 2,0. Compatível com o Pix4D. | uint16 |
| `preview` | Renderização pronta para visualização: RGB = equilíbrio de brancos + gama; multiespectral = expansão de cores falsas. | visualização de 8 bits |

### Orientação

Os ficheiros `raw` e `debayered` mantêm-se sempre na orientação **nativa** do sensor; a configuração de inversão/rotação da câmara é aplicada apenas aos outros produtos (pré-visualização, radiação, refletância, índice). As capturas em matriz mantêm os ficheiros de cada elemento na orientação nativa, a menos que se solicite a alteração da orientação (`--orient` na CLI, `orient=True` no SDK). Cada ficheiro regista a orientação dos seus píxeis no XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — e o `ChlorosOrientationApplied` indica se os seus próprios píxeis contêm essa informação (`false` em ficheiros raw e debayered). O Visualizador de Imagens Chloros converte ficheiros «raw» e «debayered» de acordo com esse registo apenas para efeitos de visualização; os próprios ficheiros nunca são reescritos. Um ficheiro sem esse registo (criado antes da versão 1.3.0 ou por outra ferramenta) é apresentado tal como está armazenado.

## Máscaras de limitação

Os produtos de radiação e refletância de uma câmara multiespectral M3C realizam uma separação de bandas 3×3. Quando a separação produz um valor negativo numa banda, esse valor é limitado a zero (as visualizações em tempo real são limitadas exatamente da mesma forma). Como novidade na versão 1.3.0, um produto em que pelo menos um pixel tenha sido cortado recebe uma **máscara de limitação** associada que indica exatamente quais os píxeis e as bandas afetados:

* **Onde:**numa pasta `clamp_masks/`**ao lado** das pastas dos produtos, nunca dentro de uma delas: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, e `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **O quê:** um ficheiro TIFF de banda única `uint8` na própria grelha de píxeis do produto (incluindo orientação, ROI, binning e alinhamento da matriz). O bit 0 (`mask & 1`) é definido onde a banda no canal R do produto foi cortada, o bit 1 (`& 2`) no canal G, o bit 2 (`& 4`) no canal B. A percentagem de píxeis cortados por banda também é indicada no produto como `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` e `Chloros:BandBUnmixClampFraction`.
* **Como o produto indica isso:** a etiqueta XMP `Chloros:UnmixClampMask` contém o caminho da máscara em relação à pasta do produto (`../clamp_masks/<product folder>/<name>_clampmask.tif`, sempre separado por `/`), `none` quando nenhum pixel foi recortado (não é gravada nenhuma máscara) ou `unavailable` quando não foi possível gravar uma máscara. Um produto sem etiqueta nunca executou a separação de cores (câmaras RGB e monocromáticas, níveis raw e pós-processamento de bayes).

Como nenhuma pasta de produto contém uma máscara, pode entregar uma pasta de produto como `Reflectance_Calibrated_Images/` ao Pix4D, Metashape ou outro software de fotogrametria tal como está, incluindo as subpastas — entregue a pasta de produto, não a pasta de formato acima dela. A própria importação do Chloros ignora as máscaras de limitação. A CLI contabiliza-as separadamente: `Image products written: N (plus M clamp masks)`.

## Produtos de índice

Cada índice é colocado na sua própria pasta `<INDEX>_Index_Images/` (por exemplo, `NDVI_Index_Images/`).

* **Formatos inteiros**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) armazenam um**estiramento linear por imagem**, e não um mapeamento −1…1: DN 0 é o valor de índice mais baixo desse fotograma e a escala total é o mais alto. O ficheiro contém `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` e `Chloros:IndexMax`, pelo que também contém `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Como cada fotograma tem os seus próprios pontos extremos, compare os fotogramas como valores, e não como DN.
* **`TIFF (32-bit, Percent)`** armazena os próprios valores de índice como float32, sem reescalonamento — a melhor escolha para trabalhos quantitativos ao longo de um voo.

Novidade na versão 1.3.0: um produto de índice descreve **a si próprio**, em vez de repetir as etiquetas do ficheiro bruto de origem. As suas etiquetas de GPS e de câmara são copiadas do produto a partir do qual foi calculado (o produto de refletância, quando existe) e, em seguida:

| Etiqueta | Valor |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (pelo que o ficheiro nunca é reimportado como uma captura) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / a fórmula utilizada |
| `Chloros:IndexLevel` | `reflectance` ou `radiance` — a partir de que o índice foi calculado (ausente quando nenhum dos dois) |
| `Chloros:IlluminationInvariant` | `True` para um índice calculado a partir da refletância, `False` a partir da radianância ou dos valores do sensor |
| `Chloros:ChlorosPipelineVersion` | a versão do Chloros que criou o índice |

`Chloros:PixelScale` foi removido dos produtos de índice — utilize `IndexMin` / `IndexMax` acima. A receita completa de leitura está em «Um raster de índice inteiro é um alongamento de contraste por imagem» na [Referência CLI](reference/cli-reference.md).

## Leitura dos valores de refletância dos píxeis

A refletância é armazenada como um número digital inteiro, e **o DN que corresponde a ρ = 1,0 (100% de refletância) depende da câmara de origem**:

| Câmara de origem | ρ = 1,0 corresponde a DN | Como determinar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem de segurança até ρ 2,0) | A etiqueta XMP `Chloros:PixelScale=32768` está impressa no ficheiro. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não existe a etiqueta `Chloros:PixelScale` — essa ausência é o indicador. (Os produtos Survey3 incluem outras etiquetas de processamento `Chloros:*`, tais como `Chloros:CalibrationApplied`, em todos os modos de processamento a partir da versão 1.3.0.) |

**Leia a etiqueta XMP `Chloros:PixelScale` e divida por ela**, em vez de assumir um valor constante. A etiqueta está definida no domínio uint16, pelo que se mantém como `32768` em todos os formatos de saída que redimensionam — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float32).

{% hint style="warning" %}
**Capturas de 8 bits.** As exportações de uma captura de 8 bits são de 8 bits. A sua refletância é redimensionada e marcada como `Chloros:PixelScale=32768`, pelo que a escala se mantém; Os produtos DN de 8 bits e os ficheiros de refletância criados antes da versão 1.3.0 são limitados ao intervalo 0–255 e não incluem qualquer etiqueta de escala. Se a etiqueta estiver ausente num ficheiro de refletância LATTICE, não presuma a existência de uma escala; em vez disso, reexporte em 16 bits ou 32 bits.
{% endhint %}

Para consultar as regras completas (incluindo as etiquetas compatíveis com a MicaSense), consulte **«Leitura de píxeis de refletância»** na [Referência da CLI](reference/cli-reference.md).
