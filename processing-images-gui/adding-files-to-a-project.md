# Adicionar ficheiros a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar as suas imagens multiespectrais para iniciar o processamento. O separador «Navegador de ficheiros»<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e a gestão do seu conjunto de dados.

## Aceder ao Navegador de ficheiros

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de ficheiros** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel «Navegador de ficheiros» exibirá a lista de ficheiros do seu projeto

{% hint style="info" %}
**Tipos de ficheiros suportados**:

* **Survey3W / Survey3N**: pares RAW+JPG e imagens JPG (recomenda-se RAW+JPG)
* **LATTICE**: capturas `.tif` / `.tiff` — gravadas pelo controlo da câmara do Chloros ou por um hub LATTICE
* **Dados do sensor de luz**: gravações `.daq` (DAQ-U/M/E) e registos de luz descendente DAQ-M `.csv` — importados juntamente com as imagens para realizar a calibração da refletância
{% endhint %}

***

## Adicionar imagens ao seu projeto

Existem duas formas principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar ficheiros

Utilize esta opção para importar ficheiros de imagem individuais ou uma pequena seleção de ficheiros.

1. Clique no botão **«Adicionar ficheiros»** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line">, na parte superior do painel «Navegador de ficheiros»
2. Navegue até à pasta que contém as suas imagens
3. Selecione um ou mais ficheiros de imagem (mantenha premida a tecla **Ctrl** para selecionar vários ficheiros)
4. Clique em **«Abrir»** para importar os ficheiros selecionados

### Método 2: Adicionar pasta

Utilize esta opção para importar todas as imagens de uma pasta de uma só vez. Pode selecionar **várias pastas** numa única caixa de diálogo.

1. Clique no botão **«Adicionar pasta»** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">, na parte superior do painel «Navegador de ficheiros»
2. Navegue até às pastas que contêm as imagens da sua sessão de captura e selecione-as
3. Clique em **«Selecionar pasta»** para importar todas as imagens suportadas

{% hint style="info" %}
**Os ficheiros que não conseguem ser carregados são assinalados.** Se uma pasta contiver ficheiros que o Chloros reconhece mas não consegue carregar, um aviso informa-o — as imagens não desaparecem silenciosamente da grelha.
{% endhint %}

***

## Importação de pastas de captura LATTICE

