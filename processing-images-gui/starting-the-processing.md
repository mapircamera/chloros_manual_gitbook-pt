# Iniciar o processamento

Depois de importar as suas imagens, marcar os seus alvos de calibração e configurar as definições do projeto, está pronto para iniciar o processamento. Esta página orienta-o na inicialização do fluxo de processamento do Chloros.

## Lista de verificação do pré-processamento

Antes de clicar no botão «Iniciar», verifique se está tudo pronto:

* [ ] **Ficheiros importados** - Todas as imagens aparecem no Navegador de ficheiros
* [ ] **Imagens-alvo marcadas** - Coluna «Alvo» verificada para imagens de calibração (ou uma gravação `.daq` importada para o LATTICE)
* [ ] **Modelos de câmara detetados** — A coluna «Modelo de câmara» apresenta as câmaras corretas
* [ ] **Definições configuradas** — As definições do projeto foram revistas e ajustadas
* [ ] **Índices selecionados** — Os índices multiespectrais pretendidos foram adicionados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado ao seu fluxo de trabalho

{% hint style="info" %}
**Dica**: Clique em algumas imagens no Navegador de Ficheiros para verificar se foram carregadas corretamente antes do processamento.
{% endhint %}

***

## Iniciar o processamento

### Localizar o botão «Iniciar»

O botão «Iniciar/Reproduzir» encontra-se na barra de cabeçalho superior do Chloros:

* Posição: Parte superior central da janela
* Ícone: **Botão «Reproduzir/Iniciar»** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Estado: O botão fica ativado (iluminado) quando estiver pronto para o processamento

### Clique para iniciar

1. Clique no **botão Reproduzir/Iniciar** na barra superior
2. O processamento inicia imediatamente
3. O botão transforma-se num botão **Parar** durante o processamento
4. A barra de progresso atualiza-se, mostrando o estado do processamento

{% hint style="success" %}
**Processamento iniciado**: Assim que for clicado, o Chloros trata automaticamente de todas as etapas do processamento — deteção de alvos, debayering, calibração, cálculo do índice e exportação. Deteta automaticamente se o seu projeto é Survey3, LATTICE ou uma combinação de ambos, e aplica o fluxo de trabalho adequado a cada câmara.
{% endhint %}

***

## Compreender os modos de processamento

O Chloros funciona em dois modos de processamento diferentes, dependendo da sua licença:

### Modo gratuito (processamento sequencial)

**Disponível para todos os utilizadores**

**Como funciona:**

* Processa as imagens uma de cada vez, sequencialmente
* Operação em thread único
* Menor consumo de memória

**A barra de progresso apresenta 2 fases:**

1.**Detecção de alvos** - Pesquisa de alvos de calibração
2. **Processamento** - Aplicação da calibração e exportação de imagens**Tempo de processamento:**

* Muito mais lento do que o modo paralelo do Chloros+
* Adequado para conjuntos de dados pequenos a médios (&lt; 200 imagens)

### Modo Chloros+ (Processamento Paralelo)

**Requer licença Chloros+**

**Como funciona:**

* Processa várias imagens simultaneamente utilizando um [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md)
* A [Adaptação Dinâmica de Cálculo](../processing-architecture/dynamic-compute-adaptation.md) seleciona automaticamente a estratégia ideal para o seu hardware no início da execução
* Aceleração por GPU (CUDA) com placas gráficas NVIDIA (computadores de secretária e Jetson)
* **O número de trabalhadores adapta-se ao hardware**: as estratégias de GPU executam**1 a 4 trabalhadores simultâneos** (escalados pela VRAM — um Jetson com pouca memória executa 1, uma GPU de computador de secretária com 12 GB ou mais executa até 4); Os sistemas apenas com CPU executam um trabalhador por núcleo físico, menos um**A barra de progresso mostra 4 fases** (correspondentes às 4 threads do pipeline):

1. **Detecção** (Thread 1) — Identificação de alvos de calibração
2. **Análise** (Thread 2) — Análise dos metadados da imagem e cálculo da calibração
3. **Calibração** (Thread 3) — Remoção do efeito «bayering», correção de vinheta, calibração, cálculo do índice
4. **Exportação** (Thread 4) — Gravação das imagens processadas e dos índices**Interação com a barra de progresso:*** **Passe o rato** sobre a barra para ver o painel suspenso detalhado com as 4 fases
* **Clique** na barra de progresso para fixar o painel suspenso no lugar
* **Clique novamente** para desbloquear e ocultar o painel**Tempo de processamento:**

* Significativamente mais rápido do que o modo gratuito
* A aceleração por GPU melhora ainda mais a velocidade

{% hint style="info" %}
**Chloros+ Velocidade**: O processamento paralelo pode ser 5 a 10 vezes mais rápido do que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que demora 2 horas no modo gratuito pode ser concluído em 15 a 20 minutos com o Chloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Fase 1: Detecção de alvos

**O que o Chloros faz:**

