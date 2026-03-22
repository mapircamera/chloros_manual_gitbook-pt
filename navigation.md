# GUI: Navegação

Quando iniciar o Chloros e o Chloros (Navegador) pela primeira vez, o backend será iniciado. Assim que estiver pronto, o ícone do menu principal, no canto superior esquerdo, será exibido <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu Principal

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

A partir do menu principal, pode:

* **Novo Projeto** — criar um novo projeto
* **Abrir Projeto** — abrir um projeto existente
* **Abrir Pasta do Projeto** — abrir a pasta do projeto no seu explorador de ficheiros
* **Adicionar Ficheiros** — adicionar ficheiros de imagem individuais ao projeto atual _(visível após a abertura de um projeto)_
* **Adicionar pasta** — adicionar uma pasta de imagens ao projeto atual _(visível após a abertura de um projeto)_
* **Iniciar processamento / Parar processamento** — iniciar ou parar o fluxo de processamento de imagens _(ativado após a adição de ficheiros)_

{% hint style="info" %}
**Apenas Windows**: A GUI do Chloros Desktop está disponível no Windows. Os utilizadores do Linux devem consultar o [CLI](CLI.md) e [Python SDK](api-python-sdk.md) para o processamento sem interface gráfica.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Botão Reproduzir/Iniciar

Quando ativado, o botão de início do processamento inicia o fluxo de processamento de imagens.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de progresso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">No modo gratuito Chloros, que processa todos os ficheiros sequencialmente, a barra de progresso mostra 2 fases: Detecção de Alvo e Processamento.

No modo licenciado Chloros+, que processa todos os ficheiros simultaneamente, a barra de progresso mostra 4 fases: Detecção, Análise, Calibração, Exportação. Se passar o cursor do rato sobre a barra de progresso do Chloros+, será apresentado um painel suspenso com as 4 fases da barra de progresso para que possa acompanhar o processo. Clicar na barra de progresso superior irá congelar o painel suspenso; clicar novamente irá descongelá-lo.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones com os quais pode interagir:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Definições do projeto](project-settings/project-settings.md)

O separador Definições do projeto permite ajustar as definições globais e de processamento do projeto. Ajuste estas definições antes de iniciar o processamento dos seus ficheiros.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de ficheiros

Adicione ficheiros/pastas e remova ficheiros do projeto. Os ficheiros duplicados são ignorados. Marque a caixa da coluna de destino para qualquer imagem de destino, e o processamento irá analisar apenas as imagens marcadas como destinos, acelerando significativamente o seu tempo de processamento. Utilize o botão de alternância Imagem/Metadados para alternar entre a visualização da grelha de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de Imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando se clica numa imagem no visualizador de imagens principal, esta é aberta em ecrã inteiro no separador Visualizador de Imagens.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Visualize as suas imagens num mapa 2D interativo com base nas suas coordenadas GPS. Compatível com os fornecedores de mosaicos do Google Maps e da ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o cursor sobre os marcadores para ver pré-visualizações das miniaturas das imagens.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registo de depuração

Analise o registo para ver as impressões de depuração quando ocorrerem problemas. Copie/descarregue o registo e envie-o para o [Suporte MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Início de sessão do utilizador](chloros+-login.md)

A barra lateral de login do utilizador permite-lhe iniciar sessão na sua conta Chloros+ para desbloquear funcionalidades avançadas. Também pode ver a versão atual da aplicação, bem como ajustar o idioma do texto apresentado na GUI do Chloros e no CLI.
