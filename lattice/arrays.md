# Matriz de Câmaras Múltiplas

Uma **matriz**LATTICE consiste em duas ou mais câmaras LATTICE ligadas como uma única unidade sincronizada. Uma das câmaras é a**mestre**: esta emite um impulso de disparo GPIO de hardware numa linha de sincronização partilhada (por predefinição, a**Linha 2**), para que todos os membros capturem o mesmo instante. O Chloros adiciona sincronização temporal PTP, uma pré-visualização em tempo real (mosaicos por câmara ou uma única composição multibanda alinhada) e captura sincronizada — cada passagem de captura produz um **grupo de fotogramas** no qual todas as câmaras partilham o mesmo carimbo temporal e ID de fotograma (indicado como `fid:N` na saída de captura).

As matrizes são a forma como as câmaras monocromáticas (M3M) produzem índices de vegetação — uma câmara contribui com uma banda e a matriz alinha-as numa pilha multibanda. Consulte [Câmaras Mono e Índices de Vegetação](mono-indices.md).

Existem três formas equivalentes de ligar uma matriz, e todas elas executam o mesmo fluxo «smart-prep»:

| Superfície | Ponto de entrada |
| --- | --- |
| GUI | Separador «Câmaras» → **Ligar Matriz** (botão azul) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primeiro número de série = mestre) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primeiro número de série = mestre) |

