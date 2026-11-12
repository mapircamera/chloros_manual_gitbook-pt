# Visão geral do Linux

O Chloros 1.3.0 oferece suporte nativo ao Linux para a **CLI**e o**Python SDK** — processamento de imagens multiespectrais sem interface gráfica, além do controlo em tempo real de câmaras LATTICE e sensores de luz DAQ — em estações de trabalho, servidores e dispositivos de borda NVIDIA Jetson com Linux.

{% hint style="info" %}
**Não existe GUI de ambiente de trabalho no Linux.**A GUI do Chloros Desktop está disponível apenas para Windows. Os utilizadores do Linux interagem com o Chloros através da [CLI](../CLI.md) e do [Python SDK](../api-python-sdk.md). O `.deb` adiciona, de facto, uma entrada**Chloros CLI** ao menu de aplicações — limita-se a abrir um emulador de terminal que executa o `chloros-cli`.
{% endhint %}

***

## Matriz de suporte de plataformas

| Funcionalidade | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI de ambiente de trabalho** | Sim | N/A | Não | Não |
| **Interface de linha de comando (CLI)** (`chloros-cli`) | Sim | Sim | Sim | Sim |
| **SDK Python para Python** (`chloros-sdk`) | Sim | Sim | Sim | Sim |
| **Pipeline de processamento de imagem** | Sim | Sim | Sim | Sim |
| **Controlo da câmara LATTICE (em tempo real)** | Sim (separador «Câmaras») | Sim (`chloros-cli lattice`, SDK) | Sim | Sim |
| **Sensores de luz DAQ (em tempo real)** | Sim (separador «Sensores de luz») | Sim (`chloros-cli daq pool-*`, SDK) | Sim | Sim |
| **Sincronização temporal PTP (o anfitrião é o «grandmaster»)** | Sim | Sim (`chloros-cli time-sync`) | Sim | Sim |
| **Aceleração por GPU (CUDA)** | Sim | Sim | Sim | Sim (JetPack 6) |
| **Desbayering com reconhecimento de textura** (requer uma GPU CUDA) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) |
| **Adaptação dinâmica de computação** | Sim | Sim | Sim | Sim |
| **Backend como serviço do sistema** (`chloros-backend@<user>.service`) | Não | Não | Sim (opcional) | Sim (opcional) |
| **Atualizador no local** (`chloros-cli update`) | Não (executar o instalador) | Não (executar o instalador) | Sim | Sim |

***

## Arquiteturas suportadas

| Arquitetura | Descrição | Pacote |
| --- | --- | --- |
| **amd64 (x86_64)** | Processadores padrão para computadores de secretária/servidores (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processadores ARM — família NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribuições Linux suportadas

* **Ubuntu 22.04 LTS ou mais recente** (amd64)
* **Debian 12 ou mais recente** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson Orin)

***

## O que os utilizadores do Linux obtêm

