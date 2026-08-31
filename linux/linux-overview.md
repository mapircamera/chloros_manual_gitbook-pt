# Visão geral do Linux

O Chloros 1.2.0 oferece suporte nativo ao Linux para o **CLI**e o**Python SDK** — processamento de imagens multiespectrais sem interface gráfica, além do controlo em tempo real de câmaras LATTICE e de sensores de luz DAQ — em estações de trabalho, servidores e dispositivos de borda NVIDIA Jetson.

{% hint style="info" %}
**Não existe interface gráfica de utilizador (GUI) no Linux.**A GUI do Chloros é exclusiva do Windows. Os utilizadores do Linux interagem com o Chloros através do [CLI](../CLI.md) e [Python SDK](../api-python-sdk.md). O `.deb` adiciona, de facto, uma**Chloros CLI** ao menu da sua aplicação — limita-se a abrir um emulador de terminal a executar o `chloros-cli`.
{% endhint %}

***

## Matriz de suporte de plataformas

| Funcionalidade | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI de ambiente de trabalho** | Sim | N/A | Não | Não |
| **CLI** (`chloros-cli`) | Sim | Sim | Sim | Sim |
| **Python SDK** (`chloros-sdk`) | Sim | Sim | Sim | Sim |
| **Pipeline de processamento de imagem** | Sim | Sim | Sim | Sim |
| **Controlo da câmara LATTICE (em tempo real)** | Sim (separador «Câmaras») | Sim (`chloros-cli lattice`, SDK) | Sim | Sim |
| **Sensores de luz DAQ (em tempo real)** | Sim (separador «Sensores de luz») | Sim (`chloros-cli daq pool-*`, SDK) | Sim | Sim |
| **Sincronização de tempo PTP (o anfitrião é o «grandmaster»)** | Sim | Sim (`chloros-cli time-sync`) | Sim | Sim |
| **Aceleração por GPU (CUDA)** | Sim | Sim | Sim | Sim (JetPack 6) |
| **Debayer com reconhecimento de textura** | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) |
| **Adaptação dinâmica de computação** | Sim | Sim | Sim | Sim |
| **Backend como serviço do sistema** (`chloros-backend.service`) | Não | Não | Sim (opcional) | Sim (opcional) |
| **Atualizador no local** (`chloros-cli update`) | Não (execute o instalador) | Não (execute o instalador) | Sim | Sim |***

## Arquiteturas suportadas

| Arquitetura | Descrição | Pacote |
| --- | --- | --- |
| **amd64 (x86_64)** | Processadores padrão para computadores de secretária/servidores (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processadores ARM — família NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (compilação do JetPack 6) |

## Distribuições Linux suportadas

* **Ubuntu 22.04 LTS ou mais recente** (amd64)
* **Debian 12 ou mais recente** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson Orin)***

## O que os utilizadores do Linux obtêm

* **Chloros CLI** — a interface de linha de comandos completa para processamento em lote, automatização e criação de scripts
* **Chloros Python SDK** — interface programática do Python para pipelines de investigação e ferramentas personalizadas (instalável a partir do PyPI e também incluída no `.deb` como um wheel com versão compatível)
* **Controlo de câmaras LATTICE** — descobrir, ligar, configurar e capturar imagens a partir de câmaras LATTICE e conjuntos sincronizados de várias câmaras através do `chloros-cli lattice` e do SDK; o `.deb` inclui o ambiente de execução Arena SDK necessário para as câmaras
* **Controlo de sensores de luz DAQ** — ligue sensores DAQ-U/M/E, transmita espectros calibrados e grave ficheiros `.daq` através do `chloros-cli daq pool-*` e do SDK
* **Sincronização de tempo PTP** — o backend Chloros executa o «grandmaster» PTP ao qual as câmaras LATTICE e os sensores DAQ-E estão subordinados; verifique-o com o `chloros-cli time-sync`, e mantenha-o a funcionar em modo headless com a unidade systemd `chloros-backend.service` (consulte [Instalação do Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatização de projetos** — execute projetos guardados em modo sem interface gráfica com o `chloros-cli project` e o `open_project` do SDK
* **Aceleração por GPU** — processamento acelerado por CUDA em GPUs NVIDIA (desktop e Jetson)
* **Adaptação dinâmica de computação** — deteção automática de hardware e seleção da estratégia de processamento, com a opção de substituição do `CHLOROS_STRATEGY` como uma via de fuga para especialistas
* **Todas as funcionalidades de processamento** — o mesmo pipeline que o Windows: calibração, correção de vinheta, índices de vegetação e todos os formatos de exportação
* **Funcionalidades do Chloros+** — processamento multithread (em pipeline), debayer com reconhecimento de textura e índices personalizados, com um plano pago Chloros+

## O que os utilizadores do Linux não têm

* **Interface gráfica de utilizador (GUI) para computador** — sem interface gráfica; toda a interação é feita através do CLI ou do Python SDK
* **Visualizador de imagens** — sem visualizador de imagens interativo, vista em grelha ou marcadores no mapa
* **Gestão visual de projetos** — os projetos são criados e geridos através de comandos do CLI e chamadas do SDK (o próprio hardware — câmaras, sensores, captura — continua a ser totalmente controlável a partir do terminal)***

## Requisitos de licença

O acesso a CLI e SDK requer um **nível Chloros+ pago — Copper ou superior**(Copper, Bronze, Silver, Gold). O nível gratuito**Iron** não tem acesso a CLI/SDK. Este limite mínimo é imposto pelo backend, não apenas pelo CLI:

| Situação | Resposta do backend |
| --- | --- |
| Não está com sessão iniciada | `401` com `error_code: AUTH_REQUIRED` |
| Iniciada sessão no nível Iron gratuito | `403` com `error_code: PLAN_UPGRADE_REQUIRED` |

O `chloros-cli status` funciona em qualquer nível — é a única rota isenta do controlo de acesso — pelo que o motivo da recusa está sempre visível.

***

## Introdução ao Linux

1. **Instale o Chloros** — consulte [Instalação do Linux](linux-installation.md) para a instalação do `.deb`
2. **Verifique** — o `chloros-cli --version` imprime o `Chloros CLI 1.2.0`; o `chloros-cli selftest` executa o diagnóstico de 7 passos
3. **Instale o Python SDK** (opcional) — `pip install chloros-sdk`
4. **Inicie sessão** — `chloros-cli login your@email.com 'your-password'` (uma vez por máquina e novamente após cada atualização do pacote)
5. **Processe o seu primeiro conjunto de dados** — `chloros-cli process ~/datasets/flight001`

Para o NVIDIA Jetson, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) dedicado para obter informações sobre a configuração específica da plataforma, o comportamento térmico e a implementação no terreno.

***

## Próximos passos

* [Instalação do Linux](linux-installation.md) — instalação detalhada, localizações de ficheiros e resolução de problemas para amd64 e arm64
* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — configuração específica do Jetson, comportamento térmico e da memória, implementação no terreno
* [CLI : Linha de comandos](../CLI.md) — o guia CLI
* [API : Python SDK](../api-python-sdk.md) — o guia SDK
* [Referência do CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas exaustivas de comandos/API para a versão 1.2.0
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware

{% hint style="info" %}
**Ler este manual programaticamente.** Cada página também está disponível em formato Markdown bruto na sua própria URL mais `.md` (por exemplo, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), e um índice de todo o manual está publicado em [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
