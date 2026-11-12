---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>O

Chloros é uma aplicação de software da [MAPIR](https://www.mapir.camera) destinada ao processamento de imagens multiespectrais, ao controlo em tempo real do hardware MAPIR e ao registo de dados de sensores. O Chloros 1.3.0 suporta toda a família de produtos MAPIR:

* **Câmaras Survey3** — processam capturas RAW+JPG, convertendo-as em mapas calibrados de refletância e índices de vegetação. Consulte [Câmaras compatíveis](supported-cameras.md).
* **Câmaras LATTICE** — ligue os módulos de câmara multiespectral GigE em tempo real, individualmente ou como conjuntos sincronizados de várias câmaras: pré-visualize, capture e processe em produtos calibrados de radiação e refletância. Consulte a [secção LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — Sensores espectrais DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E e DAQ-E-S (Ethernet): registos espectrais calibrados em tempo real, gravações `.daq` e iluminação descendente para processamento de refletância. Consulte a [secção DAQ](daq/README.md).

{% hint style="success" %}
**Novidades no Chloros 1.3.0**: os assistentes de IA podem controlar o Chloros através do `chloros-mcp`, e a CLI, o SDK e o `chloros-mcp` oferecem as mesmas funcionalidades. Os produtos LATTICE index foram melhorados, as matrizes de sensores completas são mais rápidas e funcionam em qualquer rede, e o backend continua a funcionar entre comandos. Consulte a lista de novidades abaixo e [Descarregue](download.md) os instaladores.
{% endhint %}

{% hint style="info" %}
**Está a utilizar o Chloros com um assistente de IA?** Este manual foi concebido para isso. Indique ao seu assistente:

* `https://mapir.gitbook.io/chloros/llms.txt` — índice legível por máquina de todas as páginas.
* Qualquer página em Markdown bruto — acrescente `.md` ao seu URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência CLI](reference/cli-reference.md) e a [Referência SDK](reference/sdk-reference.md) — páginas de referência completas, com valores exatos, escritas para consumo por LLM.

Exemplo de prompt: *«Leia https://mapir.gitbook.io/chloros/reference/cli-reference.md, e, em seguida, escreva um script que inicie sessão e processe a pasta ~/flights/flight_001 em ficheiros GeoTIFF de refletância + NDVI.»*

Ou deixe-o executar o Chloros diretamente: o `chloros-mcp`, um servidor MCP instalado com o SDK, funciona com o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP.

Guia completo: [Utilizar o Chloros com assistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novidades no Chloros 1.3.0

* **Os assistentes de IA podem controlar o Chloros.** O `chloros-mcp`, um servidor MCP instalado com o SDK do Python, permite que o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP processem imagens e controlem câmaras LATTICE, matrizes e sensores de luz DAQ. Consulte [Utilizar o Chloros com assistentes de IA](ai-assistants.md).
* **A CLI, o SDK e o `chloros-mcp` oferecem as mesmas funcionalidades.** Por motivos de segurança, apenas o início de sessão, o aprovisionamento do DAQ-E e os servidores DAQ diretos permanecem na CLI. Alguns scripts da versão 1.2.0 necessitam de alterações: consulte **Alterações na versão 1.3.0** no início da [Referência da CLI](reference/cli-reference.md) e da [Referência do SDK](reference/sdk-reference.md).
* **Produtos de índice LATTICE melhorados.** O NDVI e os outros produtos de índice exportados a partir de capturas do LATTICE foram melhorados, em todos os modos de processamento.
* **Opções do instalador do Windows.**Em**Configurações do sistema**, o instalador oferece as opções**Ativar caminhos longos do Windows**e**Excluir o backend do Chloros da verificação do Microsoft Defender**, ambas marcadas por predefinição. A primeira permite que as pastas de projeto tenham nomes com mais de 260 caracteres; a segunda impede que a verificação em tempo real abrande a captura da câmara (cerca de 30% da taxa de captura em taxas elevadas). O instalador agora também adiciona a pasta CLI ao `PATH` apenas uma vez e remove as entradas duplicadas deixadas por instaladores anteriores. Ver [Download](download.md).
* **As pastas de projeto com nomes muito longos são rejeitadas logo à partida.** Com os caminhos longos do Windows desativados (a predefinição do Windows), uma pasta de projeto cujo nome exceda o limite de 260 caracteres é rejeitada antes de qualquer execução, apresentando a pasta de projeto mais longa que cabe e indicando como ativar os caminhos longos. Anteriormente, essa execução não gerava quaisquer produtos. Com os caminhos longos ativados, os produtos em pastas muito profundas mantêm agora os seus metadados (etiquetas GPS e XMP).
* **Máscaras de limitação.** Quando a separação de bandas de um produto de radiação ou refletância LATTICE produz um valor de banda negativo, esse valor é limitado a zero, e uma máscara por pixel numa pasta `clamp_masks/`, ao lado das pastas dos produtos, regista quais os píxeis e bandas que foram afetados. O produto nomeia a sua máscara na etiqueta `Chloros:UnmixClampMask`. As visualizações em tempo real agora cortam exatamente da mesma forma que a exportação. Consulte [Formatos de imagem de saída](output-image-formats.md).
* **Os produtos descrevem-se a si próprios.** Os produtos de índice incluem as suas próprias etiquetas de processamento (`Chloros:ProcessingLevel=index`, a expressão de índice e se foram calculados a partir da radianância ou da refletância), em vez das do ficheiro bruto de origem. Os produtos Survey3 incluem etiquetas de processamento Chloros em todos os modos de processamento. Os ficheiros brutos e debayered do LATTICE mantêm a orientação nativa do sensor e registam a sua orientação no XMP; o Visualizador de Imagens apenas os roda para efeitos de visualização.
* **Os pacotes para Linux são fornecidos descompactados.** O backend e a CLI são instalados como pastas na diretória `/usr/lib/chloros/`, pelo que já não se descompactam automaticamente a cada inicialização. No x86_64, o backend já não demora cerca de 100 segundos a iniciar, e o `.deb` tem agora cerca de metade do tamanho anterior. O ExifTool incluído funciona agora numa instalação limpa do Ubuntu. O `.deb` instala agora também o Python SDK para o sistema `python3` (instale-o com o `sudo apt install ./chloros_1.3.0_<arch>.deb`), aumenta os buffers de receção de sockets do kernel necessários para os fluxos da câmara e adiciona um serviço de backend que é executado como o seu utilizador. O pacote Jetson, `chloros_1.3.0_arm64_jp6.deb`, foi compilado para o JetPack 6 e processa na GPU do Jetson com a compilação CUDA do PyTorch do JetPack. No Linux, cada imagem exportada chega agora ao disco uma única vez (gravada e etiquetada na memória e, em seguida, copiada para o local de destino): num Jetson, onde o disco determina o ritmo, uma execução do LATTICE com 288 imagens demora 14 minutos em vez de 24. Consulte [Instalação no Linux](linux/linux-installation.md).
* **As câmaras LATTICE OCN têm uma leitura mais precisa sob luz solar.** A radiação, a refletância e os índices recebem uma correção rápida após a separação de bandas, pelo que um alvo cinzento apresenta um valor de NDVI próximo do neutro sob luz solar. As câmaras RGN permanecem inalteradas.
* **Relatórios de erros na aplicação.** Um ícone de erro na parte superior do separador «Registo de Depuração» envia à MAPIR uma descrição, até 10 imagens, os registos e os detalhes do sistema. Ver [GUI: Navegação](navigation.md).
* **Matrizes LATTICE.**Por predefinição, as matrizes mantêm o sensor completo, e o transporte compactado de 12 bits (`--packed`, predefinição `auto`) aumenta as taxas de fotogramas do sensor completo em cerca de um terço; uma matriz funciona em qualquer rede à taxa de fotogramas que esta permitir (uma ligação lenta reduz a taxa de fotogramas em vez de recusar a matriz ou reduzir os seus fotogramas); uma matriz cujas câmaras sejam reiniciadas recupera por si própria, com o PTP novamente ativado; a exposição automática já não reduz essa taxa em condições de pouca luz (preenche o período e, em seguida, aumenta o ganho); A captura contínua acompanha as matrizes rápidas; um controlo**Destaques** para câmaras RGB; e o `ArraySession` do SDK ganha o `grab()`, `frame_stream()` e `continuous()`. Consulte a [secção LATTICE](lattice/README.md).
* **Exposição automática mais estável.** A exposição mantém-se constante sob luz constante e estabiliza-se poucos segundos após uma alteração; pequenos realces já não escurecem a cena; uma exposição manual mantém o ganho em «Auto»; e uma câmara que deixa de responder sob sol intenso é mantida no nível adequado à sua luz, lido pelo sensor de luz ou, na ausência deste, pelos próprios fotogramas da câmara. Consulte [Definições da câmara](lattice/camera-settings.md).
* **Sensores de luz.** **Suporte para DAQ-E-S:** o seu difusor fixo não requer correção de tampa e cada leitura inclui a inclinação, rotação e ângulo de inclinação da unidade. Uma gravação indica se a tampa foi selecionada ou assumida, e o processamento emite um aviso no caso de uma tampa assumida: o DAQ-U, o DAQ-M e o DAQ-E são fornecidos sem tampa, pelo que deve indicar a tampa instalada. As gravações em tempo real `.daq` sobrevivem a um corte de energia (mantenha um ficheiro `.daq-journal` com o seu `.daq`), e um DAQ-E que deixe de transmitir é reconectado com a lacuna marcada no seu registo. Um projeto que contenha apenas gravações do sensor de luz lista-as e processa-as a partir da aplicação e da CLI, e uma gravação feita com o `chloros_scripts` é listada como o DAQ que a realizou, com um limite que pode definir. Cada captura guarda a leitura do DAQ que utilizou. Consulte a [secção DAQ](daq/README.md).
* **Processamento.**Os quadros LATTICE iniciam o processamento durante a deteção do alvo. As referências do alvo e do DAQ são determinadas por banda. Um projeto movido volta a ligar os seus ficheiros. As exportações de uma captura de 8 bits são de 8 bits, com um novo formato**TIFF (8 bits)**. Uma execução do Survey3 sem alvo de calibração é arquivada como o seu produto alternativo (`Vignette_Corrected_Images` ou `Sensor_Response_Images`), e não como refletância. As etiquetas DLS por banda de uma imagem de refletância são sempre as do seu próprio quadro. A opção «Texture Aware» é recusada para capturas LATTICE, que recebem sempre o debayer padrão.
* **CLI e SDK.** Os caminhos de saída relativos passam agora a ser direcionados para a pasta a partir da qual o comando ou script é executado (um backend instalado costumava resolvê-los na sua própria pasta de instalação e falhava). O backend continua a funcionar entre comandos e scripts e encerra-se automaticamente após 30 minutos sem utilização; o `chloros-cli backend status` e o `backend stop` gerem este comportamento. Os comandos de captura indicam todos os ficheiros que não conseguiram gravar e terminam com o código 1; as capturas repetidas param quando o disco estiver cheio. Uma execução bem-sucedida do `process` gera o relatório `Image products written: N (plus M clamp masks)`. No Windows, o `ChlorosLocal` do SDK / `process_folder` do SDK localizam o backend instalado (e utilizam um que já esteja a ser executado), e o `read_image_audit_tags` funciona a partir de um `pip install` simples.

***

## Novidades no Chloros 1.2.0

* **Controlo de câmara em tempo real — novo separador «Câmaras».** Ligue câmaras LATTICE uma de cada vez ou como conjuntos sincronizados de várias câmaras (sincronização de tempo PTP, captura acionada por hardware), com sobreposições de pré-visualização em tempo real, histogramas por banda, exposição automática inteligente, um calculador de índice em tempo real e atualizações de firmware da câmara diretamente na aplicação.
* **Sensores de luz — novo separador «Sensores de luz».** Ligue sensores DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet); visualize espectros calibrados em tempo real (W/m²/nm), grave ficheiros `.daq` no seu projeto, selecione perfis de correção de cap e atualize o firmware do DAQ-E através da rede.
* **Modos de captura e gravadores.** Captura única / contínua / por intervalo, além de um modo de captura mais rápida apenas em formato bruto; seleção, por projeto, das câmaras e tipos de exportação que a opção «Capturar tudo» produz; gravadores em matriz para vídeo indexado de nível de monitorização e rajadas brutas de nível de análise com compilações de vídeo offline.
* **Pipeline de processamento LATTICE.** Importe pastas de captura LATTICE e divida cada fotograma em bruto em produtos de remoção de bayering, pré-visualização, radianância float32 (W/m²/sr/nm) e refletância, com opções de ativação/desativação por produto. A refletância pode provir de um alvo de calibração no quadro ou de luz descendente do DAQ; o alinhamento da matriz é aplicado às exportações; a calibração de fábrica em falta é descarregada automaticamente através do número de série da câmara.
* **Os projetos memorizam o hardware.** As câmaras e os sensores de luz ligados são guardados com o projeto (`cameras.json` / `sensors.json`) e voltam a ligar-se com as suas definições guardadas quando reabre o projeto. Consulte [GUI: Projetos](projects.md).
* **Atualizações do visualizador de imagens.** Leitura do pixel/índice do cursor com escalonamento correto da refletância por ficheiro, histogramas de camadas, um controlo deslizante de agrupamento GSD, modos de grelha «Por Disparador» / «Por Câmara», visualizações de produtos LATTICE e exportações da área de teste de Índice/LUT para o disco.
* **CLI e SDK, significativamente expandidos.** Novas famílias de comandos `lattice`, `daq pool-*`, `project` e `time-sync`; novas opções `process` (`--input-level`, opções de ativação/desativação por produto, `--reflectance-source`, sinalizadores de alinhamento de matriz); identificadores de ligação inteligente do SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) que iniciam automaticamente o backend; Automação do `open_project()`; o pacote «SDK wheel» está incluído nos instaladores e publicado no PyPI como `chloros-sdk`.
* **Semântica de falha honesta.** Uma execução do `chloros-cli process` que solicitou produtos mas não gravou nenhum falha agora de forma evidente e termina com um valor diferente de zero; as execuções bem-sucedidas indicam quantos produtos de imagem foram gravados.
* **Novo layout de saída.** Os produtos são guardados em pastas `<project>/<camera>/<format>/<Product>_Images/` e mantêm o nome do ficheiro de origem — é a pasta, e não um sufixo do nome do ficheiro, que identifica o produto. Consulte [Formatos de Imagem de Saída](output-image-formats.md).
* **Mais entradas, planos e idiomas.** Suporte à entrada `.dng`; todos os 38 idiomas da interface totalmente implementados; limites de hardware por plano com utilização gratuita (sem login) de até 4 câmaras e 2 sensores de luz.
* **Fiabilidade.** A opção «Parar Processamento» encerra corretamente com um resumo honesto da execução, os projetos com várias câmaras exportam todas as câmaras e as atualizações do instalador já não fazem com que saia da sua conta.

