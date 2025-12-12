# Finalizando o Processamento

Assim que o Chloros conclui o processamento, é hora de revisar seus resultados, verificar a qualidade da saída e preparar as imagens processadas para uso em seu fluxo de trabalho. Esta página orienta você nas etapas finais e nas próximas ações.

## Processando Indicação Completa

Quando o processamento para terminar com êxito, você verá vários indicadores:

* ✅ **Barra de progresso**: alcance 100% de conclusão
* ✅ **Log de depuração**: mostra a mensagem "Processamento concluído"
* ✅ **Botão Iniciar**: Torna-se ativado novamente (pronto para a próxima execução de processamento)
* ✅ **Arquivos de saída**: todas as imagens processadas salvas na subpasta do modelo da câmera

***

## Localizando suas imagens processadas

### Abrindo a pasta de saída

1. Clique no ícone **Menu Principal** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (canto superior esquerdo)
2. Selecione **"Abrir pasta do projeto"**
3. Seu explorador de arquivos abre no diretório do projeto
4. Localize seu projeto pelo nome

***

## Revisão de imagens processadas

### Visualização rápida no File Explorer

**Visualização integrada do Windows:**

1. Navegue até a subpasta do modelo da câmera
2. Selecione um arquivo de imagem
3. A visualização aparece no painel de visualização do Windows Explorer
4. Use as teclas de seta para navegar pelas imagens

### Visualização em visualizadores de imagens externas

**Visualizadores recomendados:**

* **QGIS** - Software GIS gratuito (melhor para análise georreferenciada multiespectral)
* **IrfanView** - Visualizador de imagens rápido e leve (suporta TIFF)
* **Adobe Photoshop** – Edição profissional (suporte TIFF)
* **GIMP** – Alternativa gratuita ao Photoshop
* **Fotos do Windows** – Visualização básica (pode não suportar TIFF de 16 bits)

### Pré-visualização no visualizador de imagens Cloros

Use o Visualizador de imagens integrado do Chloros para visualização avançada:

1. Clique em uma miniatura de imagem no navegador de arquivos
2. A imagem é aberta na área de visualização principal
3. Clique no **Visualizador de imagens** guia <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de imagens](../image-viewer-gui/page-3.md) para obter instruções detalhadas.

***

## Revendo o log de depuração

### Verifique se há avisos ou erros

1. Abra **Log de depuração** guia <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Percorrer as mensagens
3. Adquira avisos amarelos ou erros vermelhos
4. Revise quaisquer problemas observados
5. Entre em contato com o suporte do MAPIR para obter assistência

### Salvando o registro

Para manter um registro de processamento ou enviar ao Suporte MAPIR:

1. Clique no botão **"Copiar"** ou **"Baixar"**
2. Salvar como arquivo de texto na pasta do projeto
3. Incluir com a documentação do projeto
4. Envie para o suporte do MAPIR se encontrar problemas

***

## Problemas e soluções comuns de saída

### Problema: arquivos de saída ausentes

**Causas possíveis:**

* Os arquivos não atenderam aos critérios de processamento
* Imagens somente de destino (excluídas de exportação)
* O espaço em disco acabou durante a exportação
* Corrupção de arquivo durante o processamento

**Soluções:**

1. Verifique o log de depuração para mensagens de erro/ignorar
2. Verifique se o espaço no disco era suficiente
3. Contar arquivos: deve ser responsável (contagem original - contagem alvo) × (índices + 1)
4. Reimporte e reprocesse quaisquer arquivos ausentes

### Problema: bordas escuras ou escondidas (vinhetas ainda visíveis)

**Causas possíveis:**

* Correção de vinha desativada
* Câmera/lente que não está no banco de dados de perfis do Chloros
* Vinhetas extremas além da capacidade de correção

**Soluções:**

1. Verifique se a correção da vinheta foi habilitada nas configurações do projeto
2. Verifique o modelo da câmera detectada corretamente
3. Entre em contato com o suporte do MAPIR se a vinheta persistir

### Problema: núcleos ou valores incorretos

**Causas possíveis:**

* Nenhum alvo de detecção detectado
* Modelo de destino de escolha escolhido
* Calibração de refletância desativada
* Imagens alvo de baixa qualidade

**Soluções:**

1. Verifique se a retenção de refletância foi habilitada
2. Verifique as mensagens "Destino encontrado" no log de depuração
3. Revise a qualidade da imagem alvo
4. Reprocesse com alvos adequados marcados

### Problema: os valores do NDVI parecem errados

**Intervalos esperados de NDVI:**

* **Água, pedras, solo**: -0,1 a 0,2
* **Vegetação esparsa/não saudável**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação densa e saudável**: 0,6 a 0,9

**Se os valores estiverem fora desses momentos:**

1. Verifique se a concentração de refletância foi aplicada
2. Verifique se o registro do sensor de luz foi incluído
3. Verifique se os alvos de descoberta foram detectados
4. -se de que o modelo correto da câmera foi detectado
5. Revise o tempo e as condições de captura da imagem alvo

***

## Usando suas imagens processadas

### Para Fotogrametria/Criação Ortomosaica

