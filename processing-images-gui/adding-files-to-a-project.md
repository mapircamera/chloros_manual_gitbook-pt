# Adicionando arquivos a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar suas imagens multiespectrais para iniciar o processamento. O navegador de arquivos<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">facilita a importação de imagens e o gerenciamento de seu conjunto de dados.

## Acessando o navegador de arquivos

1. Abra ou chore um projeto no Chloros
2. Clique no ícone **Navegador de arquivos**<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">na barra lateral esquerda
3. O painel Navegador de Arquivos exibirá uma lista de arquivos do seu projeto

{% dica estilo = "info" %}
**Tipos de arquivos suportados**: O Chloros suporta arquivos de imagem RAW+JPG e JPG das câmeras MAPIR Survey3W e Survey3N. Somente RAW+JPG são recomendados.
{% endhint %}

***

## Adicionando imagens ao seu projeto

Existem duas maneiras principais de adicionar imagens ao seu projeto:

### Método 1: adicionar arquivos

Use esta opção para importar arquivos de imagens individuais ou uma pequena seleção de arquivos.

1. Clique no botão **"Adicionar arquivos"** na parte superior do painel do navegador de arquivos
2. Navegue até a pasta que contém suas imagens
3. Selecione um ou mais arquivos de imagem (segure **Ctrl** para selecionar vários arquivos)
4. Clique em **"Abrir"** para importar os arquivos selecionados

### Método 2: adicionar pasta

Use esta opção para importar todas as imagens de uma pasta de uma só vez.

1. Clique no botão **"Adicionar pasta"** na parte superior do painel do navegador de arquivos
2. Navegue e selecione a pasta que contém as imagens da sessão de captura
3. Clique em **"Selecionar pasta"** para importar todas as imagens suportadas dessa pasta

***

## Compreendendo a tabela do navegador de arquivos

Depois que as imagens são importadas, elas aparecem em uma tabela com as seguintes colunas:

### Miniatura

* Pequena visualização de cada imagem
* Clique na miniatura para ver a imagem completa na área de visualização principal

### Nome do arquivo

* Nome do arquivo original da câmera
* Mantém a convenção de nomenclatura de câmeras (por exemplo, IMG\_0001.RAW)

### Carimbo de dados e hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF​​da imagem
* Usado para sincronização PPK e detecção de alvo de falha

### Modelo de câmera

* Configuração de câmera e filtro bloqueados automaticamente
* Exemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Usado para aplicar perfis de processamento corretos

### Coluna de destino (caixa de seleção)

* Marque esta caixa para imagens que contenham alvos de confusão
* Acelera bastante a detecção de alvos durante o processamento
* Consulte [Escolher imagens de destino](choosing-target-images.md) para obter detalhes

***

## Gerenciando arquivos em seu projeto

### Removendo arquivos

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de Arquivos
2. Clique no botão **"Remover selecionados"**
3. Confirme a remoção (os arquivos não são excluídos do disco, apenas removidos do projeto)

### Classificando e Filtrando

* **Classificar por coluna**: clique em qualquer cabeçalho da coluna para classificar as imagens
* **Classificação de carimbo de dados/hora**: útil para organizar sequências de captura cronológica
* **Filtro de modelo de câmera**: um grupo de imagens por tipo de câmera se você estiver usando várias câmeras

***

## Pré-visualização da imagem

### Visualizando imagem completa

Clique em qualquer miniatura de imagem no Navegador de Arquivos para exibir a área de visualização principal:

1. Uma imagem aparece no painel central de visualização
2. Use controles de zoom para ver detalhes da imagem
3. Navegue entre as imagens usando as teclas de seta

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou pressione a tecla ←
* **Próxima imagem**: Clique na seta para a direita ou pressione a tecla →
* **Aumentar/diminuir zoom**: use a roda do mouse ou os botões de zoom
* **Pan**: clique e arraste a imagem quando ampliada

***

## Tratamento de arquivos duplicados

O Chloros detecta e ignora automaticamente arquivos duplicados:

* Arquivos com nomes idênticos são ignorados
* Evita o processamento duplo acidental
* Mensagem de aviso exibida quando duplicatas estiverem bloqueadas

{% dica estilo = "aviso" %}
**Importante**: Não renomeie ou modifique seus arquivos de imagem originais antes de importar. O Chloros depende de nomes de arquivos e metadados originais para o processamento adequado.
{% endhint %}

***

## Conjuntos de dados combinados de câmeras

Se o seu projeto contém imagens de múltiplas câmeras MAPIR:

1. O Chloros detecta automaticamente cada modelo de câmera
2. Cada tipo de câmera é processado com seu perfil de segurança adequado
3. O Navegador de Arquivos exibe o modelo da câmera na coluna Modelo da Câmera
4. O processamento aplica configurações corretas para cada tipo de câmera

**Cenário de exemplo**: Configuração de câmera dupla Survey3W RGN + Survey3N OCN

***

## Melhores práticas

### Organize antes de importar

* Manter as imagens do alvo de encontro na mesma pasta das imagens do levantamento
* Mantenha a estrutura de pastas original da sua câmera/cartão SD
* Não misture conjuntos de dados de sessões diferentes em um projeto

### Nomenclatura de arquivos

* Preservar os nomes dos arquivos originais da câmera (IMG\_0001.RAW, etc.)
* Não renomeie arquivos antes de importar
* Os nomes originais contêm metadados importantes

### Imagens alvo de confusão

* Sempre incluindo 1-2 imagens alvo de encontro por sessão
* Capture alvos antes e depois da sessão de captura
* Coloque os alvos nas mesmas condições de iluminação da área de captura
* Marque as imagens alvo usando a caixa de seleção Destino para acelerar o processamento

***

## Problemas e soluções comuns

### As imagens não aparecem após a importação

**Causas possíveis:**

* Formato de arquivo não suportado (somente RAW+JPG e JPG de câmeras MAPIR)
* As imagens são de câmeras não MAPIR (consulte [Câmeras suportadas](../supported-cameras.md))
* Corrupção de arquivo ou transferência incompleta do cartão SD

**Solução**: verifique a compatibilidade do formato do arquivo e do modelo da câmera

### Modelo de câmera não detectado

**Causas possíveis:**

* Metadados EXIF​​modificados
* Imagens editadas em software externo
* Transferência de arquivo incompleta

**Solução**: Reimporte arquivos originais e não modificados da câmera/cartão SD

### Carimbos de dados e horas ausentes

**Causas possíveis:**

* O relógio da câmera não está definido corretamente
* Dados EXIF ​​removidos por software externo

**Solução**: verifique se as configurações de tempo da câmera estavam corretas durante a captura

***

## Próximas etapas

Depois que seus arquivos foram importados:

1. **Revise a lista de arquivos** - Certifique-se de que todas as imagens foram incluídas corretamente
2. **Verifique os modelos de câmera** - Verifique a detecção correta da câmera
3. **Marcar imagens alvo** - Consulte [Escolher imagens alvo](choosing-target-images.md)
4. **Ajustar configurações** - Configurar as opções de processamento em [Configurações do projeto](adjusting-project-settings.md)
5. **Iniciar processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).