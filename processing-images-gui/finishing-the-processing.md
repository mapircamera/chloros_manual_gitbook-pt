# Conclusão do processamento

Assim que o Chloros concluir o processamento, é altura de analisar os resultados, verificar a qualidade da saída e preparar as imagens processadas para utilização no seu fluxo de trabalho. Esta página orienta-o através dos passos finais e das próximas ações.

## Indicação de conclusão do processamento

Quando o processamento terminar com sucesso, verá vários indicadores:

* ✅ **Barra de progresso**: Atinge 100% de conclusão
* ✅ **Registo de depuração**: Mostra as últimas linhas `[RUN-SUMMARY]` com as contagens (imagens, grupos de câmaras, alvos, imagens calibradas, ficheiros gravados)
* ✅ **Botão Iniciar**: Volta a ficar ativado (pronto para a próxima execução do processamento)
* ✅ **Ficheiros de saída**: Todas as imagens processadas são guardadas na árvore de saída do projeto (abaixo)

{% hint style="warning" %}
**Uma execução que não grava imagens é considerada uma falha.** Se solicitou produtos de imagem e a execução não gravou nenhum, o Chloros reporta-a como uma falha — o `[RUN-SUMMARY]` no nome do registo sugere a causa provável (nada importado, nenhum alvo detetado ou todos os produtos solicitados ignorados por serem inaplicáveis). O equivalente na CLI sai com um valor diferente de zero. Uma execução deliberada apenas com metadados (todos os produtos de exportação desativados, sem índices) continua a ser considerada bem-sucedida. Consulte [a Referência da CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localizar as suas imagens processadas

### Abrir a pasta de saída

1. Clique no ícone do **Menu Principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(canto superior esquerdo)
2. Selecione **«Abrir pasta do projeto»**

3. O seu explorador de ficheiros abre-se no diretório do projeto
4. Localize o seu projeto pelo nome

### A árvore de saída

Os produtos são guardados **na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Pasta da câmara**: `LATT-<sensor>-<lens>-F<filter>` para a LATTICE (correspondendo ao EXIF da captura `Model`), `<model>_<filter>` para o Survey3 (por exemplo, `Survey3N_RGN`). Duas câmaras que partilham um sensor e um filtro, mas que têm lentes diferentes, mantêm árvores separadas — a vinheta, o campo de visão e a distorção diferem.
* **Pasta de formato**: segue a sua configuração de formato de exportação — `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` para TIFF (32 bits, Porcentagem). A radianância é sempre float32 e é sempre guardada em `tiff32`.
* **Pastas de produtos**:
  * `Reflectance_Calibrated_Images/` — refletância calibrada
  * `Debayered_Images/` — sem debayering linear (LATTICE): desvio de escuridão removido e sem demosaico, sem campo plano nem desmistura, na orientação nativa do sensor
  * `Preview_Images/` — pré-visualização no ecrã (LATTICE)
  * `Radiance_Images/` — radiação espectral em float32, W/m²/sr/nm (multiespectral LATTICE)
  * `Vignette_Corrected_Images/` **ou** `Sensor_Response_Images/` — o valor de recurso não calibrado para fotogramas sem referência de refletância; existe exatamente um dos dois por execução, escolhido pela definição de correção de vinheta
  * `<INDEX>_Index_Images/` — uma pasta por índice selecionado (por exemplo, `NDVI_Index_Images`)
* **`clamp_masks/`** (novidade na versão 1.3.0, apenas para câmaras multiespectrais M3C da LATTICE): aparece numa pasta de formato, ao lado das pastas de produtos, quando a separação de bandas reduz valores negativos a zero num produto de radiação ou refletância. Contém uma máscara por pixel por cada produto afetado, por exemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. As máscaras nunca se encontram dentro de uma pasta de produto, pelo que uma pasta de produto pode ser entregue ao software de fotogrametria tal como está. Ver [Máscaras de limitação](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Todos os produtos exportados mantêm o nome do ficheiro de ORIGEM.**Uma exportação de radiação de `capture_..._raw.tif` continua a chamar-se `capture_..._raw.tif` — apenas se encontra em `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do ficheiro**, pelo que uma pesquisa por `*radiance*.tif` não encontra nada; em vez disso, procure pelo diretório.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Quantos ficheiros devem existir?

Não conte com base numa fórmula — o número de resultados depende dos produtos que foram ativados e dos que se aplicam a cada câmara (por exemplo, as câmaras RGB não têm radiação/refletância). A contagem oficial encontra-se no registo: a linha final `[RUN-SUMMARY]` indica exatamente quantos ficheiros foram gravados, e as linhas de sugestão explicam tudo o que foi omitido.

***

## Análise das imagens processadas

### Pré-visualização rápida no Explorador de Ficheiros

**Pré-visualização integrada do Windows:**

1. Navegue até uma pasta de produtos (por exemplo, `tiff16/Reflectance_Calibrated_Images/`)
2. Selecione um ficheiro de imagem
3. A pré-visualização aparece no painel de pré-visualização do Explorador do Windows

4. Utilize as setas do teclado para percorrer as imagens

### Pré-visualização em visualizadores de imagens externos

**Visualizadores recomendados:**

* **QGIS** - Software SIG gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** - Visualizador de imagens rápido e leve (suporta TIFF)
* **Adobe Photoshop** - Edição profissional (suporte a TIFF)
* **GIMP** - Alternativa gratuita ao Photoshop
* **Fotos do Windows** - Visualização básica (pode não suportar TIFF de 16 bits)

### Pré-visualização no Visualizador de Imagens do Chloros


Utilize o Visualizador de Imagens integrado do Chloros para uma visualização avançada:

1. Clique numa miniatura de imagem no Navegador de Ficheiros
2. A imagem abre-se na área principal de pré-visualização
3. Clique no separador **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
na barra lateral esquerda
4. Utilize [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para uma análise interativa

Consulte [Visualizador de Imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Leitura dos valores de refletância dos píxeis (SIG / Pix4D / Scripts)

A refletância é armazenada como um valor inteiro DN, e **o valor de DN que corresponde a ρ = 1,0 depende da câmara de origem**:

| Origem          | ρ = 1,0 corresponde a | Como determinar                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margem até ρ 2,0) | A etiqueta XMP `Chloros:PixelScale=32768` está inserida no ficheiro |
| Survey3          | **65535** (limitado a ρ 1,0)     | Não existe a etiqueta `Chloros:PixelScale` — essa ausência é o indício (os produtos Survey3 incluem outras etiquetas de processamento `Chloros:*`) |

**Leia a etiqueta `Chloros:PixelScale` e divida por esse valor**, em vez de assumir um valor genérico de 65535 — dividir a refletância do LATTICE por 65535 reduz silenciosamente todos os valores para metade. Um ficheiro de refletância anterior à versão 1.3.0, gravado em 8 bits, foi cortado, não redimensionado, e não possui qualquer etiqueta de escala: reexporte-o em 16 bits ou 32 bits, em vez de o dividir. Consulte [Formatos de Imagem de Saída](../output-image-formats.md) para obter a informação completa.

***

## Metadados Transferidos para as Exportações

Cada produto mantém o **bloco GPS**da captura de origem e o seu**sub-IFD EXIF**, pelo que uma
exportação transporta os ficheiros `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber`, bem como a georreferência.

{% hint style="warning" %}
**Se um ortomosaico for gerado numa escala absurda, verifique primeiro o `FocalLength`.**
O Pix4D calcula a distância entre amostras no solo a partir da distância focal mais a altitude. Sem a etiqueta,
resulta numa escala extremamente errada — num voo medido com 49 capturas, um pomar de laranjeiras de 411 m × 160 m
foi reconstruído como tendo 47,8 km × 13 km, produzindo uma ortomosaico de 455 megapixéis composta principalmente
por espaço vazio. A lentidão na divisão em mosaicos e um ficheiro inesperadamente grande são sintomas disto, não
problemas separados.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nem *todas* as etiquetas são copiadas. As etiquetas estruturais do IFD0 são deliberadamente deixadas de fora (copiá-las
corrompe a saída do LATTICE), e as etiquetas `ExifImageWidth` / `ExifImageHeight` são excluídas
porque descrevem a captura original — uma exportação que foi redimensionada acabaria, de outra forma,
por reivindicar dimensões que o seu próprio raster contradiz.

O Chloros também grava as suas próprias etiquetas XMP `Chloros:*`, que descrevem o que foi feito a cada produto
(por exemplo, `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). A partir da versão 1.3.0, os produtos do Survey3 incluem-nas em todos os modos de processamento; os produtos de índice descrevem o próprio índice (`Chloros:ProcessingLevel=index`, a
fórmula e se foi calculado a partir da refletância ou da radianância), em vez de repetirem as
etiquetas dos dados brutos de origem; e os produtos em pastas com nomes com mais de 260 caracteres também mantêm os seus metadados. Consulte [Formatos de Imagem de Saída](../output-image-formats.md#index-products).

***

## Análise do Registo de Depuração

### Verificar se existem avisos ou erros

1. Abra o separador **Registo de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Percorra as mensagens
3. Procure avisos a amarelo ou erros a vermelho
4. Leia as linhas `[RUN-SUMMARY]` e quaisquer sugestões
5. Contacte o suporte da MAPIR para obter assistência

### Guardar o registo

Para manter um registo do processamento ou para enviar ao Suporte da MAPIR:

1. Clique no botão **«Copiar»**ou**«Descarregar»**

2. Guarde como ficheiro de texto na pasta do projeto
3. Inclua-o na documentação do projeto
4. Envie para o suporte da MAPIR caso surjam problemas

Para enviar um problema diretamente à MAPIR, utilize o ícone de bug (**Reportar um bug**) na parte superior do separador «Registo de Depuração» — consulte [Reportar um bug a partir do separador «Registo de Depuração»](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Problemas comuns de saída e soluções

### Problema: Ficheiros de saída em falta

**Possíveis causas:**

* O produto não se aplica a essa câmara (por exemplo, radianância/refletância para câmaras RGB — o registo indica isso)
* Faltava uma referência obrigatória (por exemplo, refletância sem alvo e sem `.daq` de radiação descendente)
* A caixa de seleção de exportação do produto estava desativada nas «Definições do projeto»
* Esgotou-se o espaço em disco durante a exportação

**Soluções:**

1. Verifique as sugestões `[RUN-SUMMARY]` e as linhas `[EXPORT-CHECK]` no Registo de Depuração — estas explicam as omissões por câmara
2. Verifique as caixas de seleção de exportação do produto em [Definições do Projeto](adjusting-project-settings.md)
3. Verifique se havia espaço em disco suficiente
4. Reprocesse após corrigir a causa

### Problema: Bordas escuras ou claras (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmara/lente não consta na base de dados de perfis dChloros

* Vinheta extrema, para além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta estava ativada nas Definições do projeto
2. Verifique se o modelo da câmara foi detetado corretamente
3. Contacte o suporte da MAPIR se a vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Não foram detetados alvos de calibração
* Foi selecionado o modelo errado de alvo de calibração
* Calibração de refletância desativada
* Imagens dos alvos de má qualidade

**Soluções:**

1. Verifique se a calibração de refletância estava ativada
2. Verifique as mensagens «Alvo encontrado» no registo de depuração
3. Analise a qualidade das imagens dos alvos
4. Reprocesse com os alvos adequados marcados

### Problema: Os valores de NDVI parecem estar errados

**Intervalos de NDVI esperados:**

* **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação escassa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9**Se os valores estiverem fora destes intervalos:**

1. Verifique se a calibração da refletância foi aplicada
2. Verifique se o registo do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detetados
4. Certifique-se de que foi detetado o modelo correto da câmara
5. Analise o momento e as condições de captura das imagens-alvo
6. Se estiver a calcular os índices por conta própria a partir de ficheiros de refletância, confirme se dividiu pelo valor `Chloros:PixelScale` do ficheiro (ver acima)

***

## Utilização das suas imagens processadas

### Para fotogrametria / criação de ortomosaicos

**Fluxo de trabalho recomendado:**

1.**Importar imagens de refletância calibradas** para software de fotogrametria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Conservar os metadados EXIF**: Certifique-se de que os dados GPS são preservados para georreferenciamento
3. **Fluxos de trabalho calibrados**: Utilize imagens de refletância para obter precisão científica — a refletância LATTICE contém as etiquetas de calibração XMP que o Pix4D lê
4. **Processar mosaicos de índice**: Crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exportar GeoTIFF georreferenciado**: Para utilização em aplicações SIG

### Para análise SIG

**Fluxo de trabalho recomendado:**

1.**Carregar no QGIS, ArcGIS ou similar**

2.**Utilize imagens de refletância TIFF de 16 bits** para análise multibanda (divida pelo valor `Chloros:PixelScale` do ficheiro)
3. **Utilize imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas a utilizar
4. **Calculadora raster**: Combine bandas para análises personalizadas
5. **Exportar**: Crie mapas de classificação, deteção de alterações e mapas de saúde da vegetação

### Para análise direta / elaboração de relatórios

**Fluxo de trabalho recomendado:**

1.**Utilize imagens de índice com cores LUT** para relatórios visuais
2. **Extrair estatísticas**: NDVI médio por campo/parcela
3. **Séries temporais**: Comparar índices entre várias sessões
4. **Gerar relatórios**: Incluir mapas, estatísticas e visualizações***

## Arquivamento e cópia de segurança

### Estratégia de cópia de segurança recomendada

**O que guardar:**

* ✅ **Imagens RAW/JPG originais ou capturas brutas do LATTICE** — Arquivar numa unidade separada ou na nuvem; os ficheiros brutos são a fonte do pipeline e tudo o resto pode ser regenerado a partir deles
* ✅ **Ficheiros do sensor de luz `.daq` / `.csv`** — Necessários para recalcular a refletância posteriormente
* ✅ **Resultados processados** — Guarde as imagens calibradas e os índices
* ✅ **Pasta do projeto** (`project.json` e ficheiros associados) — Contém todas as definições para reprocessamento, caso seja necessário
* ✅ **Registo de depuração** — Documenta os detalhes do processamento
* ✅ **Imagens de alvo de calibração** — Para verificação e reprocessamento

**Recomendações de armazenamento:**

* **Cópia de segurança imediata**: disco rígido externo
* **Arquivo a longo prazo**: armazenamento na nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: guarde 2 a 3 cópias em locais diferentes***

## Próximas execuções de processamento

### Reutilização das definições do projeto

Se vier a processar conjuntos de dados semelhantes no futuro:

1. **Guarde o modelo do projeto** (se ainda não o tiver feito)
2. **Crie um novo projeto** utilizando o modelo guardado
3. **Importe novas imagens**

4.**Processe**com definições idênticas para garantir a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:**Opção 1: GUI - Vários projetos**

* Crie um projeto separado para cada sessão
* Utilize definições de modelo consistentes
* Processe um de cada vez

**Opção 2: Chloros CLI (apenas Chloros+)**

* Automatizar o processamento em lote
* Processar várias pastas com scripts
* Consulte a [Documentação da CLI](../CLI.md) e a [Referência da CLI](../reference/cli-reference.md)

**Opção 3: SDK Python do Python (apenas Chloros+)**

* Controlo programático
* Integração com pipelines de análise
* Consulte a [Documentação da API](../api-python-sdk.md) e a [Referência do SDK](../reference/sdk-reference.md)

***

## Resolução de problemas no pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Guarde as imagens originais (nunca as elimine)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel «Definições do projeto»
4. Processe novamente — os resultados são guardados nas mesmas pastas do produto, pelo que os ficheiros com o mesmo nome da execução anterior são substituídos

### Processamento de um subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que necessitam de reprocessamento
3. Utilize o mesmo modelo de definições
4. Processe um conjunto de dados mais pequeno

### Obter ajuda

Se tiver problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o registo de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Perguntas frequentes**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)

***

## Resumo: Fluxo de trabalho completo

Concluiu agora o fluxo de trabalho completo de processamento do Chloros:

1. ✅ **Projeto criado** - Consulte [Projetos](../projects.md)
2. ✅ **Adicionou ficheiros** - Consulte [Adicionar ficheiros](adding-files-to-a-project.md)
3. ✅ **Ajustou as definições** - Consulte [Ajustar as definições do projeto](adjusting-project-settings.md)
4. ✅ **Marcou os alvos** - Consulte [Escolher imagens-alvo](choosing-target-images.md)
5. ✅ **Iniciou o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)
6. ✅ **Monitorizou o progresso** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revistos** - Esta página**As suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Funcionalidades avançadas

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualização e análise interativas
* [**Área de testes de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Testes personalizados de índices
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) - Referência completa de índices

### Automatização e integração

* [**Documentação da CLI**](../CLI.md) - Processamento em lote na linha de comandos
* [**Python SDK**](../api-python-sdk.md) - Automatização programática
* [**Funcionalidades do Chloros+**](../#chloros) - Capacidades avançadas de processamento

### Suporte e Formação

* [**Perguntas frequentes**](../faq.md) - Respostas às perguntas mais comuns
* [**Alvos de calibração**](../calibration-targets.md) - Compreender a calibração da refletância
* [**Câmaras suportadas**](../supported-cameras.md) - Hardware compatível
