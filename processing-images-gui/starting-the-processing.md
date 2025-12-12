# Iniciando o processamento

Depois de importar suas imagens, marcar seus alvos de dificuldades e definir as configurações do projeto, você estará pronto para iniciar o processamento. Esta página orienta você no início do pipeline de processamento da Cloros.

## Lista de verificação de pré-processamento

Antes de clicar no botão Iniciar, verifique se tudo está pronto:

* [ ] **Arquivos importados** - Todas as imagens aparecem no Navegador de Arquivos
* [ ] **Imagens alvo marcadas** - Coluna alvo verificada para imagens encontradas
* [ ] **Modelos de câmera detectados** - A coluna Modelo de câmera mostra as câmeras corretas
* [ ] **Configurações ajustadas** - Configurações do projeto revisadas e ajustadas
* [ ] **Índices selecionados** - Índices multiespectrais desejados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado para seu fluxo de trabalho

{% dica estilo = "info" %}
**Dica**: clique em algumas imagens no Navegador de arquivos para verificar se foram incluídas corretamente antes do processamento.
{% endhint %}

***

## Iniciando o processamento

### Localize o botão Iniciar

O botão/Reproduzir está localizado na barra de cabeçalho superior do Chloros:

* Posição: Centro superior da janela
* Ícone: **Botão Reproduzir/Iniciar**<img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: O botão está ativado (brilhante) quando pronto para processar

### Clique para começar

1. Clique no **botão Reproduzir/Iniciar** no cabeçalho superior
2. O processamento começa imediatamente
3. O botão fica desativado (acinzentado) durante o processamento
4. Atualizações da barra de progresso, mostrando o status do processamento

{% dica estilo = "sucesso" %}
**Processamento iniciado**: Uma vez clicado, o Cloros lida automaticamente com todas as etapas de processamento - detecção de alvo, depuração, exclusão, cálculo de índice e exportação.
{% endhint %}

***

## Compreendendo os modos de processamento

A Chloros opera em dois modos de processamento diferentes dependendo de sua licença:

### Modo Livre (Processamento Sequencial)

**Disponível para todos os usuários**

**Como funciona:**

* Processa imagens uma de cada vez, sequencialmente
* Operação de thread único
* Menor uso de memória

**A barra de progresso mostra 2 etapas:**

1. **Target Detect** - Procurando alvos específicos
2. **Processando** - Aplicando internamente e exportando imagens

**Tempo de processamento:**

* Muito mais lento que o modo paralelo Cloros+
* Adequado para conjuntos de dados pequenos e médios (<200 imagens)

### Modo Cloros+ (processamento paralelo)

**Solicitar licença Cloros+**

**Como funciona:**

* Processo múltiplas imagens simultaneamente
* Operação multithread (até 16 trabalhadores paralelos)
* Utilize vários núcleos de CPU
* Aceleração GPU (CUDA) opcional com placas gráficas NVIDIA

**A barra de progresso mostra 4 etapas:**

1. **Detectando** - Encontrando alvos específicos
2. **Analisando** – Examinando metadados de imagem e preparando pipeline
3. **Calibrando** - Aplicando correções e calibrações
4. **Exportando** - Salvando imagens e índices processados

**Interação da barra de progresso:**

* **Passe o mouse** sobre a barra para ver o painel suspenso detalhado de 4 estágios
* **Clique** na barra de progresso para congelar o painel suspenso no lugar
* **Clique novamente** para descongelar e ocultar o painel

**Tempo de processamento:**

* Significativamente mais rápido que o modo livre
* Escala com contagem de núcleos de CPU
* A aceleração da GPU melhorou ainda mais a velocidade

{% dica estilo = "info" %}
**Cloros+ Speed**: o processamento paralelo pode ser de 5 a 10 vezes mais rápido que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que leva 2 horas no modo livre pode ser concluído em 15 a 20 minutos com o Cloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Estágio 1: Detecção de Alvo

**O que o Cloros faz:**

