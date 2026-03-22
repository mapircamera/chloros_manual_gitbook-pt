# Definições do projeto

A barra lateral «Definições do projeto» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> no Chloros permite-lhe configurar todos os aspetos do processamento de imagens, deteção de alvos de calibração, cálculos de índices multiespectrais e opções de exportação para o seu projeto. Estas definições são guardadas com o seu projeto e podem ser guardadas como modelos para reutilização em vários projetos.

## Aceder às Definições do Projeto

Para aceder às Definições do Projeto:

1. Abra um projeto no Chloros
2. Clique no separador **Definições do Projeto**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel de definições apresentará todas as opções de configuração disponíveis organizadas por categoria

***

## Detecção de alvos

Estas definições controlam a forma como o Chloros deteta e processa alvos de calibração nas suas imagens.

### Área mínima da amostra de calibração (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10 000 píxeis
* **Padrão**: 25 píxeis
* **Descrição**: Define a área mínima (em píxeis) necessária para que uma região detetada seja considerada uma amostra válida de alvo de calibração. Valores mais baixos detetarão alvos mais pequenos, mas podem aumentar os falsos positivos. Valores mais altos requerem regiões-alvo maiores e mais nítidas para a deteção.
* **Quando ajustar**:
  * Aumente se estiver a obter detecções falsas em pequenos artefactos da imagem
  * Diminua se os seus alvos de calibração parecerem pequenos nas suas imagens e não estiverem a ser detetados

### Agrupamento mínimo de alvos (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Padrão**: 60
* **Descrição**: Controla o limiar de agrupamento para agrupar regiões de cores semelhantes ao detetar alvos de calibração. Valores mais altos exigem que mais cores semelhantes sejam agrupadas, resultando numa deteção de alvos mais conservadora. Valores mais baixos permitem mais variação de cor dentro de um grupo de alvos.
* **Quando ajustar**:
  * Aumente se os alvos de calibração estiverem a ser divididos em várias deteções
  * Diminua se os alvos de calibração com variação de cor não estiverem a ser totalmente detetados

***

## Processamento

Estas definições controlam a forma como o Chloros processa e calibra as suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Aplica correção de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é um fenómeno ótico comum em que os cantos e as bordas de uma imagem parecem mais escuros do que o centro devido às características da lente.
* **Quando desativar**: Desative apenas se a sua combinação de câmara/lente já tiver aplicado correção de vinheta, ou se pretender corrigir manualmente a vinheta no pós-processamento.

### Calibração de refletância / equilíbrio de brancos

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Ativa a calibração automática de refletância utilizando alvos de calibração detetados nas suas imagens. Isto normaliza os valores de refletância em todo o seu conjunto de dados e garante medições consistentes, independentemente das condições de iluminação.
* **Quando desativar**: Desative apenas se pretender processar imagens raw não calibradas ou se estiver a utilizar um fluxo de trabalho de calibração diferente.

### Método de demosaicing

* **Tipo**: Menu suspenso
* **Opções**:
  * Padrão (Rápido, Qualidade Média)
  * Sensível à textura (Lento, Qualidade Máxima) \[Chloros+]
* **Padrão**: Padrão (Rápido, Qualidade Média)
* **Descrição**: Seleciona o algoritmo de demosaicing utilizado para converter dados brutos do sensor com padrão Bayer em imagens a cores. O método &quot;Padrão (Rápido, Qualidade Média)&quot; proporciona um equilíbrio ideal entre velocidade de processamento e qualidade de imagem. O &quot;Sensível à textura (Lento, Qualidade máxima)&quot; \[Chloros+] utiliza um algoritmo de demosaicing sensível às arestas de alta qualidade, combinado com um modelo de redução de ruído baseado em IA/ML que elimina quase todo o ruído do demosaicing. O modelo Sensível à textura requer memória da GPU (VRAM) para funcionar. Recomendamos a sua utilização quando tiver &gt;4 GB de VRAM disponível para um processamento mais rápido.
* **Nota**: Podem ser adicionados métodos de debayer adicionais em versões futuras do Chloros.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3.600 segundos
* **Padrão**: 0 segundos
* **Descrição**: Define o intervalo de tempo mínimo (em segundos) entre a utilização de alvos de calibração. Quando definido como 0, o Chloros utilizará todos os alvos de calibração detetados. Quando definido com um valor superior, o Chloros utilizará apenas alvos de calibração separados por, pelo menos, este número de segundos, reduzindo o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de calibração.
* **Quando ajustar**:
  * Defina como 0 para obter a máxima precisão de calibração quando as condições de iluminação variam
  * Aumente (por exemplo, para 60-300 segundos) para um processamento mais rápido quando a iluminação for consistente e houver imagens frequentes de alvos de calibração

