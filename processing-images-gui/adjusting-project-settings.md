# Ajustar as definições do projeto

Antes de processar as suas imagens, é importante configurar as definições do projeto de forma a corresponder aos requisitos do seu fluxo de trabalho. O painel «Definições do projeto» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferece um controlo abrangente sobre a calibração, as opções de processamento, os índices multiespectrais e os formatos de exportação.

## Aceder às definições do projeto

1. Abra o seu projeto no Chloros
2. Clique no ícone **Definições do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel «Definições do projeto» apresenta todas as opções de configuração

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>O painel «Definições do projeto» — Visualização, deteção de alvos e processamento</p></figcaption></figure>{% hint style="info" %}
**As definições são guardadas automaticamente** com o seu projeto. Quando reabrir um projeto, todas as definições são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Definições predefinidas (recomendadas para a maioria dos utilizadores)

As definições predefinidas funcionam bem para fluxos de trabalho típicos do Survey3 e do LATTICE:

* ✅ **Correção de vinheta**: Ativada
* ✅ **Calibração de refletância / equilíbrio de brancos**: Ativada (utiliza alvos MAPIR e/ou dados do sensor de luz DAQ)
* ✅ **Método de debayering**: Padrão (Rápido, Qualidade Média)
* ✅ **Formato de exportação**: TIFF (16 bits)
* ✅ **Todos os produtos de exportação**: Ativado (o LATTICE captura automaticamente a distribuição em imagens sem efeito de mozaico, pré-visualização, radiação e refletância)

Basta importar as suas imagens e iniciar o processamento com estas predefinições.

***

## Visão geral das definições do projeto

O painel «Definições do projeto» está organizado nas secções abaixo. Duas secções adicionais — **Sensor de luz DAQ**e**Alinhamento da matriz** — aparecem automaticamente quando o seu projeto contém os ficheiros relevantes. Para obter a documentação completa, consulte [Definições do projeto](../project-settings/project-settings.md).

### Exibição

* **Resolução das miniaturas das imagens**: Resolução das miniaturas da grelha de imagens. Opções:**Padrão (512 px)**,**1024 px**,**2048 px**,**Resolução total**. Apenas para visualização — nunca afeta o processamento. Valores mais elevados proporcionam uma imagem mais nítida quando ampliada, mas demoram mais tempo a carregar.

### Detecção de alvos

Controla a forma como o Chloros identifica os alvos de calibração nas suas imagens.

**Principais definições:*** **Área mínima da amostra de calibração (px)**: Limite de tamanho para a deteção de alvos (padrão:**25**, intervalo 0–10 000)
* **Agrupamento mínimo de alvos (0-100)**: Limiar de similaridade para agrupar regiões-alvo (padrão:**60**)**Quando ajustar:**

* Aumente a área de amostragem se ocorrerem deteções falsas
* Diminua se os alvos não estiverem a ser detetados
* Ajuste o agrupamento se os alvos estiverem a ser divididos em várias deteções

{% hint style="info" %}
Estas definições ficam desativadas quando a **calibração de refletância / equilíbrio de brancos** está desativada — com esta função desativada, a deteção de alvos nunca é executada.
{% endhint %}

### Processamento

Principais opções de processamento de imagem e calibração.

**Definições principais:*** **Correção de vinheta**: Compensa o escurecimento da lente nas bordas ✅ Recomendado
* **Calibração de refletância / equilíbrio de brancos**: Calibra as imagens utilizando alvos detetados (Survey3) e/ou dados do sensor de luz DAQ (LATTICE) ✅ Recomendado
* **Método de debayerização**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo mínimo, em segundos, entre a utilização de alvos de calibração (predefinição:**0** = utilizar todos, intervalo 0–3600)**Produtos alternativos não calibrados:**Quando um fotograma não pode ser calibrado em termos de refletância (sem alvo disponível ou calibração desativada), é exportado como um de dois produtos alternativos —**existe exatamente um do par por execução**, escolhido pela opção de correção de vinheta:

* **Exportar resposta do sensor**: grava `Sensor_Response_Images` — utilizado quando a correção de vinheta está**desativada*** **Exportar com correção de vinheta**: grava `Vignette_Corrected_Images` — utilizado quando a correção de vinheta está**ativada**A caixa de seleção que não está ativa aparece a cinzento. Desmarcar a caixa ativa impede que esse ficheiro seja gravado.**Produtos de exportação do LATTICE** (apresentados para cada projeto; aplicam-se às capturas do LATTICE):

* **Exportar sem efeito de bayering**: a imagem linear sem efeito de bayering (`Debayered_Images`). Aplica-se a RGB e aos módulos multiespectrais.
* **Exportar pré-visualização**: a pré-visualização no ecrã (`Preview_Images`). RGB = equilíbrio de brancos (iluminante DAQ, quando disponível; caso contrário, «gray-world») + gama; multiespectral = alongamento de cores falsas.
* **Exportação de radianância**: radianância espectral float32 (`Radiance_Images`, W/m²/sr/nm). Apenas módulos multiespectrais — não aplicável aos mestres RGB.
* ****Exportar refletância**: refletância uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) quando uma leitura descendente `.daq` ou um alvo no quadro abrange o quadro. Apenas módulos multiespectrais.

Todas as quatro opções estão **ativadas por predefinição**— um quadro bruto LATTICE importado é distribuído por todos os produtos ativados e aplicáveis numa única passagem de processamento. A caixa de seleção**Exportar refletância** fica desativada quando a calibração da refletância está desativada. As definições cujo interruptor principal as torna impossíveis estão sempre desativadas, com uma dica de ferramenta que indica o interruptor a alterar.**Definições avançadas:*** **Desvio do fuso horário do sensor de luz**: Horas em relação ao UTC para sincronização horária do sensor de luz (predefinição: 0, intervalo de −12 a +12)
* **Aplicar correções PPK**: Utiliza dados de GPS/pinos de exposição dos ficheiros `.daq` (predefinição: desativado)
* **Pino de exposição 1/2**: Atribui câmaras aos pinos de exposição para configurações com duas câmaras

{% hint style="info" %}
**O nível de entrada do LATTICE é automático.** As capturas do LATTICE incluem o seu nível de processamento nos metadados XMP, e o processamento entra sempre no fluxo de trabalho a partir do fotograma bruto — não há nada para configurar na GUI. (O sinalizador CLI existe como uma opção de substituição para utilizadores avançados no caso de capturas com metadados perdidos; consulte a [Referência do CLI](../reference/cli-reference.md).)
{% endhint %}

### Método de remoção de pixelização

Atualmente, disponibilizamos 2 métodos de remoção de pixelização no Chloros:

#### Padrão (Rápido, Qualidade Média)

O método padrão de remoção de pixelização processa rapidamente, mas apresenta ruído de cor resultante da remoção de pixelização, o que resulta em imagens menos precisas e com mais ruído.

#### Sensível à Textura (Lento, Qualidade Máxima) \[Apenas Chloros+]

O método Sensível à Textura utiliza um debayer de alta qualidade sensível às arestas, combinado com um modelo de remoção de ruído baseado em IA/ML que elimina praticamente todo o ruído do processo de debayering. O modelo requer memória da GPU (VRAM) para funcionar: com **7 GB ou mais de VRAM**, pode processar várias imagens em simultâneo; com menos de 7 GB, processa uma imagem de cada vez (de forma visivelmente mais lenta). Consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**As capturas do LATTICE utilizam sempre o demosaico padrão.**Não existe um modelo «Texture Aware» treinado para o LATTICE, pelo que a opção não é disponibilizada para imagens do LATTICE — as imagens**Survey3** no mesmo projeto podem, no entanto, utilizá-la.
{% endhint %}

### Índice (Índices Multiespectrais)

Configure quais os índices de vegetação a calcular e exportar. O menu suspenso da interface gráfica oferece **27 fórmulas de índices predefinidas**.**Como adicionar índices:**

1. Clique no botão**«Adicionar índice»**

2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as definições de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices, conforme necessário

**Índices populares:*** **NDVI**: Saúde geral da vegetação (o mais comum)
* **NDRE**: Detecção precoce de stress com o RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com elevado índice de área foliar (LAI)**Fórmulas personalizadas:**