* Analisa as imagens que selecionou na coluna «Alvo» (todas as imagens, apenas se nenhuma estiver marcada)
* Identifica os painéis de calibração em cada alvo
* Extrai os valores de refletância dos painéis dos alvos
* Regista os carimbos de data e hora dos alvos para o agendamento da calibração

**Duração:** 1 a 30 segundos (com alvos marcados), 5 a 30+ minutos (sem alvos marcados)

### Fase 2: Debayering (Conversão RAW)

**O que o Chloros faz:**

* Converte dados RAW no padrão Bayer em imagens completas de 3 canais (os módulos LATTICE mono permanecem em banda única — o debayering é ignorado para estes, com uma nota no registo)
* Aplica o algoritmo de demosaicing selecionado
* Preserva a máxima qualidade de imagem e detalhe

**Duração:** Varia consoante o número de imagens e a velocidade da CPU/GPU

### Fase 3: Calibração

**O que o Chloros faz:*** **Correção de vinheta**: Remove o escurecimento da lente nas bordas
* **Calibração da refletância**: Normaliza utilizando valores-alvo de refletância e/ou dados de radiação descendente do DAQ
* Aplica correções em todas as bandas/canais
* Utiliza a referência de calibração adequada para cada imagem com base no carimbo temporal

**Duração:** A maior parte do tempo de processamento

### Fase 4: Cálculo do índice

**O que o Chloros faz:**

* Calcula os índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplica cálculos matemáticos às bandas das imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Fase 5: Exportação

**O que o Chloros faz:**

* Guarda as imagens processadas no formato selecionado
* **LATTICE fan-out**: cada fotograma LATTICE em bruto é exportado como todos os produtos ativados numa única passagem — sem bayering, pré-visualização, radiação (sempre float32), refletância
* Grava os ficheiros na árvore de saída do projeto: `<project>/<camera>/<format>/<Product>_Images/`
* **Mantém o nome do ficheiro de origem** — a pasta identifica o produto, não é adicionado qualquer sufixo**Duração:** Varia consoante o formato de exportação e o tamanho do ficheiro***

## Comportamento do processamento

### Pipeline de processamento automático

Uma vez iniciado, todo o pipeline é executado automaticamente:

* Não é necessária qualquer interação do utilizador
* Todas as etapas configuradas são executadas em sequência
* As atualizações de progresso são apresentadas em tempo real
* Os ficheiros exportados são gravados no disco à medida que são concluídos — pode abrir os resultados finalizados enquanto a execução continua

### Utilização do computador durante o processamento

**Modo Livre:**

* Utilização relativamente baixa da CPU (monothread)
* O computador permanece responsivo para outras tarefas
* É seguro minimizar o Chloros e trabalhar noutras aplicações

**Modo Paralelo Chloros+:**

* Elevado consumo de CPU em todo o conjunto de processos da estratégia
* Com aceleração por GPU: elevado consumo de GPU
* O computador pode ficar menos responsivo durante o processamento
* Evite iniciar outras tarefas que exijam muito da CPU

{% hint style="warning" %}
**Dica de desempenho**: Para obter o melhor desempenho do Chloros+, feche outras aplicações e deixe o Chloros utilizar todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado (mas a interrupção é definitiva)

* Uma vez iniciado, o processamento não pode ser pausado e retomado posteriormente
* Clicar em **Parar** interrompe a execução de forma definitiva logo no primeiro clique
* Os produtos já exportados antes da paragem permanecem no disco
* Uma execução interrompida relata com precisão o que foi concluído (consulte as linhas `[RUN-SUMMARY]` no registo)
* Uma nova execução inicia o pipeline desde o início

**Dica de planeamento:** Para projetos muito grandes, considere o processamento em lotes ou a utilização do CLI para um melhor controlo.***

## Monitorizar o seu processamento

Enquanto o processamento está a decorrer, pode:

* **Observar a barra de progresso** – Ver a percentagem global de conclusão
* **Ver a fase atual** — Detecção, Análise, Calibração ou Exportação
* **Verificar o separador do registo** — Ver mensagens e avisos detalhados do processamento
* **Pré-visualizar imagens concluídas** — Os ficheiros de exportação aparecem no disco durante o processamento

Para informações detalhadas sobre a monitorização, consulte [Monitorizar o processamento](monitoring-the-processing.md).

***

## Interromper o processamento

Se precisar de interromper o processamento:

### Como interromper

1. Localize o **botão «Parar»** (substitui o botão «Iniciar» durante o processamento)
2. Clique nele uma vez — a barra exibe **«A parar...»** enquanto a imagem em processamento é concluída
3. A execução termina num estado de paragem definitiva e o registo apresenta um relatório detalhado `[RUN-SUMMARY]` do que foi concluído

### Quando interromper

**Motivos válidos para interromper:**

* Percebeu-se que foram utilizadas configurações incorretas
* Esqueceu-se de marcar as imagens-alvo
* Foram importadas imagens erradas
* O sistema está a funcionar demasiado lentamente ou não responde

