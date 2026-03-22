---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>O Chloros é uma aplicação de software da [MAPIR](https://www.mapir.camera) para processar imagens e outros dados de sensores.

***{% hint style="success" %}**Novidades no Chloros 1.1.0**: Suporte nativo para Linux (amd64 e arm64), computação de ponta NVIDIA Jetson, Adaptação Dinâmica de Computação, pipeline de processamento de 4 threads, novos comandos e opções do CLI. Consulte [Download](download.md) para ver o registo completo de alterações.
{% endhint %}

O Chloros está disponível em 3 modos de aplicação:

## Chloros: Aplicação GUI para ambiente de trabalho

Janela separada independente com todas as funcionalidades. _Apenas para Windows._

## [Chloros CLI: Interface de linha de comandos](CLI.md)

Processamento em lote na linha de comandos. Perfeito para automatização, criação de scripts e operação sem interface gráfica. Disponível em **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _A CLI requer uma licença Chloros+ para acesso._

## [Chloros API: Python SDK](api-python-sdk.md)

Interface programática Python para automação e fluxos de trabalho personalizados. Ideal para pipelines de investigação, integração com aplicações Python existentes e criação de ferramentas personalizadas. Disponível em **todas as plataformas** através do `pip install chloros-sdk`. _A API requer uma licença Chloros+ para acesso._***

## Plataformas suportadas

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Sim | Sim | Sim |
| **Linux amd64 (x86_64)** | Não | Sim | Sim |
| **Linux arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação do Linux, consulte a secção [Linux e Computação de Borda](linux/linux-overview.md).

***

## Chloros+

Embora o Chloros seja gratuito para a maioria das tarefas, poderá sentir que precisa de mais. É aí que uma licença paga para o Chloros+ pode ser vantajosa para si. Com uma licença Chloros+, pode desbloquear novas funcionalidades, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos de maior dimensão, processando imagens simultaneamente através do pipeline.
* **Aceleração por GPU (CUDA)**: tire partido das atuais opções de memória de GPU de maior capacidade para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Chloros+**[**CLI**](CLI.md)**Acesso**: execute o Chloros+ a partir da linha de comandos para automatizar e integrar no seu próprio software.
* **Chloros+**[**API**](api-python-sdk.md)**Acesso:** execute o Chloros+ a partir do Python para controlo programático, permitindo uma integração perfeita com os seus pipelines de investigação, fluxos de trabalho de análise de dados e aplicações personalizadas.
* **Utilização em vários dispositivos**: cada licença Chloros+ permite o registo de 2 ou mais dispositivos. Utilize a sua conta MAPIR Cloud para gerir os dispositivos registados. Adicione suporte para mais dispositivos atualizando a sua licença Chloros+.
* **Método avançado de debayer com reconhecimento de textura:** um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído de IA/ML que elimina quase todo o ruído do processo de debayering. 
* **Fórmulas personalizadas de índices multiespectrais:** introduza índices multiespectrais personalizados nas calculadoras raster do Chloros, tanto para processamento como para a área de teste de visualização de imagens.
* **Linux e computação de ponta:** execute o Chloros nas plataformas Linux x86_64 e ARM64, incluindo o NVIDIA Jetson, para processamento no terreno e de ponta. Consulte [Visão geral do Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preços e inscrição no Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
