# Ajustar as definições do projeto

Antes de processar as suas imagens, é importante configurar as definições do projeto para corresponder aos requisitos do seu fluxo de trabalho. O painel Definições do projeto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferece controle abrangente sobre calibração, opções de processamento, índices multiespectrais e formatos de exportação.

## Acessando as configurações do projeto

1. Abra o seu projeto no Chloros
2. Clique no ícone **Configurações do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Configurações do projeto exibe todas as opções de configuração

{% hint style=&quot;info&quot; %}
**As configurações são guardadas automaticamente** com o seu projeto. Quando reabre um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Configurações padrão (recomendadas para a maioria dos utilizadores)

Para fluxos de trabalho típicos da câmara MAPIR Survey3, as definições padrão funcionam bem:

* ✅ **Correção de vinheta**: Ativada
* ✅ **Calibração de refletância**: Ativada (requer imagens de alvos MAPIR)
* ✅ **Método Debayer**: Alta qualidade (mais rápido)
* ✅ **Formato de exportação**: TIFF (16 bits)

Basta importar as suas imagens e iniciar o processamento com estas predefinições.

***

## Visão geral das definições do projeto

O painel Configurações do projeto está organizado em várias categorias. Abaixo está um resumo de cada secção. Para obter a documentação completa, consulte [Configurações do projeto](../project-settings/project-settings.md).

### Detecção de alvos

Controla como o Chloros identifica alvos de calibração nas suas imagens.

**Configurações principais:**

* **Área mínima da amostra de calibração**: limite de tamanho para deteção de alvos (padrão: 25 pixels)
* **Agrupamento mínimo de alvos**: limite de similaridade para agrupar regiões-alvo (padrão: 60)

**Quando ajustar:**

* Aumente a área da amostra se estiver a obter deteções falsas
* Diminua se os alvos não estiverem a ser detetados
* Ajuste o agrupamento se os alvos estiverem a ser divididos em várias deteções

### Processamento

Principais opções de processamento e calibração de imagens.

**Configurações principais:**

* **Correção de vinheta**: Compensa o escurecimento das lentes nas bordas ✅ Recomendado
* **Calibração de refletância**: Normaliza os valores usando alvos de calibração ✅ Recomendado
* **Método Debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo entre o uso de alvos de calibração (0 = usar todos)

**Configurações avançadas:**

* **Desvio de fuso horário do sensor de luz**: Para sincronização de tempo PPK (padrão: 0)
* **Aplicar correções PPK**: Usa dados de GPS/pino de exposição de ficheiros .daq
* **Pino de exposição 1/2**: Atribui câmaras a pinos de exposição para configurações de câmara dupla

### Índice (Índices multiespectrais)

Configure quais índices de vegetação calcular e exportar.

**Como adicionar índices:**

1. Clique no botão **&quot;Adicionar índice&quot;**
2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as definições de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices, conforme necessário

**Índices populares:**

* **NDVI**: Saúde geral da vegetação (mais comum)
* **NDRE**: Detecção precoce de stress com RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com alto índice de área foliar (LAI)

**Fórmulas personalizadas (apenas Chloros+):**

* Crie fórmulas de índice multiespectral personalizadas
* Use matemática de banda com todos os canais de imagem
* Salve fórmulas personalizadas para reutilização

Para todos os índices e fórmulas disponíveis, consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla o formato e a qualidade do ficheiro de saída.

**Formatos disponíveis:**

* **TIFF (16 bits)**: Recomendado para GIS e análise científica (intervalo de 0 a 65.535)
* **TIFF (32 bits, porcentagem)**: Valores de refletância de ponto flutuante (intervalo de 0,0 a 1,0)
* **PNG (8 bits)**: Compressão sem perdas para visualização (intervalo de 0 a 255)
* **JPG (8 bits)**: Ficheiros menores, compressão com perdas (intervalo de 0 a 255)

***

## Guardar e carregar definições

