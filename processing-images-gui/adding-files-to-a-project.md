# Adicionar ficheiros a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar as suas imagens multiespectrais para iniciar o processamento. O separador Navegador de ficheiros<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e o gerenciamento do seu conjunto de dados.

## Acessando o Navegador de arquivos

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Navegador de ficheiros exibirá a lista de ficheiros do seu projeto

{% hint style=&quot;info&quot; %}
**Tipos de ficheiros suportados**: O Chloros suporta ficheiros de imagem RAW+JPG e JPG das câmaras MAPIR Survey3W e Survey3N. Recomenda-se apenas RAW+JPG.
{% endhint %}

***

## Adicionar imagens ao seu projeto

Existem duas formas principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar ficheiros

Use esta opção para importar ficheiros de imagem individuais ou uma pequena seleção de ficheiros.

1. Clique no botão **&quot;Adicionar ficheiros&quot;** na parte superior do painel Navegador de ficheiros.
2. Navegue até à pasta que contém as suas imagens.
3. Selecione um ou mais ficheiros de imagem (mantenha pressionada a tecla **Ctrl** para selecionar vários ficheiros).
4. Clique em **&quot;Abrir&quot;** para importar os ficheiros selecionados.

### Método 2: Adicionar pasta

Use esta opção para importar todas as imagens de uma pasta de uma só vez.

1. Clique no botão **&quot;Adicionar pasta&quot;** na parte superior do painel Navegador de ficheiros
2. Navegue até à pasta que contém as imagens da sua sessão de captura e selecione-a
3. Clique em **&quot;Selecionar pasta&quot;** para importar todas as imagens suportadas dessa pasta

***

## Compreender a tabela do Navegador de ficheiros

Depois de importadas, as imagens aparecem numa tabela com as seguintes colunas:

### Miniatura

* Pequena pré-visualização de cada imagem
* Clique na miniatura para ver a imagem completa na área de pré-visualização principal

### Nome do ficheiro

* Nome do ficheiro original da câmara
* Mantém a convenção de nomenclatura da câmara (por exemplo, IMG\_0001.RAW)

### Carimbo de data/hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF da imagem
* Usado para sincronização PPK e deteção de alvos de calibração

### Modelo da câmara

* Configuração da câmara e do filtro detetada automaticamente
* Exemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Usado para aplicar perfis de processamento corretos

### Coluna Alvo (caixa de seleção)

* Marque esta caixa para imagens que contêm alvos de calibração
* Acelera significativamente a deteção de alvos durante o processamento
* Consulte [Escolhendo imagens alvo](choosing-target-images.md) para obter detalhes

***

## Gerenciando ficheiros no seu projeto

### Removendo ficheiros

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de ficheiros
2. Clique no botão **&quot;Remover selecionados&quot;**
3. Confirme a remoção (os ficheiros não são excluídos do disco, apenas removidos do projeto)

### Classificar e filtrar

* **Classificar por coluna**: Clique em qualquer cabeçalho de coluna para classificar as imagens
* **Classificação por data e hora**: útil para organizar sequências de captura cronológicas
* **Filtro de modelo de câmara**: agrupe imagens por tipo de câmara se estiver a utilizar várias câmaras

***

## Pré-visualização de imagens

### Visualizar imagem completa

Clique em qualquer miniatura de imagem no Navegador de ficheiros para exibi-la na área de pré-visualização principal:

1. A imagem aparece no painel de pré-visualização central
2. Use os controlos de zoom para inspecionar os detalhes da imagem
3. Navegue entre as imagens usando as teclas de seta

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou pressione a tecla ←
* **Imagem seguinte**: Clique na seta para a direita ou pressione a tecla →
* **Aumentar/diminuir zoom**: Use a roda do mouse ou os botões de zoom
* **Panorâmica**: Clique e arraste na imagem quando estiver com zoom

***

## Tratamento de ficheiros duplicados

O Chloros deteta e ignora automaticamente ficheiros duplicados:

* Os ficheiros com nomes idênticos são ignorados
* Evita o processamento duplo acidental
* É exibida uma mensagem de aviso quando são detetadas duplicatas

{% hint style=&quot;warning&quot; %}
**Importante**: Não renomeie nem modifique os seus ficheiros de imagem originais antes de importar. O Chloros depende dos nomes de ficheiros e metadados originais para um processamento adequado.
{% endhint %}

***

## Conjuntos de dados de câmaras mistas

Se o seu projeto contiver imagens de várias câmaras MAPIR:

1. O Chloros deteta automaticamente cada modelo de câmara
2. Cada tipo de câmara é processado com o seu perfil de calibração apropriado
3. O Navegador de ficheiros exibe o modelo da câmara na coluna Modelo da câmara
4. O processamento aplica as configurações corretas para cada tipo de câmara

**Exemplo de cenário**: Survey3W RGN + Survey3N OCN configuração de câmara dupla

***

## Melhores práticas

### Organize antes de importar

* Mantenha as imagens de alvo de calibração na mesma pasta que as imagens do levantamento
* Mantenha a estrutura original da pasta da sua câmara/cartão SD
* Não misture conjuntos de dados de sessões diferentes num único projeto

### Nomeação de ficheiros

* Preserve os nomes de ficheiros originais da câmara (IMG\_0001.RAW, etc.)
* Não renomeie os ficheiros antes da importação
* Os nomes originais contêm metadados importantes

### Imagens de alvo de calibração

* Inclua sempre 1-2 imagens de alvo de calibração por sessão
* Capture os alvos antes e depois da sessão de captura
* Coloque os alvos nas mesmas condições de iluminação da área de captura
* Marque as imagens de alvo usando a caixa de seleção Alvo para acelerar o processamento

***

## Problemas comuns e soluções

### Imagens não aparecem após a importação

**Possíveis causas:**

* Formato de ficheiro não suportado (apenas RAW+JPG e JPG de câmaras MAPIR)
* As imagens são de câmaras que não são MAPIR (consulte [Câmaras suportadas](../supported-cameras.md))
* Ficheiro corrompido ou transferência incompleta do cartão SD

**Solução**: Verifique a compatibilidade do formato do ficheiro e do modelo da câmara

### Modelo da câmara não detetado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência incompleta do ficheiro

**Solução**: Reimporte os ficheiros originais, não modificados, da câmara/cartão SD

### Carimbos de data/hora em falta

**Possíveis causas:**

* Relógio da câmara não definido corretamente
* Dados EXIF removidos por software externo

**Solução**: Verifique se as configurações de hora da câmara estavam corretas durante a captura

***

## Próximos passos

Depois de importar os seus ficheiros:

1. **Revise a lista de ficheiros** - Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos de câmara** - Verifique se a deteção da câmara está correta
3. **Marque as imagens de destino** - Consulte [Escolher imagens de destino](choosing-target-images.md)
4. **Ajuste as configurações** - Configure as opções de processamento em [Configurações do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustar as definições do projeto](adjusting-project-settings.md).
