# Configurações do projeto

As configurações do projeto<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">a barra lateral em Chloros permite configurar todos os aspectos de processamento de imagem, detecção de alvo de calibração, cálculos de índice multiespectral e opções de exportação para seu projeto. Essas configurações são salvas com o seu projeto e podem ser salvas como modelos para reutilização em vários projetos.

## Acessando as configurações do projeto

Para acessar as configurações do projeto:

1. Abra um projeto no Chloros
2. Clique na guia **Configurações do projeto**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">na barra lateral esquerda
3. O painel de configurações exibirá todas as opções de configuração disponíveis organizadas por categoria

***

## Detecção de alvo

Essas configurações são controladas como o Cloros detecta e processa alvos de ocorrência em suas imagens.

### Área mínima de amostragem de descoberta (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10.000 pixels
* **Padrão**: 25 pixels
* **Descrição**: Defina a área mínima (em pixels) necessária para que uma região bloqueada seja considerada uma amostra de alvo de deficiência válida. Valores menores detectarão alvos menores, mas poderão aumentar os falsos positivos. Valores maiores regiões arquivos alvo maiores e mais claros para detecção.
* **Quando ajustar**:
  * Aumente se você estiver encontrando detecções falsas em pequenos artistas de imagem
  * Diminua se seus alvos sofisticados parecem pequenos em suas imagens e não estão sendo detectados

### Clustering de destino mínimo (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Padrão**: 60
* **Descrição**: Controle o limite de agrupamento para grupos de regiões específicas para detecção de alvos específicos. Valores mais altos excluem que núcleos mais semelhantes sejam agrupados, resultando em uma detecção de alvo mais conservadora. Valores mais baixos permitem mais variação de núcleos dentro de um grupo-alvo.
* **Quando ajustar**:
  * Aumente se os alvos de ocorrência foram divididos em múltiplas detecções
  * Diminua se os alvos de atração com variação de cor não estão sendo totalmente detectados

***

## Processamento

Essas configurações são controladas como o Cloros processa e calibra suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Habilitado (marcado)
* **Descrição**: Aplicação de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é uma aparência óptica comum em que os cantos e bordas de uma imagem parecem mais escuros que o centro devido às características da lente.
* **Quando desativar**: Desative apenas se a combinação câmera/lente já tiver aplicada a correção da vinheta ou se você quiser ajustar manualmente a vinheta no pós-processamento.

### Calibração de refletância/balanço de branco

* **Tipo**: Caixa de seleção
* **Padrão**: Habilitado (marcado)
* **Descrição**: permite a ocorrência automática de reflexão usando alvos descobertos em suas imagens. Isso normaliza os valores de refletância em todo o conjunto de dados e garante densidade consistente, independentemente das condições de iluminação.
* **Quando desativar**: desative apenas se desejar selecionar imagens brutas e não calibradas ou se estiver usando um fluxo de trabalho de ocorrência diferente.

### Método Debayer

* **Tipo**: seleção suspensa
* **Opções**:
  * Alta qualidade (mais rápida) - Atualmente a única opção disponível
* **Padrão**: Alta qualidade (mais rápido)
* **Descrição**: Selecionado o algoritmo de demosaicing usado para converter dados brutos do sensor de padrão Bayer em imagens coloridas. O método "Alta qualidade (mais rápida)" fornece um equilíbrio ideal entre velocidade de processamento e qualidade de imagem.
* **Nota**: Métodos adicionais de debayer podem ser acrescentados em versões futuras do Chloros.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3.600 segundos
* **Padrão**: 0 segundos
* **Descrição**: Defina o intervalo de tempo mínimo (em segundos) entre o uso dos alvos específicos. Quando definido como 0, o Cloros usará todos os alvos descobertos. Quando definido para um valor mais alto, o Cloros usará apenas alvos de coexistência separados, pelo menos esse número de segundos, aumentando o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de encontros.
* **Quando ajustar**:
  * Defina como 0 para máxima proteção de escuridão quando as condições de iluminação variam
  * Aumente (por exemplo, para 60-300 segundos) para um processamento mais rápido quando a iluminação for consistente e você tiver imagens alvo de benefícios especiais

### Deslocamento do fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: especifica a posição do fuso horário (em horas a partir do UTC) para carimbos de dados/hora dos dados do sensor de luz. Isso é usado ao processar arquivos de dados PPK (Post-Processed Kinematic) para garantir a sincronização de tempo correta entre as capturas de imagem e os dados de GPS.
* **Quando ajustar**: defina para a posição do fuso localização local se seus dados PPK usam a hora local em vez de UTC. Por exemplo:
  * Horário do Pacífico: -8 ou -7 (dependendo do horário de verão)
  * Horário do Leste: -5 ou -4 (dependendo do horário de verão)
  * Horário da Europa Central: +1 ou +2 (dependendo do horário de verão)

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Padrão**: Desativado (desmarcado)
* **Descrição**: Permite o uso de correções cinemáticas pós-processadas (PPK) de gravadores MAPIR DAQ contendo GPS (GNSS). Quando ativado, o Chloros usará qualquer arquivo de log .daq contendo dados de pinos de exposição no diretório do seu projeto e aplicará correções precisas de geolocalização às suas imagens.
* **Requisito**: o arquivo de log .daq com entradas de pinos de exposição deve estar presente no diretório do seu projeto
* **Quando ativar**: É raro sempre ativar a Correção PPK se você tiver entradas de feedback de exposição em seu arquivo de log .daq.

