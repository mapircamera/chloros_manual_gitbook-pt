# Monitorização do processamento

Assim que o processamento tiver início, o Chloros oferece várias formas de monitorizar o progresso, verificar se existem problemas e compreender o que está a acontecer com o seu conjunto de dados. Esta página explica como acompanhar o seu processamento e interpretar as informações fornecidas pelo Chloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o estado do processamento em tempo real e a percentagem de conclusão.

### Barra de progresso do modo gratuito

Para utilizadores sem licença Chloros+:

**Exibição do progresso em 2 fases:**

1.**Detecção de alvos** - Localização de alvos de calibração nas imagens
2. **Processamento** - Aplicação de correções e exportação**A barra de progresso mostra:**

* Percentagem de conclusão total (0-100%)
* Nome da fase atual
* Visualização simples com barra horizontal

### Barra de progresso do Chloros+

Para utilizadores com licença Chloros+:

**Exibição de progresso em 4 fases:**

1.**Detecção** - Localização de alvos de calibração
2. **Análise** - Exame das imagens e preparação do fluxo de trabalho
3. **Calibração** - Aplicação de correções de vinheta e refletância
4. **Exportação** - Gravação dos ficheiros processados**Funcionalidades interativas:*** **Passe o cursor** sobre a barra de progresso para ver o painel expandido de 4 fases
* **Clique** na barra de progresso para fixar o painel expandido
* **Clique novamente** para desbloquear e ocultar automaticamente ao afastar o cursor
* Cada fase mostra o progresso individual (0-100%)

***

## Compreender cada fase de processamento

{% hint style="info" %}
**Arquitetura do pipeline**: Estas 4 fases da GUI correspondem ao [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md). Em sistemas com aceleração por GPU, o Thread 3 (Calibração) beneficia da [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md), que otimiza o processamento para o seu hardware específico.
{% endhint %}

### Fase 1: Detecção (Detecção de Alvos)

**O que está a acontecer:**

* Chloros analisa imagens marcadas com a caixa de seleção Alvo
* Algoritmos de visão computacional identificam os 4 painéis de calibração
* Valores de refletância extraídos de cada painel
* Registo de carimbos de data/hora dos alvos para um agendamento adequado da calibração

**Duração:**

* Com alvos marcados: 10-60 segundos
* Sem alvos marcados: 5-30+ minutos (analisa todas as imagens)

**Indicador de progresso:**

* Detecção: 0% → 100%
* Número de imagens analisadas
* Contagem de alvos encontrados

**O que observar:**

* Deve concluir rapidamente se os alvos estiverem devidamente marcados
* Se demorar demasiado tempo, os alvos podem não estar marcados
* Verifique o Registo de Depuração para mensagens &quot;Alvo encontrado&quot;

### Fase 2: Análise

**O que está a acontecer:**

* Leitura dos metadados EXIF da imagem (marcas temporais, definições de exposição)
* Determinação da estratégia de calibração com base nos carimbos de data/hora dos alvos
* Organização da fila de processamento de imagens
* Preparação de trabalhadores de processamento paralelo (apenas Chloros+)

**Duração:** 5-30 segundos**Indicador de progresso:**

* Análise: 0% → 100%
* Fase rápida, geralmente conclui-se rapidamente

**O que observar:**

* Deve progredir de forma constante, sem pausas
* Avisos sobre metadados em falta aparecerão no Registo de Depuração

### Fase 3: Calibração

**O que está a acontecer:*** **Debayering**: Conversão do padrão RAW Bayer para 3 canais
* **Correção de vinheta**: Remoção do escurecimento nas bordas da lente
* **Calibração de refletância**: Normalização com valores-alvo
* **Cálculo de índices**: Cálculo de índices multiespectrais
* Processamento de cada imagem através de todo o pipeline

**Duração:** A maior parte do tempo total de processamento (60-80%)**Indicador de progresso:**

* Calibrando: 0% → 100%
* Imagem atual a ser processada
* Imagens concluídas / Total de imagens

**Comportamento do processamento:*** **Modo livre**: Processa uma imagem de cada vez, sequencialmente
* **Modo Chloros+**: Processa até 16 imagens simultaneamente
* **Aceleração por GPU**: Acelera significativamente esta fase**O que observar:**