### Desvio de fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: Especifica o desvio de fuso horário (em horas em relação ao UTC) para os carimbos de data/hora dos dados do sensor de luz. Isto é utilizado ao processar ficheiros de dados PPK (Post-Processed Kinematic) para garantir a sincronização temporal correta entre as capturas de imagem e os dados de GPS.
* **Quando ajustar**: Defina isto para o desvio do seu fuso horário local se os seus dados PPK utilizarem a hora local em vez do UTC. Por exemplo:
  * Hora do Pacífico: -8 ou -7 (dependendo do horário de verão)
  * Hora do Leste: -5 ou -4 (dependendo do horário de verão)
  * Hora da Europa Central: +1 ou +2 (dependendo do horário de verão)

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Padrão**: Desativado (desmarcada)
* **Descrição**: Ativa a utilização de correções cinemáticas pós-processadas (PPK) a partir de gravadores DAQ MAPIR que contenham um GPS (GNSS). Quando ativado, o Chloros utilizará quaisquer ficheiros de registo .daq que contenham dados de pinos de exposição no diretório do seu projeto e aplicará correções de geolocalização precisas às suas imagens.
* **Requisito**: Deve existir um ficheiro de registo .daq com entradas de pinos de exposição no diretório do seu projeto
* **Quando ativar**: Recomenda-se ativar sempre a correção PPK se tiver entradas de feedback de exposição no seu ficheiro de registo .daq.

### Pino de exposição 1

* **Tipo**: Seleção por menu suspenso
* **Visibilidade**: Apenas visível quando &quot;Aplicar correções PPK&quot; está ativado E os dados de exposição estão disponíveis para o Pino 1
* **Opções**:
  * Nomes de modelos de câmara detetados no projeto
  * &quot;Não utilizar&quot; - Ignorar este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao Pino de Exposição 1 para sincronização temporal PPK. O pino de exposição regista o momento exato em que o obturador da câmara é acionado, o que é fundamental para uma geolocalização PPK precisa.
* **Comportamento de seleção automática**:
  * Câmara única + pino único: Seleciona automaticamente a câmara
  * Câmara única + dois pinos: O Pino 1 é automaticamente atribuído à câmara
  * Várias câmaras: É necessária a seleção manual

### Pino de exposição 2

* **Tipo**: Seleção por menu suspenso
* **Visibilidade**: Apenas visível quando «Aplicar correções PPK» está ativado E os dados de exposição estão disponíveis para o Pino 2
* **Opções**:
  * Nomes de modelos de câmara detetados no projeto
  * «Não utilizar» - Ignorar este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmara específica ao Pino de exposição 2 para sincronização temporal PPK ao utilizar uma configuração de duas câmaras.
* **Comportamento da seleção automática**:
  * Câmara única + pino único: O Pino 2 é automaticamente definido como &quot;Não utilizar&quot;
  * Câmara única + dois pinos: O pino 2 é automaticamente definido como «Não utilizar»
  * Várias câmaras: É necessária a seleção manual
* **Nota**: A mesma câmara não pode ser atribuída simultaneamente ao pino 1 e ao pino 2.***

## Índice

Estas definições permitem-lhe configurar índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel de configuração de índices especiais
* **Descrição**: Abre um painel interativo onde pode selecionar e configurar índices multiespectrais de vegetação (NDVI, NDRE, EVI, etc.) para calcular durante o processamento da imagem. Pode adicionar vários índices, cada um com as suas próprias definições de visualização.
* **Índices disponíveis**: O sistema inclui mais de 30 índices multiespectrais predefinidos, incluindo:
  * NDVI (Índice de Vegetação por Diferença Normalizada)
  * NDRE (Índice de Diferença Normalizada RedEdge)
  * EVI (Índice de Vegetação Aperfeiçoado)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * E muitos mais (consulte [Fórmulas de Índices Multiespectrais](multispectral-index-formulas.md) para obter a lista completa)
* **Funcionalidades**:
  * Selecione entre fórmulas de índices predefinidas
  * Configure gradientes de cor de visualização (LUT - Tabelas de Consulta)
  * Defina valores limite para análise
  * Crie fórmulas de índices personalizadas

### Fórmulas personalizadas (Funcionalidade Chloros+)

* **Tipo**: Matriz de definições de fórmulas personalizadas
* **Descrição**: Permite criar e guardar fórmulas de índices multiespectrais personalizadas utilizando matemática de bandas. As fórmulas personalizadas são guardadas com as definições do seu projeto e podem ser utilizadas tal como os índices integrados.
* **Como criar**:
  1. No painel de configuração do Índice, procure a opção de fórmula personalizada
  2. Defina a sua fórmula utilizando identificadores de banda (por exemplo, NIR, Red, Green, Blue)
  3. Guarde a fórmula com um nome descritivo
* **Sintaxe da fórmula**: São suportadas operações matemáticas padrão, incluindo:
  * Aritmética: `+`, `-`, `*`, `/`
  * Parênteses para a ordem das operações
  * Referências de banda: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportar

Estas definições controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrada

* **Tipo**: Seleção por menu suspenso
* **Opções**:
  * **TIFF (16 bits)** - Formato TIFF de 16 bits não comprimido
  * **TIFF (32 bits, Porcentagem)** - TIFF de 32 bits em ponto flutuante com valores de refletância expressos em percentagens
  * **PNG (8 bits)** - Formato PNG de 8 bits comprimido
  * **JPG (8 bits)** - Formato JPEG de 8 bits comprimido