### Pino de exposição 1

* **Tipo**: seleção suspensa
* **Visibilidade**: visível apenas quando "Aplicar correções PPK" está ativado E os dados de exposição estão disponíveis para o Pin 1
* **Opções**:
  * Nomes de modelos de câmeras detectados no projeto
  * "Não usar" - Ignore este alfinete de exposição
* **Padrão**: selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao pino de exposição 1 para sincronização de localização PPK. O pino de exposição registra o momento exato em que o obturador da câmera é acionado, o que é fundamental para uma geolocalização PPK precisa.
* **Comportamento de seleção automática**:
  * Câmera única + pino único: selecione automaticamente a câmera
  * Câmera única + dois pinos: Pino 1 atribuído automaticamente à câmera
  * Várias câmeras: seleção manual necessária

### Pino de exposição 2

* **Tipo**: seleção suspensa
* **Visibilidade**: visível apenas quando "Aplicar correções PPK" está ativado E os dados de exposição estão disponíveis para o Pin 2
* **Opções**:
  * Nomes de modelos de câmeras detectados no projeto
  * "Não usar" - Ignore este alfinete de exposição
* **Padrão**: selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao pino de exposição 2 para sincronização de localização PPK ao usar uma configuração de câmera dupla.
* **Comportamento de seleção automática**:
  * Câmera única + pino único: Pino 2 definido automaticamente para "Não usar"
  * Câmera única + dois pinos: Pino 2 definido automaticamente para "Não usar"
  * Várias câmeras: seleção manual necessária
* **Nota**: A mesma câmera não pode ser atribuída ao Pino 1 e ao Pino 2 simultaneamente.

***

## Índice

Essas configurações permitem configurar índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel especial de configuração de índice
* **Descrição**: Abre um painel interativo onde você pode selecionar e configurar índices de vegetação multiespectrais (NDVI, NDRE, EVI, etc.) para calcular durante o processamento da imagem. Você pode adicionar vários índices, cada um com suas próprias configurações de visualização.
* **Índices disponíveis**: O sistema inclui mais de 30 índices multiespectrais predefinidos, incluindo:
  * NDVI (Índice de Vegetação por Diferença Normalizada)
  * NDRE (Diferença Normalizada RedEdge)
  * EVI (Índice de Vegetação Aprimorado)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * E muito mais (consulte [Fórmulas de índice multiespectral](multispectral-index-formulas.md) para obter a lista completa)
* **Características**:
  * Selecione entre fórmulas de índice predefinidas
  * Configurar gradientes de cores de visualização (LUT - Look-Up Tables)
  * Definir valores limite para análise
  * Crie fórmulas de índice personalizadas

### Fórmulas personalizadas (recurso Cloros +)

* **Tipo**: matriz de definições de fórmulas personalizadas
* **Descrição**: Permite criar e salvar fórmulas de índice multiespectral personalizadas usando matemática de banda. As fórmulas personalizadas são salvas com as configurações do seu projeto e podem ser usadas como índices integrados.
* **Como criar**:
  1. No painel de configuração do índice, procure uma opção de fórmula personalizada
  2. Defina sua fórmula usando identificadores de banda (por exemplo, NIR, Vermelho, Verde, Azul)
  3. Salve a fórmula com um nome descritivo
* **Sintaxe da fórmula**: operações matemáticas padrão são suportadas, incluindo:
  * Aritmética:`+`, `-`, `*`, `/`
  * Parênteses para ordem de operações
  * Referências de banda: NIR, Vermelho, Verde, Azul, RedEdge, Ciano, Laranja, NIR1, NIR2

***

## Exportar

Essas configurações controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrado

* **Tipo**: seleção suspensa
* **Opções**:
  * **TIFF (16 bits)** - Formato TIFF não compactado de 16 bits
  * **TIFF (32 bits, porcentagem)** - TIFF de ponto flutuante de 32 bits com valores de refletância como porcentagens
  * **PNG (8 bits)** - Formato PNG compactado de 8 bits
  * **JPG (8 bits)** - Formato JPEG compactado de 8 bits
