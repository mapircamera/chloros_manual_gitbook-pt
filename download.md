---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Transferir

Transfira a versão mais recente do Chloros para começar a trabalhar com o processamento de imagens multiespectrais.

### Requisitos do sistema

#### Windows

| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operativo** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processador**        | Intel Core i5 ou equivalente                          | Intel Core i7 ou superior                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB ou mais                                         |
| **Placa gráfica**    | Compatível com DirectX 11                                | GPU NVIDIA com 4 GB ou mais de VRAM                            |
| **Armazenamento**          | 6 GB de espaço livre                                       | SSD com 10 GB ou mais de espaço livre                            |
| **Ecrã**          | 1920x1080                                            | 2560x1440 ou superior                                  |
| **Internet**         | Necessária para a ativação da licença [opcional] Chloros+ | Necessária para a ativação da licença [opcional] Chloros+ |

#### Linux amd64 (x86\_64)

| Requisito       | Mínimo                    | Recomendado               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa gráfica** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB ou mais de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB+ de espaço livre       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX de 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (partilhada entre GPU e CPU)     | 16 GB ou mais partilhada                    |
| **Armazenamento**      | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Os utilizadores do Chloros+ com GPUs NVIDIA podem utilizar a aceleração CUDA para um processamento significativamente mais rápido. Isto funciona tanto no Windows (GPUs de computador de secretária) como no Linux (GPUs de computador de secretária e NVIDIA Jetson). Os utilizadores do Chloros+ também beneficiam de processamento multithread para obter a máxima velocidade.
{% endhint %}

***

## Descarregar o Chloros

### Versão estável mais recente: Versão 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descarregar o Chloros para o Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descarregar o Chloros para o Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descarregar o Chloros para o Linux arm64 / Jetson (.deb)</a>

#### Instalador do Windows (GUI + CLI + Backend)

* **Tipo de ficheiro**: .exe (Instalador do Windows)**Passos de instalação:**

