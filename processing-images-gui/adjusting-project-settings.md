# Ajustando as configurações do projeto

Before processing your images, it's important to configure your project settings to match your workflow requirements. The Project Settings <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel provides comprehensive control over calibration, processing options, multispectral indices, and export formats.

## Acessando as configurações do projeto

1. Abra seu projeto no Chloros
2. Clique no ícone **Configurações do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Configurações do projeto exibe todas as opções de configuração

{% dica estilo = "info" %}
**As configurações são salvas automaticamente** com seu projeto. Ao reabrir um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Configurações padrão (recomendadas para a maioria dos usuários)

Para fluxos de trabalho típicos de câmeras MAPIR Survey3, as configurações padrão funcionam bem:

* ✅ **Correção de vinheta**: Ativado
* ✅ **Calibração de refletância**: Ativado (requer imagens de alvos MAPIR)
* ✅ **Método Debayer**: Alta qualidade (mais rápido)
* ✅ **Formato de exportação**: TIFF (16 bits)

Basta importar suas imagens e começar a processar com esses padrões.

***

## Visão geral das configurações do projeto

The Project Settings panel is organized into several categories. Below is a summary of each section. For complete documentation, see [Project Settings](../project-settings/page-2.md).

### Detecção de alvo

Controla como o Chloros identifica alvos de calibração em suas imagens.

**Configurações principais:**

* **Área mínima de amostra de calibração**: Limite de tamanho para detecção de alvo (padrão: 25 pixels)
* **Clustering de destino mínimo**: limite de similaridade para agrupar regiões de destino (padrão: 60)

**Quando ajustar:**

* Aumente a área de amostra se obtiver detecções falsas
* Diminua se os alvos não estiverem sendo detectados
* Ajuste o clustering se os alvos estiverem sendo divididos em várias detecções

### Processamento

Principais opções de processamento e calibração de imagens.

**Configurações principais:**

* **Correção de vinheta**: Compensa o escurecimento da lente nas bordas ✅ Recomendado
* **Calibração de refletância**: normaliza valores usando alvos de calibração ✅ Recomendado
* **Método Debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo entre o uso dos alvos de calibração (0 = usar todos)

**Configurações avançadas:**

* **Deslocamento de fuso horário do sensor de luz**: Para sincronização de horário PPK (padrão: 0)
* **Aplicar correções PPK**: usa dados de GPS/pin de exposição de arquivos .daq
* **Pino de exposição 1/2**: atribui câmeras a pinos de exposição para configurações de câmera dupla

### Índice (índices multiespectrais)

Configure quais índices de vegetação serão calculados e exportados.

**Como adicionar índices:**

1. Clique no botão **"Adicionar índice"**
2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Definir configurações de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices conforme necessário

**Índices populares:**

* **NDVI**: Saúde geral da vegetação (mais comum)
* **NDRE**: detecção precoce de estresse com RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: regiões com alto índice de área foliar (IAF)

**Fórmulas personalizadas (somente Cloros+):**

* Crie fórmulas de índice multiespectral personalizadas
* Use matemática de banda com todos os canais de imagem
* Salve fórmulas personalizadas para reutilização

