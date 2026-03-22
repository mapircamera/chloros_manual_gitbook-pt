# Iniciar o processamento

Depois de importar as imagens, marcar os alvos de calibração e definir as configurações do projeto, está pronto para iniciar o processamento. Esta página orienta-o na ativação do fluxo de processamento Chloros.

## Lista de verificação do pré-processamento

Antes de clicar no botão Iniciar, verifique se tudo está pronto:

* [ ] **Ficheiros importados** - Todas as imagens aparecem no Navegador de Ficheiros
* [ ] **Imagens-alvo marcadas** - Coluna Alvo verificada para imagens de calibração
* [ ] **Modelos de câmara detetados** - A coluna Modelo de Câmara mostra as câmaras corretas
* [ ] **Definições configuradas** - Definições do projeto revistas e ajustadas
* [ ] **Índices selecionados** - Índices multiespectrais desejados adicionados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado ao seu fluxo de trabalho

{% hint style="info" %}
**Dica**: Clique em algumas imagens no Navegador de Ficheiros para verificar se foram carregadas corretamente antes do processamento.
{% endhint %}

***

## Iniciar o processamento

### Localizar o botão Iniciar

O botão Iniciar/Reproduzir está localizado na barra de cabeçalho superior do Chloros:

* Posição: Parte superior central da janela
* Ícone: **Botão Reproduzir/Iniciar** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Estado: O botão fica ativado (iluminado) quando estiver pronto para o processamento

### Clique para iniciar

1. Clique no **botão Reproduzir/Iniciar** na barra superior
2. O processamento começa imediatamente
3. O botão fica desativado (a cinzento) durante o processamento
4. A barra de progresso atualiza-se, mostrando o estado do processamento

{% hint style="success" %}
**Processamento iniciado**: Uma vez clicado, o Chloros lida automaticamente com todas as etapas de processamento - deteção do alvo, debayering, calibração, cálculo do índice e exportação.
{% endhint %}

***

## Compreender os modos de processamento

O Chloros opera em dois modos de processamento diferentes, dependendo da sua licença:

### Modo Gratuito (Processamento Sequencial)

**Disponível para todos os utilizadores**

**Como funciona:**

* Processa imagens uma de cada vez, sequencialmente
* Operação em thread único
* Menor utilização de memória

**A barra de progresso mostra 2 fases:**

1.**Detecção de alvos** - Pesquisa de alvos de calibração
2. **Processamento** - Aplicação da calibração e exportação de imagens**Tempo de processamento:**

* Muito mais lento do que o modo paralelo do Chloros+
* Adequado para conjuntos de dados pequenos a médios (&lt; 200 imagens)

### Modo Chloros+ (Processamento Paralelo)

**Requer licença Chloros+**

**Como funciona:**

* Processa várias imagens simultaneamente utilizando um [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md)
* A [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) seleciona automaticamente a estratégia ideal para o seu hardware
* Aceleração por GPU (CUDA) com placas gráficas NVIDIA (desktop e Jetson)
* Escalável desde um Jetson Nano (1 worker) até um desktop com GPU de 12 GB ou mais (3-4 workers)

**A barra de progresso mostra 4 fases** (correspondentes às 4 threads do pipeline):

1. **Detecção** (Thread 1) - Localização de alvos de calibração
2. **Análise** (Thread 2) - Exame dos metadados da imagem e cálculo da calibração
3. **Calibração** (Thread 3) - Debayering da GPU, correção de vinheta, cálculo do índice
4. **Exportação** (Thread 4) - Guardar imagens processadas e índices**Interação com a barra de progresso:*** **Passe o rato** sobre a barra para ver o painel suspenso detalhado de 4 fases
* **Clique** na barra de progresso para fixar o painel suspenso no lugar
* **Clique novamente** para desbloquear e ocultar o painel**Tempo de processamento:**

* Significativamente mais rápido do que o modo gratuito
* Escala com o número de núcleos da CPU
* A aceleração por GPU melhora ainda mais a velocidade

