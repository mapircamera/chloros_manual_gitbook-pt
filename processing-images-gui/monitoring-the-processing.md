# Monitorização do processamento

Assim que o processamento tiver início, o Chloros disponibiliza várias formas de monitorizar o progresso, verificar se existem problemas e compreender o que se passa com o seu conjunto de dados. Esta página explica como acompanhar o seu processamento e interpretar as informações fornecidas pelo Chloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o estado do processamento em tempo real e a percentagem de conclusão. O progresso é transmitido em direto a partir do backend através de Server-Sent Events (SSE), pelo que a barra reflete o que o pipeline está efetivamente a fazer.

### Barra de progresso no modo gratuito

Para utilizadores sem licença Chloros+:

**Exibição do progresso em 2 fases:**

1.**Detecção de alvos** — Localização de alvos de calibração nas imagens
2. **Processamento** – Aplicação de correções e exportação**A barra de progresso mostra:**

* Percentagem global de conclusão (0-100%)
* Nome da fase atual
* Visualização simples com barra horizontal

### Barra de progresso do Chloros+

Para utilizadores com licença Chloros+:

**Exibição de progresso em 4 fases:**

1.**Detecção** - Localização de alvos de calibração
2. **Análise** - Exame das imagens e preparação do fluxo de trabalho
3. **Calibração** - Aplicação de correções de vinheta e refletância
4. **Exportação** - Gravação dos ficheiros processados**Funcionalidades interativas:*** **Passe o cursor sobre** a barra de progresso para ver o painel expandido de 4 fases
* **Clique** na barra de progresso para fixar o painel expandido
* **Clique novamente** para desfixar e ocultar automaticamente quando o cursor sair
* Cada fase mostra o progresso individual (0-100%)

{% hint style="info" %}
**Paridade CLI**: durante uma execução do `chloros-cli process`, os mesmos quatro threads apresentam os estados «A detetar», «A analisar», Processando, Exportando, e o `chloros-cli export-status` mostra o progresso em tempo real da exportação do Thread-4 a partir de outro terminal. Consulte a [Referência do CLI](../reference/cli-reference.md).
{% endhint %}

***

## Compreender cada fase do processamento

{% hint style="info" %}
**Arquitetura do pipeline**: Estas 4 fases da GUI correspondem ao [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md). Em sistemas com aceleração por GPU, o Thread 3 (Calibração) beneficia da [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md), que otimiza o processamento para o seu hardware específico.
{% endhint %}

### Fase 1: Detecção (Detecção de Alvos)

**O que está a acontecer:**

* O Chloros analisa as imagens que marcou com a caixa de seleção «Alvo» (todas as imagens, apenas quando nenhuma estiver marcada)
* Os algoritmos de visão computacional identificam os painéis de calibração
* Valores de refletância extraídos de cada painel
* Registo dos carimbos de data/hora dos alvos para um agendamento adequado da calibração

**Duração:**

* Com alvos marcados: 10-60 segundos
* Sem alvos marcados: 5-30+ minutos (analisa todas as imagens)

**Indicador de progresso:**

* A detetar: 0% → 100%
* Número de imagens digitalizadas (conta apenas as imagens que estão efetivamente a ser digitalizadas)
* Número de alvos encontrados

**O que ter em atenção:**

* Deve concluir rapidamente se os alvos estiverem devidamente marcados
* Se demorar demasiado tempo, os alvos podem não estar marcados
* Verifique o Registo de Depuração para mensagens do tipo «Alvo encontrado»

### Fase 2: Análise

**O que está a acontecer:**

* Leitura dos metadados EXIF das imagens (marcas temporais, definições de exposição)
* Determinação da estratégia de calibração com base nos carimbos de data/hora dos alvos e nos dados de downwelling do DAQ disponíveis
* Organização da fila de processamento de imagens
* Preparação dos trabalhadores de processamento paralelo (apenas Chloros+)

**Duração:** 5-30 segundos**Indicador de progresso:**