**Fluxo de trabalho recomendado:**

1. **Importar imagens de refletância calibradas** para software de fotografia:
   * Pix4Dmapper
   * Metaforma Agisoft
   * DroneDeploy
   * WebODM
2. **Mantenha os metadados EXIF**: garanta que os dados do GPS sejam preservados para marcação geográfica
3. **Fluxos de trabalho calibrados**: use imagens de refletância para precisão científica
4. **Processar mosaicos de índice**: geração de NDVI ortomosaico a partir de imagens de índice individuais
5. **Exportar GeoTIFF georreferenciado**: Para uso em aplicativos GIS

### Para análise GIS

**Fluxo de trabalho recomendado:**

1. **Carregue no QGIS, ArcGIS ou similar**
2. **Usar imagens de refletância TIFF de 16 bits** para análise multibanda
3. **Use imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas para uso
4. **Calculadora raster**: Combine bandas para análise personalizada
5. **Exportar**: Crie mapas de classificação, detecção de alterações e mapas de saúde do vegetação

### Para análise/relatórios diretos

**Fluxo de trabalho recomendado:**

1. **Usar imagens de índice com núcleos LUT** para relatórios visuais
2. **Estatísticas de degradação**: NDVI médio por talhão/parcela
3. **Série temporal**: compare índices em várias sessões
4. **Gere relatórios**: incluindo mapas, estatísticas e visualizações

***

## Arquivamento e Backup

### Estratégia de backup recomendada

**O que salvar:**

* ✅ **Imagens RAW/JPG originais** - Arquivar em unidade/nuvem separadas
* ✅ **Saídas processadas** - Mantenha imagens e índices calibrados
* ✅ **Arquivo de projeto** - Contém todas as configurações para reprocessamento, se necessário
* ✅ **Log de depuração** - Detalhes de processamento de documentos
* ✅ **Imagens alvo de descoberta** - Para verificação e reprocessamento

**Recomendações de armazenamento:**

* **Backup imediato**: disco rígido externo
* **Arquivo de longo prazo**: armazenamento em nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: mantenha 2 a 3 cópias em locais diferentes

***

## Próximas execuções de processamento

### Reutilizando configurações do projeto

Se processar conjuntos de dados semelhantes no futuro:

1. **Salvar modelo de projeto** (se ainda não tiver feito)
2. **Crie um novo projeto** usando o modelo salvo
3. **Importar novas imagens**
4. **Processo** com configurações idênticas para consistência

### Processamento em lote de múltiplas sessões

Para várias sessões/conjuntos de dados:

**Opção 1: GUI – Vários projetos**

* Crie um projeto separado para cada sessão
* Use configurações de modelo consistentes
* Processe um de cada vez

**Opção 2: Cloros CLI (somente Cloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte [Documentação CLI](../CLI.md)

**Opção 3: Python SDK (somente Chloros+)**

* Controle programático
* Integração com pipelines de análise
* Consulte [documentação da API](../api-python-sdk.md)

***

## Solução de problemas de pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem esmagadores:

1. Manter as imagens originais (nunca excluída)
2. Abra o mesmo projeto no Chloros
3. Ajustar as configurações no painel Configurações do projeto
4. Processo novamente - as saídas substituirão os resultados anteriores

### Processando Subconjunto de Imagens

Para reprocessar apenas imagens específicas:

1. Criar novo projeto
2. Importe apenas as imagens que serão reveladas de reprocessamento
3. Use o mesmo modelo de configurações
4. Processador conjunto de dados menor

### Obtendo ajuda

Se você encontrar problemas:

* 📧 **E-mail**: info@mapir.camera (inclui registro de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual Cloros](../)

***

## Resumo: Fluxo de trabalho completo

Agora você concluiu todo o fluxo de trabalho de processamento do Cloros:

1. ✅ **Projeto criado** - Veja [Projetos](../projects.md)
2. ✅ **Arquivos adicionados** - Consulte [Adicionando arquivos](page-1.md)
3. ✅ **Configurações ajustadas** - Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
4. ✅ **Alvos marcados** - Consulte [Escolher imagens de destino](choosing-target-images.md)
5. ✅ **Processamento iniciado** - Consulte [Iniciando o processamento](starting-the-processing.md)
6. ✅ **Progresso monitorado** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página

**Suas imagens multiespectrais calibradas e corrigidas por refletância estão prontas para análise!**

***

## Recursos Adicionais

### Recursos avançados

* [**Image Viewer**](../image-viewer-gui/page-3.md) - Visualização e análise interativa
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Teste de índice personalizado
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Referência completa do índice

### Automação e Integração

* [**Documentação CLI**](../CLI.md) - Processamento em lote de linha de comando
* [**Python SDK**](../api-python-sdk.md) - Automação programática
* [**Recursos do Cloros+**](../#cloros) - Recursos avançados de processamento

### Suporte e Aprendizagem

* [**FAQ**](../faq.md) - Perguntas comuns respondidas
* [**Alvos de experiência**](../calibration-targets.md) - Noções básicas sobre rotina de refletância
* [**Câmeras suportadas**](../supported-cameras.md) - Hardware compatível
