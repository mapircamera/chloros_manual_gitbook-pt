# Definições e modos de captura

A captura no separador «Câmaras» é controlada por um botão vermelho **Capturar tudo**e por um painel**Definições de captura** que determina o que esse botão produz: quais as câmaras que participam, quais os tipos de exportação que cada câmara guarda e se o obturador dispara uma vez, continuamente ou a intervalos. Esta página documenta todo o fluxo — a configuração, a captura propriamente dita, o local onde os ficheiros são guardados no disco e como os reprocessar posteriormente para obter produtos calibrados. Os controlos relativos às câmaras e aos conjuntos de câmaras encontram-se em [Definições da Câmara](camera-settings.md).

{% hint style="info" %}
**As capturas requerem um projeto aberto.** As opções «Capturar tudo» e a roda dentada das «Definições de captura» ficam desativadas até que um projeto esteja aberto («Crie ou abra um projeto para guardar as capturas»). Cada captura é guardada na pasta do projeto em `captures/`.
{% endhint %}

## O painel **Configurações de Captura**Abra-o com a**roda dentada ao lado de «Capturar Tudo»**na lista de câmaras da barra lateral ou com o botão**«Abrir Configurações de Captura…»** na parte inferior de qualquer painel de configurações específico de cada câmara. O cabeçalho exibe «Definições de captura» com um botão ← para voltar.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

As suas seleções aqui — câmaras incluídas, caixas de seleção por tipo e o modo de captura — são guardadas **por projeto** e restauradas quando o reabrir.

### Modos de captura

Três botões de modo na parte superior do painel:

| Modo | O que faz | Subconfigurações (padrões) |
| --- | --- | --- |
| **Única** *(padrão)* | Uma captura em todas as câmaras selecionadas. | — |
| **Contínuo**| Capturas consecutivas até uma condição de paragem. | Parar por**Número de capturas** (padrão 1) *ou* **Duração da captura** (padrão 10 s; unidades: segundos / minutos / horas / dias). |
| **Intervalo**(timelapse) | Série de capturas com temporizador. |**Capturas / intervalo**(padrão 1) ·**A cada**N unidades (padrão 5 s) ·**Durante** N unidades (padrão 1 m). |

No modo Contínuo ou Intervalo, o botão «Capturar tudo» transforma-se num botão «**Parar (N)**» durante a execução, contando as capturas à medida que estas são efetuadas.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Escolha de câmaras e tipos de exportação

O texto de ajuda do painel resume a situação: escolha quais as câmaras e os tipos de exportação que a função «Capturar tudo» produz — tudo está ativado por predefinição e as escolhas são guardadas com este projeto.

* Os botões **Selecionar tudo / Não selecionar nada** ativam ou desativam de uma só vez a caixa de seleção de inclusão de todas as câmaras.
* **Botões de alternância em massa para tipos de exportação**(duas linhas de botões):**Tudo em Raw / Tudo sem debayering / Toda a pré-visualização / Toda a radiação / Toda a refletância / Todo o índice**. Cada um apresenta três estados de cor: verde ✓ = ativado para todas as câmaras que o suportam, âmbar – = ativado para algumas, cinzento = nenhuma. Um botão fica desativado quando nenhuma câmara ligada suporta esse tipo. Todos ficam a cinzento enquanto a «Captura mais rápida» estiver ativada.
* **Linhas por câmara**: uma caixa de seleção «Incluir», além de uma lista expansível (▸/▾) dos tipos de exportação aplicáveis a essa câmara, com caixas de seleção individuais. A linha mostra uma contagem de ativação, como «4/6».

### Tipos de exportação e quais as câmaras que os suportam

Existem seis tipos de exportação: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Apenas os tipos aplicáveis aparecem na linha de cada câmara:

| Tipo de exportação | Conteúdo |RGB

(FRGB) | Multiespectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico de Bayer (mono: a única banda) diretamente do sensor | ✓ | ✓ | ✓ |
| **Sem Bayer** | Desmosaico linear (mono: escala de cinzentos de 1 canal) | ✓ | ✓ | ✓ |
| **Pré-visualização** | Cadeia de exibição completa (equilíbrio de brancos + gama de acordo com o perfil da câmara; multiespectral: alongamento em cores falsas) | ✓ | ✓ | ✓ |
| **Radiação** | float32 W/m²/sr/nm através da cadeia radiométrica completa | — (não disponível) | ✓ | ✓ |
| **Refletância** | uint16 ρ (32768 = 1,0) | — (não disponível) | ✓ — apenas apresentado quando a câmara possui um sensor de luz DAQ (próprio ou herdado da sua matriz) | igual ao multiespectral |
| **Índice** | Representação do índice de vegetação (LUT) | — | ✓ — requer uma expressão de índice ativada e não vazia na câmara, e não está disponível para membros de matrizes combinadas (a matriz possui um índice partilhado) | — (um índice requer ≥2 bandas; consulte [Câmaras monocromáticas e índices de vegetação](mono-indices.md)) |