For all available indices and formulas, see [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla o formato e a qualidade do arquivo de saída.

**Formatos disponíveis:**

* **TIFF (16 bits)**: recomendado para GIS e análises científicas (faixa de 0 a 65.535)
* **TIFF (32 bits, porcentagem)**: valores de refletância de ponto flutuante (intervalo de 0,0 a 1,0)
* **PNG (8 bits)**: compactação sem perdas para visualização (faixa de 0 a 255)
* **JPG (8 bits)**: arquivos menores, compactação com perdas (intervalo de 0 a 255)

***

## Salvando e carregando configurações

### Salvar modelo de projeto

Crie modelos reutilizáveis ​​para fluxos de trabalho consistentes:

1. Defina todas as configurações desejadas no painel Configurações do projeto
2. Role até a seção **"Salvar modelo de projeto"** na parte inferior
3. Insira um nome de modelo descritivo (por exemplo, "Survey3N\_RGN\_Agriculture")
4. Clique no ícone salvar

**Benefícios:**

* Aplique configurações idênticas em vários projetos
* Compartilhe configurações com membros da equipe
* Mantenha a consistência para pesquisas repetidas

### Carregar modelo no novo projeto

Ao criar um novo projeto:

1. Selecione **"Novo Projeto"** no menu principal
2. Escolha a opção **"Carregar do modelo"**
3. Selecione seu modelo salvo
4. Todas as configurações são aplicadas automaticamente

### Diretório de trabalho

A configuração **"Salvar pasta do projeto"** especifica onde novos projetos são criados por padrão:

* **Default location**: `C:\Users\[Username]\Chloros Projects`
* **Alterar local**: Clique no ícone de edição e selecione uma nova pasta
* **Quando mudar**:
  * Unidade de rede para colaboração em equipe
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração PPK (Cinemática Pós-Processada)

Se estiver usando gravadores MAPIR DAQ com GPS para geolocalização precisa:

### Pré-requisitos

* MAPIR DAQ com módulo GPS (GNSS)
* Arquivo de log .daq com entradas de pinos de exposição
* Câmera conectada aos pinos de exposição DAQ durante a sessão de captura

### Etapas de configuração

1. Coloque o arquivo de log .daq na pasta do seu projeto
2. Nas configurações do projeto, marque a caixa de seleção **"Aplicar correções PPK"**
3. Defina **"Deslocamento de fuso horário do sensor de luz"** se necessário (padrão: 0 para UTC)
4. Atribua câmeras aos pinos de exposição:
   * **Câmera única**: atribuída automaticamente ao pino 1
   * **Câmeras duplas**: atribua manualmente cada câmera ao pino correto

**Atribuição de pino de exposição:**

* **Pino de exposição 1**: selecione o modelo da câmera no menu suspenso
* **Pino de exposição 2**: Selecione a segunda câmera ou "Não usar"
* A mesma câmera não pode ser atribuída a ambos os pinos

{% dica estilo = "aviso" %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às suas respectivas câmeras. A atribuição incorreta resultará em dados de geolocalização incorretos.
{% endhint %}

***

## Cenários Avançados

### Projetos multicâmera

Ao processar imagens de múltiplas câmeras MAPIR em um projeto:

1. O Chloros detecta automaticamente cada modelo de câmera
2. Cada câmera obtém perfil de processamento apropriado
3. PPK: Atribua manualmente cada câmera ao pino de exposição correto
4. Todas as câmeras usam o mesmo formato e índices de exportação

**Exemplo**: equipamento de câmera dupla Survey3W RGN + Survey3N OCN

### Pesquisas de lapso de tempo ou de múltiplas datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com suas configurações padrão
2. Use uma configuração de alvo de calibração consistente em cada sessão
3. Processe cada data como um projeto separado
4. Use configurações idênticas para resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Grandes conjuntos de dados

Para projetos com muitas imagens (500+):

* Considere dividir em projetos menores por data ou área
* Use o processamento paralelo Chloros+ para resultados mais rápidos
* Considere CLI ou API para automação em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de detecção do alvo

***

## Verificando suas configurações

Antes de iniciar o processamento, revise estas configurações principais:

* [] Modelo de câmera detectado corretamente no navegador de arquivos
* [] Correção de vinheta ativada
* [] Calibração de refletância habilitada
* [] Pelo menos uma imagem de alvo de calibração importada
* [] Índices multiespectrais desejados adicionados
* [] Formato de exportação apropriado para o seu fluxo de trabalho
* [] Configurações de PPK configuradas (se estiver usando .daq com eventos de exposição)

***

## Próximas etapas

Depois que suas configurações estiverem definidas:

1. **Marcar imagens alvo de calibração** - Consulte [Escolher imagens alvo](choosing-target-images.md)
2. **Iniciar processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)

For complete details on all available settings, see the [Project Settings](../project-settings/page-2.md) reference documentation.
