# Definições da câmara

O separador **Câmaras**é a superfície de controlo em tempo real do Chloros para as câmaras LATTICE: uma área de transmissão principal que mostra todas as câmaras ligadas como mosaicos em tempo real e uma barra lateral que alterna entre três páginas — a**lista de câmaras**, um**painel de definições**(configurações por câmara, de matriz ou de captura — uma de cada vez) e a**Calculadora de Índice**. Esta página documenta todos os controlos na lista de câmaras, no painel de configurações por câmara e no painel de configurações de matriz. Os modos de captura, a seleção do tipo de exportação e o fluxo «Capturar Tudo» encontram-se na página complementar [Configurações de Captura e Modos](capture.md).

O separador «Câmaras» aparece na barra lateral assim que o backend Chloros estiver pronto. Todos os controlos abaixo comunicam com o backend local através do `127.0.0.1:5000`; as alterações aplicam-se imediatamente à câmara em tempo real, salvo indicação em contrário.

## Tipos de câmara utilizados nesta página

Os controlos mostram-se ou ocultam-se consoante o tipo de câmara selecionado. O manual utiliza estes termos ao longo de todo o texto:

| Termo | Significado | Canais de filtro |
| --- | --- | --- |
| **Câmara RGB** | LATTICE M3C com o filtro FRGB (o modelo contém `-FRGB`) | Red / Green / Blue |
| **Multiespectral Bayer** | LATTICE M3C com FRGN, FOCN ou FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — um filtro de banda estreita, uma banda calibrada | Banda única |
| **Membro de matriz** | Uma câmara ligada como parte de uma matriz sincronizada (visualização combinada ou separada) | Conforme o seu filtro |

As câmaras RGB recebem processamento fotométrico (equilíbrio de brancos, perfis de cor, gama); as câmaras multiespectrais e monocromáticas passam pela cadeia radiométrica e ignoram os controlos fotométricos. Os membros da matriz transferem as definições ao nível do fluxo (formato de píxeis, resolução, binning, disparador, taxa de fotogramas) para a matriz — essas linhas passam a ser de leitura apenas no painel por câmara e são transferidas para o painel de definições da matriz.

## A área

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

principal de transmissão Sem câmaras ligadas, a área de transmissão apresenta uma mensagem inicial **«Ligue uma câmara para começar»**com dois botões:**Ligar Câmara**(verde, abre a caixa de diálogo de ligação de uma única câmara) e**Ligar Matriz** (azul, abre a caixa de diálogo de ligação da matriz). As próprias caixas de diálogo de ligação estão documentadas em [Ligar Câmaras](connecting.md); conceitos de matriz (sincronização, níveis, largura de banda) em [Matrizes de várias câmaras](arrays.md). Ao abrir um projeto guardado que contenha câmaras, a mensagem inicial mostra, em vez disso, um indicador de carregamento com «A reabrir N câmaras guardadas…&quot;, enquanto o Chloros restaura os fluxos da última sessão.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superior

| Controlo | O que faz |
| --- | --- |
| **Alternar modo de visualização**| Alterna entre a**visualização em grelha**(todos os mosaicos como células) e a**visualização em lista** (matrizes em largura total na parte superior, UMA câmara ativa abaixo). Dicas de ferramenta: «Alternar para a visualização em grelha» / «Alternar para a visualização em lista». |
| **Bloqueio da grelha**(cadeado) |**Bloqueado** por predefinição — mosaicos fixos no lugar. Desbloqueie para arrastar e reordenar os mosaicos para qualquer espaço (os espaços vazios são preservados). A grelha volta a bloquear-se automaticamente sempre que uma nova câmara se liga. Dicas de ferramenta: «Desbloquear grelha (ativar arrastar mosaicos)» / «Bloquear grelha (congelar mosaicos no lugar)». |
| Controlo deslizante **Zoom do feed** | Tamanho do mosaico, de 60 px até à largura total do contentor. As células mantêm uma proporção de 4:3. Abaixo de 200 px de largura de célula, as sobreposições de nome e fps ocultam-se para manter o mosaico limpo. |

### Mosaicos de feed

