# Definições do projeto

A barra lateral Definições do projeto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral do Chloros permitem configurar todos os aspetos do processamento de imagens, deteção de alvos de calibração, cálculos de índices multiespectrais e opções de exportação para o seu projeto. Estas configurações são guardadas com o seu projeto e podem ser guardadas como modelos para reutilização em vários projetos.

## Aceder às definições do projeto

Para aceder às definições do projeto:

1. Abra um projeto no Chloros
2. Clique no separador **Definições do projeto**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel de definições exibirá todas as opções de configuração disponíveis organizadas por categoria

***

## Detecção de alvos

Estas definições controlam como o Chloros deteta e processa alvos de calibração nas suas imagens.

### Área mínima da amostra de calibração (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10.000 pixels
* **Padrão**: 25 pixels
* **Descrição**: Define a área mínima (em pixels) necessária para que uma região detetada seja considerada uma amostra válida de alvo de calibração. Valores menores detetarão alvos menores, mas podem aumentar os falsos positivos. Valores maiores exigem regiões-alvo maiores e mais nítidas para deteção.
* **Quando ajustar**:
  * Aumente se estiver a obter deteções falsas em pequenos artefactos de imagem.
  * Diminua se os seus alvos de calibração parecerem pequenos nas suas imagens e não estiverem a ser detetados.

### Agrupamento mínimo de alvos (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Padrão**: 60
* **Descrição**: Controla o limite de agrupamento para agrupar regiões de cores semelhantes ao detetar alvos de calibração. Valores mais altos exigem que cores mais semelhantes sejam agrupadas, resultando em uma deteção de alvos mais conservadora. Valores mais baixos permitem mais variação de cores dentro de um grupo de alvos.
* **Quando ajustar**:
  * Aumente se os alvos de calibração estiverem a ser divididos em várias detecções.
  * Diminua se os alvos de calibração com variação de cor não estiverem a ser totalmente detetados.

***

## Processamento

Estas definições controlam como o Chloros processa e calibra as suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Aplica correção de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é um fenómeno óptico comum em que os cantos e bordas de uma imagem parecem mais escuros do que o centro devido às características da lente.
* **Quando desativar**: Desative apenas se a combinação da sua câmara/lente já tiver aplicado a correção de vinheta ou se pretender corrigir manualmente a vinheta no pós-processamento.

### Calibração de refletância/equilíbrio de brancos

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Ativa a calibração automática da refletância usando alvos de calibração detetados nas suas imagens. Isto normaliza os valores de refletância em todo o seu conjunto de dados e garante medições consistentes, independentemente das condições de iluminação.
* **Quando desativar**: Desative apenas se desejar processar imagens brutas e não calibradas ou se estiver a usar um fluxo de trabalho de calibração diferente.

### Método Debayer

* **Tipo**: Seleção suspensa
* **Opções**:
  * Alta qualidade (mais rápido) - Atualmente a única opção disponível
* **Padrão**: Alta qualidade (mais rápido)
* **Descrição**: Seleciona o algoritmo de demosaicing usado para converter dados brutos do sensor do padrão Bayer em imagens coloridas. O método &quot;Alta qualidade (mais rápido)&quot; oferece um equilíbrio ideal entre velocidade de processamento e qualidade da imagem.
* **Observação**: Métodos adicionais de debayer podem ser adicionados em versões futuras do Chloros.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3.600 segundos
* **Padrão**: 0 segundos
* **Descrição**: Define o intervalo de tempo mínimo (em segundos) entre o uso de alvos de calibração. Quando definido como 0, o Chloros usará todos os alvos de calibração detectados. Quando definido como um valor mais alto, o Chloros usará apenas alvos de calibração separados por pelo menos esse número de segundos, reduzindo o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de calibração.
* **Quando ajustar**:
  * Defina como 0 para obter a máxima precisão de calibração quando as condições de iluminação variam
  * Aumente (por exemplo, para 60-300 segundos) para um processamento mais rápido quando a iluminação é consistente e você tem imagens frequentes de alvos de calibração

### Desvio de fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: Especifica o desvio do fuso horário (em horas a partir do UTC) para os carimbos de data/hora dos dados do sensor de luz. Isso é usado ao processar ficheiros de dados PPK (cinemática pós-processada) para garantir a sincronização correta do tempo entre as capturas de imagem e os dados GPS.
* **Quando ajustar**: Defina isto para o desvio do fuso horário local se os seus dados PPK utilizarem a hora local em vez de UTC. Por exemplo:
  * Hora do Pacífico: -8 ou -7 (dependendo do horário de verão)
  * Hora do Leste: -5 ou -4 (dependendo do horário de verão)
  * Hora da Europa Central: +1 ou +2 (dependendo do horário de verão)

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Padrão**: Desativado (desmarcado)
* **Descrição**: Permite o uso de correções cinemáticas pós-processadas (PPK) de gravadores MAPIR DAQ contendo um GPS (GNSS). Quando ativado, o Chloros utilizará quaisquer ficheiros de registo .daq que contenham dados de pinos de exposição no diretório do seu projeto e aplicará correções precisas de geolocalização às suas imagens.
* **Requisito**: o ficheiro de registo .daq com entradas de pinos de exposição deve estar presente no diretório do seu projeto
* **Quando ativar**: recomenda-se ativar sempre a correção PPK se tiver entradas de feedback de exposição no seu ficheiro de registo .daq.