O Smart-prep executa: um teste de capacidade de rede (ping ICMP DF + teste GVSP), seleção do nível de sincronização, uma projeção da taxa de fotogramas com o tamanho de fotograma solicitado, a decisão de transporte compactado de 12 bits (ver [Transporte compactado de 12 bits](#packed-12-bit-transport)), ativação do PTP, seleção automática do formato de píxeis por câmara, inicialização da exposição automática a partir do estado guardado de cada câmara e configuração do disparador GPIO na Linha 2.

**Sensor completo por predefinição (novidade na versão 1.3.0).** O Connect mantém o tamanho de fotograma que solicitou — o sensor completo de 2048×1536, sem binning nem recorte, a menos que opte pelo contrário — e permite que a taxa de fotogramas desça para o valor que o anfitrião e a rede disponibilizam. Nunca realiza binning nem recorte por iniciativa própria (a versão 1.2.0 ativava automaticamente o binning abaixo de 1,5 fps). A decisão sobre o binning ou uma ROI mais pequena é sua: na caixa de diálogo «Connect», escolha uma opção e o «Resultado Projetado» mostra a taxa de fotogramas resultante; o comando `array-connect` apresenta as opções mais rápidas com as respetivas taxas projetadas.

{% hint style="info" %}
As câmaras têm de estar acessíveis na ligação para que tudo isto funcione — consulte [Ligar Câmaras](connecting.md) para deteção, endereçamento e o download de calibração na primeira ligação. Para configurações com várias câmaras, as definições do anel de receção da placa de rede do anfitrião são tão importantes quanto a velocidade da ligação; a tabela completa de sintomas→soluções encontra-se na [Referência CLI § Configuração e Ajuste da Placa de Rede do Anfitrião](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## A caixa de diálogo «Ligar Matriz»

O separador «Câmaras» → **Ligar Matriz**abre um assistente de três passos:**Selecionar → Modo de Visualização → Definições**.

### Passo 1 — Selecionar a câmara principal e as secundárias

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



A caixa de diálogo analisa a rede assim que é aberta («A analisar a rede...») e, em seguida, verifica os cabos de sincronização («A verificar os cabos de sincronização...») para identificar quais as câmaras que se encontram na linha de sincronização M8. São necessárias, pelo menos, **2 câmaras** para criar um conjunto.

Os cabos de sincronização M8 convergem num único ponto de junção, pelo que **qualquer câmara ligada por cabo pode ser a mestre**. A verificação preenche previamente as funções — a câmara RGB (`-FRGB`) como mestre quando estiver ligada, caso contrário a primeira câmara ligada, e as outras câmaras ligadas como escravas — e apresenta uma de três mensagens:

| Mensagem | Significado |
| --- | --- |
| «Cabo de sincronização detetado em {n} câmaras. Qualquer uma delas pode ser mestre; a câmara RGB (ou, caso contrário, a primeira) está pré-selecionada.» (verde) | O sondador detetou as câmaras na linha de sincronização; as caixas de seleção «mestre» e «escravo» já estão preenchidas. |
| «Nenhum mestre detetado — verifique o cabo GPIO» (laranja) | Nenhuma câmara detetou um impulso de disparo; verifique o cabeamento de sincronização. As funções continuam predefinidas (a câmara RGB, ou a primeira câmara da lista, como mestre) e pode alterá-las. |
| «Sem cabo de sincronização: {seriais}» (laranja) | As câmaras listadas não têm nenhum cabo de sincronização ligado. |

A tabela de câmaras tem as colunas **Câmara / Serial / IP / Mestre (rádio) / Escravo (caixa de seleção)**. As linhas estão listadas por**ordem das câmaras** — câmaras M3C (Bayer) antes das M3M (mono); entre as M3C, FRGB, FRGN, FOCN e, em seguida, FNGB; as M3M por comprimento de onda do filtro, começando pelo mais baixo (F450 antes de F808). A lista de câmaras na barra lateral e o layout predefinido da grelha utilizam a mesma ordem.

* Escolha exatamente **um mestre**e**um ou mais escravos**. Clicar novamente no botão de opção do mestre atual desmarca-o. Escolher um mestre diferente faz-se com um clique: o novo mestre deixa os escravos e o mestre anterior torna-se um escravo (a menos que não tenha cabo de sincronização).
* Uma câmara marcada com **«Sem cabo de sincronização»** nunca pode ser selecionada como escrava — uma escrava sem ligação de disparo ficaria à espera na linha de sincronização para sempre e transmitiria um sinal inativo. Em vez disso, ligue essa câmara como uma câmara autónoma.
* As câmaras já ligadas de forma autónoma *não* são desativadas: a ligação em matriz liberta a sessão autónoma e reabre a câmara dentro da matriz.

**Seguinte: Modo de Exibição →**fica disponível assim que forem escolhidos um mestre e, pelo menos, um escravo.**Reexaminar** executa novamente a deteção e a sondagem de ligações.

{% hint style="warning" %}
**Cancelar** fica desativado enquanto uma verificação ou sondagem estiver em curso — cancelar a meio da sondagem pode causar uma falha no SDK da câmara no firmware da câmara LATTICE. Aguarde até que o indicador de carregamento termine.
{% endhint %}

### Passo 2 — Modo de exibição



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modo | O que obtém |
| --- | --- |
| **Câmaras separadas** | Um mosaico em tempo real por câmara, todos acionados em simultâneo para que os fotogramas permaneçam sincronizados. Cada câmara mantém a sua própria cor e definições. |
| **Câmaras combinadas** *(predefinição)* | Um único mosaico que apresenta a composição multibanda alinhada de NDVI/índice. As câmaras partilham a cor da matriz. |

O modo de visualização altera apenas a apresentação da pré-visualização em tempo real — o comportamento de captura é o mesmo em ambos.

### Passo 3 — Definições da matriz e resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

previsto

Ao entrar nesta cena, o Chloros solicita ao backend uma **recomendação**e aplica-a: o**sensor completo — sem binning, sem recorte de ROI — sempre que qualquer taxa de fotogramas o permitir**, independentemente da taxa que o anfitrião e a ligação forneçam. Apenas quando nenhuma taxa consegue suportar o sensor completo é que a recomendação reduz o fotograma, e, nesse caso, dá preferência ao binning em detrimento do recorte de ROI, uma vez que o binning preserva o campo de visão completo. A partir daí, troca-se resolução por taxa de fotogramas. Cada alteração que efetuar reexecuta a análise em tempo real e atualiza o painel**Resultado Projetado**, à direita.

Coluna da esquerda — definições:

| Controlo | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **ROI (Campo de Visão)** | Completo (2048×1536) / Metade (1024×768) / Um quarto (512×384) | Completo | Recorte do sensor: recorte de metade/um quarto para uma região mais pequena com o passo de píxeis nativo. |
| **Binning** | 1× / 2× (soma 2×2) / 4× (soma 4×4) | 1× | Binning por hardware: 2×2 = campo de visão completo a um quarto do custo de transmissão; 4×4 = campo de visão completo a 1/16. Ocultado se as câmaras não suportarem binning. |
| **Imagem do lado do cabo** (leitura) | — | — | A largura × altura pós-binning efetivamente enviada pelo cabo, redondizada para múltiplos de 16 (mínimo 64). |
| **Taxa de disparo** | 0,5–60 fps, passo de 0,1 | vazio = automático | A taxa de disparo do mestre. Deixe em branco para que o Chloros a calcule. |
| **Capacidade da ligação**| 20–2000 MB/s, intervalo de 10 | vazio = automático | A quantidade que o anfitrião consegue efetivamente absorver, em MB/s —**o único valor do qual depende toda a alocação da matriz.** Detetado automaticamente a partir do adaptador de rede. Reduza-o se a matriz reportar quadros corrompidos: o valor detetado sobrestima os adaptadores USB e os switches partilhados. Alterá-lo faz com que a projeção seja executada novamente em tempo real. O campo abre com o orçamento com que a última matriz foi ligada (guardado com o projeto), pelo que um orçamento que tenha corrigido uma vez é mantido. |

A caixa de seleção **Resolução dos pinos** da versão 1.2.0 foi removida: a função «Ligar» mantém sempre o tamanho de quadro que escolher e, em vez disso, reduz a taxa de quadros.

Coluna da direita — **Resultado projetado**:

* **Nível de sincronização** — «Captura simultânea» (verde), «Captura simultânea (emissão escalonada FTD)» (verde), «Captura escalonada (desvio de 100 ms)» (âmbar) ou «Configuração demasiado grande» (vermelho). O Chloros escolhe o nível: captura simultânea sempre que a ligação puder ser partilhada a uma velocidade que as câmaras consigam acompanhar (a taxa de fotogramas diminui à medida que a quota de cada câmara diminui) e captura escalonada apenas quando tal não for possível — assim, um conjunto de câmaras liga-se a qualquer rede, à taxa de fotogramas que a rede permitir.
* **Projeção de fps** — a taxa de fotogramas à qual se espera que o conjunto funcione. Com a captura simultânea, trata-se de um único valor: a taxa vem em primeiro lugar e a exposição automática preenche o período que esta determina, aumentando depois o ganho, para que uma cena pouco iluminada não a reduza (ver [Como a exposição automática distribui a exposição e o ganho](camera-settings.md)). Com a captura escalonada, é apresentado como um intervalo («escuro → claro»), porque a exposição de cada câmara prolonga o escalonamento numa cena escura.
* **Linha da NIC** — velocidade de ligação e débito sustentado («NIC {mbps} Mbps · sustentado {N} MB/s»).
* **Verificação de rajada de emissão simulada** — a NIC do anel de receção do anfitrião consegue receber uma rajada simultânea de todas as câmaras («Rajada de emissão simulada: X MB · Anel NIC utilizável: Y MB ✓/✗»).
* **Verificação do orçamento da ligação** — procura agregada em estado estacionário vs. o limite máximo da ligação à prova de colisões («Orçamento da ligação: {procura} MB/s exigidos por {n} câmaras · limite máximo {limite} MB/s ✓/✗ sobre-subscrição»).
* **«Número máximo de câmaras nesta ligação: {n} — definido pelo limite mínimo de largura de banda por câmara, pelo que o agrupamento não o aumenta.»** — exibido quando se está próximo (ou acima) do limite máximo de número de câmaras.
* **«OS FRAMES SERÃO PERDIDOS com estas definições."**— aviso a vermelho com o motivo indicado pelo backend, além de uma lista de bloqueadores e**sugestões de correção** a azul ("Para encaixar esta matriz na rede" / "Para desbloquear a captura simultânea").**Aplicar e Ligar** fica bloqueado até existir uma projeção, e a sua etiqueta indica o motivo da recusa:

| Rótulo do botão | Significado | O que realmente ajuda |
| --- | --- | --- |
| «A analisar...» | Análise ainda em curso. | Aguarde. |
| **«Demasiadas câmaras para esta rede"**| A ligação não consegue suportar o conjunto nem mesmo uma câmara de cada vez (falha na verificação agregada): há muito mais câmaras do que a ligação consegue processar, ou outras câmaras a transmitir na mesma ligação. | Menos câmaras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.**Uma área de interesse (ROI) mais pequena NÃO vai ajudar** — ver abaixo. |
| **«Reduzir a área de interesse (ROI) para ativar»** | Os quadros seriam perdidos com estas definições (falha na verificação de burst/ring). | Reduzir a área de interesse (ROI), aumentar o binning ou corrigir o anel de receção da placa de rede. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Durante a ligação, poderá aparecer um **painel de transferência de calibração** verde com uma barra de progresso por porta série: na primeira vez que uma câmara é ligada a uma máquina, o Chloros obtém o seu pacote de calibração de fábrica de ~3,8 MB a partir da câmara através de GigE (cerca de 70 segundos por câmara). As câmaras armazenadas em cache nunca mostram este painel; com mais de seis câmaras, as suas linhas deslocam-se. Consulte [Ligar câmaras](connecting.md).

### Transporte compactado de 12 bits

Os formatos de 12 bits (`BayerRG12`, `Mono12`) são normalmente transmitidos como palavras de 16 bits. **O transporte**compactado**envia os mesmos 12 bits em 1,5 bytes por pixel e descompacta-os no anfitrião, pelo que cada fotograma demora 25 % menos tempo a ser enviado — com o sensor a funcionar a plena capacidade, aproximadamente mais um terço de fotogramas por segundo.**Os dados guardados são idênticos.** A «Array Connect» decide isso automaticamente (`auto`): comprime apenas quando isso é vantajoso e o anfitrião dispõe de CPU disponível para a descompressão. A GUI não tem nenhuma opção para isso; a partir da CLI, utilize `array-connect --packed {auto,on,off}` (padrão: `auto`). Os formatos de 8 e 16 bits nunca são compactados.

## Largura de banda: quantas câmaras cabem

A capacidade de um array depende da ligação, não do Chloros, pelo que os valores de planeamento constam no manual do hardware: **[Planeamento da Largura de Banda do Array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

O que o Chloros faz com esses dados: a caixa de diálogo de ligação executa um teste de rede, projeta a taxa de fotogramas alcançável e seleciona um nível adequado. Uma matriz liga-se a qualquer rede: numa ligação lenta, cada câmara é ajustada à sua quota da largura de banda e a taxa de fotogramas diminui, em vez de a matriz ser recusada ou os seus fotogramas serem reduzidos. Apenas um conjunto que a ligação não consiga suportar nem sequer uma câmara de cada vez é recusado (**Demasiadas câmaras para esta rede**), em vez de descartar pacotes silenciosamente — consulte o painel de resultados projetados descrito acima.

## Quando faltam fotogramas

Uma câmara pode estar ausente de um grupo publicado por duas razões completamente diferentes,
e estas requerem soluções opostas. O Chloros contabiliza-as separadamente, em vez de apresentar um
número «incompleto» que não especifica nenhuma delas:

| O que aconteceu | O que significa | Onde procurar |
| --- | --- | --- |
| **Corrompido**— o quadro chegou e estava estruturalmente danificado | Perda de pacotes GVSP no percurso da rede | O**orçamento de largura de banda**, o anel de receção da placa de rede, quadros jumbo, o comutador |
| **Nunca chegou**— não chegou nenhum quadro | A câmara não disparou, ou nada saiu dela | O**cabo de sincronização M8**, a linha de sincronização, se todos os membros estão armados |

A divisão é reavaliada a cada 10 segundos enquanto o conjunto de câmaras transmite. Acima de 5 %, é
registada com ambos os números indicados, e cada buffer corrompido é reportado na primeira vez que
ocorre por câmara, sendo depois agregado uma vez por minuto para que uma sessão longa permaneça legível.

**Fotogramas corrompidos com zero «nunca chegou» significam que o disparo e a sincronização do cabo estão perfeitos**e que todos os fotogramas perdidos se encontram no percurso da rede. A solução consiste em reduzir o**Wire Budget** e
reconectar.

{% hint style="warning" %}
**Reduzir a taxa de disparo não ajuda com os fotogramas corrompidos.** O ritmo de pacotes
da câmara é definido uma única vez, no momento da ligação. Reduzir a taxa de disparo altera a frequência com que ocorre uma rajada,
não a velocidade a que a própria rajada é transmitida pela rede. Num equipamento medido com 4 câmaras, uma
redução de 5× na taxa de disparo não alterou nada, enquanto que reduzir o orçamento de banda de 240 para
200 MB/s reduziu a taxa de quadros corrompidos dessa mesma configuração de 10,4 % para zero.
{% endhint %}

Uma matriz em funcionamento não pode replanejar-se — desligue e volte a ligar para que o seletor de tempo de ligação
possa funcionar com base no novo orçamento.

### Os adaptadores de rede USB têm um limite

Um adaptador Ethernet USB anuncia a sua taxa de ligação *Ethernet*, mas o que consegue realmente
manter é limitado pelo barramento USB e pelo seu controlador. Um dongle USB 10GbE costumava ser creditado
com cerca de 1000 MB/s de débito — um valor que nunca tinha sido medido — e ajustar
quatro câmaras a essa margem fantasmagórica corrompeu 6–18 % dos fotogramas, enquanto o conjunto
continuava a reportar uma taxa de fotogramas alvo satisfatória. Os adaptadores ligados por USB estão agora limitados a
**200 MB/s** no Windows e no NVIDIA Jetson, e **400 MB/s** no Linux x86_64. O limite é um valor absoluto e não uma percentagem, porque o restritor é o
barro: um adaptador USB de 1 GbE atinge cerca de 80 MB/s e não é afetado.

Se o seu anfitrião for efetivamente mais rápido do que o limite, aumente o **Wire Budget** para refletir isso.

## Sincronização temporal PTP

A *sincronização* de fotogramas provém do disparador de hardware; o **PTP** (IEEE 1588 PTPv2) fornece *carimbos de data/hora* comparáveis em todos os dispositivos. Está ativado por predefinição na ligação do conjunto:

* O **backend do anfitrião Chloros executa o grandmaster PTP**. As câmaras LATTICE e os sensores de luz DAQ-E funcionam como escravos no domínio 0, pelo que os carimbos de data/hora das imagens e os espectros DAQ ficam sincronizados num único relógio (~1 ms).
* `--no-ptp` (CLI) desativa-o para trabalhos em bancada — os carimbos de data/hora entre câmaras **não** são, então, comparáveis.
* Verifique o estado da sincronização através da CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

O próprio separador «Câmaras» não possui um indicador PTP; as informações de sincronização por câmara apresentadas nesse separador são a **Função**(Mestre/Escravo), de acesso apenas para leitura, a**Linha de Sincronização** e o nível de Capacidades do conjunto. O estado PTP do DAQ-E é apresentado nos detalhes do sensor no separador «Sensores de Luz».

## A visualização

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

do conjunto em tempo real A área principal de visualização oferece dois layouts (alternáveis na barra superior): **visualização em grelha**(cada mosaico é uma célula; arrastar para reordenar quando o cadeado da grelha estiver desbloqueado) e**visualização em lista**(linhas de largura total: uma matriz**combinada**corresponde a uma linha — o seu mosaico composto — enquanto uma matriz**separada**atribui a**cada membro a sua própria linha**, com a sua própria transmissão em direto e a sua própria tabela de Alvos de Calibração; abaixo das matrizes, uma câmara independente — a ativa). O controlo deslizante**Zoom do Feed** ajusta o tamanho dos blocos; com uma largura de célula inferior a 200 px, as sobreposições de nome/fps ocultam-se automaticamente. As câmaras que compõem a matriz aparecem na barra lateral e no layout predefinido da grelha pela ordem das câmaras (M3C antes de M3M, FRGB em primeiro lugar, monocromáticas por comprimento de onda); desbloquear a grelha e arrastar continua a substituir a ordem da grelha.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



O **modo separado** mostra um mosaico por câmara. Cada mosaico apresenta:

* o nome da câmara (canto superior esquerdo),
* uma **leitura de fps** (canto superior direito) — esta é a *verdadeira taxa de captura* da câmara comunicada pelo backend, não a taxa de atualização da pré-visualização (a pré-visualização ao vivo está limitada a 30 fps, independentemente da taxa de captura),
* um ponto de estado — verde (transmissão) / âmbar (a carregar) / vermelho (erro),
* um **indicador de quadro desatualizado** quando não chega nenhum quadro novo durante 2 s — normal durante cerca de 5 s após qualquer ligação/desligamento, enquanto o backend reequilibra a largura de banda entre as câmaras.

O **modo combinado**mostra um único mosaico composto: o backend realiza o debayering, redimensiona, alinha, remove ruído, converte para radiação por banda (mais refletância DLS quando um sensor de luz está associado), avalia a expressão do índice da matriz, aplica a LUT e transmite o resultado como MJPEG. Até que o primeiro quadro alinhado seja renderizado, o mosaico indica o seu estado: «A preparar matriz…», «A calibrar o alinhamento…», «À espera do primeiro fotograma…» ou — se as tentativas de alinhamento automático do mosaico (cerca de um minuto) se esgotarem — «Alinhamento necessário» com um botão**Calibrar alinhamento**.

A indicação de fps do mosaico da matriz também informa o estado da transmissão:**«A aplicar definições…»**enquanto uma alteração nas definições (formato, binning, ROI) reinicia as câmaras,**«Parado»**quando a matriz está ativa mas deixou de enviar grupos de fotogramas, e**«Parado"**quando deixa de transmitir — nesse caso, volte a ligar o conjunto. Quando a separação espectral teve de limitar uma banda apresentada na imagem composta numa grande parte dos seus píxeis (25 % ou mais), o mosaico apresenta**"Dados da banda de baixa confiança · N%"**: os valores de índice calculados com base nessa banda são suspeitos (normalmente, uma banda recebeu muito pouca luz).

Informações úteis sobre o modo combinado:

* A imagem composta é alinhada com o fotograma da câmara **principal**. A focagem AE-ROI e a medição pontual na imagem composta são exatas para a câmara principal e aproximadas para as secundárias; utilize a**Visão Dividida** (configurações do conjunto → «Mostrar câmaras membros») para obter mosaicos por câmara com precisão ao nível do pixel sem abrir ligações adicionais às câmaras.
* ****Camadas de exibição**(configurações da matriz; desativadas por predefinição) permitem-lhe escolher uma camada de primeiro plano e uma de fundo — qualquer câmara membro ou**Índice**. Com o primeiro plano definido como Índice, os píxeis fora dos valores mínimo/máximo da LUT mostram a camada de fundo.
* **Resolução de renderização** (padrão 720p) define a altura da transmissão em direto *e* o tamanho de exportação da imagem composta guardada. As imagens por câmara são sempre exportadas na resolução total.
* O alinhamento é calculado por sessão e nunca é guardado. O Chloros alinha **todas as matrizes automaticamente após a ligação, em ambos os modos de visualização**, assim que a exposição automática dos membros se estabilizar (continua a tentar durante um máximo de 5 minutos). A secção**Alinhamento**do painel de definições da matriz mostra os resíduos RMS e o botão**Recalibrar alinhamento** — também para matrizes separadas, que não têm mosaico composto.

## Captura: monitorização vs. análise

As superfícies de captura do conjunto dividem-se claramente em **nível de monitorização**(gravar o que se vê) e**nível de análise** (gravar dados brutos, calibrar posteriormente):

| Fluxo de trabalho | Nível | O que guarda | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Captura**(imagens estáticas) | Análise | Um grupo de fotogramas sincronizados por passagem; ficheiros por câmara em todos os níveis de exportação selecionados (bruto/sem debayering/radiação/refletância/pré-visualização/índice) + ficheiro sidecar `.daq` |**Botão **Capturar tudo** + Definições de captura | `lattice array-capture` |
| **Gravar vídeo de índice** | Monitorização | A composição do índice combinado em tempo real tal como apresentada — 8 bits, resolução de pré-visualização, LUT incorporada. A partir da CLI, sem nenhuma vista da GUI aberta, o backend renderiza o NDVI (ou `--expr`) da matriz assim que esta estiver alinhada; uma gravação que não escreva nenhum fotograma falha | ● Gravar vídeo de índice (matrizes combinadas) | `lattice array-record` |
| **Série de imagens brutas → criar vídeo**| Análise | Fotogramas brutos do sensor à taxa máxima de captura + manifesto + `.daq`, seguida de reconstrução offline em vídeo calibrado de radiação / refletância / índice, sincronizado temporalmente com as leituras do DAQ | ⦿ Gravar série de imagens brutas →**Criar vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regra geral: se os píxeis forem fornecer *medições*, utilize a captura ou a sequência (nível de análise); se apenas precisar de *ver ou demonstrar* o que o painel captou, grave o vídeo indexado (nível de monitorização).

### Definições de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



A engrenagem ao lado de **Capturar tudo** abre o painel de Definições de captura (requer um projeto aberto — as capturas são guardadas nesse projeto):

* **Modo de captura**:**Único**(uma passagem) /**Contínuo**(consecutivo; limitado por um número de capturas, predefinição 1, ou por uma duração, predefinição 10 s) /**Intervalo** (timelapse: N capturas a cada intervalo de X para um total de Y; por predefinição, 1 a cada 5 s durante 1 minuto).
* **Tipos de exportação por câmara**: Raw, Debayered, Radiance, Reflectance, Preview, Index — todas as opções aplicáveis estão ativadas por predefinição. Radiance/Reflectance ficam ocultas para câmaras com filtro RGB; **A Refletância só aparece quando a câmara possui um sensor de luz DAQ** (próprio ou herdado da matriz); o Índice requer uma expressão de índice configurada.
* **Alinhado**(por matriz, predefinição**ativada**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, para que as exportações fiquem registadas em termos de píxeis. O formato Raw permanece sempre sem distorção, mas inclui a transformação nos metadados. Os ficheiros dos membros da matriz são guardados na orientação nativa de cada câmara (qualquer configuração de Espelhamento/Rotação é registada no ficheiro, não aplicada).
* **Captura mais rápida** (alternar): apenas raw + a leitura DAQ atribuída + a composição de índice combinado livre, ignorando os cálculos de calibração no momento da captura para obter a taxa máxima — reconstrua a radiação/refletância/índice posteriormente a partir do ficheiro `.daq` guardado.
* As seleções permanecem com o projeto. As câmaras ocultas ou em pausa são ignoradas.
* **Contínuo** em matrizes é executado no backend do Chloros (novidade na versão 1.3.0): quando todas as câmaras da lista pertencem a uma matriz, o backend executa ele próprio o ciclo de captura, gravando um grupo enquanto seleciona o seguinte, sem necessidade de um pedido por captura. Uma unidade cheia interrompe a execução («Interrompido: unidade cheia. Guardado N, F falhou»). Consulte [Definições e modos de captura](capture.md#capture-modes) para ver a nota do Microsoft Defender sobre taxas de captura elevadas.

A CLI equivalente (mesmo ponto final do backend, mesma semântica):

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

A compressão TIFF para capturas é `deflate` (sem perdas, predefinição) ou `none` — as tabelas completas de sinalizadores, a estrutura da pasta de captura e as regras de reprocessamento encontram-se na [Referência da CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparelhamento de um sensor de luz DAQ

As pré-visualizações com correção de refletância e iluminação necessitam de dados de luz descendente provenientes de um sensor DAQ (ligado no separador **Sensores de Luz**):

* A **linha da matriz**na barra lateral apresenta um**botão «DAQ · ligado/desligado»** — *ligado* quando está definido um sensor de luz ao nível da matriz **ou** quando qualquer câmara membro possui o seu próprio; a sua dica de ferramenta indica exatamente qual o sensor que alimenta cada câmara.
* Atribua a nível de matriz nas definições da matriz → **Sensor de Luz Ambiente**→ menu suspenso**Sensor de Luz**. A seleção mantém-se com o projeto, propaga-se a todas as câmaras do conjunto e as câmaras individuais ainda podem substituí-la pelo seu próprio sensor.
* A linha de estado abaixo indica o estado em tempo real: **Desligado**→ «À espera do primeiro espectro…» →**«Ativo — todas as câmaras do conjunto têm a iluminação corrigida»** → ou, se não tiver chegado nenhum espectro novo nos últimos 3 s, um aviso de dados desatualizados — a última leitura continua a ser utilizada (as leituras nunca expiram no percurso de captura).

Com um sensor atribuído: o tipo de exportação «Refletância» fica disponível, as pré-visualizações em tempo real são corrigidas em termos de iluminação, a exposição automática preditiva pode utilizar o espectro e cada captura de refletância grava a leitura do DAQ efetivamente utilizada como um **ficheiro `.daq`** na pasta `daq/` para que a captura possa ser reprocessada posteriormente. Além disso, um ficheiro `.daq.cameras.json` identifica as câmaras cujos fotogramas essa leitura descreve; ao reprocessar, o Chloros a a leitura apenas com imagens dessas câmaras.

## Opções da CLI do `array-connect`

| Opção | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detetar automaticamente todas as câmaras LATTICE (requer ≥2) | **O primeiro número de série é o MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | auto | Taxa de disparo do mestre. |
| `--binning {1,2,4}` | auto | Agrupamento de pixels por hardware. «Auto» corresponde ao sensor completo (sem agrupamento) sempre que qualquer taxa de fotogramas o suportar; «connect never bins» serve apenas para manter a taxa elevada. |
| `--packed {auto,on,off}` | `auto` | [Transporte compactado de 12 bits](#packed-12-bit-transport): O `auto` comprime apenas quando isso é vantajoso e o anfitrião tem capacidade para descomprimir. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituição avançada do seletor de nível de sincronização. |
| `--wire-ceiling-mbps MB_PER_S` | detetado automaticamente | Orçamento de largura de banda do anfitrião em MB/s — a forma CLI do campo **Orçamento de largura de banda**. Reduza-o se o conjunto de discos reportar quadros corrompidos. É guardado com o projeto, pelo que uma reconexão posterior o restaura. |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativa o PTP (os carimbos de data/hora entre câmaras deixam de ser comparáveis). |

`lattice array-list`, `array-status` e `array-disconnect` gerem a sessão persistente. A referência completa aos subcomandos, incluindo o alinhamento (`align-calibrate` / `align-apply`) e as ferramentas de rede, encontra-se na [Referência da CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); os equivalentes no SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) encontram-se na [Referência do SDK](../reference/sdk-reference.md). Em Python, o orçamento de transmissão é `connect_array(..., wire_ceiling_mbps=120)`, e a divisão entre dados corrompidos em tempo real e dados que nunca chegaram encontra-se em [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
