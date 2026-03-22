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
| **Internet**         | Necessária para a ativação da licença \[opcional] Chloros+ | Necessária para a ativação da licença \[opcional] Chloros+ |

#### Linux amd64 (x86\_64)

| Requisito       | Mínimo                    | Recomendado               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa gráfica** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB+ de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB+ de espaço livre       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (GPU/CPU partilhada)         | 16 GB+ partilhada                    |
| **Armazenamento**      | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Os utilizadores do Chloros+ com GPUs NVIDIA podem utilizar a aceleração CUDA para um processamento significativamente mais rápido. Isto funciona tanto no Windows (GPUs de secretária) como no Linux (GPUs de secretária e NVIDIA Jetson). Os utilizadores do Chloros+ também beneficiam de processamento multithread para velocidade máxima.
{% endhint %}

***

## Descarregar Chloros

### Versão estável mais recente (23 de março de 2026): Versão 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descarregar o Chloros para o Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descarregar o Chloros para o Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descarregar Chloros para Linux arm64 / Jetson (.deb)</a>

#### Instalador Windows (GUI + CLI + Backend)

* **Tipo de ficheiro**: .exe (Instalador Windows)**Passos de instalação:**

1. Descarregue o ficheiro .exe acima
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\[USER]\Chloros\`)
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

Consulte [Instalação do Linux](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas sobre o Jetson.

#### Python SDK (Todas as plataformas)

```bash
pip install chloros-sdk
```

Consulte [API : Python SDK](api-python-sdk.md) para obter documentação.

{% hint style="info" %}
**Utilizadores do Linux**: O pacote `.deb` instala o CLI e o backend. O Python SDK é instalado separadamente através do pip. Não existe GUI para o Linux — toda a interação é feita através do CLI ou do SDK.
{% endhint %}

***

## Recursos adicionais

### Python SDK

Para programadores e fluxos de trabalho de automatização, instale o Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)**Requisitos**: O Chloros deve estar instalado (instalador do Windows ou pacote do Linux `.deb`), é necessário o login da licença do Chloros+***

## O que está incluído

### Instalador Windows

* ✅ **Chloros GUI** - Interface gráfica com todas as funcionalidades
* ✅ **Chloros CLI** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Chloros Backend** - Motor de processamento
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados

### Pacote .deb Linux

* ✅ **Chloros CLI** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Backend Chloros** - Motor de processamento
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados
* ❌ Sem GUI — O Linux é apenas CLI/SDK sem interface gráfica

### Python SDK (pip, todas as plataformas)

* ✅ **Chloros SDK** - Python API (requer licença Chloros+)***

## Atualizar para o Chloros+

Desbloqueie funcionalidades avançadas com uma subscrição do Chloros+:

* 🚀 **Processamento multithread** - Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** - Aproveite o poder da GPU NVIDIA
* 💻 **Acesso CLI** - Automatize com ferramentas de linha de comando
* 🐍 **Python SDK** - Acesso programático API
* 📱 **Vários dispositivos** - Utilize em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de debayer com reconhecimento de texturas** - um debayer de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que elimina quase todo o ruído do processo de debayer.
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planos e preços do Chloros+</a></p>***

## Ajuda na instalação

### Resolução de problemas

**A instalação falha com a mensagem de erro:**

* Certifique-se de que possui direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se cumpre os requisitos mínimos do sistema

**A aplicação não inicia (Windows):**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os controladores gráficos
* Verifique o Visualizador de Eventos do Windows para obter detalhes do erro
* Contacte o suporte com os registos de erros

**O CLI não inicia (Linux):**

* Verifique se o pacote `.deb` está instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se faltam bibliotecas: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas com a ativação da licença:**

* Certifique-se de que a ligação à Internet está ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se a firewall não está a bloquear o Chloros
* Consulte [Chloros+ Login](chloros+-login.md) para obter instruções detalhadas

### Obter assistência

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **FAQ**: [Perguntas frequentes](faq.md)***

## Registo de alterações

<details>

<summary>Versão 1.1.0 (Mais recente)</summary>

**Data de lançamento: março de 2026**

**Novas funcionalidades*** **Suporte a Linux** — CLI e SDK nativos para Linux amd64 (x86\_64) e arm64 (NVIDIA Jetson JetPack 6). Instale através dos pacotes `.deb`.
* **Suporte para NVIDIA Jetson** — Processamento otimizado para dispositivos de borda Jetson Nano, Orin Nano, Orin NX e AGX Orin.
* **Adaptação dinâmica de computação** — Detecção automática de hardware e otimização da estratégia de processamento. O Chloros adapta-se ao seu hardware, desde um Jetson Nano até uma estação de trabalho com várias GPUs.
* **Pipeline de processamento de 4 threads** — Threads simultâneas de deteção, calibração, processamento e exportação com alocação dinâmica de memória da GPU.
* **Novos comandos CLI** — `selftest` (diagnóstico do sistema) e `update` (gestão de atualizações do Linux).
* **Novos sinalizadores de processo CLI** — `--debayer` (padrão/sensível à textura), `--indices` (especificar índices), `--target` (procurar primeiro na subpasta de destino para uma deteção mais rápida).
* **Novos itens do menu da GUI** — Adicionar ficheiros, Adicionar pasta e Iniciar/Parar processamento agora acessíveis a partir do menu suspenso principal.**Melhorias**

* Detecção automática de backend multiplataforma (caminhos Windows e Linux)
* SDK e `get_status()` melhorados com acompanhamento do progresso por thread
* Novas exceções SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gestão térmica e limitação adaptativa para NVIDIA Jetson
* Gestão automática da memória com fallback OOM para processamento GPU em mosaico

</details>

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novas funcionalidades*** **Método de debayer com reconhecimento de textura \[Apenas Chloros+] -** O Texture Aware utiliza um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do debayer.
* **Suporte para alvos de calibração T4P*** **Processamento GPU Chloros+ mais rápido, melhor gestão de memória**

**Correções de erros*** Interface de utilizador (GUI) completamente nova, que deverá funcionar agora em todos os computadores Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novas funcionalidades*** **Alternância entre imagem e metadados**: Adicionada alternância no Navegador de ficheiros para visualizar os metadados da imagem selecionada numa tabela em vez da grelha de imagens
* **Controlo deslizante de zoom da grelha de imagens**: Novo controlo deslizante na interface do utilizador para ajustar o tamanho das miniaturas (também suporta CTRL + roda do rato)
* **Botões de exportação da grelha de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Separador Mapa**: Novo mapa 2D interativo que mostra marcadores de localização GPS das imagens
  * Suporta mosaicos de mapas do Google Maps e da ESRI (seleciona automaticamente o melhor serviço de mosaicos com base na disponibilidade do nível de zoom)
  * Pré-visualização das miniaturas ao passar o rato sobre os marcadores do mapa

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

**Versão gratuita**: Disponível para uso pessoal e comercial com limitações de funcionalidades**Chloros+**: Licença por assinatura para funcionalidades avançadas e implementações comerciais
