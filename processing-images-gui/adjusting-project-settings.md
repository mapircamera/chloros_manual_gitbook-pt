# Ajustar as definições do projeto

Antes de processar as suas imagens, é importante configurar as definições do projeto de modo a corresponder aos requisitos do seu fluxo de trabalho. O painel «Definições do projeto» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferecem um controlo abrangente sobre a calibração, as opções de processamento, os índices multiespectrais e os formatos de exportação.

## Aceder às definições do projeto

1. Abra o seu projeto no Chloros
2. Clique no ícone **Definições do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Configurações do projeto exibe todas as opções de configuração

{% hint style="info" %}
**As configurações são guardadas automaticamente** com o seu projeto. Quando reabre um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Definições Padrão (Recomendadas para a Maioria dos Utilizadores)

Para fluxos de trabalho típicos com câmaras MAPIR e Survey3, as definições padrão funcionam bem:

* ✅ **Correção de vinheta**: Ativada
* ✅ **Calibração de refletância**: Ativada (requer imagens de alvos MAPIR)
* ✅ **Método de debayer**: Padrão (Rápido, Qualidade Média)
* ✅ **Formato de exportação**: TIFF (16 bits)

Basta importar as suas imagens e iniciar o processamento com estas predefinições.

***

## Visão geral das definições do projeto

O painel de definições do projeto está organizado em várias categorias. Segue-se um resumo de cada secção. Para obter a documentação completa, consulte [Definições do projeto](../project-settings/project-settings.md).

### Detecção de alvos

Controla a forma como o Chloros identifica os alvos de calibração nas suas imagens.

**Principais definições:*** **Área mínima da amostra de calibração**: Limite de tamanho para deteção de alvos (padrão: 25 píxeis)
* **Agrupamento mínimo de alvos**: Limite de similaridade para agrupar regiões-alvo (padrão: 60)**Quando ajustar:**

* Aumente a área da amostra se obtiver deteções falsas
* Diminua se os alvos não estiverem a ser detetados
* Ajuste o agrupamento se os alvos estiverem a ser divididos em várias deteções

### Processamento

Principais opções de processamento de imagem e calibração.

**Configurações principais:*** **Correção de vinheta**: Compensa o escurecimento da lente nas bordas ✅ Recomendado
* **Calibração de refletância**: Normaliza valores utilizando alvos de calibração ✅ Recomendado
* **Método de debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo entre a utilização de alvos de calibração (0 = utilizar todos)**Definições avançadas:*** **Desvio de fuso horário do sensor de luz**: Para sincronização temporal PPK (padrão: 0)
* **Aplicar correções PPK**: Utiliza dados de GPS/pinos de exposição de ficheiros .daq
* **Pino de exposição 1/2**: Atribui câmaras aos pinos de exposição para configurações de câmara dupla

### Método de debayering

Atualmente, oferecemos 2 métodos de debayering no Chloros:

#### Padrão (Rápido, Qualidade Média)

O debayer padrão processa rapidamente, mas apresenta ruído de cor no debayering, resultando em imagens menos precisas e com mais ruído.

#### Sensível à textura (Lento, Qualidade máxima) \[Apenas Chloros+]

O Sensível à textura utiliza um debayer de alta qualidade sensível às arestas, combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do debayering. O modelo Sensível à Textura requer memória da GPU (VRAM) para funcionar. Recomendamos a sua utilização quando tiver &gt;4 GB de VRAM disponível para um processamento mais rápido.

### Índice (Índices Multiespectrais)

Configure quais os índices de vegetação a calcular e exportar.

**Como adicionar índices:**

1. Clique no botão**&quot;Adicionar índice&quot;**

2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as definições de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices conforme necessário

**Índices populares:*** **NDVI**: Saúde geral da vegetação (mais comum)
* **NDRE**: Detecção precoce de stress com RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com elevado índice de área foliar (LAI)**Fórmulas personalizadas (apenas Chloros+):**

* Crie fórmulas de índices multiespectrais personalizadas
* Utilize cálculos matemáticos de bandas com todos os canais de imagem
* Guarde fórmulas personalizadas para reutilização

Para todos os índices e fórmulas disponíveis, consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla o formato e a qualidade do ficheiro de saída.