* Progresso constante na contagem de imagens
* Verifique o Registo de Depuração para mensagens de conclusão por imagem
* Avisos sobre qualidade de imagem ou problemas de calibração

### Fase 4: Exportação

**O que está a acontecer:**

* Gravação de imagens calibradas no disco no formato selecionado
* Exportação de imagens de índice multiespectral com cores LUT
* Criação de subpastas de modelos de câmara
* Preservação dos nomes de ficheiros originais com sufixos apropriados

**Duração:** 10-20% do tempo total de processamento**Indicador de progresso:**

* Exportação: 0% → 100%
* Ficheiros a serem gravados
* Formato de exportação e destino

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de ficheiros
* Conclusão de todas as saídas configuradas

***

## Separador Registo de Depuração

O Registo de Depuração fornece informações detalhadas sobre o progresso do processamento e quaisquer problemas encontrados.

### Aceder ao Registo de Depuração

1. Clique no ícone **Registo de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na barra lateral esquerda
2. O painel de registo abre-se, apresentando mensagens de processamento em tempo real
3. Desloca-se automaticamente para mostrar as mensagens mais recentes

### Compreender as mensagens de registo

#### Mensagens de informação (brancas/cinzentas)

Atualizações normais do processamento:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mensagens de aviso (amarelas)

Problemas não críticos que não interrompem o processamento:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Ação:** Analise os avisos após o processamento, mas não interrompa

#### Mensagens de erro (Red)

Problemas críticos que podem causar falha no processamento:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ação:** Interrompa o processamento, resolva o erro e reinicie

### Mensagens comuns de registo

| Mensagem                          | Significado                                | Ação necessária                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Alvo detetado em \[nome do ficheiro]&quot; | Alvo de calibração encontrado com sucesso  | Nenhuma - normal                                         |
| &quot;A processar imagem X de Y&quot;        | Atualização do progresso atual                | Nenhuma - normal                                         |
| &quot;Não foram encontrados alvos&quot;               | Não foram detetados alvos de calibração        | Marcar imagens de alvo ou desativar a calibração de refletância |
| &quot;Espaço em disco insuficiente&quot;        | Armazenamento insuficiente para saída          | Liberte espaço em disco                                    |
| &quot;Ignorando ficheiro corrompido&quot;        | O ficheiro de imagem está danificado                  | Volte a copiar o ficheiro do cartão SD                             |
| &quot;Dados PPK aplicados&quot;               | Correções de GPS do ficheiro .daq aplicadas | Nenhum - normal                                         |

### Copiar dados de registo

Para copiar o registo para resolução de problemas ou assistência:

1. Abra o painel de registo de depuração
2. Clique no botão **&quot;Copiar registo&quot;** (ou clique com o botão direito → Selecionar tudo)
3. Cole num ficheiro de texto ou e-mail
4. Envie para o suporte MAPIR, se necessário

***

## Monitorização de recursos do sistema

### Utilização da CPU

**Modo Livre:**

* 1 núcleo da CPU a ~100%
* Outros núcleos inativos ou disponíveis
* O sistema continua a responder

**Modo Paralelo Chloros+:**

* Vários núcleos a 80-100% (até 16 núcleos)
* Elevada utilização geral da CPU
* O sistema pode parecer menos responsivo

**Para monitorizar:**

* Gestor de Tarefas (Ctrl+Shift+Esc)
* Separador Desempenho → secção CPU
* Procure os processos &quot;Chloros&quot; ou &quot;chloros-backend&quot;

### Utilização da memória (RAM)

**Utilização típica:**

* Projetos pequenos (&lt; 100 imagens): 2-4 GB
* Projetos médios (100-500 imagens): 4-8 GB
* Projetos grandes (mais de 500 imagens): 8-16 GB
* O modo paralelo do Chloros+ utiliza mais RAM

**Se a memória estiver baixa:**

* Processe lotes mais pequenos
* Feche outras aplicações
* Atualize a RAM se processar regularmente grandes conjuntos de dados

### Utilização da GPU (Chloros+ com CUDA)

Quando a aceleração da GPU está ativada:

* A GPU NVIDIA apresenta uma elevada utilização (60-90%)
* O uso de VRAM aumenta (requer 4 GB ou mais de VRAM)
* A fase de calibração é significativamente mais rápida