* **Padrão**: TIFF (16 bits)
* **Descrição**: Seleciona o formato de ficheiro para guardar imagens processadas e calibradas.
* **Recomendações de formato**:
  * **TIFF (16 bits)**: Recomendado para análise científica e fluxos de trabalho profissionais. Preserva a máxima qualidade dos dados sem artefactos de compressão. Ideal para análise multiespectral e processamento posterior em software SIG.
  * **TIFF (32 bits, Porcentagem)**: Ideal para fluxos de trabalho que requerem valores de refletância expressos em percentagens (0-100%). Oferece a máxima precisão para medições radiométricas.
  * **PNG (8 bits)**: Adequado para visualização na Web e visualização geral. Tamanhos de ficheiro mais pequenos com compressão sem perdas, mas com gama dinâmica reduzida.
  * **JPG (8 bits)**: Tamanhos de ficheiro mais pequenos, ideal apenas para pré-visualizações e exibição na Web. Utiliza compressão com perdas, o que não é adequado para análises científicas.***

## Guardar modelo de projeto

Esta funcionalidade permite-lhe guardar as definições atuais do seu projeto como um modelo reutilizável.

* **Tipo**: Entrada de texto + botão Guardar
* **Descrição**: Introduza um nome descritivo para o seu modelo de definições e clique no ícone Guardar. O modelo armazenará todas as definições atuais do seu projeto (detecção de alvos, opções de processamento, índices e formato de exportação) para fácil reutilização em projetos futuros.
* **Casos de utilização**:
  * Criar modelos para diferentes sistemas de câmaras (RGB, multiespectral, NIR)
  * Guardar configurações padrão para tipos específicos de culturas ou fluxos de trabalho de análise
  * Partilhar definições consistentes entre uma equipa
* **Como utilizar**:
  1. Configure todas as definições de projeto desejadas
  2. Introduza um nome para o modelo (por exemplo, «RedEdge Survey3 NDVI Padrão»)
  3. Clique no ícone de guardar
  4. O modelo pode agora ser carregado ao criar novos projetos

***

## Pasta de projeto de guarda

Esta configuração especifica onde os novos projetos são guardados por predefinição.

* **Tipo**: Exibição do caminho do diretório + botão Editar
* **Predefinição (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Padrão (Linux)**: `~/.local/share/chloros/projects`
* **Descrição**: Mostra o diretório padrão atual onde novos projetos Chloros são criados. Clique no ícone de edição para selecionar um diretório diferente.
* **Quando alterar**:
  * Defina para uma unidade de rede para colaboração em equipa
  * Altere para uma unidade com mais espaço de armazenamento para conjuntos de dados de grande dimensão
  * Organize projetos por ano, cliente ou tipo de projeto em pastas diferentes
* **Nota**: A alteração desta definição afeta apenas os projetos NOVOS. Os projetos existentes permanecem nas suas localizações originais.***

## Persistência das configurações

Todas as configurações do projeto são guardadas automaticamente com o seu ficheiro de projeto (formato de projeto `.mapir`). Quando reabre um projeto, todas as configurações são restauradas exatamente como as deixou.

### Hierarquia das configurações

As configurações são aplicadas na seguinte ordem:

1. **Predefinições do sistema** - Predefinições integradas definidas pelo Chloros
2. **Definições do modelo** - Se carregar um modelo ao criar um projeto
3. **Definições do projeto guardadas** - Definições guardadas com o ficheiro do projeto
4. **Ajustes manuais** - Quaisquer alterações que efetue durante a sessão atual

### Configurações e processamento de imagens

A maioria das alterações nas configurações (especialmente nas categorias Processamento e Exportação) irá desencadear o reprocessamento das imagens para refletir as novas configurações. No entanto, algumas configurações são «apenas para exportação» e não requerem reprocessamento imediato:

* Guardar modelo de projeto
* Diretório de trabalho
* Formato de imagem calibrada (aplica-se ao exportar)

***

## Melhores práticas

1. **Comece com as predefinições**: As predefinições funcionam bem para a maioria dos sistemas de câmaras MAPIR e fluxos de trabalho típicos.
2. **Crie modelos**: Depois de otimizar as definições para um fluxo de trabalho ou câmara específicos, guarde-as como um modelo para garantir a consistência entre projetos.
3. **Teste antes do processamento completo**: Ao experimentar novas configurações, teste numa pequena subamostra de imagens antes de processar todo o seu conjunto de dados.
4. **Documente as suas configurações**: Utilize nomes de modelos descritivos que indiquem o sistema de câmara, o tipo de processamento e a utilização pretendida (por exemplo, «Survey3\_RGB\_NDVI\_Agricultura»).
5. **Seleção do formato de exportação**: Escolha o seu formato de exportação com base na sua utilização final:
   * Análise científica → TIFF (16 bits ou 32 bits)
   * Processamento SIG → TIFF (16 bits)
   * Visualização rápida → PNG (8 bits)
   * Partilha na Web → JPG (8 bits)

***

Para mais informações sobre índices multiespectrais no Chloros, consulte a página [Fórmulas de Índices Multiespectrais](multispectral-index-formulas.md).
