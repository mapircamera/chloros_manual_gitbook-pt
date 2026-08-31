# GUI: Navegação

Quando inicia o Chloros pela primeira vez, este inicia o seu backend de processamento. Assim que o backend estiver pronto, o ícone do menu principal, no canto superior esquerdo, é exibido <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> e os separadores «Câmaras» e «Sensores de luz» ficam disponíveis na barra lateral esquerda (até então, estão desativados).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### Menu principal do <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>A partir do menu principal, pode:

* **Novo Projeto**— criar um novo projeto. Se tiver modelos de projeto guardados, aparece um menu suspenso**Selecionar Modelo** para que o novo projeto comece com as definições de um modelo.
* **Abrir Projeto**— abrir um projeto existente. A lista inclui um botão**Abrir pasta do projeto** que abre a pasta dos projetos no seu explorador de ficheiros.
* **Duplicar projeto** — copiar o projeto atualmente aberto com um novo nome (é sugerido um nome livre, como «MeuProjeto (2)») e abrir a cópia. _(visível após a abertura de um projeto)_
* **Adicionar ficheiros** — adiciona ficheiros de imagem individuais ao projeto atual _(visível após a abertura de um projeto)_
* **Adicionar pasta** — adiciona uma ou mais pastas de imagens ao projeto atual _(visível após a abertura de um projeto)_
* **Iniciar Processamento / Parar Processamento** — inicia ou pára o fluxo de processamento de imagens _(ativado após a adição de ficheiros)_
* **Ligar à Câmara** — acede ao [separador Câmaras](lattice/) para ligar uma câmara ou matriz LATTICE. Funciona sem ter um projeto aberto.
* **Ligar ao sensor de luz** — acede ao [separador «Sensores de luz»](daq/) para ligar um sensor de luz DAQ. Funciona sem ter um projeto aberto.

{% hint style="info" %}
** Apenas para oWindows**: A interface gráfica do utilizador (GUI) doChloros

Desktop está disponível emWindows

. Os utilizadores doLinux

devem consultar a documentação [CLI

](CLI.md) e [Python

SDK

](api-python-sdk.md) para o processamento sem interface gráfica.
{% endhint %}

### Botão «Play/Start» do<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Quando ativado, o botão «Start Processing» inicia o fluxo de processamento de imagens.

### Barra de progresso do<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">



No modo gratuitoChloros

, que processa todos os ficheiros sequencialmente, a barra de progresso mostra duas fases: «Target Detection» e «Processing».

No modo pagoChloros

+ com licença, que processa todos os ficheiros em simultâneo, a barra de progresso apresenta 4 fases: Detecção, Análise, Calibração e Exportação. Se passar o cursor do rato sobre a barra de progressoChloros

+, será exibido um painel expandido com as 4 fases da barra de progresso, para que possa acompanhar o processo. Clicar na barra de progresso superior congela o painel suspenso; clicar novamente descongela-o.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones com os quais pode interagir, nesta ordem, de cima para baixo:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Definições do projeto](project-settings/project-settings.md)

O separador «Definições do projeto» permite-lhe ajustar as definições globais do projeto e as definições de processamento do projeto. Ajuste estas definições antes de iniciar o processamento dos seus ficheiros.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de ficheiros

Adicione ficheiros/pastas e remova ficheiros do projeto. Os ficheiros duplicados são ignorados. Assinale a caixa da coluna «destino» para qualquer imagem de destino, e o processamento analisará apenas as imagens assinaladas como destinos, acelerando significativamente o tempo de processamento. Utilize o botão de alternância «Imagem/Metadados» para alternar entre a visualização da grelha de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de Imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando se clica numa imagem no visualizador de imagens principal, esta é aberta em ecrã inteiro no separador «Visualizador de Imagens».

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visualizador de Mapas](image-viewer-gui/map-markers.md)

Visualize as suas imagens num mapa 2D interativo com base nas suas coordenadas GPS. Suporta os fornecedores de mosaicos do Google Maps e da ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o cursor sobre os marcadores para ver pré-visualizações em miniatura das imagens.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Câmaras](lattice/)

Ligue e controle câmaras LATTICE em tempo real — uma de cada vez ou como conjuntos sincronizados de várias câmaras. O separador apresenta mosaicos de pré-visualização em tempo real com sobreposições e histogramas, definições por câmara e por conjunto, bem como as Definições de Captura que determinam quais as câmaras e os tipos de exportação que a opção «Capturar Tudo» produz. Disponível assim que o backend estiver pronto; consulte a [secção LATTICE](lattice/) para obter o guia completo.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensores de luz](daq/)

Ligue sensores de luz DAQ — DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet) — e visualize os seus gráficos de espectro calibrados em tempo real em W/m²/nm. A partir daqui, pode gravar ficheiros `.daq` no projeto aberto, renomear sensores, selecionar perfis de correção de tampa e atualizar o firmware do DAQ-E. Disponível assim que o backend estiver pronto; consulte a [secção DAQ](daq/) para obter o guia completo.

#### Registo de depuração do <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Analise o registo para verificar as mensagens de depuração quando ocorrerem problemas. Copie/descarregue o registo e envie-o para o [Suporte da MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### [Início de sessão do utilizador](chloros+-login.md) <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [User Login](chloros+-login.md)

A barra lateral de início de sessão permite-lhe iniciar sessão na sua conta Chloros+ para desbloquear funcionalidades avançadas. Também pode visualizar a versão atual da aplicação, bem como ajustar o idioma do texto apresentado na GUI Chloros e no CLI.