A radianância e a refletância nunca estão disponíveis para câmaras dRGB

o — a radianância por pixel Bayer não é significativa para um sensor fotométrico de banda larga.

### Captura mais rápida

O botão de alternância **⚡ Captura mais rápida — apenas raw**(laranja quando ativado) substitui todas as seleções de exportação por**apenas raw** — além de uma composição gratuita de índice combinado para matrizes — para que o fotograma seja guardado o mais rapidamente possível: os cálculos de radiação/refletância/visualização são totalmente ignorados no momento da captura.

{% hint style="info" %}
**Um `.daq` continua a ser guardado.** Quando é atribuído um sensor de luz, a Captura Mais Rápida continua a registar a leitura de radiação descendente do DAQ junto aos fotogramas em formato raw — para que os produtos de radiação, refletância e índice possam ser todos criados posteriormente através de um novo processamento (ver [Reprocessamento de capturas](#re-processing-captures-into-calibrated-products)). O Fastest Capture também não altera as suas seleções nas caixas de seleção: desative-o e estas voltam ao normal.
{% endhint %}

### Controlos por matriz

Cada matriz ligada tem o seu próprio cartão de grupo no painel:

* **Caixa de seleção «Incluir»** (três estados para todos os membros) e o nome da matriz com o seu modo de exibição: «(combinado | separado)».
* Caixa de seleção **Alinhado**(padrão**ativado**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, para que as exportações fiquem alinhadas em termos de píxeis entre as câmaras. Os ficheiros Raw permanecem não distorcidos, mas contêm a transformação nos seus metadados. (O próprio perfil é calculado no [painel de definições da matriz](camera-settings.md#alignment-co-registration-combined-only).)
* As linhas das câmaras dos membros estão aninhadas dentro do cartão.

O cartão da matriz também aloja dois gravadores. Pense neles como **monitorização vs. análise**:

| Gravador | Nível | O que grava |
| --- | --- | --- |
| **● Gravar vídeo de índice combinado / ■ Parar a gravação** *(apenas matrizes combinadas)* | **Monitorização** | A composição ao vivo do índice combinado em vídeo a 10 fps — 8 bits, resolução de pré-visualização, LUT incorporada. Requer um projeto aberto e uma visualização em direto por streaming. Mostra fotogramas + tempo decorrido durante a gravação. |
| **⦿ Gravar rajada em bruto / ■ Parar sequência bruta** *(qualquer matriz)* | **Análise**| Fotogramas Bayer brutos à taxa de captura ao vivo (sem processamento), juntamente com um manifesto por fotograma e leituras `.daq`, convertidos para `captures/bursts/`. Após uma sequência, surge um botão**Criar vídeo**: este reprocessa a sequência offline, transformando-a num vídeo calibrado — índice combinado e/ou radiação / refletância / índice por câmara — além de ficheiros TIFF opcionais. A criação do índice combinado inicia-se automaticamente quando interrompe a sequência. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## O fluxo

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

«Capturar tudo» Clique em **Capturar tudo** na lista de câmaras da barra lateral:

1. Todas as câmaras incluídas, visíveis e não pausadas capturam com os tipos de exportação selecionados. **Os conjuntos disparam como um único disparador sincronizado** (um único grupo sincronizado entre todos os membros — ver [Conjuntos de várias câmaras](arrays.md)); as câmaras autónomas captam individualmente.
2. As câmaras ocultas (olho) ou em pausa são ignoradas. Uma matriz só fica totalmente bloqueada quando *todos* os seus membros estão ocultos ou em pausa.
3. Sempre que um sensor de luz for atribuído, a leitura correspondente do DAQ de radiação descendente é guardada como um ficheiro `.daq` juntamente com as imagens — mesmo para capturas apenas em formato bruto — para que os produtos radiométricos possam sempre ser derivados posteriormente.
4. O botão mostra o progresso em tempo real — «A capturar… concluído/total» — e, no modo Contínuo/Intervalo, passa a ser **Parar (N)**. Cada item de captura tem um tempo limite de 300 s.
5. Quando a passagem termina, um aviso de resultado indica **«N ficheiros guardados»**ou**«N guardados, F falhou»**, além de «(S ocultos/em pausa ignorados)» quando as câmaras foram ignoradas.

## Onde as capturas são guardadas

As capturas são guardadas no projeto aberto em `<project>/captures/`. Cada tipo de exportação é guardado na sua **própria subpasta**, pelo que uma captura multinível nunca mistura tipos:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` é o carimbo de data/hora da captura e `<serial>` o número de série da câmara. As capturas autónomas são denominadas `capture_<ts>_SN<serial>_<level>`; as capturas em série a partir de um disparador sincronizado são denominadas `sync_<ts>_SN<serial>_<level>` e **partilham um único carimbo de data/hora entre todas as câmaras do grupo** (o sufixo do nível é omitido quando uma câmara guarda apenas um único nível).
* **Uma assimetria a ter em conta:** o nível de visualização é armazenado numa pasta denominada `preview/`, enquanto os ficheiros mantêm `_display` no nome — a pasta e o sufixo diferem apenas para esse nível.
* Os níveis desconhecidos são guardados numa pasta com o seu próprio nome; se não for possível criar uma subpasta, o ficheiro é gravado na raiz das capturas, em vez de se perder.
* Os ficheiros TIFF de captura são comprimidos sem perdas (DEFLATE) por predefinição e contêm todos os seus metadados de calibração e processamento **dentro do ficheiro XMP** — as capturas são autodescritivas, sem ficheiros complementares além do ficheiro `.daq`.

Este é o mesmo esquema que o `chloros-cli lattice capture` / `array-capture` que escrevem no seu diretório `-o` — documentado na [Referência doCLI

§ Como é uma pasta de capturas](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocessamento de capturas em produtos calibrados

Os fotogramas brutos capturados, juntamente com o ficheiro `.daq` guardado, são tudo o que o pipeline de processamento necessita — é por isso que o Fastest Capture é seguro para trabalho real.

* **GUI**: adicione a pasta de capturas a um projeto ([Adicionar ficheiros a um projeto](../processing-images-gui/adding-files-to-a-project.md)) e processe como habitualmente.
* **CLI**: aponte o `process` para a **raiz das capturas**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

O `process` normalmente importa apenas a pasta que indicar, mas quando essa pasta não contém imagens e possui subpastas, percorre-as automaticamente — assim, as subpastas de cada nível e os ficheiros da raiz `.daq` são recolhidos de uma só vez. Cada captura é importada como uma **única imagem**, com os seus outros níveis anexados como modos de visualização, e não como uma imagem por nível.

Nomear diretamente uma subpasta de nível (por exemplo, `…/captures/raw/`) também funciona, mas deixa para trás os ficheiros raiz `.daq` — copie-os em simultâneo quando voltar a derivar um produto radiométrico a partir de `raw/`; caso contrário, a correspondência de data e hora não terá nada com que se resolver.

{% hint style="warning" %}
**O processamento começa sempre a partir de `raw`.** Dentro de cada captura, o fotograma bruto é a fonte do pipeline; `debayered`, `radiance`, `reflectance`, e `preview` surgem como modos visualizáveis, mas nunca são reenviados através do pipeline — o reprocessamento de um produto derivado reaplicaria a vinheta, a cor e os cálculos de radiação já incorporados nos seus píxeis, pelo que oChloros

recusa-se a fazê-lo em vez de efetuar um processamento duplo. As renderizações `index/` e `composite/` nunca são processadas (são saídas, não capturas). Uma pasta de capturas guardada **sem** importações em formato RAW é apresentada normalmente, mas o `process` ignora-a e indica isso mesmo; o `--input-level {raw,debayered,processed}` é a via de escape deliberada que força um ponto de entrada. Consulte a [Referência doCLI

](../reference/cli-reference.md#what-a-captures-folder-looks-like) para ver as mensagens exatas de omissão.
{% endhint %}

Mais dois comportamentos que vale a pena conhecer ao criar scripts de reprocessamento:

* Uma execução `chloros-cli process` que tenha solicitado produtos, mas que **não tenha gerado nenhum produto de imagem, falha de forma evidente e termina com um valor diferente de zero** — nunca obterá uma execução vazia silenciosa. As execuções bem-sucedidas comunicam o número de produtos. (Uma execução deliberada apenas com metadados continua a ser considerada um sucesso.)
* As exportações processadas reimportadas nunca ocupam o slot de dados brutos de uma captura — os dados brutos originais permanecem sempre como fonte do pipeline.

## Equivalentes doCLI



Tudo nesta página pode ser executado em modo headless. Os modos de captura da GUI correspondem diretamente ao `chloros-cli lattice array-capture`:

| GUI |CLI

|
| --- | --- |
| Única | `chloros-cli lattice array-capture` |
| Contínua | `array-capture --continuous [--count N] [--duration S]` |
| Por intervalo | `array-capture --interval S [--duration S]` |
| Captura mais rápida | `array-capture --fastest` |
| Caixa de seleção «Alinhado» | `--aligned / --no-aligned` |
| Caixas de seleção do tipo de exportação | `--processing LEVEL` ou `--levels L1,L2,…` (padrão `all`) |
| Gravar vídeo de índice | `chloros-cli lattice array-record` |
| Gravar sequência em formato bruto / Criar vídeo | `chloros-cli lattice array-burst` / `array-build-video` |

As tabelas completas de sinalizadores, a opção de captura estabilizada com AE inteligente (`--smart`) e o modelo de taxa sustentada encontram-se na [Referência doCLI

, § Modos de Captura, Gravadores e Reprocessamento Offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