* A analisar: 0% → 100%
* Fase rápida, normalmente conclui-se rapidamente

**A que se deve estar atento:**

* Deve avançar de forma constante, sem pausas
* Serão apresentados avisos sobre metadados em falta no Registo de Depuração

### Fase 3: Calibração

**O que está a acontecer:*** **Debayering**: Conversão do padrão RAW Bayer para 3 canais (ignorada para módulos mono LATTICE, com uma nota)
* **Correção de vinheta**: Remoção do escurecimento nas bordas da lente
* **Calibração de refletância**: Normalização com valores-alvo e/ou downwelling do DAQ
* **Cálculo de índices**: Cálculo de índices multiespectrais
* Processamento de cada imagem ao longo de todo o pipeline

**Duração:** A maior parte do tempo total de processamento (60-80%)**Indicador de progresso:**

* A calibrar: 0% → 100%
* Imagem atual a ser processada
* Imagens concluídas / Total de imagens

**Comportamento do processamento:*** **Modo livre**: Processa uma imagem de cada vez, sequencialmente
* **Modo Chloros+**: Executa um conjunto de trabalhadores adaptável ao hardware — 1 a 4 trabalhadores simultâneos em sistemas com GPU (dependendo da VRAM), um trabalhador por núcleo físico (menos um) em sistemas apenas com CPU. Consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md)
* **Aceleração por GPU**: Acelera significativamente esta fase**O que observar:**

* Progresso constante na contagem de imagens
* Verifique o registo de depuração para mensagens de conclusão por imagem
* Avisos sobre problemas de qualidade de imagem ou de calibração

### Fase 4: Exportação

**O que está a acontecer:**

* Gravação das imagens processadas no disco no formato selecionado, à medida que são concluídas
* **LATTICE**: cada fotograma é distribuído por todos os produtos ativados (debayering / pré-visualização / radiação / refletância)
* Exportação de imagens de índice multiespectral com cores LUT
* Criação da árvore de saída `<project>/<camera>/<format>/<Product>_Images/` — os ficheiros exportados mantêm o nome do ficheiro de origem; a pasta identifica o produto

**Duração:** 10-20% do tempo total de processamento**Indicador de progresso:**

* Exportação: 0% → 100%
* Ficheiros a serem gravados
* Formato de exportação e destino

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de ficheiros
* Conclusão de todas as saídas configuradas

***

## Separador «Registo de Depuração»

O Registo de Depuração fornece informações detalhadas sobre o progresso do processamento e quaisquer problemas encontrados. As mensagens de arranque do backend também são reproduzidas na consola de registo, pelo que o registo apresenta o histórico completo, mesmo que o abra mais tarde.

### Aceder ao Registo de Depuração

1. Clique no ícone **Registo de Depuração**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

na barra lateral esquerda
2. O painel de registo abre-se, apresentando mensagens de processamento em tempo real
3. A rolagem automática mostra as mensagens mais recentes

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Compreender as mensagens de registo