* Crie fórmulas de índices multiespectrais personalizadas com cálculos entre bandas em todos os canais da imagem
* Guarde fórmulas personalizadas para reutilização
* As fórmulas personalizadas são uma funcionalidade do Chloros+; a disponibilidade depende do seu nível de subscrição

Para consultar todos os índices e fórmulas disponíveis — incluindo quais os nomes que existem apenas na interface gráfica e quais os que também funcionam no CLI/SDK — consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md).

### Exportação

Controla o formato do ficheiro de saída.

**Formatos disponíveis**(configuração:**Formato de imagem calibrada**, predefinição**TIFF (16 bits)**):

* **TIFF (16 bits)**: Recomendado para SIG e análise científica
* **TIFF (32 bits, por cento)**: Valores de ponto flutuante
* **PNG (8 bits)**: compressão sem perdas para visualização
* **JPG (8 bits)**: ficheiros mais pequenos, compressão com perdas

Os ficheiros de saída são gravados na pasta do projeto, agrupados por câmara e formato: `<project>/<camera>/<format>/<Product>_Images/`. O Radiance é **sempre** gravado como float32 na pasta `tiff32`, independentemente desta configuração. Os ficheiros exportados mantêm o nome do ficheiro de origem — a pasta identifica o produto. Consulte [Conclusão do processamento](finishing-the-processing.md) para ver a árvore completa de resultados.

{% hint style="warning" %}
**Leitura dos valores de refletância**: o DN que corresponde a ρ = 1,0 depende da câmara de origem — o LATTICE utiliza 32768 (marcado como XMP `Chloros:PixelScale`), enquanto o Survey3 utiliza 65535. Leia a etiqueta em vez de assumir um valor constante. Consulte [Formatos de imagem de saída](../output-image-formats.md).
{% endhint %}

### Sensor de luz DAQ

Esta secção lista todos os ficheiros de radiação descendente do DAQ (`.daq` / `.csv`) no seu projeto, uma linha por ficheiro, apresentando o modelo do sensor, o nome do ficheiro e a correção do **limite** do difusor em vigor para esse ficheiro.

* **Substituição do limite (todos os ficheiros)**: um único menu suspenso para todo o projeto. A opção**Auto** (predefinição) utiliza o limite registado em cada ficheiro — presume-se luz solar quando nada foi registado, uma vez que todos os DAQs MAPIR vêm equipados com o corretor de luz solar. A seleção de um limite substitui todos os ficheiros: as gravações em bruto são corrigidas com esse valor e as gravações que já incluem um limite são reajustadas (a correção registada é anulada e o limite selecionado é aplicado).
* As linhas alertam quando um limite máximo registado era o valor predefinido assumido pelo hub, em vez de ter sido confirmado pelo operador, e quando o limite máximo selecionado não tem um perfil para esse modelo de dispositivo (a substituição é recusada para esse ficheiro).

As gravações do DAQ feitas no separador «Sensores de Luz» são adicionadas automaticamente ao projeto aberto, e os ficheiros `.daq` / `.csv` importados aparecem aqui assim que são adicionados.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Definições inferiores do projeto — Índice, Formato de exportação, a secção «Sensores de luz» do DAQ e os controlos do modelo/pasta do projeto</p></figcaption></figure>### Alinhamento de matrizes

Esta secção aparece **apenas** quando pelo menos uma imagem no projeto contém a transformação de alinhamento módulo a módulo que as matrizes LATTICE aplicam no momento da captura (`Chloros:Alignment*` XMP). Mostra quantas imagens contêm essas etiquetas e qual a câmara que serve de referência, com os seguintes controlos:

* **Aplicar alinhamento da matriz** (predefinição: ativado): distorce todos os produtos processados (debayering / pré-visualização / radiação / refletância / índice) para a geometria de referência partilhada da matriz. Desativado = exportação na geometria nativa do sensor.
* **Recortar para sobreposição comum** (padrão: ativado): recorta as exportações alinhadas para a região que todos os módulos partilham, de modo que cada banda tenha a mesma área de cobertura. Desativado mantém a área total do sensor (preenchimento a preto fora da fonte).
* **Reamostragem**:**Bilinear (suave, predefinição)**,**Mais próximo (preserva valores exatos)**— sem mistura entre píxeis, para análise radiométrica rigorosa — ou**Cúbica (mais nítida)**.***