**Após a interrupção:**

* Os produtos exportados antes da interrupção permanecem no disco
* Analise e corrija quaisquer problemas, ajuste as configurações conforme necessário
* Reinicie o processamento — a execução recomeça do início

***

## Estimativas do tempo de processamento

O tempo real de processamento varia consideravelmente consoante:

* Número de imagens
* Resolução das imagens
* Formato de entrada RAW vs. JPG
* Modo de processamento (Free vs. Chloros+)
* Velocidade da CPU e número de núcleos
* Disponibilidade da GPU (apenas Chloros+)
* Número de índices a calcular
* Número de produtos de exportação ativados (LATTICE)

### Estimativas aproximadas (Chloros+, imagens de 12 MP, CPU moderna)

| Número de imagens | Modo Gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagens  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagens  | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 imagens  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1000 imagens | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style="info" %}
**Primeira execução**: O processamento inicial pode demorar mais tempo, uma vez que o Chloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns no arranque

### Botão «Iniciar» desativado (a cinzento)

**Possíveis causas:**

* Não foram importadas imagens
* O backend não foi totalmente iniciado
* O processamento anterior ainda está a decorrer
* O projeto não foi totalmente carregado

**Soluções:**

1. Aguarde até que o backend se inicialize totalmente (verifique o ícone do menu principal)
2. Verifique se as imagens foram importadas no Navegador de Ficheiros
3. Reinicie o Chloros se o botão continuar desativado
4. Verifique o Registo de Depuração para ver se há mensagens de erro

### O processamento inicia e falha imediatamente

**Possíveis causas:**

* Não existem imagens válidas no projeto
* Ficheiros de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o registo de depuração <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> para ver se há mensagens de erro
2. Verifique o espaço disponível em disco
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### A execução termina, mas não grava imagens

Uma execução que solicitou produtos de imagem, mas não gravou nenhum, é tratada como uma **falha, não como um sucesso** — O Chloros assinala isso claramente:

* O registo da GUI apresenta mensagens do tipo `[RUN-SUMMARY]` que indicam a causa provável — nenhuma imagem importada, nenhum alvo detetado ou todos os produtos solicitados ignorados por serem inaplicáveis (por exemplo, solicitar radianância/refletância de câmaras que suportam apenas RGB)
* O equivalente ao CLI (`chloros-cli process`) apresenta o `Processing finished but wrote no image products.` e **termina com um valor diferente de zero**, pelo que os scripts podem detetá-lo
* Uma execução deliberada apenas com metadados (todos os produtos de exportação desativados, sem índices) continua a ser considerada bem-sucedida

Consulte [a Referência do CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) para a semântica completa.

### Aviso «Nenhum alvo detetado»

**Possíveis causas:**

* Esqueceu-se de marcar as imagens-alvo
* As imagens-alvo não contêm alvos visíveis
* Configurações de deteção de alvos demasiado restritivas

**Soluções:**

1. Consulte [Como escolher imagens-alvo](choosing-target-images.md)
2. Marque as imagens adequadas na coluna «Alvo»
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as definições de deteção de alvos, se necessário

***

## Dicas para um processamento bem-sucedido

### Antes de começar

1. **Teste primeiro com um pequeno subconjunto** — Processe 10 a 20 imagens para verificar as definições
2. **Verifique o espaço disponível em disco** — Garanta que haja espaço livre equivalente a 2 a 3 vezes o tamanho do conjunto de dados (mais, se todos os produtos LATTICE estiverem ativados)
3. **Feche aplicações desnecessárias** — Liberte recursos do sistema
4. **Verifique as imagens dos alvos** — Visualize os alvos marcados para garantir a qualidade
5. **Guarde o projeto** — O projeto é guardado automaticamente, mas é boa prática guardá-lo manualmente

### Durante o processamento

1. **Evite que o sistema entre em modo de suspensão** - Desative os modos de poupança de energia
2. **Mantenha o Chloros em primeiro plano** - Ou, pelo menos, visível na barra de tarefas
3. **Acompanhe o progresso ocasionalmente** - Verifique se há avisos ou erros
4. **Não carregue outras aplicações pesadas** — Especialmente com o modo paralelo do Chloros+

### Aceleração por GPU do Chloros+

Se estiver a utilizar a aceleração por GPU da NVIDIA:

1. Atualize os controladores da NVIDIA para a versão mais recente
2. Certifique-se de que a GPU tem 4 GB ou mais de VRAM (7 GB ou mais para o «debayering» simultâneo com reconhecimento de texturas)
3. Feche aplicações que exigem muito da GPU (jogos, edição de vídeo)
4. Monitorize a temperatura da GPU (garanta um arrefecimento adequado)

***

## Próximos passos

Assim que o processamento tiver começado:

1. **Acompanhe o progresso** — Consulte [Monitorização do Processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** — O processamento decorre automaticamente
3. **Analise os resultados** — Consulte [Concluir o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorizar o processamento](monitoring-the-processing.md).
