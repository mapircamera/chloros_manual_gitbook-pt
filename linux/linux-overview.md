# Visão geral do Linux

O Chloros 1.1.0 oferece suporte nativo ao Linux para o **CLI**e o**Python SDK**, permitindo o processamento de imagens multiespectrais sem interface gráfica em estações de trabalho, servidores e dispositivos de borda NVIDIA Jetson.

{% hint style="info" %}
**Sem GUI no Linux.** A GUI do Chloros Desktop está disponível apenas no Windows. Os utilizadores do Linux interagem com o Chloros através do [CLI](../CLI.md) e do [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matriz de suporte de plataformas

| Funcionalidade | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI de ambiente de trabalho** | Sim | N/A | Não | Não |
| **CLI** | Sim | Sim | Sim | Sim |
| **Python SDK** | Sim | Sim | Sim | Sim |
| **Aceleração por GPU (CUDA)** | Sim | Sim | Sim | Sim (JetPack 6) |
| **Desbayering sensível à textura** | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) |
| **Adaptação dinâmica de computação** | Sim | Sim | Sim | Sim |***

## Arquiteturas suportadas

| Arquitetura | Descrição | Método de instalação |
| --- | --- | --- |
| **amd64 (x86_64)** | Processadores padrão para desktop/servidor (Intel, AMD) | Pacote `.deb` |
| **arm64 (aarch64)** | Processadores baseados em ARM, principalmente NVIDIA Jetson | Pacote `.deb` (JetPack 6) |

## Distribuições Linux suportadas

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson)***

## O que os utilizadores do Linux obtêm

* **Chloros CLI** — Interface de linha de comandos completa para processamento em lote, automatização e criação de scripts
* **Chloros Python SDK** — Interface programática Python (`pip install chloros-sdk`) para integração em pipelines de investigação e ferramentas personalizadas
* **Aceleração por GPU** — Processamento acelerado por CUDA em GPUs NVIDIA (desktop e Jetson)
* **Adaptação Dinâmica de Computação** — Detecção automática de hardware e otimização da estratégia de processamento
* **Todas as Funcionalidades de Processamento** — O mesmo pipeline de processamento multiespectral que o Windows (calibração, correção de vinheta, índices de vegetação, todos os formatos de exportação)
* **Funcionalidades do Chloros+** — Processamento multithread, debayer sensível à textura, índices personalizados (com licença Chloros+)

## O que os utilizadores do Linux não têm

* **GUI de ambiente de trabalho** — Sem interface gráfica; toda a interação é feita através do CLI ou do Python SDK
* **Visualizador de imagens** — Sem visualizador de imagens interativo, visualização em grelha ou marcadores de mapa
* **Gestão visual de projetos** — Os projetos são geridos através de comandos CLI e chamadas SDK***

## Introdução ao Linux

1. **Instalar o Chloros** — Consulte [Instalação do Linux](linux-installation.md) para a instalação do pacote `.deb`
2. **Instale o Python SDK** (opcional) — `pip install chloros-sdk`
3. **Ative a sua licença** — `chloros-cli login your@email.com 'password'`
4. **Processe o seu primeiro conjunto de dados** — `chloros-cli process ~/datasets/flight001`

Para utilizadores do NVIDIA Jetson, consulte o [Guia NVIDIA Jetson](nvidia-jetson-guide.md) dedicado para configuração e otimização específicas da plataforma.

***

## Próximos passos

* [Instalação do Linux](linux-installation.md) — Instruções detalhadas de instalação para amd64 e arm64
* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — Configuração específica para Jetson, gestão térmica e implementação no terreno
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
* [API : Python SDK](../api-python-sdk.md) — Referência completa do SDK
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Como o Chloros se adapta ao seu hardware
