# Concluindo o processamento

Assim que o Chloros concluir o processamento, é hora de rever os resultados, verificar a qualidade da saída e preparar as imagens processadas para uso no seu fluxo de trabalho. Esta página orienta-o nas etapas finais e nas próximas ações.

## Indicação de processamento concluído

Quando o processamento for concluído com sucesso, você verá vários indicadores:

* ✅ **Barra de progresso**: atinge 100% de conclusão
* ✅ **Registo de depuração**: mostra a mensagem &quot;Processamento concluído&quot;
* ✅ **Botão Iniciar**: fica ativado novamente (pronto para a próxima execução do processamento)
* ✅ **Ficheiros de saída**: Todas as imagens processadas são guardadas na subpasta do modelo da câmara

***

## Localizar as suas imagens processadas

### Abrir a pasta de saída

1. Clique no ícone **Menu principal** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (canto superior esquerdo)
2. Selecione **«Abrir pasta do projeto»**
3. O seu explorador de ficheiros abre no diretório do projeto
4. Localize o seu projeto pelo nome

***

## Revendo imagens processadas

### Pré-visualização rápida no explorador de ficheiros

**Pré-visualização integrada no Windows:**

1. Navegue até à subpasta do modelo da câmara
2. Selecione um ficheiro de imagem
3. A pré-visualização aparece no painel de pré-visualização do Windows Explorer
4. Use as setas do teclado para navegar pelas imagens

### Pré-visualização em visualizadores de imagens externos

**Visualizadores recomendados:**

* **QGIS** - Software GIS gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** - Visualizador de imagens rápido e leve (compatível com TIFF)
* **Adobe Photoshop** - Edição profissional (suporte a TIFF)
* **GIMP** - Alternativa gratuita ao Photoshop
* **Windows Photos** - Visualização básica (pode não suportar TIFF de 16 bits)

### Pré-visualização no Visualizador de Imagens Chloros

Use o Visualizador de Imagens integrado do Chloros para visualização avançada:

1. Clique numa miniatura de imagem no Navegador de Ficheiros
2. A imagem abre na área de pré-visualização principal
3. Clique no separador **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de Imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Revendo o Log de Depuração

### Verifique se há avisos ou erros

1. Abra a guia **Registo de depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Percorra as mensagens.
3. Procure avisos amarelos ou erros vermelhos.
4. Revise quaisquer problemas observados.
5. Entre em contacto com o suporte MAPIR para obter assistência.

### Guardar o registo

Para manter um registo do processamento ou enviar para o suporte MAPIR:

1. Clique no botão **&quot;Copiar&quot;** ou **&quot;Transferir&quot;**
2. Guarde como ficheiro de texto na pasta do projeto
3. Inclua na documentação do projeto
4. Envie para o suporte MAPIR se encontrar problemas

***

## Problemas comuns de saída e soluções

### Problema: Ficheiros de saída em falta

**Possíveis causas:**

* Os ficheiros não cumpriram os critérios de processamento
* Imagens apenas de destino (excluídas da exportação)
* O espaço em disco esgotou-se durante a exportação
* Corrupção de ficheiros durante o processamento

**Soluções:**

1. Verifique o registo de depuração para mensagens de omissão/erro
2. Verifique se o espaço em disco era suficiente
3. Conte os ficheiros: deve corresponder (contagem original - contagem de destino) × (índices + 1)
4. Reimporte e reprocesse quaisquer ficheiros em falta

### Problema: Bordas escuras ou brilhantes (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmara/lente não presente na base de dados de perfis Chloros
* Vinheta extrema além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta foi ativada nas configurações do projeto
2. Verifique se o modelo da câmara foi detetado corretamente
3. Entre em contacto com o suporte MAPIR se a vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Nenhuma meta de calibração detetada
* Modelo de meta de calibração incorreto selecionado
* Calibração de refletância desativada
* Imagens de meta de baixa qualidade

**Soluções:**

1. Verifique se a calibração da refletância foi ativada
2. Verifique as mensagens «Alvo encontrado» no registo de depuração
3. Reveja a qualidade da imagem do alvo
4. Reprocesse com os alvos corretos marcados

### Problema: Os valores NDVI parecem incorretos

**Intervalos NDVI esperados:**

* **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação escassa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9

**Se os valores estiverem fora destes intervalos:**

1. Verifique se a calibração da refletância foi aplicada
2. Verifique se o registo do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detetados
4. Certifique-se de que o modelo correto da câmara foi detetado
5. Reveja o tempo e as condições de captura da imagem do alvo

***

## Usando as suas imagens processadas

### Para fotogrametria/criação de ortomosaico

