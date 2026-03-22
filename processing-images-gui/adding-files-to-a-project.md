# Adicionar ficheiros a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar as suas imagens multiespectrais para iniciar o processamento. O separador «File Browser»<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e a gestão do seu conjunto de dados.

## Aceder ao Navegador de ficheiros

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de ficheiros** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel do Navegador de Ficheiros exibirá a lista de ficheiros do seu projeto

{% hint style="info" %}
**Tipos de ficheiros suportados**: O Chloros suporta ficheiros de imagem RAW+JPG e JPG das câmaras MAPIR, Survey3W e Survey3N. Recomenda-se apenas o formato RAW+JPG.
{% endhint %}

***

## Adicionar imagens ao seu projeto

Existem duas formas principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar ficheiros

Utilize esta opção para importar ficheiros de imagem individuais ou uma pequena seleção de ficheiros.

1. Clique no botão **&quot;Adicionar ficheiros&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> na parte superior do painel do Navegador de Ficheiros
2. Navegue até à pasta que contém as suas imagens
3. Selecione um ou mais ficheiros de imagem (mantenha premida a tecla **Ctrl** para selecionar vários ficheiros)
4. Clique em **&quot;Abrir&quot;** para importar os ficheiros selecionados

### Método 2: Adicionar pasta

Utilize esta opção para importar todas as imagens de uma pasta de uma só vez.

1. Clique no botão **&quot;Adicionar pasta&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> na parte superior do painel do Navegador de Ficheiros
2. Navegue até à pasta que contém as imagens da sua sessão de captura e selecione-a
3. Clique em **&quot;Selecionar Pasta&quot;** para importar todas as imagens suportadas dessa pasta***

## Compreender a Tabela do Navegador de Ficheiros

Depois de importadas, as imagens aparecem numa tabela com as seguintes colunas:

### Nome do Ficheiro

* Nome de ficheiro original da câmara
* Mantém a convenção de nomenclatura da câmara (por exemplo, IMG\_0001.RAW)

### Carimbo de data/hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF da imagem
* Utilizado para sincronização PPK e deteção de alvos de calibração

### Modelo da câmara

* Configuração da câmara e do filtro detetada automaticamente
* Exemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Utilizado para aplicar os perfis de processamento corretos

### Coluna Alvo (Caixa de seleção)

* Marque esta caixa para imagens que contenham alvos de calibração
* Acelera significativamente a deteção de alvos durante o processamento
* Consulte [Escolher imagens-alvo](choosing-target-images.md) para mais detalhes

### Visualizar metadados da imagem

Clicar no botão de alternância no canto superior direito, acima da tabela, mostra os metadados da imagem selecionada na área da grelha de imagens.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Gerir ficheiros no seu projeto

### Remover ficheiros

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de ficheiros
2. Clique no botão **&quot;Remover selecionados&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> 3. Confirme a remoção (os ficheiros não são eliminados do disco, apenas removidos do projeto)

### Ordenar e filtrar

* **Ordenar por coluna**: Clique em qualquer cabeçalho de coluna para ordenar as imagens
* **Ordenação por data e hora**: Útil para organizar sequências de captura cronológicas
* **Filtro por modelo de câmara**: Agrupe as imagens por tipo de câmara se estiver a utilizar várias câmaras***

## Pré-visualização da imagem

### Visualização da imagem completa

Clique em qualquer miniatura de imagem no Navegador de ficheiros para a exibir na área de pré-visualização principal:

1. A imagem aparece no painel de pré-visualização central
2. Use os controlos de zoom para inspecionar os detalhes da imagem
3. Navegue entre as imagens usando as teclas de seta

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou pressione a tecla ←
* **Imagem seguinte**: Clique na seta para a direita ou prima a tecla →
* **Aumentar/Diminuir zoom**: Utilize a roda do rato ou os botões de zoom
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

## Conjuntos de dados de câmaras mistos

Se o seu projeto contiver imagens de várias câmaras MAPIR:

1. O Chloros deteta automaticamente cada modelo de câmara
2. Cada tipo de câmara é processado com o seu perfil de calibração apropriado
3. O Navegador de Ficheiros exibe o modelo da câmara na coluna Modelo da Câmara
4. O processamento aplica as definições corretas para cada tipo de câmara

**Exemplo de cenário**: Survey3W RGN + Survey3N OCN configuração de câmara dupla***

## Melhores práticas

### Organizar antes da importação

* Mantenha as imagens de alvo de calibração na mesma pasta que as imagens de levantamento
* Mantenha a estrutura de pastas original da sua câmara/cartão SD
* Não misture conjuntos de dados de sessões diferentes num único projeto

### Nomeação de ficheiros

* Preserve os nomes de ficheiro originais da câmara (IMG\_0001.RAW, etc.)
* Não renomeie os ficheiros antes da importação
* Os nomes originais contêm metadados importantes

### Imagens de alvo de calibração

* Inclua sempre 1-2 imagens de alvo de calibração por sessão
* Capture os alvos antes e depois da sessão de captura
* Coloque os alvos nas mesmas condições de iluminação que a área de captura
* Marque as imagens de alvo utilizando a caixa de seleção «Alvo» para acelerar o processamento

***

## Problemas comuns e soluções

### Imagens não aparecem após a importação

**Possíveis causas:**

* Formato de ficheiro não suportado (apenas RAW+JPG e JPG de câmaras MAPIR)
* As imagens são de câmaras que não são MAPIR (consulte [Câmaras suportadas](../supported-cameras.md))
* Ficheiro corrompido ou transferência incompleta do cartão SD

**Solução**: Verifique a compatibilidade do formato de ficheiro e do modelo da câmara

### Modelo da câmara não detetado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência de ficheiros incompleta

**Solução**: Reimporte os ficheiros originais, não modificados, da câmara/cartão SD

### Falta de carimbos de data/hora

**Possíveis causas:**

* Relógio da câmara não definido corretamente
* Dados EXIF removidos por software externo

**Solução**: Verifique se as definições de hora da câmara estavam corretas durante a captura***

## Próximos passos

Depois de importar os seus ficheiros:

1. **Revise a lista de ficheiros** - Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos das câmaras** - Verifique se a deteção da câmara está correta
3. **Marque as imagens de destino** - Consulte [Escolher imagens de destino](choosing-target-images.md)
4. **Ajuste as definições** - Configure as opções de processamento em [Definições do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustar as definições do projeto](adjusting-project-settings.md).
