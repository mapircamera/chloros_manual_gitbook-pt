# Definições da câmara

O separador **Câmaras** é a superfície de controlo em tempo real do Chloros

para as câmaras LATTICE: uma área de transmissão principal que mostra todas as câmaras ligadas como mosaicos em tempo real e uma barra lateral que alterna entre três páginas — a **lista de câmaras**, um**painel de definições**(configurações por câmara, de matriz ou de captura — uma de cada vez) e a**Calculadora de Índice**. Esta página documenta todos os controlos na lista de câmaras, no painel de configurações por câmara e no painel de configurações de matriz. Os modos de captura, a seleção do tipo de exportação e o fluxo «Capturar Tudo» encontram-se na página complementar [Definições e Modos de Captura](capture.md).

A secção «Câmaras» aparece na barra lateral assim que o backend do Chloros

estiver pronto. Todos os controlos abaixo comunicam com o backend local através do `127.0.0.1:5000`; as alterações aplicam-se imediatamente à câmara em direto, salvo indicação em contrário.

## Tipos de câmara utilizados nesta página

Os controlos mostram-se ou ocultam-se consoante o tipo de câmara selecionado. O manual utiliza estes termos ao longo de todo o texto:

| Termo | Significado | Canais de filtro |
| --- | --- | --- |
| **Câmara RGB** | LATTICE M3C com o filtro FRGB (o modelo contém `-FRGB`) | Red

ro / Green

/ Blue

ro |
| **Multiespectral Bayer** | LATTICE M3C com FRGN, FOCN ou FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — um filtro de banda estreita, uma banda calibrada | Banda única |
| **Membro da matriz** | Uma câmara ligada como parte de uma matriz sincronizada (visualização combinada ou separada) | De acordo com o seu filtro |

As câmaras RGB

recebem processamento fotométrico (equilíbrio de brancos, perfis de cor, gama); as câmaras multiespectrais e monocromáticas recebem a cadeia radiométrica e ignoram os controlos fotométricos. Os membros da matriz transmitem as definições ao nível do fluxo (formato de píxeis, resolução, agrupamento, disparo, taxa de fotogramas) à matriz — essas linhas passam a ser de leitura exclusiva no painel por câmara e são transferidas para o painel de definições da matriz.

## A área principal de transmissão



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Sem câmaras ligadas, a área de transmissão apresenta uma mensagem inicial **«Ligue uma câmara para começar»**com dois botões:**Ligar Câmara**(verde, abre a caixa de diálogo de ligação de uma única câmara) e**Ligar Matriz** (azul, abre a caixa de diálogo de ligação da matriz). As próprias caixas de diálogo de ligação estão documentadas em [Ligar Câmaras](connecting.md); os conceitos da matriz (sincronização, níveis, largura de banda) em [Matrizes de várias câmaras](arrays.md). Quando abre um projeto guardado que contém câmaras, a mensagem inicial mostra, em vez disso, um indicador de carregamento com «A reabrir N câmaras guardadas…», enquanto o Chloros

restaura os fluxos da última sessão.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superior

| Controlo | O que faz |
| --- | --- |
| **Alternar modo de visualização**| Alterna entre a**visualização em grelha**(todos os mosaicos como células) e a**visualização em lista**(linhas de largura total: cada conjunto combinado como uma linha composta, cada membro de um conjunto**separado** como a sua própria linha com o seu próprio feed e tabela de alvos de calibração, e, em seguida, UMA câmara independente — a ativa — abaixo). Dicas de ferramenta: «Alternar para a visualização em grelha» / «Alternar para a visualização em lista». |
| **Bloqueio da grelha**(cadeado) |**Bloqueado** por predefinição — mosaicos fixos no lugar. Desbloqueie para arrastar e reordenar os mosaicos para qualquer posição (os espaços vazios são preservados). A grelha volta a bloquear-se automaticamente sempre que uma nova câmara se liga. Dicas de ferramenta: «Desbloquear grelha (ativar arrastar de blocos)» / «Bloquear grelha (congelar blocos no lugar)». |
| Controlo deslizante **Zoom do feed** | Tamanho do bloco, desde 60 px até à largura total do contentor. As células mantêm uma proporção de 4:3. Abaixo de 200 px de largura de célula, as sobreposições do nome e dos fps ocultam-se para manter o mosaico limpo. |

### Mosaicos do feed

