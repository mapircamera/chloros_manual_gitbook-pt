Iniciar o processamento

Depois de importar as imagens, marcar os alvos de calibração e configurar as definições do projeto, está pronto para iniciar o processamento. Esta página orienta-o na inicialização do pipeline de processamento Chloros.

## Lista de verificação pré-processamento

Antes de clicar no botão Iniciar, verifique se tudo está pronto:

* [ ] **Ficheiros importados** - Todas as imagens aparecem no Navegador de ficheiros
* [ ] **Imagens-alvo marcadas** - Coluna Alvo verificada para imagens de calibração
* [ ] **Modelos de câmara detetados** - A coluna Modelo da câmara mostra as câmaras corretas
* [ ] **Configurações definidas** - Configurações do projeto revistas e ajustadas
* [ ] **Índices selecionados** - Índices multiespectrais desejados adicionados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado para o seu fluxo de trabalho

{% hint style=&quot;info&quot; %}
**Dica**: Clique em algumas imagens no Navegador de ficheiros para verificar se elas foram carregadas corretamente antes do processamento.
{% endhint %}

***

## Iniciando o processamento

### Localize o botão Iniciar

O botão Iniciar/Reproduzir está localizado na barra de cabeçalho superior do Chloros:

* Posição: parte superior central da janela
* Ícone: **botão Reproduzir/Iniciar** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Estado: O botão fica ativado (brilhante) quando está pronto para processar

### Clique para iniciar

1. Clique no **botão Reproduzir/Iniciar** no cabeçalho superior
2. O processamento começa imediatamente
3. O botão fica desativado (acinzentado) durante o processamento
4. A barra de progresso é atualizada, mostrando o estado do processamento

{% hint style=&quot;success&quot; %}
**Processamento iniciado**: Depois de clicado, o Chloros lida automaticamente com todas as etapas do processamento - deteção do alvo, debayering, calibração, cálculo do índice e exportação.
{% endhint %}

***

## Compreender os modos de processamento

O Chloros funciona em dois modos de processamento diferentes, dependendo da sua licença:

### Modo gratuito (processamento sequencial)

**Disponível para todos os utilizadores**

**Como funciona:**

* Processa imagens uma de cada vez, sequencialmente
* Operação de thread único
* Menor uso de memória

**A barra de progresso mostra 2 etapas:**

1. **Detecção de alvo** - Verificação de alvos de calibração
2. **Processamento** - Aplicação de calibração e exportação de imagens

**Tempo de processamento:**

* Muito mais lento do que o modo paralelo Chloros+
* Adequado para conjuntos de dados pequenos a médios (&lt; 200 imagens)

### Modo Chloros+ (Processamento Paralelo)

**Requer licença Chloros+**

**Como funciona:**

* Processa várias imagens simultaneamente
* Operação multithread (até 16 trabalhadores paralelos)
* Utiliza vários núcleos de CPU
* Aceleração GPU (CUDA) opcional com placas gráficas NVIDIA

**A barra de progresso mostra 4 etapas:**

1. **Detecção** - Encontrar alvos de calibração
2. **Análise** - Examinar metadados da imagem e preparar pipeline
3. **Calibrando** - Aplicando correções e calibrações
4. **Exportando** - Salvando imagens processadas e índices

**Interação com a barra de progresso:**

* **Passe o mouse** sobre a barra para ver o painel suspenso detalhado de 4 etapas
* **Clique** na barra de progresso para congelar o painel suspenso no lugar
* **Clique novamente** para descongelar e ocultar o painel

**Tempo de processamento:**

* Significativamente mais rápido do que o modo gratuito
* Escala com a contagem de núcleos da CPU
* A aceleração da GPU melhora ainda mais a velocidade

{% hint style=&quot;info&quot; %}
**Chloros+ Velocidade**: O processamento paralelo pode ser 5 a 10 vezes mais rápido do que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que leva 2 horas no modo gratuito pode ser concluído em 15 a 20 minutos com o Chloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Fase 1: Detecção do alvo

**O que o Chloros faz:**