1. Descarregue o ficheiro .exe acima referido
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (predefinição: `C:\Program Files\MAPIR\Chloros\`)
5. Conclua a instalação e inicie o Chloros ou o Chloros CLI
6. Inicie sessão com a sua [conta MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

{% hint style="success" %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema para acesso à linha de comandos.
{% endhint %}

#### Linux amd64 (Pacote .deb — CLI + Backend)

* **Tipo de ficheiro**: .deb (pacote Debian/Ubuntu)
* **Arquitetura**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (Pacote .deb — CLI + Backend)

* **Tipo de ficheiro**: .deb (JetPack 6)
* **Arquitetura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consulte [Instalação do Linux](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas sobre o Jetson.

#### Python SDK (Todas as plataformas)

Cada instalador inclui um «wheel» `chloros_sdk` correspondente, pelo que a versão SDK corresponde sempre à GUI/CLI/backend instalado. No Windows, o instalador instala-o automaticamente no seu sistema Python; no Linux, o `.deb` coloca o «wheel» em `/usr/lib/chloros/sdk/` e apresenta o comando de instalação:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para hosts que utilizam apenas o pip (sem o pacote Chloros instalado), o SDK também se encontra no PyPI:

```bash
pip install chloros-sdk
```

Consulte [API : Python SDK](api-python-sdk.md) e a [Referência do SDK](reference/sdk-reference.md) para documentação.

{% hint style="info" %}
**Utilizadores do Linux**: O pacote `.deb` instala o CLI e o backend. Não existe uma interface gráfica para o Linux — toda a interação é feita através do CLI ou do SDK.
{% endhint %}

***

## Recursos adicionais

### Python SDK

Para programadores e fluxos de trabalho de automatização, instale o Chloros, o Python e o SDK:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)**Requisitos**: É necessário ter o Chloros instalado (programa de instalação do Windows ou pacote do Linux `.deb`), é necessário o login da licença do Chloros+***

## O que está incluído

### Instalador do Windows

* ✅ **Chloros GUI** - Interface gráfica com todas as funcionalidades
* ✅ **Chloros CLI** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Backend Chloros** - Motor de processamento
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados

### Pacote .deb do Linux

* ✅ **Chloros CLI** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Backend Chloros** - Motor de processamento
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados
* ❌ Sem GUI — o Linux é apenas uma versão sem interface gráfica do CLI/SDK

### Python SDK (pip, todas as plataformas)

* ✅ **Chloros SDK** - Python API (requer licença Chloros+)***

## Atualizar para o Chloros+

Desbloqueie funcionalidades avançadas com uma subscrição do Chloros+:

* 🚀 **Processamento multithread** - Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** - Aproveite a potência da GPU da NVIDIA
* 💻 **Acesso ao CLI** - Automatize com ferramentas de linha de comandos
* 🐍 **Python SDK** - Acesso programático ao API
* 📱 **Vários dispositivos** - Utilização em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de debayering com reconhecimento de textura** - um debayering de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que elimina praticamente todo o ruído do processo de debayering.
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planos e preços do Chloros+</a></p>***

## Ajuda na instalação

### Resolução de problemas

**A instalação falha com a seguinte mensagem de erro:**

* Certifique-se de que possui direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se cumpre os requisitos mínimos do sistema

**A aplicação não inicia (Windows):**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os controladores gráficos
* Verifique o Visualizador de Eventos do Windows para obter detalhes sobre o erro
* Contacte o suporte técnico com os registos de erros

**O CLI não inicia (Linux):**

* Verifique se o pacote `.deb` está instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se faltam bibliotecas: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas de ativação da licença:**

* Certifique-se de que a ligação à Internet está ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se a firewall não está a bloquear o Chloros
* Consulte [Chloros+ Iniciar sessão](chloros+-login.md) para obter instruções detalhadas

### Obter apoio

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)***

## Atualizações de software

O Chloros verifica se existem atualizações, avisa quando está disponível uma nova versão e apresenta um link para esta página de download — a atualização é feita executando o novo instalador assinado. As suas definições e projetos são mantidos após as atualizações. No Linux e no Jetson, o `chloros-cli update` verifica se existe uma versão mais recente e oferece a possibilidade de descarregar e instalar o `.deb` correspondente (este comando é exclusivo do Linux).

***

## Registo de alterações**Versão 1.2.0 (Mais recente)**— consulte**Novidades no Chloros 1.2.0** na página [Introdução](./) para ver a lista completa de funcionalidades.

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novos recursos*** **Método de debayering com reconhecimento de textura \[Apenas Chloros+] —** O Texture Aware utiliza um debayering de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que remove praticamente todo o ruído do processo de debayering.
* **Suporte para alvos de calibração T4P*** **Processamento mais rápido na GPU do Chloros+, melhor gestão de memória**

**Correções de erros*** Interface de utilizador (GUI) totalmente nova, que deverá funcionar agora em todos os computadores Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novas funcionalidades*** **Alternância entre imagem e metadados**: Adicionada uma opção no Navegador de Ficheiros para visualizar os metadados da imagem selecionada numa tabela, em vez da grelha de imagens
* **Controlo deslizante de zoom da grelha de imagens**: Novo controlo deslizante na interface do utilizador para ajustar o tamanho das miniaturas (também suporta CTRL + roda do rato)
* **Botões de exportação da grelha de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Separador «Mapa»**: Novo mapa 2D interativo que mostra marcadores de localização GPS das imagens
  * Suporta o Google Maps e mosaicos de mapas ESRI (seleciona automaticamente o melhor serviço de mosaicos com base na disponibilidade do nível de zoom)
  * Pré-visualização das miniaturas ao passar o rato sobre os marcadores no mapa

**Correções de erros*** Suporte melhorado para a instalação do Chloros em computadores com idiomas diferentes do inglês

</details>

<details>

<summary>Versão 1.0.3</summary>

**Data de lançamento: 20 de dezembro de 2025**

**Novas funcionalidades*** Lançamento inicial

**Melhorias*** Lançamento inicial

**Correções de erros*** Lançamento inicial

**Problemas conhecidos*** Lançamento inicial

</details>***

## Contrato de Licença**Software Proprietário** - Copyright (c) 2026 MAPIR Inc.

É proibida a utilização, distribuição ou modificação não autorizadas.

**Versão Gratuita**: Disponível para uso pessoal e comercial com limitações de funcionalidades**Chloros+**: Licença baseada em subscrição para funcionalidades avançadas e implementações comerciais