***

O Chloros está disponível em 3 interfaces de aplicação:

## Chloros: Aplicação GUI para ambiente de trabalho

Janela independente com todas as funcionalidades, incluindo os separadores «Câmaras em direto» e «Sensores de luz». _Apenas para Windows._

## [Chloros CLI: Interface de linha de comandos](CLI.md)

Processamento em lote por linha de comandos, além dos comandos em tempo real `lattice`, `daq pool-*`, `project` e `time-sync`. Perfeito para automatização, criação de scripts e operação sem interface gráfica. Disponível em **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _A CLI requer um plano Chloros+ pago para acesso._

## [Chloros API: Python SDK](api-python-sdk.md)

Interface programática em Python para automação e fluxos de trabalho personalizados: processamento de pipeline completo, sessões ao vivo de câmaras/matrizes, sessões de sensores DAQ e automação de projetos guardados. Instalado com o pacote para computador/CLI e também publicado como `pip install chloros-sdk`. _A API requer um plano Chloros+ pago para acesso._

***

## Plataformas suportadas

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Sim | Sim | Sim |
| **Linux amd64 (x86_64)** | Não | Sim | Sim |
| **Linux arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação no Linux, consulte a secção [Linux e Computação de Periferia](linux/linux-overview.md).

***

## Comece em três passos