### Pino de exposição 1

* **Tipo**: Seleção suspensa
* **Visibilidade**: Visível apenas quando «Aplicar correções PPK» está ativado E os dados de exposição estão disponíveis para o pino 1
* **Opções**:
  * Nomes de modelos de câmaras detetados no projeto
  * «Não usar» - Ignorar este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao Pino de exposição 1 para sincronização de tempo PPK. O pino de exposição regista o momento exato em que o obturador da câmara é acionado, o que é fundamental para uma geolocalização PPK precisa.
* **Comportamento de seleção automática**:
  * Câmara única + pino único: Seleciona automaticamente a câmara
  * Câmara única + dois pinos: O pino 1 é automaticamente atribuído à câmara
  * Várias câmaras: seleção manual necessária

### Pino de exposição 2

* **Tipo**: seleção suspensa
* **Visibilidade**: visível apenas quando «Aplicar correções PPK» está ativado E os dados de exposição estão disponíveis para o pino 2
* **Opções**:
  * Nomes de modelos de câmaras detetados no projeto
  * «Não usar» - Ignorar este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao Pino de exposição 2 para sincronização de tempo PPK ao usar uma configuração de duas câmaras.
* **Comportamento de seleção automática**:
  * Câmara única + pino único: Pino 2 definido automaticamente como &quot;Não usar&quot;
  * Câmara única + dois pinos: O pino 2 é automaticamente definido como «Não utilizar»
  * Várias câmaras: É necessária a seleção manual
* **Nota**: A mesma câmara não pode ser atribuída simultaneamente ao pino 1 e ao pino 2.

***

## Índice

Estas definições permitem-lhe configurar índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel de configuração de índices especiais
* **Descrição**: Abre um painel interativo onde pode selecionar e configurar índices multiespectrais de vegetação (NDVI, NDRE, EVI, etc.) para calcular durante o processamento da imagem. Pode adicionar vários índices, cada um com as suas próprias configurações de visualização.
* **Índices disponíveis**: O sistema inclui mais de 30 índices multiespectrais predefinidos, incluindo:
  * NDVI (Índice de Vegetação por Diferença Normalizada)
  * NDRE (Diferença Normalizada RedEdge)
  * EVI (Índice de Vegetação Aprimorado)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * E muito mais (consulte [Fórmulas de Índice Multiespectral](multispectral-index-formulas.md) para obter a lista completa)
* **Recursos**:
  * Selecione entre fórmulas de índice predefinidas
  * Configure gradientes de cor de visualização (LUT - Tabelas de Consulta)
  * Defina valores limite para análise
  * Crie fórmulas de índice personalizadas

### Fórmulas personalizadas (Recurso Chloros+)

* **Tipo**: Matriz de definições de fórmulas personalizadas
* **Descrição**: Permite criar e salvar fórmulas de índice multiespectral personalizadas usando matemática de banda. As fórmulas personalizadas são salvas com as configurações do seu projeto e podem ser usadas da mesma forma que os índices integrados.
* **Como criar**:
  1. No painel de configuração do Índice, procure a opção de fórmula personalizada
  2. Defina a sua fórmula usando identificadores de banda (por exemplo, NIR, Red, Green, Blue)
  3. Guarde a fórmula com um nome descritivo
* **Sintaxe da fórmula**: São suportadas operações matemáticas padrão, incluindo:
  * Aritmética: `+`, `-`, `*`, `/`
  * Parênteses para a ordem das operações
  * Referências de banda: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportar

Estas definições controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrado

* **Tipo**: Seleção suspensa
* **Opções**:
  * **TIFF (16 bits)** - Formato TIFF de 16 bits não comprimido
  * **TIFF (32 bits, percentagem)** - TIFF de 32 bits com valores de refletância em percentagem
  * **PNG (8 bits)** - Formato PNG de 8 bits comprimido
  * **JPG (8 bits)** - Formato JPEG de 8 bits comprimido