* Verifique as imagens marcadas (ou todas as imagens, se não estiverem marcadas)
* Identifique os 4 painéis defeituosos em cada alvo
* Extrai valores de refletância de painéis de destino
* Registrar carimbos de dados e hora alvo para agendamento de reserva

**Duração:** 1 a 30 segundos (com alvos marcados), 5 a 30+ minutos (sem marcação)

### Etapa 2: Debayering (conversão RAW)

**O que o Cloros faz:**

* Converta dados RAW do padrão Bayer em imagens RGB completas
* Aplicativo algoritmo de demosaicing de alta qualidade
* Preservar a máxima qualidade e detalhes da imagem

**Duração:** Varia de acordo com a contagem de imagens e a velocidade da CPU

### Etapa 3: Calibração

**O que o Cloros faz:**

* **Correção de vinheta**: remoção do escurecimento da lente nas bordas
* **Calibração de refletância**: normaliza usando valores de refletância alvo
* Aplicar correções em todas as bandas/canais
* Use alvo de proteção protetor para cada imagem com base no carimbo de dados/hora

**Duração:** Maioria do tempo de processamento

### Etapa 4: Cálculo do Índice

**O que o Cloros faz:**

* Calcular índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplicação matemática de banda a imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Etapa 5: Exportar

**O que o Cloros faz:**

* Salvar imagens calibradas no formato selecionado
* Exportar imagens de índice com núcleos LUT configurados
* Grava arquivos nas subpastas do modelo da câmera
* Preservar nomes de arquivos originais com sufixos

**Duração:** Varia de acordo com o formato de exportação e tamanho do arquivo

***

## Comportamento de processamento

### Pipeline de processamento automático

Depois de iniciado, todo o pipeline é executado automaticamente:

* Nenhuma interação do usuário necessária
* Todas as etapas definidas são realizadas em sequência
* Atualizações de progresso mostradas em tempo real

### Uso do computador durante o processamento

**Modo Gratuito:**

* Uso relativamente baixo de CPU (threaded único)
* O computador permanece responsivo para outras tarefas
* Seguro para minimizar cloros e trabalhar em outras aplicações

**Modo Cloros+ Paralelo:**

* Alto uso de CPU (multithread, até 16 núcleos)
* Com influência de GPU: Alto uso de GPU
* O computador pode responder menos durante o processamento
* Evite iniciar outras tarefas com uso intensivo de CPU

{% dica estilo = "aviso" %}
**Dica de desempenho**: Para obter melhor desempenho do Cloros+, feche outros aplicativos e deixe o Cloros usar todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado

**Limitações importantes:**

* Uma vez iniciado, o processamento não pode ser interrompido
* Você pode cancelar o processamento, mas o progresso será perdido
* Os resultados parciais não são salvos
* Deve reiniciar do início se cancelado

**Dica de planejamento:** para projetos muito grandes, considere o processamento em lotes ou o uso da CLI para melhor controle.

***

## Monitorando seu processamento

Durante a execução do processamento, você pode:

* **Assista à barra de progresso** - Veja a porcentagem geral de conclusão
* **Ver estágio atual** - Detectar, analisar, calibrar ou exportar
* **Verificar guia de log** - Veja mensagens e avisos detalhados de processamento
* **Visualizar imagens concluídas** - Alguns arquivos de exportação podem aparecer durante o processamento

Para obter informações detalhadas sobre monitoramento, consulte [Monitorando o Processamento](monitoring-the-processing.md).

***

## Cancelando processamento

Se você precisar interromper o processamento:

### Como cancelar

1. Localize o **botão Parar/Cancelar** (substitui o botão Iniciar durante o processamento)
2. Clique no botão Parar
3. O processamento é interrompido imediatamente
4. Resultados parciais são descartados

### Quando cancelar

**Motivos válidos para cancelar:**

* Configurações incorretas percebidas foram usadas
* Esqueci de marcar as imagens alvo
* Imagens erradas importadas
* Sistema funcionando muito lento ou sem resposta

**Após cancelar:**

* Revise e corrija quaisquer problemas
* Ajuste as configurações conforme necessário
* Reinicie o processamento desde o início
* Para uma experiência mais limpa, feche completamente o Cloros e reinicie