* **Padrão**: TIFF (16 bits)
* **Descrição**: Selecionado o formato de arquivo para salvar imagens processadas e calibradas.
* **Recomendações de formato**:
  * **TIFF (16 bits)**: Recomendado para análises científicas e fluxos de trabalho profissionais. Preservar a qualidade máxima dos dados sem artefatos de compactação. Melhor para análise multiespectral e processamento adicional em software GIS.
  * **TIFF (32 bits, porcentagem)**: melhor para fluxos de trabalho que desativa valores de refletância como porcentagens (0-100%). Oferece soluções máximas para soluções radiométricas.
  * **PNG (8 bits)**: Bom para visualização na Web e visualização geral. Tamanhos de arquivo menores com compactação sem perdas, mas faixa dinâmica diminuta.
  * **JPG (8 bits)**: tamanhos de arquivo menores, ideais apenas para visualizações e exibição na Web. Usa compactação com perdas que não é adequada para análise científica.

***

## Salvar modelo de projeto

Este recurso permite que você salve as configurações atuais do projeto como um modelo reutilizável.

* **Tipo**: entrada de texto + botão Salvar
* **Descrição**: insira um nome descritivo para seu modelo de configurações e clique no ícone salvar. O modelo armazenará todas as configurações atuais do projeto (detecção de alvo, opções de processamento, índices e formato de exportação) para fácil reutilização em projetos futuros.
* **Casos de uso**:
  * Crie modelos para diferentes sistemas de câmeras (RGB, multiespectral, NIR)
  * Salve configurações padrão para tipos de culturas ou fluxos de trabalho de análise específicos
  * Compartilhe configurações consistentes entre uma equipe
* **Como usar**:
  1. Defina todas as configurações de projeto desejadas
  2. Insira um nome de modelo (por exemplo, "RedEdge Survey3 NDVI Standard")
  3. Clique no ícone salvar
  4. O modelo agora pode ser carregado ao criar novos projetos

***

## Salvar macarrão do projeto

Esta configuração especifica onde os novos projetos são salvos por padrão.

* **Tipo**: exibição do caminho do diretório + botão Editar
* **Padrão**:`C:\Users\[Username]\Chloros Projects`
* **Descrição**: Mostra o diretório padrão atual onde novos projetos do Chloros são criados. Clique no ícone de edição para selecionar um diretório diferente.
* **Quando mudar**:
  * Defina como uma unidade de rede para colaboração em equipe
  * Mude para uma unidade com mais espaço de armazenamento para grandes conjuntos de dados
  * Organize projetos por ano, cliente ou tipo de projeto em diferentes pastas
* **Observação**: A alteração desta configuração afeta apenas NOVOS projetos. Os projetos existentes permanecem em seus locais originais.

***

## Persistência de configurações

Todas as configurações do projeto são salvas automaticamente com o arquivo do projeto (formato de projeto`.mapir`). Ao reabrir um projeto, todas as configurações serão restauradas exatamente como você deixou.

### Hierarquia de configurações

As configurações são aplicadas na seguinte ordem:

1. **Padrões do sistema** - Padrões integrados definidos pelo Chloros
2. **Configurações do modelo** - Se você carregar um modelo ao criar um projeto
3. **Configurações do projeto salvas** - Configurações salvas com o arquivo do projeto
4. **Ajustes manuais** - Quaisquer alterações feitas durante a sessão atual

### Configurações e processamento de imagem

A maioria das alterações nas configurações (especialmente nas categorias Processamento e Exportação) irá ativar o reprocessamento de imagens para refletir as novas configurações. No entanto, algumas configurações são "somente exportadas" e não desativam o reprocessamento imediato:

* Salvar modelo de projeto
* Diretório de trabalho
* Formato de imagem calibrado (aplicar ao exportar)

***

## Melhores práticas

1. **Comece com os padrões**: as configurações padrão funcionam bem para a maioria dos sistemas de câmera MAPIR e fluxos de trabalho típicos.
2. **Crie modelos**: depois de atualizar as configurações para um fluxo de trabalho ou câmera específica, salve-os como um modelo para garantir consistência entre os projetos.
3. **Teste antes do processamento completo**: ao experimentar novas configurações, teste em um pequeno subconjunto de imagens antes de processar todo o conjunto de dados.
4. **Documente suas configurações**: use nomes de modelos descritivos que indiquem o sistema de câmera, o tipo de processamento e o uso pretendido (por exemplo, "Survey3\_RGB\_NDVI\_Agriculture").
5. **Seleção do formato de exportação**: Escolha o formato de exportação com base no uso final:
   * Análise científica → TIFF (16 bits ou 32 bits)
   * Processamento GIS → TIFF (16 bits)
   * Visualização rápida → PNG (8 bits)
   * Compartilhamento na Web → JPG (8 bits)

***

Para obter mais informações sobre índices multiespectrais em Cloros, consulte a página [Fórmulas de índice multiespectral](multispectral-index-formulas.md).