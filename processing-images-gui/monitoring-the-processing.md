# Monitorando o Processamento

Após o início do processamento, o Chloros oferece diversas maneiras de monitorar o progresso, verificar problemas e entender o que está acontecendo com seu conjunto de dados. Esta página explica como rastrear seu processamento e interpretar as informações fornecidas pela Cloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o status do processamento em tempo real e a porcentagem de conclusão.

### Barra de progresso do modo livre

Para usuários sem licença Chloros+:

**Exibição de progresso em 2 estágios:**

1. **Target Detect** - Encontrando alvos encontrados em imagens
2. **Processando** - Aplicando correções e exportando

**A barra de progresso mostra:**

* Porcentagem geral de conclusão (0-100%)
* Nome artístico atual
* Visualização simples da barra horizontal

### Barra de Progresso Cloros+

Para usuários com licença Cloros+:

**Exibição de progresso em 4 estágios:**

1. **Detectando** - Encontrando alvos específicos
2. **Analisando** - Examinando imagens e preparando pipeline
3. **Calibrando** - Aplicando correções de vinheta e refletância
4. **Exportando** - Salvando arquivos processados

**Recursos interativos:**

* **Passe o mouse sobre** a barra de progresso para ver o painel expandido de quatro estágios
* **Clique** na barra de progresso para congelar/fixar o painel expandido
* **Clique novamente** para descongelar e ocultar automaticamente ao sair do mouse
* Cada estágio mostra o progresso individual (0-100%)

***

## Compreendendo cada etapa do processamento

### Estágio 1: Detecção (Detecção de Alvo)

**O que está acontecendo:**

* O Cloros verifica imagens marcadas com a caixa de seleção Target
* Algoritmos de visão computacional identificam os 4 painéis defeituosos
* Valores de refletância extraídos de cada painel
* Carimbos de dados e hora alvo registrados para agendamento de agendamento adequado

**Duração:**

* Com alvos marcados: 10-60 segundos
* Sem alvos marcados: 5-30+ minutos (verifique todas as imagens)

**Indicador de progresso:**

* Detecção: 0% → 100%
* Número de imagens digitalizadas
* Contagem de alvos encontrados

**O que observar:**

* Deve ser concluído rapidamente se os alvos forem devidamente marcados
* Se demorar muito, os alvos podem não ser marcados
* Verifique o log de depuração para mensagens "Destino encontrado"

### Etapa 2: Análise

**O que está acontecendo:**

* Lendo metadados EXIF​​de imagem (carimbos de dados e hora, configurações de exposição)
* Determinando a estratégia de privacidade com base em carimbos de dados/hora alvo
* Organizando fila de processamento de imagens
* Preparando trabalhadores de processamento paralelo (somente Cloros+)

**Duração:** 5 a 30 segundos

**Indicador de progresso:**

* Analisando: 0% → 100%
* Estágio rápido, geralmente concluído rapidamente

**O que observar:**

* Deve progredir de forma constante, sem pausas
* Avisos sobre metadados ausentes aparecem no log de depuração

### Etapa 3: Calibração

**O que está acontecendo:**

* **Debayering**: Convertendo o padrão RAW Bayer em 3 canais
* **Correção de vinheta**: remoção do escurecimento das bordas da lente
* **Calibração de refletância**: Normalização com valores alvo
* **Cálculo de índice**: Cálculo de índices multiespectrais
* Processando cada imagem por meio do pipeline completo

**Duração:** Maioria do tempo total de processamento (60-80%)

**Indicador de progresso:**

* Calibrando: 0% → 100%
* Imagem atual sendo processada
* Imagens concluídas / Total de imagens

**Comportamento de processamento:**

* **Modo livre**: processa uma imagem por vez sequencialmente
* **Modo Cloros+**: Processo até 16 imagens simultaneamente
* **Aceleração de GPU**: Aceleração significativamente esse estágio

**O que observar:**

