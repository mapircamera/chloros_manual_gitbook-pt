# Matriz de câmaras múltiplas

Uma **matriz**LATTICE consiste em duas ou mais câmaras LATTICE ligadas como uma única unidade sincronizada. Uma das câmaras é a**mestre**: esta emite um impulso de disparo GPIO de hardware numa linha de sincronização partilhada (por predefinição, a**Linha 2**), de modo a que todas as câmaras registem o mesmo instante. O Chloros adiciona sincronização temporal PTP, uma pré-visualização em tempo real (mosaicos por câmara ou uma única imagem composta multibanda alinhada), e captura sincronizada — cada passagem de captura produz um**grupo de fotogramas** no qual todas as câmaras partilham o mesmo carimbo de data/hora e ID de fotograma (indicado como `fid:N` na saída de captura).

As matrizes são a forma como as câmaras mono (M3M) produzem índices de vegetação — uma câmara contribui com uma banda, e a matriz alinha-as numa pilha multibanda. Consulte [Câmaras Mono e Índices de Vegetação](mono-indices.md).

Existem três formas equivalentes de ligar uma matriz, e todas elas executam o mesmo fluxo «smart-prep»:

| Superfície | Ponto de entrada |
| --- | --- |
| GUI | Separador «Câmaras» → **Ligar Matriz** (botão azul) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primeiro número de série = mestre) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primeiro número de série = mestre) |

O Smart-prep executa, por ordem: um teste de capacidade de rede (ping ICMP DF + teste GVSP), seleção do nível de sincronização, redução automática do tamanho do quadro para se adaptar à ligação, ativação do PTP, seleção automática do formato de píxeis por câmara, inicialização automática da exposição a partir do estado guardado de cada câmara e configuração do disparador GPIO na Linha 2.

