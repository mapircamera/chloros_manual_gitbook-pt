# Monitorização do processamento

Depois de iniciado o processamento, o Chloros oferece várias maneiras de monitorizar o progresso, verificar se há problemas e compreender o que está a acontecer com o seu conjunto de dados. Esta página explica como acompanhar o seu processamento e interpretar as informações fornecidas pelo Chloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o estado do processamento em tempo real e a percentagem de conclusão.

### Barra de progresso do modo gratuito

Para utilizadores sem licença do Chloros+:

**Exibição do progresso em duas etapas:**

1. **Detecção do alvo** - Encontrar alvos de calibração nas imagens
2. **Processamento** - Aplicar correções e exportar

**A barra de progresso mostra:**

* Percentagem de conclusão geral (0-100%)
* Nome da etapa atual
* Visualização simples em barra horizontal

### Barra de progresso do Chloros+

Para utilizadores com licença Chloros+:

**Exibição de progresso em 4 etapas:**

1. **Detecção** - Localização de alvos de calibração
2. **Análise** - Exame de imagens e preparação do pipeline
3. **Calibração** - Aplicação de correções de vinheta e refletância
4. **Exportação** - Salvamento de ficheiros processados

**Recursos interativos:**

* **Passe o cursor sobre** a barra de progresso para ver o painel expandido de 4 etapas
* **Clique** na barra de progresso para congelar/fixar o painel expandido
* **Clique novamente** para descongelar e ocultar automaticamente ao sair com o mouse
* Cada etapa mostra o progresso individual (0-100%)

***

## Entendendo cada etapa do processamento

### Etapa 1: Detecção (detecção de alvo)

**O que está a acontecer:**

* O Chloros digitaliza imagens marcadas com a caixa de seleção Alvo
* Algoritmos de visão computacional identificam os 4 painéis de calibração
* Valores de refletância extraídos de cada painel
* Registos de data e hora dos alvos para programação adequada da calibração

**Duração:**

* Com alvos marcados: 10-60 segundos
* Sem alvos marcados: 5-30+ minutos (digitaliza todas as imagens)

**Indicador de progresso:**

* Detecção: 0% → 100%
* Número de imagens digitalizadas
* Contagem de alvos encontrados

**O que observar:**

* Deve ser concluído rapidamente se os alvos estiverem devidamente marcados
* Se demorar muito, os alvos podem não estar marcados
* Verifique o registo de depuração para mensagens &quot;Alvo encontrado&quot;

### Fase 2: Análise

**O que está a acontecer:**

* Leitura dos metadados EXIF da imagem (carimbos de data/hora, definições de exposição)
* Determinação da estratégia de calibração com base nos carimbos de data/hora dos alvos
* Organização da fila de processamento de imagens
* Preparação dos trabalhadores de processamento paralelo (apenas Chloros+)

**Duração:** 5-30 segundos

**Indicador de progresso:**

* A analisar: 0% → 100%
* Fase rápida, geralmente concluída rapidamente

**O que observar:**

* Deve progredir de forma constante, sem pausas
* Avisos sobre metadados em falta aparecerão no registo de depuração

### Fase 3: Calibração

**O que está a acontecer:**

* **Debayering**: Conversão do padrão RAW Bayer para 3 canais
* **Correção de vinheta**: Remoção do escurecimento das bordas da lente
* **Calibração de refletância**: Normalização com valores-alvo
* **Cálculo de índice**: Cálculo de índices multiespectrais
* Processamento de cada imagem através do pipeline completo

**Duração:** Maioria do tempo total de processamento (60-80%)

**Indicador de progresso:**

* Calibração: 0% → 100%
* Imagem atual a ser processada
* Imagens concluídas / Total de imagens

**Comportamento do processamento:**

* **Modo livre**: Processa uma imagem de cada vez sequencialmente
* **Modo Chloros+: Processa até 16 imagens simultaneamente
* **Aceleração GPU**: Acelera significativamente esta etapa

**O que observar:**