As capturas LATTICE são guardadas com **uma subpasta por nível de exportação** — por exemplo, `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — com o ficheiro correspondente de fluxo descendente `.daq` na raiz:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Selecione «Adicionar pasta» na raiz das capturas** (`output/` acima). Quando a pasta selecionada não contém imagens, mas possui subpastas, o Chloros percorre-as automaticamente — as subpastas do nível e a pasta raiz `.daq` são todas recolhidas de uma só vez.**Como as capturas são importadas:*** Cada captura é importada como uma **única imagem**, agrupada por captura (não uma entrada por nível). Os outros níveis da mesma captura aparecem como modos de visualização dessa única imagem.
* **O processamento começa sempre a partir do fotograma bruto.** Os outros níveis são visíveis, mas apenas o `raw` é submetido ao pipeline — reprocessar um produto já processado aplicaria as correções duas vezes, pelo que o Chloros recusa fazê-lo. Uma exportação reimportada nunca pode ocupar o espaço de um ficheiro bruto de uma captura.
* Uma pasta de capturas guardada **sem** ficheiros brutos é importada e apresentada normalmente, mas o processamento ignora-a e indica isso no registo. (O sinalizador da CLI `--input-level` pode forçar um ponto de entrada para este caso — consulte [a Referência da CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* A partir da versão 1.3.0, uma pasta de capturas também pode conter uma pasta `clamp_masks/` (máscaras por pixel para os níveis de radiação e refletância — consulte [Formatos de Imagem de Saída](../output-image-formats.md#clamp-masks)). As máscaras não são imagens da cena, e o Chloros ignora-as durante a importação, independentemente de onde se encontrem.

**As sessões do hub LATTICE** são importadas da mesma forma: selecione «Adicionar pasta» na pasta da sessão copiada do hub (que contém `raw/` e `previews/`), juntamente com qualquer registo de fluxo descendente do DAQ-M `.csv`. Se a calibração da câmara ou do DAQ ainda não estiver armazenada na cache do seu computador, o Chloros obtém-na automaticamente através do número de série durante a importação (requer ligação à Internet uma vez).***

## Compreender a tabela do navegador de ficheiros

Assim que as imagens forem importadas, aparecem numa tabela com as seguintes colunas:

### Nome do ficheiro

* Nome de ficheiro original da câmara
* Mantém a convenção de nomenclatura da câmara (por exemplo, IMG\_0001.RAW ou capture\_20260816\_101500\_SN213800234\_raw.tif)

### Data e hora

* Data e hora em que a imagem foi capturada
* Extraídas dos metadados EXIF da imagem
* Utilizadas para a correspondência do sensor de luz, sincronização PPK e agendamento de alvos de calibração

### Modelo da câmara

* Configuração da câmara e do filtro detetada automaticamente
* Exemplos do Survey3: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Exemplos do LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Utilizado para aplicar os perfis de processamento corretos

### Coluna «Alvo» (caixa de seleção)

* Assinale esta caixa para imagens que contenham alvos de calibração
* Quando pelo menos uma imagem estiver assinalada, **apenas as imagens assinaladas são analisadas** à procura de alvos
* Consulte [Escolher imagens-alvo](choosing-target-images.md) para mais detalhes

### Visualização dos metadados da imagem

Ao clicar no botão de alternância no canto superior direito, acima da tabela, são apresentados os metadados da imagem selecionada na área da grelha de imagens.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Ficheiros do sensor de luz no seu projeto

* Os ficheiros `.daq` e `.csv` aparecem na lista do Navegador de Ficheiros, mas não são imagens clicáveis — fornecem a irradiância descendente para a calibração da refletância.
* Cada ficheiro `.daq`/`.csv` importado é listado em **Definições do projeto → Sensor de luz DAQ**, onde pode verificar a correção da tampa difusora aplicada a cada ficheiro. Consulte [Ajustar as definições do projeto](adjusting-project-settings.md).
* As gravações que efetuar no separador **Sensores de luz** são adicionadas automaticamente ao projeto aberto — não é necessária qualquer importação manual.***

## Gerir ficheiros no seu projeto

### Remover ficheiros

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de Ficheiros
2. Clique no botão **«Remover selecionados»** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirme a remoção (os ficheiros não são apagados do disco, apenas removidos do projeto)

### Ordenação e filtragem

* **Ordenar por coluna**: clique em qualquer cabeçalho de coluna para ordenar as imagens
* **Ordenação por data e hora**: útil para organizar sequências de captura cronológicas
* **Filtro por modelo de câmara**: agrupe as imagens por tipo de câmara, caso utilize várias câmaras***

## Pré-visualização de imagens

### Visualização da imagem completa

Clique em qualquer miniatura de imagem no Navegador de ficheiros para a apresentar na área principal de pré-visualização:

1. A imagem aparece no painel central de pré-visualização
2. Utilize os controlos de zoom para inspecionar os detalhes da imagem
3. Navegue entre as imagens utilizando as teclas de seta

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou prima a tecla ←
* **Imagem seguinte**: Clique na seta para a direita ou prima a tecla →
* **Aumentar/diminuir o zoom**: Utilize a roda do rato ou os botões de zoom
* **Deslocar**: Clique e arraste na imagem quando estiver com o zoom aumentado***

## Tratamento de ficheiros duplicados

O Chloros deteta e ignora automaticamente ficheiros duplicados:

* Os ficheiros com nomes idênticos são ignorados
* Evita o processamento duplo acidental
* É exibida uma mensagem de aviso quando são detetadas duplicatas

{% hint style="warning" %}
**Importante**: Não renomeie nem modifique os seus ficheiros de imagem originais antes da importação. O Chloros depende dos nomes de ficheiro e metadados originais para um processamento adequado.
{% endhint %}

***

## Conjuntos de dados de câmaras mistas

Se o seu projeto contiver imagens de várias câmaras MAPIR:

1. O Chloros deteta automaticamente cada modelo de câmara — Survey3, LATTICE ou uma combinação
2. Cada tipo de câmara é processado com o seu perfil de calibração adequado
3. O Navegador de Ficheiros apresenta o modelo da câmara na coluna «Modelo da Câmara»
4. Cada câmara recebe a sua própria árvore de pastas de saída quando processada

**Exemplos de cenários**: configuração de duas câmaras Survey3W RGN + Survey3N OCN, ou um conjunto LATTICE com uma câmara principal RGB e vários módulos de banda estreita***

## Melhores práticas

### Organizar antes da importação

* Mantenha as imagens de referência de calibração na mesma pasta que as imagens do levantamento
* Mantenha os ficheiros do sensor de luz `.daq` / `.csv` de cada sessão de captura juntamente com as imagens dessa sessão
* Mantenha a estrutura de pastas original da sua câmara/cartão SD/hub
* Não misture conjuntos de dados de sessões diferentes num único projeto

### Nomeação de ficheiros

* Preserve os nomes de ficheiro originais da câmara (IMG\_0001.RAW, capture\_..., etc.)
* Não renomeie os ficheiros antes da importação
* Os nomes originais contêm metadados importantes

### Imagens de alvo de calibração

* Inclua sempre 1 a 2 imagens de alvo de calibração por sessão (no Survey3; no LATTICE, uma gravação DAQ pode substituí-las — consulte [Escolha de imagens de alvo](choosing-target-images.md))
* Capture as imagens-alvo antes e depois da sessão de captura
* Coloque as imagens-alvo nas mesmas condições de iluminação que a área de captura
* Marque as imagens-alvo utilizando a caixa de seleção «Alvo»

***

## Problemas comuns e soluções

### Imagens que não aparecem após a importação

**Possíveis causas:**

* Formato de ficheiro não suportado (consulte a lista de tipos suportados no topo desta página)
* As imagens são de câmaras que não são da MAPIR (consulte [Câmaras suportadas](../supported-cameras.md))
* Ficheiro corrompido ou transferência incompleta do cartão SD

**Solução**: Verifique a compatibilidade do formato de ficheiro e do modelo da câmara e consulte o aviso de carregamento de ficheiros para identificar exatamente os ficheiros que falharam

### Modelo da câmara não detetado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência de ficheiros incompleta

**Solução**: Reimporte os ficheiros originais, não modificados, da câmara/cartão SD

### Falta de registos de data e hora

**Possíveis causas:**

* Relógio da câmara mal configurado
* Dados EXIF removidos por software externo

**Solução**: Verifique se as definições de hora da câmara estavam corretas durante a captura

### Projeto reaberto indica ficheiros em falta

Se os ficheiros de origem tiverem sido movidos ou eliminados desde a última vez que o projeto foi aberto, o Chloros indica-lhe **quais**os ficheiros que faltam, em vez de abrir uma grelha em branco. Se a própria pasta do projeto tiver sido movida, o Chloros redireciona os seus caminhos de origem para a pasta onde o `project.json` se encontra agora e mantém uma cópia de segurança (`project.json.before-relink-<date>-<time>`). Quando não os encontra, a janela**Ficheiros de origem não encontrados**oferece a opção**Procurar** para o novo local, seguida de Novo projeto ou Eliminar projeto. A partir da CLI, utilize `chloros-cli project open PATH --relink FOLDER`; a partir do SDK, `open_project(path, relink=...)`.***

## Próximos passos

Depois de importar os ficheiros:

1. **Revise a lista de ficheiros** – Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos das câmaras** – Verifique se a deteção das câmaras está correta
3. **Marque as imagens-alvo** — Consulte [Escolher imagens-alvo](choosing-target-images.md)
4. **Ajuste as definições** — Configure as opções de processamento em [Definições do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** — Consulte [Iniciar o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustar as definições do projeto](adjusting-project-settings.md).