**Formatos disponíveis:*** **TIFF (16 bits)**: Recomendado para SIG e análise científica (intervalo de 0 a 65 535)
* **TIFF (32 bits, Porcentagem)**: Valores de refletância em ponto flutuante (intervalo de 0,0 a 1,0)
* **PNG (8 bits)**: Compressão sem perdas para visualização (intervalo de 0 a 255)
* **JPG (8 bits)**: Ficheiros mais pequenos, compressão com perdas (intervalo de 0 a 255)***

## Guardar e carregar definições

### Guardar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as definições desejadas no painel Definições do projeto
2. Desça até à secção **&quot;Guardar modelo do projeto&quot;** na parte inferior
3. Introduza um nome descritivo para o modelo (por exemplo, &quot;Survey3N\_RGN\_Agricultura&quot;)
4. Clique no ícone de guardar

**Vantagens:**

* Aplique definições idênticas em vários projetos
* Partilhe configurações com os membros da equipa
* Mantenha a consistência em inquéritos repetidos

### Carregar modelo num novo projeto

Ao criar um novo projeto:

1. Selecione **&quot;Novo projeto&quot;** no menu principal
2. Escolha a opção **&quot;Carregar a partir do modelo&quot;**

3. Selecione o seu modelo guardado
4. Todas as definições são aplicadas automaticamente

### Diretório de Trabalho

A configuração **&quot;Pasta de Salvamento do Projeto&quot;** especifica onde os novos projetos são criados por padrão:

* **Localização padrão**: `C:\Users\[Username]\Chloros Projects`
* **Alterar localização**: Clique no ícone de edição e selecione uma nova pasta
* **Quando alterar**:
  * Unidade de rede para colaboração em equipa
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração PPK (Cinemática Pós-Processada)

Se estiver a utilizar gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Ficheiro de registo .daq com entradas de pinos de exposição
* Câmara ligada aos pinos de exposição do DAQ durante a sessão de captura

### Passos de configuração

1. Coloque o ficheiro de registo .daq na pasta do seu projeto
2. Em Definições do projeto, ative a caixa de seleção **&quot;Aplicar correções PPK&quot;**

3. Defina**&quot;Desvio de fuso horário do sensor de luz&quot;**, se necessário (padrão: 0 para UTC)
4. Atribua câmaras aos pinos de exposição:
   * **Câmara única**: Atribuída automaticamente ao Pino 1
   * **Câmaras duplas**: Atribua manualmente cada câmara ao pino correto**Atribuição de pinos de exposição:*** **Pino de exposição 1**: Selecione o modelo da câmara no menu suspenso
* **Pino de exposição 2**: Selecione a segunda câmara ou &quot;Não utilizar&quot;
* A mesma câmara não pode ser atribuída a ambos os pinos

{% hint style="warning" %}
**Importante**: Os pinos de exposição devem ser corretamente atribuídos às respetivas câmaras. Uma atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmaras

Ao processar imagens de várias câmaras MAPIR num único projeto:

1. O Chloros deteta automaticamente o modelo de cada câmara
2. Cada câmara recebe o perfil de processamento adequado
3. PPK: Atribua manualmente cada câmara ao pino de exposição correto
4. Todas as câmaras utilizam o mesmo formato de exportação e índices

**Exemplo**: Survey3W RGN + Survey3N OCN equipamento de duas câmaras

### Levantamentos em time-lapse ou com várias datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com as suas definições padrão
2. Utilize uma configuração consistente do alvo de calibração em cada sessão
3. Processe cada data como um projeto separado
4. Utilize definições idênticas para obter resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Conjuntos de dados de grande dimensão

Para projetos com muitas imagens (mais de 500):

* Considere dividir em projetos mais pequenos por data ou área
* Utilize o processamento paralelo Chloros+ para resultados mais rápidos
* Considere o CLI ou o API para automatização em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de deteção do alvo

***

## Verificar as suas definições

Antes de iniciar o processamento, verifique estas definições essenciais:

* [ ] Modelo da câmara detetado corretamente no Navegador de Ficheiros
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Pelo menos uma imagem de alvo de calibração importada
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação adequado ao seu fluxo de trabalho
* [ ] Definições PPK configuradas (se estiver a utilizar .daq com eventos de exposição)

***

## Próximos passos

Depois de configurar as suas definições:

1. **Marque as imagens-alvo de calibração** - Consulte [Escolher imagens-alvo](choosing-target-images.md)
2. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)
3. **Monitorize o progresso** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as definições disponíveis, consulte a documentação de referência [Definições do projeto](../project-settings/project-settings.md).
