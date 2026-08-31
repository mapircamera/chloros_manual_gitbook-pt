---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagem de saída

O Chloros exporta os produtos processados em quatro formatos de ficheiro. Selecione o formato nas Definições do Projeto (GUI), com o `--format` (CLI) ou com o `export_format` (SDK). O CLI e o SDK aceitam exatamente as cadeias de caracteres abaixo.

| String de formato | Extensão | Tipo de pixel | Intervalo de pixels | Notas |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(padrão)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametria / SIG. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % de refletância. Algumas aplicações não conseguem ler ficheiros TIFF com pontos flutuantes; os ficheiros são maiores. |
| `PNG (8-bit)` | `.png` | número digital uint8 | 0 – 255 | Compressão sem perdas, adequada para visualização na Web e em ecrãs. |
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

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para a Survey3. **Cada produto exportado mantém o nome do ficheiro de origem — é a pasta que identifica o produto, não um sufixo do nome do ficheiro.** Consulte [Onde são guardados os resultados](reference/cli-reference.md) na Referência do CLI para conhecer as regras completas.

## Produtos LATTICE (níveis de captura e exportação)

Um fotograma bruto do LATTICE é distribuído por todos os produtos solicitados numa única passagem. Cada tipo de produto tem o seu próprio botão de ativação/desativação (caixas de seleção da GUI ou CLI, `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos ativados por predefinição):

| Nível | Conteúdo | Tipo de dados |
| --- | --- | --- |
| `raw` | Dados Bayer diretamente do sensor (câmaras monocromáticas: a única banda). O processamento começa sempre a partir dos dados brutos. | Tal como captados |
| `debayered` | Desmosaico linear — 3 canais para M3C, escala de cinzentos de 1 canal para M3M. | DN linear |
| `radiance` | Radiação espectral absoluta proveniente da cadeia radiométrica completa, em **W/m²/sr/nm**. Sempre gravado como TIFF de 32 bits (`tiff32/Radiance_Images/`), independentemente do formato de exportação selecionado. | float32 |
| `reflectance` | Refletância ρ, em que **DN 32768 = ρ 1,0 (100 %)**, com margem até ρ 2,0. Compatível com o Pix4D. | uint16 |
| `preview` | Renderização pronta para visualização: RGB = equilíbrio de brancos + gama; multiespectral = alongamento de cores falsas. | visualização de 8 bits |

## Leitura dos valores de refletância dos píxeis

A refletância é armazenada como um número digital inteiro, e **o DN que corresponde a ρ = 1,0 (100 % de refletância) depende da câmara de origem**:

| Câmara de origem | ρ = 1,0 corresponde a DN | Como determinar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem até ρ 2,0) | A etiqueta XMP `Chloros:PixelScale=32768` está gravada no ficheiro. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há etiquetas XMP `Chloros:*` — essa ausência é o sinal. |

**Leia a etiqueta XMP `Chloros:PixelScale` e divida por ela**, em vez de assumir uma constante. A etiqueta está definida no domínio uint16, pelo que se mantém `32768` em todos os formatos de saída que redimensionam — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float32).

{% hint style="warning" %}
**Um caso não tem escala, por definição.** Quando uma captura de fonte de 8 bits (BayerRG8) é gravada como TIFF de 8 bits, o pipeline limita o valor a 0–255 em vez de reescalar, pelo que o ficheiro não possui escala — o Chloros omite deliberadamente o `Chloros:PixelScale` nesse caso. Se a etiqueta estiver ausente num ficheiro de refletância LATTICE, não presuma que existe uma escala; em vez disso, reexporte a 16 bits ou 32 bits.
{% endhint %}

Para consultar as regras completas (incluindo as etiquetas compatíveis com a MicaSense), consulte **«Leitura de píxeis de refletância»** na [Referência do CLI](reference/cli-reference.md).