**Fluxo de trabalho recomendado:**

1. **Importe imagens de refletância calibradas** para o software de fotogrametria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Mantenha os metadados EXIF**: Certifique-se de que os dados GPS sejam preservados para geotagging
3. **Fluxos de trabalho calibrados**: Use imagens de refletância para precisão científica
4. **Processe mosaicos de índice**: Crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exporte GeoTIFF georreferenciado**: para uso em aplicações GIS

### Para análise GIS

**Fluxo de trabalho recomendado:**

1. **Carregue no QGIS, ArcGIS ou similar**
2. **Use imagens de refletância TIFF de 16 bits** para análise multibanda
3. **Use imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas para uso
4. **Calculadora raster**: combine bandas para análise personalizada
5. **Exportar**: crie mapas de classificação, deteção de alterações, mapas de saúde da vegetação

### Para análise direta/relatórios

**Fluxo de trabalho recomendado:**

1. **Use imagens de índice com cores LUT** para relatórios visuais
2. **Extraia estatísticas**: média NDVI por campo/parcela
3. **Série temporal**: compare índices em várias sessões
4. **Gere relatórios**: inclua mapas, estatísticas e visualizações

***

## Arquivamento e backup

### Estratégia de backup recomendada

**O que salvar:**

* ✅ **Imagens RAW/JPG originais** - arquive em unidade/nuvem separada
* ✅ **Resultados processados** - Mantenha imagens e índices calibrados
* ✅ **Ficheiro do projeto** - Contém todas as configurações para reprocessamento, se necessário
* ✅ **Registo de depuração** - Documenta detalhes do processamento
* ✅ **Imagens de alvo de calibração** - Para verificação e reprocessamento

**Recomendações de armazenamento:**

* **Cópia de segurança imediata**: Disco rígido externo
* **Arquivo de longo prazo**: Armazenamento na nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: Mantenha 2-3 cópias em locais diferentes

***

## Próximas execuções de processamento

### Reutilização das definições do projeto

Se processar conjuntos de dados semelhantes no futuro:

1. **Salve o modelo do projeto** (se ainda não tiver feito isso)
2. **Crie um novo projeto** usando o modelo salvo
3. **Importe novas imagens**
4. **Processe** com configurações idênticas para manter a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:

**Opção 1: GUI - Vários projetos**

* Crie um projeto separado para cada sessão
* Use configurações de modelo consistentes
* Processe um de cada vez

**Opção 2: Chloros CLI (somente Chloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte [Documentação do CLI](../CLI.md)

**Opção 3: Python SDK (apenas Chloros+)**

* Controlo programático
* Integração com pipelines de análise
* Consulte a [Documentação API](../api-python-sdk.md)

***

## Resolução de problemas pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Mantenha as imagens originais (nunca as elimine)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel Configurações do projeto
4. Processe novamente - os resultados substituirão os resultados anteriores

### Processamento de subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que precisam ser reprocessadas
3. Use o mesmo modelo de configurações
4. Processe um conjunto de dados menor

### Obtenha ajuda

Se tiver problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o registo de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Perguntas frequentes**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)

***

## Resumo: fluxo de trabalho completo

Agora concluiu todo o fluxo de trabalho de processamento do Chloros:

1. ✅ **Projeto criado** - Consulte [Projetos](../projects.md)
2. ✅ **Arquivos adicionados** - Consulte [Adicionar arquivos](adding-files-to-a-project.md)
3. ✅ **Configurações ajustadas** - Consulte [Ajustar configurações do projeto](adjusting-project-settings.md)
4. ✅ **Alvos marcados** - Consulte [Escolher imagens alvo](choosing-target-images.md)
5. ✅ **Processamento iniciado** - Consulte [Iniciar o processamento](starting-the-processing.md)
6. ✅ **Progresso monitorado** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página

**Suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Recursos avançados

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualização e análise interativas
* [**Sandbox de índice/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Teste de índice personalizado
* [**Fórmulas de Índice Multiespectral**](../project-settings/multispectral-index-formulas.md) - Referência completa do índice

### Automação e integração

* [**Documentação CLI**](../CLI.md) - Processamento em lote por linha de comando
* [**Python SDK**](../api-python-sdk.md) - Automatização programática
* [**Recursos do Chloros+**](../#chloros) - Recursos avançados de processamento

### Suporte e aprendizagem

* [**Perguntas frequentes**](../faq.md) - Respostas a perguntas comuns
* [**Alvos de calibração**](../calibration-targets.md) - Compreender a calibração da refletância
* [**Câmaras compatíveis**](../supported-cameras.md) - Hardware compatível
