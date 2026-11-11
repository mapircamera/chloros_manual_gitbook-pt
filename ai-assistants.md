# Utilizar o Chloros com assistentes de IA

Este manual destina-se a dois públicos: os seres humanos e os assistentes de IA com os quais os seres humanos trabalham cada vez mais. Cada página apresenta valores exatos, predefinições e comandos que podem ser copiados e colados, para que um assistente (Claude, ChatGPT, Copilot, um agente de programação, etc.) possa escrever uma automação funcional do Chloros à primeira tentativa.

Versão do Chloros: **

1.3.0**. Plataformas CLI/SDK: Windows 10/11 x64 e Linux (x86\_64 / Jetson aarch64).

## O que fornecer ao seu assistente

| Recurso | URL | Para que serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legível por máquina de todas as páginas deste manual. |
| **Referência da CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A superfície de comandos completa do `chloros-cli`: todos os comandos, sinalizadores, valores por predefinição, códigos de saída e regras relativas à pasta de saída. Escrita para utilização por LLM. |
| **Referência do SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | A Python API completa do `chloros_sdk`: classes, assinaturas, exceções e exemplos práticos. Escrito para utilização por LLM. |
| **Qualquer página em Markdown bruto** | acrescente `.md` ao URL da página | por exemplo, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` devolve a página em Markdown bruto — ideal para colar numa janela de contexto ou para obter a partir de um agente. |

Ligações no manual: [Referência da CLI](reference/cli-reference.md) · [Referência do SDK](reference/sdk-reference.md).

{% hint style="info" %}
As duas páginas de referência são autónomas: um assistente que tenha lido uma delas não precisa do resto do manual para escrever um script correto.
{% endhint %}

## Deixe o seu assistente executar o Chloros diretamente: `chloros-mcp`

Tudo o que foi referido acima implica que o seu assistente *escreva* um script ou um comando. Com o `chloros-mcp`, um assistente que utilize o [Model Context Protocol](https://modelcontextprotocol.io) (MCP), como o Claude Desktop, o Claude Code ou o Cursor, controla o próprio Chloros. Basta pedir em linguagem simples e ele aciona as ferramentas do servidor:

* **Processar imagens:** criar um projeto, importar uma pasta de capturas, escolher definições, processar e apresentar um relatório do que a execução exportou.
* **Operar o hardware:** ligar câmaras LATTICE, matrizes e sensores de luz DAQ, pré-visualizar, definir a exposição e a cor, capturar, gravar e atualizar o firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

As ferramentas do servidor abrangem o que o [Python SDK](api-python-sdk.md) faz. É instalado juntamente com o SDK, pelo que normalmente não há mais nada para instalar: no Windows, o instalador do Chloros coloca o `chloros-mcp.exe` na pasta `Scripts` do Python, e, no Linux, o `.deb` coloca o `chloros-mcp` em `/usr/bin`.

**Código do Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**adicione isto a `claude_desktop_config.json` (**Definições &gt; Desenvolvedor &gt; Editar Configuração**abre-o) e, em seguida, reinicie o Claude Desktop. O**cursor** recebe a mesma entrada em `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Se o cliente indicar que o servidor não conseguiu iniciar, é provável que o `chloros-mcp` não conste no `PATH` que o cliente deteta: indique o caminho completo em vez disso (`where chloros-mcp` no Windows, `which chloros-mcp` no Linux).

É necessário ter o Chloros instalado no mesmo computador e ter efetuado o início de sessão nesse computador, na aplicação ou através do `chloros-cli login`, num plano pago do Chloros+ (Copper ou superior). O servidor não pode iniciar sessão por si.

