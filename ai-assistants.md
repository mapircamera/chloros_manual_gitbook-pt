# Utilização do Chloros com assistentes de IA

Este manual destina-se a dois públicos: os seres humanos e os assistentes de IA com os quais os seres humanos trabalham cada vez mais. Cada página apresenta valores exatos, predefinições e comandos que podem ser copiados e colados, para que um assistente (Claude, ChatGPT, Copilot, um agente de programação, etc.) consiga criar uma automação Chloros funcional logo à primeira tentativa.

Versão do Chloros: **

1.2.0**. Plataformas CLI/SDK: Windows 10/11 x64 e Linux (x86_64 / Jetson aarch64).

## O que entregar ao seu assistente

| Recurso | URL | Para que serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legível por máquina de todas as páginas deste manual. |
| **Referência CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A superfície de comandos completa do `chloros-cli`: todos os comandos, sinalizadores, valores por predefinição, códigos de saída e regras relativas à pasta de saída. Escrito para utilização por LLM. |
| **Referência do SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | O `chloros_sdk` Python API completo: classes, assinaturas, exceções e exemplos práticos. Escrito para consumo por LLM. |
| **Qualquer página como Markdown bruto** | acrescente `.md` à página URL | por exemplo, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` devolve a página como Markdown bruto — ideal para colar numa janela de contexto ou para recuperar a partir de um agente. |

Ligações no manual: [Referência CLI](reference/cli-reference.md) · [Referência do SDK](reference/sdk-reference.md).

{% hint style="info" %}
As duas páginas de referência são autónomas: um assistente que tenha lido uma delas não precisa do resto do manual para escrever um script correto.
{% endhint %}

## Receitas de prompt

Copie, preencha o `<placeholders>` e cole no seu assistente.

### 1. Processar uma pasta de voos no NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Monitorizar em lote um diretório de capturas

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Ligar uma matriz LATTICE e capturar

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registar espectros do sensor de luz DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
A criação de scripts DAQ a partir da linha de comandos passa sempre pela família `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Outros subcomandos `daq` que o seu assistente possa inventar não estão disponíveis nas versões fornecidas e resultam num erro.
{% endhint %}

## Por que razão os scripts escritos por IA funcionam bem com o Chloros

Cada um destes é um comportamento real e verificado do Chloros 1.2.0 — eliminam os modos de falha clássicos da automação escrita por máquinas:

* **Sem complicações de configuração.**Os auxiliares de ligação inteligente do SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e os pontos de entrada de processamento (`ChlorosLocal`, `process_folder`)**iniciam automaticamente o backend local**. Um script gerado não necessita que a GUI esteja aberta nem de um servidor iniciado manualmente — basta que o pacote desktop/CLI esteja instalado.
* **Todo o pipeline consiste numa única chamada.** O `chloros_sdk.process_folder("path", indices=["NDVI"])` executa a importação → calibração → refletância → exportação do índice de ponta a ponta. Menos área de superfície, menos pontos em que um script gerado pode falhar.
* **As execuções sem resultados realizam um autodiagnóstico.** Após o `process()`, o resumo da execução é anexado ao resultado, e todas as indicações de processamento (por exemplo, *por que razão* uma execução não produziu resultados) é também reemitida como um Python `UserWarning` — assim, mesmo um script que nunca inspecione o dicionário de resultados apresenta o diagnóstico.
* **O CLI falha de forma evidente.**Uma execução `chloros-cli process` que solicitou produtos mas não escreveu nenhum imprime `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, pelo que os scripts de shell e a CI o detetam através de uma simples verificação do código de saída. As execuções bem-sucedidas apresentam o código `Image products written: N`.

Uma assimetria que um assistente deve conhecer: o `process()` do SDK **não** gera deliberadamente uma exceção numa execução sem produtos — em vez disso, reporta através do resumo/dicas. Se um pipeline Python tiver de parar numa execução vazia, verifique o resumo (a receita 2 faz isso).

## Advertências

* **É necessário o login no Chloros+.**O CLI e o SDK requerem um nível**pago** Chloros+, aplicada do lado do servidor: os pedidos falham com o código de erro `401 AUTH_REQUIRED` se não estiveres autenticado e com o código `403 PLAN_UPGRADE_REQUIRED` no plano gratuito. Execute o `chloros-cli login` uma vez por máquina antes de executar os scripts gerados. Consulte [Chloros+ Início de sessão](chloros+-login.md).
* **Os comandos de captura controlam hardware real.** Os comandos `lattice` / `daq` / `project` e os objetos de sessão SDK estabelecem ligação, transmitem e acionam câmaras e sensores físicos. Analise um script gerado antes da sua primeira execução e execute-o com o hardware supervisionado.
* **Verifique aleatoriamente os resultados.** Verifique as pastas do produto e alguns valores de píxeis antes de publicar os resultados. Em particular, os ficheiros TIFF de refletância são dimensionados por fonte — leia a etiqueta XMP do `Chloros:PixelScale` (LATTICE: 32768 = 1,0 de refletância; Survey3: 65535) em vez de assumir um divisor. Ambas as páginas de referência documentam isto na secção «Leitura de píxeis de refletância».
* **Pequenas armadilhas que causam erros no código gerado:**o `pool-record` grava no sistema de ficheiros do**host do backend** (padrão: `~/Documents/DAQ Live View/`); em máquinas com várias interfaces de rede, prefira o `daq pool-connect --eth-host <ip-or-hostname>` em vez da deteção automática; e utilize o `http://127.0.0.1:5000` (nunca o `localhost`) sempre que aparecer um backend URL.
