---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros

é uma aplicação de software da [MAPIR

](https://www.mapir.camera) destinada ao processamento de imagens multiespectrais, ao controlo em tempo real do hardwMAPIR

e e ao registo de dados dos sensores. A versão 1.2.0 doChloros

suporta toda a família de produtosMAPIR

:

* **CâmarasSurvey3** — processam capturas RAW+JPG, transformando-as em mapas calibrados de refletância e índices de vegetação. Consulte [Câmaras compatíveis](supported-cameras.md).
* **Câmaras LATTICE** — ligue os módulos de câmara multiespectral GigE em tempo real, individualmente ou como conjuntos sincronizados de várias câmaras: pré-visualize, capture e processe em produtos calibrados de radiação e refletância. Consulte a [secção LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — sensores espectrais DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet): espectros calibrados em tempo real, gravações `.daq` e iluminação descendente para processamento de refletância. Consulte a [secção DAQ](daq/README.md).

{% hint style="success" %}
**Novidades noChloros

1.2.0**: controlo em tempo real da câmara e da matriz LATTICE, integração dos sensores de luz DAQ, modos de captura e gravadores, um fluxo de processamento radiométrico LATTICE completo, automatização de projetos a partir doCLI

/SDK

e muito mais. Consulte a lista de novidades abaixo e [Descarregue](download.md) para aceder ao registo de alterações.
{% endhint %}

{% hint style="info" %}
**Está a utilizar oChloros

com um assistente de IA?** Este manual foi concebido para isso. Indique ao seu assistente:

* `https://mapir.gitbook.io/chloros/llms.txt` — índice legível por máquina de todas as páginas.
* Qualquer página em Markdown bruto — acrescente `.md` ao seuURL

(por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência doCLI

](reference/cli-reference.md) e a [Referência doSDK

](reference/sdk-reference.md) — páginas de referência completas, com valores exatos, escritas para consumo por LLM.

Exemplo de prompt: *«Leia https://mapir.gitbook.io/chloros/reference/cli-reference.md, e, em seguida, escreva um script que inicie sessão e processe a pasta ~/flights/flight_001 em ficheiros GeoTIFF de refletância +NDVI

.»*

Guia completo: [UtilizarChloros

com assistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novidades na versão 1.2.0 doChloros



* **Controlo de câmara em tempo real — novo separador «Câmaras».** Ligue câmaras LATTICE uma de cada vez ou como conjuntos sincronizados de várias câmaras (sincronização de tempo PTP, captura acionada por hardware), com sobreposições de pré-visualização em tempo real, histogramas por banda, exposição automática inteligente, um calculador de índice em tempo real e atualizações de firmware da câmara diretamente na aplicação.
* **Sensores de luz — nova separador «Sensores de luz».** Ligue sensores DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet); visualize espectros calibrados em tempo real (W/m²/nm), grave ficheiros `.daq` no seu projeto, selecione perfis de correção de cap e atualize o firmware do DAQ-E através da rede.
* **Modos de captura e gravadores.** Captura única / contínua / por intervalos, além de um modo de «Captura Mais Rápida» apenas para dados brutos; seleção, por projeto, das câmaras e tipos de exportação que a opção «Capturar Tudo» produz; gravadores de matriz para vídeo indexado de nível de monitorização e rajadas de dados brutos de nível de análise com compilações de vídeo offline.
* **Pipeline de processamento LATTICE.** Importe pastas de captura LATTICE e processe cada fotograma bruto separadamente, obtendo produtos de remoção de bayering, pré-visualização, radiação em float32 (W/m²/sr/nm) e refletância, com opções de ativação/desativação por produto. A refletância pode provir de um alvo de calibração no quadro ou de radiação descendente do DAQ; o alinhamento da matriz é aplicado às exportações; a calibração de fábrica em falta é descarregada automaticamente através do número de série da câmara.
* **Os projetos memorizam o hardware.** As câmaras e os sensores de luz ligados são guardados com o projeto (`cameras.json` / `sensors.json`) e voltam a ligar-se com as suas definições guardadas quando reabre o projeto. Consulte [GUI: Projetos](projects.md).
* **Atualizações do visualizador de imagens.** Leitura do pixel/índice do cursor com escala de refletância correta por ficheiro, histogramas de camadas, um controlo deslizante de agrupamento GSD, modos de grelha «Por Disparador» / «Por Câmara», visualizações de produtos LATTICE e exportações da área de teste de Índice/LUT para o disco.
* **Funções «CLI

» e «SDK

», significativamente ampliadas.** Novas famílias de comandos `lattice`, `daq pool-*`, `project` e `time-sync`; novas opções `process` (`--input-level`, opções de ativação/desativação por produto, `--reflectance-source`, sinalizadores de alinhamento de matrizes);SDK

identificadores «smart-connect» (`connect_camera` / `connect_array` / `connect_daq_sensor`) que iniciam automaticamente o backend; Automação do `open_project()`; o «SDK

» está incluído nos instaladores e publicado no PyPI como `chloros-sdk`.
* **Semântica de falha honesta.** Uma execução do `chloros-cli process` que tenha solicitado produtos, mas não tenha gravado nenhum, falha agora de forma evidente e termina com um valor diferente de zero; as execuções bem-sucedidas indicam quantos produtos de imagem foram gravados.
* **Novo layout de saída.** Os produtos são guardados em pastas `<project>/<camera>/<format>/<Product>_Images/` e mantêm o nome do ficheiro de origem — é a pasta, e não um sufixo do nome do ficheiro, que identifica o produto. Consulte [Formatos de imagem de saída](output-image-formats.md).
* **Mais entradas, planos e idiomas.** Suporte para entradas `.dng`; todos os 38 idiomas da interface totalmente implementados; limites de hardware por plano com utilização gratuita (sem login) de até 4 câmaras e 2 sensores de luz.
* **Fiabilidade.** A função «Parar Processamento» encerra corretamente com um resumo honesto da execução, os projetos com várias câmaras exportam todas as câmaras e as atualizações do instalador já não provocam o logout do utilizador.***

OChloros

está disponível em 3 interfaces de aplicação:

##Chloros

: Aplicação GUI para ambiente de trabalho

Janela independente com todas as funcionalidades, incluindo os separadores «Câmaras» e «Sensores de luz» em tempo real. _Apenas para Windows._

## [Chloros

CLI

: Interface de linha de comandos](CLI.md)

