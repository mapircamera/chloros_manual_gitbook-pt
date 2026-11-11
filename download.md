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
| **Internet**         | Necessária para a ativação da licença [opcional] do Chloros+ | Necessária para a ativação da licença [opcional] do Chloros+ |

#### Linux amd64 (x86_64)

| Requisitos       | Mínimos                    | Recomendados               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa gráfica** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB ou mais de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB ou mais de espaço livre       |
| **Python**        | Python 3.7+ (para o SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisitos      | Mínimos                      | Recomendados                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX de 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (partilhada entre GPU e CPU)         | 16 GB ou mais partilhada                    |
| **Armazenamento**      | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       | Python 3.7+ (para o SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Os utilizadores do Chloros+ com GPUs NVIDIA podem utilizar a aceleração CUDA para um processamento significativamente mais rápido. Isto funciona com GPUs de secretária da NVIDIA no Windows e no Linux amd64, bem como no NVIDIA Jetson com o JetPack 6. Os utilizadores do Chloros+ também beneficiam de processamento multithread para obter a máxima velocidade.
{% endhint %}

***

## Descarregar o Chloros


### Versão estável mais recente: Versão 1.3.0 (11 de novembro de 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Descarregar o Chloros para Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Descarregar o Chloros para Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Descarregar o Chloros para Linux arm64 / Jetson (.deb)</a>


#### Instalador do Windows (GUI + CLI + Backend)

* **Tipo de ficheiro**: .exe (Instalador do Windows)

**Passos de instalação:**

1. Descarregue o ficheiro .exe acima indicado
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\Chloros\`)
5. Na página **Selecionar tarefas adicionais**, mantenha ambas as opções de**Configurações do sistema** marcadas (recomendado — ver abaixo)
6. Conclua a instalação e inicie o Chloros ou o Chloros CLI
7. Inicie sessão com a sua [conta MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

**Opções de configurações do sistema** (ambas marcadas por predefinição):

| Opção, tal como aparece no instalador | O que faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** (permite que as pastas de projeto tenham nomes com mais de 260 caracteres; recomendado) | Define a configuração `LongPathsEnabled` do Windows, válida para todo o sistema, para 1. O Windows vem com esta opção desativada, pelo que nenhum caminho de ficheiro pode ter mais de 260 caracteres; os produtos Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto, pelo que, com a opção de caminhos longos desativada, uma pasta de projeto mais profunda é recusada. Entra em vigor para programas iniciados após a instalação. Permanece ativada após a desinstalação, uma vez que outros programas podem depender dela. | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender** (captura de câmara muito mais rápida; recomendado) | Adiciona uma exclusão de processo do Microsoft Defender para o `chloros-backend.exe` na pasta de instalação, para que a proteção em tempo real não verifique todos os ficheiros que o backend escreve, independentemente do local onde os escreve. A taxas de captura elevadas, a verificação consumia cerca de 30% da taxa de captura (77,7 contra 52,7 capturas por segundo numa captura contínua com 9 câmaras). Nenhum outro programa é excluído, e a exclusão é removida aquando da desinstalação. Quando o Defender está ausente ou bloqueado por política, este passo falha silenciosamente e a instalação prossegue. | `/MERGETASKS="!defenderexclusion"` |

Uma instalação silenciosa (`/SILENT` ou `/VERYSILENT`) aplica ambas as opções, a menos que opte por não as aplicar com o valor `/MERGETASKS` apresentado.

{% hint style="success" %}
O instalador adiciona a pasta CLI (`C:\Program Files\Chloros\cli\`) ao seu sistema `PATH` uma única vez — a reinstalação ou atualização já não acrescenta uma entrada duplicada e remove as duplicatas adicionadas por instaladores anteriores. Abra um novo terminal após a instalação para utilizar o `chloros-cli`. O instalador também adiciona duas regras do Firewall do Windows para o backend, em todos os perfis de rede: **Backend Chloros (UDP-In)**, para fluxos de câmara, PTP, mDNS e o fluxo de dados DAQ-E, e**Backend Chloros (TCP-In)**, limitado à sub-rede local, para uma atualização do firmware do DAQ-E, na qual o sensor se liga novamente a este computador para recolher a imagem. Ambas são removidas durante a desinstalação.
{% endhint %}

#### Linux amd64 (Pacote .deb — CLI + Backend)

* **Tipo de ficheiro**: .deb (pacote Debian/Ubuntu)
* **Arquitetura**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (pacote .deb — CLI + Backend)

* **Tipo de ficheiro**: .deb (JetPack 6)
* **Arquitetura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Utilize o nome do ficheiro que descarregou efetivamente e mantenha o `./` para que o `apt` instale esse ficheiro em vez de procurar um pacote pelo nome. Instale com o `apt install`, e não com o `dpkg -i`: o apt também instala os pré-requisitos do Python SDK, que o pacote recomenda em vez de exigir, e o `dpkg -i` ignora-os. Desde a versão 1.3.0, o backend e a CLI são instalados descompactados no `/usr/lib/chloros/`, pelo que são executados sem necessidade de se descompactarem primeiro.

O pacote Jetson utiliza a compilação CUDA do PyTorch do JetPack 6, pelo que o processing, incluindo o debayer Texture Aware, é executado na GPU do Jetson. Utiliza as bibliotecas CUDA e cuDNN que o JetPack 6 instala (`sudo apt install nvidia-jetpack` numa imagem que não as possua).

Consulte [Instalação no Linux](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas do Jetson.

#### Python SDK (Todas as plataformas)

Cada instalador inclui um «wheel» `chloros_sdk` correspondente, pelo que a versão do SDK corresponde sempre à GUI/CLI/backend instalada. No Windows, o instalador instala-o automaticamente no Python do sistema. No Linux, o `.deb` instala o SDK para o próprio `python3` do sistema (no `/usr/lib/python3/dist-packages`), pelo que o `import chloros_sdk` funciona logo após a instalação, sem necessidade do pip nem de ligação à rede. Apenas um virtualenv, que não tem acesso aos pacotes do sistema, necessita que o wheel incluído seja instalado nele:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para hosts que utilizam apenas o pip (sem o pacote Chloros instalado; o SDK, nesse caso, controla um backend noutro computador), o SDK também está disponível no PyPI:

```bash
pip install chloros-sdk
```

Consulte [API: Python SDK](api-python-sdk.md) e a [Referência do SDK](reference/sdk-reference.md) para obter documentação.

{% hint style="info" %}
**Utilizadores de Linux**: O pacote `.deb` instala a CLI, o backend e o SDK para Python. Não existe uma GUI para Linux — toda a interação é feita através da CLI ou do SDK.
{% endhint %}

***

## Recursos adicionais

### SDK Python para Chloros


Para programadores e fluxos de trabalho de automatização, o Chloros Python SDK para Chloros vem incluído em todos os instaladores: o instalador do Windows instala-o no seu Python, e o `.deb` do Linux instala-o para o sistema `python3`. Numa máquina sem o pacote Chloros (uma que controla um backend noutra máquina), instale-o a partir do PyPI:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)

**Requisitos**: um backend do Chloros para controlar (o instalador do Windows ou o pacote `.deb` para Linux instala um) e um login do Chloros+ num plano pago

***

## O que está incluído

### Instalador do Windows


* ✅ **GUI do Chloros** - Interface gráfica com todas as funcionalidades
* ✅ **CLI do Chloros** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Backend do Chloros** - Motor de processamento
* ✅ **Python SDK** — instalado no seu Python, caso este esteja presente (requer licença Chloros+)
* ✅ **Perfis de câmara** — modelos de câmara MAPIR pré-configurados

### Pacote .deb para Linux


* ✅ **Chloros CLI** - Interface de linha de comandos (requer licença Chloros+)
* ✅ **Chloros Backend** - Motor de processamento
* ✅ **Python SDK** - instalado para o sistema `python3` (requer licença Chloros+)
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados
* ❌ Sem GUI — o Linux é apenas CLI/SDK sem interface gráfica

### Python SDK (PyPI, para máquinas sem um pacote Chloros)

* ✅ **Chloros SDK** - Python API (requer licença Chloros+)

***

## Atualizar para o Chloros+

Desbloqueie funcionalidades avançadas com uma subscrição do Chloros+:

* 🚀 **Processamento multithread** — Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** - Aproveite a potência da GPU da NVIDIA
* 💻 **Acesso à CLI** - Automatize com ferramentas de linha de comandos
* 🐍 **Python SDK** - Acesso programático à API

* 📱 **Vários dispositivos** - Utilize em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de remoção de pixelização com reconhecimento de textura** - um método de remoção de pixelização de alta qualidade, sensível aos contornos, combinado com um modelo de redução de ruído baseado em IA/ML que elimina praticamente todo o ruído resultante da remoção de pixelização.
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planos e preços do Chloros+</a></p>***

## Ajuda na instalação

### Resolução de problemas

**A instalação falha com a seguinte mensagem de erro:**

* Certifique-se de que possui direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se cumpre os requisitos mínimos do sistema

**A aplicação não inicia (em Windows):**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os controladores gráficos
* Verifique o Visualizador de Eventos do Windows para obter detalhes do erro
* Contacte o suporte técnico com os registos de erros

**A CLI não inicia (Linux):**

* Verifique se o pacote `.deb` está instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se faltam bibliotecas: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (a partir da versão 1.3.0, o `/usr/lib/chloros/chloros-backend` é um pequeno script de lançamento; o programa propriamente dito encontra-se no `/usr/lib/chloros/backend/`)

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
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)***

## Atualizações de software O

Chloros verifica se há atualizações, anuncia quando uma nova versão está disponível e fornece um link para esta página de download — a atualização é feita executando o novo instalador assinado. As suas configurações, projetos e dados de início de sessão são mantidos após as atualizações. No Linux e no Jetson, o comando ``chloros-cli update`` verifica se existe uma versão mais recente e oferece a possibilidade de descarregar e instalar a versão correspondente ``.deb`` (este comando é exclusivo do Linux).

***

## Registo de alterações**Versão 1.3.0 (Mais recente, 11 de novembro de 2026)**— consulte**Novidades no Chloros

1.3.0** na página [Introdução](./) para ver a lista completa.**Versão 1.2.0**— consulte**Novidades no Chloros 1.2.0** na página [Introdução](./).

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novas funcionalidades*** **Método de debayer com reconhecimento de textura \[Apenas Chloros+] -** O Texture Aware utiliza um debayer de alta qualidade com reconhecimento de contornos, combinado com um modelo de redução de ruído baseado em IA/ML que remove praticamente todo o ruído do processo de debayer.
* **Suporte para alvos de calibração T4P*** **Processamento mais rápido do Chloros+ na GPU, melhor gestão de memória**

**Correções de erros*** Interface de utilizador (GUI) completamente nova, que deverá agora funcionar em todos os computadores com Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novas funcionalidades*** **Alternância entre imagem e metadados**: Adicionada uma opção no Navegador de Ficheiros para visualizar os metadados da imagem selecionada numa tabela, em vez da grelha de imagens
* **Controlo deslizante de zoom da grelha de imagens**: Novo controlo deslizante na interface do utilizador para ajustar o tamanho das miniaturas (também suporta CTRL + roda do rato)
* **Botões de exportação da grelha de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Separador «Mapa»**: Novo mapa 2D interativo que mostra marcadores de localização GPS das imagens
  * Suporta o Google Maps e os mosaicos de mapas ESRI (seleciona automaticamente o melhor serviço de mosaicos com base na disponibilidade do nível de zoom)
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

</details>

***

## Contrato de Licença**Software Proprietário** - Copyright (c) 2026 MAPIR Inc.

É proibida a utilização, distribuição ou modificação não autorizadas.

**Versão gratuita**: Disponível para uso pessoal e comercial com limitações de funcionalidades**Chloros+**: Licença baseada em subscrição para funcionalidades avançadas e implementações comerciais