{% hint style="info" %}
As câmaras têm de estar acessíveis na ligação antes de qualquer uma destas funções funcionar — consulte [Ligar Câmaras](connecting.md) para obter informações sobre a deteção, o endereçamento e o download da calibração na primeira ligação. Para configurações com váriascâmaras, as definições do anel de receção da placa de rede do anfitrião são tão importantes quanto a velocidade da ligação; a tabela completa de sintomas→soluções encontra-se na [Referência § Configuração e Ajuste da Placa de Rede do Anfitrião](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## A caixa de diálogo «Array Connect»

O separador «Câmaras» → **«Connect Array»**abre um assistente de três passos:**«Selecionar» → «Modo de exibição» → «Definições»**.

### Passo 1 — Selecionar mestre e escravos

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

A caixa de diálogo analisa a rede assim que é aberta («A analisar a rede...») e, em seguida, verifica a ligação do disparador GPIO («A verificar a ligação GPIO...»). São necessárias, pelo menos, **2 câmaras** para criar um conjunto.

A verificação da ligação preenche automaticamente a seleção de funções sempre que possível e apresenta uma das três mensagens seguintes:

| Mensagem | Significado |
| --- | --- |
| «Mestre GPIO detetado — seleções pré-preenchidas» (verde) | A verificação encontrou a topologia de disparo; as caixas de seleção «mestre» e «escravo» já estão preenchidas. |
| «Nenhum mestre detetado — verifique o cabo GPIO» (laranja) | Nenhuma câmara detetou um pulso de disparo; verifique a cablagem de sincronização. Ainda é possível selecionar as funções manualmente. |
| «Sem cabo de sincronização: {números de série}» (laranja) | As câmaras listadas não têm nenhum cabo de sincronização ligado. |

A tabela de câmaras tem as colunas **Câmara / Número de série / IP / Mestre (rádio) / Escravo (caixa de seleção)**:

* Selecione exatamente **um mestre**e**um ou mais escravos**. Clicar novamente no botão de opção do mestre atual desmarca-o.
* Uma câmara marcada com **«Sem cabo de sincronização»** nunca pode ser selecionada como escrava — uma escrava sem ligação de disparo ficaria à espera na linha de sincronização para sempre e transmitiria um sinal inativo. Ligue essa câmara como uma câmara autónoma.
* As câmaras já ligadas de forma autónoma *não* são desativadas: a ligação à matriz liberta a sessão autónoma e reabre a câmara dentro da matriz.

**Seguinte: Modo de Exibição →**fica disponível assim que forem escolhidos um mestre e, pelo menos, um escravo.**Reexaminar** volta a executar a deteção e a verificação da ligação.

{% hint style="warning" %}
**Cancelar** fica desativado enquanto uma verificação ou sondagem estiver em curso — cancelar a meio de uma sondagem pode causar uma falha na câmara SDK no firmware da câmara LATTICE. Aguarde até que o indicador de carregamento termine.
{% endhint %}

### Passo 2 — Modo

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

de exibição | Modo | O que obtém |
| --- | --- |
| **Câmaras separadas** | Um mosaico em tempo real por câmara, todos acionados em simultâneo para que os fotogramas permaneçam sincronizados. Cada câmara mantém a sua própria cor e definições. |
| **Câmaras combinadas** *(padrão)* | Um único mosaico que apresenta a composição multibanda alinhada NDVI/index. As câmaras partilham a cor da matriz. |

O modo de exibição altera apenas a apresentação da pré-visualização em tempo real — o comportamento de captura é o mesmo em ambos.

### Passo 3 — Definições da matriz e resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

projetado Ao entrar nesta cena, o Chloros solicita ao backend uma **recomendação**e aplica automaticamente uma combinação de ROI + binning que se adapta ao anel de receção do seu NIC (dá preferência ao binning em vez do recorte de ROI, porque o binning preserva o campo de visão completo). Cada alteração que efetuar reexecuta a análise em tempo real e atualiza o painel**Resultado Projetado**, à direita.

Coluna da esquerda — definições:

| Controlo | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **ROI (Campo de Visão)** | Completo (2048×1536) / Metade (1024×768) / Quarto (512×384) | Completo | Recorte do sensor: recorte de metade/quarto para uma região mais pequena com o passo de píxeis nativo. |
| **Binning** | 1× / 2× (soma 2×2) / 4× (soma 4×4) | 1× | Binning de hardware: 2×2 = campo de visão completo a um quarto do custo de transmissão; 4×4 = campo de visão completo a 1/16. Ocultado se as câmaras não suportarem binning. |
| **Imagem do lado da transmissão** (leitura) | — | — | A largura × altura pós-binning efetivamente enviada pela transmissão, redondizada para múltiplos de 16 (mínimo 64). |
| **Resolução dos pinos**| caixa de seleção | desativada | O Chloros normalmente aumenta o agrupamento automaticamente na ligação quando a taxa projetada fica abaixo de**1,5 fps**. O pinning mantém o tamanho de fotograma escolhido e aceita a taxa mais baixa — transformando uma configuração com excesso de subscrição numa recusa de ligação definitiva, em vez de uma redução automática. |
| **Taxa de disparo** | 0,5–60 fps, intervalo de 0,1 | vazio = automático | A taxa de disparo do mestre. Deixe em branco para permitir que o Chloros a calcule. |
| **Capacidade da ligação**| 20–2000 MB/s, intervalo de 10 | vazio = automático | A quantidade que o anfitrião consegue efetivamente absorver, em MB/s —**o único valor do qual depende toda a alocação do conjunto.** Detetado automaticamente a partir do adaptador de rede. Reduza-o se a matriz reportar quadros corrompidos: o valor detetado sobrestima os adaptadores USB e os switches partilhados. Alterá-lo faz com que a projeção seja executada novamente em tempo real. |

Coluna da direita — **Resultado projetado**:

* **Nível de sincronização** — «Captura simultânea» (verde), «Captura simultânea (emissão escalonada FTD)» (verde), «Captura escalonada (desvio de 100 ms)» (âmbar) ou «Configuração demasiado grande» (vermelho).
* **Projeção de fps** — apresentada como um intervalo («escuro → claro»), porque a taxa de um conjunto sincronizado é limitada pela exposição da câmara mais lenta.
* **Linha da NIC** — velocidade da ligação e débito sustentado («NIC {mbps} Mbps · sustentado {N} MB/s»).
* **Verificação de rajada de emissão simulada** — a porta de rede do anel do anfitrião consegue receber uma rajada simultânea de todas as câmaras («Rajada de emissão simulada: X MB · anel de rede utilizável: Y MB ✓/✗»).
* ****Verificação do orçamento de banda** — procura agregada em estado estacionário face ao limite máximo de banda seguro contra colisões («Orçamento de banda: {procura} MB/s exigidos por {n} câmaras · limite máximo {limite} MB/s ✓/✗ sobre-subscrito»).
* **«Número máximo de câmaras neste cabo: {n} — definido pelo limite mínimo de largura de banda por câmara, pelo que o agrupamento não o aumenta.»** — exibido quando se está próximo (ou acima) do limite máximo de número de câmaras.
* **«HAVERÁ PERDA DE FRAMES com estas configurações.»**— aviso a vermelho com o motivo indicado pelo backend, além de uma lista de bloqueadores e**sugestões de correção** a azul («Para adaptar este conjunto à rede» / «Para desbloquear a captura simultânea»).**Aplicar e Ligar** fica bloqueado até existir uma projeção, e o seu rótulo indica o motivo da recusa:

| Rótulo do botão | Significado | O que ajuda realmente |
| --- | --- | --- |
| «A analisar...» | Análise ainda em curso. | Aguardar. |
| **«Demasiadas câmaras para esta rede»**| O conjunto sobrecarrega a ligação (falha na verificação agregada). | Menos câmaras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.**Um ROI mais pequeno NÃO ajudará** — ver abaixo. |
| **«Reduzir ROI para ativar»** | Os quadros seriam perdidos com estas definições (falha na verificação de burst/anel). | Reduzir o ROI, aumentar o binning ou corrigir o anel de receção da placa de rede. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Durante a ligação, poderá aparecer um **painel de transferência de calibração** verde com uma barra de progresso por porta serial: na primeira vez que uma câmara é ligada a um equipamento, o Chloros transfere o seu pacote de calibração de fábrica de ~3,8 MB da câmara através de GigE (cerca de 70 segundos por câmara). As câmaras armazenadas em cache nunca apresentam este painel. Consulte [Ligar câmaras](connecting.md).

## Largura de banda: quantas câmaras cabem

A capacidade de uma matriz depende da ligação, não do Chloros, pelo que os valores de planeamento constam no manual do hardware: **[Planeamento da largura de banda da matriz](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

O que o Chloros faz com esses dados: a caixa de diálogo de ligação executa um teste de rede, projeta a taxa de fotogramas alcançável e seleciona um nível adequado. Se a matriz sobrecarregar a ligação, recusa a ligação em vez de descartar pacotes silenciosamente — consulte o painel de resultados projetados descrito acima.

## Quando faltam quadros

Uma câmara pode estar ausente de um grupo publicado por duas razões completamente diferentes,
e estas requerem soluções opostas. O Chloros contabiliza-as separadamente, em vez de apresentar um
número «incompleto» que não especifica nenhuma das duas:

| O que aconteceu | O que significa | Onde procurar |
| --- | --- | --- |
| **Corrompido**— o quadro chegou e estava estruturalmente danificado | Perda de pacotes GVSP no percurso da rede | O**orçamento da ligação**, o anel de receção da NIC, quadros jumbo, o switch |
| **Nunca chegou**— não chegou qualquer quadro | A câmara não disparou, ou nada saiu dela | O**cabo de sincronização M8**, a linha de sincronização, se todos os membros estão armados |

A divisão é reavaliada a cada 10 segundos enquanto o conjunto de câmaras transmite. Acima de 5 %, é
registada com ambos os números indicados, e cada buffer corrompido é reportado na primeira vez que
ocorre por câmara, sendo depois agregado uma vez por minuto para que uma sessão longa permaneça legível.

**Quadros corrompidos com zero «nunca chegou» significam que o disparo e a sincronização do cabo estão perfeitos**e que todos os quadros perdidos se encontram no percurso da rede. A solução consiste em reduzir o**Wire Budget** e
reconectar.

{% hint style="warning" %}
**Reduzir a taxa de disparo não ajuda com os fotogramas corrompidos.** O ritmo de envio de pacotes
da câmara é definido uma única vez, no momento da ligação. Reduzir a taxa de disparo altera a frequência com que ocorre uma rajada
, não a velocidade a que a própria rajada é transmitida pela rede. Num equipamento medido com 4 câmaras, uma
redução de 5× na taxa de disparo não alterou nada, enquanto que reduzir o orçamento de banda de 240 para
200 MB/s fez com que o mesmo equipamento passasse de 10,4 % de quadros corrompidos para zero.
{% endhint %}

Uma matriz em funcionamento não pode replanejar-se — desligue e volte a ligar para que o seletor de tempo de ligação
possa funcionar de acordo com o novo orçamento.

### Os adaptadores de rede USB estão limitados a 200 MB/s

Um adaptador Ethernet USB anuncia a sua taxa de ligação *Ethernet*, mas o que consegue realmente
manter está limitado pelo barramento USB e pelo seu controlador. Um dongle USB 10GbE costumava ser creditado
com cerca de 1000 MB/s de débito — um valor que nunca tinha sido medido — e a regulação do ritmo
quatro câmaras com base nesse limite fantasmagórico corrompia 6–18 % dos fotogramas, enquanto o conjunto
continuava a reportar uma taxa de fotogramas alvo satisfatória. Os adaptadores ligados por USB estão agora limitados a
**200 MB/s**. Este limite é um valor absoluto e não uma percentagem, porque o restrição é o
barramento: um adaptador USB de 1 GbE atinge cerca de 80 MB/s e não é afetado.

Se o seu host for efetivamente mais rápido do que o limite, aumente o **Wire Budget** para refletir isso.

## Sincronização de tempo PTP

A *sincronização* de fotogramas provém do disparador de hardware; o **PTP** (IEEE 1588 PTPv2) fornece *carimbos de data/hora* comparáveis em todos os dispositivos. Está ativado por predefinição quando a matriz é ligada:

* O **backend do anfitrião Chloros executa o «grandmaster» PTP**. As câmaras LATTICE e os sensores de luz DAQ-E funcionam como escravos neste no domínio 0, pelo que os carimbos de data/hora das imagens e os espectros DAQ ficam sincronizados num único relógio (~1 ms).
* O `--no-ptp` (CLI) desativa-o para trabalhos em bancada — os carimbos de data/hora entre câmaras **não** são, então, comparáveis.
* Verifique o estado da sincronização com o CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

O próprio separador «Câmaras» não tem um indicador PTP; as informações de sincronização por câmara apresentadas nesse separador são a **Função**(Mestre/Escravo), de acesso apenas para leitura, a**Linha de Sincronização** e o nível de Capacidades do conjunto. O estado PTP do DAQ-E é apresentado nos detalhes do sensor no separador «Sensores de Luz».

## A visualização

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

ao vivo da matriz A área principal de transmissão oferece dois layouts (alternáveis na barra superior): **visualização em grelha**(cada mosaico é uma célula; arraste para reordenar quando o cadeado da grelha estiver desbloqueado) e**visualização em lista**(matrizes em largura total na parte superior, uma câmara ativa abaixo). O controlo deslizante**Zoom do Feed** ajusta o tamanho dos mosaicos; com largura de célula inferior a 200 px, as sobreposições de nome/fps ocultam-se automaticamente.

O **modo separado** mostra um mosaico por câmara. Cada mosaico apresenta:

* o nome da câmara (canto superior esquerdo),
* uma **leitura de fps** (canto superior direito) — esta é a *taxa de captura real* da câmara comunicada pelo backend, não a taxa de atualização da pré-visualização (a pré-visualização em direto está limitada a 30 fps, independentemente da taxa de captura),
* um ponto de estado — verde (transmissão) / âmbar (a carregar) / vermelho (erro),
* um **indicador de quadro desatualizado** quando não chega nenhum quadro novo durante 2 s — normal durante cerca de 5 s após qualquer ligação/desligamento, enquanto o backend reequilibra a largura de banda entre as câmaras.

O **modo combinado**apresenta um único mosaico composto: o backend realiza o debayering, redimensiona, alinha, remove ruído, converte para radianância por banda (mais refletância DLS quando um sensor de luz está associado), avalia a expressão do índice da matriz, aplica a LUT e transmite o resultado como MJPEG. Até que o primeiro quadro alinhado seja renderizado, o mosaico indica o seu estado: «A preparar matriz…», «A calibrar o alinhamento…», «À espera do primeiro quadro…» ou — se o tempo de tentativa de alinhamento automático (~30 s) se esgotar — «Alinhamento necessário», com um botão**Calibrar alinhamento**.

Informações úteis sobre o modo combinado:

* A composição é registada em relação ao fotograma da câmara **mestre**. A focagem AE-ROI e a medição pontual na composição são exatas para a câmara mestre e aproximadas para as escravas; utilize a**Visualização Dividida** (configurações da matriz → «Mostrar câmaras membros») para mosaicos por câmara com precisão ao nível do pixel sem abrir ligações de câmara adicionais.
* **Camadas de exibição**(configurações da matriz; desativadas por predefinição) permite escolher uma camada de primeiro plano e uma de fundo — qualquer câmara membro ou**Índice**. Com primeiro plano = Índice, os píxeis fora do Min/Max da LUT mostram a camada de fundo.
* **Resolução de renderização** (padrão 720p) define a altura da transmissão em direto *e* o tamanho de exportação da composição guardada. As imagens por câmara são sempre exportadas em resolução total.
* O alinhamento é calculado por sessão e nunca é guardado — consulte a secção de alinhamento do painel de definições da matriz para ver os resíduos RMS e o botão «Recalibrar».

## Captura: monitorização vs. análise

As superfícies de captura da matriz dividem-se claramente em **nível de monitorização**(gravar o que se vê) e**nível de análise** (gravar dados brutos, calibrar posteriormente):

| Fluxo de trabalho | Nível | O que guarda | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Captura**(imagens estáticas) | Análise | Um grupo de fotogramas sincronizados por passagem; ficheiros por câmara em cada nível de exportação selecionado (bruto/sem debayering/radiação/refletância/pré-visualização/índice) + ficheiro sidecar `.daq` | Botão**Capturar Tudo** + Definições de Captura | `lattice array-capture` |
| **Gravar vídeo de índice** | Monitorização | A composição do índice combinado em direto tal como apresentada — 8 bits, resolução de pré-visualização, LUT incorporada; requer que a transmissão em direto esteja aberta | ● Gravar vídeo de índice (matrizes combinadas) | `lattice array-record` |
| **Série de imagens brutas → criar vídeo**| Análise | Fotogramas brutos do sensor à taxa de captura total + manifesto + `.daq`, seguidos de reconstrução offline em vídeo calibrado de radiação / refletância / índice, sincronizado temporalmente com as leituras do DAQ | ⦿ Gravar série de imagens brutas →**Criar vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regra geral: se os píxeis forem alimentar *medições*, utilize a captura ou o burst (nível de análise); se apenas precisar de *ver ou demonstrar* o que o conjunto de sensores captou, grave o vídeo indexado (nível de monitorização).

### Definições de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

A engrenagem ao lado de **Capturar tudo** abre o painel de Definições de captura (requer um projeto aberto — as capturas são guardadas nesse projeto):

* **Modo de captura**:**Único**(uma passagem) /**Contínuo**(consecutivo; limitado por um número de capturas, valor predefinido 1, ou por uma duração, valor predefinido 10 s) /**Intervalo** (timelapse: N capturas a cada intervalo de X para um total de Y, valores predefinidos 1 a cada 5 s durante 1 minuto).
* **Tipos de exportação por câmara**: Raw, Debayered, Radiance, Reflectance, Preview, Index — todas as opções aplicáveis estão ativadas por predefinição. Radiance/Reflectance estão ocultas para câmaras RGB-filter;**Reflectance aparece apenas quando a câmara tem um sensor de luz DAQ** (próprio ou herdado da matriz); Index requer uma expressão de índice configurada.
* **Alinhado**(por matriz, predefinição**ativada**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, para que as exportações sejam registadas ao nível do pixel. O formato Raw permanece sempre sem distorção, mas transporta a transformação nos metadados.
* **Captura mais rápida** (alternar): apenas raw + a leitura DAQ atribuída + o composto de índice combinado gratuito, ignorando os cálculos de calibração no momento da captura para obter a taxa máxima — reconstrua a radianância/refletância/índice posteriormente a partir do `.daq` guardado.
* As seleções permanecem com o projeto. As câmaras ocultas ou em pausa são ignoradas.

O equivalente a CLI (mesmo ponto final do backend, mesma semântica):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

A compressão para capturas do TIFF é o `deflate` (sem perdas, predefinição) ou o `none` — tabelas de sinalizadores completas, o layout da pasta de captura e as regras de reprocessamento encontram-se na [Referência do CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparelhamento de um sensor de luz DAQ

As pré-visualizações com correção de refletância e iluminação necessitam de dados de luz descendente provenientes de um sensor DAQ (ligado no separador **Sensores de Luz**):

* A **linha da matriz**na barra lateral apresenta um**botão &quot;DAQ · ligado/desligado&quot;** — *ligado* quando está definido um sensor de luz ao nível da matriz **ou** quando qualquer câmara membro possui o seu próprio; a sua dica de ferramenta indica exatamente qual o sensor que alimenta cada câmara.
* Atribua a nível da matriz nas definições da matriz → **Sensor de Luz Ambiente**→ menu suspenso**Sensor de Luz**. A seleção mantém-se com o projeto, propaga-se a todas as câmaras do conjunto e as câmaras individuais ainda podem substituí-la pelo seu próprio sensor.
* A linha de estado abaixo indica o estado em tempo real: **Desligado**→ «À espera do primeiro espectro…» →**«Ativo — todas as câmaras da matriz têm a iluminação corrigida»** → ou, se não tiver chegado nenhum espectro novo nos últimos 3 s, um aviso de dados desatualizados — a última leitura continua a ser utilizada (as leituras nunca expiram no percurso de captura).

Com um sensor atribuído: o tipo de exportação «Refletância» fica disponível, as pré-visualizações em tempo real são corrigidas quanto à iluminação, a exposição automática preditiva pode utilizar o espectro e cada captura de refletância grava a leitura do DAQ efetivamente utilizada como um **`.daq` sidecar** ao lado da imagem, para que a captura possa ser reprocessada posteriormente.

## `array-connect` CLI opções

| Indicador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | deteção automática de todas as câmaras LATTICE (requer ≥2) | **A primeira porta série é a MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | auto | Taxa de disparo do disparador principal. |
| `--binning {1,2,4}` | automático | Agrupamento por hardware. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituição avançada do seletor de nível de sincronização. |
| `--wire-ceiling-mbps MB_PER_S` | detecção automática | Orçamento de largura de banda do anfitrião em MB/s — a forma CLI do campo **Orçamento de largura de banda**. Reduza-o se a matriz reportar quadros corrompidos. Guardado com o projeto, pelo que uma reconexão posterior o restaura. |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativa o PTP (os carimbos de data/hora entre câmaras deixam de ser comparáveis). |

`lattice array-list`, `array-status` e `array-disconnect` gerem a sessão persistente. A referência completa aos subcomandos, incluindo o alinhamento (`align-calibrate` / `align-apply`) e as ferramentas de rede, encontra-se na [Referência do CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); os equivalentes a SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) encontram-se na [Referência SDK](../reference/sdk-reference.md). A partir de Python, o orçamento de cablagem é `connect_array(..., wire_ceiling_mbps=120)`, e a divisão entre «corrupto» e «nunca chegou» encontra-se em [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