* Digitaliza imagens de alvos marcados (ou todas as imagens, se nenhuma estiver marcada)
* Identifica os 4 painéis de calibração em cada alvo
* Extrai valores de refletância dos painéis alvo
* Regista os carimbos de data/hora dos alvos para agendamento da calibração

**Duração:** 1-30 segundos (com alvos marcados), 5-30+ minutos (sem marcação)

### Fase 2: Debayering (conversão RAW)

**O que o Chloros faz:**

* Converte dados RAW do padrão Bayer em imagens RGB completas
* Aplica algoritmo de demosaicing de alta qualidade
* Preserva a máxima qualidade e detalhes da imagem

**Duração:** Varia de acordo com a quantidade de imagens e a velocidade da CPU

### Fase 3: Calibração

**O que o Chloros faz:**

* **Correção de vinheta**: remove o escurecimento da lente nas bordas
* **Calibração de refletância**: normaliza usando valores de refletância alvo
* Aplica correções em todas as bandas/canais
* Usa alvo de calibração apropriado para cada imagem com base no carimbo de data/hora

**Duração:** a maior parte do tempo de processamento

### Fase 4: Cálculo do índice

**O que o Chloros faz:**

* Calcula os índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplica matemática de banda às imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Fase 5: Exportação

**O que o Chloros faz:**

* Guarda imagens calibradas no formato selecionado
* Exporta imagens de índice com cores LUT configuradas
* Grava ficheiros em subpastas do modelo da câmara
* Preserva nomes de ficheiros originais com sufixos

**Duração:** Varia de acordo com o formato de exportação e o tamanho do ficheiro

***

## Comportamento do processamento

### Pipeline de processamento automático

Uma vez iniciado, todo o pipeline é executado automaticamente:

* Não é necessária interação do utilizador
* Todas as etapas configuradas são executadas em sequência
* Atualizações de progresso exibidas em tempo real

### Utilização do computador durante o processamento

**Modo livre:**

* Utilização relativamente baixa da CPU (single-threaded)
* O computador permanece responsivo para outras tarefas
* É seguro minimizar o Chloros e trabalhar noutras aplicações

**Chloros+ Modo paralelo:**

* Utilização elevada da CPU (multithread, até 16 núcleos)
* Com aceleração da GPU: utilização elevada da GPU
* O computador pode ficar menos responsivo durante o processamento
* Evite iniciar outras tarefas que exijam muito da CPU

{% hint style=&quot;warning&quot; %}
**Dica de desempenho**: Para obter o melhor desempenho do Chloros+, feche outras aplicações e deixe o Chloros usar todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado

**Limitações importantes:**

* Uma vez iniciado, o processamento não pode ser pausado
* É possível cancelar o processamento, mas o progresso será perdido
* Resultados parciais não são guardados
* É necessário reiniciar do início se for cancelado

**Dica de planeamento:** Para projetos muito grandes, considere processar em lotes ou usar o CLI para um melhor controlo.

***

## Monitorizar o seu processamento

Enquanto o processamento estiver em execução, pode:

* **Observar a barra de progresso** - Veja a percentagem geral de conclusão
* **Ver o estágio atual** - Detetar, Analisar, Calibrar ou Exportar
* **Verificar a guia de registo** - Veja mensagens e avisos detalhados do processamento
* **Visualizar imagens concluídas** - Alguns ficheiros de exportação podem aparecer durante o processamento

Para obter informações detalhadas sobre o monitoramento, consulte [Monitorando o processamento](monitoring-the-processing.md).

***

## Cancelando o processamento

Se precisar interromper o processamento:

### Como cancelar

1. Localize o **botão Parar/Cancelar** (substitui o botão Iniciar durante o processamento)
2. Clique no botão Parar
3. O processamento é interrompido imediatamente
4. Os resultados parciais são descartados

### Quando cancelar

**Motivos válidos para cancelar:**

* Percebeu que foram utilizadas configurações incorretas
* Esqueceu de marcar as imagens de destino
* Imagens erradas importadas
* Sistema muito lento ou sem resposta

