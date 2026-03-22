# Conclusão do processamento

Assim que o Chloros concluir o processamento, é hora de analisar os resultados, verificar a qualidade da saída e preparar as imagens processadas para utilização no seu fluxo de trabalho. Esta página orienta-o através dos passos finais e das próximas ações.

## Indicação de processamento concluído

Quando o processamento for concluído com sucesso, verá vários indicadores:

* ✅ **Barra de progresso**: Atinge 100% de conclusão
* ✅ **Registo de depuração**: Mostra a mensagem «Processamento concluído»
* ✅ **Botão Iniciar**: Volta a ficar ativado (pronto para a próxima execução do processamento)
* ✅ **Ficheiros de saída**: Todas as imagens processadas guardadas na subpasta do modelo da câmara***

## Localizar as suas imagens processadas

### Abrir a pasta de saída

1. Clique no ícone do **Menu Principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (canto superior esquerdo)
2. Selecione **«Abrir pasta do projeto»**

3. O seu explorador de ficheiros abre-se no diretório do projeto
4. Localize o seu projeto pelo nome

***

## Visualização das imagens processadas

### Pré-visualização rápida no Explorador de ficheiros

**Pré-visualização integrada do Windows:**

1. Navegue até à subpasta do modelo da câmara
2. Selecione um ficheiro de imagem
3. A pré-visualização aparece no painel de pré-visualização do Windows Explorer
4. Use as teclas de seta para percorrer as imagens

### Pré-visualização em visualizadores de imagens externos

**Visualizadores recomendados:*** **QGIS** - Software GIS gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** - Visualizador de imagens rápido e leve (compatível com TIFF)
* **Adobe Photoshop** - Edição profissional (compatível com TIFF)
* **GIMP** - Alternativa gratuita ao Photoshop
* **Windows Photos** - Visualização básica (pode não ser compatível com TIFF de 16 bits)

### Pré-visualização no Visualizador de Imagens do Chloros

Utilize o Visualizador de Imagens integrado do Chloros para uma visualização avançada:

1. Clique numa miniatura de imagem no Navegador de Ficheiros
2. A imagem abre-se na área de pré-visualização principal
3. Clique no separador **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
4. Utilize [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de Imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Analisar o Registo de Depuração

### Verificar se há avisos ou erros

1. Abra o separador **Registo de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Percorra as mensagens
3. Procure avisos amarelos ou erros vermelhos
4. Analise quaisquer problemas detetados
5. Contacte o suporte da MAPIR para obter assistência

### Guardar o registo

Para manter um registo do processamento ou para enviar ao Suporte da MAPIR:

1. Clique no botão **&quot;Copiar&quot;**ou**&quot;Descarregar&quot;**

2. Guarde como ficheiro de texto na pasta do projeto
3. Inclua na documentação do projeto
4. Envie para o suporte da MAPIR caso surjam problemas

***

## Problemas comuns de saída e soluções

### Problema: Ficheiros de saída em falta

**Possíveis causas:**

* Os ficheiros não cumpriram os critérios de processamento
* Imagens apenas de destino (excluídas da exportação)
* Esgotou-se o espaço em disco durante a exportação
* Corrupção de ficheiros durante o processamento

**Soluções:**

1. Verifique o Registo de Depuração para mensagens de omissão/erro
2. Verifique se o espaço em disco era suficiente
3. Conte os ficheiros: Deve corresponder a (contagem original - contagem de destino) × (índices + 1)
4. Reimporte e reprocesse quaisquer ficheiros em falta

### Problema: Bordas escuras ou claras (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmara/lente não consta na base de dados de perfis do Chloros
* Vinheta extrema além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta foi ativada nas Definições do Projeto
2. Verifique se o modelo da câmara foi detetado corretamente
3. Contacte o suporte MAPIR se o efeito de vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Não foram detetados alvos de calibração
* Modelo de alvo de calibração errado selecionado
* Calibração de refletância desativada
* Imagens de alvo de baixa qualidade

**Soluções:**

1. Verifique se a calibração de refletância estava ativada
2. Verifique as mensagens «Alvo encontrado» no Registo de Depuração
3. Analise a qualidade da imagem do alvo
4. Reprocesse com os alvos adequados marcados

### Problema: Os valores do NDVI parecem errados

**Intervalos esperados do NDVI:*** **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação esparsa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9**Se os valores estiverem fora destes intervalos:**

1. Verifique se a calibração de refletância foi aplicada
2. Verifique se o registo do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detetados
4. Certifique-se de que o modelo de câmara correto foi detetado
5. Reveja o momento e as condições de captura da imagem-alvo

***

## Utilização das suas imagens processadas

### Para fotogrametria / criação de ortomosaicos

**Fluxo de trabalho recomendado:**

1.**Importe imagens de refletância calibradas** para software de fotogrametria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Mantenha os metadados EXIF**: Certifique-se de que os dados GPS são preservados para georreferenciamento
3. **Fluxos de trabalho calibrados**: Utilize imagens de refletância para obter precisão científica
4. **Processe mosaicos de índice**: Crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exporte GeoTIFF georreferenciados**: Para utilização em aplicações SIG

### Para análise SIG

**Fluxo de trabalho recomendado:**

1.**Carregue no QGIS, ArcGIS ou similar**

2.**Utilize imagens de refletância de 16 bits TIFF** para análise multibanda
3. **Utilize imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas a utilizar
4. **Calculadora raster**: Combine bandas para análise personalizada
5. **Exportar**: Crie mapas de classificação, deteção de alterações e mapas de saúde da vegetação

### Para Análise Direta / Relatórios

**Fluxo de trabalho recomendado:**

1.**Utilize imagens de índice com cores LUT** para relatórios visuais
2. **Extraia estatísticas**: Média NDVI por campo/parcela
3. **Série temporal**: Compare índices em várias sessões
4. **Gerar relatórios**: Inclua mapas, estatísticas e visualizações***

## Arquivamento e cópia de segurança

### Estratégia de cópia de segurança recomendada

**O que guardar:*** ✅ **Imagens RAW/JPG originais** - Arquive numa unidade separada/na nuvem
* ✅ **Resultados processados** - Guarde imagens calibradas e índices
* ✅ **Ficheiro do projeto** - Contém todas as configurações para reprocessamento, se necessário
* ✅ **Registo de depuração** - Documenta detalhes do processamento
* ✅ **Imagens de alvo de calibração** - Para verificação e reprocessamento**Recomendações de armazenamento:*** **Cópia de segurança imediata**: Disco rígido externo
* **Arquivo de longo prazo**: Armazenamento na nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: Guarde 2-3 cópias em locais diferentes***

## Próximas execuções de processamento

### Reutilização das definições do projeto

Se for processar conjuntos de dados semelhantes no futuro:

1. **Guarde o modelo do projeto** (se ainda não tiver sido feito)
2. **Crie um novo projeto** utilizando o modelo guardado
3. **Importe novas imagens**

4.**Processe**com definições idênticas para garantir a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:**Opção 1: GUI - Vários projetos**

* Crie um projeto separado para cada sessão
* Utilize definições de modelo consistentes
* Processe um de cada vez

**Opção 2: Chloros CLI (apenas Chloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte a [Documentação do CLI](../CLI.md)

**Opção 3: Python SDK (apenas Chloros+)**

* Controlo programático
* Integração com pipelines de análise
* Consulte a [Documentação do API](../api-python-sdk.md)

***

## Resolução de problemas no pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Mantenha as imagens originais (nunca apague)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel Configurações do projeto
4. Processe novamente - os resultados substituirão os resultados anteriores

### Processamento de um subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que precisam de reprocessamento
3. Use o mesmo modelo de configurações
4. Processe um conjunto de dados menor

### Obter ajuda

Se tiver problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o registo de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)***

## Resumo: Fluxo de trabalho completo

Concluiu agora o fluxo de trabalho completo de processamento do Chloros:

1. ✅ **Projeto criado** - Consulte [Projetos](../projects.md)
2. ✅ **Adicionou ficheiros** - Consulte [Adicionar ficheiros](adding-files-to-a-project.md)
3. ✅ **Ajustou as definições** - Consulte [Ajustar definições do projeto](adjusting-project-settings.md)
4. ✅ **Marcou alvos** - Ver [Escolher imagens-alvo](choosing-target-images.md)
5. ✅ **Iniciou o processamento** - Ver [Iniciar o processamento](starting-the-processing.md)
6. ✅ **Progresso monitorizado** - Consulte [Monitorizar o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revistos** - Esta página**As suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Funcionalidades avançadas

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualização e análise interativas
* [**Área de testes de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Testes de índices personalizados
* [**Fórmulas de Índices Multiespectrais**](../project-settings/multispectral-index-formulas.md) - Referência completa de índices

### Automatização e Integração

* [**Documentação do CLI**](../CLI.md) - Processamento em lote via linha de comando
* [**Python SDK**](../api-python-sdk.md) - Automatização programática
* [**Chloros+ Funcionalidades**](../#chloros) - Capacidades de processamento avançadas

### Suporte e aprendizagem

* [**Perguntas frequentes**](../faq.md) - Respostas a perguntas comuns
* [**Alvos de calibração**](../calibration-targets.md) - Compreender a calibração da refletância
* [**Câmaras compatíveis**](../supported-cameras.md) - Hardware compatível