Cada câmara renderiza um mosaico composto em tempo real; uma câmara pode, adicionalmente, mostrar três mosaicos em escala de cinzentos **divididos por canal** (ver [Divisões de Canal](#display-overlays-drawn-over-the-live-feed)), e as matrizes apresentam um mosaico combinado. O mosaico ativo exibe um anel de seleção na cor da câmara (ou da matriz).

Ao passar o cursor sobre um mosaico, aparece um botão de fecho **X**:

* Fechar um mosaico **composto** enquanto as suas divisões de canal permanecem visíveis apenas oculta o mosaico composto.
* Fechar o **último mosaico visível de uma câmara independente** desliga essa câmara.
* **Os mosaicos divididos que fazem parte de um conjunto combinado nunca desligam** a câmara — apenas ficam ocultos.

Com a grelha desbloqueada, arraste qualquer mosaico para qualquer espaço; o layout é guardado com o projeto.

## Barra lateral — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

de câmaras A primeira página da barra lateral lista todas as câmaras e matrizes ligadas:

* **Ligar câmara**(verde) /**Ligar matriz** (azul, mostra «A detetar...» durante a verificação). Ambas ficam desativadas enquanto estiver aberta uma caixa de diálogo de ligação.
* **Capturar tudo**(vermelho) — captura todas as câmaras listadas com os tipos de exportação escolhidos nas**Configurações de captura**. Requer um projeto aberto. Totalmente documentado em [Configurações e modos de captura](capture.md).
* **Engrenagem das**Configurações de Captura**(ao lado de**Capturar tudo**) — abre o [painel**Configurações de Captura**](capture.md#the-capture-settings-pane). Desativado sem um projeto ou durante a captura.

### Linhas de câmara

Cada linha de câmara apresenta uma borda codificada por cores (a cor personalizada da câmara), uma etiqueta «CAM» — com uma letra **M**(mestre) ou**S** verde (escravo) para os membros da matriz — e o nome de exibição. O nome predefinido é `LATTICE-MODEL (serial)`; renomeie-o a partir do painel de definições de cada câmara. Botões da linha:

| Botão | Efeito |
| --- | --- |
| **Olho**| Alternar a visibilidade. As câmaras ocultas saem da grelha e são**excluídas da opção «Capturar tudo»**. |
| **Engrenagem** | Abrir o painel de definições de cada câmara (próxima secção). |
| **Pausa / Reproduzir**| Congela a pré-visualização ao vivo**apenas no lado da exibição** — a captura no backend continua a funcionar. As câmaras em pausa não podem capturar. |
| **X** | Desligar. A interface do utilizador atualiza-se imediatamente (de forma otimista); o processo de desligamento no backend pode demorar 10–30 s a concluir-se. |

### Linhas da matriz

Uma linha da matriz apresenta um emblema «ARRAY» na cor da matriz, o nome da matriz (renomeável nas definições da matriz) e um botão **DAQ · ligado/desligado**—**ligado** quando o sensor de luz ao nível da matriz está definido *ou* qualquer membro tem um; a sua dica de ferramenta indica exatamente qual o sensor que alimenta o quê. As câmaras dos membros são listadas com recuo logo abaixo, nas suas próprias linhas. Botões da linha da matriz: **olho**(oculta/mostra TODOS os membros em conjunto),**engrenagem**(painel de definições da matriz),**X**(desliga toda a matriz).

O estado do sensor de luz (DLS) utilizado nas linhas da matriz e no painel de definições da matriz tem quatro estados:**desligado**,**em espera**(ainda sem espectro),**ativo**(chegou um espectro nos últimos 3 s) e**desatualizado** — sem espectro recente nos últimos 3 s, mas a última leitura *continua a ser utilizada* (as leituras do DAQ nunca expiram no percurso de captura).

Pode arrastar câmaras individuais e grupos de matrizes inteiros uns sobre os outros na barra lateral para reordenar a lista; os membros da matriz não podem ser arrastados individualmente.

## Painel de definições por câmara

Abra com a **roda dentada** numa linha de câmara. O painel desliza sobre a lista de câmaras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Cabeçalho**: a**amostra de cor**da câmara (clique para abrir um seletor de cores nativo — define a cor da borda da barra lateral e do anel de seleção de mosaicos), o**nome**com um botão de**Renomear**em forma de lápis (guardar um nome vazio reverte para o padrão `MODEL (serial)`) e**×** para fechar.

### Histograma em tempo real

Na parte superior do painel encontra-se um histograma de luma em tempo real, calculado a partir da pré-visualização JPEG a ~8 Hz. A média é ponderada segundo o método Bayer — (R+2G+B)/4 — para corresponder à medição AE da própria câmara.

* **Linha tracejada Orange**= o alvo de AE.**Arraste-a horizontalmente para redefinir o alvo** — é enviado um comando ao soltar, e ao arrastar o modo do alvo de AE muda para Manual.
* **Green linha contínua** = a luminância média real (o que a AE está a fornecer neste momento).
* **Botão RGB** (canto superior direito): alterna entre histogramas sobrepostos por banda, coloridos de acordo com o filtro da câmara (por exemplo, no FRGN: cinzento NIR, verde, vermelho). Nas câmaras monocromáticas (M3M), o botão exibe «MONO» e está desativado — o modo monocromático mostra sempre o histograma de luminância de banda única.
* Os rótulos do eixo X seguem a profundidade de bits do sensor do formato de píxel atual: 0..255, 0..1023, 0..4095 ou 0..65535.

### Linhas

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

de informação da câmara | Linha | Comportamento |
| --- | --- |
| **Modelo** | Apenas leitura (por exemplo, `LATT-M3C-L87-FRGN`). |
| **Calibração radiométrica**| Emblema**«Ativo»**do Green com uma legenda que mostra o nível de calibração, o hash, a data de calibração e a lista de bandas, carregados a partir do pacote de calibração da câmara (ver [Calibração Radiométrica de Fábrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ocultado para câmaras RGB** — estas possuem uma calibração fotométrica do equilíbrio de brancos, e não radiação por banda. |
| **Relatório de Calibração**| Botão**Descarregar** — abre o certificado de calibração NIST por número de série da câmara, em formato PDF, no visualizador do seu sistema operativo. Se o certificado ainda não estiver armazenado em cache, a Chloros mostra uma sugestão em vez disso. |
| **Número de série** | Apenas de leitura. |
| **Firmware**| Mostra a versão atual e, em seguida, determina a versão disponível para este modelo (armazenada em cache por modelo — um conjunto de N câmaras verifica o servidor uma vez). Mensagens: «A verificar…» → Botão**«Atualizar para X»**→ «A atualizar…» → «Atualizado de A para B» / «Falha: …» / «Ignorado: …» /**«Atualizado»** a verde. Dica de ferramenta do botão de atualização: «Redefinição de fábrica + atualização de firmware + reprogramação do UserSet1. ~2–3 minutos; não desligue.» |
| **IP** | Apenas leitura. |
| **Temperatura** | Apenas leitura, atualizada a cada 3 s. Fica laranja a partir de ≥65 °C e vermelho com um ⚠ a partir de ≥75 °C. |
| Caixa de seleção **Alvo de calibração** | Ativa a deteção do alvo de refletância ArUco com uma tabela de validação NDVI por painel, abaixo da visualização em tempo real (vista em lista). Apenas para a sessão — abre-se sempre desativada. |
| Menu suspenso **Sensor de luz** | Associa um sensor de luz DAQ (DAQ-E/M/U, da lista do separador «Sensores de luz») a esta câmara para correção da iluminação por luz descendente (DLS) e exposição automática preditiva. «Nenhum» anula a ligação. Se não estiverem ligados sensores, o menu suspenso mostra «(nenhum sensor ligado — abrir o separador DAQ)». A associação é guardada com o projeto. |

### Exposição e Ganho

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Todas as entradas numéricas aqui utilizam controlos deslizantes do tipo «manter premido para acelerar»: tocar = ±1, manter premido &gt;1,5 s = ±10, manter premido &gt;3 s = ±100. O valor é enviado para a câmara quando soltar.

| Controlo | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Exposição (us)**| Valores mínimo/máximo em tempo real da câmara | Automático | Todos | Tempo de exposição em microssegundos, com um botão de alternância**Automático/Manual**. Automático = exposição automática contínua por parte da câmara. |
| **Ganho (dB)**| Valores mínimo/máximo em tempo real da câmara (por exemplo, até 48 dB) | Manual (desativado) | Todos | Ganho analógico/digital com o seu próprio botão de alternância**Automático/Manual**. |
| **Brilho alvo da AE**| 0–255 | 80, modo**Automático**| Todos (editável quando a AE ou o ganho automático estão ativados) | O brilho que a AE pretende atingir. No modo**Automático**(predefinição), um controlador interno baseado no histograma seleciona ele próprio o valor-alvo, mantendo a exposição entre 60–75 % do máximo do sensor. Digitar um valor ou arrastar a linha laranja do histograma alterna para**Manual**. |
| **Suavização do AE** | 0,5–40, incrementos de 0,1 | 8,0 | Todos | Amortecimento do AE. Dica: «Valor mais baixo = o AE reage mais rapidamente (pode apresentar picos em fps elevados). Valor mais alto = mais suave / mais lento.» Valores muito abaixo do padrão podem fazer com que a AE oscile e desestabilize a transmissão a taxas de fotogramas elevadas; 8,0 é o padrão estável. |
| **Região de Interesse do AE**| Caixa de seleção «Ativar» + botão**Aim**| Desativado | Todos | Quando ativado, o AE mede apenas a região a verde tracejada em vez de todo o fotograma. O botão**Aim** ativa a função «clicar para posicionar» na transmissão ao vivo: um clique centra uma região a 30 % do fotograma; clicar e arrastar traça um retângulo personalizado (mínimo 5 % × 5 %). O «Aim» desativa-se automaticamente após uma colocação. A região é remapeada para as coordenadas nativas da câmara, tendo em conta qualquer rotação/espelhamento que tenha definido, e é guardada com o projeto. |
| **Velocidade de Ajuste do AE** | 0,1–5, intervalo de 0,1 | 1,0 | Apenas para membros do Array | A rapidez com que o alvo do AE automático acompanha as alterações de luminosidade da cena; 1,0× verifica novamente a cada 2,5 s. |
| **Proteção de realces** | Rigorosa (1 %) / Normal (5 %) / Flexível (15 %) | Rigorosa | Câmaras que permitem definir esta configuração | A percentagem do enquadramento que pode ficar esbranquiçada antes de o AE escurecer a imagem. |

{% hint style="info" %}
**Requisitos de iluminação para câmaras multiespectrais Bayer (RGN / OCN / NGB):** a cena deve ter luz suficiente nos três canais, caso contrário a calibração não funciona corretamente — uma única exposição do sensor abrange os três espectros. Utilize um sensor de luz DAQ para medir a luz, ou opte pelo modo totalmente monocromático (M3M) para que cada banda tenha a sua própria exposição. Se uma captura violar esta condição, o Chloros deteta-a e avisa-o (notificação «unmix-clamp»).
{% endhint %}

### Formato de píxeis e

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

resolução**Os membros da matriz** apresentam as linhas de leitura apenas «Atual» (formato + LxA) e «Binning» com a nota «Definido nas definições da matriz» — um reinício do fluxo num membro quebraria a sincronização, pelo que estas são geridas no [painel de definições da matriz](#array-settings-pane).**As câmaras autónomas** têm:

| Controlo | Opções | O que faz |
| --- | --- | --- |
| **Formato de píxeis** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de píxeis do sensor (profundidade de bits). |
| **Resolução** | Completa / Metade / Um quarto | Relativa ao binning atual: Completa = 2048/N × 1536/N para binning N×N. |
| **Binning** | 1x1 (nenhum) / 2x2 / 4x4 | Agrupamento N×N por hardware — valores maiores reduzem a resolução, mas aumentam a relação sinal-ruído (SNR) e a taxa de fotogramas. A alteração reinicia a transmissão e redefine qualquer ROI para o novo campo de visão completo. |
| **Atual** | só de leitura | As dimensões WxH reais e o desvio (x, y) em vigor. |

### Pré-visualização ao vivo

Tudo nesta secção é **apenas do lado da visualização**— altera o que vê na transmissão em direto, enquanto as capturas guardadas permanecem lineares e inalteradas — com uma exceção: a**Vignette** é radiométrica e afeta também as exportações (indicado abaixo).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Controlo | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Resolução de renderização** | 360p (mais rápida) / 480p / 720p / 1080p / Resolução nativa do sensor (mais lenta) | 720p | Todos | A altura à qual o backend executa a cadeia de pré-visualização radiométrica. Um valor mais baixo aumenta a taxa de fotogramas sem alterar o campo de visão. |
| **Índice**| Caixa de seleção «Ativar» + engrenagem | Desativado | Apenas multiespectral Bayer,**não** membros de matriz combinada | Pré-visualização ao vivo do índice de vegetação. A engrenagem abre a [Calculadora de Índice](#index-calculator-pane) partilhada, pré-carregada com as bandas naturais do filtro da câmara (por exemplo, `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). A expressão personalizada, juntamente com a LUT (ligado/desligado, nível predefinido 3, mínimo predefinido 0,2, máximo predefinido 1) é calculada em cada fotograma de pré-visualização. Os membros da matriz combinada ocultam esta linha — a matriz possui um único índice partilhado. |
| **Equilíbrio de brancos** | Desligado / Uma vez / Contínuo + um botão de recaptura | Contínuo | Apenas RGB | Equilíbrio de brancos em tempo real. O botão de atualização recaptura o equilíbrio de brancos a partir do espectro DLS atual (desativado quando o modo está em Desligado). |
| **Gama** | Ligado / Desligado | Ligado | Apenas RGB | Exibe a gama (γ = 2,2 LUT) na pré-visualização em tempo real. As capturas guardadas mantêm-se lineares. |
| **Redução de ruído** | Caixa de seleção + intensidade 0–100 | Desligado / 50 | Todos (por câmara, mesmo dentro de matrizes) | Filtro bilateral na pré-visualização em tempo real. Valores mais elevados = detalhes mais suaves, mas menos nítidos. |
| **Nitidez** | Caixa de seleção + intensidade 0–100 | Desligado / 30 | Todos | Máscara de nitidez na pré-visualização em tempo real, aplicada em último lugar. Pode amplificar o ruído. Apenas na pré-visualização. |
| **Vinheta**| Caixa de seleção + intensidade 0–100 | Desativado / 0 | Todos | Remoção manual da vinheta residual (clareia os cantos), sobreposta à estimativa de Vinheta Inteligente da matriz.**Radiométrico — afeta a visualização em tempo real E as exportações**, ao contrário do Redutor de Ruído/Nitidez. |
| **Perfil de Cor** | Raw / Linear / Natural / Melhorado / Temperatura Personalizada | Natural | Apenas RGB | Ver abaixo. |
| **Temperatura de cor** | 2000–10000 K, intervalo de 100 | 5500 K | Apenas RGB, perfil de temperatura personalizada | Fixa o equilíbrio de brancos numa temperatura de cor correlacionada fixa (a entrada DLS é ignorada). O último valor em Kelvin selecionado é memorizado mesmo quando se alterna entre perfis. |
| **Saturação** | 0–200 (100 = neutro) | 100 | Apenas RGB | Saturação HSV na pré-visualização em tempo real. |
| **Contraste** | 0–200 (100 = neutro) | 100 | Apenas RGB | Contraste linear em torno do cinzento médio na pré-visualização em tempo real. |
| **Espelhar H / Espelhar V** | Caixas de seleção | Desativado | Todos | Inverte a pré-visualização horizontalmente / verticalmente. |
| **Rotação**| 0° / 90° / 180° / 270° | 0° | Todas | Rodar a pré-visualização. A orientação é aplicada no final da cadeia de pré-visualização do backend —**as capturas guardadas mantêm a orientação nativa da câmara** e as vistas compostas em matriz ignoram-na. |**Semântica do perfil de cor** (câmaras RGB):

* **Raw** — ignora completamente a cadeia de processamento.
* **Linear** — sinal escuro + campo plano + equilíbrio de brancos; sem matriz de cor, sem gama.
* **Natural** *(predefinição)* — linear mais a matriz de correção de cor medida e uma curva de tom adaptativa à cena.
* **Melhorada**— Natural mais vibração e contraste local CLAHE. O custo adicional aplica-se**apenas à pré-visualização em tempo real** — as capturas guardadas obtêm sempre o acabamento completo, independentemente do perfil.
* **Temperatura personalizada** — Natural com o equilíbrio de brancos fixado no valor em Kelvin que escolher.

{% hint style="warning" %}
Para os modos Natural, Aprimorado e Temperatura Personalizada, o painel apresenta uma nota sobre a tonalidade: os fotogramas são clareados de acordo com a sua própria cena, pelo que as imagens *de visualização* guardadas não são comparáveis entre si. **Exporte a radiação ou a refletância para medições.**
{% endhint %}

### Sobreposições de ecrã (desenhadas sobre a transmissão em direto)

Estas são exclusivas do front-end — são aplicadas sobre o vídeo, sem nunca afetar a transmissão ou as capturas.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sobreposição | Controlos | Padrão | O que faz |
| --- | --- | --- | --- |
| **Zebra** | Caixa de seleção + limiar 200–255 | Desativado / 250 | Listras diagonais magenta nos píxeis cortados. |
| **Miragem** | Caixa de seleção | Desativado | Marca no centro do fotograma. |
| **Grelha** | Desativado / 3 × 3 / 9 × 9 | Desativado | Grelha de composição. |
| **Histograma** | Caixa de seleção + largura 0,10–0,90 do enquadramento | Desativado / 0,25 | Uma faixa de histograma em tempo real. |
| **Pico de Foco** | Caixa de seleção + limiar 20–200 + amostra de cor | Desativado / 80 / `#ff5722` | Destaque de arestas Sobel para focagem. |
| **Divisões de canal** | &quot;Mostrar Divisões (Red / Green / NIR)&quot; / &quot;Botão «Ocultar divisões» | Oculto | Adiciona três mosaicos independentes em escala de cinzentos por canal ao lado da imagem composta (o rótulo do botão segue os canais de filtro da câmara). Cada mosaico dividido pode ser arrastado e partilha a cor da moldura da câmara. Não disponível em câmaras monocromáticas. Guardado com o projeto. |

### Medidor pontual

* Caixa de seleção **Clicar para amostrar**: clique na imagem em direto para amostrar um único píxel (uma cruz marca-o), ou clique e arraste uma região para obter uma média de píxeis.**Limpar**apaga a amostra e a cruz. É mutuamente exclusivo com o modo**Aim** do AE-ROI.
**Menu suspenso **Mostrar**:**Raw (profundidade de bits)**— valores digitais nativos com a profundidade de bits do sensor (por exemplo, 12 bits → 0..4095) — ou**Ecrã (8 bits)** (predefinição). Quando um índice em tempo real está ativo, a opção «Exibição» mostra, em vez disso, o valor do índice calculado (por exemplo, NDVI).
* O painel de leitura apresenta as coordenadas dos píxeis, o tamanho do fotograma, o formato dos píxeis, a profundidade de bits e uma tabela de canais (Canal / Valor / %) com rótulos de banda e comprimentos de onda; os pares verdes de Bayer são calculados como média; as amostras da região apresentam «Média de N píxeis».

O estado do medidor pontual é válido apenas para a sessão.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposição Automática Preditiva (controlada por DLS)

Esta secção só aparece quando **pelo menos um sensor de luz DAQ estiver ligado** — o solucionador necessita de um espectro descendente em tempo real para o controlar.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Controlo | Intervalo | Padrão | O que faz |
| --- | --- | --- | --- |
| **Ativar** | Caixa de seleção | Ativado (câmaras autónomas) | Um solucionador de forma fechada utiliza o espectro do DLS, juntamente com os escalares do pacote de calibração da câmara, para ajustar a banda mais brilhante perto da saturação, mantendo a banda mais escura acima do limiar de SNR — uma única exposição por resolução, sem ciclo de estabilização. Concebido para timelapse alimentado a energia solar, em que cada fotograma deve ser corretamente exposto. O backend recorre silenciosamente ao AE reativo sempre que a leitura DLS estiver desatualizada/ausente ou o pacote de calibração não estiver carregado. |
| **Suavização (α)** | 0,05–1,0, intervalo de 0,05 | 0,3 | Suavização de soluções preditivas sucessivas (valor mais baixo = maior suavização). |
| **Refletância da cena**| Botão**Recalibrar ρ** | — | Reestima o fator de refletância da cena utilizado pelo solucionador. |

{% hint style="info" %}
**A ligação em matriz desativa a AE preditiva por predefinição** — para matrizes, a AE inteligente do Chloros, juntamente com a exposição automática do lado da câmara, controla a exposição (com proteção contra saturação), e a estimativa única da refletância da cena da AE preditiva não é segura em cenas mistas. Pode-ativá-la por câmara aqui, caso pretenda especificamente uma exposição radiométrica controlada pelo DLS.
{% endhint %}

**Limite de exposição controlado por DAQ e AE fixada na luz incidente.**Independentemente da caixa de seleção acima, quando um sensor de luz DAQ é atribuído a uma câmara RGB, o Chloros calcula — a partir da irradiação descendente absoluta medida — a exposição máxima × ganho na qual uma superfície com 100 % de refletância permanece abaixo do clipping e aplica-a como um**limite máximo**na exposição automática. Enquanto o limite máximo estiver ativo, a câmara fica**fixada na luz incidente**: funciona em malha aberta com a exposição medida pela luz incidente e o ganho a 0 dB — a exposição segue a luz medida, não o conteúdo da cena. Como o limite máximo só pode reduzir a exposição, não pode, por si só, provocar clipping. O limite máximo desativa-se automaticamente — e a exposição automática (AE) normal da cena é retomada — sempre que a leitura do DAQ estiver em falta, estiver desatualizada (&gt;30 s), ou escura, ou se ≥15 % do fotograma sofrer clipping na exposição fixada (o que significa que o sensor e a câmara estão a detetar iluminação diferente). Não existe um botão na GUI; este é o comportamento padrão sempre que uma câmara RGB tem um DAQ associado.

### Os membros da matriz de aquisição e disparo

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

apresentam adicionalmente as linhas de **Função**(Mestre a azul / Escravo a verde),**Linha de sincronização**e**Parceiros**, que são de leitura apenas.

| Controlo | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **Modo de Acionamento** | Desligado / Ligado | Ligado | Desativado para membros da matriz (a matriz gere o acionamento). |
| **Fonte de disparo** | Software / Linha 0 (M8) / Linha 1 / Linha 2 | Linha 0 | Oculta enquanto o Modo de disparo estiver Desligado; desativada para os membros da matriz. A Linha 0 é a entrada de disparo externa opto-isolada M8. |
| **Taxa de fotogramas**| Automático / Manual + valor | Automático |**Automático**: o limite da taxa de fotogramas da câmara está desativado — a exposição determina os fps, e a caixa mostra a taxa real em tempo real.**Manual**: limita os fps com um controlo deslizante (de 1 até ao máximo limitado pela largura de banda), com base na taxa real atual. Os membros da matriz veem um valor de «N fps (em tempo real)» apenas para leitura com a indicação «Definido nas definições da matriz». |

### Rede / Transporte

| Linha | Comportamento |
| --- | --- |
| **Tamanho do pacote**| 1500 (Padrão) / 9000 (Jumbo) — predefinição**Jumbo**. |
| **Largura de banda** | Limite de largura de banda do link, de leitura apenas, em MB/s. O backend reequilibra este valor entre todas as câmaras ligadas a cada ligação/desligamento. |
| **Gestão do buffer** | Modo de gestão do buffer, de leitura apenas. |

### Captura

O painel termina com um botão **«Abrir definições de captura…»** que leva ao [painel de definições de captura](capture.md#the-capture-settings-pane) (desativado até que um projeto esteja aberto — «Crie ou abra um projeto para guardar capturas»). Se a câmara estiver oculta ou em pausa, uma dica lembra-o de a revelar/retomar antes de capturar.

## Painel de definições da matriz

Abra com a **roda dentada**numa linha de MATRIZ. Cabeçalho: nome da matriz com um lápis para renomear e**×** para fechar. As secções abaixo marcadas como *apenas combinadas* aparecem apenas para matrizes ligadas no modo de visualização combinado.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sincronização

Linhas **Mestre**,**Escravos**e**Linha de Sincronização** apenas de leitura.

### Sensor de luz ambiente

Exibido tanto para matrizes combinadas como separadas:

* Caixa de seleção **Alvo de calibração** — «Detetar o alvo ArUco MAPIR e validar a LUT de refletância do painel NDVI»; controla a sobreposição do alvo e a tabela de validação do mosaico combinado.
* Menu suspenso **Sensor de luz**— associa um DAQ a todo o conjunto. A seleção é aplicada imediatamente, propaga-se para o menu suspenso**Sensor de luz** de cada câmara do conjunto (ainda é possível substituir a configuração por câmara) e começa a enviar espectros para o conjunto.
* Linha **Estado** em tempo real: Desligado · «À espera do primeiro espectro…» · «Ativo — todas as câmaras da matriz têm a iluminação corrigida» · «Sem novos espectros nos últimos 3 s — ainda a utilizar a última leitura (sem tempo limite para dados desatualizados)…».
* Nota no painel: «Correção radiométrica em toda a matriz. As definições por câmara substituem esta.»

### Captura — definições uniformes do sensor *(apenas combinadas)*

Estas definições aplicam-se uniformemente a todos os membros (alterações por membro quebrariam a sincronização). As edições são preparadas e aplicadas em conjunto.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Controlo | Opções / intervalo | O que faz |
| --- | --- | --- |
| **Formato de pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de sensor uniforme para todos os membros. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning de hardware — mantém o campo de visão completo enquanto aumenta a relação sinal-ruído (SNR) e a taxa de fotogramas. A alteração redefine os campos da ROI para o novo campo de visão completo. |
| **Predefinição de**resolução** | Completa / Metade / Um quarto | Relativa ao binning; preenche os campos de ROI com um recorte centrado. |
| **Recorte de ROI (px)**| Campos numéricos L / A / X / Y | Recorte do sensor. A largura e a altura ajustam-se a múltiplos de 16 (mínimo 64); os deslocamentos ajustam-se a múltiplos de 4. Uma indicação «LxA máx.» mostra o limite máximo e**Reiniciar** volta ao campo de visão total. Durante a edição, é desenhada uma caixa laranja de pré-visualização do recorte em tempo real no mosaico da matriz (incluindo um esquema do sensor completo quando se amplia o recorte para o exterior). |
| **Taxa de disparo**| Alternância Automático / Manual + fps 0,5–10, incrementos de 0,5 |**Automático**(predefinição): o backend calcula a taxa de disparo a partir da resolução e da largura de banda — a entrada fica desativada e apresenta o valor calculado.**Manual**: fixa o seu valor ao clicar em Aplicar. |

Nota no painel: «Alterações de formato/resolução reiniciam todas as câmaras brevemente. A taxa de disparo aplica-se em tempo real.» Os botões **Aplicar / Cancelar** encontram-se na parte inferior do painel.

### Alinhamento (co-registo) *(apenas combinado)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Caixa **Residual**: «RMS x,xx px» — verde se for inferior a 1 px, âmbar se for inferior a 3 px, vermelho nos restantes casos ou se alguma câmara falhar; «sem perfil» antes da primeira resolução.
* Linha de resumo: «✓ Todas as câmaras alinhadas (N)» / «⚠ p/N câmaras alinhadas —  <serial (filter)="">falha&quot; / &quot;Recorte ativo — Recalibrar para alinhar (utiliza o sensor completo)&quot; / &quot;À espera que a exposição estabilize…&quot;.
* Tabela por câmara: câmara (últimos 4 dígitos do número de série + filtro), erro de reprojeção em px com contagem de correspondências (&quot;ref&quot; para a câmara principal), e a pontuação de correlação cruzada normalizada da sobreposição em relação ao limite mínimo de aprovação de 0,35.
* Botão **Recalibrar alinhamento** (exibe «Calibrar alinhamento» antes do primeiro perfil) — executa novamente o co-registo em fotogramas novos.
* Caixa de seleção **«Exposição automática das câmaras para alinhamento»** (marcada por predefinição) — clareia temporariamente as câmaras escuras ou sem contraste (primeiro a exposição, depois o ganho) para que tenham textura para corresponder, e depois restaura a exposição automática.

A pré-visualização combinada alinha-se automaticamente ao abrir; recalibre se o foco ou a profundidade da cena tiverem mudado. O alinhamento é **projetado para ser válido apenas durante a sessão** — nunca é guardado num perfil, porque depende da distância da cena naquele momento. As capturas ainda podem ser exportadas com registo de píxeis (ver [Exportações alinhadas](capture.md#per-array-controls)).

### Vinheta Inteligente

* Caixa de seleção ***Ativar correção***— aplica a estimativa de vinheta por câmara à cadeia radiométrica (em tempo real**e** nas exportações).
* **Calibrar a partir da vista atual**— aponte primeiro o conjunto de câmaras para um alvo uniforme (ecrã plano, parede ou céu); cada câmara é nivelada individualmente e o estado indica «n/N câmaras · −x,x %» de ganho de nivelamento.**Limpar** remove a estimativa.
* Ajuste com precisão por câmara com o controlo deslizante **Vignette** por câmara na [Pré-visualização ao vivo](#live-preview).

### Pré-visualização ao vivo *(apenas combinada)** **Índice**: ative a caixa de seleção + engrenagem — abre a [Calculadora de Índice](#index-calculator-pane) partilhada com faixas traçadas a partir de**todas** as câmaras participantes. Uma linha de pré-visualização da expressão, logo abaixo, mostra a expressão atual («Nenhuma expressão definida — abra a calculadora para criar uma»), atualizada a cada segundo.
* **Menu suspenso**Resolução de renderização**(as mesmas predefinições que por câmara, predefinição 720p): a altura do fluxo de visualização em direto**e** o tamanho da exportação composta guardada. Nota no painel: «Pré-visualização + tamanho da composição guardada. As imagens por câmara são sempre exportadas em resolução total.»

### Camadas de exibição *(apenas combinadas)** Caixa de seleção **Ativar** (desativada por predefinição — a câmara principal é apresentada diretamente; ativada = composição em camadas).
* Menus suspensos **Primeiro plano**/**Fundo**: cada câmara membro (por nome) ou**Índice**. Quando o Primeiro plano é Índice, os píxeis fora dos valores Mín./Máx. da LUT mostram a camada de Fundo.

### Visualização dividida *(apenas combinada)*

**«Mostrar câmaras participantes»**— um botão**Dividir / Ocultar câmaras participantes** que adiciona a transmissão em direto de cada participante como mosaicos separados na grelha, ao lado da composição. Os mosaicos leem o buffer de fotogramas existente da matriz (sem ligação adicional de câmara). Apenas na vista em grelha; guardado por matriz com o projeto.

### Funcionalidades

Um painel de leitura apenas atualizado a cada 5 s:

* **Rótulo do nível**: «Captura simultânea» (verde) · «Captura simultânea (emissão escalonada FTD)» (verde) · «Captura escalonada (desvio de 100 ms)» (âmbar) · «Configuração demasiado grande» (vermelho).
* **Estado do quadro**: «x,xx % incompleto» — verde abaixo de 1 %, âmbar abaixo de 5 %, vermelho a partir de 5 % ou mais.
* **Linha de ligação**: «NIC {mbps} Mbps - sustentada {MB/s} MB/s».

Este é o orçamento de largura de banda em tempo real do conjunto. Para obter informações sobre os fps subjacentes e o modelo de rede — e o que alterar quando o nível fica âmbar ou vermelho — consulte [Conjuntos de várias câmaras](arrays.md) e a [Referência CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Painel «Calculadora de Índice»

A terceira página da barra lateral, partilhada pela engrenagem «Índice» por câmara e pela engrenagem «Índice» da matriz combinada (uma de cada vez — o título indica «Calculadora de Índice — <camera name="">» ou «Calculadora de Índice —<array name="">

“). Recebe a lista de bandas (as bandas naturais do filtro da câmara ou todas as bandas dos membros do conjunto), a expressão atual e a configuração da LUT (ligada/desligada, nível — predefinição 3, mínimo — predefinição 0,2, máximo — predefinição 1), além de um histograma de índice em tempo real. **Aplicar** confirma a expressão; as alterações na LUT são aplicadas em tempo real à pré-visualização.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Definições por câmara vs. geridas pela matriz

Referência rápida sobre o que se encontra onde quando uma câmara é membro de uma matriz:

| Gerido pela matriz (apenas leitura no painel da câmara) | Continua por câmara dentro de uma matriz |
| --- | --- |
| Formato de píxeis, resolução, binning | Exposição automática (exposição, ganho, alvo, suavização, ROI) |
| Modo/fonte de disparo, taxa de fotogramas | Redução de ruído, nitidez, vinheta |
| | Orientação (espelhamento/rotação), sobreposições de ecrã, medidor pontual |
| | Índice (matrizes com ecrãs separados), ligação do sensor de luz |

Outros comportamentos transversais:

* **Exibição combinada vs. separada** é escolhida na ligação da matriz: combinada = um mosaico composto alinhado (as imagens dos membros são transmitidas apenas através da Vista Dividida); separada = cada membro renderiza o seu próprio mosaico sincronizado. Uma câmara nunca mostra simultaneamente uma transmissão independente e um mosaico da matriz.
* **Reconexão automática**: ao abrir um projeto guardado, as câmaras e matrizes são restauradas e todas as definições guardadas são reaplicadas ao backend antes de os fluxos serem retomados.
* ****Controlo de captura**: as câmaras ocultas ou em pausa são excluídas da opção «Capturar Tudo»; uma matriz só fica totalmente bloqueada quando TODOS os membros estiverem ocultos/em pausa. Consulte [Definições e modos de captura](capture.md).

## Como as definições são mantidas

O estado do separador «Câmara» é guardado **com o projeto**, não no navegador:

* Cada alteração reativa cria um instantâneo das câmaras e matrizes no ficheiro `cameras.json` do projeto (com um tempo de espera de 500 ms). Isto inclui nomes e cores das câmaras, definições de exposição/ganho/AE, formato de píxeis/resolução/binning, frequência de disparo, definições de pré-visualização (resolução de renderização, redução de ruído, nitidez, vinheta, perfil de cor, saturação/contraste), orientação, sobreposições, divisões de canais, configuração do índice, definições de AE preditivo, ROI de AE, nomes das matrizes, modo de exibição, definições de captura das matrizes (incluindo a posição de recorte do ROI) e o bloco de grelha (zoom de alimentação, modo de visualização, bloqueio da grelha, ordem manual dos mosaicos, câmaras ocultas, mosaicos fechados, câmara ativa).
* As ligações dos sensores de luz são guardadas no ficheiro `sensors.json` do projeto.
* Ao reabrir o projeto, o hardware é reconectado e todas as configurações são reaplicadas.
* **Sem projeto aberto = apenas sessão**: sem projeto, nada sobrevive ao fecho do Chloros.
* Apenas sessão, independentemente do projeto: estado de pausa, amostras do medidor pontual, a caixa de seleção «Alvo de calibração» por câmara (que se abre sempre desativada) e o perfil de alinhamento da matriz (recalculado por sessão, por definição).
* Uma exceção: as seleções de exportação das **Definições de Captura** e o modo de captura persistem por projeto no armazenamento local da aplicação, em vez de no `cameras.json` — consulte [Definições e Modos de Captura](capture.md).</array></camera></serial>
