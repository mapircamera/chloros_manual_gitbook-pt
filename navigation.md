# GUI: Navegação

Quando iniciar o Chloros e o Chloros (Navegador) pela primeira vez, o seu backend será iniciado. Quando estiver pronto, o ícone do menu principal no canto superior esquerdo será revelado <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

No menu principal, pode iniciar um novo projeto, abrir um projeto existente ou abrir a pasta do projeto.

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Botão Reproduzir/Iniciar

Quando ativado, o botão Iniciar processamento inicia o pipeline de processamento de imagens.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de progresso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">No modo gratuito Chloros, que processa todos os ficheiros sequencialmente, a barra de progresso mostrará 2 etapas: Detecção do alvo e Processamento.

No modo licenciado pago Chloros+, que processa todos os ficheiros simultaneamente, a barra de progresso mostra 4 etapas: Detecção, Análise, Calibração, Exportação. Se passar o cursor do rato sobre a barra de progresso Chloros+, será apresentado um painel alargado com quatro barras de progresso para que possa acompanhar o processo. Ao clicar na barra de progresso superior, o painel suspenso será congelado; ao clicar novamente, será descongelado.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones para interagir:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Definições do projeto](project-settings/project-settings.md)

O separador Definições do projeto permite ajustar as definições globais do projeto e as definições de processamento do projeto. Ajuste-as antes de começar a processar os seus ficheiros.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de ficheiros

Adicione ficheiros/pastas e remova ficheiros do projeto. Ficheiros duplicados são ignorados. Marque a caixa da coluna de destino para qualquer imagem de destino, e o processamento irá considerar apenas as imagens marcadas como destinos, acelerando significativamente o tempo de processamento. Use o botão Imagem/Metadados para alternar entre a visualização da grade de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando uma imagem é clicada no visualizador de imagens principal, ela é aberta em tela cheia na guia Visualizador de imagens.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Veja as suas imagens num mapa 2D interativo com base nas suas coordenadas GPS. Suporta os fornecedores de mosaicos Google Maps e ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o cursor sobre os marcadores para ver pré-visualizações das miniaturas das imagens.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registo de depuração

Revise o registo para impressões de depuração quando ocorrerem problemas. Copie/descarregue o registo e envie para o [Suporte MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Login do utilizador](chloros+-login.md)

A barra lateral de login do utilizador permite que você faça login na sua conta Chloros+ para desbloquear recursos avançados. Você também pode ver a versão atual do aplicativo, bem como ajustar o idioma do texto exibido na GUI Chloros e no CLI.