* **Padrão**: TIFF (16 bits)
* **Descrição**: Seleciona o formato de ficheiro para guardar imagens processadas e calibradas.
* **Recomendações de formato**:
  * **TIFF (16 bits)**: Recomendado para análises científicas e fluxos de trabalho profissionais. Preserva a máxima qualidade dos dados sem artefactos de compressão. Ideal para análises multiespectrais e processamento adicional em software GIS.
  * **TIFF (32 bits, percentagem)**: Ideal para fluxos de trabalho que exigem valores de refletância em percentagem (0-100%). Oferece precisão máxima para medições radiométricas.
  * **PNG (8 bits)**: Ideal para visualização na web e visualização geral. Tamanhos de ficheiro menores com compressão sem perdas, mas com gama dinâmica reduzida.
  * **JPG (8 bits)**: Tamanhos de ficheiro menores, ideal apenas para pré-visualizações e exibição na web. Utiliza compressão com perdas, o que não é adequado para análises científicas.

***

## Guardar modelo de projeto

Esta funcionalidade permite-lhe guardar as configurações atuais do seu projeto como um modelo reutilizável.

* **Tipo**: Entrada de texto + botão Guardar
* **Descrição**: Introduza um nome descritivo para o seu modelo de definições e clique no ícone Guardar. O modelo armazenará todas as definições atuais do seu projeto (detecção de alvos, opções de processamento, índices e formato de exportação) para fácil reutilização em projetos futuros.
* **Casos de uso**:
  * Crie modelos para diferentes sistemas de câmaras (RGB, multiespectral, NIR)
  * Guarde configurações padrão para tipos específicos de culturas ou fluxos de trabalho de análise
  * Partilhe configurações consistentes entre uma equipa
* **Como usar**:
  1. Configure todas as configurações desejadas do projeto
  2. Introduza um nome para o modelo (por exemplo, «RedEdge Survey3 NDVI Padrão»)
  3. Clique no ícone de guardar
  4. O modelo pode agora ser carregado ao criar novos projetos

***

## Pasta Guardar Projeto

Esta configuração especifica onde os novos projetos são guardados por predefinição.

* **Tipo**: Exibição do caminho do diretório + botão Editar
* **Predefinição**: `C:\Users\[Username]\Chloros Projects`
* **Descrição**: Mostra o diretório predefinido atual onde os novos projetos Chloros são criados. Clique no ícone Editar para selecionar um diretório diferente.
* **Quando alterar**:
  * Defina para uma unidade de rede para colaboração em equipa
  * Altere para uma unidade com mais espaço de armazenamento para grandes conjuntos de dados
  * Organize projetos por ano, cliente ou tipo de projeto em pastas diferentes
* **Observação**: alterar esta configuração afeta apenas projetos NOVOS. Os projetos existentes permanecem nos seus locais originais.

***

## Persistência das configurações

Todas as configurações do projeto são salvas automaticamente com o seu ficheiro de projeto (formato de projeto `.mapir`). Quando reabre um projeto, todas as configurações são restauradas exatamente como as deixou.

### Hierarquia das configurações

As configurações são aplicadas na seguinte ordem:

1. **Padrões do sistema** - Padrões integrados definidos pelo Chloros
2. **Definições do modelo** - Se carregar um modelo ao criar um projeto
3. **Definições do projeto guardadas** - Definições guardadas com o ficheiro do projeto
4. **Ajustes manuais** - Quaisquer alterações que fizer durante a sessão atual

### Definições e processamento de imagens

A maioria das alterações nas definições (especialmente nas categorias Processamento e Exportação) irá desencadear o reprocessamento das imagens para refletir as novas definições. No entanto, algumas definições são «apenas para exportação» e não requerem reprocessamento imediato:

* Guardar modelo do projeto
* Diretório de trabalho
* Formato de imagem calibrado (aplica-se ao exportar)

***

## Melhores práticas

1. **Comece com as predefinições**: as definições predefinidas funcionam bem para a maioria dos sistemas de câmaras MAPIR e fluxos de trabalho típicos.
2. **Crie modelos**: depois de otimizar as definições para um fluxo de trabalho ou câmara específicos, guarde-as como um modelo para garantir a consistência entre projetos.
3. **Teste antes do processamento completo**: ao experimentar novas configurações, teste em um pequeno subconjunto de imagens antes de processar todo o conjunto de dados.
4. **Documente as suas configurações**: use nomes de modelos descritivos que indiquem o sistema de câmera, o tipo de processamento e o uso pretendido (por exemplo, &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Seleção do formato de exportação**: escolha o formato de exportação com base na sua utilização final:
   * Análise científica → TIFF (16 bits ou 32 bits)
   * Processamento GIS → TIFF (16 bits)
   * Visualização rápida → PNG (8 bits)
   * Partilha na Web → JPG (8 bits)

***

Para obter mais informações sobre índices multiespectrais no Chloros, consulte a página [Fórmulas de índices multiespectrais](multispectral-index-formulas.md).