* Progresso constante através da contagem de imagens
* Verifique o log de depuração para mensagens de conclusão por imagem
* Avisos sobre problemas de qualidade de imagem ou sintonia

### Etapa 4: Exportação

**O que está acontecendo:**

* Gravando imagens calibradas em disco no formato selecionado
* Exportando imagens de índice multiespectral com núcleos LUT
* Criando subpastas de modelo de câmera
* Preservando nomes de arquivos originais com sufixos protegidos

**Duração:** 10-20% do tempo total de processamento

**Indicador de progresso:**

* Exportando: 0% → 100%
* Arquivos sendo gravados
* Formato de exportação e destino

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de arquivo
* Conclusão de todas as saídas definidas

***

## Guia Log de depuração

O log de detalhamento fornece informações planejadas sobre o andamento do processamento e quaisquer problemas encontrados.

### Acessando o log de depuração

1. Clique no ícone **Log de depuração**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">na barra lateral esquerda
2. O painel de registro é aberto mostrando mensagens de processamento em tempo real
3. Rola automaticamente para mostrar as mensagens mais recentes

### Compreendendo as mensagens de log

#### Mensagens informativas (branco/cinza)

Atualizações normais de processamento:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mensagens de aviso (amarelo)

Problemas não críticos que não param de ser processados:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Ação:** Revisar os avisos após o processamento, mas não interrompa

#### Mensagens de erro (vermelho)

Problemas críticos que podem causar falha no processamento:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ação:** Interrompa o processamento, resolva o erro e reinicie

### Mensagens de registro comuns

| Mensagem | Significado | Ação necessária |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| "Alvo detectado em \[nome do arquivo]" | Alvo de encontrado com sucesso | Nenhum - normal |
| "Processando imagem X de Y" | Atualização do progresso atual | Nenhum - normal |
| "Nenhum alvo encontrado" | Não foram detectados alvos de descoberta | Marcar imagens alvo ou desativar a refletância |
| “Espaço em disco insuficiente” | Armazenamento insuficiente para saída | Espaço livre em discoteca |
| "Ignorando arquivo corrompido" | O arquivo de imagem está danificado | Copie novamente o arquivo do cartão SD |
| “Dados PPK aplicados” | Correções GPS do arquivo .daq aplicadas | Nenhum - normal |

### Copiando dados de registro

Para copiar o log para solução de problemas ou suporte:

1. Abra o painel Log de depuração
2. Clique no botão **"Copiar Log"** (ou clique com o botão direito → Selecionar tudo)
3. Cole em arquivo de texto ou e-mail
4. Envie para o suporte do MAPIR, se necessário

***

## Monitoramento de recursos do sistema

### Uso da CPU

**Modo Gratuito:**

* 1 núcleo de CPU em \~100%
* Outros núcleos ociosos ou disponíveis
* O sistema permanece responsivo

**Modo Cloros+ Paralelo:**

* Vários núcleos em 80-100% (até 16 núcleos)
* Alta utilização geral da CPU
* O sistema pode parecer menos responsivo

**Para monitorar:**

* Gerenciador de Tarefas do Windows (Ctrl+Shift+Esc)
* Guia Desempenho → seção CPU
* Procure por processos "Cloros" ou "cloros-backend"

### Uso de memória (RAM)

**Uso típico:**

* Projetos pequenos (<100 imagens): 2-4 GB
* Projetos médios (100-500 imagens): 4-8 GB
* Projetos grandes (mais de 500 imagens): 8-16 GB
* O modo paralelo Chloros+ usa mais RAM

**Se a memória estiver baixa:**

* Processar lotes menores
* Buscar outros aplicativos
* Atualizar RAM para registrar regularmente grandes conjuntos de dados

### Uso de GPU (Chloros+ com CUDA)

Quando a aceleração da GPU está habilitada:

* GPU NVIDIA mostra alta utilização (60-90%)
* O uso de VRAM aumenta (requer 4 GB + VRAM)
* O estágio de desligamento é significativamente mais rápido

**Para monitorar:**