* Progresso constante através da contagem de imagens
* Verifique o Log de Depuração para mensagens de conclusão por imagem
* Avisos sobre qualidade de imagem ou problemas de calibração

### Etapa 4: Exportação

**O que está a acontecer:**

* Gravação de imagens calibradas no disco no formato selecionado
* Exportação de imagens de índice multiespectral com cores LUT
* Criação de subpastas do modelo da câmara
* Preservação dos nomes de ficheiros originais com sufixos apropriados

**Duração:** 10-20% do tempo total de processamento

**Indicador de progresso:**

* Exportação: 0% → 100%
* Ficheiros a serem gravados
* Formato e destino da exportação

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de ficheiros
* Conclusão de todas as saídas configuradas

***

## Guia Log de depuração

O Log de depuração fornece informações detalhadas sobre o progresso do processamento e quaisquer problemas encontrados.

### Acessando o Log de depuração

1. Clique no ícone **Log de depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na barra lateral esquerda
2. O painel de registo abre, mostrando mensagens de processamento em tempo real
3. Rola automaticamente para mostrar as mensagens mais recentes

### Compreender as mensagens de registo

#### Mensagens informativas (brancas/cinzentas)

Atualizações normais de processamento:

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

**Ação:** Revise os avisos após o processamento, mas não interrompa

#### Mensagens de erro (Red)

Problemas críticos que podem causar falha no processamento:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ação:** Interrompa o processamento, resolva o erro e reinicie.

### Mensagens comuns do registo

| Mensagem                          | Significado                                | Ação necessária                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| «Alvo detetado em \[nome do ficheiro]» | Alvo de calibração encontrado com sucesso  | Nenhuma - normal                                         |
| «Processando imagem X de Y»        | Atualização do progresso atual                | Nenhuma - normal                                         |
| «Nenhum alvo encontrado»               | Nenhum alvo de calibração detetado        | Marque as imagens-alvo ou desative a calibração de refletância |
| «Espaço em disco insuficiente»        | Armazenamento insuficiente para saída          | Liberar espaço em disco                                    |
| «Ignorando ficheiro corrompido»        | Ficheiro de imagem danificado                  | Recopiar ficheiro do cartão SD                             |
| «Dados PPK aplicados»               | Correções GPS do ficheiro .daq aplicadas | Nenhuma - normal                                         |

### Copiando dados de log

Para copiar o log para solução de problemas ou suporte:

1. Abra o painel Debug Log (Log de depuração)
2. Clique no botão **&quot;Copy Log&quot;** (Copiar log) (ou clique com o botão direito do mouse → Selecionar tudo)
3. Cole em um arquivo de texto ou e-mail
4. Envie para o suporte MAPIR, se necessário

***

## Monitoramento de recursos do sistema

### Utilização da CPU

**Modo livre:**

* 1 núcleo da CPU a ~100%
* Outros núcleos inativos ou disponíveis
* O sistema permanece responsivo

**Chloros+ Modo paralelo:**

* Vários núcleos a 80-100% (até 16 núcleos)
* Alta utilização geral da CPU
* O sistema pode parecer menos responsivo

**Para monitorizar:**

* Windows Gestor de tarefas (Ctrl+Shift+Esc)
* Separador Desempenho → secção CPU
* Procure os processos «Chloros» ou «chloros-backend»

### Utilização da memória (RAM)

**Utilização típica:**

* Projetos pequenos (&lt; 100 imagens): 2-4 GB
* Projetos médios (100-500 imagens): 4-8 GB
* Projetos grandes (mais de 500 imagens): 8-16 GB
* O modo paralelo Chloros+ utiliza mais RAM

**Se a memória estiver baixa:**

* Processe lotes menores
* Feche outras aplicações
* Atualize a RAM se processar regularmente grandes conjuntos de dados

### Utilização da GPU (Chloros+ com CUDA)

Quando a aceleração da GPU está ativada:

* A GPU NVIDIA apresenta alta utilização (60-90%)
* O uso da VRAM aumenta (requer 4 GB+ de VRAM)
* A fase de calibração é significativamente mais rápida

**Para monitorar:**