As linhas de registo Chloros são precedidas por etiquetas entre parênteses que identificam o subsistema — por exemplo, `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. A mais importante a conhecer é o **resumo da execução**, apresentado no final de cada execução (incluindo as execuções interrompidas):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

São apresentadas linhas de indicação adicionais **`[RUN-SUMMARY]`**sempre que algo necessite de explicação — por exemplo, uma execução que não produziu nada ou uma câmara cujo produto solicitado foi ignorado por ser inaplicável. As linhas**`[EXPORT-CHECK]`**explicam as omissões por câmara (por exemplo, porque é que uma câmara**RGB** não obteve nenhum produto de radiação).

Os níveis de gravidade gerais das mensagens (os exemplos abaixo são ilustrativos, não literais):

#### Mensagens de Informação (Branco/Cinzento)

Atualizações normais do processamento: processamento iniciado, alvos detetados (com contagem de painéis), progresso da calibração por imagem, ficheiros exportados, processamento concluído.

#### Mensagens de Aviso (Amarelo)

Problemas não críticos que não interrompem o processamento — por exemplo, dados de GPS em falta num fotograma, uma grande diferença de data e hora entre imagens-alvo ou baixo contraste num painel de calibração.

**Ação:** Rever os avisos após o processamento, mas não interromper

#### Mensagens de erro (Red)

Problemas críticos que podem causar a falha do processamento — por exemplo, disco cheio, um ficheiro de imagem corrompido ou ausência de alvos detetados enquanto a calibração de refletância foi solicitada.

**Ação:** Interromper o processamento, resolver o erro e reiniciar

### Situações comuns no registo

| Situação                             | Significado                                       | Ação necessária                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Alvo detetado em \[nome do ficheiro]        | Alvo de calibração encontrado com sucesso         | Nenhuma — normal                                         |
| Linhas de progresso por imagem              | Atualização do progresso atual                       | Nenhuma - normal                                         |
| Nenhum alvo encontrado                      | Não foram detetados alvos de calibração               | Marcar imagens de alvo ou desativar a calibração de refletância |
| Espaço em disco insuficiente               | Armazenamento insuficiente para a saída                 | Libertar espaço em disco                                    |
| A ignorar ficheiro corrompido               | O ficheiro de imagem está danificado                         | Volte a copiar o ficheiro do cartão SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Não é possível processar uma captura sem um fotograma bruto | Volte a capturar com fotograma bruto ou utilize CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | A execução não produziu imagens — relatada como uma falha com sugestões | Leia as linhas de sugestão; verifique o que foi ignorado e porquê |

### Copiar dados de registo

Para copiar o registo para resolução de problemas ou assistência técnica:

1. Abra o painel «Registo de Depuração»
2. Clique no botão **«Copiar Registo»»** (ou clique com o botão direito → Selecionar tudo)
3. Cole num ficheiro de texto ou num e-mail
4. Envie para o suporte MAPIR, se necessário

***

## Monitorização de Recursos do Sistema

### Utilização da CPU

**Modo Livre:**

* 1 núcleo da CPU a ~100%
* Outros núcleos inativos ou disponíveis
* O sistema continua a responder

**Modo Paralelo Chloros+:**

* Vários núcleos com elevada utilização — o número depende da estratégia escolhida pela [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md)
* O sistema pode parecer menos responsivo

**Para monitorizar:**

* Gestor de Tarefas (Ctrl+Shift+Esc)
* Separador «Desempenho» → secção «CPU»
* Procure os processos «Chloros» ou «chloros-backend»

### Memória (RAM) Utilização

**Utilização típica:**

* Projetos pequenos (&lt; 100 imagens): 2-4 GB
* Projetos médios (100-500 imagens): 4-8 GB
* Projetos grandes (mais de 500 imagens): 8-16 GB
* O modo paralelo do Chloros+ consome mais RAM

**Se a memória estiver baixa:**

* Processe lotes mais pequenos
* Feche outras aplicações
* Aumente a memória RAM se processar regularmente conjuntos de dados de grande dimensão

### Utilização da GPU (Chloros+ com CUDA)

Quando a aceleração por GPU está ativada:

* A GPU NVIDIA apresenta uma elevada utilização (60-90%)
* A utilização de VRAM aumenta (requer 4 GB ou mais de VRAM; 7 GB ou mais para debayering «Texture Aware» simultâneo)
* A fase de calibração é significativamente mais rápida

**Para monitorizar:**

* Ícone da NVIDIA na bandeja do sistema
* Gestor de Tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitorização semelhante

### E/S do disco

**O que esperar:**

* Elevada leitura do disco durante a fase de análise
* Elevada gravação no disco durante a fase de exportação
* O SSD é significativamente mais rápido do que o HDD

**Dica de desempenho:**

* Utilize um SSD para a pasta do projeto, sempre que possível
* Evite unidades de rede para conjuntos de dados de grande dimensão
* Certifique-se de que o disco não está quase cheio (afeta a velocidade de escrita)

***

## Detetar problemas durante o processamento

### Sinais de aviso

**O progresso fica estagnado (sem alterações durante mais de 5 minutos):**

* Verifique o registo de depuração para detetar erros
* Verifique o espaço disponível no disco
* Verifique o Gestor de Tarefas para garantir que o Chloros está a ser executado

**Aparecem mensagens de erro com frequência:**

* Interrompa o processamento e analise os erros
* Causas comuns: espaço no disco, ficheiros corrompidos, problemas de memória
* Consulte a secção «Resolução de problemas» abaixo

**O sistema deixa de responder:**

* O modo paralelo do Chloros+ está a utilizar demasiados recursos
* Considere reduzir o número de tarefas simultâneas ou atualizar o hardware
* O modo livre consome menos recursos

### Quando interromper o processamento

Interrompa o processamento se observar:

* ❌ Erros do tipo «Disco cheio» ou «Não é possível gravar o ficheiro»
* ❌ Erros repetidos de corrupção de ficheiros de imagem
* ❌ Sistema completamente bloqueado (não responde)
* ❌ Percebeu que foram definidas configurações erradas
* ❌ Imagens erradas importadas

**Como interromper:**

1. Clique no**botão «Parar»** (que substitui o botão «Iniciar») — basta uma vez
2. A barra exibe «A parar...» enquanto a imagem em processamento é concluída; em seguida, a execução termina num estado de paragem
3. Os produtos já exportados permanecem no disco; o registo apresenta um código `[RUN-SUMMARY]` que indica o que foi concluído
4. Resolva os problemas e reinicie — a execução recomeça do início

***

## Resolução de problemas durante o processamento

### O processamento está muito lento

**Possíveis causas:**

* Imagens-alvo não marcadas (digitalização de todas as imagens)
* Armazenamento em HDD em vez de SSD
* Recursos do sistema insuficientes
* Muitos índices configurados
* Acesso a unidades de rede

**Soluções:**

1. Se acabou de iniciar e está na fase de deteção: interrompa, marque os alvos e reinicie
2. Para o futuro: utilize um SSD, reduza os índices e atualize o hardware
3. Considere o CLI para o processamento em lote de grandes conjuntos de dados

### Avisos de «Espaço em disco»

**Soluções:**

1. Liberte espaço em disco imediatamente
2. Mude o projeto para uma unidade com mais espaço
3. Reduza o número de índices a exportar
4. Desative os produtos de exportação do LATTICE de que não necessita (Definições do projeto → Processamento)
5. Utilize o formato JPG em vez de TIFF (ficheiros mais pequenos)

### Mensagens frequentes de «Ficheiro corrompido»

**Soluções:**

1. Volte a copiar as imagens do cartão SD para garantir a integridade
2. Verifique se o cartão SD apresenta erros
3. Remova os ficheiros corrompidos do projeto
4. Continue a processar as imagens restantes

### Sobreaquecimento / Limitação do sistema

**Soluções:**

1. Assegure uma ventilação adequada
2. Limpe o pó das aberturas de ventilação do computador
3. Reduza a carga de processamento (utilize o modo «Free» em vez de Chloros+)
4. Processe durante os períodos mais frescos do dia

***

## Notificação de processamento concluído

Quando o processamento terminar:

* A barra de progresso atinge os 100%
* As linhas `[RUN-SUMMARY]` aparecem no Registo de Depuração com as contagens finais
* O botão «Iniciar» volta a ficar ativo
* Todos os ficheiros de saída encontram-se na árvore de saída por câmara do projeto: `<project>/<camera>/<format>/<Product>_Images/`

***

## Próximos passos

Assim que o processamento estiver concluído:

1. **Analise os resultados** - Consulte [Concluir o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** — Verifique se todos os ficheiros foram exportados corretamente
3. **Analise o registo de depuração** — Verifique se existem avisos ou erros
4. **Pré-visualizar as imagens processadas** — Utilize o Visualizador de Imagens ou software externo

Para obter informações sobre como rever e utilizar os resultados processados, consulte [Concluir o processamento](finishing-the-processing.md).