* Ícone da bandeja do sistema NVIDIA
* Gerenciador de Tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitoramento semelhante

### E/S de discoteca

**O que esperar:**

* Alta leitura do disco durante a etapa de análise
* Alta gravação em disco durante o estágio de exportação
* SSD significativamente mais rápido que HDD

**Dica de desempenho:**

* Use SSD para pasta do projeto quando possível
* Evite unidades de rede para grandes conjuntos de dados
* -se de que o disco não está próximo da capacidade máxima (afeta a velocidade de gravação)

***

## Detectando problemas durante o processamento

### Sinais de alerta

**Progresso paralisado (sem alteração por mais de 5 minutos):**

* Verifique o log de depuração em busca de erros
* Confira o espaço em disco disponível
* Verifique o Gerenciador de Tarefas para garantir que o Chloros esteja funcionando

**Mensagens de erro aparecem com frequência:**

* Pare o processamento e revise os erros
* Causas comuns: espaço em disco, arquivos corrompidos, problemas de memória
* Consulte a seção Solução de problemas abaixo

**O sistema não responde:**

* Modo paralelo Cloros+ usando muitos recursos
* Considere reduzir tarefas simultâneas ou atualizar hardware
* O modo gratuito consome menos recursos

### Quando parar o processamento

Pare o processamento se você vir:

* ❌ Erros "Disco cheio" ou "Não é possível gravar arquivo"
* ❌ Erros repetidos de corrupção de arquivos de imagem
* ❌ Sistema completamente congelado (não responde)
* ❌ Percebi que configurações erradas foram definidas
* ❌ Imagens erradas importadas

**Como parar:**

1. Clique no **botão Parar/Cancelar** (substitua o botão Iniciar)
2. O processamento está interrompido, o progresso está perdido
3. Corrija problemas e reinicie desde o início

***

## Solução de problemas durante o processamento

### O processamento é muito lento

**Causas possíveis:**

* Imagens de destino não marcadas (verificando todas as imagens)
* HDD em vez de armazenamento SSD
* Recursos de sistemas insuficientes
* Muitos índices configurados
* Acesso à unidade de rede

**Soluções:**

1. Se acabou de iniciar e está no estágio de detecção: cancelar, marcar alvos, reiniciar
2. Para o futuro: usar SSD, reduzir índices, atualizar hardware
3. Considere CLI para processamento em lote de grandes conjuntos de dados

### Avisos de “espaço em disco”

**Soluções:**

1. Espaço livre na discoteca imediatamente
2. Mova o projeto para dirigir com mais espaço
3. Redusar o número de índices para exportar
4. Use o formato JPG em vez de TIFF (arquivos menores)

### Mensagens de “arquivo danificado”

**Soluções:**

1. Copie novamente as imagens do cartão SD para garantir a integridade
2. Teste o cartão SD em busca de erros
3. Remoção de arquivos incluídos no projeto
4. Continuar processando as imagens restantes

### Superaquecimento/estrangulamento do sistema

**Soluções:**

1. Garanta ventilação adequada
2. Limpe a poeira das aberturas de ventilação do computador
3. Reduza a carga de processamento (use o modo Livre em vez do Cloros+)
4. Processe durante os horários mais frios do dia

***

## Processando notificação completa

Quando o processamento terminar:

* Barra de progresso chega a 100%
* **A mensagem "Processamento concluído"** aparece no log de depuração
* O botão Iniciar fica ativado novamente
* Todos os arquivos de saída estão na subpasta do modelo da câmera

***

## Próximas etapas

Assim que o processamento para conclusão:

1. **Revisar resultados** - Consulte [Concluindo o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** - Verifique todos os arquivos exportados corretamente
3. **Revisar o registro de depuração** - Verifique se há avisos ou erros
4. **Visualizar imagens processadas** - Utilize o Image Viewer ou software externo

Para obter informações sobre como revisar e usar os resultados processados, consulte [Finalizando o Processamento](finishing-the-processing.md).