1. **Instalar** — descarregue e execute o programa de instalação para a sua plataforma. Consulte [Descarregar](download.md).
2. **Inicie sessão (opcional para a GUI)** — a GUI processa imagens gratuitamente sem necessidade de uma conta. Um [login no Chloros+](chloros+-login.md) desbloqueia o processamento paralelo, a aceleração por GPU, limites mais elevados para dispositivos e acesso à CLI e ao SDK.
3. **Crie o seu primeiro projeto** — abra o Chloros, crie um [Novo Projeto](projects.md), [adicione as suas imagens](processing-images-gui/adding-files-to-a-project.md) e [inicie o processamento](processing-images-gui/starting-the-processing.md). Para controlar hardware em tempo real, abra o separador «Câmaras» ou «Sensores de luz» — consulte [GUI: Navegação](navigation.md).

***

## Chloros+

Embora o Chloros seja gratuito para a maioria das tarefas, poderá sentir que precisa de mais funcionalidades. É aí que uma licença paga do Chloros+ pode ser vantajosa para si. Com uma licença do Chloros+, pode desbloquear novas funcionalidades, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos de maior dimensão, processando imagens simultaneamente ao longo do pipeline.
* **Aceleração por GPU (CUDA)**: tire partido das atuais opções de maior memória de GPU para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Chloros+**[**CLI**](CLI.md) **Acesso**: execute o Chloros+ a partir da linha de comandos para automatizar e integrar no seu próprio software. Disponível em qualquer nível de subscrição paga; aplicado do lado do servidor.
* **Chloros+**[**API**](api-python-sdk.md) **Acesso:** execute o Chloros+ a partir de Python para controlo programático, permitindo uma integração perfeita com os seus fluxos de trabalho de investigação, fluxos de trabalho de análise de dados e aplicações personalizadas. Disponível em qualquer plano pago; aplicado do lado do servidor.
* **Limites de hardware mais elevados**: ligue mais câmaras e sensores de luz ao mesmo tempo. Sem necessidade de iniciar sessão, a interface gráfica permite ligar até 4 câmaras e 2 sensores de luz DAQ; os planos pagos aumentam ambos os limites:

| Plano | Câmaras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sem login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilização em vários dispositivos**: cada licença do Chloros+ permite o registo de 2 ou mais dispositivos. Utilize a sua conta MAPIR Cloud para gerir os dispositivos registados. Adicione suporte para mais dispositivos atualizando a sua licença do Chloros+.
* **Método avançado de debayer com reconhecimento de textura:** um debayer de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que elimina praticamente todo o ruído do processo de debayer.
* **Fórmulas personalizadas de índices multiespectrais:** introduza índices multiespectrais personalizados nas calculadoras raster do Chloros, tanto para processamento como para a área de teste de visualização de imagens.
* **Linux e computação de ponta:** execute o Chloros em plataformas Linux x86_64 e ARM64, incluindo o NVIDIA Jetson, para processamento no terreno e de ponta. Consulte a [Visão Geral do Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preços e inscrição no Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