**Após o cancelamento:**

* Revise e corrija quaisquer problemas
* Ajuste as configurações conforme necessário
* Reinicie o processamento desde o início
* Para uma experiência mais limpa, feche completamente o Chloros e reinicie

{% hint style=&quot;warning&quot; %}
**Sem resultados parciais**: O cancelamento descarta todo o progresso. O Chloros não guarda imagens parcialmente processadas.
{% endhint %}

***

## Estimativas de tempo de processamento

O tempo de processamento real varia muito com base em:

* Número de imagens
* Resolução da imagem
* Formato de entrada RAW vs JPG
* Modo de processamento (Free vs Chloros+)
* Velocidade da CPU e número de núcleos
* Disponibilidade da GPU (apenas Chloros+)
* Número de índices a calcular
* Complexidade do formato de exportação

### Estimativas aproximadas (Chloros+, imagens de 12 MP, CPU moderna)

| Número de imagens | Modo gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagens  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagens  | 1-1,5 horas | 20-30 min      | 10-15 min      |
| 500 imagens  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1000 imagens | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Primeira execução**: O processamento inicial pode demorar mais tempo, pois o Chloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns no início

### Botão Iniciar desativado (acinzentado)

**Possíveis causas:**

* Nenhuma imagem importada
* Backend não totalmente iniciado
* Processamento anterior ainda em execução
* Projeto não totalmente carregado

**Soluções:**

1. Aguarde até que o backend seja totalmente inicializado (verifique o ícone do menu principal)
2. Verifique se as imagens foram importadas no Navegador de Ficheiros
3. Reinicie o Chloros se o botão permanecer desativado
4. Verifique o Registo de Depuração para ver se há mensagens de erro

### O processamento inicia e falha imediatamente

**Possíveis causas:**

* Não há imagens válidas no projeto
* Ficheiros de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o registo de depuração <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> para ver se há mensagens de erro
2. Verifique o espaço disponível em disco
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### Aviso «Nenhum alvo detetado»

**Possíveis causas:**

* Esqueceu-se de marcar as imagens de destino
* As imagens de destino não contêm alvos visíveis
* As configurações de deteção de alvos são muito rígidas

**Soluções:**

1. Reveja [Escolher imagens-alvo](choosing-target-images.md)
2. Marque as imagens apropriadas na coluna Alvo
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as definições de deteção de alvos, se necessário

***

## Dicas para um processamento bem-sucedido

### Antes de começar

1. **Teste primeiro com um pequeno subconjunto** - Processe 10 a 20 imagens para verificar as configurações
2. **Verifique o espaço disponível em disco** - Certifique-se de que há 2 a 3 vezes o tamanho do conjunto de dados livre
3. **Feche aplicações desnecessárias** - Libere recursos do sistema
4. **Verifique as imagens-alvo** - Visualize os alvos marcados para garantir a qualidade
5. **Guarde o projeto** - O projeto é guardado automaticamente, mas é boa prática guardá-lo manualmente

### Durante o processamento

1. **Evite o modo de suspensão do sistema** - Desative os modos de poupança de energia
2. **Mantenha o Chloros em primeiro plano** - Ou, pelo menos, visível na barra de tarefas
3. **Monitorize o progresso ocasionalmente** - Verifique se há avisos ou erros
4. **Não carregue outras aplicações pesadas** - Especialmente com o modo paralelo Chloros+

### Chloros+ Aceleração da GPU

Se estiver a utilizar a aceleração da GPU NVIDIA:

1. Atualize os controladores NVIDIA para a versão mais recente
2. Certifique-se de que a GPU tem 4 GB+ de VRAM
3. Feche aplicações que utilizam muito a GPU (jogos, edição de vídeo)
4. Monitorize a temperatura da GPU (garanta um arrefecimento adequado)

***

## Próximos passos

Depois de iniciar o processamento:

1. **Monitorize o progresso** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** - O processamento é executado automaticamente
3. **Analise os resultados** - Consulte [Concluindo o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorando o processamento](monitoring-the-processing.md).