### Guardar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as definições desejadas no painel Definições do projeto
2. Role até a secção **&quot;Guardar modelo do projeto&quot;** na parte inferior
3. Insira um nome descritivo para o modelo (por exemplo, &quot;Survey3N\_RGN\_Agricultura&quot;)
4. Clique no ícone Guardar

**Benefícios:**

* Aplique configurações idênticas em vários projetos
* Partilhe configurações com membros da equipa
* Mantenha a consistência para pesquisas repetidas

### Carregar modelo em novo projeto

Ao criar um novo projeto:

1. Selecione **«Novo projeto»** no menu principal
2. Escolha a opção **«Carregar a partir do modelo»**
3. Selecione o modelo guardado
4. Todas as configurações são aplicadas automaticamente

### Diretório de trabalho

A configuração **&quot;Guardar pasta do projeto&quot;** especifica onde os novos projetos são criados por predefinição:

* **Localização predefinida**: `C:\Users\[Username]\Chloros Projects`
* **Alterar localização**: Clique no ícone de edição e selecione uma nova pasta
* **Quando alterar**:
  * Unidade de rede para colaboração em equipa
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração PPK (cinemática pós-processada)

Se estiver a utilizar gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Ficheiro de registo .daq com entradas de pinos de exposição
* Câmara conectada aos pinos de exposição DAQ durante a sessão de captura

### Etapas de configuração

1. Coloque o ficheiro de registo .daq na pasta do seu projeto
2. Em Configurações do projeto, marque a caixa de seleção **&quot;Aplicar correções PPK&quot;**
3. Defina **&quot;Desvio de fuso horário do sensor de luz&quot;**, se necessário (padrão: 0 para UTC)
4. Atribua câmaras aos pinos de exposição:
   * **Câmara única**: Atribuída automaticamente ao pino 1
   * **Duas câmaras**: atribua manualmente cada câmara ao pino correto

**Atribuição de pinos de exposição:**

* **Pino de exposição 1**: selecione o modelo da câmara no menu suspenso
* **Pino de exposição 2**: selecione a segunda câmara ou &quot;Não usar&quot;
* A mesma câmara não pode ser atribuída a ambos os pinos

{% hint style=&quot;warning&quot; %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às respetivas câmaras. A atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmaras

Ao processar imagens de várias câmaras MAPIR num único projeto:

1. Chloros deteta automaticamente cada modelo de câmara
2. Cada câmara recebe o perfil de processamento adequado
3. PPK: atribua manualmente cada câmara ao pino de exposição correto
4. Todas as câmaras utilizam o mesmo formato de exportação e índices

**Exemplo**: Survey3W RGN + Survey3N OCN equipamento com duas câmaras

### Levantamentos com intervalos de tempo ou em várias datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com as suas configurações padrão
2. Use uma configuração de alvo de calibração consistente em cada sessão
3. Processe cada data como um projeto separado
4. Use configurações idênticas para obter resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Conjuntos de dados grandes

Para projetos com muitas imagens (mais de 500):

* Considere dividir em projetos menores por data ou área
* Use o processamento paralelo Chloros+ para resultados mais rápidos
* Considere CLI ou API para automação em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de deteção do alvo

***

## Verificando as suas configurações

Antes de iniciar o processamento, verifique estas configurações importantes:

* [ ] Modelo da câmara detetado corretamente no Navegador de ficheiros
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Pelo menos uma imagem de alvo de calibração importada
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação adequado para o seu fluxo de trabalho
* [ ] Configurações PPK configuradas (se estiver a usar .daq com eventos de exposição)

***

## Próximos passos

Depois de configurar as suas definições:

1. **Marque as imagens de alvo de calibração** - Consulte [Escolher imagens de alvo](choosing-target-images.md)
2. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)
3. **Monitorize o progresso** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as configurações disponíveis, consulte a documentação de referência [Configurações do projeto](../project-settings/project-settings.md).