{% hint style="info" %}
Configuração do cliente para cada aplicação, opções, todas as ferramentas e como uma execução se comporta: [Servidor MCP (Assistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants) na Referência do SDK.
{% endhint %}

## Receitas de prompts

Copie, preencha o `<placeholders>` e cole no seu assistente.

### 1. Processar uma pasta de voos para NDVI

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

### 3. Ligar uma matriz LATTICE e efetuar a captura

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
A criação de scripts para o DAQ a partir da linha de comandos passa sempre pela família `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Outros subcomandos `daq` que o seu assistente possa inventar não estão disponíveis nas versões fornecidas e resultam num erro.
{% endhint %}

## Por que razão os scripts escritos por IA funcionam bem com o Chloros

Cada um destes é um comportamento real e verificado do Chloros 1.3.0 — eliminam os modos de falha clássicos da automação escrita por máquinas:

* **Sem complicações de configuração.**Os auxiliares de ligação inteligente do SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e os pontos de entrada de processamento (`ChlorosLocal`, `process_folder`)**iniciam automaticamente o backend local**. Um script gerado não necessita que a GUI esteja aberta nem de um servidor iniciado manualmente — basta que o pacote para ambiente de trabalho ou CLI esteja instalado.
* **Todo o pipeline é uma única chamada.** O `chloros_sdk.process_folder("path", indices=["NDVI"])` executa a importação → calibração → refletância → exportação do índice de ponta a ponta. Menos área de superfície, menos pontos em que um script gerado pode falhar.
* **As execuções sem resultados realizam um autodiagnóstico.** Após o `process()`, o resumo da execução é anexado ao resultado, e todas as indicações de processamento (por exemplo, *por que razão* uma execução não produziu resultados) é igualmente reemitida como um `UserWarning` em Python — pelo que mesmo um script que nunca inspecione o dicionário de resultados apresenta o diagnóstico.
* **A CLI falha de forma evidente.**Uma execução `chloros-cli process` que solicitou produtos mas não escreveu nenhum imprime `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, pelo que os scripts de shell e a CI a detetam com uma simples verificação do código de saída. As execuções bem-sucedidas reportam o código `Image products written: N`, além do código `(plus M clamp masks)` quando foram gravadas máscaras. Os comandos de captura também indicam todos os ficheiros que não conseguiram gravar e terminam com o código `1`.
* **Os caminhos relativos têm o significado que o script espera.** Com um backend local, um caminho de saída relativo (`-o out/` na CLI, `output_dir="out/"` no SDK) é resolvido em relação à pasta a partir da qual o comando ou script é executado.

Uma assimetria que um assistente deve conhecer: o `process()` do SDK **não** gera deliberadamente uma exceção numa execução sem resultados — em vez disso, reporta-a através do resumo/dicas. Se um pipeline em Python tiver de parar numa execução vazia, verifique o resumo (a receita 2 faz isso).

## Advertências

* **É necessário um login no Chloros+.**A CLI e o SDK requerem um nível**pago** do Chloros+, aplicado do lado do servidor: os pedidos falham com o erro `401 AUTH_REQUIRED` quando não se está com sessão iniciada e com o erro `403 PLAN_UPGRADE_REQUIRED` no nível gratuito. Execute o comando `chloros-cli login` uma vez por máquina antes de executar os scripts gerados. Consulte [Início de sessão no Chloros+](chloros+-login.md).
* **Os comandos de captura controlam hardware real.** `lattice` / `daq` / `project`, os objetos de sessão do SDK e as ferramentas do `chloros-mcp` estabelecem ligação, transmitem e acionam câmaras e sensores físicos. Reveja um script gerado antes da sua primeira execução e execute-o com alguém a supervisionar o hardware.
* **Verifique aleatoriamente os resultados.** Verifique as pastas do produto e alguns valores de píxeis antes de publicar os resultados. Em particular, os ficheiros TIFF de refletância são dimensionados por fonte — leia a etiqueta XMP do `Chloros:PixelScale` (os ficheiros LATTICE incluem-na: 32768 = 1,0 de refletância; os ficheiros Survey3 não a incluem e utilizam 65535 = 1,0) em vez de assumir um divisor. Ambas as páginas de referência documentam isto na secção «Leitura de píxeis de refletância».
* **Pequenas armadilhas que causam erros no código gerado:**o `pool-record` grava no sistema de ficheiros do**host do backend** (padrão: `~/Documents/DAQ Live View/`); em máquinas com várias interfaces de rede, prefira o `daq pool-connect --eth-host <ip-or-hostname>` em vez da deteção automática; utilize o `http://127.0.0.1:5000` (nunca o `localhost`) sempre que aparecer um URL de backend; um script que percorra a árvore de saída de um projeto deve ignorar as pastas `clamp_masks/` (os seus ficheiros `*_clampmask.tif` são máscaras, não imagens — ver [Formatos de Imagem de Saída](output-image-formats.md#clamp-masks)); e no Windows, com os caminhos longos desativados, mantenha as pastas do projeto curtas — uma pasta de projeto com mais de cerca de 130 caracteres é rejeitada antes do início do processamento.