* **Chloros CLI** — a interface de linha de comandos completa para processamento em lote, automatização e criação de scripts
* **Chloros Python SDK** — interface programática em Python para pipelines de investigação e ferramentas personalizadas, instalada pelo `.deb` para o sistema `python3` (uma roda com versão compatível para virtualenvs encontra-se no `/usr/lib/chloros/sdk/`; o PyPI destina-se a máquinas sem um pacote Chloros)
* **Controlo de câmaras LATTICE** — detetar, ligar, configurar e capturar imagens a partir de câmaras LATTICE e conjuntos sincronizados de várias câmaras através do `chloros-cli lattice` e do SDK; o `.deb` inclui o ambiente de execução do SDK Arena necessário para as câmaras
* **Controlo de sensores de luz DAQ** — ligue sensores DAQ-U/M/E, transmita espectros calibrados e grave ficheiros `.daq` através do `chloros-cli daq pool-*` e do SDK
* **Sincronização de tempo PTP** — o backend do Chloros executa o «grandmaster» PTP ao qual as câmaras LATTICE e os sensores DAQ-E estão subordinados; inspecione-o com o `chloros-cli time-sync` e mantenha-o a funcionar em modo headless com a unidade systemd `chloros-backend@<user>.service` (consulte [Instalação em Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatização de projetos** — execute projetos guardados em modo headless com o `chloros-cli project` e o `open_project` do SDK
* **Aceleração por GPU** — processamento acelerado por CUDA em GPUs NVIDIA (GPUs de computador de secretária em amd64 e a GPU do Jetson com o JetPack 6)
* **Adaptação dinâmica de computação** — deteção automática de hardware e seleção da estratégia de processamento, com a opção de substituição `CHLOROS_STRATEGY` como uma via de escape para especialistas
* **Todas as funcionalidades de processamento** — o mesmo pipeline que no Windows: calibração, correção de vinheta, índices de vegetação e todos os formatos de exportação
* **Funcionalidades do Chloros+** — processamento multithread (em pipeline), debayer com reconhecimento de textura (numa GPU CUDA) e índices personalizados, com um plano Chloros+ pago

## O que os utilizadores de Linux não têm

* **GUI de ambiente de trabalho** — sem interface gráfica; toda a interação é feita através da CLI ou do Python SDK
* **Visualizador de imagens** — sem visualizador de imagens interativo, vista em grelha ou marcadores de mapa
* **Gestão visual de projetos** — os projetos são criados e geridos através de comandos de CLI e chamadas de SDK (o próprio hardware — câmaras, sensores, captura — continua a ser totalmente controlável a partir do terminal)

***

## Requisitos de licença

O acesso à CLI e ao SDK requer um **nível Chloros+ pago — Copper ou superior**(Copper, Bronze, Silver, Gold). O nível gratuito**Iron** não tem acesso à CLI/SDK. Esta restrição é aplicada pelo backend, não apenas pela CLI:

| Situação | Resposta do backend |
| --- | --- |
| Não está com sessão iniciada | `401` com `error_code: AUTH_REQUIRED` |
| Iniciada sessão no nível gratuito Iron | `403` com `error_code: PLAN_UPGRADE_REQUIRED` |

O `chloros-cli status` funciona em qualquer nível — é a única rota isenta do controlo — pelo que o motivo da recusa é sempre visível.

***

## Introdução no Linux

1. **Instale o Chloros** — consulte [Instalação no Linux](linux-installation.md) para a instalação do `.deb`
2. **Verifique** — o `chloros-cli --version` apresenta o `Chloros CLI 1.3.0`; o `chloros-cli selftest` executa o diagnóstico de 7 passos
3. **Verifique o SDK Python** — o `.deb` instala-o: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (num virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Inicie sessão** — `chloros-cli login your@email.com 'your-password'` (uma vez por máquina; as atualizações do pacote mantêm o início de sessão; a remoção do pacote apaga-o)
5. **Processe o seu primeiro conjunto de dados** — `chloros-cli process ~/datasets/flight001`

Para o NVIDIA Jetson, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) dedicado para obter informações sobre a configuração específica da plataforma, o comportamento térmico e a implementação no terreno.

***

## Próximos passos

* [Instalação no Linux](linux-installation.md) — instalação detalhada, localização de ficheiros e resolução de problemas para amd64 e arm64
* [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) — configuração específica para o Jetson, comportamento térmico e da memória, e implementação no terreno
* [CLI: Linha de Comandos](../CLI.md) — o guia da CLI
* [API: Python SDK](../api-python-sdk.md) — o guia do SDK
* [Referência da CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas exaustivas de comandos/API para a versão 1.3.0
* [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware

{% hint style="info" %}
**Ler este manual programaticamente.** Cada página é também disponibilizada como Markdown bruto na sua própria URL, acrescida de `.md` (por exemplo, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), e um índice de todo o manual está publicado em [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