* Ícone da bandeja do sistema NVIDIA
* Gestor de tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitoramento semelhante

### E/S do disco

**O que esperar:**

* Alta leitura do disco durante a fase de análise
* Alta gravação do disco durante a fase de exportação
* SSD significativamente mais rápido que HDD

**Dica de desempenho:**

* Use SSD para a pasta do projeto, quando possível
* Evite unidades de rede para grandes conjuntos de dados
* Certifique-se de que o disco não esteja perto da capacidade máxima (afeta a velocidade de gravação)

***

## Detecção de problemas durante o processamento

### Sinais de alerta

**Progresso parado (sem alterações por mais de 5 minutos):**

* Verifique se há erros no registo de depuração
* Verifique o espaço disponível em disco
* Verifique o Gestor de Tarefas para garantir que o Chloros está em execução

**Mensagens de erro aparecem com frequência:**

* Interrompa o processamento e analise os erros
* Causas comuns: espaço em disco, ficheiros corrompidos, problemas de memória
* Consulte a secção Resolução de problemas abaixo

**O sistema deixa de responder:**

* O modo paralelo Chloros+ está a utilizar demasiados recursos
* Considere reduzir as tarefas simultâneas ou atualizar o hardware
* O modo livre consome menos recursos

### Quando interromper o processamento

Interrompa o processamento se observar:

* ❌ Erros «Disco cheio» ou «Não é possível gravar o ficheiro»
* ❌ Erros repetidos de corrupção de ficheiros de imagem
* ❌ Sistema completamente congelado (não responde)
* ❌ Percebeu que foram configuradas definições erradas
* ❌ Imagens erradas importadas

**Como parar:**

1. Clique no **botão Parar/Cancelar** (substitui o botão Iniciar)
2. O processamento é interrompido e o progresso é perdido
3. Corrija os problemas e reinicie desde o início

***

## Resolução de problemas durante o processamento

### O processamento é muito lento

**Possíveis causas:**

* Imagens de destino não marcadas (digitalização de todas as imagens)
* Armazenamento em HDD em vez de SSD
* Recursos do sistema insuficientes
* Muitos índices configurados
* Acesso à unidade de rede

**Soluções:**

1. Se acabou de começar e está na fase de deteção: cancele, marque os alvos, reinicie
2. Para o futuro: use SSD, reduza os índices, atualize o hardware
3. Considere o CLI para processamento em lote de grandes conjuntos de dados

### Avisos de «Espaço em disco»

**Soluções:**

1. Liberte espaço em disco imediatamente
2. Mova o projeto para uma unidade com mais espaço
3. Reduza o número de índices a exportar
4. Use o formato JPG em vez de TIFF (ficheiros menores)

### Mensagens frequentes de «ficheiro corrompido»

**Soluções:**

1. Recopiar as imagens do cartão SD para garantir a integridade
2. Testar o cartão SD para verificar se há erros
3. Remover os ficheiros corrompidos do projeto
4. Continuar a processar as imagens restantes

### Sobreaquecimento/limitação do sistema

**Soluções:**

1. Garanta ventilação adequada
2. Limpe o pó das aberturas de ventilação do computador
3. Reduza a carga de processamento (use o modo Livre em vez de Chloros+)
4. Processe durante os períodos mais frios do dia

***

## Notificação de processamento concluído

Quando o processamento terminar:

* A barra de progresso atinge 100%
* A mensagem **«Processamento concluído»** aparece no registo de depuração
* O botão Iniciar fica novamente ativado
* Todos os ficheiros de saída estão na subpasta do modelo da câmara

***

## Próximos passos

Quando o processamento estiver concluído:

1. **Revise os resultados** - Consulte [Concluindo o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** - Verifique se todos os ficheiros foram exportados corretamente
3. **Revise o registo de depuração** - Verifique se há avisos ou erros
4. **Visualize as imagens processadas** - Use o Visualizador de Imagens ou um software externo

Para obter informações sobre como revisar e usar os resultados processados, consulte [Concluindo o processamento](finishing-the-processing.md).