**Para monitorizar:**

* Ícone da NVIDIA na bandeja do sistema
* Gestor de Tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitorização semelhante

### E/S do disco

**O que esperar:**

* Elevada leitura do disco durante a fase de análise
* Elevada gravação do disco durante a fase de exportação
* SSD significativamente mais rápido do que HDD

**Dica de desempenho:**

* Utilize SSD para a pasta do projeto, sempre que possível
* Evite unidades de rede para conjuntos de dados de grande dimensão
* Certifique-se de que o disco não está quase cheio (afeta a velocidade de gravação)

***

## Detecção de problemas durante o processamento

### Sinais de aviso

**O progresso fica parado (sem alterações durante mais de 5 minutos):**

* Verifique o Registo de Depuração para ver se há erros
* Verifique o espaço disponível no disco
* Verifique o Gestor de Tarefas para garantir que o Chloros está a ser executado

**Aparecem mensagens de erro com frequência:**

* Interrompa o processamento e analise os erros
* Causas comuns: espaço no disco, ficheiros corrompidos, problemas de memória
* Consulte a secção Resolução de problemas abaixo

**O sistema deixa de responder:**

* O modo paralelo do Chloros+ está a utilizar demasiados recursos
* Considere reduzir as tarefas simultâneas ou atualizar o hardware
* O modo livre consome menos recursos

### Quando interromper o processamento

Interrompa o processamento se observar:

* ❌ Erros «Disco cheio» ou «Não é possível gravar o ficheiro»
* ❌ Erros repetidos de corrupção de ficheiros de imagem
* ❌ Sistema completamente bloqueado (não responde)
* ❌ Percebeu que foram configuradas definições erradas
* ❌ Imagens erradas importadas

**Como parar:**

1. Clique no**botão Parar/Cancelar** (substitui o botão Iniciar)
2. O processamento é interrompido, o progresso é perdido
3. Resolva os problemas e reinicie desde o início

***

## Resolução de problemas durante o processamento

### O processamento está muito lento

**Possíveis causas:**

* Imagens de destino não marcadas (digitalização de todas as imagens)
* Armazenamento em HDD em vez de SSD
* Recursos do sistema insuficientes
* Muitos índices configurados
* Acesso a unidade de rede

**Soluções:**

1. Se acabou de iniciar e está na fase de deteção: Cancelar, marcar alvos, reiniciar
2. Para o futuro: Utilizar SSD, reduzir índices, atualizar hardware
3. Considerar o CLI para o processamento em lote de grandes conjuntos de dados

### Avisos de «Espaço em disco»

**Soluções:**

1. Liberte espaço em disco imediatamente
2. Mova o projeto para uma unidade com mais espaço
3. Reduzir o número de índices a exportar
4. Utilizar o formato JPG em vez de TIFF (ficheiros mais pequenos)

### Mensagens frequentes de &quot;Ficheiro corrompido&quot;

**Soluções:**

1. Voltar a copiar as imagens do cartão SD para garantir a integridade
2. Testar o cartão SD para detetar erros
3. Remover ficheiros corrompidos do projeto
4. Continuar a processar as imagens restantes

### Sobreaquecimento / Limitação do sistema

**Soluções:**

1. Assegure uma ventilação adequada
2. Limpe o pó das aberturas de ventilação do computador
3. Reduza a carga de processamento (utilize o modo Free em vez de Chloros+)
4. Processe durante os períodos mais frescos do dia

***

## Notificação de processamento concluído

Quando o processamento terminar:

* A barra de progresso atinge 100%
* A mensagem **«Processamento concluído»** aparece no Registo de Depuração
* O botão Iniciar fica novamente ativado
* Todos os ficheiros de saída estão na subpasta do modelo da câmara

***

## Próximos passos

Assim que o processamento estiver concluído:

1. **Analise os resultados** - Consulte [Concluir o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** - Verifique se todos os ficheiros foram exportados corretamente
3. **Analise o Registo de Depuração** - Verifique se existem avisos ou erros
4. **Pré-visualize as imagens processadas** - Utilize o Visualizador de Imagens ou software externo

Para obter informações sobre como analisar e utilizar os resultados processados, consulte [Concluir o Processamento](finishing-the-processing.md).