{% dica estilo = "aviso" %}
**Sem resultados parciais**: cancelar descartado todo o progresso. O Chloros não salva imagens parcialmente processadas.
{% endhint %}

***

## Estimativas de tempo de processamento

O tempo real de processamento varia muito com base em:

* Número de imagens
* Resolução da imagem
* Formato de entrada RAW vs JPG
* Modo de processamento (Livre vs Cloros+)
* Velocidade da CPU e contagem de núcleos
* Disponibilidade de GPU (somente Chloros+)
* Número de índices a serem calculados
* Complexidade do formato de exportação

### Estimativas aproximadas (Chloros +, imagens de 12 MP, CPU moderna)

| Contagem de imagens | Modo Livre | Cloro+ (CPU) | Cloro + (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens | 15-20 minutos | 5-8 minutos | 3-5 minutos |
| 100 imagens | 30-40 minutos | 10-15 minutos | 5-8 minutos |
| 200 imagens | 1-1,5 horas | 20-30 minutos | 10-15 minutos |
| 500 imagens | 2-3 horas | 45-60 minutos | 20-30 minutos |
| 1000 imagens | 4-6 horas | 1,5-2 horas | 40-60 minutos |

{% dica estilo = "info" %}
**Primeira execução**: o processamento inicial pode demorar mais, pois o Cloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns no início

### Botão Iniciar desativado (acinzentado)

**Causas possíveis:**

* Nenhuma imagem importada
* Back-end não totalmente iniciado
* O processamento anterior ainda está em execução
* Projeto não totalmente carregado

**Soluções:**

1. Mantenha a inicialização completa do back-end (verifique o ícone do menu principal)
2. Verifique se as imagens são importadas no navegador de arquivos
3. Reinicie o Cloros se o botão permanecer desativado
4. Verifique o log de depuração para mensagens de erro

### O processamento começa e falha imediatamente

**Causas possíveis:**

* Nenhuma imagem válida no projeto
* Arquivos de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o registro de depuração<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> para mensagens de erro
2. Confira o espaço em disco disponível
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### Aviso "Nenhum alvo detectado"

**Causas possíveis:**

* Esqueci de marcar as imagens alvo
* As imagens alvo não contêm alvos visíveis
* Configurações de detecção de alvo muito rigorosas

**Soluções:**

1. Revisar [Escolhendo imagens de destino](choosing-target-images.md)
2. Marque as imagens comentadas na coluna Destino
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as configurações de detecção de alvo, se necessário

***

## Dicas para processamento bem sucedido

### Antes de começar

1. **Teste primeiro com um subconjunto pequeno** - Processo de 10 a 20 imagens para verificar as configurações
2. **Verifique o espaço em disco disponível** - Garanta que o tamanho do conjunto de dados fique livre de 2 a 3 vezes
3. **Feche aplicativos desnecessários** - Recursos gratuitos do sistema
4. **Verifique as imagens alvo** - Visualize os alvos marcados para garantir a qualidade
5. **Salvar projeto** – O projeto é salvo automaticamente, mas é uma boa prática salvar manualmente

### Durante o processamento

1. **Evite suspensão do sistema** - Desative os modos de economia de energia
2. **Mantenha o Cloros no primeiro plano** - Ou pelo menos visível na barra de tarefas
3. **Monitore o progresso ocasionalmente** - Verifique se há avisos ou erros
4. **Não carregue outras aplicações pesadas** - Especialmente com o modo paralelo Chloros+

### Aceleração GPU Cloros+

Se você estiver usando GPU NVIDIA:

1. Atualizar os drivers NVIDIA para a versão mais recente
2. Se de que a GPU tenha 4 GB + VRAM
3. Buscar aplicativos com uso intensivo de GPU (jogos, edição de vídeo)
4. Monitore a temperatura da GPU (garanta o resfriamento adequado)

***

## Próximas etapas

Assim que o processamento foi iniciado:

1. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** - O processamento é executado automaticamente
3. **Revisar resultados** - Consulte [Concluindo o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorando o processamento](monitoring-the-processing.md).