{% hint style="info" %}
**Chloros+ Velocidade**: O processamento paralelo pode ser 5 a 10 vezes mais rápido do que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que demora 2 horas no modo gratuito pode ser concluído em 15 a 20 minutos com o Chloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Fase 1: Detecção de alvos

**O que o Chloros faz:**

* Analisa imagens de alvos marcados (ou todas as imagens, se nenhuma estiver marcada)
* Identifica os 4 painéis de calibração em cada alvo
* Extraia valores de refletância dos painéis de alvo
* Regista carimbos de data/hora dos alvos para agendamento da calibração

**Duração:** 1-30 segundos (com alvos marcados), 5-30+ minutos (sem marcação)

### Fase 2: Debayering (Conversão RAW)

**O que o Chloros faz:**

* Converte dados RAW do padrão Bayer em imagens RGB completas
* Aplica um algoritmo de demosaicing de alta qualidade
* Preserva a máxima qualidade de imagem e detalhe

**Duração:** Varia de acordo com o número de imagens e a velocidade da CPU

### Fase 3: Calibração

**O que o Chloros faz:*** **Correção de vinheta**: Remove o escurecimento das bordas causado pela lente
* **Calibração de refletância**: Normaliza utilizando valores de refletância de referência
* Aplica correções em todas as bandas/canais
* Utiliza alvo de calibração apropriado para cada imagem com base na data e hora

**Duração:** A maior parte do tempo de processamento

### Fase 4: Cálculo de Índices

**O que o Chloros faz:**

* Calcula os índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplica cálculos matemáticos de banda às imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Etapa 5: Exportação

**O que o Chloros faz:**

* Guarda imagens calibradas no formato selecionado
* Exporta imagens de índice com cores LUT configuradas
* Grava ficheiros em subpastas do modelo da câmara
* Mantém os nomes de ficheiro originais com sufixos

**Duração:** Varia de acordo com o formato de exportação e o tamanho do ficheiro***

## Comportamento do processamento

### Pipeline de processamento automático

Uma vez iniciado, todo o pipeline é executado automaticamente:

* Não é necessária interação do utilizador
* Todas as etapas configuradas são executadas em sequência
* Atualizações de progresso apresentadas em tempo real

### Utilização do computador durante o processamento

**Modo Livre:**

* Utilização relativamente baixa da CPU (single-threaded)
* O computador permanece responsivo para outras tarefas
* É seguro minimizar o Chloros e trabalhar noutras aplicações

**Chloros+ Modo Paralelo:**

* Elevada utilização da CPU (multi-threaded, até 16 núcleos)
* Com aceleração por GPU: Elevada utilização da GPU
* O computador pode ficar menos responsivo durante o processamento
* Evite iniciar outras tarefas que exijam muito da CPU

{% hint style="warning" %}
**Dica de desempenho**: Para obter o melhor desempenho do Chloros+, feche outras aplicações e deixe o Chloros utilizar todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado

**Limitações importantes:**

* Uma vez iniciado, o processamento não pode ser pausado
* Pode cancelar o processamento, mas o progresso será perdido
* Os resultados parciais não são guardados
* É necessário reiniciar desde o início se for cancelado

**Dica de planeamento:** Para projetos muito grandes, considere processar em lotes ou utilizar o CLI para um melhor controlo.***

## Monitorizar o seu processamento

Enquanto o processamento está a decorrer, pode:

* **Observar a barra de progresso** - Ver a percentagem global de conclusão
* **Ver a fase atual** - Detetar, Analisar, Calibrar ou Exportar
* **Verificar o separador de registo** - Ver mensagens e avisos detalhados do processamento
* **Pré-visualizar imagens concluídas** - Alguns ficheiros de exportação podem aparecer durante o processamento

Para informações detalhadas sobre monitorização, consulte [Monitorização do processamento](monitoring-the-processing.md).

***

## Cancelar o processamento

Se precisar de interromper o processamento:

### Como cancelar

1. Localize o **botão Parar/Cancelar** (substitui o botão Iniciar durante o processamento)
2. Clique no botão Parar
3. O processamento é interrompido imediatamente
4. Os resultados parciais são descartados

### Quando cancelar

**Motivos válidos para cancelar:**

* Percebeu que foram utilizadas configurações incorretas
* Esqueceu-se de marcar as imagens de destino
* Importou imagens erradas
* O sistema está a funcionar muito lentamente ou não responde

**Após o cancelamento:**

* Reveja e corrija quaisquer problemas
* Ajuste as configurações conforme necessário
* Reinicie o processamento desde o início
* Para uma experiência mais limpa, feche completamente o Chloros e reinicie

{% hint style="warning" %}
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

| Número de imagens | Modo Gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagens  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagens  | 1-1,5 horas | 20-30 min      | 10-15 min      |
| 500 imagens  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1000 imagens | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style="info" %}
**Primeira execução**: O processamento inicial pode demorar mais tempo, uma vez que o Chloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns no arranque

### Botão Iniciar desativado (a cinzento)

**Possíveis causas:**

* Não foram importadas imagens
* O backend não foi totalmente iniciado
* O processamento anterior ainda está a decorrer
* O projeto não foi totalmente carregado

**Soluções:**

1. Aguarde até que o backend seja totalmente inicializado (verifique o ícone do menu principal)
2. Verifique se as imagens foram importadas no Navegador de Ficheiros
3. Reinicie o Chloros se o botão continuar desativado
4. Verifique o Registo de Depuração para mensagens de erro

### O processamento inicia e falha imediatamente

**Possíveis causas:**

* Não há imagens válidas no projeto
* Ficheiros de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o Registo de Depuração <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> para verificar se há mensagens de erro
2. Verifique o espaço disponível em disco
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### Aviso «Nenhum alvo detetado»

**Possíveis causas:**

* Esqueceu-se de marcar as imagens-alvo
* As imagens-alvo não contêm alvos visíveis
* Configurações de deteção de alvos demasiado restritas

**Soluções:**

1. Consulte [Escolha de imagens-alvo](choosing-target-images.md)
2. Marque as imagens apropriadas na coluna «Alvo»
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as definições de deteção de alvos, se necessário

***

## Dicas para um processamento bem-sucedido

### Antes de começar

1. **Teste primeiro com um pequeno subconjunto** - Processe 10-20 imagens para verificar as definições
2. **Verifique o espaço disponível em disco** - Garanta que haja espaço livre equivalente a 2-3 vezes o tamanho do conjunto de dados
3. **Feche aplicações desnecessárias** - Liberte recursos do sistema
4. **Verifique as imagens-alvo** - Pré-visualize os alvos marcados para garantir a qualidade
5. **Guarde o projeto** - O projeto é guardado automaticamente, mas é boa prática guardá-lo manualmente

### Durante o processamento

1. **Evite que o sistema entre em modo de suspensão** - Desative os modos de poupança de energia
2. **Mantenha o Chloros em primeiro plano** - Ou, pelo menos, visível na barra de tarefas
3. **Monitorize o progresso ocasionalmente** - Verifique se há avisos ou erros
4. **Não carregue outras aplicações pesadas** - Especialmente com o modo paralelo do Chloros+

### Aceleração por GPU do Chloros+

Se estiver a utilizar a aceleração por GPU da NVIDIA:

1. Atualize os controladores da NVIDIA para a versão mais recente
2. Certifique-se de que a GPU tem 4 GB ou mais de VRAM
3. Feche aplicações que exigem muito da GPU (jogos, edição de vídeo)
4. Monitorize a temperatura da GPU (garanta um arrefecimento adequado)

***

## Próximos passos

Assim que o processamento tiver começado:

1. **Monitorize o progresso** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** - O processamento é executado automaticamente
3. **Analise os resultados** - Consulte [Concluir o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorizar o processamento](monitoring-the-processing.md).