Cada câmara renderiza um mosaico composto em tempo real; uma câmara pode, adicionalmente, mostrar três mosaicos **divididos por canal** em escala de cinzentos (ver [Divisões de Canal](#overlays-de-exibição-desenhados-sobre-o-feed-ao-vivo)), e as matrizes apresentam um mosaico combinado. O mosaico ativo exibe um anel de seleção na cor da câmara (ou da matriz).

Ao passar o cursor sobre um mosaico, surge um botão de fecho **X**:

* Fechar um mosaico **composto** enquanto as suas divisões de canal permanecem visíveis apenas oculta o mosaico composto.
* Fechar o **último mosaico visível de uma câmara independente** desliga essa câmara.
* **Os mosaicos de divisão de membros de um conjunto combinado nunca desligam** a câmara — apenas a ocultam.

Com a grelha desbloqueada, arraste qualquer mosaico para qualquer espaço; o layout é guardado com o projeto.

## Barra lateral — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

de câmaras A primeira página da barra lateral lista todas as câmaras e matrizes ligadas:

* **Ligar Câmara**(verde) /**Ligar Matriz** (azul, mostra «A detetar...» durante a verificação). Ambas ficam desativadas enquanto estiver aberta uma caixa de diálogo de ligação.
* **Capturar Tudo**(vermelho) — captura todas as câmaras listadas com os tipos de exportação escolhidos nas**Configurações de Captura**. Requer um projeto aberto. Totalmente documentado em [Definições e Modos de Captura](capture.md).
* **Engrenagem de Definições de Captura** (ao lado de «Capturar tudo») — abre o [painel de Definições de Captura](capture.md#the-capture-settings-pane). Desativado sem um projeto ou durante a captura.

### Linhas das câmaras

Cada linha de câmara apresenta uma borda codificada por cores (a cor personalizada da câmara), uma etiqueta «CAM» — com uma letra **M**azul (mestre) ou**S** verde (escravo) para os membros da matriz — e o nome de exibição. O nome predefinido é `LATTICE-MODEL (serial)`; renomeie-o a partir do painel de definições de cada câmara. Botões da linha:

| Botão | Efeito |
| --- | --- |
| **Olho**| Alternar a visibilidade. As câmaras ocultas saem da grelha e são**excluídas da função «Capturar Tudo»**. |
| **Engrenagem** | Abre o painel de definições por câmara (próxima secção). |
| **Pausa / Reproduzir**| Congela a pré-visualização ao vivo**apenas no lado da visualização** — a captura no backend continua a funcionar. As câmaras em pausa não podem capturar. |
| **X** | Desligar. A interface do utilizador atualiza-se imediatamente (na melhor das hipóteses); a desconexão do backend pode demorar entre 10 e 30 s a ser concluída. |

### Linhas da matriz

Uma linha da matriz apresenta um emblema «ARRAY» na cor da matriz, o nome da matriz (renomeável nas definições da matriz) e um botão **DAQ · ligado/desligado**—**ligado** quando o sensor de luz ao nível da matriz está definido *ou* qualquer membro possui um sensor por câmara; a sua dica de ferramenta indica exatamente qual o sensor que alimenta o quê. As câmaras que compõem a matriz são listadas com recuo abaixo, nas suas próprias linhas. Botões da linha da matriz: **olho**(oculta/mostra TODOS os membros em conjunto),**engrenagem**(painel de definições da matriz),**X**(desliga toda a matriz).

O estado do sensor de luz (DLS) utilizado nas linhas da matriz e no painel de definições da matriz tem quatro estados:**desligado**,**em espera**(ainda sem espectro),**ativo**(chegou um espectro nos últimos 3 s) e**desatualizado** — sem espectro recente nos últimos 3 s, mas a última leitura *continua a ser utilizada* (as leituras do DAQ nunca expiram no percurso de captura).

Até que a reordene, a lista mostra primeiro as matrizes, depois as câmaras autónomas, com as câmaras (e os membros de cada matriz) na **ordem das câmaras**: M3C (Bayer) antes de M3M (mono); entre M3C, FRGB, FRGN, FOCN e, por fim, FNGB; M3M por comprimento de onda do filtro, começando pelo mais baixo. O layout predefinido da grelha segue a mesma ordem. Pode arrastar câmaras autónomas e grupos inteiros de matrizes uns sobre os outros na barra lateral para reordenar a lista; os membros das matrizes não podem ser arrastados individualmente.

## Painel de definições por câmara

Abra com a **roda dentada** numa linha de câmara. O painel desliza sobre a lista de câmaras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Cabeçalho**: a**amostra de cor**da câmara (clique para abrir um seletor de cor nativo — define a cor da borda da barra lateral e do anel de seleção de mosaicos), o**nome**com um botão de lápis**Renomear**(guardar um nome vazio reverte para o valor predefinido `MODEL (serial)`) e**×** para fechar.

### Histograma em tempo real

Na parte superior do painel encontra-se um histograma de luminância em tempo real, calculado a partir da pré-visualização JPEG

a cerca de 8 Hz. A média é ponderada pelo método Bayer — (R+2G+B)/4 — para corresponder à medição AE da própria câmara.

* **Linha tracejada Orange**= o alvo de AE. **Arraste-a horizontalmente para redefinir o alvo** — é enviado um comando ao soltar, e o ato de arrastar alterna o modo do alvo de AE para Manual.
* **Linha contínua Green** = a média real de luma (o que a AE está a fornecer atualmente).
* **Botão RGB** (canto superior direito): alterna entre histogramas sobrepostos por banda, coloridos de acordo com o filtro da câmara (por exemplo, no FRGN: NIR

cinzento, verde, vermelho). Em câmaras monocromáticas (M3M), o botão exibe «MONO» e está desativado — o modo monocromático mostra sempre o histograma de luminância de banda única.
* Os rótulos do eixo X seguem a profundidade de bits do sensor do formato de pixel atual: 0..255, 0..1023, 0..4095 ou 0..65535.

### Linhas

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

de informação da câmara

| Linha | Comportamento |
| --- | --- |
| **Modelo** | Apenas leitura (por exemplo, `LATT-M3C-L87-FRGN`). |
| **Calibração radiométrica** | Emblema Green**«Ativo»**com uma legenda que mostra o nível de calibração, o hash, a data de calibração e a lista de bandas, carregados a partir do pacote de calibração da câmara (ver [Calibração radiométrica de fábrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ocultado para câmaras RGB

** — estas possuem uma calibração fotométrica do equilíbrio de brancos, e não radiação por banda. |
| **Relatório de Calibração**| Botão**Descarregar** — abre o PDF do certificado de calibração NIST específico para cada número de série da câmara no visualizador do seu sistema operativo. Se o certificado ainda não estiver armazenado em cache, o Chloros

mostra uma sugestão em vez disso. |
| **Número de série** | Apenas para leitura. |
| **Firmware**| Mostra a versão atual e, em seguida, determina a versão disponível para este modelo (armazenada em cache por modelo — um conjunto de N câmaras verifica o servidor uma única vez). Estados: «A verificar…» → botão**«Atualizar para X»**→ «A atualizar…» → «Atualizado de A para B» / «Falha: …» / «Ignorado: …» / verde**«Atualizado»**. Dica de ferramenta do botão de atualização: «Reinicialização de fábrica + flash + reprogramação do UserSet1. ~2–3 minutos; não desligue.» |
| **IP** | Apenas leitura. |
| **Temperatura** | Apenas leitura, atualizada a cada 3 s. Fica laranja a partir de ≥65 °C e vermelha com um ⚠ a partir de ≥75 °C. |
| **Alvo de calibração** caixa de seleção | Ativa a deteção de alvos de refletância ArUco com uma tabela de validação NDVI

por painel abaixo da transmissão em direto (vista de lista) — consulte [A tabela de alvos de calibração](#the-calibration-target-table). Apenas para a sessão — abre-se sempre desativada. |
| Menu suspenso **Sensor de luz** | Associa um sensor de luz DAQ (DAQ-E/M/U, da lista do separador «Sensores de luz») a esta câmara para correção da iluminação da luz descendente (DLS) e exposição automática preditiva. ««Nenhum» anula a associação. Se não houver sensores ligados, o menu suspenso apresenta «(sem sensores ligados — abra o separador DAQ)». A associação é guardada com o projeto. |

#### A tabela «Alvo de Calibração»

A tabela é uma **leitura apenas para validação** («apenas validação — não modifica a calibração»). O cabeçalho indica o número de série da câmara, o filtro, o número do alvo e o tamanho do marcador em píxeis, e possui uma linha por painel:

* **Painel**, seguido de uma coluna por banda com o valor do painel — uma câmara Bayer apresenta as suas três bandas por nome e comprimento de onda (por exemplo, FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); uma **câmara mono (M3M) apresenta a sua única banda** (por exemplo, `F750`) e não possui colunas NDVI

.
* **Sat** — a percentagem de píxeis do painel que estão saturados (âmbar a partir de 1 %, vermelho a partir de 5 %).
* **NDVI

Medido**,**NDVI

Teórico** (o NDVI

de refletância de referência do painel) e **Δ**— apenas para câmaras multiespectrais Bayer. Uma pequena marca após «NDVI

Medido» indica a partir de que valor foi calculado o NDVI

medido:**ρ**(refletância — está disponível uma leitura do sensor de luz),**L**(radianância — sem leitura do sensor de luz) ou**DN** (valores brutos da câmara). Apenas um NDVI**ρ** é comparável com o NDVI

Teórico: para L e DN, a coluna Δ aparece a cinzento e uma nota indica isso («Sem leitura do sensor de luz: o NDVI

provém da radianância, pelo que Δ não constitui uma verificação de calibração» / «O NDVI

provém dos valores brutos da câmara, pelo que Δ não constitui uma verificação de calibração»). Com ρ, Δ fica a verde se for inferior a 0,02, a âmbar se for inferior a 0,05 e a vermelho nos restantes casos.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Exposição e Ganho

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Todas as entradas numéricas aqui utilizam controlos deslizantes do tipo «manter premido para acelerar»: tocar = ±1, manter premido &gt;1,5 s = ±10, manter premido &gt;3 s = ±100. O valor é enviado para a câmara quando se solta o botão.

| Controlo | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Exposição (us)**| Mín./máx. em tempo real da câmara | Automático | Todos | Tempo de exposição em microssegundos, com um botão de alternância**Automático/Manual**. Automático = exposição automática contínua por parte da câmara. Definir uma exposição manual mantém o**Ganho** em Automático, se assim estiver, permitindo utilizar toda a gama de ganho da câmara, até que edite o ganho ou o defina para Manual. |
| **Ganho (dB)**| Valores mínimo/máximo em tempo real da câmara (por exemplo, até 48 dB) | Manual (desativado) | Todos | Ganho analógico/digital com o seu próprio botão de alternância**Automático/Manual**. |
| **Brilho-alvo da AE**| 0–255 | 80, modo**Automático**| Todos (editável quando a AE ou o ganho automático estão ativados) | O brilho que a AE pretende atingir. Em**Automático**(a predefinição), um controlador interno baseado no histograma seleciona ele próprio o valor-alvo, mantendo a exposição entre 60–75 % do máximo do sensor. Digitar um valor ou arrastar a linha laranja do histograma muda a opção para**Manual**. |
| **Suavização da AE** | 0,5–40, incrementos de 0,1 | 8,0 | Todos | Amortecimento da AE. Dica: «Valor mais baixo = a AE reage mais rapidamente (pode apresentar picos em fps elevados). Valor mais alto = mais suave / mais lento.” Valores muito abaixo do padrão podem fazer com que a AE apresente picos e desestabilize a transmissão em taxas de fotogramas elevadas; 8,0 é o padrão estável. |
| **Região de Interesse da AE**| Caixa de seleção «Ativar» + botão**Apontar**| Desativado | Todos | Quando ativado, o AE mede apenas a região a verde tracejada em vez de todo o fotograma.**Aim** ativa a função «clicar para posicionar» na transmissão em direto: um clique centra uma região a 30 % do fotograma; clicar e arrastar traça um retângulo personalizado (mínimo 5 % × 5 %). O «Aim» desativa-se automaticamente após uma colocação. A região é remapeada para as coordenadas nativas da câmara, tendo em conta qualquer rotação ou espelhamento que tenha definido, e é guardada com o projeto. |
| **Velocidade de Ajuste do AE** | 0,1–5, intervalo de 0,1 | 1,0 | Apenas para membros do Array | A rapidez com que o alvo do AE automático acompanha as alterações de luminosidade da cena; 1,0× verifica novamente a cada 2,5 s. |
| **Proteção de realces** | Rigorosa (1 %) / Normal (5 %) / Flexível (15 %) | Rigorosa | Câmaras que expõem a configuração | A quantidade do quadro que pode ficar esbranquiçada antes de o AE escurecer a imagem. |

{% hint style="info" %}
**Requisito de iluminação para câmaras multiespectrais Bayer (RGN

/ OCN

/ NGB

):** a cena deve ter luz suficiente nos três canais, caso contrário a calibração não funciona corretamente — uma única exposição do sensor abrange os três espectros. Utilize um sensor de luz DAQ para medir a iluminação ou opte pelo modo totalmente monocromático (M3M), para que cada banda tenha a sua própria exposição. Se uma captura violar esta condição, o Chloros

deteta-a e avisa-o (notificação de limitação da separação de canais). Em tempo real, um mosaico de divisão de canais mostra **«Dados da banda com baixa confiança · N%»** quando o unmix limitou essa banda em 25 % ou mais dos seus píxeis.
{% endhint %}

{% hint style="info" %}
**Como a exposição automática distribui a exposição e o ganho (1.3.0).** Com o ganho em «Auto», o Chloros

mantém o ganho em 0 dB até que a exposição atinja o valor máximo permitido (o limite superior da AE, o período de disparo de uma matriz ou o limite máximo de exposição do DAQ); só então é que o ganho aumenta. Numa matriz, a taxa de disparo tem prioridade: a exposição automática preenche o período definido pela taxae, em seguida, aumenta o ganho; assim, uma cena que escurece já não reduz a taxa de disparo automática (na versão 1.2.0, isso acontecia). Uma exposição **manual**num membro da matriz com duração superior ao período de disparo retarda o disparo da matriz para se adaptar a essa duração; a taxa automática é restabelecida assim que todos os elementos que a tinham abrandado voltarem à exposição automática. Com a**Exposição**e o**Brilho-alvo da AE** ambos em Automático e um sensor de luz DAQ a fornecer uma leitura, a exposição de cada câmara é mantida assim que a exposição automática se estabilizar, desde que a luz medida se mantenha dentro de cerca de ±25 %; uma variação de luz mais significativa e prolongada, um reinício das definições (por exemplo, uma alteração da ROI) ou uma alteração de exposição por parte do utilizador reativa-a.
{% endhint %}

{% hint style="info" %}
**Como funciona a exposição automática (1.3.0).**

* Uma exposição manual mantém o ganho em «Auto» até que o altere.
* Pequenos realces não escurecem a cena: um realce especular até 1 % do enquadramento é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5 % de um painel.
* O limite máximo de exposição do DAQ aplica-se apenas enquanto um alvo de calibração estiver visível; sem nenhum painel visível, a câmara expõe para a cena.
* A exposição mantém-se estável sob luz constante e estabiliza-se poucos segundos após uma grande alteração. Um recorte já não limita a exposição de forma definitiva, e uma alteração na área de interesse (ROI) restabelece a exposição.
* O alvo de brilho não ultrapassa oalcance da câmara, pelo que o regresso da luz não provoca reflexos no enquadramento.
* Uma câmara cuja exposição automática deixe de responder sob sol intenso mantém-se no nível adequado à luz em que se encontra, lido pelo sensor de luz ou, na ausência deste, pelos próprios fotogramas da câmara, e é libertada quando a luz muda. Um sensor de luz escuro ou coberto não perturba este processo.
{% endhint %}

### Formato de Píxeis e Resolução

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Os membros da matriz** apresentam as linhas de só leitura «Atual» (formato + LxA) e «Agrupamento» com a nota «Definido nas definições da matriz» — um reinício do fluxo num membro quebraria a sincronização, pelo que estas são geridas no [painel de definições da matriz](#painel-de-configurações-da-matriz).**As câmaras autónomas** têm:

| Controlo | Opções | O que faz |
| --- | --- | --- |
| **Formato de Píxeis** | 8 bits / 10 bits / 12 bits / 16 bits | Profundidade de bits do sensor; a família de formatos é a do sensor. O valor predefinido é 12 bits, e um projeto guarda o que definir, incluindo 8 bits, e restabelece a ligação com essa configuração. |
| **Resolução** | Completa / Metade / Um quarto | Relativa ao binning atual: Completa = 2048/N × 1536/N para binning N×N. |
| **Binning** | 1x1 (nenhum) / 2x2 / 4x4 | Binning N×N por hardware — valores maiores reduzem a resolução, mas aumentam a relação sinal-ruído (SNR) e a taxa de fotogramas. A alteração reinicia o fluxo e redefine qualquer ROI para o novo campo de visão completo. |
| **Atual** | só de leitura | As dimensões WxH reais e o desvio (x, y) atualmente em vigor. |

### Pré-visualização ao vivo

Tudo nesta secção é **apenas do lado da visualização**— altera o que vê na transmissão ao vivo, enquanto as capturas guardadas permanecem lineares e inalteradas — com uma exceção: a**Vignette** é radiométrica e afeta também as exportações (conforme indicado abaixo).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Controlo | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Resolução de renderização** | 360p (mais rápida) / 480p / 720p / 1080p / Resolução nativa do sensor (mais lenta) | 720p | Todos | A altura à qual o backend executa a cadeia de pré-visualização radiométrica. Um valor mais baixo proporciona uma taxa de fotogramas mais elevada sem alterar o campo de visão. |
| **Índice**| Caixa de seleção «Ativar» + engrenagem | Desativado | Apenas multiespectral Bayer,**não** membros de matriz combinada | Pré-visualização em tempo real do índice de vegetação. A engrenagem abre a [Calculadora de Índice](#index-calculator-pane) partilhada, pré-carregada com as bandas naturais do filtro da câmara (por exemplo, `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). A expressão personalizada, juntamente com a LUT (ligado/desligado, nível predefinido de 7 stops, mínimo predefinido de 0,2, máximo predefinido de 1) é calculada em cada fotograma de pré-visualização. Os membros da matriz combinada ocultam esta linha — a matriz possui um único índice partilhado. |
| **Equilíbrio de brancos** | Desligado / Uma vez / Contínuo + um botão de recaptura | Contínuo | Apenas RGB

| Equilíbrio de brancos em tempo real. O botão de atualização volta a capturar o equilíbrio de brancos a partir do espectro DLS atual (desativado quando o modo está em Desligado). |
| **Gama** | Ligado / Desligado | Ligado | Apenas RGB

| Apresenta a gama (γ = 2,2 LUT) na pré-visualização em tempo real. As capturas guardadas permanecem lineares. |
| **Redução de ruído** | Caixa de seleção + intensidade 0–100 | Desligado / 50 | Todos (por câmara, mesmo dentro de matrizes) | Filtro bilateral na pré-visualização em tempo real. Valores mais elevados = detalhes mais suaves, mas menos nítidos. |
| **Nitidez** | Caixa de seleção + intensidade 0–100 | Desligado / 30 | Todos | Máscara de nitidez na pré-visualização em tempo real, aplicada em último lugar. Pode amplificar o ruído. Apenas na pré-visualização. |
| **Vinheta**| Caixa de seleção + intensidade 0–100 | Desativado / 0 | Todas | Remoção manual da vinheta residual (clareia os cantos), sobreposta à estimativa de Vinheta Inteligente do conjunto.**Radiométrico — afeta a visualização em tempo real E as exportações**, ao contrário do Redutor de ruído/Nitidez. |
| **Perfil de cor** | Raw / Linear / Natural / Melhorado / Temperatura personalizada | Natural | Apenas RGB

| Ver abaixo. |
| **Temperatura de cor** | 2000–10000 K, intervalo de 100 | 5500 K | Apenas RGB

e perfil de temperatura personalizada | Fixa o equilíbrio de brancos numa temperatura de cor correlacionada fixa (a entrada DLS é ignorada). O último valor em Kelvin selecionado é memorizado mesmo quando se muda de perfil. |
| **Destaques** | −100 a 0, em incrementos de 1 | 0 (desativado) | Apenas RGB

| Novidade na versão 1.3.0. Escurece as áreas mais brilhantes e mais brancas da imagem (0 = inalterado, −100 = mais intenso). Aplica-se à pré-visualização em tempo real **e às capturas ao nível do ecrã (Pré-visualização)**, que registam a configuração (`Chloros:DisplayHighlights`); as exportações de radiação e refletância nunca são afetadas. Não é possível recuperar dados do sensor que tenham sofrido clipping — para tal, reduza a exposição. A sobreposição zebra acompanha a imagem escurecida. Desativado com o perfil de cor Raw («Não aplicado com o perfil de cor Raw.»). |
| **Saturação** | 0–200 (100 = neutro) | 100 | Apenas RGB

| Saturação HSV na pré-visualização em tempo real. |
| **Contraste** | 0–200 (100 = neutro) | 100 | Apenas RGB

| Contraste linear em torno do cinzento médio na pré-visualização em tempo real. |
| **Espelhar H / Espelhar V** | Caixas de seleção | Desativado | Todos | Inverte a pré-visualização na horizontal / na vertical. |
| **Rotação**| 0° / 90° / 180° / 270° | 0° | Todos | Rota a pré-visualização. A orientação é aplicada no final da cadeia de pré-visualização do backend.**Capturas guardadas (1.3.0):**os produtos Pré-visualização, Radiação, Refletância e Índice de uma câmara autónoma são invertidos por Espelhamento/Rotação, enquanto**os ficheiros Raw e Debayered mantêm a orientação nativa da câmara**; cada ficheiro de captura regista a configuração no seu XMP, pelo que o processamento posterior de um ficheiro raw aplica-a da mesma forma.**Os ficheiros membros de um conjunto mantêm a orientação nativa** (a configuração é registada, não aplicada), e as visualizações compostas do conjunto ignoram-na. |**Semântica dos perfis de cor** (câmaras RGB

):

* **Raw** — ignora completamente a cadeia de processamento.
* **Linear** — sinal escuro + campo plano + equilíbrio de brancos; sem matriz de cor, sem gama.
* **Natural** *(predefinição)* — linear mais a matriz de correção de cor medida e uma curva de tom adaptativa à cena.
* **Melhorado**— Natural mais vibração e contraste local CLAHE. O custo adicional aplica-se**apenas à pré-visualização em tempo real** — as capturas guardadas recebem sempre o acabamento completo, independentemente do perfil.
* **Temperatura personalizada** — Natural com o equilíbrio de brancos fixado no valor em Kelvin que escolher.

{% hint style="warning" %}
Para os modos Natural, Aprimorado e Temperatura Personalizada, o painel apresenta uma nota sobre o tom: os fotogramas são clareados de acordo com a sua própria cena, pelo que as imagens *de visualização* guardadas não são comparáveis entre si. **Exporte a radiação ou a refletância para medições.**
{% endhint %}

### Sobreposições de visualização (desenhadas sobre a transmissão em direto)

Estas são exclusivas do front-end — são aplicadas sobre o vídeo, sem nunca afetar a transmissão ou as capturas.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sobreposição | Controlos | Padrão | O que faz |
| --- | --- | --- | --- |
| **Zebra** | Caixa de seleção + limiar 200–255 | Desativado / 250 | Listras diagonais magenta nos píxeis cortados. |
| **Retículo** | Caixa de seleção | Desativado | Marca no centro do fotograma. |
| **Grelha** | Desativado / 3 × 3 / 9 × 9 | Desativado | Grelha de composição. |
| **Histograma** | Caixa de seleção + largura 0,10–0,90 do quadro | Desativado / 0,25 | Uma faixa de histograma no feed. |
| **Pico de Focagem** | Caixa de seleção + limiar 20–200 + amostra de cor | Desativado / 80 / `#ff5722` | Destaque de bordas Sobel para focagem. |
| **Divisões de Canal** | «Mostrar Divisões (Red

/ Green

/ NIR

)» / botão «Ocultar divisões» | Oculto | Adiciona três mosaicos independentes em escala de cinzentos, um por canal, ao lado da imagem composta (o rótulo do botão segue os canais de filtro da câmara). Cada mosaico de divisão pode ser arrastado e partilha a cor da moldura da câmara. Não disponível em câmaras monocromáticas. Guardado com o projeto. |

### Medidor pontual

* Caixa de seleção **Clicar para amostrar**: clique na imagem em direto para amostrar um único pixel (uma cruz marca-o) ou clique e arraste uma região para obter uma média de pixels.**Limpar**apaga a amostra e a cruz. É mutuamente exclusivo com o modo**Aim** do AE-ROI.
* **Menu suspenso**Mostrar**:**Raw (profundidade de bits)**— valores digitais nativos com a profundidade de bits do sensor (por exemplo, 12 bits → 0..4095) — ou**Exibição (8 bits)** (predefinição). Quando um índice em tempo real está ativo, a opção «Exibição» mostra, em vez disso, o valor do índice calculado (por exemplo, NDVI

).
* O painel de leitura apresenta as coordenadas dos píxeis, o tamanho do fotograma, o formato dos píxeis, a profundidade de bits e uma tabela de canais (Canal / Valor / %) com rótulos de banda e comprimentos de onda; os pares verdes de Bayer são calculados como média; as amostras da região apresentam «Média de N píxeis».

O estado do medidor pontual é válido apenas para a sessão.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposição Automática Preditiva (controlada por DLS)

Esta secção só aparece quando **pelo menos um sensor de luz DAQ estiver ligado** — o solucionador necessita de um espectro descendente em tempo real para o controlar.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Controlo | Intervalo | Predefinição | O que faz |
| --- | --- | --- | --- |
| **Ativar** | Caixa de seleção | Ligado (câmaras autónomas) | Um solucionador de forma fechada utiliza o espectro do DLS, juntamente com os escalares do pacote de calibração da câmara, para ajustar a faixa mais brilhante perto da saturação, mantendo a faixa mais escura acima do limiar de SNR — uma única exposição por resolução, sem ciclo de estabilização. Concebido para timelapse alimentado a energia solar, em que cada fotografia deve ser corretamente exposta. O backend recorre silenciosamente à exposição automática reativa sempre que a leitura do DLS estiver desatualizada/ausente ou o pacote de calibração não estiver carregado. |
| **Suavização (α)** | 0,05–1,0, passo de 0,05 | 0,3 | Suavização de soluções preditivas sucessivas (valor mais baixo = suavização maior). |
| **Refletância da cena**| Botão**Recalibrar ρ** | — | Reestima o fator de refletância da cena utilizado pelo solucionador. |

{% hint style="info" %}
**A ligação em matriz desativa a AE preditiva por predefinição** — para matrizes, a AE inteligente da Chloros

, juntamente com a exposição automática do lado da câmara, controla a exposição (com proteção contra saturação) e a estimativa de refletância de cena única da AE preditivanão é segura em cenas mistas. Pode reativá-la por câmara aqui, se pretender especificamente uma exposição radiométrica orientada por DLS.
{% endhint %}

**Limite máximo de exposição orientado por DAQ e AE fixada na luz incidente.** Independentemente da caixa de seleção acima, quando um sensor de luz DAQ é atribuído a uma câmara RGB

, o Chloros

calcula — a partir da irradiação descendente absoluta medida — a exposição máxima × ganho na qual uma superfície com 100 % de refletância permanece abaixo do clipping, e aplica-a como um **limite máximo**na exposição automática. Desde a versão 1.3.0, o limite máximo aplica-se apenas enquanto um alvo de calibração estiver no campo de visão dessa câmara (detetado no seu enquadramento nos últimos 10 s); sem nenhum alvo no campo de visão, a exposição automática normal da câmara, com a sua própria proteção contra realces, é ativada, para que o resto da cena não fique mais escuro do que o necessário. Enquanto o limite máximo estiver ativo, a câmara está**fixada na luz incidente**: funciona em malha aberta com a exposição medida pela luz incidente e com o ganho a 0 dB — a exposição segue a luz medida, não o conteúdo da cena. Como o limite máximo só pode encurtar a exposição, não pode, por si só, provocar recorte. Desde a versão 1.3.0, o limite máximo é calculado com a constante de exposição própria da câmara, proveniente do seu pacote de calibração (um valor médio da frota apenas quando o pacote não inclui um). O limite máximo desativa-se automaticamente — e a exposição automática (AE) normal da cena é retomada — sempre que não houver nenhum alvo de calibração no campo de visão, a leitura do DAQ estiver em falta, desatualizada (&gt;30 s), ou escura, ou se ≥15 % do fotograma sofrer clipping na exposição fixada (o que significa que o sensor e a câmara estão a detetar iluminação diferente). Não existe qualquer opção na GUI; este é o comportamento padrão sempre que uma câmara RGB

tem um DAQ associado.

### Aquisição e Acionamento

Os membros da<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

matriz apresentam adicionalmente as linhas de só leitura **Função**(Mestre a azul / Escravo a verde),**Linha de Sincronização**e**Parceiros**.

| Controlo | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **Modo de Disparo** | Desligado / Ligado | Ligado | Desativado para membros da matriz (a matriz gere o disparo). |
| **Fonte de Disparo** | Software / Linha 0 (M8) / Linha 1 / Linha 2 | Linha 0 | Oculta enquanto o Modo de Disparo estiver Desligado; desativada para membros da matriz. A Linha 0 é a entrada de disparo externa opto-isolada M8. |
| **Taxa de fotogramas**| Automático / Manual + valor | Automático |**Automático**: o limite de taxa de fotogramas da câmara está desativado — a exposição determina os fps, e a caixa mostra a taxa real em tempo real.**Manual**: limita os fps com um controlo deslizante (de 1 até ao máximo limitado pela largura de banda), com base na taxa real atual. Os membros da matriz veem um valor de «N fps (em tempo real)» apenas para leitura com a indicação «Definido nas definições da matriz». |

### Rede / Transporte

| Linha | Comportamento |
| --- | --- |
| **Tamanho do pacote**| 1500 (Padrão) / 9000 (Jumbo) — predefinição**Jumbo**. |
| **Largura de banda** | Limite de largura de banda do link, de leitura apenas, em MB/s. O backend reequilibra este valor entre todas as câmaras ligadas sempre que há uma ligação ou desligamento. |
| **Gestão do buffer** | Modo de gestão do buffer, de leitura apenas. |

### Captura

O painel termina com um botão **«Abrir definições de captura…»** que leva ao [painel de definições de captura](capture.md#the-capture-settings-pane) (desativado até que um projeto esteja aberto — «Crie ou abra um projeto para guardar as capturas»). Se a câmara estiver oculta ou em pausa, uma dica lembra-o de a revelar/retomar antes de capturar.

## Painel de definições da matriz

Abra com a **roda dentada**numa linha de MATRIZ. Cabeçalho: nome da matriz com um lápis para renomear e**×** para fechar. As secções abaixo marcadas com *apenas combinadas* aparecem apenas para matrizes ligadas no modo de visualização combinada.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sincronização

Linhas **Mestre**,**Escravos**e**Linha de Sincronização** apenas para leitura.

### Sensor de luz ambiente

Exibido tanto para matrizes combinadas como separadas:

* Menu suspenso **Sensor de luz**— associa um DAQ a toda a matriz. A seleção é aplicada imediatamente, propaga-se para o menu suspenso**Sensor de luz** de cada câmara membro (ainda é possível substituir a configuração por câmara) e começa a encaminhar espectros para a matriz.
* Linha **Estado** em tempo real: Desligado · «À espera do primeiro espectro…» · «Ativo — todas as câmaras da matriz têm a iluminação corrigida» · «Sem espectros novos nos últimos 3 s — continua a utilizar a última leitura (sem tempo limite para dados desatualizados)…».
* Nota no painel: «Correção radiométrica em toda a matriz. As definições por câmara substituem esta.»

### Alvo de calibração

O seu próprio grupo imediatamente a seguir ao «Sensor de luz ambiente», tanto para matrizes combinadas como separadas:

* Caixa de seleção **Ativar** — «Detetar alvo ArUco da MAPIR

e validar NDVI

vs. LUT de refletância do painel»; controla a sobreposição do alvo e as tabelas de validação do mosaico combinado. Num conjunto **separado**, ativa a [tabela de alvo de calibração](#the-calibration-target-table) de cada membro (uma por linha de câmara na vista de lista).

### Captura — definições uniformes do sensor

Apresentado tanto para matrizes combinadas como separadas (o cabeçalho do grupo indica «Captura (uniforme em todas as câmaras)»). Estas definições aplicam-se uniformemente a todos os membros (alterações por membro quebrariam a sincronização). As edições são preparadas e aplicadas em conjunto com **Aplicar**— exceto o botão de alternância**ROI ativado / ROI desativado**, que se aplica imediatamente.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Controlo | Opções / intervalo | O que faz |
| --- | --- | --- |
| **Formato de pixel** | 8 bits / 10 bits / 12 bits / 16 bits | Profundidade de bits uniforme para todos os membros; cada câmara adota o formato dessa profundidade para o seu próprio sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning por hardware — mantém o campo de visão completo, ao mesmo tempo que aumenta a relação sinal-ruído (SNR) e a taxa de fotogramas. Alterá-lo redefine os campos da ROI para o novo campo de visão completo. |
| Predefinição de **resolução** | Completa / Metade / Um quarto | Relativa ao binning; preenche os campos da ROI com um recorte centrado. |
| **Recorte ROI (px)**| Alternador**ROI ativado / ROI desativado**, botão**Selecionar**, campos L / A / X / Y, mapa do sensor completo | Recorte do sensor — consulte [O editor de ROI](#the-roi-editor). A largura e a altura ajustam-se a múltiplos de 16 (mínimo 64); os deslocamentos ajustam-se a múltiplos de 4. Uma dica «max WxH» indica o limite máximo e**Reiniciar** repõe os campos para o campo de visão completo. Durante a edição, a caixa de recorte laranja acompanha todas as alterações nos campos no mapa e no mosaico do conjunto de sensores. |
| **Taxa de disparo**| Alternar entre Automático / Manual + fps 0,5–10, passo de 0,5 |**Automático**(predefinição): o backend calcula a taxa de disparo com base na resolução e na largura de banda — a entrada está desativada e apresenta o valor calculado.**Manual**: fixa o seu valor ao clicar em «Aplicar». Uma taxa fixada que um novo tamanho de fotograma não consiga suportar é mantida na taxa mais rápida que esse tamanho permite, e volta ao valor original quando o tamanho do fotograma o permitir novamente. |

Nota no painel: «Alterações de formato/resolução reiniciam todas as câmaras brevemente. A taxa de disparo aplica-se em tempo real.» Os botões **Aplicar / Cancelar** encontram-se na parte inferior do painel.

#### O editor de ROI

Novidade na versão 1.3.0. A linha de recorte da ROI e o mapa abaixo dela funcionam em conjunto:

* **ROI ativada / ROI desativada**— ativa ou desativa o recorte do sensor**imediatamente**(as câmaras reiniciam-se brevemente; o mosaico da matriz mostra «A aplicar definições…»).**Desativado**lê o sensor na íntegra e**guarda a sua área de recorte**— os campos continuam a mostrá-la;**Ativado** aplica-a novamente. Ativar a ROI sem nenhuma área de recorte definida aplica um recorte centrado com metade do tamanho («Não foi definida nenhuma área de recorte, pelo que foi aplicado um recorte centrado com metade do tamanho. Ajuste-o através dos campos.»).
* **Selecionar**— desenha o recorte no mapa: arraste pelo recorte ou clique em dois cantos opostos (Esc cancela). O retângulo é registado nos campos L/A/X/Y, alinhado de acordo com os passos acima; nada é aplicado até premir**Aplicar**, pelo que pode ajustá-lo primeiro com as setas dos campos. Um retângulo por cada vez que premir**Selecionar**.
* **O mapa**mostra o recorte em todo o sensor, pelo que pode alargá-lo novamente: «Sensor completo, em tempo real» enquanto as câmaras lêem o sensor completo, ou «Instantâneo de quadro completo de {hora}. As câmaras lêem apenas o recorte», enquanto estão recortadas. Se o conjunto de câmaras só tiver transmitido imagens recortadas, o mapa exibe «Ainda sem visualização de quadro completo: este conjunto de câmaras só transmitiu imagens recortadas.» com um botão**Mostrar quadro completo**, que desativa a ROI. A etiqueta `W×H @ X,Y` do recorte fica fora da caixa (acima dela, ou abaixo quando a caixa toca na parte superior), no mapa e no mosaico em tempo real.
* **Aplicar**confirma os campos: um recorte ativa a ROI; o quadro completo (após**Reiniciar**) mantém-na desativada, com o recorte anterior ainda memorizado; uma ROI que esteja desativada e cujos campos ainda mostrem o recorte memorizado permanece desativada quando se aplica outra alteração (formato, binning, taxa).
* **Numa matriz alinhada, a janela de recorte de cada membro é posicionada pelo alinhamento**, de modo que todas as câmaras enquadram a mesma área da cena. Uma matriz recortada antes de ter sido alinhada é primeiro alinhada no sensor completo (o recorte é removido, o alinhamento resolvido e o recorte restaurado com janelas específicas para cada câmara).
* Após uma alteração no recorte, a exposição automática reconverge para o novo enquadramento. Sob um recorte, a exposição de um membro pode continuar até ao período de disparo.
* O estado de ativação/desativação da ROI e o recorte memorizado são guardados com o projeto.

### Alinhamento (co-registo)

Apresentado para **matrizes combinadas e separadas** (a versão 1.2.0 apresentava-o apenas para matrizes combinadas); o cabeçalho do grupo indica «Alinhamento».

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Caixa **Residual**: «RMS x,xx px» — verde se for inferior a 1 px, âmbar se for inferior a 3 px, vermelho nos restantes casos ou se alguma câmara falhar; «sem perfil» antes da primeira resolução.
* Linha de resumo: «✓ Todas as câmaras alinhadas (N)» / «⚠ p/N câmaras alinhadas — <serial (filter)="">falha» / «Recorte ativo — Recalibrar para alinhar (utiliza o sensor completo)» / «À espera que a exposição estabilize…» / «A aplicar definições…» (as câmaras estão a reiniciar) / «O conjunto deixou de enviar fotogramas — volte a ligar o conjunto».
* Tabela por câmara: câmara (últimos 4 dígitos do número de série + filtro), erro de reprojeção em px com contagem de correspondências («ref» para a câmara principal) e a pontuação de correlação cruzada normalizada da sobreposição em relação ao limiar mínimo de aprovação de 0,35.
* **Recalibrar alinhamento** (exibe «Calibrar alinhamento» antes do primeiro perfil) — executa novamente o co-registo em fotogramas novos.
* **Caixa de seleção «Exposição automática das câmaras para alinhamento»** (marcada por predefinição) — clareia temporariamente as câmaras escuras ou sem contraste (primeiro a exposição, depois o ganho) para que tenham textura para corresponder, e depois restaura a exposição automática.

O Chloros alinha automaticamente cada matriz após a ligação, em ambos os modos de visualização, assim que a exposição automática dos membros se estabilizar (continua a tentar durante um máximo de 5 minutos); a pré-visualização combinada também se alinha ao abrir. Uma matriz separada não tem mosaico composto, pelo que esta secção verifica o estado do alinhamento e executa a própria «Recalibrar». Recalibre se o foco ou a profundidade da cena tiverem mudado. O alinhamento é **projetado para ser válido apenas durante a sessão** — nunca é guardado num perfil, porque depende da distância da cena naquele momento. As capturas ainda podem ser exportadas com registo de píxeis (ver [Exportações alinhadas](capture.md#per-array-controls)).

### Vinheta Inteligente

* Caixa de seleção **Ativar correção**— aplica a estimativa de vinheta por câmara à cadeia radiométrica (em tempo real**e** nas exportações).
* **Calibrar a partir da vista atual**— aponte primeiro a matriz para um alvo uniforme (ecrã plano, parede ou céu); cada câmara é nivelada individualmente e o estado indica «n/N câmaras · −x,x %» de ganho de nivelamento.**Limpar** remove a estimativa.
* Estado: «Calibrado», «Não calibrado» ou «Parcialmente calibrado — repetir» quando apenas as estimativas de alguns membros ainda se aplicam no seu enquadramento atual (por exemplo, após uma «Recalibração» ter alterado a janela de recorte de um membro) — execute «Calibrar» novamente.
* Ajuste com precisão por câmara com o controlo deslizante **Vignette** específico de cada câmara em [Pré-visualização ao vivo](#live-preview).

### Pré-visualização ao vivo

* Caixa de seleção **Histograma**+ controlo deslizante de largura (0,10–0,90 do quadro) — a faixa do histograma no feed em todos os mosaicos dos membros de uma só vez (incluindo mosaicos de canais divididos), a [Sobreposição de histograma](#display-overlays-drawn-over-the-live-feed) definida para todo o conjunto. Aparece como ativada quando todos os membros já a têm. Num conjunto**separado**, esta é a única linha do grupo.
* *Apenas combinado:* **Índice**: caixa de seleção «Ativar» + engrenagem — abre a [Calculadora de Índice](#index-calculator-pane) partilhada com faixas desenhadas a partir de**todas** as câmaras dos membros. Uma linha de pré-visualização da expressão, logo abaixo, mostra a expressão atual («Nenhuma expressão definida — abra a calculadora para criar uma»), atualizada a cada segundo.
* *Apenas combinado:* Menu suspenso **Resolução de renderização**(as mesmas predefinições que por câmara, predefinição 720p): a altura do fluxo de visualização em direto**e** o tamanho de exportação da composição guardada. Nota no painel: «Pré-visualização + tamanho da composição guardada. As imagens por câmara são sempre exportadas em resolução total.»

### Camadas de exibição *(apenas na visualização combinada)** Caixa de seleção **Ativar** (desativada por predefinição — a câmara principal é apresentada diretamente; ativada = composição em camadas).
* **Menus suspensos**Primeiro plano**/**Fundo**: cada câmara membro (por nome) ou**Índice**. Quando o Primeiro plano é Índice, os píxeis fora dos valores Mín./Máx. da LUT mostram a camada de Fundo.

### Vista dividida *(apenas combinada)*

**«Mostrar câmaras membros»**— um botão**Dividir / Ocultar câmaras membros** que adiciona a transmissão em direto de cada câmara membro como mosaicos separados na grelha, ao lado da composição. Os mosaicos leem o buffer de fotogramas existente da matriz (sem ligação adicional de câmara). Apenas na vista em grelha; guardado por matriz com o projeto.

### Funcionalidades

Um painel de leitura apenas atualizado a cada 5 s:

* **Rótulo do nível**: «Captura simultânea» (verde) · «Captura simultânea (emissão escalonada FTD)» (verde) · «Captura escalonada (desvio de 100 ms)» (âmbar) · «Configuração demasiado grande» (vermelho).
* **Estado do quadro**: «x,xx % incompleto» — verde abaixo de 1 %, âmbar abaixo de 5 %, vermelho a partir de 5 %.
* ****Linha de ligação**: «NIC {mbps} Mbps - sustentada {MB/s} MB/s».

Este é o orçamento de largura de banda em tempo real do conjunto de câmaras. Para obter informações sobre os fps subjacentes e o modelo de rede — e o que alterar quando o nível fica âmbar ou vermelho — consulte [Matrizes multicâmara](arrays.md) e a [Referência da CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Painel «Calculadora de Índice»

A terceira página da barra lateral, partilhada pela engrenagem «Índice» por câmara e pela engrenagem «Índice» da matriz combinada (uma de cada vez — o título indica «Calculadora de Índice — <camera name="">» ou «Calculadora de Índice — <array name="">»). Recebe a lista de bandas (as bandas naturais do filtro— bandas naturais ou todas as bandas dos membros da matriz), a expressão atual e a configuração da LUT (ligada/desligada, nível — predefinição de 7 stops, mínimo — predefinição de 0,2, máximo — predefinição de 1), além de um histograma de índice em tempo real. **Aplicar** confirma a expressão; as alterações na LUT aplicam-se em tempo real à pré-visualização.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Definições por câmara vs. definidas pela matriz

Referência rápida sobre o que se encontra onde quando uma câmara é membro de uma matriz:

| Geridas pela matriz (apenas leitura no painel da câmara) | Ainda por câmara dentro de uma matriz |
| --- | --- |
| Formato de píxeis, resolução, binning | Exposição automática (exposição, ganho, alvo, suavização, ROI) |
| Modo/fonte de disparo, taxa de fotogramas | Redução de ruído, nitidez, vinheta |
| | Orientação (espelhamento/rotação), sobreposições de ecrã (configurações da matriz → a Pré-visualização em Tempo Real define o histograma em todos os membros de uma só vez), medidor pontual |
| | Índice (matrizes com ecrãs separados), ligação do sensor de luz |

Outros comportamentos transversais:

* **Exibição combinada vs. separada** é escolhida na ligação da matriz: combinada = um mosaico composto alinhado (as imagens dos membros são transmitidas apenas através da Vista Dividida); separada = cada membro apresenta o seu próprio mosaico sincronizado. Uma câmara nunca mostra simultaneamente uma imagem independente e um mosaico da matriz.
* ****Reconexão automática**: ao abrir um projeto guardado, as câmaras e matrizes são restauradas e todas as definições guardadas são reaplicadas ao backend antes de os fluxos serem retomados.
* **Controlo de captura**: as câmaras ocultas ou em pausa são excluídas da opção «Capturar Tudo»; uma matriz só fica totalmente bloqueada quando TODOS os membros estiverem ocultos/em pausa. Consulte [Definições e modos de captura](capture.md).

## Como as definições são mantidas

O estado do separador da câmara é guardado **com o projeto**, não no navegador:

* Cada alteração reativa cria um instantâneo das câmaras e matrizes no ficheiro `cameras.json` do projeto (com tempo de espera de 500 ms). Isto inclui nomes e cores das câmaras, definições de exposição/ganho/AE, formato de píxeis/resolução/binning, taxa de disparo, definições de pré-visualização (resolução de renderização, redução de ruído, nitidez, vinheta, perfil de cor, realces, saturação/contraste), orientação, sobreposições, divisões de canais, configuração do índice, definições de AE preditivo, ROI de AE, nomes de matrizes, modo de exibição, definições de captura da matriz (incluindo o recorte da ROI, se a ROI está ativada e o recorte memorizado enquanto está desativada), o «Wire Budget» da matriz e o bloco de grelha (zoom de feed, modo de visualização, bloqueio da grelha, ordem manual dos mosaicos, câmaras ocultas, mosaicos fechados, câmara ativa).
* As ligações dos sensores de luz são guardadas no ficheiro `sensors.json` do projeto.
* Ao reabrir o projeto, o hardware é reconectado e todas as configurações são reaplicadas.
* **Sem projeto aberto = apenas sessão**: sem projeto, nada sobrevive ao encerramento do Chloros.
* Apenas sessão, independentemente do projeto: estado de pausa, amostras do medidor pontual, a caixa de seleção «Alvo de calibração» por câmara (que se abre sempre desativada) e o perfil de alinhamento da matriz (recalculado por sessão, por definição).
* Uma exceção: as seleções de exportação das **Definições de Captura** e o modo de captura permanecem por projeto no armazenamento local da aplicação, em vez de no `cameras.json` — consulte [Definições e Modos de Captura](capture.md).</array></camera></serial>