Processamento em lote por linha de comandos, além dos comandos em tempo real `lattice`, `daq pool-*`, `project` e `time-sync`. Ideal para automatização, criação de scripts e operação sem interface gráfica. Disponível em **Windows

,Linux

amd64 eLinux

arm64 (NVIDIA Jetson)**. _A CLI requer um plano pagoChloros

+ para acesso._

## [Chloros

API

:Python

SDK

](api-python-sdk.md)

Interface programáticaPython

para automação e fluxos de trabalho personalizados: processamento de pipeline completo, sessões de câmara/matriz em tempo real, sessões de sensores DAQ e automação de projetos guardados. Instalado com o pacote desktop/CLI

e também publicado como `pip install chloros-sdk`. _A API requer um plano pagoChloros

+ para acesso._

***

## Plataformas suportadas

| Plataforma | GUI |CLI

|Python

SDK

|
| --- | --- | --- | --- |
| **Windows

10/11 (x64)** | Sim | Sim | Sim |
| **Linux

amd64 (x86_64)** | Não | Sim | Sim |
| **Linux

arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação nLinux

, consulte a secção [Linux

&amp; Edge Computing](linux/linux-overview.md).

***

## Comece em três passos

1. **Instalar** — descarregue e execute o programa de instalação para a sua plataforma. Consulte [Descarregar](download.md).
2. **Inicie sessão (opcional para a GUI)** — a GUI processa imagens gratuitamente sem necessidade de uma conta. Um [Chloros

+ login](chloros+-login.md) desbloqueia o processamento paralelo, a aceleração por GPU, limites mais elevados de dispositivos e acesso aCLI

/SDK

.
3. **Crie o seu primeiro projeto** — abraChloros

, crie um [Novo Projeto](projects.md), [adicione as suas imagens](processing-images-gui/adding-files-to-a-project.md) e [inicie o processamento](processing-images-gui/starting-the-processing.md). Para controlar hardware em tempo real, abra o separador «Câmaras» ou «Sensores de luz» — consulte [GUI: Navegação](navigation.md).

***

##Chloros

+

Embora oChloros

seja gratuito para a maioria das tarefas, poderá dar-se o caso de precisar de mais funcionalidades. É aí que uma licença paga para oChloros

+ pode ser-lhe útil. Com uma licençaChloros

+, pode desbloquear novas funcionalidades, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos de maior dimensão, processando simultaneamente as imagens ao longo do pipeline.
* **Aceleração por GPU (CUDA)**: tire partido das opções atuais de maior memória de GPU para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Acesso aoChloros

+**[**CLI**](CLI.md): execute oChloros

+ a partir da linha de comandos para automatizar e integrar no seu próprio software. Disponível em qualquer plano pago; aplicado do lado do servidor.
* **Chloros

+**[**API**](api-python-sdk.md) **Acesso:** executeChloros

+ a partir dePython

para controlo programático, permitindo uma integração perfeita com os seus fluxos de trabalho de investigação, fluxos de trabalho de análise de dados e aplicações personalizadas. Disponível em qualquer plano pago; aplicado do lado do servidor.
* **Limites de hardware mais elevados**: ligue mais câmaras e sensores de luz ao mesmo tempo. Sem necessidade de iniciar sessão, a interface gráfica (GUI) liga até 4 câmaras e 2 sensores de luz DAQ; os planos pagos aumentam ambos os limites:

| Plano | Câmaras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sem login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Utilização em vários dispositivos**: cada licençaChloros

+ permite o registo de 2 ou mais dispositivos. Utilize a sua conta na nuvemMAPIR

para gerir os dispositivos registados. Adicione suporte para mais dispositivos atualizando a sua licençaChloros

+.
* **Método avançado de remoção de pixelização com reconhecimento de textura:** um método de remoção de pixelização de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que elimina praticamente todo o ruído resultante da remoção de pixelização.
* **Fórmulas personalizadas de índices multiespectrais:** introduza índices multiespectrais personalizados nas calculadoras de raster doChloros

, tanto para o processamento como para a área de teste de visualização de imagens.
* **Computação de ponta (Linux

) e computação de borda (Edge Computing):** execute oChloros

nas plataformas x86_64 e ARM64 daLinux

, incluindo a NVIDIA Jetson, para processamento no terreno e de ponta. Consulte a [Visão Geral doLinux

](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Preços e Inscrição</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