## Guardar e carregar definições

### Guardar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as definições desejadas no painel «Definições do projeto»
2. Deslize até à secção **«Guardar modelo de projeto»** na parte inferior
3. Introduza um nome descritivo para o modelo (por exemplo, «Survey3N\_RGN\_Agriculture»)
4. Clique no ícone de guardar

**Vantagens:**

* Aplicar definições idênticas em vários projetos
* Partilhe configurações com os membros da equipa
* Mantenha a consistência em inquéritos repetidos

### Carregar modelo num novo projeto

Ao criar um novo projeto:

1. Selecione **«Novo projeto»** no menu principal
2. Escolha um modelo de projeto no seletor de modelos opcional
3. Todas as definições do modelo são aplicadas automaticamente

### Diretório de Trabalho

A definição **&quot;Diretório de Trabalho&quot;** especifica onde os novos projetos são criados por predefinição:

* **Localização predefinida**: `C:\Users\[Username]\Chloros Projects`
* **Alterar localização**: Clique no ícone de edição e selecione uma nova pasta
* **Partilhado com o CLI**: o `chloros-cli` utiliza a mesma configuração predefinida da pasta de projetos
* **Quando alterar**:
  * Unidade de rede para colaboração em equipa
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração do PPK (Cinética Pós-Processada)

Se estiver a utilizar gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Ficheiro de registo .daq com entradas dos pinos de exposição
* Câmara ligada aos pinos de exposição do DAQ durante a sessão de captura

### Passos de configuração

1. Coloque o ficheiro de registo .daq na pasta do seu projeto
2. Nas Definições do Projeto, ative a caixa de seleção **«Aplicar correções PPK»**

3. Defina**«Desvio do fuso horário do sensor de luz»**, se necessário (predefinição: 0 para UTC)
4. Atribua as câmaras aos pinos de exposição:
   * **Câmara única**: atribuída automaticamente ao Pino 1
   * **Câmaras duplas**: atribua manualmente cada câmara ao pino correto**Atribuição de pinos de exposição:*** **Pino de exposição 1**: Selecione o modelo da câmara no menu suspenso
* **Pino de exposição 2**: Selecione a segunda câmara ou «Não utilizar»
* Não é possível atribuir a mesma câmara a ambos os pinos

{% hint style="warning" %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às respetivas câmaras. Uma atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmaras

Ao processar imagens de várias câmaras MAPIR num único projeto:

1. O Chloros deteta automaticamente cada modelo de câmara (tanto Survey3 como LATTICE)
2. Cada câmara recebe os perfis de processamento adequados e cada uma tem a sua própria árvore de pastas de saída
3. PPK: Atribua manualmente cada câmara Survey3 ao pino de exposição correto
4. Todas as câmaras utilizam o mesmo formato de exportação e os mesmos índices

**Exemplos**: Survey3W RGN + Survey3N OCN — configuração com duas câmaras, ou uma matriz LATTICE que combina uma câmara principal RGB com módulos de banda estreita

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
* Utilize o processamento paralelo Chloros+ para obter resultados mais rápidos
* Considere o CLI ou o API para automatização em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de deteção do alvo

***

## Verificação das suas definições

Antes de iniciar o processamento, verifique estas definições essenciais:

* [ ] Modelo da câmara detetado corretamente no Navegador de Ficheiros
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Para o Survey3: pelo menos uma imagem de alvo de calibração importada e verificada; para o LATTICE: um alvo e/ou uma gravação descendente do `.daq` presente
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação adequado ao seu fluxo de trabalho
* [ ] Definições PPK configuradas (se estiver a utilizar ficheiros .daq com eventos de exposição)

***

## Próximos passos

Depois de configurar as suas definições:

1. **Marque as imagens-alvo de calibração** - Consulte [Escolha de imagens-alvo](choosing-target-images.md)
2. **Inicie o processamento** - Consulte [Iniciar o processamento](starting-the-processing.md)
3. **Acompanhe o progresso** - Consulte [Acompanhar o processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as definições disponíveis, consulte a documentação de referência [Definições do projeto](../project-settings/project-settings.md).
