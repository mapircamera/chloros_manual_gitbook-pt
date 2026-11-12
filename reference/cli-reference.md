# Referência da Chloros CLI

**Versão:**

1.3.0**Gerada:**29/07/2026 19:19 ·**Revisada:** 06/11/2026**Público-alvo:** Otimizado para consumo por LLM; legível por humanos.**Âmbito:** Todos os subcomandos do `chloros-cli` destinados ao utilizador, com opções e exemplos que podem ser copiados e colados.

Este documento é a referência completa para a ferramenta de linha de comandos `chloros-cli` fornecida com o MAPIR Chloros. É intencionalmente exaustivo para que um LLM (ou um ser humano) possa compor qualquer fluxo de trabalho suportado a partir das listas abaixo sem inspecionar o código-fonte.

Se precisar apenas dos pontos principais, avance para:
- [Guia de início rápido de cinco minutos](#five-minute-quickstart)
- [Fluxo de trabalho de primeira ligação da câmara LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxo de trabalho de primeira ligação do sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess)

> **A controlar o Chloros a partir de um assistente de IA?** O Claude Desktop, o Claude Code, o Cursor e outros clientes MCP podem executar o processamento, as câmaras e matrizes LATTICE, bem como os sensores de luz DAQ através do `chloros-mcp`, o servidor MCP que é instalado com o Python SDK (e com o .deb para Linux, no `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. A configuração, as opções e as ferramentas encontram-se na secção [Servidor MCP (Assistentes de IA)](sdk-reference.md#mcp-server-ai-assistants). Um assistente com um shell também pode executar diretamente todos os comandos desta página.

> **Alterado na versão 1.3.0** (o que um script ou fluxo de trabalho existente da versão 1.2.0 deteta).
>
> **Alterações que um script da versão 1.2.0 poderá ter de tratar:**
>
> - **Aplica-se o `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` e `align-stream` foram removidos na versão 1.2.0 e passaram a funcionar com as definições próprias da câmara. Agora define o formato, a exposição, o ganho e a taxa de fotogramas da predefinição: o `--preset high_speed` captura em 8 bits a 5 ms, 6 dB, 60 fps. O `lattice cam-connect --preset` também mantém o formato da predefinição (a versão 1.2.0 configurava a câmara para 12 bits após a aplicação da predefinição). Consulte [Opções comuns](#common-options-shared-by-most-subcommands).
> - **Um formato de píxeis Bayer numa câmara M3M (mono) corresponde à sua profundidade.** As predefinições referem-se a formatos Bayer, que um sensor mono não consegue captar: na versão 1.2.0, a câmara rejeitou o formato e manteve o que já tinha, pelo que o `--preset high_speed` não era de 8 bits nesse caso. Agora é o formato Mono com a mesma profundidade: o `high_speed` e o `--pixel-format BayerRG8` resultam no `Mono8`, `high_quality`, `triggered` e `BayerRG12` resultam em `Mono12`. Numa matriz mista de M3C + M3M, cada câmara adota o formato partilhado da sua própria família, e o projeto `network-analysis` / `analyze-array --pixel-format` projetam cada câmara nessa profundidade. O mesmo se aplica no sentido inverso: uma M3C solicitada num formato Mono adota o formato Bayer dessa profundidade, pelo que uma matriz cujo mestre seja uma M3M deixa de reabrir as suas câmaras M3C no formato Mono. Ver [Câmaras Mono (M3M) de banda única](#mono-m3m-single-band-cameras).
> - **As exportações do `lattice array-capture` são alinhadas por predefinição, tal como as da aplicação.** Todos os ficheiros `debayered`, `display`, `radiance`, `reflectance` e ficheiros de índice de cada membro são distorcidos para se adequarem ao enquadramento da câmara principal e recortados de acordo com a sobreposição dos membros (menor do que o sensor devido aos deslocamentos das câmaras, `XMP-mapir:AlignmentJSON` `"applied": true`), pelo que uma captura CLI corresponde aos mesmos ficheiros que a captura da mesma cena pela aplicação Chloros; o `raw` mantém-se no formato nativo. Anteriormente, uma captura sem `--aligned` guardava todos os ficheiros no tamanho nativo, sem correção de distorção. O `--no-aligned` mantém essa funcionalidade. O mesmo comportamento por predefinição aplica-se ao `--smart`, `--continuous` e `--interval`, bem como ao SDK e ao `chloros-mcp`. Consulte [modos `array-capture`](#array-capture-modes).
> - **O `lattice array-capture --compression` tem como predefinição o `none`.** Os ficheiros são gravados sem compressão (cerca de 6,3 MB por fotograma em resolução total) para obter a taxa sustentada mais rápida; a versão 1.2.0 gravava em `deflate` (cerca de 4,1 MB). Utilize o `--compression deflate` para obter ficheiros mais pequenos. Ambos são sem perdas.
> - **Os formatos PNG e JPEG aceitam apenas os produtos `display` / `preview` e os ficheiros de índice.** Uma captura com `--format png` ou `jpg` com um nível que um ficheiro de 8 bits não consegue suportar é recusada antes de qualquer captura ser efetuada (código HTTP 400, saída `1`), assim como o `--processing all` predefinido com o `--format png`/`jpg`. Consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default).
> - **O `lattice dls` necessita do `--mac`.**Já não existe um MAC predefinido (era o endereço de uma unidade); o `daq pool-discover --only ble` encontra o de um sensor.**O `lattice dls --camera SERIAL`** é novo.
> - **Mais falhas saem do `1`.** `process` quando as exportações de uma execução não puderam ser verificadas ou se perdeu uma imagem, embora os outros produtos tenham sido gravados; um comando de captura com qualquer ficheiro que não tenha sido possível gravar (os outros ficheiros continuam a ser guardados); `lattice continuous` e `multi-stream` quando a unidade está cheia; `array-burst` / `array-record` quando o gravador reporta um erro; `array-build-video` quando um produto solicitado foi ignorado ou não foi criado conforme solicitado; `project burst` / `stream` quando uma captura falha a meio; `project sensor log` quando a gravação não contém qualquer leitura; `lattice exposure --set`, `gain --set` e `configure --set` com um valor que a câmara recusa. `lattice network-analysis --json` termina tal como o resumo (`1` para `needs_force_slip`). Ao contrário: o `daq pool-disconnect` de um sensor que já não se encontra no conjunto encerra o `0`. Consulte [Códigos de saída](#exit-codes).
> - **Um `lattice array-connect` lento nunca falha com a matriz ligada.** Espera até 600 s (180 s na versão 1.2.0; `project connect` 90 s), apresentando a etapa do backend à medida que avança, e uma ligação que não esteja concluída até esse momento é desfeita pelo backend — cada câmara é reposto no estado em que foi encontrada — pelo que um `Error:` (saída `1`) significa que não há nenhum conjunto ligado. Na versão 1.2.0, uma ligação efetuada imediatamente após as câmaras serem ligadas podia prolongar-se para além do tempo de espera e ligar o conjunto de câmaras após o comando ter falhado. Agora, Ctrl+C cancela a ligação. Consulte [Ligações Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` mantém o sensor completo.** Sem `--width` / `--height` / `--binning`, todas as câmaras são configuradas para o sensor completo e a taxa de fotogramas desce para o que este computador e a rede permitem; a saída de ligação apresenta opções mais rápidas (agrupamento ou uma ROI mais pequena) com as respetivas taxas projetadas. A versão 1.2.0 reduziu ou agrupou os fotogramas para manter a taxa elevada. Consulte [Opções do `array-connect`](#array-connect-options).
> - **Uma rede lenta já não impede o funcionamento de um conjunto de câmaras.** Cada câmara é ajustada à sua quota-parte da largura de banda disponível e a taxa de fotogramas diminui em conformidade. Nos casos em que as câmaras não podem ser ajustadas a um ritmo tão lento, o conjunto de câmaras funciona em emissão em série (`slip-emit-and-capture`): todos os fotogramas chegam, mas as câmaras não são expostas em simultâneo. O `--force-tier sim-capture-sim-emit` exige exposição simultânea. Consulte [FPS do conjunto e modelo de disparo em rajada](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` definem qual o «connect» a aplicar**: sem o `--pixel-format`, os formatos definidos pelo «connect» (Mono12 / BayerRG12, anteriormente BayerRG8) e a decisão relativa ao [transporte de 12 bits compactado](#packed-12-bit-transport---packed). O mesmo comando pode, portanto, indicar uma taxa de fotogramas diferente da da versão 1.2.0.
> - **Os caminhos relativos referem-se ao local onde o comando é executado.** `-o` e os outros caminhos que o backend abre (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), o `output` de um passo de receita, e o `output/` predefinido de `lattice index --live` são resolvidos em relação à pasta a partir da qual executa o `chloros-cli`. Na versão 1.2.0, o backend resolvia-os na sua própria pasta de instalação, o que, numa compilação instalada, resultava num erro com a mensagem «Acesso negado». Consulte [Convenções](#conventions).
> - **O `lattice index` é executado no backend em todas as formas**, tal como os outros comandos `lattice`, pelo que deve iniciar primeiro o backend. O índice offline (`--input`) é calculado pelo backend, que lê o ficheiro TIFF e grava os resultados ao lado deste ou em `-o`, e os comandos `--list-presets` / `--list-gradients` imprimem as tabelas do backend. No Windows, estes comandos funcionam agora: anteriormente, a CLI do Windows respondia a todas as formas de `lattice index`, incluindo `--live`, com `Error: lattice_sdk not available`. No Linux, o índice offline costumava ser executado dentro da CLI sem backend e sem login; agora necessita de ambos, tal como qualquer outro comando da CLI. Consulte [Índice / Cálculos de Vegetação](#index--vegetation-maths).
> - **Novos ficheiros junto aos produtos.** Cada produto de radianância, refletância e índice em que a separação de bandas limitou um pixel recebe uma máscara numa pasta `clamp_masks/` ao lado da pasta do produto e a captura imprime uma linha `clamp mask:`. Consulte [Máscara de limitação da separação de bandas](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientação da câmara.** Uma captura com uma única câmara guarda os ficheiros `raw` e `debayered` tal como o sensor os leu e regista a inversão/rotação da câmara no XMP; o ficheiro `process` aplica essa informação aos produtos que cria. Os ficheiros membros da matriz permanecem no formato nativo, a menos que se passe o novo `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Consulte [Orientação da câmara](#como-é-uma-pasta-de-capturas).
> - **Um nome de nível de exportação que não seja «um» é recusado.** `lattice capture --processing` e todas as entradas da lista `--levels` aceitam `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou um alias como `preview`; `--processing` e também `all`). Qualquer outro nome é recusado antes de se aceder à câmara (400 HTTP `UNKNOWN_LEVEL`, sair `1`); 1.2.0 guardou `raw` para isso e saiu `0`.
> - **O `daq pool-set-cap` necessita do `--cap-id`.** Sem ele, o comando sai do `2`; a versão 1.2.0 removeu a correção da tampa, deixando o sensor à vista. O `--cap-id none` solicita o sensor sem filtros.
> - **O `lattice resolution --binning-mode` foi removido.** O binning é sempre «Média»; o sinalizador continua a ser aceite, exibe um aviso e não realiza nenhuma ação. Consulte [Ajuste do sensor](#sensor-tuning).
> - **O `process --debayer texture-aware` é recusado para capturas LATTICE.** Para uma pasta que contenha capturas LATTICE, o `process` sai do `1` após a importação, indicando quantas existem, antes de processar qualquer coisa. As capturas LATTICE recebem sempre o demosaico padrão, pelo que a versão 1.2.0 aceitou a opção e ignorou-a; a aplicação Chloros também não a disponibiliza para estas capturas. Consulte [`chloros-cli process`](#chloros-cli-process).
> - **As câmaras voltam ao estado em que foram encontradas — a menos que se utilize o comando `--keep-settings`.** `lattice array-disconnect`, todos os comandos transitórios (`multi-capture` sem matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) e o `lattice continuous` repõem cada câmara tal como a encontraram: a sua ligação do disparador e da linha de sincronização, o seu PTP e todas as definições da câmara alteradas durante a sessão (exposição e ganho, o alvo AE, limites e amortecimento, equilíbrio de brancos, nível de preto, formato de píxeis, binning e ROI, inversões, taxa de fotogramas). O `--keep-settings`, por sua vez, guarda asna própria memória da câmara, para que esta inicie com essas configurações; o `--clear-settings` repõe essa memória para as predefinições de fábrica. Em qualquer dos casos, as câmaras ficam a funcionar de forma autónoma. Um projeto configura as suas câmaras a partir das suas próprias predefinições guardadas quando as liga. Consulte [Memória da câmara](#camera-memory----keep-settings----clear-settings).
> - **Todos os modos guardam um projeto da mesma forma.** Qualquer alteração numa câmara, num conjunto ou num sensor de luz ligado a um projeto — a partir da aplicação, da CLI, do SDK ou do servidor MCP — é guardada no ficheiro `cameras.json` / `sensors.json` desse projeto, e ao abrir o projeto em qualquer modo, os dispositivos são configurados a partir dele. Anteriormente, apenas a aplicação guardava o projeto, e o seu próximo salvamento substituía tudo o resto. Consulte [Um registo de projeto, todos os modos](#one-project-record-every-mode).
> - **Os ficheiros criados a partir de várias câmaras identificam cada uma delas.** Uma pilha multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), um índice em tempo real criado a partir das bandas de várias câmaras e uma imagem `composite/` de uma captura em matriz contêm o número de série, o modelo e o pacote de calibração de cada câmara de origem sob a forma de listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), e as listas EXIF `SerialNumber` apresentam os números de série separados por vírgulas. Anteriormente, não incluíam nenhuma câmara ou apenas o número de série da primeira câmara. Consulte [Ficheiros criados a partir de várias câmaras](#files-made-from-several-cameras).
> - **Apenas um ficheiro `raw` contém as etiquetas de calibração da câmara.** O conjunto compatível com MicaSense (`BlackLevel`, `RadiometricCalibration`, o `Band{R,G,B}RadiometricCalibration` por canal, os conjuntos `MicaSenseVignetting*`) e a carga útil `CalibrationJSON` descrevem os valores brutos do sensor, pelo que se encontram apenas no ficheiro de captura `raw`. Todos os outros ficheiros — os ficheiros de captura `debayered`, `display`, `radiance`, `reflectance` e ficheiros de índice, os quadros `lattice continuous` e `multi-stream`, e cada produto `process` — contém a identidade da câmara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, numa captura, o registo próprio do fotograma (`XMP-mapir:FrameJSON`: exposição, ganho, temperatura, formato de píxeis, hora da captura). Na versão 1.2.0, incluíam também o conjunto de dados brutos, que uma ferramenta que o aplicasse (Pix4D, Metashape) aplicava a píxeis que não são valores brutos. O `process` identifica a calibração desse ficheiro através do seu número de série e do `CalBundleSha256`.
>
> **Outras alterações:**
>
> - **A taxa de fotogramas automática de um conjunto de câmaras ajusta-se ao que o conjunto fornece.** Uma taxa que perca mais de 1 % dos seus disparos é reduzida, avaliada ao longo de várias observações prolongadas em vez de uma única, e a taxa a partir da qual foi reduzida é tentada novamente de vez em quando. Uma taxa definida pelo utilizador permanece inalterada.
> - **A exposição automática está mais estável.** Mantém-se constante em condições de luz estável, estabiliza-se em segundos após uma alteração, ignora pequenos pontos de luz intensa e reativa uma câmara que deixa de responder sob sol intenso. Consulte [Como funciona a exposição automática](#how-auto-exposure-behaves).
> - **O `process` conta apenas as suas imagens e gravações.** Ao executar novamente numa pasta que contenha o ficheiro `camera_cal/` de um projeto, esses ficheiros de calibração deixam de ser contados («Encontradas 96 imagens» num conjunto de 90 imagens), e um projeto que já os tenha incluído descarta-os.
> - **O `lattice multi-test` parte do estado de ligação das câmaras.** Ativa o PTP das câmaras para a execução, aguarda que os seus relógios se sincronizem e estabilizem antes do primeiro disparo — cada câmara fica sincronizada durante 120 s, cerca de 2 minutos após a ligação (`--ptp-settle SECONDS` define-o), uma vez que um relógio recém-sincronizado apresenta um desvio de até ~200 us durante cerca de 2 minutos — e repõe o PTP de cada câmara tal como o encontrou. Não se aguarda novamente por uma câmara que ainda esteja sincronizada de um teste anterior. Anteriormente, desativava o PTP numa rede com um único comutador onde nenhum DAQ-E respondia e interpretava o relógio de funcionamento livre de cada câmara como um relógio PTP que não se sincronizaria (`PTP is not locked`, em cada execução). Ver [Multicâmaras (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **Uma gravação `.daq` indica se a tampa foi escolhida ou assumida.** Os DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e o Chloros não consegue detetar se está instalada uma, pelo que um sensor ligado sem uma tampa especificada recebe, por predefinição, o código `sunshine_cosine`, uma *presunção*. Todas as gravações do Chloros (esquema 1.30) armazenam agora o `cap_id_source` juntamente com o `cap_id`: `operator` quando atribuiu um nome à tampa (incluindo `none`), e `auto_default` quando foi a predefinição. Ao processar uma gravação cuja tampa foi assumida, com a tampa DAQ do projeto definida como «Auto», apresenta um `[daq-dls] WARNING` a indicar o nome do ficheiro: um DAQ-U sem correção de luz solar apresenta um valor cerca de 30 vezes superior. **Indique o nome do limite que definiu** ao ligar, ou defina o limite de DAQ do projeto, e o aviso desaparece.
> - **Acabaram-se as janelas de consola pip no Windows.** Num PC com o Python instalado, o backend e cada um dos seus processadores de imagem executavam o `pip` uma vez no arranque, e cada um abria uma janela de consola. Já não o fazem.
> - **As gravações em tempo real do `.daq` sobrevivem a um corte de energia.**Uma gravação é gravada com um registo e uma sincronização completa do disco a cada commit, por trás do sensor, pelo que um disco lento atrasa o ficheiro, mas nunca as leituras. Uma gravação interrompida a meio de um commit (corte de energia, encerramento forçado) pode deixar o `<name>.daq-journal` ao seu lado:**copie os dois em conjunto.** O Chloros reverte o diário na primeira vez que lê o ficheiro, pelo que a gravação abre com tudo o que foi submetido antes da paragem (perdem-se cerca dos últimos 3 s); na versão 1.2.0, uma falha de energia no momento errado podia tornar todo o ficheiro ilegível («a imagem do disco da base de dados está malformada»).
> - **Uma gravação num disco ocupado já não bloqueia o sensor.** O `daq pool-record` coloca as leituras em fila e grava-as numa thread própria; uma leitura que não pôde ser gravada é contabilizada (o `daq pool-list` apresenta o `N write error(s)` com o último erro) em vez de bloquear o sensor (uma gravação demorou 8,4 s ao lado de quatro câmaras a guardar ficheiros TIFF).
> - **O `lattice multi-capture` sem uma matriz já não perde os fotogramas que guardou quando um disparador atinge o tempo limite.** O disparador de uma câmara que atinge o tempo limite (`SC_ERR_TIMEOUT -1011`, observado com quatro câmaras num switch de 1 GbE) encerra a execução nesse grupo de fotogramas: os grupos já guardados são listados com `Saved:`, seguido do erro, sair do `1`. Na versão 1.2.0, era um erro HTTP500 sem qualquer lista, embora os ficheiros estivessem no disco.
> - **As câmaras acionadas por software já não ignoram cada segundo acionamento.** Uma câmara no modo de acionamento por software coloca agora em fila um acionamento que chegue durante a sua leitura (ou o Chloros aguarda que esteja pronto), tal como as câmaras sincronizadas por cabo já faziam.
> - **`daq pool-discover` identifica um DAQ-E-S `DAQ-E-S`** na sua coluna `MODEL` (e o `--json` na coluna `model`). Indicava `DAQ-E` para todos os sensores Ethernet; aquele que não responde à verificação de identidade no prazo de cerca de 2 s continua listado, como `DAQ-E`.
> - **Produtos de índices LATTICE melhorados** (NDVI e outros índices) a partir de `process`, em todos os modos de processamento.
> - **Câmaras OCN: produtos de radiação, refletância e índices melhorados sob luz solar.** É aplicada uma correção de frota após a separação de bandas; as câmaras RGN permanecem inalteradas e os valores OCN diferem dos da versão 1.2.0. Isto aplica-se também a uma captura OCN feita com uma versão anterior, quando esta é processada num computador que contenha a calibração utilizada na captura (o Chloros mantém a calibração de uma câmara assim que esta é ligada).
> - **O `lattice continuous` grava o fotograma bruto.** Uma execução do `tiff` escreveu fotogramas demosaicados em 3, a partir dos quais o `process` não pode iniciar. Os seus quadros são agora o ficheiro `raw` que uma captura em matriz grava: um plano mosaico com os valores próprios do sensor, não comprimido, com o pacote de calibração. Um script que lia três planos a partir destes ficheiros passa a ler apenas um.
>
> - **Cada captura guarda a leitura do DAQ que utilizou, tal como a GUI faz.** O `lattice capture`, `lattice continuous` (um `.daq` por fotograma), `array-capture` e `multi-capture` com uma matriz ligada (e o SDK) gravam o `.daq` e o seu `.match.json` sempre que um DAQ é vinculado ou agrupado, incluindo uma captura apenas em formato bruto; na versão 1.2.0, uma captura em formato bruto ou com debayering é guardada apenas com o `--force-daq`. O `--no-daq` desativa essa funcionalidade, o `--daq-tolerance-ms MS` define a tolerância de correspondência para essa captura e o novo `daq pool-bind` vincula uma câmara ou um conjunto de câmaras a um DAQ a partir da CLI. Com o `.daq`, uma captura guardada é emparelhada, no `process`, apenas com as câmaras que nela são indicadas. Uma captura aguarda agora a próxima leitura do DAQ para poder combinar as leituras de ambos os lados da exposição. Consulte [A leitura do DAQ que uma captura guarda](#the-daq-reading-a-capture-saves) e [Uma leitura do DAQ distante da exposição](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** adiciona o índice ao projeto para as bandas de cada câmara, incluindo um novo projeto. Na versão 1.2.0, um nome que o projeto ainda não listava era ignorado com o `Unknown index name(s) ignored`.
> - **`process` junto a uma matriz ativa:**já não reinicia um backend que contém câmaras ou DAQs e aguarda até 15 s por uma execução que esteja a chegar ao fim.**Ctrl+C** interrompe a execução no backend, não apenas na CLI. Consulte [Notas sobre o `process`](#notes).
> - **Uma pasta de projeto demasiado profunda para o limite de 260 caracteres do caminho do Windows** é rejeitada logo à partida, sendo utilizada a pasta mais longa que couber, em vez de uma execução que não grava nada. O instalador do Windows oferece agora a opção «Ativar caminhos longos do Windows» e uma exclusão do Microsoft Defender para o backend, ambas marcadas por predefinição. Consulte [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** executa a captura contínua própria do backend: uma linha de progresso a cada 2 s, um resumo em vez de uma lista por ficheiro e um código de saída diferente de zero caso haja algum ficheiro não gravado ou a unidade esteja cheia. Consulte [Modos do `array-capture`](#array-capture-modes).
> - **O primeiro `array-capture` após o `array-connect`** aguarda até 20 s pelo primeiro grupo de fotogramas da matriz, em vez de falhar com o código HTTP 503 «Matriz ainda a aquecer».
> - **Exposição da matriz numa ROI:** uma ROI pequena já não limita a exposição de um membro da matriz ao período de funcionamento livre do sensor (4,9 ms a 768 × 304). Consulte [Exposição num membro da matriz](#exposure-on-an-array-member).
> - **A exposição automática já não reduz a taxa de fotogramas automática de um conjunto.** Um conjunto sincronizado por cabo funciona à taxa permitida pela sua rede e pela leitura das câmaras; a exposição automática preenche o período que essa taxa proporciona e, em seguida, aumenta o ganho. Uma exposição manual mais longa do que o período (`lattice exposure --set`, `configure --set`) abranda o disparo para se adaptar a ela, em vez de ser recusada, e a taxa volta ao normal quando o membro regressa ao modo automático. Consulte [Exposição num membro da matriz](#exposure-on-an-array-member).
> - **As matrizes produzem menos grupos de fotogramas incompletos** e menos erros de captura 503 («Não há nenhum grupo de fotogramas sincronizado recente»): a entrada de disparo de cada câmara escrava está agora livre de falhas.
> - **Transporte compactado de 12 bits** (`--packed`, predefinição `auto`) aumenta as taxas de fotogramas da matriz de sensor completo em cerca de um terço. O `array-connect --packet-size` está agora aplicado (a versão 1.2.0 aceitava-o e ignorava-o). Consulte [Transporte compactado de 12 bits](#packed-12-bit-transport---packed).
> - **Um conjunto cujas câmaras se recuperam automaticamente após um reinício de alimentação.** Após um reinício de alimentação do comutador, o conjunto do `array-connect`-abre as câmaras quando estas respondem e retoma a transmissão com o PTP novamente ativado; na versão 1.2.0, parava definitivamente e era necessário um novo `array-connect`. O `array-status` reporta o `ptp_enabled: False` (e o `ptp_off:`) enquanto uma câmara deteta o PTP desativado, quando antes reportava a configuração com a qual o conjunto estava ligado. Ver [Ligações Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **O `lattice array-record` grava sem a GUI aberta.** Com a vista combinada da GUI fechada, o backend renderiza a própria vista: oNDVI ou o índice que passar com o novo `--expr`. Uma gravação que não escreve nenhum fotograma falha agora (`no-frames: …`, saída `1`). Anteriormente, a gravação só ocorria enquanto a vista combinada da GUI estivesse aberta, e uma gravação que não gravasse nada continuava a sair do `0`. Consulte [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Uma câmara que o Chloros já detém não é aberta uma segunda vez.** Um comando que a abriria por si próprio responde, em vez disso, com um código HTTP 409 `CAMERA_HELD`, indicando o que a detém e o comando que a liberta. Isso inclui uma câmara que outro comando esteja a utilizar neste momento (um `multi-test` ou `multi-capture` a abrir as suas câmaras, um `lattice capture`): o segundo comando é recusado até que o primeiro termine. Abri-la novamente poderia interromper o fluxo do detentor ou causar uma falha no backend. Consulte [Ligações Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Os comandos transitórios para várias câmaras transmitem todas as câmaras ou falham.** `lattice multi-capture` sem nenhum conjunto ligado, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` e `index --live` saem do `1` indicando uma câmara que não transmitiu nenhum fotograma. Os fluxos saltam um grupo de fotogramas em que falta uma câmara e param após três consecutivos; o `multi-test` imprime uma linha por acionamento, o `(no frame)` indica onde uma câmara falhou; o `align-stream` também sai do `1` devido a um erro na tarefa. Cada execução repõe então todas as câmaras tal como as encontrou: a sua ligação de disparador e linha de sincronização, o seu PTP (`PtpEnable`) e todas as definições da câmara que a execução alterou — exposição e ganho, alvo de AE, limites e amortecimento, equilíbrio de brancos, nível de preto, formato de píxeis, binning e ROI, inversões, taxa de fotogramas — a menos que `--keep-settings` / `--clear-settings` (ver [Memória da câmara](#camera-memory----keep-settings----clear-settings)). Anteriormente, estes comandos guardavam todos os fotogramas recebidos e saíam do `0` (o `align-stream` apresentava um erro de tarefa como `Note:`), e deixavam todas as câmaras em `TriggerMode` `On`. Também definiam a exposição de cada câmara com base na cena anterior à captura de um fotograma (no máximo alguns segundos), no alvo em que uma matriz ligada inicia cada câmara com: 40 multiespectral, 60 RGB (o `--brightness N` define-a), e mantém-na durante a execução; o mesmo acontece com um único `lattice capture` de uma câmara que o Chloros não mantém. Anteriormente, o primeiro fotograma de cada câmara era captado no limite de exposição da própria câmara, cerca de 100 ms, independentemente da luz, e era frequentemente cortado. Ver [Multicâmaras (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **Os comandos transitórios de multicâmaras partilham a rede da mesma forma que o `array-connect` e abrem apenas o `--serials`.** Cada câmara é ajustada à sua quota da ligação do anfitrião, pelo que quatro câmaras funcionam numa ligação de 1 Gbps; anteriormente, as quatro solicitavam 125 MB/s entre si, o total de 1 GbE, e cada fotograma perdia pacotes. Uma matriz que a rede não consiga suportar é recusada antes de qualquer câmara transmitir. O `--serials` escolhe agora as câmaras, sendo a primeira a acionar o gatilho; anteriormente, cada comando abria todas as câmaras na rede, independentemente do que o `--serials` indicasse. Uma falha que consiste na perda de pacotes GVSP agora indica isso, em vez de apontar para o cabo de sincronização M8. Ver [Multicâmaras (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **O `lattice array-disconnect` deixa as câmaras a funcionar livremente, tal como foram encontradas.** `TriggerMode`, `Off` e tudo o resto tal como a ligação o encontrou — as definições do disparador e da linha de sincronização (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP e as definições da câmara alteradas pela sessão — de modo que um únicocomando de câmara num antigo membro obtém os fotogramas de uma só vez, e o mestre já não fica em `TriggerSource` `Software`. O `--keep-settings` guarda as definições da sessão na memória de cada câmara, enquanto o `--clear-settings` reinicia essa memória (ver [Memória da câmara](#camera-memory----keep-settings----clear-settings)). O `lattice continuous` também define uma câmara que encontre armada para funcionamento livre durante a corrida (e volta a colocá-la no modo normal depois), e uma corrida sem quadros sai do `1`.
> - **`lattice configure --get` / `lattice exposure`** imprimem um `note:` com aexposição ou ganho reais enquanto o modo automático está ativo. Um `lattice exposure --set` / `configure --set` rejeitado indica o intervalo que a câmara aceita e reativa o modo automático, em vez de desativar a exposição automática. Consulte [Ajuste do sensor](#sensor-tuning).
> - **Novos controlos da câmara:**`lattice highlights` (reduz o brilho das superfícies brancas na renderização do ecrã RGB) e `lattice exposure --brightness-auto`.**`lattice multi-detect`** deteta as câmaras com fio numa única ronda de sondagem e define por predefinição a câmara RGB como câmara principal.
> - **Novos comandos DAQ:** `daq pool-bind`; `pool-enable-ptp`; e para as definições próprias de um DAQ-E: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` e `pool-update-firmware`. Consulte [Definições do dispositivo DAQ-E](#daq-e-device-settings).
> - **Um DAQ-E que deixa de transmitir é reconectado pelo backend.** Quando o fluxo de um DAQ-E em pool fica inativo (devido a um reinício do switch que o alimenta, uma perda de ligação da câmara), o backend volta a ligá-lo com as definições que tinha, tentando novamente a cada intervalo crescente enquanto estiver ausente. Uma gravação nesse dispositivo continua no mesmo `.daq` com a lacuna marcada no ficheiro, e o `pool-record --stop` lista cada interrupção. Na versão 1.2.0, o sensor permanecia inativo até ao `pool-disconnect` + `pool-connect`, o que também encerrava a gravação. Ver [Um DAQ-E que deixa de transmitir](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** indica se os pacotes do «grandmaster» PTP chegam à rede (`health`), com um `WARNING:` no stderr quando isso não acontece. Consulte [`time-sync`](#chloros-cli-time-sync).
> - **.deb para Linux:** aumenta os buffers de receção de sockets do kernel necessários para os fluxos das câmaras (na configuração padrão do kernel, as matrizes perdem fotogramas e a taxa de fotogramas), adiciona um serviço de backend que é executado como o seu utilizador e o `lattice network` verifica os buffers. A CLI nunca interrompe um backend executado por um serviço, e uma atualização reinicia os serviços de backend ativados. Consulte [Executar o backend como um serviço](#running-the-backend-as-a-service-headless) e [Hosts Linux — buffers de receção de socket](#linux-hosts--socket-receive-buffers).
> - **O `lattice array-capture` indica quando não foi possível alinhar:**cada motivo pelo qual uma captura foi guardada desalinhada (ausência de perfil de alinhamento, membros com resoluções diferentes, uma matriz não sincronizada com o cabo, um quadro no qual o perfil não pode ser mapeado) é apresentada como uma linha `Alignment:`.**`chloros-cli daq help`** é `daq --help` (saiu de `2`) e, no Linux, a ajuda lista os comandos de hardware direto `daq` que essa compilação inclui.
> - **Jetson:** o pacote é o `chloros_1.3.0_arm64_jp6.deb`, para o JetPack 6, e o seu processamento por GPU é executado no Orin com o CUDA PyTorch do JetPack. Consulte [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Convenções

- Todos os comandos têm o prefixo `chloros-cli`. No Windows, o binário é `chloros-cli.exe`; no Linux/Jetson é `chloros-cli`.
- Os argumentos opcionais são apresentados como `--flag`. Os argumentos posicionais obrigatórios são apresentados sem parênteses.
- Quando é fornecido um valor por predefinição, a omissão do sinalizador utiliza esse valor.
- A CLI é um cliente HTTP leve sobre o backend Chloros (o servidor em `127.0.0.1:5000` que a aplicação de ambiente de trabalho também utiliza). **Quais os comandos que a iniciam:**
  - Os comandos principais — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` e `selftest` — utilizam um backend em execução, ou iniciam um quando nenhum estiver a funcionar e param-no novamente quando terminam.
  - **Os comandos `lattice`, `project` e `daq` nunca iniciam um.** Estes requerem um backend que já esteja a funcionar: o da aplicação para o ambiente de trabalho do Windows, o [serviço de backend](#running-the-backend-as-a-service-headless) do Linux, ou um que inicie você mesmo (consulte [Iniciar o Backend](#start-the-backend)). Sem um backend, exibem o código de erro `Error: Chloros backend not reachable at http://127.0.0.1:5000`, juntamente com o comando que o inicia, e saem com o código de erro `1`.
- O `CHLOROS_BACKEND_URL=<url>` redireciona as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para outro backend (o `lattice network-analysis` direciona para o `--backend-url`, em vez disso). Os comandos principais fixam deliberadamente o `http://127.0.0.1:<port>` e ignoram-no (o literal IPv4 evita a penalização de cerca de 2 s por pedido do Windows para `localhost`→`::1`). Consulte [Variáveis de ambiente](#environment-variables).
- **Os caminhos relativos são resolvidos em relação à pasta a partir da qual o `chloros-cli` é executado.** O backend é o processo que abre as pastas de saída e de entrada; por isso, antes de enviar um caminho, a CLI torna-o absoluto: `-o` / `--output` (incluindo o seu valor por predefinição, `output`), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask`, um etapa [recipe](#recipe-dsl) `output` (padrão `output`), e a pasta predefinida `output` de `lattice index --live`. `-o captures/` grava, portanto, em `<current folder>/captures/`, e os caminhos impressos são absolutos. `~` é expandido. No caso de um backend remoto (`CHLOROS_BACKEND_URL` que indica outra máquina), os caminhos são enviados tal como digitados e resolvidos nessa máquina.
- É necessário um login com uma conta Chloros+ para todas as chamadas SDK/CLI (execute o `chloros-cli login` uma vez por máquina; armazenado em cache no `~/.chloros/`).
- Os exemplos utilizam caminhos do Linux; no Windows, substitua `/home/user/...` por `C:/Users/.../...`. A porta série de um DAQ-Ué do tipo `COM3` no Windows e `/dev/ttyUSB0` / `/dev/ttyACM0` no Linux.
- As sinopses indicam os sinalizadores mutuamente exclusivos como `{--a | --b}` e as opcionais em `[...]`.

---

## Sinopse de nível superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opções globais

| Sinalizador | Descrição |
| --- | --- |
| `--backend-exe PATH` | Substituir o executável do backend detetado automaticamente. |
| `--port N` | Porta HTTP do backend (predefinição: `5000`). Utilizada apenas pelos comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); os comandos `lattice`, `project` e `daq` utilizam o comando `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` por predefinição). |
| `-v, --verbose` | Ativar saída detalhada. |
| `--restart` | Reinicia forçadamente o backend (encerra qualquer instância do `backend_server.py` em execução). Um backend executado por um serviço do systemd (.deb do Linux) nunca é interrompido: o comando apresenta `sudo systemctl restart <unit>` e sai `1`. |
| `--version` | Exibe a versão (`Chloros CLI 1.3.0`). |
| `--help` | Mostra a ajuda de nível superior. |

### Índice de comandos

| Comando | Finalidade |
| --- | --- |
| [`process`](#chloros-cli-process) | Processar uma pasta de capturas do Survey3 ou do LATTICE de ponta a ponta. |
| [`login`](#chloros-cli-login) | Autenticar esta máquina com uma conta Chloros+. |
| [`logout`](#chloros-cli-logout) | Limpar as credenciais armazenadas em cache. |
| [`status`](#chloros-cli-status) | Mostrar o estado atual da licença/autenticação. |
| [`export-status`](#chloros-cli-export-status) | Progresso em tempo real da exportação do Thread-4 durante uma execução do `process`. |
| [`language`](#chloros-cli-language) | Definir ou listar o idioma de exibição da CLI (38 idiomas suportados). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Pasta de projeto predefinida (partilhada com a GUI). |
| [`update`](#chloros-cli-update) | Verificar e instalar atualizações do Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnósticos do sistema + testes de funcionamento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Estado/controlo do «grandmaster» PTP. |
| [`lattice`](#chloros-cli-lattice) | Controlo e captura da câmara LATTICE (mais de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Controlo do sensor espectral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir e executar um projeto Chloros guardado (câmaras + DAQs). |

---

## Instalação

O `chloros-cli` é fornecido no instalador do Chloros para computador em todas as plataformas suportadas — não existe um download separado da CLI. A instalação do pacote da plataforma adiciona o `chloros-cli` ao seu `PATH`, juntamente com a aplicação para computador e o binário de backend que esta controla.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> O instalador também inclui scripts de lançamento práticos (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abrem um shell CLI pronto a utilizar.

### Windows (.exe)

1. Descarregue o instalador para Windows a partir da página de downloads.
2. Execute o `Chloros Installer 1.3.0.exe` e siga as instruções do assistente. O caminho de instalação predefinido é `C:\Program Files\Chloros\` (o CLI fica em `C:\Program Files\Chloros\cli\`, que o instalador adiciona à variável PATH).
3. Abra um novo terminal (`cmd.exe`, PowerShell ou Terminal do Windows) para que o `PATH` atualizado seja detetado.

```powershell
chloros-cli --version
```

O instalador adiciona automaticamente o `chloros-cli.exe` ao seu sistema `PATH` e inclui os controladores da câmara LATTICE. O backend é o `C:\Program Files\Chloros\backend\chloros-backend.exe` (a pasta `backend` da sua pasta de instalação), que não se encontra no `PATH`.

**Definições do sistema.** A página *Selecionar tarefas adicionais* do instalador apresenta duas opções em **Definições do sistema**, ambas**marcadas por predefinição**:

| Opção (conforme descrita no instalador) | O que faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** (permite que as pastas de projeto tenham mais de 260 caracteres; recomendado) | Define o parâmetro `LongPathsEnabled` do Windows, válido para todo o computador, para 1. O Windows vem com esta opção desativada, e, nesse caso, nenhum caminho pode ultrapassar os 260 caracteres; os produtos da Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto, pelo que uma pasta de projeto mais profunda é recusada (ver [Resolução de problemas](#dicas-de-resolução-de-problemas)). Entra em vigor para programas iniciados após a instalação. Mantém-se ativado após a desinstalação, uma vez que outros programas podem depender desta configuração. | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender**(captura de imagem muito mais rápida; recomendado) | Adiciona uma exclusão de**processo** do Defender para o `chloros-backend.exe` na pasta de instalação, para que a proteção em tempo real não verifique cada ficheiro que o backend escreve, independentemente do local onde o escreve. Nenhum outro programa é excluído; a exclusão é removida na desinstalação. Se o Defender estiver ausente ou bloqueado por política, o passo falha silenciosamente e a instalação prossegue. | `/MERGETASKS="!defenderexclusion"` |

**Por que razão a opção do Defender é importante:** em taxas de captura elevadas, a verificação em tempo real dos ficheiros de captura consome cerca de 30 % da taxa de captura (medido numa captura contínua com 9 câmaras: 77,7 capturas/s excluídas, 52,7 verificadas). O `lattice array-capture --continuous` apresenta uma nota quando deteta que o Defender está a utilizar um núcleo da CPU ou mais durante a captura; volte a executar o instalador com a opção marcada.

Uma instalação silenciosa (`/SILENT` ou `/VERYSILENT`) aplica ambas as opções, a menos que sejam desativadas, por exemplo, `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall do Windows.** O instalador adiciona sempre duas regras de entrada para o `chloros-backend.exe`, em todos os perfis de rede, e o desinstalador remove-as: `Chloros Backend (UDP-In)`, para fluxos de câmara, PTP, mDNS e o fluxo de dados do DAQ-E, e `Chloros Backend (TCP-In)`, limitada à sub-rede local, para uma atualização do firmware do DAQ-E, na qual o sensor se liga novamente a este computador para recolher a imagem (ver [Definições do dispositivo DAQ-E](#daq-e-device-settings)). A API HTTP do backend permanece ligada a `127.0.0.1`.

### Linux amd64 (.deb)

Para estações de trabalho x86_64 com Ubuntu 22.04 LTS ou mais recente / baseadas em Debian.

> **O Ubuntu 20.04 não é suportado.** A lista de dependências do pacote deriva daquilo
> a que o backend está efetivamente ligado, o que inclui o `libc6 (>= 2.35)`;
> o Focal inclui a glibc 2.31. O `apt` recusa a instalação em vez de permitir que esta falhe em
> tempo de execução. A lista também menciona o `libgl1`, que o OpenCV incluído carrega e que pode faltar numa
> imagem mínima de servidor; o `apt install` fornece-o.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

O ficheiro .deb instala:
- `chloros-cli` até `/usr/bin/chloros-cli` e `chloros-backend` (inicia o backend) até ao `/usr/bin/chloros-backend`
- O backend compilado e a CLI, descompactados, em `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`
- Os controladores da câmara LATTICE
- Modelos de redução de ruído, pacotes de calibração e configuração do canal de atualização
- O SDK Python para o sistema `python3` (`import chloros_sdk`). O `apt install` também inclui os seus pré-requisitos, `python3-numpy` e `python3-requests` (recomendados, mas não obrigatórios: os programas acima são autónomos; o `dpkg -i` ignora-os, e a saída da instalação indica o que falta). Num virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, que aumenta os buffers de receção de sockets do kernel para os fluxos da câmara para 32 MB e aplica-os de uma só vez (um valor já definido para um valor superior é mantido). A remoção do pacote remove o ficheiro. Consulte [Hosts Linux — buffers de receção de sockets](#linux-hosts--socket-receive-buffers).
- Dois serviços systemd opcionais para o backend, nenhum deles ativado (próxima secção)

#### Executar o backend como um serviço (sem interface gráfica)

Sem um serviço, o backend (e, com ele, o grandmaster PTP) só funciona enquanto algo o tiver iniciado: `chloros-backend` num terminal, um script SDK ou um comando principal `chloros-cli` durante o tempo em que estiver em execução. Para um backend sempre ativo numa máquina sem interface gráfica, ative o serviço **como o seu utilizador**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

O `chloros-backend@<user>.service` executa o backend como esse utilizador. As capturas que grava são dele, o seu registo detalhado encontra-se no seu `~/.cache/chloros/logs`, e utiliza o seu `chloros-cli login` (`~/.chloros`), pelo que deve iniciar sessão uma vez como esse utilizador primeiro. O PTP não necessita de direitos de root. Tal como qualquer backend instalado, escuta apenas no `127.0.0.1:5000`.

O `chloros-backend.service` executa o backend como **root**. Continua instalado para que uma máquina que o tenha ativado continue a funcionar, mas tudo o que um backend com privilégios de root escreve pertence ao root, pelo que um utilizador não pode eliminar as suas próprias capturas. Para mudar para o seu utilizador:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

O registo detalhado do serviço root encontra-se em `/var/log/chloros`, legível pelos membros do grupo `adm` (o serviço define `CHLOROS_LOG_DIR`; ver [Variáveis de Ambiente](#environment-variables)). A instalação do pacote apresenta o comando `enable` com o seu nome de utilizador preenchido, bem como a opção de mudança quando o serviço root está ativado.

**Ativar uma unidade de backend.** Apenas uma unidade de backend funciona de cada vez. Uma segunda unidade ativada (`chloros-backend.service` juntamente com `chloros-backend@<user>.service`, ou duas unidades `@` de utilizadores diferentes) recusa-se a iniciar e permanece como `failed` com o estado de saída 98; o systemd não a reinicia. O seu `systemctl status` indica o motivo: `Another instance is already running. Exiting.`, ou seja, o que retém o backend (a porta 5000, ou o ficheiro de bloqueio e o utilizador a quem pertence), e, em seguida, o `systemd will not restart <unit> (exit status 98)`. Após um reinício, qualquer uma das unidades pode ser a que fica a funcionar. Para alternar, desative a que não utiliza, conforme descrito acima.**A CLI nunca interrompe o backend de um serviço.** O `chloros-cli` utiliza-o tal como está, tal como a GUI e o SDK: o `process` é executado nele, carregando o seu próprio projeto no lugar do último. Nos casos em que a CLI, de outra forma, reiniciaria o backend (`--restart`, ou um `process` cuja importação não encontrou nada), ela identifica a unidade, exibe «`sudo systemctl restart <unit>`» e sai do «`1`», porque um backend parado dessa forma permaneceria inativo. O `GET /api/status` nomeia a unidade como `service` (`null` para um backend em que nenhum serviço está a ser executado).**Atualização e remoção.** A atualização do pacote interrompe todos os serviços de backend e, em seguida, reinicia cada um dos que estão ativados, começando pelas unidades `@<user>` (um serviço iniciado manualmente, mas nunca ativado, permanece parado, tal como após um reinício). Com mais do que um serviço ativado, apenas um é executado e os restantes permanecem com falha, tal como acima; a atualização apresenta os códigos `Chloros: N backend units are enabled: …` e `Keep one enabled: sudo systemctl disable --now <unit>`. A remoção interrompe e desativa esses serviços. A purga também elimina o `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Tem o mesmo layout que o ficheiro .deb para amd64, incluindo buffers de receção e [serviços de backend](#running-the-backend-as-a-service-headless), para Jetson Orin / Orin NX / Orin Nano no **JetPack 6** (L4T R36). O seu PyTorch é a compilação CUDA do JetPack 6 para o Orin, pelo que o processamento na GPU, incluindo o debayer sensível à textura, é executado na GPU do Jetson. O pacote não inclui o CUDA propriamente dito: utiliza as bibliotecas CUDA e cuDNN que o JetPack 6 instala (`sudo apt install nvidia-jetpack` numa imagem que não as possua). Quando não há nenhuma GPU CUDA disponível, o processamento é executado na CPU e o processamento com reconhecimento de textura utiliza o debayer padrão (ver [`process`](#chloros-cli-process)).

### Autenticação única por máquina

Todas as plataformas requerem um login único no Chloros+ antes de as chamadas SDK/CLI funcionarem:

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache em `~/.chloros/user_session.json`.

### Iniciar o backend

Os comandos `lattice`, `project` e `daq` comunicam com um backend que já se encontra em execução; nunca iniciam um (consulte [Convenções](#convenções)). Mantenha um em execução enquanto os utiliza:

| Plataforma | Como |
| --- | --- |
| Windows | Abra a aplicação de ambiente de trabalho Chloros (o seu backend também serve a CLI), ou execute o `"C:\Program Files\Chloros\backend\chloros-backend.exe"` num terminal próprio (na pasta `backend` da sua pasta de instalação) e deixe-o aberto. |
| Linux/Jetson | Execute o `chloros-backend` num terminal próprio ou mantenha um sempre em execução com o [serviço de backend](#running-the-backend-as-a-service-headless). |

Apenas um backend pode estar em execução de cada vez; todos os clientes (a GUI, a CLI, os scripts do SDK) partilham-no. Os comandos principais (`process`, `login`, `status`, …) também o utilizam e iniciam um quando nenhum estiver a funcionar.

**Um backend continua a funcionar entre comandos**, pelo que o seguinte não tem de pagar novamente a sua inicialização. Ele pára quando se desligar (`chloros-cli logout`, a menos que a aplicação Chloros esteja aberta ou que uma execução ou uma compilação offline esteja em curso), quando executar o `chloros-cli backend stop`, ou automaticamente após 30 minutos sem que nada o esteja a utilizar: sem comandos, scripts ou aplicações, sem execução ou compilação offline, sem câmara ligada ou sensor de luz. O `backend stop` recusa enquanto uma execução ou uma compilação offline estiver em curso, indicando o nome da mesma; o `--force` interrompe-o imediatamente e essa execução ou compilação é perdida (uma sequência é mantida, e o `array-build-video` compila-a novamente). Defina o `CHLOROS_BACKEND_IDLE_MIN` antes de o backend iniciar para alterar os 30 minutos (`0`: nunca); o [serviço backend](#running-the-backend-as-a-service-headless) nunca pára por si próprio. O `chloros-cli backend status` indica se está algum a ser executado, o que o está a utilizar e se há alguma compilação offline em execução. O `process` abre o seu projeto no backend em execução, a menos que a aplicação Chloros esteja aberta com um projeto que abriu: nesse caso, recusa a ação, e deve processar a partir da aplicação ou fechar esse projeto primeiro. Um projeto que um `process` anterior ou um script SDK tenha deixado aberto é simplesmente substituído.

### Verificar a instalação

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

O `selftest` só sai do `0` quando todas as sete verificações forem bem-sucedidas. A última requer uma GPU CUDA (uma GPU NVIDIA, ou de um Jetson Orin) e os modelos de redução de ruído; por isso, num computador sem uma GPU, apresenta o código de erro `Failed: CUDA + Denoiser` e encerra o `1`; tudo, exceto o debayer sensível à textura exclusivo da GPU, continua a funcionar nesse caso.

> **É necessária uma subscrição do Chloros+.**A CLI requer um plano Chloros+ ativo.**Copper**é o nível básico do Chloros+ — todos os níveis pagos do Chloros+ têm acesso à CLI/SDK; apenas o nível gratuito**Iron** não tem. (Correspondência de IDs de plano: `0`=Iron/gratuito, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Atualize em [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Este limite mínimo é imposto pelo backend, não apenas pela CLI: um pedido marcado como SDK/CLI sem um plano pago é rejeitado com o código `403 PLAN_UPGRADE_REQUIRED`, quer provenha do `chloros-cli`, do Python SDK ou de um cliente HTTP personalizado. Um utilizador que tenha saído da sua conta recebe, em vez disso, o código de erro `401 AUTH_REQUIRED`. O acesso funciona offline durante o período de carência do plano (30 dias para planos mensais; até ao vencimento para planos anuais) e cessa quando este expira; o código de erro `chloros-cli status` continua a funcionar para que o motivo seja visível (é a única rota SDK/CLI isenta do controlo de níveis — `GET /api/license-status`).

---

## Guia de início rápido em cinco minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

O `process` (passo 2) inicia um backend para a sua própria execução quando nenhum estiver a funcionar e interrompe-o quando terminar; por isso, inicie um a partir do passo 4. Consulte [Iniciar o back-end](#start-the-backend).

---

## `chloros-cli process`

Processe uma pasta de imagens através do pipeline completo do Chloros (detecção de alvo → calibração → vinheta → refletância → exportação do índice).

### Sinopse

```
chloros-cli process INPUT [OPTIONS]
```

### Argumentos posicionais

| Argumento | Descrição |
| --- | --- |
| `INPUT` | Caminho para a pasta de entrada que contém ficheiros `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou `.dng`. |

### Opções comuns

| Sinal | Padrão | Descrição |
| --- | --- | --- |
| `-o, --output PATH` | uma nova pasta com o nome `-n` na pasta padrão do projeto | A pasta do projeto. O projeto recebe o nome desta pasta, e uma pasta que já contenha um ficheiro `project.json` recebe um nome do tipo `_1` / `_2` em vez de ser substituída. A pasta de projeto predefinida é aquela escolhida pelo `set-project-folder` (ou pela GUI); caso contrário, é a `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automático (carimbo de data/hora) | Nome do projeto, utilizado apenas **sem** o `-o` (com o `-o`, o projeto assume o nome da pasta). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utiliza um debayer neural «Chloros+» para imagens do Survey3; mais lento, mas de maior qualidade. Destina-se apenas ao Survey3: As capturas LATTICE utilizam sempre o demosaico padrão; por isso, no caso de uma pasta que contenha capturas LATTICE, o `process` rejeita o `texture-aware` após a importação, apresentando um erro que indica o número de imagens existentes, e encerra o `1` antes de processar qualquer coisa (a aplicação Chloros também não oferece o Texture Aware para estas imagens). É necessária uma GPU CUDA (uma GPU NVIDIA no Windows / Linux x64, ou a GPU do Orin num Jetson): sem ela, a execução utiliza o `standard`, e o backend regista uma linha `[DEBAYER] … needs a CUDA GPU …` a explicar o motivo (mostrada com o `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Correção de vinheta. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibração de refletância (utiliza o alvo do painel, se encontrado; calibração NIST por série para o LATTICE). Para o LATTICE multiespectral, isto funciona também como o **produto** de refletância — consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desativado | Aplicar correções PPK GNSS a partir de ficheiros sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | nenhum | O modelo da câmara ligado ao pino de exposição 1, para sincronização temporal PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | nenhum | O modelo da câmara ligado ao pino de exposição 2, para sincronização temporal PPK. |
| `--recal-interval SECONDS` | `0` | Intervalo mínimo de recalibração, em segundos (o *Intervalo mínimo de recalibração* do projeto). |
| `--timezone-offset HOURS` | `0` | Desvio do fuso horário do sensor de luz em relação ao UTC, em horas (o *Desvio do fuso horário do sensor de luz* do projeto). |
| `--format FORMAT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. O Chloros nunca aumenta a profundidade de bits de uma imagem: em imagens totalmente de 8 bits, um formato com maior profundidade é gravado como `TIFF (8-bit)` e o código indica isso uma vez. Num projeto que misture capturas de 8 bits e de 12/16 bits, cada captura é exportada com a sua própria profundidade ou uma profundidade inferior. O Radiance é sempre de 32 bits em ponto flutuante. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação por nome de predefinição (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), sem distinção entre maiúsculas e minúsculas. Cada um é adicionado ao projeto para as bandas da respetiva câmara, sendo incluído um novo projeto. Um nome que não seja uma predefinição, ou que as bandas da câmara não consigam gerar (por exemplo, «`NDRE`» requer RE e NIR, que nenhuma câmara fornece sozinha), é indicado num aviso e não recebe nenhuma imagem de índice. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para ficheiros TIFF do LATTICE (os ficheiros .raw do Survey3 não são afetados). Além disso, a saída de emergência que permite que uma captura **sem ficheiro raw** seja processada na totalidade — ver [Como é uma pasta de capturas](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ativado | Emite o produto linear pós-debayering (`Debayered_Images`). Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ativado | Emite a pré-visualização do ecrã (`Preview_Images`): RGB = equilíbrio de brancos (iluminante DAQ, quando disponível; caso contrário, «gray-world») + gama; multiespectral = alongamento de cores falsas. |
| `--radiance / --no-radiance` | ativado | Emite radianância em float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referência para o produto de refletância LATTICE: `auto` = o alvo dentro do quadro que passou no controlo de qualidade (QA) é a referência absoluta (um alvo com um sinalizador de QA compete com o DAQ na pontuação medida), recurso de fallback para o DAQ descendente (ρ = π·L/E); `target` = rigoroso (sem substituição do DAQ); `daq` = autoridade do DAQ. Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nenhum | Diretório de varreduras de refletância-alvo **medidas** por unidade (`<serial>.csv`); recorre aos espectros T3/T4P nominais em caso de falha. |
| `--array-alignment / --no-array-alignment` | ativado | Matrizes LATTICE: aplicam o alinhamento módulo a módulo registado no ficheiro XMP de cada captura (`Chloros:Alignment*`) a todos os produtos processados (desbayering / pré-visualização / radianância / refletância / índice). Não tem efeito em imagens sem as etiquetas. |
| `--array-alignment-crop / --no-array-alignment-crop` | recorte | Recorta as exportações alinhadas à região de sobreposição comum da matriz, para que todos os módulos partilhem uma única área de cobertura; o `--no-…` mantém a área total do sensor (preenchimento a preto fora da fonte). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reamostragem para a distorção de alinhamento. `nearest` preserva os DNs exatos da fonte (sem mistura interpixel de valores radiométricos). |

### Opções de deteção de alvos

| Indicador | Descrição |
| --- | --- |
| `--min-target-size PIXELS` | Tamanho mínimo do alvo de calibração em píxeis para o detetor. |
| `--target-clustering 0-100` | Percentagem de agrupamento de alvos (valor predefinido `60`). |
| `--target / --targets` | Pesquisar alvos de calibração apenas numa subpasta `target` ou `targets` da entrada (mais rápido). |

### Exemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Opções de exportação por produto (LATTICE multiespectral)

O processamento LATTICE ramifica-se para **todos os produtos aplicáveis numa única passagem**. As quatro opções por tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — estão todas**ativadas por predefinição**; utilize o formulário `--no-<type>` para desativar uma. As câmaras principais RGB emitem apenas imagens sem debayering + pré-visualização (sem radiação/refletância por banda), pelo que `--radiance`/`--reflectance` não têm qualquer efeito para elas. Os interruptores são ignorados para o `.raw` do Survey3 (que segue o caminho padrão de refletância/alvo). *(O antigo sinalizador `--radiometric-output {reflectance,radiance,sensor-response}` foi **removido** e substituído por estes comutadores; já não existe o nível `sensor-response`.)*

| Produto | Saída | É necessária aquisição de dados de radiação descendente? |
| --- | --- | --- |
| `--debayered` | Desmosaico linear (`Debayered_Images`): o mesmo ficheiro que uma captura guarda no seu nível `debayered` — o desvio de escuro (DSNU) do pacote de calibração aplicado ao ficheiro bruto, seguido do demosaico sensível a bordas do OpenCV; sem LUT de linearidade, campo plano ou separação de cores. | Não. |
| `--preview` | Pré-visualização no ecrã (`Preview_Images`): RGB = WB + gama; multiespectral = alongamento de cores falsas. | N.º |
| `--radiance` | float32 W/m²/sr/nm proveniente da cadeia radiométrica completa (`Radiance_Images`). | N.º |
| `--reflectance` | uint16 refletância ρ (`32768` = 1,0), Compatível com Pix4D. | **Sim**, a menos que seja ancorada por um alvo no quadro que tenha passado no controlo de qualidade (ver abaixo). |

`--reflectance-source` seleciona a referência de refletância:**`auto`**(predefinição) torna um alvo no quadro que tenha passado no controlo de qualidade a**referência absoluta**— as cadeias de linhas empíricas ancoradas no alvo são comparadas com painéis de teste e o resultado vencedor é aplicado, banda a banda: uma banda que o DAQ não consiga calibrar, ou cujo ajuste da inclinação do DAQ falhe, adota a própria linha do alvo (`ReflectanceElBandMethods` indica qual) — um alvo que passa no controlo de qualidade com uma marcação (um painel excluído, uma banda fora de ordem) também compete com a divisão do DAQ com base nessa pontuação medida, e o quadro ao qual perde é marcado como `ReflectanceChain=daq` — recorrendo à divisão descendente do DAQ (ρ = π·L/E) quando não há alvo presente ou o controlo de qualidade falha;**`target`**é rigoroso (sem substituição do DAQ);**`daq`**opta pelo comportamento autoritário do DAQ. A geometria do alvo (ArUco / ROI fixa / faixa) provém da configuração do alvo do projeto; `--target-reflectance-dir DIR` mantém, por unidade, as**varreduras *medidas** por unidade (`<serial>.csv`), consultados através do número de série/QR da unidade-alvo, utilizando os espectros T3/T4P nominais como alternativa.

O caminho de refletância do DAQ determina automaticamente a **radiação descendente com carimbo temporal correspondente**a partir de um**`.daq`**(DAQ-U/M/E)**ou de um `.csv` nativo do DAQ-M**encontrado juntamente com as imagens. Se um pacote de calibração por câmara ou DAQ não estiver armazenado em cache localmente, o pipeline**obtém-no automaticamente da AWS** na primeira utilização (requer ligação à Internet uma vez; armazenado em cache sob `~/.chloros/`).**A quem pertence esta leitura.**Um `.daq` guardado por uma captura corresponde à leitura da(s) câmara(s) que descreve — identificada(s) num ficheiro `<file>.daq.cameras.json` anexo ao seu lado (`lattice capture`, `multi-capture`, `array-capture` e na GUI; uma sequência de imagens em bruto indica a leitura de cada membro no seu manifesto) — e o `process` associa-a**apenas a imagens dessas câmaras**. Uma câmara autónoma sem leitura própria, portanto, já não utiliza o DAQ de outra câmara; associe-lhe um DAQ ou grave uma. A gravação própria de um DAQ (`daq pool-record`, ou uma gravação iniciada na GUI), ou qualquer `.daq` sem esse ficheiro auxiliar (incluindo capturas mais antigas), continua a servir todas as câmaras.

#### Leitura de píxeis de refletância (Pix4D / Metashape / os seus próprios scripts)

A refletância é armazenada como um DN inteiro, e **o DN que corresponde a ρ = 1,0 depende da câmara de origem**:

| Origem | ρ = 1,0 corresponde a | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem até ρ 2,0) | O ficheiro contém a marca XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há marcas XMP `Chloros:*` — essa ausência *é* o indício. |

**Leia o valor de `Chloros:PixelScale` e divida por ele**, em vez de assumir um valor constante. A etiqueta está definida no domínio uint16, pelo que se mantém como `32768` em todos os formatos de saída que redimensionam — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` são todos autodescritivos (normalize primeiro o tipo de dados armazenado de volta para uint16: ×257 a partir de 8 bits, ×65535 a partir de float).

> **Os ficheiros de compilações mais antigas podem não conter escala.** Um ficheiro TIFF de refletância de 8 bits é redimensionado e rotulado como `Chloros:PixelScale`, tal como os outros. As compilações anteriores à versão 1.3.0, em vez disso, *limitaram* o intervalo a 0..255, pelo que todos os valores acima de ρ≈0,008 foram nivelados para 255, e omitiam tanto o `Chloros:PixelScale` como o tuplo `MicaSense:RadiometricCalibration`. **Se a etiqueta estiver ausente num ficheiro de refletância LATTICE, não presuma que existe uma escala — reprocesse-o** em vez de dividir píxeis que nunca foram divisíveis.

#### Leitura de píxeis de índice (NDVI e outros índices de vegetação)

**Uma raster de índice inteiro é um alongamento de contraste por imagem, não um mapa de −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — incluindo o ficheiro TIFF de 8 bits em que uma captura de 8 bits é gravada — armazenam

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

onde `IndexMin` / `IndexMax` são **os valores de índice finitos mais baixo e mais alto próprios desse fotograma**. DN 0 não é −1, e a escala completa não é +1.

O Chloros marca ambos os extremos para que o mapeamento se mantenha invertível:

| Etiqueta | Significado |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — a convenção abaixo |
| `Chloros:IndexMin` | o valor de índice armazenado como DN 0 |
| `Chloros:IndexMax` | o valor de índice armazenado como escala total |

Leia-os e inverta-os, normalizando o DN armazenado para 0…1 com base no valor máximo do tipo de dados (dtype) armazenado, de modo a que uma única expressão abranja todos os formatos inteiros:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

A leitura não requer o `-config pix4d.config` — o exiftool lê estas propriedades XMP por si próprio. A configuração só é necessária para *gravá-las*.

> **`IndexMin == IndexMax`** significa que cada pixel desse fotograma continha o mesmo valor de índice. A rasterização é preenchida a meio da escala e não há nada por que dividir — utilize o ponto final.

> **A ausência de tags `Chloros:Index*` significa que não há mapeamento definido.** Uma exportação LUT é uma renderização colorida cujos píxeis são cores em gradiente associadas à sua janela de limiar LUT, e um fotograma sem píxeis de índice finito não tem intervalo a definir. Não presuma a existência de um mapeamento em nenhum dos casos — aplica-se a mesma regra que para a ausência de `PixelScale`.**Como o alongamento é por imagem, dois fotogramas de um voo não são comparáveis em termos de DN**— cada um tem os seus próprios pontos finais, pelo que o mesmo NDVI resulta em valores de DN diferentes em imagens diferentes. Para trabalhos quantitativos ao longo de um voo (estatísticas zonais, limiares fixos, séries temporais), exporte**`TIFF (32-bit, Percent)`**: as rasters de índice são gravadas nesse formato como float32**os próprios valores de índice**, sem necessidade de reescalonamento nem de etiquetas. Apesar do nome do formato, os produtos de índice não são ×100 — essa escala aplica-se apenas à refletância com tipo de dados inteiro.

#### Máscara de limitação da separação espectral (em que píxeis uma banda foi limitada)

A separação espectral 3×3 pode tornar uma banda negativa — quando a luz mal contém essa banda, ou quando a cena fica fora do que a matriz de separação da câmara descreve. Esse não é um valor real, pelo que a exportação o fixa em 0 — e a visualização em tempo real mostra o mesmo 0. Uma banda pode ser limitada a alguns píxeis ou a quase todos eles; a máscara regista qualquer uma das situações. O `Chloros:Band{R,G,B}UnmixClampFraction` indica em que proporção de um fotograma isso ocorreu; a máscara de limitação indica **quais os píxeis**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— numa pasta `clamp_masks/`**ao lado** da pasta do produto, nunca dentro dela (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; uma exportação de `process` resulta em `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — para cada produto de radiação, refletância e índice em que pelo menos um pixel tenha sido limitado: capturas e exportações `process` semelhantes (e os ficheiros TIFF por fotograma de `array-build-video --save-tiffs`). `uint8` de banda única, na própria grelha de píxeis do produto (incluindo orientação, ROI, binning e alinhamento da matriz). Cada máscara identifica a câmara do seu produto (`Make` / `Model` / número de série), a hora da captura e as etiquetas da lente, e lista as bandas que os seus bits representam (`XMP-Camera:BandName`, começando pelo bit 0); os seus `ImageDescription` e `Software` identificam-na como uma máscara de limitação, e não contém qualquer calibração.
- **Bits**: bit 0 = a banda no canal R do produto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Um bit definido significa que o valor desse pixel nessa banda foi limitado a 0 — considere-o, bem como qualquer índice a partir dele, como de baixa confiança.
- **`Chloros:UnmixClampMask`** é gravado em todos os produtos que executaram a separação de cores: o caminho da máscara relativo à própria pasta do produto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre separado por `/` — junte-o à pasta da imagem para abrir a máscara), ou `none` quando nenhum pixel foi limitado. A ausência de etiqueta significa que o produto nunca passou pelo processo de separação de cores (câmaras RGB ou monocromáticas, níveis raw/sem debayering).
- `lattice capture` / `array-capture` imprimem o caminho da máscara no ficheiro guardado (`clamp mask: …`), e a lista de resultados da captura apresenta-a como o `clamp_mask` do fotograma.
- O `lattice index --live --save-multiband` grava uma máscara por câmara de origem para a pilha multibanda — ver [`lattice index` — máscaras de pilha multibanda](#lattice-index--multiband-stack-masks). Um vídeo não tem máscara: o `array-build-video` apresenta, em vez disso, as frações de limitação de cada fotograma (consulte [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Como nenhuma pasta de produto contém uma máscara, a qualquer nível, uma pasta de produto (`radiance/`, `reflectance/`, a pasta `*_Images/` de uma exportação `process`) pode ser entregue ao Pix4D, Metashape ou outro software de fotogrametria tal como está — incluindo subpastas — e este aceita apenas imagens. Envie a pasta do produto, não a pasta acima dela (essa também contém `clamp_masks/` e todos os outros tipos de exportação). A própria importação do Chloros ignora as máscaras. Interpretação de uma máscara: `(mask & 1) != 0` significa «a banda do canal R foi limitada aqui», `& 2` G, `& 4` B.

#### Dados EXIF transferidos para a exportação

`process` copia o **bloco GPS e o seu ExifIFD** da captura de origem para cada produto, pelo que uma
exportação inclui `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` juntamente com a georreferência.

**O `FocalLength` não é opcional para a fotogrametria.** O Pix4D calcula a distância amostral ao solo a partir da
distância focal mais a altitude; na ausência da etiqueta, recorre a uma escala extremamente errada. Num
voo sobre um pomar de laranjeiras com 49 capturas, a falta da etiqueta transformou um local de 411 m × 160 m numa reconstrução
de 47,8 km × 13 km — uma ortofotografia de 455 MP composta principalmente por «nodata», o que foi então interpretado como um problema de mosaico e
um problema de BigTIFF antes de alguém verificar a GSD. Se a sua ortofotografia apresentar uma escala implausível, execute primeiro o `exiftool -FocalLength` no produto exportado.

A cópia **não** é deliberadamente o `-all:all`: as etiquetas estruturais do IFD0'as suas etiquetas estruturais corrompem a saída do LATTICE quando
copiadas, e o `ExifImageWidth` / `ExifImageHeight` são excluídos porque descrevem a
captura *de origem* — uma exportação que alguma vez tenha sido redimensionada apresentaria, de outra forma, dimensões
contraditórias em relação à sua própria imagem rasterizada. O XMP é gravado diretamente em vez de ser copiado, porque o ExifTool
descarta as etiquetas XMP da mesma invocação quando o bloco XMP é copiado (o que eliminaria as etiquetas de calibração do MAPIR).

### Onde as saídas são guardadas

Os produtos são gravados **na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmara é `LATT-<sensor>-<lens>-F<filter>` para o LATTICE (correspondendo ao EXIF da captura
`Model`) e `<model>_<filter>` para a Survey3 — duas câmaras que partilham um sensor e um filtro, mas que diferem
na lente, mantêm árvores separadas, uma vez que a vinheta, o campo de visão e a distorção diferem. A pasta de formato
pasta segue o padrão `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` para
`TIFF (32-bit, Percent)`.

> **Todos os produtos exportados mantêm o nome do ficheiro ORIGINAL.** Uma exportação de radiação de
> `capture_…_raw.tif` continua a chamar-se `capture_…_raw.tif` — apenas se encontra em
> `tiff32/Radiance_Images/`. **É a pasta que identifica o produto, não o nome do ficheiro**, por isso, uma pesquisa global
> por `*radiance*.tif` não encontra nada; em vez disso, procure no diretório.

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

O `process` também processa as gravações `.daq` na sua pasta de entrada e **não**
precisa de quaisquer imagens para o fazer: um DAQ-U / DAQ-M / DAQ-E utilizado isoladamente constitui uma
captura completa, e uma pasta que contenha apenas ficheiros `.daq` é uma entrada válida.

Um DAQ pode ser gravado **sem** a sua calibração — é isso que os gravadores públicos
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fazem por predefinição: gravam as contagens brutas do sensor e marcam o ficheiro para que
o Chloros obtenha a calibração de fábrica desse sensor **por porta série** (primeiro na cache local,
depois na nuvem MAPIR) e a aplica. O `process` volta a gravar o resultado:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

O `.csv` contém uma linha por leitura: carimbo de data/hora UTC, tempo de integração, potência total,
lux fotópico/escotópico, PPFD (e a sua divisão em azul/verde/vermelho), comprimento de onda de pico e, em seguida, o
espectro completo na grelha de comprimentos de onda do próprio sensor. O `.daq` é reimportado sem ser
calibrado uma segunda vez.

O `<name>` é o nome do próprio ficheiro da gravação. Duas gravações em pastas diferentes podem partilhar
uma — leituras por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) guardadas no
mesmo milésimo de segundo por duas câmaras — e cada uma continua a obter os seus próprios produtos: a primeira mantém
`<name>`, a seguinte acrescenta as pastas que a distinguem **depois** do nome
(`<name>__<capture>_daq_calibrated.daq`), pelo que a data e a hora da gravação continuam a figurar no início.

Se for bem-sucedida, a execução apresenta o relatório `Light-sensor products written: N (calibrated .daq + .csv)`.
O texto entre parênteses descreve o que foi efetivamente gravado, pelo que se lê
`(RAW COUNTS — this sensor has no calibration bundle)` para um sensor sem pacote e
`(N calibrated, M raw counts)` para uma pasta que contém ambos. Os próprios títulos do backend,
`[DAQ-EXPORT]` e `[RUN-SUMMARY]`, derivam a sua redação da mesma forma — nenhum dos
três pode considerar uma exportação em bruto como calibrada.

Uma gravação DAQ-U / DAQ-M / DAQ-E cujo pacote de calibração não possa ser obtido — porque está
offline ou porque esse sensor não tem nenhuma calibração registada — é **ignorada com um motivo** numa
linha `[DAQ-EXPORT]`, nunca sendo gravada como um ficheiro «calibrado» contendo contagens brutas.
Ligue-se à Internet e volte a executar. O motivo é aquele que o leitor efetivamente
estabeleceu para esse ficheiro (esquema ilegível, ausência de pacote, um erro de gravação), e o resumo da
execução lista motivos **distintos** — vinte ficheiros ignorados por uma única causa são interpretados como uma
única causa, e não como vinte repetições da mesma. A CLI identifica cada gravação ignorada no final da
execução, com o respetivo motivo:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Uma pasta de gravações que não exporta **nenhuma** delas falha: ver
[Uma execução que não grava imagens falha](#a-run-that-writes-no-images-fails).

Uma gravação cujo nome termine em `_calibrated` ou `_raw` (`field_raw.daq`) **não é
importada**: esses são os nomes que o `process` atribui aos seus próprios produtos, e um produto lido
como uma gravação seria contado duas vezes. A CLI indica isso e identifica o ficheiro —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— em vez de reportar uma pasta vazia. Renomeie essa gravação antes de a processar.

#### As gravações DAQ-A são exportadas como contagens brutas

A família **DAQ-A** é anterior ao sistema de pacotes por número de série e não tem nenhum pacote de calibração
para obter — em vez disso, é calibrada no terreno em relação a um alvo de refletância, razão pela qual
nunca precisou de um. Recusar essas gravações deixava-as sem qualquer forma de
extrair os seus valores, pelo que são exportadas com um **nome diferente**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Um nome de ficheiro diferente, em vez de um indicador no interior do ficheiro, porque a informação tem de permanecer intacta
ao ser enviada por e-mail apenas como um nome simples. O cabeçalho `.csv` indica
`raw spectral sensor counts (NOT irradiance)` e avisa que os valores são comparáveis
**dentro** do ficheiro — que é exatamente para isso que a calibração baseada em alvo os utiliza — e
não entre sensores. As colunas fotométricas dependentes da potência (potência total, lux fotópico e
escotópico, PPFD) são registadas como **NULL** em vez de serem integradas a partir de contagens, e o resumo
da execução indica `RAW COUNTS`, pelo que «exportado» num registo não pode ser interpretado como irradiância.

As gravações antigas **v1.01 / v1.02** (um DAQ-A-SD grava estas) não incluem uma época por leitura,
apenas a hora de gravação do ficheiro. O comparador de imagens↔radiação descendente continua a rejeitá-las — comparar um
quadro com uma hora de gravação seria um erro imperceptível — mas o exportador lê-as, e
o ficheiro CSV apresenta `clock=daq_created_on`, pelo que o produto indica em que relógio se encontra.

### Notas

- O `process` deteta automaticamente se a sua pasta é do tipo Survey3, «LATTICE» ou mista.
- O progresso é transmitido através de «Server-Sent Events»; a CLI mostra o progresso em tempo real por thread (Detetando, Analisando, Processando, Exportando).
- Para Linux/Jetson, a CLI verifica a memória swap e pode emitir um aviso antes de processar pastas de grande dimensão. O debayer sensível à textura também aplica automaticamente um limite de frequência da GPU em Jetsons de baixo consumo (Nano, Orin Nano).
- Se for bem-sucedida, a execução indica quantos produtos de imagem foram gravados (`Image products written: N`, ou `(plus M clamp masks)` quando a separação de cores limitou algum pixel).
- **O `process` partilha um backend em execução de forma segura.** Reinicia automaticamente um backend já em execução apenas quando esse backend está inativo, com um projeto anterior carregado e sem câmaras, matrizes ou DAQs. Um backend que contenha qualquer um desses elementos (a GUI a transmitir uma matriz, um script SDK, uma gravação DAQ) é utilizado tal como está, nunca sendo reiniciado, pelo que o `process` pode ser executado em simultâneo com uma matriz ativa. Se o backend indicar que está a decorrer uma execução, o `process` aguarda até 15 s pela sua conclusão (uma execução que acabou de terminar demora cerca de um segundo a ser desativada), e, em seguida, rejeita com o código de erro `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. O sinalizador global `--restart` continua a forçar um backend novo e encerra qualquer sessão de câmara, matriz ou DAQ nele. Um backend executado por um serviço «systemd» do Linux nunca é reiniciado: o `process` utiliza-o, e o `--restart` apresenta o `sudo systemctl restart <unit>` em vez disso (ver [Executar o backend como um serviço](#running-the-backend-as-a-service-headless)). O mesmo se aplica ao backend próprio da aplicação Chloros: com a aplicação aberta, o `process` utiliza-o, carregando o seu projeto em vez do da aplicação (um backend iniciado no seu lugar bloquearia o acesso da aplicação aberta).
- **Uma execução em curso mantém o seu projeto.** Enquanto uma execução de processamento estiver em curso — iniciada pela aplicação, pelo `process`, por um script do SDK ou pelo `chloros-mcp` — o backend recusa-se a criar, abrir, importar para, adicionar ou remover ficheiros, mover, apagar ou alterar as definições de processamento de um projeto, com o código de erro HTTP 409 `PROCESSING_IN_PROGRESS` e `A processing run is in progress on project '<name>', so ... now is refused`; um `process` que consiga fazê-lo sai do `1`. As definições de visualização (tamanho das miniaturas) continuam a ser guardadas. O modo de processamento e as definições de refletância-alvo do projeto são recusados da mesma forma (uma execução mantém o modo em que foi iniciada). Uma captura guardada no projeto a meio da execução (a partir da aplicação, da CLI, do SDK ou do `chloros-mcp`) é guardada como habitualmente, mas só é adicionada à lista de ficheiros do projeto quando a execução termina; a resposta indica isso (`project_add`), tal como acontece com um registo do sensor de luz interrompido no projeto. Anteriormente, um segundo cliente podia alternar para o projeto aberto a meio da execução, e a execução em curso era exportada para o outro projeto.
- **Ctrl+C interrompe a execução no backend, não apenas na CLI.** A CLI solicita ao backend que interrompa o processamento, exibe `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` e aguarda até 120 s pelo `Processing stopped.`, para que o `chloros-cli process … && next` não inicie o `next` num backend ainda ocupado com a execução. Um segundo Ctrl+C encerra a espera. O `process` encerra então o `130`, tal como qualquer comando interrompido faz.

#### Uma execução que não grava imagens falha

Se solicitou produtos e a execução não gravou **nenhum** — apenas o `project.json` e
o `calibration_data.json` — o `process` trata isso como uma falha: apresenta
`Processing finished but wrote no image products.` e **termina com um valor diferente de zero**, pelo que um script pode
detetá-lo. A mensagem indica o nome da pasta do projeto e as causas habituais:

- a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou
- todos os produtos solicitados foram ignorados por serem inaplicáveis a essas câmaras (por exemplo, solicitar
  radianância/refletância de câmaras apenas RGB).

Execute novamente com o `--verbose` e verifique o registo do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explicam as omissões por câmara que, de outra forma, não chegariam à saída da CLI.

Uma execução deliberada apenas com metadados — todos os produtos desativados e sem `--indices` — continua a ser um
**sucesso**, porque uma saída de imagem vazia é o resultado correto nesse caso.

#### Uma execução cujas exportações não puderam ser verificadas falha

Após uma execução, o `process` verifica as etiquetas dos ficheiros exportados com o ExifTool. Se o ExifTool não puder ser executado,
exibe `Verificação da exportação FALHOU: O ExifTool não pôde ser executado, pelo que as etiquetas de N de M ficheiros exportados
não foram verificadas (consulte as linhas [EXIFTOOL] no registo do backend)` and `Os produtos são gravados e permanecem
no local; execute novamente para os verificar.`, does not print `Processamento concluído!`, e **termina com um valor diferente de zero**. Os
produtos são mantidos.

#### Quais as capturas que uma execução inclui

As capturas concluídas antes da execução do `process` são incluídas. As capturas guardadas durante a execução não são; o `process`
avisa quantas foram excluídas, e executá-lo novamente inclui-as.

#### Uma execução que perdeu uma imagem falha

Se uma imagem falhar durante uma execução, o `process` apresenta o `N image(s) failed and were not exported` com a
causa mais comum, não apresenta o `Processing complete!` e **termina com um valor diferente de zero**. Os outros produtos são
gravados e permanecem no local, e o registo do backend contém cada falha.

O mesmo acontece com uma **execução apenas com o sensor de luz**: uma pasta de gravações `.daq` não contém imagens para exportar
por definição, e a execução é avaliada com base nos ficheiros `.daq` / `.csv` calibrados que escreveu em vez disso.
Se não gravou **nenhuma** — todas as gravações foram ignoradas —, apresenta
`Processing finished but wrote no light-sensor products.`, nomeia cada gravação com o
motivo pelo qual foi ignorada e **termina com um valor diferente de zero**.

---

## `chloros-cli login`

Autenticar esta máquina com uma conta na nuvem Chloros+. As credenciais são armazenadas em cache de forma segura em `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (removendo partes da palavra-passe ou duplicando-as). Em caso de erro 401, a CLI tenta automaticamente novamente com `$$` re, e depois com metade da palavra-passe sem duplicações; se uma nova tentativa for bem-sucedida, o sistema inicia a sessão e apresenta a sintaxe correta com aspas simples a utilizar na próxima vez.

> **Utilização sem interface gráfica/por script: a ausência de uma sessão em cache implica um prompt de início de sessão.** Um comando principal (`process`, `status`, `export-status`, `time-sync`, …) executado sem uma sessão em cache solicita `Email:` / `Password:` na entrada padrão (stdin) antes de prosseguir. Com o stdin fechado (cron, uma unidade do systemd), falha imediatamente com o erro `login cancelled`; com o stdin num terminal aberto ou num pipe, aguarda a entrada de dados. Execute o `chloros-cli login EMAIL PASSWORD` uma vez por máquina antes de agendar tarefas sem interface gráfica.

---

## `chloros-cli logout`

Limpa a sessão armazenada em cache e força um novo início de sessão na próxima chamada.

Sair enquanto uma execução de processamento estiver em curso não a interrompe: a execução termina no modo de processamento em que foi iniciada e a próxima execução utiliza o processamento padrão. O código `logout` indica isso (`The processing run in progress finishes in <mode> mode; the next run uses standard`), tal como a aplicação quando se sai a sessão nesse local.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Mostra o nível de licença atual (Iron/Copper/Bronze/Silver/Gold), o utilizador autenticado e o número de ligações de dispositivos.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifica o progresso em tempo real da exportação do Thread-4. É seguro chamar **durante** a execução de um `process` a partir de outro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Define o idioma de exibição da CLI (38 idiomas suportados, incluindo CJK, RTL e Indic). Recorre ao inglês de forma harmoniosa em consolas antigas que não conseguem apresentar o script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemplos

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandos da pasta de projetos

Estes comandos gerem a localização predefinida da pasta de projetos (partilhada com a GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` apresenta a pasta para onde os novos projetos são guardados. Se a pasta configurada já não existir (foi eliminada, renomeada ou encontra-se numa unidade que não está ligada), a CLI, tal como a GUI, recorre à pasta predefinida: a pasta predefinida é apresentada na saída padrão (stdout), um aviso indicando o nome da pasta em falta é enviado para a saída de erro (stderr) e o código de saída é `0`.

---

## `chloros-cli update`

Apenas Linux/Jetson. Verifica se existe o `version_url` a partir do `/etc/chloros/update.conf` e, caso exista uma versão mais recente, descarrega o `.deb` correspondente e instala-o com o `sudo dpkg -i` imediatamente (sem pedido de confirmação; o `sudo` poderá solicitar a sua palavra-passe). Num Jetson, a atualização é oferecida apenas a um sistema JetPack 6. No Windows, o comando não faz nada e apresenta a mensagem `up to date`: atualize com um novo instalador.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

No Linux/Jetson, a CLI também executa uma **verificação automática de atualizações em cada arranque** (sem bloqueio, nunca atrasa o comando): lê `/etc/chloros/update.conf`, armazena o resultado em cache durante 1 hora em `~/.chloros/update_cache.json` e apresenta `Update available: vX.Y.Z / Run: chloros-cli update` quando existe uma versão mais recente. Ignora silenciosamente qualquer erro e no Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` desativa esta funcionalidade. Devido ao cache de 1 hora, uma versão publicada dentro dessa hora pode ainda ser considerada atualizada.

---

## `chloros-cli selftest`

Executa um teste de funcionamento de 7 etapas: versão, disponibilidade de portas, arranque do backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presença do modelo de redução de ruído, prontidão de CUDA + denoiser. Utiliza um backend em execução ou inicia um para o teste, parando-o posteriormente. Exibe `N/7 checks passed` e as verificações que falharam, e só sai de `0` quando todas as sete forem aprovadas; a verificação de CUDA + denoiser falha num computador sem uma GPU CUDA (ver [Verificar a instalação](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Estado e controlo do «grandmaster» PTP. O anfitrião Chloros executa o «grandmaster» PTP; as câmaras LATTICE e as unidades DAQ-E funcionam como escravas deste para obter carimbos de data/hora entre dispositivos (cerca de 1 ms entre dispositivos). O PTP alinha **carimbos de data/hora**: as câmaras de um conjunto expõem-se em simultâneo porque o disparador de hardware M8 as aciona (em menos de 100 µs entre módulos), e não devido ao PTP. O comando `chloros-cli time-sync`, sem subcomando, executa o comando `status`.

| Subcomando | Descrição |
| --- | --- |
| `status` | Mostrar o estado do grandmaster, as prioridades BMCA, a identidade do relógio e se os seus pacotes estão a chegar à rede: o `health` é o `OK` (`STOPPED` quando o «grandmaster» não está a funcionar), ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` com um `health_detail`, e, em seguida, é impressa uma linha `WARNING:` no stderr (o stdout mantém-se em JSON). `sync_sent` / `announce_sent` contam apenas as mensagens que foram enviadas; `*_send_errors` e `last_send_error` contam as que não foram. Após uma ligação de câmara ser interrompida e restabelecida, o «grandmaster» reconstrói os seus sockets automaticamente (`rebinds`, `last_rebind`). |
| `peers` | Lista os escravos detetados através de Delay_Req (câmaras + sensores DAQ-E). |
| `cameras` | Estado do PTP por câmara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar o processo do grandmaster. |
| `set-priority --priority1 N --priority2 N` | Substituir as prioridades do BMCA. |

O grandmaster anuncia nas placas de rede das câmaras (qualquer adaptador com um endereço `169.254.x.x` incluído). Se nenhum estiver ativo, recorre ao adaptador da rota predefinida apenas quando este estiver ligado por cabo, nunca por Wi-Fi ou a um adaptador virtual (Tailscale, WSL), uma vez que nenhuma câmara ou DAQ-E se encontra por trás de um. Sem nada para anunciar, funciona sem sockets (`health` `NO_SOCKETS`), e a próxima câmara ou DAQ a ligar-se reconstrói-o na placa de rede que estiver ativa nessa altura; o `time-sync restart` faz isso de imediato.

O «grandmaster» funciona num processo próprio, ao lado do backend (`own_process` / `process_pid` em `time-sync status`), com alta prioridade no Windows (`process_priority`), para que um backend ocupado (uma execução de processamento, uma exportação, vários comandos em simultâneo) não possa atrasar os tempos que regista; anteriormente, um atraso desse tipo podia alterar o relógio de todas as câmaras de uma só vez. Esse processo termina com o backend; se não for possível iniciá-lo, o «grandmaster» é executado dentro do backend, tal como antes (`own_process` false, e o registo do backend indica isso).

### Exemplos

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controlo de câmaras LATTICE. Todos os subcomandos passam pelo backend Chloros; o backend é responsável pelo conjunto de câmaras, pelo que as chamadas CLI subsequentes reutilizam o mesmo identificador aberto.

### Opções comuns (partilhadas pela maioria dos subcomandos)

| Sinalizador | Descrição |
| --- | --- |
| `-d, --device N` | Índice da câmara (predefinição: 0). |
| `-s, --serial SN` | Número de série específico; substitui `--device`. |
| `--serials SN1,SN2,…` | Números de série separados por vírgulas para operação com várias câmaras. Os [comandos transitórios para várias câmaras](#multi-camera-transient-sessions) abrem apenas estas câmaras, sendo que a primeira aciona o disparador. |
| `--all` | Operar em todas as câmaras detetadas. |
| `--exposure US` | Tempo de exposição em microssegundos. |
| `--gain DB` | Ganho em dB. |
| `--pixel-format FMT` | por exemplo, `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` num M3M. O nome define a profundidade de bits na própria família da câmara: num M3M, um nome «Bayer» define o formato «Mono» da sua profundidade (`BayerRG8` define `Mono8`); num M3C, um nome «Mono» define o «Bayer» (`Mono12` define `BayerRG12`). |
| `--width N` / `--height N` | Dimensões da imagem. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplicar uma predefinição de configurações: o seu formato de píxeis, exposição, ganho e taxa de fotogramas (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` lista-as). Num M3M, o formato é o Mono com a mesma profundidade: o `high_speed` captura o `Mono8`, `high_quality` e `triggered` `Mono12`. Um sinalizador fornecido com ele (`--exposure`, `--gain`, `--pixel-format`) prevalece sobre a predefinição, e uma configuração que a predefinição não especifique mantém a configuração própria da câmara. Todas as execuções, exceto `triggered`, que arma a câmara para uma variação de nível na Linha 2 — sem nada a acionar essa linha, a câmara ficará à espera indefinidamente em vez de capturar. |
| `-o, --output DIR` | Diretório de saída (padrão: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamanho do pacote GVSP. `auto` executa sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxo de trabalho da primeira ligação da câmara LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referência de subcomandos

#### Detecção e informação

| Subcomando | Finalidade |
| --- | --- |
| `lattice info` | Listar câmaras ligadas (fabricante, modelo, número de série, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analisa o sistema anfitrião para obter a configuração ideal da câmara. O `--no-discover` ignora a deteção da câmara (mais rápido, análise apenas pela placa de rede). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Verificar/corrigir as definições da placa de rede (no Linux, também os buffers de receção de sockets do kernel — ver [hosts Linux](#linux-hosts--socket-receive-buffers)); estimar largura de banda/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Recomendação de capacidade de rede do backend com esquema estável+ recomendação de matriz (retorna `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` assumem por predefinição o sensor completo, `2048` × `1536`. Termina em `0` para `ok`, `auto_capped_fps` e `auto_shrunk`, e `1` para `needs_force_slip` e `error`, com ou sem o `--json`. Chega ao backend no `--backend-url` (padrão `http://127.0.0.1:5000`), e não no `CHLOROS_BACKEND_URL`. O `auto_capped_fps` mantém a resolução solicitada, mas limita os fps de destino — leia o `recommended.recommended_target_fps` e passe-o como destino da ligação; considere-o um sucesso, não como um erro. O `auto_shrunk` só aparece quando o fotograma solicitado não pode ser entregue a qualquer taxa de fotogramas; o connect nunca reduz os fotogramas por iniciativa própria. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Análise hipotética sem abrir câmaras (`--width` / `--height` predefinição `2048` × `1536`). Tanto esta como a `network-analysis` projetam o que o «connect» irá transmitir: sem a `--pixel-format`, aplicam-se os formatos por câmara que o «connect» utiliza (Mono12 / BayerRG12), e com o `--packed auto` (padrão), a decisão que o «connect» tomaria neste computador [transporte de 12 bits compactado---compactado](#transportado-12-bits-compactado---compactado) que o Connect tomaria neste computador — pelo que os fps indicados correspondem à taxa a que a matriz funciona. Ambos apresentam a decisão de compactação. **`--n-active` é o número total de câmaras na rede, não apenas as desta matriz**— aumente-o quando câmaras autónomas transmitirem em simultâneo, ou quando a capacidade da ligação for calculada com base numa procura que as subestime (padrão: `len(--models)`). Exibe sempre as linhas agregadas `Wire budget:` (MB/s exigidos vs. limite máximo seguro contra colisões) e `Max cameras:`, e sinaliza `** OVER-SUBSCRIBED**` quando a matriz excede a capacidade da ligação — consulte [Modelo de fps e burst da matriz](#array-fps--burst-model). |
| `lattice gpu` | Mostrar o estado da GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verificar ou atualizar o firmware da câmara. A seleção local do `.fwa` está fixada: o ficheiro no `firmware/<MODEL_PREFIX>/` que corresponde ao `MIN_FIRMWARE_VERSION` da compilação é gravado quando presente (apenas a versão mais recente como alternativa), pelo que uma imagem do fornecedor mais recente armazenada no disco permanece inativa até que esse pino seja alterado — as versões mais recentes chegam deliberadamente às unidades através do manifesto AWS assinado, o que é preferível quando se trata de versões mais recentes. |
| `lattice presets [--apply NAME]` | Listar ou aplicar predefinições da câmara. `--apply` define o que os nomes das predefinições representam (o seu formato de píxeis, exposição, ganho, taxa de fotogramas, disparador) e apresenta essas definições tal como a câmara as lê posteriormente: num M3M, o formato Mono da profundidade da predefinição; a taxa de fotogramas a que funciona, que pode ser inferior à solicitada pela predefinição (o `high_speed` solicita 60 fps; uma câmara full-frame numa ligação de 1 Gbps funciona a cerca de 35). Todas as outras definições que o comando «open» escreveu são repostas tal como foram encontradas. O comando «`--apply default`» repõe a câmara para os valores predefinidos de fábrica: as definições de fábrica com exposição automática em modo livre na Linha 2, o limite de largura de banda da ligação e a correção de defeitos desativadas, mantendo-se o equilíbrio de brancos calibrado de uma câmara RGB (unidade em qualquer outra). Nada é guardado na memória da câmara, pelo que, na próxima vez que for ligada, inicia a partir da sua própria memória, tal como antes. Como `--preset` de uma captura ou ligação, o `default` não é uma predefinição. |
| `lattice status` | Estado da câmara em tempo real. |

#### Captura

| Subcomando | Finalidade |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Fotograma único. **Guarda todos os tipos de exportação por predefinição**(`--processing all`); consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). O comando `--processing` aceita os subcomandos `all`, `raw`, `debayered`, `radiance`, `reflectance` ou `display` (`preview` é aceite para `display`); qualquer outro nome é recusado antes de a câmara ser tocada (sair de `1`). `--levels` guarda um subconjunto explícito (substitui `--processing`). Uma câmara que o Chloros não possui é aberta para a captura, e a sua exposição é definida a partir da cena inicialmente (no máximo alguns segundos), tal como fazem os [comandos transitórios](#multi-camera-transient-sessions); quando a captura termina, independentemente da forma como termina, essa câmara é reposto no estado em que foi encontrada (formato, exposição, pipeline de cor, todas as definições alteradas pela captura — a lista em [Multicâmara (Sessões Transitórias)](#multi-camera-transient-sessions)). A leitura do DAQ utilizada na captura é guardada como um ficheiro sidecar `.daq`, incluindo uma captura apenas em formato bruto; `--no-daq` / `--daq-tolerance-ms` ajustam-na ([O DAQ que lê uma captura guarda](#the-daq-reading-a-capture-saves)). O `--force-daq` continua a ser aceite. `--jpeg-quality` = qualidade JPEG 1–100 (padrão 95).**Os ficheiros guardados no projeto aberto são adicionados a este.** Tal como na aplicação, uma captura cujos ficheiros sejam guardados na pasta do projeto aberto no Chloros adiciona-os à lista de ficheiros desse projeto; os ficheiros guardados noutro local não são afetados. O comando apresenta uma linha a indicar isso (`Project: ...`) ou, enquanto uma execução de processamento estiver em curso, que estão a ser retidos e serão incorporados quando a execução terminar. O mesmo acontece com `multi-capture`, `array-capture` (`--smart` também) e `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Transfere para o disco até premir Ctrl+C. Um processo `tiff`são o ficheiro `raw`, um conjunto de dados que a captura escreve dessa câmara: o mosaico do sensor, um plano com os seus próprios valores, não comprimido, com o pacote de calibração (um quadro de 2048 x 1536 a 12 bits tem 6,0 MiB mais o pacote, independentemente da cena), pelo que o `process` parte desses dados. `png` e `jpg` contêm 8 bits, pelo que essas sequências são fotogramas desmosaicados para visualização, dos quais o `process` não parte. Com um DAQ vinculado (`daq pool-bind`) ou agrupada, cada fotograma recebe os seus próprios `.daq` e `.match.json` na pasta `daq/` da sequência, uma vez que o `lattice capture` guarda um; o `--no-daq` desativa essa funcionalidade e, ao parar, indica quantos. Cada fotograma é gravado uma vez, com os seus metadados completos; ao premir Ctrl+C, os fotogramas já capturados são gravados antes de a câmara ser libertada. Interrompe-se automaticamente quando o disco fica cheio, imprime `Failed:` / `Disk full:` e sai com o nome `1`; qualquer fotograma que não tenha sido possível gravar também gera o ficheiro `1`. Uma câmara deixada armada para um disparador de hardware (`TriggerMode` `On`) é configurada para funcionamento livre durante a sessão. Quando a sessão termina, a câmara é reposto no estado em que foi encontrada — as suas definições de disparo e todas as definições alteradas durante a sessão (exposição, ganho, formato, …) — a menos que seja executado o comando `--keep-settings`, que guarda as definições da sessão na própria memória da câmara (funcionamento livre), ou o comando `--clear-settings`, que repõe essa memória para as definições de fábrica. Uma execução que não grava nenhum fotograma é considerada uma falha: se não houver nenhum fotograma durante 10 s, ela interrompe-se automaticamente, imprime `No frames:` com o motivo e sai com o código `1`. Os fotogramas que os gravadores não conseguiram acompanhar são contabilizados como perdidos, e a paragem imprime `Dropped:` com o motivo (a taxa de fotogramas da câmara, dos gravadores e da fila); uma taxa de fotogramas mais baixa ou um formato de píxeis de 8 bits permite guardar todos. O comando verifica o backend a cada 2 s; se falhar sem interromper a gravação (`kill -9`, falta de memória), o backend interrompe-a 30 s após a última verificação e coloca a câmara de volta (`stop_reason` `client_gone`, e uma linha no registo do backend). `multi-stream` e `align-stream` fazem o mesmo. Um arranque em que a câmara demora mais tempo a abrir do que o tempo de espera do comando (60 s; 180 s para `multi-stream` / `align-stream`) — uma câmara ainda a arrancar — é anulado pelo backend: a câmara é recolocada e encerrada e nenhuma gravação fica em execução (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Pré-visualização MJPEG ao vivo no navegador: liga-se à câmara, apresenta o URL do fluxo e abre-o no seu navegador. `--ae-damping` define o amortecimento da exposição automática (0,4–100). |

##### Transmissões em direto (MJPEG; sem RTSP)

O Chloros transmite vídeo em direto como **MJPEG sobre HTTP**(`multipart/x-mixed-replace`, limite `frame`) a partir do backend.**O RTSP não é suportado.** Abra um URL num navegador ou leia-o com qualquer cliente MJPEG (`cv2.VideoCapture(url)` do OpenCV, ffmpeg, VLC). A câmara deve ser ligada primeiro no backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` ou na GUI).

| URL (`http://127.0.0.1:5000` + …) | Fluxo |
| --- | --- |
| `/api/camera/stream/<serial>` | A pré-visualização do ecrã, até cerca de 30 fps. Para um elemento da matriz, `?max_h=` (padrão: `720`) e `?jpeg_q=` (padrão: `80`) definem a altura e a qualidade JPEG. Esta é a URL que o `lattice viewer` abre. |
| `/api/camera/<serial>/stream-annotated` | A mesma pré-visualização com as sobreposições da GUI desenhadas em: `zebra=1`, `crosshair=1`, `grid=3x3` (ou `9x9`), `histogram=1`, `peaking=1`, `spot=1`; sem qualquer solicitação, o projeto utiliza as definições de sobreposição guardadas para essa câmara. |
| `/api/camera/<serial>/stream-reflectance` | Refletância de 8 bits por banda: ρ = pixel ÷ a parte`X-Chloros-Pixel-Scale` (255, a menos que `refl_max` seja acionado). `fps` (padrão `10`), `jpeg_q` (padrão: `95`), `max_h` (padrão: `0` = nativo), `band=<name>`, `align=1`. JPEG, portanto com perdas. Recusado com código HTTP 409 quando a câmara não apresenta refletância (uma câmara RGB, sem pacote de calibração, sem leitura DAQ). |

- **Correspondência de fotogramas entre câmaras.**Cada parte que um**membro da matriz** envia, nos três fluxos, contém os cabeçalhos `X-Chloros-Group-Id` (o contador do grupo sincronizado da matriz), os cabeçalhos `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Partes de câmaras diferentes com o mesmo `X-Chloros-Group-Id` provêm do mesmo disparador. Uma câmara autónoma não tem grupo e não envia nenhum identificador de grupo. Os navegadores ignoram os cabeçalhos de cada parte; um cliente que analisa ele próprio o fluxo multiparte lê-os.
- **Em pausa enquanto a vista de matriz estiver aberta.** O `/api/camera/stream/<serial>` de um membro da matriz envia um fotograma estático «em pausa» enquanto o fluxo ao vivo combinado da matriz estiver aberto (por exemplo, a vista de matriz da GUI) e retoma quando este for fechado.

#### Afinação do sensor

| Subcomando | Finalidade |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Ler/gravar qualquer nó GenICam. `--set ExposureTime=…` (`Gain=…`) sem os seus comutadores `ExposureAuto` (`GainAuto`) que desativam primeiro o modo automático, tal como o `lattice exposure --set` faz; um valor que a câmara recusa imprime `Failed to set <node>: …` no stderr e sai do `1`. O `--get` e a visualização agrupada por predefinição apresentam um `note:` em `ExposureTime` / `Gain` enquanto esse modo automático estiver em execução — consulte [Leitura da exposição e do ganho no modo automático](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Exposição e AE. Uma exposição `--set` ou um alvo `--brightness` é mantido como o seu (o controlo automático de exposição do Chloros não o altera); o ganho permanece como estava, pelo que, no modo automático, continua a ajustar-se em direção ao alvo até definir o `lattice gain`. O `--auto` devolve o controlo da exposição, enquanto o `--brightness-auto` devolve o alvo ao sistema automático do Chloros. Sem sinalizadores, lê os nós de exposição, com o mesmo `note:` que o `configure --get`. Se a câmara recusar um valor `--set`, imprime `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` no stderr — no modo automático, a escrita desativada é restaurada em vez de permanecer desativada — não imprime nenhuma linha `Exposure: … us (auto OFF)` e sai do `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Ganho e ganho automático. Um valor `--set` que a câmara rejeita imprime `Failed to set Gain: …` no stderr, não imprime a linha `Gain: … dB (auto OFF)` e sai do `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | ROI do sensor e binning. O binning é sempre **Média** — ver abaixo. |
| `lattice format [--set FMT] [--list]` | Formato de píxeis. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Disparador de hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sem sinalizadores = equilíbrio de brancos de disparo único) | Operações de equilíbrio de brancos. Apenas câmaras RGB/Bayer; uma operação nula (ignorada) em M3M monocromáticas. O one-shot mantém o equilíbrio de brancos que calcula (`BalanceWhiteAuto` e os ganhos); todas as outras definições cuja escrita aberta é efetuada são repostas tal como encontradas, e todas elas quando o one-shot não puder ser executado. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Pipeline de cor de exibição RGB. `natural` (predefinição) é o acabamento em tempo real mais simples; `enhanced` adiciona remoção de franjas + vibração + contraste local CLAHE para o aspeto completo de paridade de hub a cerca de 2× o custo de processamento por fotograma, o que resulta numa taxa de fotogramas **em tempo real** mais baixa — as capturas guardadas obtêm sempre o processamento completo, independentemente do caso. Apenas câmaras RGB/Bayer; ignorado em M3M monocromático. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturação/contraste da pré-visualização em tempo real (câmaras com filtro RGB); `100` é neutro, `--reset` define ambos para 100. As capturas guardadas não são afetadas. Ignorado na M3M monocromática. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (sem sinalizadores = ler) | Exibir **Destaques** (câmaras com filtro RGB): atenua os píxeis mais brilhantes e mais brancos — por exemplo, uma superfície branca brilhante que é representada como uma mancha branca plana. `0` = desativado, `-100` = mais forte. A pré-visualização em tempo real e as capturas ao nível do ecrã seguem esta configuração; a radiação/refletância nunca o fazem, e não é possível recuperar dados do sensor que tenham sofrido clipping (reduza a exposição para isso). Ignorado no M3M monocromático. |
| `lattice filter [--list]` | Lista os modelos de filtro que o Chloros conhece (`RGN-IMX265`, `OCN`, `NGB`, …). O filtro de uma câmara é lido a partir da sua cadeia de caracteres do modelo e não pode ser definido: o `--set NAME` apenas procura um modelo e, em seguida, sai do `1` com o `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Deteta os nós de alimentação/térmicos; alterna para o modo de inatividade de baixo consumo. `--sleep` deixa a câmara em modo de suspensão (o limite de largura de banda da ligação ativado (`DeviceLinkThroughputLimitMode` `On`) no seu valor mínimo, o modo de poupança de energia ativado, caso a câmara o possua) com todas as outras definições a sua configuração inicial a ser reposto tal como se encontrava inicialmente; um modo de suspensão que falhe é reposto na íntegra. O despertar repõe a ligação tal como estava quando a suspensão a encontrou (o limite, o seu modo e o atraso de pacotes `GevSCPD`, que a câmara mantém em 0 enquanto o limite estiver ativado; o registo é mantido neste computador, para que um despertar após o reinício do backend também o encontre), ou, no caso de uma câmara que este computador não colocou em modo de suspensão mas que parece estar em suspensão (com o limite ativado no valor mínimo), com o limite desativado e um `GevSCPD` mantido em 0 a 80 (as predefinições de fábrica); um limite definido acima do mínimo é mantido tal como está. A sonda e o despertar repõem o que a sua abertura escreveu. |

> **O binning é sempre Média.** O `lattice resolution --binning N` define o binning Média e
> não oferece uma opção de mudança de modo. O binning de Soma multiplica cada DN por N², custando aproximadamente dois stops de
> margem de realces por passo de binning; o Chloros divide esse fator novamente antes da
> cadeia radiométrica, mas os píxeis que a Soma levou até ao teto do sensor são cortados e não podem ser
> recuperados. Não traz nada que a Média não traga — um índice é invariante em relação à escala e os escalares de radiação
> são calibrados na escala da Média. Se uma câmara chegar já configurada para Somar (deixada
> assim por outra ferramenta ou por uma reposição de fábrica), o Chloros deteta isso a partir de uma leitura do nó e
> compensa; a execução do `lattice resolution --binning N` volta a colocá-la em «Average». A versão 1.2.0
> `--binning-mode` ainda é aceite, pelo que os scripts mais antigos funcionam, mas apenas exibe um aviso:
> o binning permanece em «Média».

> **Alterar o agrupamento, o formato de píxeis e o disparador numa câmara não tem qualquer efeito.** Numa câmara em que o
> backend permanece aberto e transmite dados (`cam-connect`, a GUI), o `lattice resolution --binning`
> não surtem efeito (exibe que o binning continua em vigor, `Binning: 1x1`), e
> `lattice format --set` e `lattice trigger --mode` são rejeitados pela câmara
> (`SC_ERR_ERROR -1001`). Liberte-o primeiro (`lattice cam-disconnect -s <SN>`) ou defina-os
> no momento da ligação (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Leitura da exposição e do ganho no modo automático

Enquanto o `ExposureAuto` (`GainAuto`) de uma câmara estiver em execução, o seu nó `ExposureTime` (`Gain`) mantém o **último valor gravado**, e não o que a câmara está a fazer neste momento. Para uma câmara cujo backend se mantém aberto (`cam-connect`, `array-connect`, a GUI), o `lattice configure --get`, a vista predefinida `lattice configure` e o `lattice exposure` (sem sinalizadores) apresentam, portanto, o valor do nó e, abaixo deste, uma nota com o quadro mais recente da câmara:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Leia o valor do quadro a partir da nota; o valor do nó corresponde apenas à última gravação. Através de HTTP, a mesma resposta `POST /api/camera/<serial>/read-nodes` mantém `nodes` inalterado e adiciona `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, do quadro mais recente) e `notes` (a frase acima). Ambos estão presentes apenas enquanto o modo automático não for `Off` e se conhecer um quadro; uma câmara aberta apenas para leitura não tem quadro e não obtém nenhum dos dois. `lattice gain` (sem sinalizadores) imprime apenas o nó — utilize `lattice configure --get Gain` para a nota.

##### Exposição num membro da matriz

Enquanto uma matriz transmite, cada membro funciona com **`AcquisitionFrameRateEnable`**, a uma taxa de fotogramas ligeiramente superior à taxa de disparo da matriz: o seu período é o período de disparo menos 15 % (pelo menos 3 ms), ou a taxa máxima da própria câmara quando o disparo já estiver próximo dessa taxa. `configure --get` num membro, portanto, lê `AcquisitionFrameRateEnable` como `True` e `AcquisitionFrameRate` nessa configuração; a ativação é desativada quando a matriz pára.**Não escreva estes dois nós num membro da matriz** — a matriz mantém-nos sincronizados com o seu disparador e reescreve-os.

Essa configuração é também o limite máximo de exposição do membro: cerca de 85 ms com um disparador de 10 fps (período de 100 ms), em **qualquer ROI ou binning**. Antes da versão 1.3.0, uma ROI pequena limitava a exposição de um membro ao período de funcionamento livre do sensor (4,9 ms a 768 × 304), independentemente da lentidão com que a matriz fosse acionada.**Um `ExposureTime` manual com duração superior ao limite máximo abranda o disparador para se adequar a esse valor** quando é gravado com `lattice exposure --set` ou `configure --set` (ou a partir da GUI). A matriz abre primeiro o limite máximo desse membro para o valor e permite que a taxa mais lenta funcione durante alguns períodos de disparo, pelo que a gravação pode demorar mais alguns segundos; o disparo ocorre então a uma velocidade não superior à permitida pela exposição. Numa matriz com taxa automática, a taxa a que funcionava anteriormente é restabelecida assim que todos os membros cuja exposição manual a tinha abrandado voltarem à exposição automática. Um valor que a própria câmara não possa aceitar continua a ser recusado, com o intervalo que aceita indicado no erro, e a exposição automática do membro é restabelecida se tiver sido desativada apenas para essa gravação. Uma receita de exposição `apply` não abranda o disparador: este é mantido no limite máximo. O Chloros nunca encurta uma exposição manual para se adequar a um disparador mais rápido; uma taxa fixada (`array-connect --target-fps F`) mais rápida do que a exposição permite deixa esse membro com um limite máximo suficientemente longo para ela, e esse membro não dispara a uma velocidade superior à permitida pela sua exposição.

#### Calibração e Sensores

| Subcomando | Finalidade |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de um alvo de refletância. São pesquisados até `--attempts` fotogramas (padrão `30`) para o mesmo; um quadro incompleto (perda de pacotes GVSP) conta como uma tentativa, não como uma execução. Falha com `no target detected in N attempts`, ou com `no complete frame in N attempts: <last error>` quando nenhum quadro chega completo. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | O espectro e as magnitudes de banda de um DAQ-M, lidos via Bluetooth (BLE) através do backend. `--mac` é o próprio MAC BLE do sensor e é necessário para qualquer uma das três ações: não existe um valor predefinido, uma vez que cada unidade tem o seu próprio (`daq pool-discover --only ble` lista-os); sem ele, o comando sai com o código de erro `1`. Se nenhuma ação for selecionada, exibe a ajuda. `--camera SERIAL` integra as magnitudes de banda exibidas através da resposta espectral medida pela própria câmara, em vez da curva de frota do modelo de filtro — câmaras RGB, com o pacote de calibração da câmara carregado (ligado) ou armazenado em cache neste computador. O título indica qual a curva utilizada: `Band magnitudes (camera <SERIAL>):`, ou `Band magnitudes (model <filter>):` quando a câmara não possui nenhuma (não é uma câmara RGB, ou sem pacote carregado ou armazenado em cache) e, em vez disso, foi utilizada a curva padrão. Através de HTTP, o `POST /api/dls/cli-read` recebe o `camera_serial` e devolve o `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplicar correção de vinheta a imagens existentes, cada uma com o campo plano da sua própria lente (a partir do seu Modelo EXIF). O `--lens-model` aplica-se a imagens cujo Modelo não indique nenhuma câmara; uma imagem cujo Modelo contradiga isso é ignorada. |

#### Multicâmara (Sessões Transitórias)

Um comando transitório abre as câmaras para uma execução e fecha-as quando esta termina: `multi-capture` sem nenhuma matriz persistente ligada, `multi-stream` e `multi-test` aqui, e `align-calibrate`, `align-apply`, `align-stream` e `index --live` abaixo. Estas regras aplicam-se a todas elas:

- **O `--serials SN1,SN2,…` seleciona as câmaras.** Apenas essas são abertas, e a primeira aciona o disparador (para `align-calibrate` e `--reference`, quando especificadas). Uma câmara serial que não esteja na rede faz com que a execução falhe antes de qualquer câmara ser aberta. Sem o `--serials`, todas as câmaras na rede são abertas. Uma câmara que o Chloros já detém (a GUI, `cam-connect`, `array-connect`) recusa a execução, quer o `--serials` a identifique ou não: ela partilha a rede para a qual a execução está configurada. (O `index --live`, sem o `--serials`, abre, em vez disso, as câmaras ativadas do seu perfil: ver [Índice / Matemática da Vegetação](#index--vegetation-maths).)
- **As câmaras partilham a rede da mesma forma que o `array-connect` a partilha.** Cada câmara é sincronizada com a sua quota-parte do tráfego transportado pela ligação do anfitrião (um atraso entre pacotes, não o limite de largura de banda da câmara, cujo limite mínimo de 31,25 MB/s já é excedido por quatro câmaras numa ligação de 1 GbE), pelo que quatro câmaras funcionam num link de 1 Gbps. Quando as câmaras não conseguem enviar todas ao mesmo tempo ao ritmo mais lento que atingem, são acionadas uma a seguir à outra, cada uma com o link dedicado a si própria, e o comando apresenta `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Quando sequer uma câmara de cada vez não cabe, a execução é recusada antes de qualquer câmara transmitir: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, saída `1`.
- **Todas as câmaras têm de transmitir, ou a execução falha.** Uma câmara que não transmita nenhum fotograma constitui um erro: o comando apresenta `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, em que `<why>` é o erro de recolha dessa câmara, seguido da indicação da causa, e termina com `1`. Um quadro que chegou com perda de pacotes GVSP (`IncompleteBufferError: GVSP packet loss …`) significa que a rede não está a suportar a largura de banda combinada das câmaras (`Every missing frame arrived with GVSP packet loss: …`): verifique a velocidade da ligação do anfitrião e da ligação ascendente do comutador (`lattice network`), utilize quadros jumbo, ou abra menos câmaras com o comando `--serials`. Uma câmara que não transmitiu nada normalmente não está a receber o sinal de ativação da câmara principal: verifique o seu cabo de sincronização M8 e, em seguida, execute o comando `lattice multi-detect`. O que cada comando faz quando falta uma câmara num grupo de quadros está indicado na respetiva linha.
- **Cada câmara é reposto tal como foi encontrada.** Quando a execução termina, cada câmara recupera o que tinha quando a execução a abriu: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, e `LineMode` / `LineSource` em `Line2` e `Line3` (e a entrada de disparo elimina o ruído de uma matriz que alimenta a linha de um escravo, `LineFilterSelector` / `LineFilterWidth`); o seu `PtpEnable` (e o conjunto `PtpSlaveOnly` / `PtpDomainNumber` associado a ele); `ExposureAuto` / `GainAuto` (e o `ExposureTime` / `Gain`, onde estava definido com um valor manual); o `TargetBrightness`, o `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (e cada `BalanceRatio` em que o equilíbrio de brancos era manual); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (e `Gamma` em que a gama estava ativada), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; o tamanho do pacote (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); o ritmo, `DeviceLinkThroughputLimitMode` (e `DeviceLinkThroughputLimit`, em que o modo era `On`), ode pacotes `GevSCPD` e o desfasamento de início `GevSCFTD` que uma matriz atribui a cada câmara; os blocos de dados que cada fluxo ativa (`ChunkModeActive`, cada `ChunkEnable` sob o seu `ChunkSelector`); `AcquisitionFrameRateEnable` (e `AcquisitionFrameRate` onde a ativação estava ligada — com ela desligada, a taxa é de leitura apenas e lê o máximo que o ritmo permite, pelo que não se trata de uma configuração). Apenas o que difere é gravado: a câmara parou primeiro, o ritmo e a taxa de fotogramas antes de uma exposição que necessita do seu período de fotograma (uma câmara limita o `ExposureTime` a um período de fotograma). Com o `--keep-settings`, a ligação do disparador e da linha de sincronização é desativada, a câmara funciona em modo livre e o resto é guardado na própria memória da câmara conforme definido na configuração de funcionamento; o `--clear-settings` restaura as predefinições de fábrica.
- **A exposição é definida a partir da cena antes de o primeiro fotograma ser guardado.** A execução recolhe grupos de fotogramas sincronizados e descarta-os enquanto define a exposição e o ganho de cada câmara a partir desses fotogramas. O objetivo é obter um brilho médio no alvo da câmara, sem que mais de 1 % do fotograma seja cortado; uma cena com realces brilhantes fica aquém do alvo. Isto demora normalmente entre 2 e 6 grupos de fotogramas e, no máximo, 5 s.
  - Cada câmara aponta para o alvo a partir do qual uma matriz conectada a inicia, através do seu filtro: TargetBrightness **40**numa câmara multiespectral (RGN, OCN, NGB) e**60** numa câmara RGB. Isso corresponde a metade do valor-alvo de exposição fornecido pelo 80 (três quartos no RGB), pelo que é menos provável que um painel de refletância brilhante sofra recorte. `--brightness N` (0-255) define, em vez disso, o valor-alvo de cada câmara (`multi-test` não tem `--brightness`: a sua exposição automática está desativada). Os registos do backend identificam o valor-alvo de cada câmara: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - A exposição vem em primeiro lugar, até ao limite de exposição automática da própria câmara. O ganho cobre apenas o que resta.
  - A exposição e o ganho determinados são mantidos durante toda a execução, pelo que cada fotograma de um `multi-stream` é capturado com a mesma exposição. Uma captura que tenha de acompanhar variações de luz deve ser efetuada numa matriz ligada (`array-connect`), cuja exposição automática se ajusta continuamente.
  - O `--exposure` e o `--gain` são mantidos exatamente como foram definidos. Com ambas especificadas, nenhum fotograma é descartado.
  - Quando a execução termina, a exposição automática de cada câmara é reativada.
  - O registo do backend apresenta o resultado: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, ou `exposure NOT solved after …` com o motivo e `(not settled)` em cada câmara que não foi incluída. Nesse caso, os fotogramas continuam a ser captados.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` altera o limite de 5 s; `0` desativa esta funcionalidade (ver [Variáveis de Ambiente](#environment-variables)).

**Enquanto o Chloros mantiver qualquer câmara** (a GUI, o `cam-connect` ou um conjunto ligado), o `multi-stream`, o `multi-test`, `multi-detect`, `align-apply`, `align-stream` e `index --live` recusam-se a ser executados, porque abrem todas as câmaras da rede e uma segunda abertura de uma câmara retida pode provocar uma falha no backend. (O `index --live` recusa-se quando uma câmara que iria abrir está retida: a sua `--serials` ou as câmaras do seu perfil, e qualquer câmara de um perfil que não indique nenhuma.) O backend responde com um código HTTP 409 ao `CAMERA_HELD`, indicando cada câmara bloqueada e os comandos de libertação (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), e o comando termina com `1`. Com uma matriz ligada, o comando `array-capture --continuous` grava-a no lugar do comando `multi-stream`, e o comando `array-capture` (alinhado por predefinição) guarda um conjunto alinhado no lugar de `align-apply`; `multi-capture` e `align-calibrate` utilizam uma matriz ligada que contém as suas câmaras em vez de as abrirem.

| Subcomando | Finalidade |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Lista todas as câmaras com funções de sincronização; com o `--serials`, apenas essas; o primeiro como `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Um fotograma sincronizado de cada câmara. Guarda **todos os tipos de exportação por predefinição**quando está ligada uma matriz persistente e a leitura do DAQ utilizada ([A leitura do DAQ que uma captura guarda](#the-daq-reading-a-capture-saves)); o recurso de fallback transitório sem matriz é**apenas despixelarizado** (execute primeiro o `array-connect` para o resto). O `--brightness` aplica-se apenas ao recurso de fallback: um conjunto ligado mantém a sua própria exposição automática. O recurso de fallback não guarda o `.daq` (`--no-daq` / `--daq-tolerance-ms` não têm efeito nesse caso), e o `multi-capture` não aceita o `--force-daq`. No recurso de fallback, uma câmara que não forneça nenhum fotograma falha na captura (`Error: multi-capture failed: no frame from …`, sair de `1`) e nenhum ficheiro desse grupo de fotogramas é guardado. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Transmitir fotogramas sincronizados (transitório), `--fps` (padrão) `3`, até Ctrl+C ou `--count` grupos. Um grupo de fotogramas para o qual uma câmara não transmitiu nada não é gravado, e três consecutivos encerram o fluxo. Se algum grupo foi ignorado, a execução termina imprimindo `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, além de `Error: the stream stopped: …` indicando o nome da câmara quando esta encerrou o fluxo, e sai de `1`X. Interrompe-se automaticamente quando a unidade fica cheia, exibe `Failed:` / `Disk full:` e sai com `1`; qualquer ficheiro que não tenha conseguido gravar também encerra o `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Teste de temporização de sincronização GPIO (transitório), `--count` gatilhos por defeito `10`. Imprime como a rede é partilhada (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, e a linha `Serial emit:` quando as câmaras são acionadas uma a seguir à outra, cujo desfasamento é então incluído nos carimbos de data/hora), uma linha `Sync group: master SN:<serial> -> SN:<serial>, …` por grupo de sincronização (`(no slaves: not synced)` para uma câmara sem cabo de sincronização, que é acionada de forma independente), depois uma linha por disparo: `[  0] SN:<serial>=<timestamp> \| …`, com `SN:<serial>=(no frame)` para uma câmara que não transmitiu nada para esse disparo. Em seguida, uma linha `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` e uma linha `SN:<serial>: mean <±x> us, jitter <y> us` por câmara, cada câmara em relação ao mestre dentro de um ciclo de relógio: um disparador cujos carimbos misturam os relógios dos dispositivos das câmaras e o PTP (PTP desativado ou ainda a sincronizar) é omitido, com um `Note:` a contá-los. Qualquer fotograma perdido faz com que o teste falhe (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, saída `1`); três gatilhos incompletos consecutivos encerram-no antecipadamente. O teste também falha (sair de `1`) quando menos de dois gatilhos mantêm todas as câmaras no mesmo relógio (`… the clocks cannot be compared`), ou quando os relógios nunca coincidem: relógios PTP com um desfasamento superior a 1 ms (mais qualquer desfasamento da camada serial) em relação ao mestre (`… PTP is not locked`), ou relógios de dispositivos cujo desfasamento varie mais do que esse valor entre os gatilhos. Inicia o «grandmaster» PTP, caso este não esteja a funcionar, ativa o PTP de todas as câmaras (tal como faz o `array-connect`, independentemente da configuração da rede) e, antes do primeiro disparador, aguarda que o relógio PTP de todas as câmaras se sincronize com o «grandmaster» e estabilize: `PtpStatus` Escravo num mestre principal, com um desvio inferior a 100 us em relação a este e estável durante 3 s, e cada câmara sincronizada dessa forma, sem interrupção, durante 120 s — cerca de 2 minutos após a ligação. Um relógio que acabou de se sincronizar apresenta um desvio de até ~200 us durante cerca de 2 minutos após indicar «Slave», e o próprio `PtpOffsetFromMaster` da câmara não o mostra; 120 s colocam as câmaras a ~10 us de um relógio estabilizado. Uma câmara ainda se mantinha sincronizada desde um teste anterior que deixou o PTP ativado, tal como este backend o detetou, começa a contar a partir desse bloqueio, pelo que um teste realizado imediatamente a seguir a outro não tem de esperar novamente; uma câmara que o teste liga, ou uma encontrada com o PTP ativado que este backend ainda não tenha visto bloqueada (ligada noutro local, ou antes de um reinício do backend), aguarda os seus 120 s, e num conjunto de ambas as câmaras que acabaram de se sincronizar, estas continuam a aguardar o seu tempo. Uma câmara que sai do modo Escravo, ultrapassa os 100 us ou segue outro grandmaster durante 3 leituras consecutivas (1,5 s) durante a espera, reinicia a sua contagem; uma ou duas dessas leituras são registadas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e a contagem prossegue; um ID de «grandmaster» que não possa ser lido é considerado desconhecido, e não como uma alteração. O reinício simultâneo da contagem de todas as câmaras (no espaço de um segundo) deve-se à variação do tempo do «grandmaster», e não das câmaras: o registo do backend indica `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lista-o em `grandmaster_steps`, e uma espera que depois expira inicia o seu erro com `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. O limite não é alargado para este caso. `--ptp-settle SECONDS` define o tempo de estabilização para a execução (0-600; `0` para ausência de espera de estabilização). A espera está limitada ao tempo de estabilização mais 90 s (210 s por predefinição, no mínimo 90 s), e o comando aguarda o backend durante 240 s mais o tempo de estabilização (360 s por predefinição). Apresenta a mensagem `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, e o registo do backend apresenta uma linha a cada 5 s com o estado e o desvio de cada câmara, há quanto tempo está bloqueada e quanto tempo ainda falta: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Todas as câmaras ligam-se com o PTP desativado; a partir daí, os relógios sincronizam-se em cerca de 10 s. Posteriormente, cada câmara é reposto no estado em que o teste a encontrou — o seu PTP, a exposição automática e o ganho que o teste mantém desativados durante a execução, e o resto (a menos que `--keep-settings`) — assim, uma câmara que tenha ficado com o PTP ativado (`lattice configure -s <serial> --set PtpEnable=True`, ou uma execução com `--keep-settings`) permanece sincronizada e, assim que este backend verificar que está sincronizada durante o tempo de estabilização, um teste posterior não aguarda por ela. Se os relógios não se estabilizarem durante a espera, o teste falha antes de ser acionado (`PTP: NOT locked …`, saída `1`) e indica o que fazer: uma câmara que nunca ficou bloqueada (`PtpStatus Listening`) significa que o «grandmaster» não está a conseguir contactá-la (`time-sync status`); uma que continuou a perder a ligação é identificada pelo tempo durante o qual esteve bloqueada (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); relógios ainda a convergir: verifique `time-sync status`, e, para que fiquem sincronizadas antes do teste, ative o PTP e mantenha-o ativado (`lattice configure -s <serial> --set PtpEnable=True` para cada câmara, ou execute o teste com `--keep-settings`; o teste repõe o que encontrou, pelo que permanece ativado), depois execute-o. |
| `lattice multi-detect [--line LINE] [--json]` | Identifique as câmaras na linha de sincronização M8 e uma câmara mestre predefinida: a câmara RGB (modelo que termina em `-FRGB`) se estiver ligada por cabo; caso contrário, a primeira câmara ligada por cabo por ordem de número de série. Os cabos M8 encontram-se numa junção, pelo que **qualquer câmara ligada por cabo pode ser a mestre** — o `MASTER` impresso é um valor predefinido, não um facto de ligação. Cada câmara, por sua vez, aciona a linha e as outras são verificadas quanto ao seu pulso; uma câmara já encontrada no grupo ligado não é verificada novamente, pelo que um conjunto totalmente ligado por cabo demora uma ronda. Na ausência de `--line`, verifica `Line2`, depois o `Line3`. Imprime o `MASTER: SN:…`, um `SLAVE: SN:…` por cada outra câmara ligada e o `(probed on <line>)`; o `--json` adiciona o resultado completo (`master`, `slaves`, `no_cable` — câmaras às quais o impulso não chegou — `line` e, por câmara, `results`). Quando termina, cada câmara recupera as definições de disparo e linha de sincronização que tinha (os mesmos nós que acima). |

#### Alinhamento

| Subcomando | Finalidade |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — mais os controlos do detetor/comparador `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parâmetros RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinação de múltiplos fotogramas `[--averaging mean\|median\|inlier_weighted]`, restrições geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrição espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` e substituiçõessubordinado `[--per-cam-override SN:KEY=VALUE]` (repetível) | Calcular o perfil de alinhamento a partir de câmaras em tempo real. Valores predefinidos: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (na pasta a partir da qual o executa), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (O `project align calibrate` tem as suas próprias predefinições; consulte [a sua tabela](#project-align-calibrate-options).) O `--prefilter` tem como predefinição o `gradient` (mapa de arestas; corresponde ao alinhador GUI/matriz — as arestas mantêm-se entre as bandas espectrais). O `--matcher flann` compensa a partir de cerca de 5000 características; o `--averaging median` é robusto face a uma captura defeituosa; o `inlier_weighted` pondera com base no número de correspondências; o `--lock-scale` projeta para a rotação mais próxima (sem escala), `--lock-axis` zera uma componente de translação; O `--mask` aplica-se a todas as câmaras (utilize o `--per-cam-override` para definições específicas por câmara, por exemplo, o `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` recusa guardar uma calibração cujo RMS de reprojeção exceda o limite. `--method manual` obtém os seus pontos a partir de `--points PATH`: um objeto JSON, o `{"<serial>": [[x, y], ...], ...}`, que fornece as coordenadas em píxeis de cada câmara para as mesmas características, incluindo as da câmara de referência (pelo menos 2 por câmara); cada um dos `--frames` é calculado a partir desses pontos. Quando um array ligado contém as câmaras, a calibração utiliza os fotogramas em tempo real desse array, e o perfil passa também a ser o alinhamento do array para as suas capturas. Caso contrário, abre-se um [array transitório](#multi-camera-transient-sessions): uma câmara que não forneça nenhum fotograma em qualquer um dos `--frames` falha na calibração (sai de `1`, nomeando-a), e nenhum perfil é guardado. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Captura um fotograma multibanda alinhado. O `--bit-depth`, por predefinição, faz a correspondência com a câmara; `--no-crop` mantém o fotograma completo (preenchendo com preto); `--interpolation` (padrão `linear`) e `--border-mode`/`--border-value` (padrão `constant`/0) controlam a distorção da CPU — o caminho da GPU é bilinear independentemente disso. Abre uma [matriz transitória](#multi-camera-transient-sessions): uma câmara que não forneça nenhum fotograma falha na captura (sai de `1`, nomeando-a), e nada é gravado. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Transmite fotogramas multibanda alinhados (os mesmos controlos de warp que `align-apply`), `--fps` por predefinição `3`. Abre uma [matriz transitória](#multi-camera-transient-sessions): um grupo de fotogramas para o qual uma câmara não forneceu nada não é guardado, e três seguidos encerram o fluxo. Se algum grupo tiver sido ignorado, a execução termina com a impressão de `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` e `Error: …`, e sai com o código `1`. Um fluxo que termine devido a qualquer outro erro também exibe `Error: …` e sai com o código `1`. |
| `lattice align-info --profile PATH [--json]` | Apresentar detalhes do perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Alterar a ordem das camadas. |

#### Índice / Matemática da Vegetação

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` mapeia os símbolos de uma predefinição para bandas** (NDVI: `red`, `nir`), um `--channel SYMBOL=BAND` para cada um.

- **`--live`: apenas para um símbolo ao qual a pilha não consegue responder.** Tal como no caso do `--input`, um símbolo de uma predefinição que seja omitido assume a única banda da pilha (as bandas das câmaras ativadas do perfil) cujo nome comece por esse símbolo. Isto é verificado antes de qualquer câmara ser aberta: um símbolo ao qual nenhuma banda, ou várias bandas, respondem é então recusado, indicando-as, e sai do `1`. Um `--formula` personalizado necessita de um `--channel` para cada símbolo.
- **Os produtos `--live` indicam a quem pertencem.** O `index.tif` e o `index_LUT.png` contêm os dados da câmara da pilha (marca, modelo, número de série), a hora da captura (exposição da câmara de referência, UTC) e a lente, e ambos designam o índice como a sua única banda (`XMP-Camera:BandName` `NDVI`; o ficheiro PNG a cores designa o índice, não os seus canais de cor), tal como faz um produto `--input`. Um índice proveniente das bandas de várias câmaras nomeia cada uma delas, e o seu `Model` é `LATTICE` — ver [Ficheiros criados a partir de várias câmaras](#files-made-from-several-cameras).
- **`--input`: apenas para um símbolo que o ficheiro não consegue identificar.** Um símbolo que não for especificado assume a única banda do ficheiro cujo nome comece por esse símbolo: `red` assume o valor de `Red_660` ou `Red_660_RGN`; `nir` assume o valor de `NIR_850`. Os nomes provêm do próprio ficheiro: a lista de bandas da pilha ou, numa captura de radianância/refletância do Chloros ou num produto `process`, as suas etiquetas `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Assim, o `lattice index --input <reflectance capture> --preset NDVI` não necessita de `--channel`.
- **Não se faz a distinção entre duas bandas que correspondem a um único símbolo** (uma pilha de 9 câmaras tem duas bandas NIR): identifique essa. Um símbolo ao qual nada corresponde é rejeitado com `channel_map missing entries`; saia de `1`.**Os nomes das bandas numa pilha** indicam o filtro da câmara: as bandas de uma câmara M3C são `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` num RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` num OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` num RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` num NGB — e abanda única é `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` lista as bandas de um perfil.

Conjunto completo de indicadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (repetível), `--capture-level raw|debayered|radiance|reflectance|unknown` (substitui o nível de captura registado no ficheiro TIFF de origem; predefinição: lido a partir dos metadados do TIFF, recorrendo, em caso de falha, ao  de um ficheiro Chloros, pelo que uma captura de refletância indica `reflectance`), `--output PATH` / `-o` (offline: um caminho; com uma extensão de imagem, os ficheiros são nomeados a partir da sua raiz — o `-o ndvi_FRGN.png` grava o `ndvi_FRGN.tif` e o `ndvi_FRGN_LUT.png` — e, sem extensão, é um prefixo, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; valor por predefinição ao lado da entrada, como `<input name>_<preset>`; com `--live` e sem `-o`, uma pasta `output/` na pasta a partir da qual executa o comando), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Com o `--live`, os botões de distorção de alinhamento também se aplicam: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (o alvo de exposição automática das câmaras, ver as regras transitórias), `--serials SN1,SN2,…`. `--live` abre um [matriz transitória](#sessões-transitórias-multicâmara): uma câmara que não forneça nenhum fotograma faz com que a execução falhe (sai de `1`, nomeando-a), e não é gravada nenhuma pilha nem índice. Abre o `--serials` (o primeiro dispara o gatilho); caso contrário, as câmaras do perfil cuja entrada `layer_order` esteja ativada, começando pela referência — mesmo que as bandas da referência estejam desativadas, uma vez que as outras estão alinhadas com ela e é esta que dispara o gatilho. Uma câmara cujas bandas estejam desativadas nunca é calibrada, pelo que uma câmara RGB desativada no perfil já não falha num índice de refletância (`no-absolute-radiance-scale`). Um perfil que não nomeie nenhuma câmara abre todas as câmaras da rede, tal como anteriormente.

Todos os formulários são executados no backend, pelo que este deve estar em execução: inicie-o a partir da GUI do Chloros, ou com o comando que a CLI apresenta quando não consegue aceder ao backend. Offline, o backend lê o `--input` e grava o ficheiro TIFF de precisão flutuante e o PNG colorido ele próprio, pelo que um `--input` ou `-o` é resolvido em relação à pasta a partir da qual executa o comando (ver [Convenções](#conventions)). Ambos os produtos offline incluem o registo da câmara da entrada, tal como acontece numa exportação de índice `process`: Marca / Modelo / número de série, GPS, as etiquetas da lente, a hora de captura (a própria da entrada; quando os seus dados EXIF não indicam nenhuma, o instante de exposição que registou ou a data e hora do nome do ficheiro, em UTC) e a identidade de calibração MAPIR — nunca a calibração raw-DN, que não se aplica a um índice — além de `Chloros:ProcessingLevel=index` e da expressão do índice; o TIFF de tipo float identifica uma banda, a predefinição. `--list-presets` e `--list-gradients` imprimem as tabelas de predefinições e gradientes do backend. Uma fórmula que o backend rejeita, ou uma banda `--channel` que o TIFF não possui, sai de `1` com o motivo indicado pelo backend.

> **Os símbolos `--channel` distinguem maiúsculas de minúsculas.** O lado do símbolo deve corresponder exatamente aos nomes dos canais da predefinição (as predefinições utilizam minúsculas, por exemplo, NDVI = `red`,`nir` — verifique `--list-presets`), e o lado da banda deve corresponder a um nome de banda na pilha alinhada (ou ser um índice de banda com base em 0 no modo offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funciona; `--channel RED=660` falha com um erro `channel_map missing entries` .

#### `lattice index` — máscaras de pilha multibanda

Em `--capture-level radiance` ou `reflectance`, a separação 3×3 de cada câmara pode limitar uma banda a 0 (ver [Máscara de limitação de desmistura](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Uma pilha multibanda contém bandas de várias câmaras, e cada câmara limita a sua própria banda; por isso, o `--live --save-multiband` escreve **uma máscara por câmara de origem** ao lado da pilha:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Cada máscara é de banda única `uint8` na grelha de píxeis **da pilha** (através do mesmo alinhamento, distorção e recorte), com os próprios bits dessa câmara: bit 0 = a banda do seu canal R, bit 1 = G, bit 2 = B. Uma banda da pilha retirada do canal *c* de uma câmara é limitada onde o bit *c* dessa câmara (R = 0, G = 1, B = 2) está definido.
- A proveniência incorporada de cada banda indica o nome da máscara da respetiva câmara (`unmix_clamp_mask`, em relação à pasta da pilha, ou `none`) e o bit a ler (`unmix_clamp_mask_bit`).
- Uma câmara cuja separação de cores não limitou nada não recebe nenhum ficheiro; uma câmara monocromática ou RGB não executa a separação de cores e também não recebe nenhum. A CLI apresenta cada máscara sob a pilha (`clamp mask SN…: …`); a resposta lista-as como `clamp_masks`.
- O ficheiro TIFF/PNG de índice da mesma execução é calculado a partir da mesma pilha, na mesma grelha, pelo que as máscaras também o descrevem. O `--capture-level raw` não executa a separação de cores e não grava nenhuma máscara.
- A pilha e o índice encontram-se diretamente em `--output`, pelo que as máscaras vão para o próprio `clamp_masks/` de `--output` — nunca acima da pasta que nomeou. Essa pasta mistura tipos de produtos (pilha, índice, renderização colorida): selecione os seus ficheiros, não entregue a pasta inteira ao software de fotogrametria.

#### Ligações Persistentes (Smart-Prep, Fluxo Equivalente à GUI)

Estes comandos mantêm as câmaras abertas no conjunto de recursos de fundo entre invocações dCLI.

| Subcomando | Finalidade |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adiciona uma câmara ao conjunto (câmara única, sem matriz). As [opções comuns](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` e `--packet-size` (`standard` = 1500, `jumbo` = 9000, ou bytes; sondas `auto`) são aplicadas na ligação (um dos comutadores `--exposure` ou `--gain` que desativa o modo automático). Retorna assim que o fluxo da câmara tiver começado (aguardando cerca de 15 s por isso, depois 10 s pelo seu primeiro buffer completo) e apresenta `stream: streaming` ou `stream: no_frames -- connected, no frames yet: …` quando o fluxo não transmitiu nada — no caso de uma câmara a utilizar pacotes jumbo, com a recomendação de se reconectar com `--packet-size standard`. Enquanto uma câmara se liga (uma primeira ligação descarrega a sua calibração, cerca de 70 s), outro comando que necessite da câmara aguarda por ela no máximo 5 s, depois falha com o código HTTP 503 `CAMERA_POOL_BUSY`, indicando a câmara que está a ligar-se; execute-o novamente daqui a pouco. `lattice status` e a lista de câmaras nunca esperam: uma câmara a meio da ligação aparece como a ligar-se, com a sua fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Libertação. A câmara é reposto tal como foi encontrada no momento da ligação (ligação do disparador, PTP, definições); `--keep-settings` guarda as suas definições na sua própria memória; `--clear-settings` repõe essa memória para as definições de fábrica (ver [Memória da câmara](#camera-memory----keep-settings----clear-settings)). Uma câmara que faça parte de um conjunto liberta todo o conjunto. |
| `lattice cam-list` | Listar câmaras no conjunto. |
| **`lattice array-connect`**|**Ligar um conjunto sincronizado persistente (O ponto de entrada recomendado).** Executa todo o fluxo de preparação inteligente da GUI. Demora cerca de um minuto para 9 câmaras; logo após as câmaras serem ligadas, pode demorar vários minutos. Enquanto aguarda, apresenta cada etapa alcançada pelo backend, como `    <s> s: <step>` (por exemplo, `    41 s: open SN:214000535 (4 of 9)`), e o registo do backend apresenta uma linha de tempo por etapa (`[array-connect <id>] <step>: <s> s …`) e uma última linha com as etapas mais demoradas. A espera dura até 600 s. Uma ligação que não tenha sido estabelecida até esse momento é anulada pelo backend: cada câmara que tenha sido aberta é reposto no estado em que foi encontrada e fechada, o comando apresenta `Error:` com a etapa em que tinha chegado e sai em `1`, e nenhuma matriz fica ligada (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C cancela a ligação da mesma forma (`Interrupted: …`); se a matriz já se tivesse ligado, a linha indica isso e menciona o `array-disconnect` que a desliga. Um comando interrompido enquanto aguarda (`kill -9`) faz com que a ligação seja desfeita quando o backend não receber resposta durante 30 s. Uma matriz que esteja ativa, mas que ainda não tenha um grupo de quadros sincronizado, continua a ser reportada como ligada, com o seu fluxo indicado como `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Liberta uma matriz. Todas as câmaras ficam a funcionar livremente (`TriggerMode` `Off`), com tudo o resto tal como a ligação o encontrou — as suas definições de disparador e linha de sincronização (a do mestre`TriggerSource` volta ao que era, a sua linha de sincronização volta a ser uma entrada), o seu PTP e as definições da câmara alteradas pela sessão — para que um comando de câmara única num membro funcione imediatamente. `--keep-settings` guarda as definições de cada câmaratal como a sessão as deixou na própria memória da câmara; o `--clear-settings` repõe essa memória para as predefinições de fábrica. Consulte [Memória da câmara](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Listar matrizes ligadas. |
| `lattice array-status [--array-id ID]` | fps em tempo real, PTP, último erro. `ptp_enabled` é igual a `True` apenas quando todos os membros apresentam PTP ativado; `ptp_off:` identifica um membro que não o faz. A linha `state:` indica o `restarting (applying settings)` durante um reinício das definições, o `recovering (…)` enquanto um membro que perdeu alimentação é reaberto (ver a nota abaixo), `stalled (…)` enquanto o ciclo de captura está em execução mas não publica nenhum grupo de quadros, e `stopped (…)` assim que o ciclo de captura desistir (reconecte a matriz). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Uma captura sincronizada da matriz ativa — Única / Contínua / Intervalar / Mais rápido. **O valor predefinido é `all`**(um ficheiro por tipo de exportação aplicável por câmara),**alinhado** tal como a captura da aplicação (`--no-aligned` para ficheiros nativos). As câmaras ignoradas (por exemplo, RGB, excluídas da radianância/refletância) são relatadas com `Skipped: SN:<serial> (<reason>)`; a leitura do DAQ utilizada na captura é guardada em simultâneo, incluindo uma captura apenas em formato bruto, e relatada com `DAQ: <path>` ([O DAQ que lê uma captura guarda](#o-daq-que-lê-uma-captura-guarda)). Consulte [Modos de Captura, Gravadores e Reprocessamento Offline](#modos-de-captura-gravadores--reprocessamento-offline). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Gravar a visualização do índice combinado em tempo real em vídeo/GIF (nível de monitorização; sem nenhuma janela da GUI aberta, o backend renderiza o NDVI da matriz, ou `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Série de imagens Bayer brutas com alta taxa de fotogramas (nível de análise; reprocessar offline). Tal como o `array-record` e o `array-capture --continuous`, este processo comunica com o backend em cada sondagem: é encerrado sem ser interrompido (`kill -9`), o backend interrompe-o 30 s após a última verificação (`stop_reason` `client_gone`; o manifesto da sequência é finalizado e a construção do índice combinado é iniciada, tal como acontece numa interrupção) e a matriz permanece ligada. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocessar uma sequência em bruto guardada em vídeo(s) calibrado(s). |

> **Uma câmara que fica sem alimentação ou perde a ligação volta a funcionar por si própria.** Quando um membro deixa de responder (um ciclo de reinicialização do interruptor reinicia todas as câmaras a que fornece alimentação), o conjunto aguarda por ele, reabre a ligação quando este volta a responder, e devolve-lhe o que o `array-connect` definiu: PTP, a linha de disparo e sincronização, o ritmo de rede, o formato de píxeis e o tamanho do fotograma, bem como a exposição automática. Em seguida, retoma a transmissão sem nenhum novo `array-connect`. Entretanto, o `array-status` apresenta o `state: recovering (...)` e o `last_error` indica qual a câmara pela qual está à espera; posteriormente, imprime o `re-opened:`. Uma câmara que reiniciou volta com o PTP desativado; por isso, até ser reaberta, o `ptp_enabled` é o `False` e o `ptp_off:` identifica-a.

> **Uma câmara retida nunca é aberta uma segunda vez.** Uma segunda abertura interrompe o fluxo do detentor e reescreve as suas definições, podendo causar uma falha no backend. Assim, numa câmara retida pelo pool (a GUI ou `cam-connect`), um membro de um array ou uma câmara retida por uma tarefa em segundo plano (um `lattice continuous` em execução, `multi-stream` ou `align-stream`, seja o seu próprio ou de outro cliente), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, umúnica tentativa `white-balance` e `presets --apply` recusam com código HTTP 409 `CAMERA_HELD`, indicando o titular e o comando de libertação (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, ou, no caso de uma tarefa, o seu ID e quem a iniciou), e sair `1`. O mesmo acontece com uma câmara que outro comando esteja a utilizar neste momento (um `multi-test` ou `multi-capture` a abrir as suas câmaras, outro comando único), para todos os comandos aqui: o erro indica para aguardar que esse comando termine. O `lattice power`, sem o `--sleep`, consulta o próprio identificador do detentor, caso exista (o conjunto, uma matriz, a câmara aberta de um trabalho), e é recusado se a câmara estiver a ser utilizada por outro comando. Cada resposta `CAMERA_HELD` contém o código tanto em `code` como em `error_code`. Para os comandos que abrem todas as câmaras, consulte [Multicâmaras (Sessões Transitórias)](#multi-camera-transient-sessions).

##### Opções do `array-connect`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | deteção automática de todas as câmaras LATTICE (requer ≥2) | A primeira porta série é a MASTER. Quando omitida, a deteção filtra os modelos LATTICE (M3C / M3M) e liga-os todos. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | automático | Taxa de disparo do mestre. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituir o seletor de nível. |
| `--wire-ceiling-mbps MB_PER_S` | detetado automaticamente | **A largura de banda sustentada do anfitrião, em MB/s — o valor do qual depende toda a alocação da matriz.** Reduza-o quando a matriz reportar quadros corrompidos pelo GVSP: o valor automático é derivado da taxa de ligação anunciada pela NIC, que sobrestima adaptadores USB, pistas PCIe estreitas e estruturas partilhadas ocupadas. É guardado no bloco de captura da matriz do projeto, pelo que uma reabertura ou uma reconexão via CLI/SDK o restaura. Consulte [Integridade da matriz](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | nenhum (sensor completo) | Agrupamento de pixels por hardware (sempre Média). |
| `--width N` / `--height N` / `--pixel-format FMT` | sensor completo; formato por câmara | Um tamanho de fotograma ou formato de píxeis explícito para cada câmara, em vez do sensor completo e da seleção do formato por câmara (Mono12 / BayerRG12). |
| `--no-recommend` | desativado | Ignorar a etapa de análise de rede (cada câmara mantém então o seu tamanho atual, em vez de ser definida para o sensor completo). |
| `--no-ptp` | desativado | Desativar PTP (os carimbos de data/hora entre câmaras **não** são então comparáveis). |
| `--packed {auto,on,off}` | `auto` | Transmite fotogramas de 12 bits compactados e descompacta-os neste computador — mais fotogramas por segundo com o sensor completo, dados idênticos. `auto` compacta apenas quando isso for útil e este computador tiver capacidade para tal; o `off` nunca compacta. Ver [Transporte compactado de 12 bits](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Tamanho do pacote GVSP para cada câmara na matriz. `auto` O  permite que cada câmara explore a rede; `jumbo`  = 9000, `standard`  = 1500 ou um tamanho em bytes bloqueia-a. Antes da versão 1.3.0, o `array-connect` aceitava este sinalizador e ignorava-o. |

> **Sensor completo por predefinição.** Sem `--width` / `--height` / `--binning` /
> `--pixel-format`, o `array-connect` configura todas as câmaras para o sensor completo — sem
> binning, sem recorte de ROI — e permite que a taxa de fotogramas desça para o que este computador e
> a rede conseguem fornecer, em vez de reduzir os fotogramas para manter a taxa elevada. A
> linha de análise de rede mostra a taxa projetada, e a linha seguinte lista
> opções mais rápidas (agrupamento ou um ROI mais pequeno) com as respetivas taxas projetadas; passe
> `--binning` ou `--width`/`--height` para selecionar uma. O «Connect» nunca realiza binning nem recortes
> por si próprio.

##### Transporte compactado de 12 bits (`--packed`)

Com o sensor a funcionar a plena capacidade, a taxa de fotogramas de uma matriz é determinada pela velocidade a que cada câmara pode enviar
os seus fotogramas. Os formatos de 12 bits (`BayerRG12`, `Mono12`) são normalmente transmitidos como palavras de 16 bits;
o transporte **compactado** envia os mesmos 12 bits em 1,5 bytes por pixel e
descompacta-os neste computador, pelo que cada fotograma demora 25% menos tempo a ser enviado —
cerca de **33% mais fotogramas por segundo com o sensor a funcionar na capacidade máxima** (medido: 10,7 → 14,0 fps
num conjunto de 9 câmaras). **Os dados guardados são idênticos**: os mesmos valores, os mesmos
ficheiros de 16 bits, os mesmos metadados.

- O `auto` (predefinição) comprime apenas quando isso é vantajoso **e** este computador tem capacidade para
  efectuar a descompressão. Ao estabelecer a ligação, o backend mede a sua própria velocidade de descompactação, o número de núcleos da CPU,
  a carga atual da CPU e a largura de banda disponível, pelo que um computador de secretária com núcleos disponíveis compacta
  com o sensor a funcionar a plena capacidade, enquanto um computador ocupado ou com pouca potência transmite os dados já descompactados. As pequenas áreas de interesse (ROIs) e os fotogramas agrupados ganham
  pouco.
- `off` nunca compacta — utilize-o para desativar a compactação durante o diagnóstico, ou num
  computador que tenha de reservar toda a sua CPU para outras tarefas.
- `on` compacta sempre formatos de 12 bits.

A saída do «connect» mostra a decisão e o motivo («compactado a 12 bits: em (automático:
projeção de 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` relata
`packed_transport` e `packed_decision`. Para definir um valor predefinido para toda a máquina
em vez de passar o sinalizador de cada vez, defina `CHLOROS_ARRAY_PACKED12=off` (ou
`on`) no ambiente do backend — um `--packed on|off` explícito continua a ter prioridade.
Os formatos de 8 e 16 bits nunca são compactados.

### Um registo de projeto, todos os modos

Uma pasta de projeto contém as definições de todos os dispositivos: `cameras.json` — de cada câmara (`cameras[].settings`, para uma única câmara e para cada membro do conjunto) e as configurações globais de cada conjunto (`arrays[].capture`: formato, ROI, binning, taxa de disparo, orçamento de fios) — e `sensors.json`, de cada sensor de luz (tempo de integração, média de fotogramas, limite). A aplicação, a CLI, o SDK e o servidor MCP leem e escrevem os mesmos ficheiros, da mesma forma:

- **Um dispositivo ligado a um projeto é guardado nesse projeto e regista todas as alterações nele.** «No âmbito de um projeto» significa com um projeto aberto no backend (da aplicação) ou — a partir do SDK — através de um `ChlorosProject`, que define o nome da sua própria pasta. Uma câmara, matriz ou sensor de luz que o projeto ainda não contenha é adicionada aos seus ficheiros quando se liga (`lattice cam-connect`, `array-connect`, `daq pool-connect`, a aplicação, o SDK, `chloros-mcp`), com as suas definições. Qualquer alteração em qualquer modo (`lattice exposure`, `gain`, `configure --set`, `resolution`, asas definições e a frequência de disparo, o limite do sensor de luz, os controlos da aplicação) é guardada nos ficheiros desse projeto um instante depois. Uma exposição, ganho ou valor-alvo de AE definido numa câmara é guardado como mantido, pelo que, ao reabrir o projeto, esses valores são mantidos novamente.
- **Ao abrir um projeto, os seus dispositivos são configurados de acordo com o projeto**, em todos os modos: uma câmara, matriz ou sensor de luz ligada sem definições próprias recebe as definições guardadas do projeto (`lattice cam-connect` / `array-connect` com um projeto aberto, `chloros-cli project connect`, `open_project().connect_all()`, ao reabrir a aplicação). Um conjunto ligado com um novo ID atualiza o conjunto que o projeto guardou com as mesmas câmaras; um conjunto com outras câmaras é adicionado ao lado deste.
- **O próprio processo de gravação da aplicação nunca sobrescreve as definições ativas de um dispositivo** com a sua cópia mais antiga e mantém as substituições por membro guardadas por um chamador CLI ou SDK.
- **O formato de píxeis que definir é aquele que o projeto mantém.** Um formato definido nas Definições da Matriz, por `--pixel-format` ou por `connect_array(pixel_format=)`, incluindo o de 8 bits, é guardado e restabelecido exatamente. Se não for definido, cada câmara liga-se com a sua predefinição de 12 bits (`BayerRG12`, `Mono12` num sensor monocromático). O nome de um formato indica a sua profundidade de bits; a família corresponde à do sensor, pelo que uma câmara a cores nunca é definida para um formato monocromático, nem uma câmara monocromática para um formato Bayer, e a linha `[PixelFormat <serial>] ... written` indica isso quando um formato foi mapeado. As exportações de uma captura de 8-bits são de 8 bits, incluindo a refletância: redimensionadas e marcadas com o seu `PixelScale`.
- Um dispositivo ligado sem estar associado a nenhum projeto não grava nada.
- **Uma gravação segue um projeto criado posteriormente; ao ser aberta, mantém o seu projeto.**Quando um projeto é criado — um SDK `create_project()`, o `chloros-mcp` do `create_project`, ou o**Novo Projeto**da aplicação — ou duplicado (a opção**Duplicar**da aplicação, que abre a cópia) enquanto uma gravação ainda está a gravar no projeto que estava aberto anteriormente, a gravação continua e o que é gravado acaba por ir para o novo projeto, no mesmo local dentro deste (abaixo). Ao abrir um projeto existente — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, a opção**Abrir Projeto** da aplicação — deixa uma gravação ainda em curso no projeto em que está a ser gravada: termina ali e os seus ficheiros juntam-se à lista de ficheiros desse projeto (a aplicação interrompe todas as gravações antes de abrir um projeto).
  - `array-record` e `array-burst`: o vídeo, ou a pasta de sequência com todos os fotogramas, é transferido na íntegra para lá quando a gravação termina (o seu `--duration`, Ctrl+C). O `Burst dir` que é gerado é o novo, e o `--build` e o `array-build-video` funcionam a partir dele. Se a unidade recusar a transferência, a gravação permanece onde foi gravada; odo log indica o motivo, assim como a resposta HTTP da paragem (`relocation_error`). Os fotogramas brutos de uma sequência interrompida e o `.daq` são adicionados à lista de ficheiros do projeto em que acabam por ser incluídos; um vídeo (um `array-record`, um vídeo criado a partir de uma sequência) não é adicionado à lista de ficheiros de um projeto.
  - `array-capture --continuous`: todas as capturas após a alteração são guardadas no novo projeto; as que foram guardadas anteriormente permanecem onde estão.
  - `daq pool-record` e `project sensor log` (e a gravação do próprio sensor de luz da aplicação): o `.daq` é movido para a nova pasta do projetopasta `light_sensor` desse projeto quando a gravação termina e passa a fazer parte da lista de ficheiros desse projeto.

  Uma gravação numa pasta fora do projeto aberto (um `-o` noutro local) permanece onde está, e o `lattice continuous` e o `multi-stream` mantêm a pasta que lhes foi atribuída.
- **Na aplicação, as capturas efetuadas após a criação de um novo projeto são incluídas nesse projeto.** A criação ou duplicação de um projeto mantém todas as sessões de captura e gravações em curso, e cada captura efetuada a partir desse momento (Capturar Tudo, Iniciar Captura) é guardada no novo projeto e passa a fazer parte da sua lista de ficheiros. Ao abrir um projeto existente na aplicação, todas as sessões são primeiro interrompidas.
- **Um projeto não é movido enquanto estiver a gravar.**A tentativa de mover o projeto aberto para outro diretório de trabalho (em**Definições → Diretório de Trabalho** da aplicação; nenhum comando CLI move um projeto, mas um cliente HTTP que envie dados para `/api/move-project` sim) é recusado com o código HTTP 409 `PROJECT_RECORDING` enquanto uma gravação em matriz ou em rajada, uma captura contínua em matriz, um trabalho `lattice continuous` / `multi-stream` ou uma gravação do sensor de luz a gravar nele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` A aplicação recusa da mesma forma, antes de solicitar uma pasta, enquanto a sua própria função «Capturar Tudo», «Iniciar Captura» ou gravação do sensor de luz está em execução.

Isto é independente do estado em que as câmaras ficam no momento da desconexão (ver [Memória da câmara](#camera-memory----keep-settings----clear-settings)): um projeto configura as suas próprias câmaras quando é aberto.

### Memória da câmara — `--keep-settings` / `--clear-settings`

Quando um conjunto está ligado, as suas câmaras assumem as definições do conjunto, podendo o utilizador alterá-las posteriormente. Quando o Chloros liberta as câmaras — `array-disconnect`, o fim de um comando transitório (`multi-capture` sem matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), a paragem do `lattice continuous`, o fim de um `lattice capture` de uma câmara que não possui — repõe cada um tal como estava antes da ligação: a sua ligação do disparador e da linha de sincronização, as definições PTP e da câmara (a lista completa encontra-se em [Multicâmara (Sessões Transitórias)](#multi-camera-transient-sessions)).

Para utilizar as câmaras fora de um projeto, dois sinalizadores em qualquer um desses comandos gravam na **própria memória** da câmara — as configurações com que ela arranca, independentemente do software:

- **`--keep-settings`** guarda as definições de cada câmara tal como a sessão as deixou na sua memória (UserSet2, definido como conjunto de arranque). As câmaras continuam a funcionar de forma autónoma (`TriggerMode` `Off`), com a ligação da linha de sincronização tal como se encontra. Uma câmara sem UserSet2 guarda as definições no UserSet1; este computador memoriza-as, pelo que a autocorreção da matriz dChloros — o que redefine um UserSet1 que se desviou da linha de base do Chloros — não o altera.
- **`--clear-settings`** redefine a memória de cada câmara para as definições de fábrica: a linha de base do Chloros, aquela que a autocorreção da matriz incorpora — predefinições de fábrica com `TriggerMode`, `Off`, `TriggerSource` e `Line2`, exposição automática, o equilíbrio de brancos calibrado de uma câmara RGB mantido — no UserSet1, que constitui o conjunto de arranque.
- Os dois não podem ser combinados. Cada câmara recebe uma linha: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` ou `SN:…: camera memory reset to the factory settings.`; a resposta `camera_memory` é a mesma para cada câmara.

A aplicação não disponibiliza estes sinalizadores.

**Qualquer saída repõe as câmaras.**A mesma reposição tal como encontrada é executada em cada caminho de saída, não apenas nos comandos acima: `cam-disconnect` (e a câmara**X**da aplicação); o**X** da aplicação numa matriz ou na linha de qualquer membro da matriz (uma matriz não pode continuar a funcionar sem um membro, pelo que toda a matriz é libertada); ao fechar a aplicação (esta concede ao backend até 20 s para recolocar todas as câmaras, matriz e o `lattice continuous` em execução de volta antes de o interromper; um comando ainda em execução nessa altura — um comando transitório a abrir as suas câmaras ou à espera dos seus relógios PTP, um `lattice capture` de execução única, `power`, `white-balance` ou `presets --apply` — pára no seu próximo ponto seguro, restabelece as suas câmaras em primeiro lugar, durante um máximo de 8 s, e responde com um HTTP 503 `SHUTTING_DOWN`; uma ligação em série ainda em execução — `lattice array-connect`, a ligação da aplicação — pára antes da sua próxima câmara e fecha as câmaras que tinha aberto, durante um máximo de 10 s, e responde com HTTP 503 `SHUTTING_DOWN`; e uma que chegue durante o encerramento não abre nada: HTTP 503 `SHUTTING_DOWN`); e um comando CLI que seja interrompido ou cujo terminal feche (SIGTERM / SIGHUP interrompem a sua tarefa — e recoloca as suas câmaras no estado anterior — antes de sair (`143` / `129`, tal como Ctrl+C). Um `lattice continuous`, `multi-stream` ou `align-stream` encerrado imediatamente (SIGKILL), que não consegue interromper nada, tem a sua tarefa interrompida e as suas câmaras repostas pelo backend 30 s após o seu último registo. Um backend que entre em falha ou seja encerrado à força não consegue restabelecer nada; um reinício de energia devolve cada câmara à sua memória, e a próxima ligação do Chloros reconfigura qualquer câmara que encontre armada.**Um projeto configura as suas próprias câmaras.** Quando um conjunto se liga com um projeto aberto, cada membro recebe as definições que o projeto guardou para ele — uma exposição, ganho ou valor-alvo de AE mantido nele (considerado como seu, pelo que a exposição automática não o altera) e o seu modo de equilíbrio de brancos (correção do Chloros; o equilíbrio de brancos da própria câmara permanece desativado) — independentemente do que a memória da câmara contenha.

### Smart-AE / Smart-Capture

As matrizes LATTICE executam AE contínuo em segundo plano assim que são ligadas, mas uma cena recém-focalizada demora um momento a convergir. `array-capture --smart` é a **solução prática**: aguarda que a AE se estabilize em todas as câmaras do conjunto e, em seguida, aciona a captura. Utilize-a quando mudar de cena a meio de uma sessão.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

A política de estabilização é conservadora por predefinição: tempo limite de 5 s, janela de estabilidade de 1,5 s, tolerância de variação de exposição de ±5 %. Ajuste através do SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se precisar de um comportamento diferente da automação.

#### Como funciona a exposição automática

- **Primeiro a exposição, depois o ganho.** Com o ganho em «Auto», este permanece em 0 dB até que a exposição atinja o valor máximo permitido.
- **Uma exposição manual mantém o ganho em «Auto»** até que altere o ganho ou o defina para «Manual». De volta ao modo «Auto», o limite do ganho é o mesmo de antes.
- **Pequenas áreas de realce não escurecem a cena.** Um reflexo especular até 1 % do enquadramento é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5 % de um painel.
- **O limite máximo de exposição do DAQ aplica-se apenas enquanto um alvo de calibração estiver visível.** Impede que um painel branco sofra clipping; sem nenhum painel visível, a câmara expõe de acordo com a cena.
- **A luz constante mantém-se estável e a luz alterada estabiliza em segundos.** Com luz constante, a exposição e o seu limite permanecem inalterados e, após uma grande alteração na luz, a imagem estabiliza em poucos segundos. Um clipping já não limita a exposição de forma definitiva e uma alteração na área de interesse (ROI) restabelece a exposição.
- **O alvo de brilho permanece ao alcance.** Não aumenta enquanto a câmara estiver no seu limite de exposição e o seu ganho não puder subir, pelo que uma luz que regresse não provoca um reflexo no enquadramento.
- **Uma câmara cuja exposição automática deixa de responder é mantida no nível adequado à luz em que se encontra.** Sob sol intenso, uma câmara pode descer para a sua exposição mais curta e permanecer nesse estado com um quadro escuro, com todas as suas definições a indicarem valores normais. O Chloros monitoriza o enquadramento, mantém essa câmara no nível que manteve para aquela luz e liberta-a quando a luz muda. Um sensor de luz escuro ou coberto não perturba este processo.

### Níveis de captura e exportação (a predefinição do `all`)

A partir desta versão, `lattice capture`, `lattice multi-capture` e `lattice array-capture` **o valor predefinido é `--processing all`** — um ficheiro guardado por tipo de exportação que se aplica a cada câmara, correspondendo ao comportamento da opção «Capturar tudo» da interface gráfica. Os níveis são:

| Nível | Saída | Aplica-se a |
| --- | --- | --- |
| `raw` | Bayer de canal único (câmaras monocromáticas: a banda única) diretamente do sensor. | Todas as câmaras. |
| `debayered` | Cor de 3 canais, guardada na ordem de canais **RGB**(um ficheiro TIFF com interpretação fotométrica RGB; PNG/JPEG também abrem como RGB): o desvio de escuro (DSNU) do pacote de calibração é retirado do ficheiro bruto e, em seguida, é aplicado o demosaico sensível a bordas do OpenCV— sem LUT de linearidade, campo plano ou desmistura (câmaras monocromáticas: escala de cinzentos de 1 canal). Os fotogramas sem debayering na memória que o SDK Python devolve (`grab()`, `frame_stream()`) estão em**BGR**, a ordem do OpenCV; apenas o ficheiro guardado está em RGB. | Todas as câmaras. |
| `radiance` | float32 W/m²/sr/nm através da cadeia radiométrica completa. | Apenas multiespectral (M3C/M3M) — **ignorado para câmaras com filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), compatível com Pix4D. | Apenas multiespectral, e **apenas quando um DAQ estiver associado + a câmara estiver calibrada**; caso contrário, ignorado. |
| `display` | Cadeia completa de pré-visualização na GUI (CCM + WB + gama de acordo com o perfil da câmara). Guardada numa pasta `preview/`. O `lattice capture` também aceita o nome `preview` para a mesma; `array-capture` / `multi-capture` aceitam apenas `display`. | Todas as câmaras. |

Indique um único nível para guardar apenas esse (`--processing debayered`). Quando se solicita o `all`, os níveis que não se aplicam a uma determinada came são ignorados (e são indicados), não gerando erros — uma câmara não ligada ou não calibrada continua a receber `raw` / `debayered` / `preview`.

#### A leitura do DAQ que uma captura guarda

Sempre que um DAQ está associado à câmara ou ao conjunto de câmaras, ou incluído num conjunto, a leitura descendente utilizada numa captura é gravada num ficheiro **`.daq`** na subpasta `daq/` da pasta de capturas, com o seu `.match.json` (a distância a que a leitura se situava em relação à exposição) e o seu `.cameras.json` (a quem pertence a leitura), e é relatada numa linha `DAQ:`. Isto aplica-se a todos os níveis, incluindo capturas em formato bruto ou sem bayering, pelo que qualquer captura pode ser posteriormente reprocessada em radianância, refletância ou um índice. Trata-se do mesmo ficheiro, com o mesmo nome e ficheiros auxiliares, quer a captura tenha sido efetuada a partir da GUI, da CLI ou do SDK. Na ausência total de uma leitura DAQ, não existe um `.daq`, e um `multi-capture` sem matriz associada (o recurso de fallback transitório) não guarda nada.

| Sinalizador | Efeito |
| --- | --- |
| `--no-daq` | Não guardar a leitura numa captura cujos níveis não a utilizem (`raw`, `debayered`). Um nível `radiance`, `reflectance` ou `display` continua a gravá-la, porque a reimportação necessita da leitura com a qual o produto foi fabricado. |
| `--daq-tolerance-ms MS` | A que distância da exposição a leitura pode estar e ainda ser considerada dentro da tolerância (padrão 500). Apenas o veredicto muda (o `.match.json` do `in_tolerance` e do `tolerance_ms`, o `DlsInTolerance` da refletância), nunca a leitura utilizada. Ver [Uma leitura DAQ distante da exposição](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | A forma mais antiga de solicitar o `.daq` numa captura apenas em formato bruto. É agora a predefinição, e o `lattice capture` e o `array-capture` ainda aceitam o sinalizador (o `multi-capture` não aceita). O `--no-daq` tem prioridade sobre este. |

Qual DAQ: aquele associado à câmara ou ao conjunto de sensores (a seleção do sensor de luz na GUI, uma associação guardada num projeto ou o `chloros-cli daq pool-bind`), ou então o mais recente que o backend tiver. Uma sequência de imagens em formato raw (`array-burst`) mantém o seu próprio `.daq` por cada leitura distinta na pasta da sequência, tal como anteriormente.

**Os formatos PNG e JPEG contêm 8 bits por canal, pelo que utilizam apenas os ficheiros `display` / `preview` e os produtos de índice.** Os ficheiros `--format png` ou `jpg` com um nível que não conseguem suportar são rejeitados antes de qualquer captura, indicando o nível e o formato TIFF que o suporta:

- `radiance` (float32) e `reflectance` (uint16, `32768` = 1,0) — sempre, pelo que o `--processing all` (o valor por predefinição) com o `--format png`/`jpg` também é rejeitado: utilize `--format tiff` ou `--levels display`.
- `raw` e `debayered` provenientes de uma câmara que transmite mais de 8 bits (os quadros M3C de 12 bits / Mono12 M3M) — a CLI não consegue detetar a profundidade de bits da câmara, pelo que o backend rejeita estes (HTTP 400). O único`lattice capture --processing debayered` de quadro único capta um quadro de 8 bits para PNG/JPEG e não é rejeitado, tal como um `multi-capture` sem uma matriz persistente (desbayering de 8 bits).

Todas as rotas de captura respondem da mesma forma (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, e as rotas HTTP por trás delas.

### Como é uma pasta de capturas

Cada tipo de exportação é guardado na sua **própria subpasta**dentro de `-o`, pelo que uma captura com vários níveis nunca mistura tipos. O esboço mostra a**captura predefinida `all` de uma câmara pelo `lattice capture`** — vários níveis por câmara, pelo que cada ficheiro tem o sufixo `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Como os nomes dos ficheiros variam:

- **Campo da câmara — número de série, seguido do filtro:** cada ficheiro de captura indica a câmara que o captou como `SN<serial>-<filter>`, em que `<filter>` é a parte do filtro do modelo da câmara: `SN261902142-F450` para uma `M3M-L87-F450`, `SN213800234-FRGN` para uma `M3C-L87-FRGN`, `SN214701292-FRGB` para uma câmara RGB. O filtro junta-se ao número de série como `-`, e não com o `_`; assim, um nome apresenta os mesmos campos separados por «`_`» que anteriormente (prefixo, data, hora, milissegundos, câmara, nível). Uma câmara cujo modelo não possa ser lido (um DeviceUserID apagado) é nomeada apenas como `SN<serial>`. As capturas de versões anteriores não têm filtro no nome; o Chloros lê ambas.
- **Prefixo:** `lattice capture` nomeia os ficheiros `capture_…`; `array-capture`, `multi-capture` com uma matriz associada, e `project capture` / `burst` / `stream` de uma matriz nomeia-os `sync_<ts>_SN<serial>-<filter>…`.
- **Sufixo de nível apenas com vários níveis.**Uma câmara que guarda**vários**níveis numa única captura (`all` ou `--levels a,b`) acrescenta `_<level>`. Uma câmara que guarda**um** nível mantém o nome simples, sem sufixo — o `--processing reflectance` grava como `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — para que os scripts escritos para capturas de nível único continuem a funcionar. O nível continua a ser indicado pela pasta.
- **As máscaras de limitação seguem o nome do produto:** `<product name>_clampmask.tif`. Uma captura de refletância de nível único grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; uma captura `all` grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **A extensão** segue `--format`: o valor predefinido `tiff` resulta em `.tiff`, `--format tif` resulta em `.tif` (as máscaras de limitação são sempre `.tif`).
- **Uma exceção:** o `lattice capture --processing debayered` (ou `--levels debayered`) recebe um únicoe grava `capture_<ts>_SN<serial>-<filter>.tiff` diretamente em `-o` — sem subpasta.
- **Fluxos e rajadas** transportam o mesmo campo: `lattice continuous` grava `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` e uma rajada em formato bruto `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` é o carimbo de data/hora da captura e `<serial>` o número de série da câmara, pelo que um grupo sincronizado partilha um
carimbo de data/hora entre as câmaras. **Note-se a única assimetria:** o nível `display` é armazenado numa pasta
denominada `preview/`, enquanto os ficheiros multinível mantêm `_display` no nome — a pasta e o sufixo diferem
apenas para esse nível. Os níveis desconhecidos recorrem a uma pasta com o seu próprio nome e, se a subpasta
não puder ser criada, o ficheiro é gravado na raiz de saída, em vez de se perder.

**Reprocessamento de uma pasta de capturas:**aponte o `chloros-cli process` para a**raiz das capturas**
(`output/`). O `process` normalmente importa apenas a pasta que indicar, mas quando essa pasta não contém
imagens e tiver subpastas, percorre-as automaticamente — pelo que as subpastas de nível e a
subpasta `daq/` são todas recolhidas de uma só vez. Cada nível de uma captura é importado como uma única imagem, com
os outros níveis disponíveis como modos, em vez de uma imagem por nível.

Nomear diretamente uma **subpasta de nível** (por exemplo, `output/raw/`) também funciona. Ao fazê-lo, a pasta `daq/`
fica para trás, pelo que deve copiar ou indicar a leitura do DAQ juntamente com ela quandoderivar um produto radiométrico
a partir de `raw/` — caso contrário, a correspondência do carimbo de data/hora não terá nada com que se relacionar.

**O processamento começa sempre a partir de `raw`.** Dentro de cada captura, o fotograma bruto é a fonte do pipeline;
`debayered`, `radiance`, `reflectance` e `preview` surgem como modos visualizáveis, mas nunca são
reencaminhados através do pipeline. O reprocessamento de um produto derivado reaplicaria a vinheta, o CCM e
os cálculos de radiação que já estão incorporados nos seus píxeis, pelo que o Chloross opta por recusar em vez de
efectuar um duplo processamento. Duas consequências que vale a pena conhecer:

- As renderizações `index/` e `composite/` **nunca** processadas. São saídas, não capturas —
  uma renderização LUT de NDVI não tem uma interpretação significativa da radiação.
- Uma pasta de capturas exportada **sem** o `raw` (por exemplo, `array-capture --processing reflectance`) não tem
  nenhuma fonte válida no pipeline. Essas capturas importam-se e exibem-se normalmente, mas o `process` ignora-as
  e indica isso:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se precisar realmente de fazer passar um produto derivado — uma sessão de hub capturada com
  o `demosaic` ativado, ou uma pasta legada — o `--input-level {raw,debayered,processed}` força o ponto de entrada
  e anula a omissão. Esse sinalizador é a via de escape deliberada; o `auto` (o valor predefinido)
  nunca processa uma captura que não tenha dados brutos.

**Etiquetas de calibração: apenas no `raw`.** O ficheiro `raw` de uma captura contém o pacote de calibração da câmara: a carga útil `CalibrationJSON` e as etiquetas compatíveis com MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que descrevem os valores em bruto do sensor. Todos os restantes ficheiros de nível (`debayered`, `display`, `radiance`, `reflectance`, a renderização `index/`) e cada quadro `lattice continuous` / `multi-stream` contém a identidade da câmara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e o próprio registo do fotograma, `XMP-mapir:FrameJSON` (exposição, ganho, temperatura, formato de píxeis, hora de captura, correspondência DAQ), uma vez que cada produto `process` contém essa identificação. O `process` localiza a calibração desse ficheiro através do seu número de série e o `CalBundleSha256` no cache de calibração do Chloros, descarregando o pacote uma única vez caso o computador não o tiver.

#### Ficheiros criados a partir de várias câmaras

Um ficheiro criado a partir das bandas de mais do que uma câmara — uma pilha multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), um índice em tempo real criado a partir de várias câmaras (`index.tif`, `index_LUT.png`), uma imagem de captura em matriz `composite/` — indica todas as câmaras que contribuíram para a sua criação:

- **Listas por câmara, por ordem de banda** (a composição: camada de fundo, seguida da camada de primeiro plano; uma camada de índice, com as câmaras de onde provêm as suas bandas): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (o pacote de calibração pelo qual as faixas de cada câmara passaram; caso contrário, — uma pilha `align-apply` / `align-stream`, cujas bandas não passam por nenhuma — o pacote da câmara que o Chloros armazenou em cache, aquele cujo nome corresponde aos seus ficheiros de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, sendo que o item *n* de cada uma descreve a mesma câmara; `-`, onde uma câmara não tem valor (sem pacote de calibração, uma lente desconhecida). Cada câmara corresponde a **um** item: quando o seu pacote tem vários valores para um campo — os certificados de lâmpadas halógenas de um pacote —, estes são unidos com `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca com uma vírgula, pelo que uma lista apresenta um item por câmara, independentemente da forma como for impressa.
- **EXIF `SerialNumber`** contém os mesmos números de série, separados por vírgulas (`214000535,214000551`).
- Não contém **nenhuma** identidade de câmara individual (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): esses identificam uma câmara e o seu conjunto, e o `process` determinaria a calibração do ficheiro a partir deles. Um índice calculado a partir das bandas de várias câmaras (`index.tif`, `index_LUT.png`) contém EXIF `Model` `LATTICE` (o conjunto), e não o modelo de uma única câmara `LATT-…`.

Um ficheiro com as bandas de **uma** câmara — um ficheiro TIFF por banda (`--per-band`), um fotograma `align-apply --per-camera`, uma pilha ou índice em tempo real de uma única câmara — contém as etiquetas de identidade próprias dessa câmara, tal como qualquer outro ficheiro de captura. A pilha do `align-apply`, os ficheiros por banda e por câmara, bem como cada pilha do `align-stream` e os seus ficheiros por banda, contêm a hora de captura do seu grupo (a exposição da câmara de referência, UTC), e um quadro `--per-camera` de uma câmara mono é um plano, tal como a pilha `--bit-depth`.**Orientação da câmara (inverter / rodar).** A configuração de orientação de uma câmara (as opções «Mirror H» /
Mirror V / rotação) da interface gráfica determina o que é guardado numa captura:

- **`raw` e `debayered` são guardados exatamente como o sensor os leu**, independentemente da configuração. Um
  mosaico de Bayer virado ou invertido representaria um padrão de filtro de cor diferente, e os mapas de calibração
  (desvio escuro, campo plano) estão na própria grelha do sensor.
- **`preview`/`display`, `radiance`, `reflectance` e a renderização `index/` estão orientados** — rodados
  uma vez, após todos os cálculos de calibração, para que se abram da forma como a câmara foi enquadrada. Os seus
  `clamp_masks/…_clampmask.tif` encontram-se na mesma grelha.
- **Cada ficheiro de captura regista a configuração** em XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, no sentido horário, após
  as inversões) e `ChlorosOrientationApplied` — `false` em imagens em bruto e sem debayering (píxeis nativos),
  `true` nos produtos orientados. `ChlorosProcessing` lista as operações aplicadas aos
  pixéis desse ficheiro (`none` quando nativo). Isso inclui todos os fotogramas que o `lattice continuous` transmite (a
  própria configuração da câmara, não aplicada) e todos os fotogramas do `lattice multi-stream` (a identidade: os seus
  fotogramas constituintes são nativos, tal como numa captura em matriz). As pilhas `lattice align-stream` / `align-apply`
  registam a configuração da câmara principal, não aplicada: as suas bandas são distorcidas para o quadro da câmara principal,
  mas não são rodadas.
- **O `process` aplica a orientação registada** quando processa um ficheiro bruto: desvio de escuridão, campo plano, desmosaico e separação de cores nas coordenadas do sensor em primeiro lugar, a distorção de alinhamento da matriz em seguida e, por último, a
  orientação por último — para que os produtos de pré-visualização, radiação, refletância e índice resultem exatamente como
  os da captura em tempo real, e registem o `ChlorosOrientationApplied=true`. Os produtos de recurso não calibrados
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) são orientados da mesma forma, após
  o campo plano. O produto sem debayering mantém-se no formato nativo.
- **O visualizador de imagens Chloros mostra um ficheiro bruto ou sem debayering rodado de acordo com o seu registo** (apenas para visualização —
  o ficheiro não é alterado), para que se pareça com os seus produtos; a miniatura, a visualização em tamanho real, a
  leitura do cursor e a área de teste do Índice/LUT utilizam todos a mesma rotação. Um ficheiro sem registo e
  um produto (já orientado) são apresentados tal como estão armazenados. Uma exportação para a área de teste regista a orientação que os seus
  píxeis apresentam (`ChlorosOrientationApplied=true`; a identidade para uma camada sem registo).
- **As capturas sem o registo são processadas como nativas** (comportamento inalterado). Antes da versão 1.3.0, uma
  captura com uma única câmara com uma inversão ou rotação definida rodava ela própria os seus ficheiros RAW e debayered;
  o Chloros não consegue distinguir um ficheiro RAW já- a partir de um nativo e não faz suposições, pelo que
  o processa tal como foi guardado. Os mapas de calibração encontram-se na própria grelha do sensor; por isso, para uma
  captura deste tipo, utilize uma feita com a versão 1.3.0 quando a sua calibração for relevante.
- **As matrizes não rodam nenhum ficheiro membro por predefinição** (a distorção de alinhamento e o ficheiro XMP `Alignment*` encontram-se nos
  próprios fotogramas das câmaras, sem rotação) e registam a identidade. O `array-capture --orient` gira os produtos de cada
  membro, após a transformação de alinhamento: de acordo com a orientação do **mestre** quando foram
  transformados para o seu quadro (para que os membros permaneçam co-registados), ou de acordo com a orientação da sua própria câmara. Cada
  ficheiro regista a orientação do quadro em que os seus próprios píxeis se encontram — um ficheiro bruto, da sua própria câmara —
  e um ficheiro bruto que contém a transformação de alinhamento também regista `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: a do mestre, para um produto alinhado
  para o quadro da câmara principal. O `process` gira cada produto de acordo com o que pertence aos seus píxeis:
  o alinhado quando distorce o ficheiro raw (`--array-alignment`, o valor predefinido — os produtos da captura alinhada em tempo real), o próprio da câmara quando não o faz (`--no-array-alignment` — os da captura em tempo real
  não alinhada). A mesma opção existe para `array-burst`, `project capture` / `burst` /
  `stream`, receitas e o SDK.

### Capturas ignoradas em matrizes com filtros mistos

Quando se misturam câmaras RGB e multiespectrais numa matriz, o `array-capture --processing radiance` (ou `reflectance`) guarda os fotogramas multiespectrais e **ignora** as câmaras RGB — a radiação por pixel Bayer não é significativa para um sensor de banda larga. A CLI apresenta explicitamente cada ficheiro guardado (com o seu nível de exportação), cada `.daq` gravado e cada omissão, pelo que o número de ficheiros não é surpreendente:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Os tokens de motivo de omissão seguem o padrão `<level>-not-applicable-to-rgb-cam`. Uma câmara cujo pacote de calibração não esteja carregado omite `radiance` / `reflectance` com `<level>-needs-calibration (connect the camera so its cal pack loads)`, e, na ausência de uma leitura de refletância utilizável do DAQ, salta com `reflectance-skipped-no-dls`. A refletância também pode ser ignorada com `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, e com o `dls-uncalibrated-band-<nm>` quando a banda se situa maioritariamente fora da gama radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm) — entre os SKUs disponíveis, apenas o F988, cujo percurso suportado é o fluxo de trabalho do painel de refletância.

Utilize o `--processing debayered` (ou o `display`) para incluir todas as câmaras, independentemente do tipo de filtro, ou o `all` predefinido para obter todos os níveis aplicáveis por câmara de uma só vez.

### Uma leitura DAQ distante da exposição

Cada `.daq` que uma captura guarda corresponde à leitura DAQ associada a essa exposição, e o `.match.json` ao lado regista a distância entre as duas (`gap_ms`) e se essa diferença se encontra dentro da tolerância (`in_tolerance`, 500 ms, a menos que a captura tenha definido `--daq-tolerance-ms`). Uma leitura fora desse intervalo continua a ser utilizada, pelo que nenhum fotograma é perdido, e é identificada: o ficheiro XMP da refletância contém os valores `DlsInTolerance=False` e `DlsMatchGapS`. `lattice capture`, `multi-capture` e `array-capture` também a imprimem sob o `.daq` a que diz respeito, com aquando se explica o seguinte:

**Como uma leitura é associada a uma exposição.** Uma captura aguarda a primeira leitura do DAQ após a exposição, para que possa combinar as leituras de ambos os lados da mesma (`method` `interpolated`). Um DAQ-E envia uma leitura a cada 0,4 s à luz do dia e a cada 1-2 s em ambientes interiores, pelo que a espera acrescenta cerca de um intervalo e meio desses ao comando, e não à exposição. Não se aguarda um DAQ que tenha deixado de enviar. Uma leitura do DAQ-E é a média da luz ao longo do intervalo desde a sua leitura anterior, pelo que é colocada no meio desse intervalo: `gap_ms` é medido a partir da exposição até esse ponto médio, e `bracket_stamps_ns` fornece os próprios carimbos de data/hora das leituras. `array-capture --continuous` aguarda sem abrandar a captura: os seus ficheiros brutos são gravados imediatamente e os seus `.daq` assim que a leitura chega. Com o `--interval`, cada passagem aguarda como se fosse uma única captura.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Quando não foi gravado nenhum ficheiro de refletância a partir dessa leitura, a primeira linha termina com `saved anyway; its .match.json says in_tolerance: false`.

A captura ainda termina com `0`: todos os ficheiros foram gravados e indicam o seu conteúdo. Um DAQ cujo fluxo tenha ficado bloqueado continua a fornecer a sua última leitura; por isso, verifique-o com `chloros-cli daq pool-list` antes de confiar na refletância dessa captura. O backend restabelece a ligação a um DAQ-E em pool que deixou de fornecer dados (ver [Um DAQ-E que deixa de fornecer dados](#a-daq-e-that-stops-delivering)). Uma leitura fora da tolerância apresenta apenas a sua linha `DAQ:`, a menos que seja uma leitura mantida (abaixo).

**Uma leitura mantida após o seu DAQ ter deixado de estar disponível.** Quando nenhum DAQ pode fornecer uma leitura para uma exposição (o DAQ vinculado está indisponível ou nenhum DAQ em pool tem uma, por exemplo, após `daq pool-disconnect`), a captura reutiliza a última leitura que o Chloros tinha, e o seu `.match.json` descreve essa leitura em relação a esta exposição: `method`, `retained`, `gap_ms` desta exposição para as amostras a partir das quais a leitura foi efetuada, o `in_tolerance` desse intervalo e o `retained_from_epoch_ns`, a exposição à qual a leitura foi inicialmente associada. Uma captura efetuada muito tempo depois de o DAQ ter sido desligado imprime, portanto, as linhas acima e, por baixo delas mais uma, que é impressa mesmo enquanto a leitura ainda se encontra dentro da tolerância:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

O seu `.daq` continua a indicar o DAQ que efetuou a leitura, juntamente com a calibração e a tampa sob as quais essa leitura foi efetuada, pelo que uma reimportação sabe qual o DAQ, a calibração e a tampa que a produziram.

### Ficheiros que não conseguem ser gravados

Uma câmara cujo ficheiro não possa ser gravado — devido a um disco cheio ou a um erro de permissões — não impede que os ficheiros das outras câmaras sejam guardados. Cada comando de captura (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; e os `capture`, `burst` e `stream` de uma receita) imprime cada ficheiro com falha no stderr, e, em seguida, **encerra o `1`**, mesmo que o resto tenha sido guardado:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Todos os ficheiros a que uma captura dá origem são abrangidos, incluindo o único`lattice capture --levels debayered` com um único fotograma submetido a debayering. Uma leitura `.daq` cujo ficheiro de proveniência sidecar `.match.json` não pôde ser gravado é listada como `[match]` (o próprio `.daq` foi guardado).

Num disco cheio, uma captura repetida (`array-capture --continuous` / `--interval`, `project burst` / `stream`, uma receita `burst` / `stream`) **pára nessa passagem** em vez de não gravar nada na mesma unidade durante o resto da execução. Qualquer outra falha é comunicada em cada passagem em que ocorre, e a execução prossegue até ao fim, terminando depois em `1`. `array-capture --continuous` (a captura contínua do backend) não lista nenhum ficheiro, incluindo os que falharam: conta-os nas suas linhas de progresso e resumo (`N failed`, `N file(s) failed to write.`) e sai do `1` — ver [Modo contínuo](#continuous-mode---continuous).

Os gravadores — `array-burst` e `array-record` — comunicam as suas próprias falhas. O `array-burst` interrompe-se automaticamente quando a unidade fica cheia, ou imediatamente antes disso. Ambos continuam a funcionar mesmo após qualquer outra falha de gravação. Quando qualquer um deles comunica um erro, este é apresentado após os contadores, e o comando **sai do `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` em vez de `Disk full:` significa que a sequência foi interrompida antes de encher a unidade. Os quadros gravados antes da paragem são mantidos. O `array-burst --build` ignora a compilação quando o espaço se esgota, porque os vídeos iriam para a mesma unidade. Liberte espaço primeiro e, em seguida, execute o `array-build-video --burst-dir …`. Após qualquer outro erro, a compilação continua a ser executada.

Um `array-record` que não gravou nenhum fotograma também é considerado uma gravação falhada (`Recorder error: no-frames: the combined view published no frame to record …`, sair do `1`) — ver [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Modos de captura, gravadores e reprocessamento offline

Todos estes operam numa **matriz persistente** (execute primeiro o `array-connect`). Espelham o painel de captura da GUI.

### Modos do `array-capture`

O `array-capture` é um único comando com quatro modos de obturador, além de um conjunto de opções de exportação:

| Modo | Sinalizador | Comportamento |
| --- | --- | --- |
| **Único** *(predefinição)* | (nenhum) | Um grupo de captura sincronizado e, em seguida, sair. |
| **Contínuo** | `--continuous` | Capturas consecutivas até que o `Ctrl+C`, o `--count N` capture, `--duration S` ou até que a unidade esteja cheia. Executa a captura contínua própria do backend (o modo Contínuo da GUI) — consulte [Modo Contínuo](#continuous-mode---continuous) abaixo. |
| **Intervalo** | `--interval S` | Uma passagem a cada `S` segundos (medidos a partir do início de cada passagem), com os mesmos limites. |
| **Mais rápido** | `--fastest` | Apenas dados brutos+ a leitura DAQ atribuída + o índice composto combinado; ignora os cálculos de radianância/refletância/visualização para que o fotograma seja apresentado rapidamente. Implica `--processing raw --force-daq`. Reprocesse posteriormente o `.daq` guardado em produtos calibrados. |

Opções de exportação (combináveis com qualquer modo; todas partilham o ponto final da GUI/SDK):

| Indicador | Efeito |
| --- | --- |
| `--array-id ID` | A matriz a partir da qual se deve capturar (omitir quando apenas uma estiver ligada). |
| `-o, --output DIR` | Pasta de saída (por predefinição `output`, na pasta a partir da qual executa o comando). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formato de ficheiro (padrão: `tiff`). PNG/JPEG aceitam apenas `display` e a renderização do índice; consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Nível único de exportação (`raw`, `debayered`, `radiance`, `reflectance`, `display`), ou `all` (padrão). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportação (por exemplo, `raw,radiance,reflectance`); **substitui `--processing`**. |
| `--aligned` / `--no-aligned` | **Predefinição: alinhado, tal como a captura da mesma matriz pela aplicação Chloros**(a sua definição de captura**Alinhado** por matriz, ativada a menos que a desative). A exportação não bruta de cada membro (`debayered`, `display`, `radiance`, `reflectance`, a renderização do índice) é distorcida para o enquadramento da câmara principal e recortada para a sobreposição comum dos membros, de modo que os ficheiros ficam alinhados ao nível dos píxeis e apresentam o tamanho, o recorte e as etiquetas `Alignment*` iguais aos da aplicação — menores do que o sensor (ou da ROI) em função dos deslocamentos dos membros; no caso de uma ROI, a transformação é recalibrada com base na janela. `XMP-mapir:AlignmentJSON` indica `"applied": true` e fornece o `crop_roi`. O ficheiro bruto permanece sem distorção no tamanho nativo, mas contém a transformação nos metadados (`"applied": false`). O `--no-aligned` guarda todos os ficheiros no tamanho nativo, sem distorção. O `--aligned` é a configuração predefinida. Independentemente de a distorção ser aplicada ou não, cada ficheiro membro regista a transformação `Alignment*` da sua câmara assim que a matriz tiver um perfil de alinhamento — exceto uma câmara que o perfil não consiga alinhar (excluída ou forçada à identidade devido a uma resolução falhada) e a câmara de referência quando nenhuma outra câmara estiver alinhada: estas não contêm etiquetas `Alignment*`. Na ausência de perfil na matriz, a captura é guardada sem alinhamento, e a CLI apresenta o motivo numa linha `Alignment:`, tal como faz em todos os outros casos em que a distorção não pôde ser aplicada (elementos com resoluções diferentes, uma matriz não sincronizada por cabo, um fotograma no qual o perfil não pode ser mapeado). O alinhamento automático da matriz cria o perfil assim que a exposição se estabiliza; `align-calibrate` cria um a pedido. |
| `--orient` | Transforme os **produtos**de cada membro — `display`, `radiance`, `reflectance`, a renderização `index/` e o `composite/` — pela orientação da câmara, após a distorção de alinhamento: um produto distorcido para o quadro do mestre (por predefinição) pela**orientação do master**, para que os membros permaneçam co-registados; um não alinhado (`--no-aligned`, ou uma captura à qual a distorção não pôde ser aplicada) pela orientação da sua própria câmara.**Predefinição: desativado — todos os ficheiros membros permanecem nativos**, tal como sempre acontece com as capturas em matriz. `raw` e `debayered` permanecem na forma nativa em ambos os casos. Cada ficheiro regista a orientação que os seus produtos apresentam (ver [Orientação da câmara](#como-é-uma-pasta-de-capturas)); o ficheiro XMP do `Alignment*` mantém a transformação do fotograma nativo, aplicada antes da orientação, pelo que o `process` (distorção, seguida da orientação) reproduz o produto em tempo real. |
| `--index` / `--no-index` | Guardar / ignorar a sobreposição do índice de vegetação por câmara, caso esteja configurada. Predefinição: renderizá-la. |
| `--no-daq` | Ignora o ficheiro «sidecar» `.daq` quando nenhum nível selecionado utiliza o DAQ (uma captura apenas em formato bruto). Por predefinição, a leitura é guardada, para que os fotogramas possam ser reprocessados em refletância/índice offline. O `--force-daq` continua a ser aceite (predefinição). |
| `--daq-tolerance-ms MS` | A tolerância de correspondência do DAQ para estas capturas (padrão 500 ms). |
| `--smart` | Aguardar que o AE se estabilize em todas as câmaras antes do disparo (ver [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compressão de píxeis TIFF. `none` (padrão desde a versão 1.3.0; a versão 1.2.0 gravava `deflate`) = sem compressão — os píxeis do sensor exatamente como são lidos, ~6,3 MB por fotograma em resolução total e a taxa sustentada mais rápida; `deflate` = zlib L1 sem perdas + preditor horizontal, ~4,1 MB por fotograma, mais lento. Ambos são sem perdas e são lidos de forma idêntica na importação. |

> **TIFF de gravação única + o modelo de taxa sustentada.**Cada ficheiro é gravado**uma vez**: píxeis, o pacote XMP, IFD0 Marca/Modelo/Número de Série da Câmara e as etiquetas EXIF `ExposureTime` / `SerialNumber` numa única passagem, sem que nada o reescreva posteriormente. `--continuous` captura grupos sincronizados consecutivos — cada captura recebe o grupo seguinte ao anterior, incluindo um publicado enquanto essa captura ainda estava a ser guardada — pelo que funciona à velocidade da matriz. O DEFLATE retém o GIL do Python, pelo que as gravações comprimidas**não**são paralelizadas entre os threads de gravação de cada câmara; a captura sustentada em resolução total à taxa do sensor requer gravações não comprimidas (a predefinição)**e** um disco de classe NVMe (~600 MB/s sustentados para um conjunto de 9 câmaras a ~10 fps). O mesmo parâmetro é exposto como `compression` em `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Modo contínuo (`--continuous`)

O `array-capture --continuous` (sem o `--interval` ou o `--smart`) passa o comando para a **própria captura contínua do backend** — a sessão em que o modo Contínuo da GUI é executado. O backend guarda cada grupo sincronizado por si próprio, com duas capturas em curso e sem qualquer pedido da CLI entre grupos, pelo que acompanha um conjunto rápido em que um pedido por captura fica para trás (medido em 73–77 capturas/s num conjunto de 9 câmaras com uma ROI de 512 píxeis). Cada opção de exportação acima aplica-se a cada captura.

- **Limites:** o `--count N` pára após N capturas (uma captura = um grupo sincronizado: os ficheiros de cada membro); o `--duration S` após S segundos. Na ausência de qualquer uma destas condições, o programa continua a funcionar até ao `Ctrl+C`, até o disco ficar cheio, ocorrer um erro ou a matriz deixar de estar disponível. O `Ctrl+C` encerra a sessão do backend, depois apresenta o resumo.
- **Progresso** a cada 2 s, com `, N failed` e `, N groups skipped` acrescentados quando o valor for diferente de zero (um grupo ignorado é aquele que a matriz publicou e para o qual não houve captura):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Sem listagem por ficheiro.** A execução termina com uma linha de resumo — capturas, taxa, ficheiros e a pasta de saída — seguida de `N file(s) failed to write.` / `N group(s) skipped.` quando o valor for diferente de zero:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Interrompe** quando o disco fica cheio com `Continuous capture stopped: the drive is full.`, e em caso de erro ou se a matriz for removida com `Continuous capture stopped: <reason>.` (ambos no stderr).
- **Estado de saída:** `0` quando as capturas foram guardadas e todos os ficheiros foram gravados, incluindo o `Ctrl+C`; `1` se houver algum ficheiro cuja gravação falhou, uma unidade cheia, um erro, a matriz a desaparecer ou na ausência total de capturas.
- **Microsoft Defender:** quando o Defender utiliza um núcleo da CPU ou mais para analisar os ficheiros que a captura grava, a execução apresenta uma vez, no stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Consulte as [Configurações do sistema](#windows-exe).
- Uma captura contínua por matriz de cada vez; uma segunda é recusada com o código `a continuous capture is already running for this array`. Através de HTTP, a sessão é `POST /api/camera/array/continuous/start` (o corpo `/api/camera/array/capture` mais `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` e `POST /api/camera/array/continuous/stop`.

**Intervalo**(`--interval S`, com ou sem `--continuous`) e**`--continuous --smart`** mantêm o ciclo por pedido: um pedido de captura por passagem — em cada momento agendado para o Intervalo, após a estabilização do AE para o `--smart` — com os ficheiros de cada passagem listados como no modo Único, uma linha `Done — N capture pass(es).` no final e o `--count N` a contar as passagens.

### `array-record` — vídeo/GIF com índice combinado (nível de monitorização)

Grava a **visualização em tempo real do índice combinado** da matriz num `.avi` (e, opcionalmente, num `.gif`). Com a visualização combinada da GUI aberta, grava essa visualização tal como é apresentada. Se nenhuma estiver aberta, o backend renderiza a própria visualização: o índice que passar com o `--expr` ou, por predefinição, o NDVI da matriz, assim que esta estiver alinhada. O NDVI predefinido utiliza as bandas vermelha e NIR mais próximas de 660 nm e 850 nm, tal como acontece na visualização em tempo real. Uma visualização da GUI aberta durante a gravação é gravada enquanto permanecer aberta. Verifica o progresso a cada 2 s e pára em `--duration`, `Ctrl+C`, ou quando o gravador termina automaticamente.

Uma gravação que não escreve nenhum fotograma falha: o gravador reporta `no-frames: …` (a matriz pode ainda estar a alinhar-se ou a estabilizar a sua exposição) e o comando **encerra com `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino (omitir se apenas uma estiver ligada). |
| `-o, --output DIR` | `output` | Diretório de saída. Um caminho relativo está na pasta a partir da qual executar o comando. |
| `--fps F` | `10` | Taxa de fotogramas de gravação. |
| `--duration S` | até Ctrl+C | Paragem automática após `S` segundos. |
| `--gif` | desativado | Gravar também um GIF animado. |
| `--gif-only` | desativado | Gravar apenas um GIF (sem `.avi`). |
| `--expr EXPR` | o NDVI da matriz | Expressão de índice a registar quando nenhuma vista da GUI estiver aberta, nos nomes das bandas da matriz (por exemplo, `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; ver [nomes das bandas](#index--vegetation-maths)). Um nome de cor curto (`NIR`, `Red`) só funciona quando uma banda da matriz o possui. Em vez disso, uma vista da GUI que esteja aberta é registada tal como é apresentada. |

### `array-burst` — sequência de imagens raw-Bayer de alta taxa de fotogramas (nível de análise)

Lê diretamente o buffer do grupo sincronizado do ciclo de captura — **sem necessidade de cadeia de calibração, exiftool ou visualização em tempo real** — pelo que funciona à taxa máxima de captura da câmara. Grava fotogramas RAW + um manifesto por fotograma + um `.daq` por cada leitura DLS distinta sob o `<output>/bursts/<base>/`. Reprocesse offline (próximo comando) ou passe o `--build` para o fazer imediatamente ao parar.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Diretório de saída (os dados em rajada são colocados em `<DIR>/bursts/<base>/`). |
| `--duration S` | até Ctrl+C | Paragem automática após `S` segundos. |
| `--max-frames N` | ilimitado | Paragem automáticaapós `N` fotogramas brutos. |
| `--orient` | desativado | Os produtos gerados por uma compilação a partir desta sequência são orientados de acordo com a orientação da câmara — os produtos por câmara são orientados pela orientação da sua própria câmara, o índice combinado pela orientação da câmara principal — tal como faz o `array-capture --orient`. Os fotogramas em bruto permanecem no formato nativo; o `manifest.json` da sequência regista a orientação de cada membro e a opção. |
| `--build` | desativado | Após a paragem, reprocessar imediatamente a sequência (tal como o `array-build-video`). |
| `--products …` | `combined:index` | Com `--build`: que vídeo(s) criar (ver abaixo). |
| `--fps F` | `10` | Com `--build`: fps do vídeo de saída. |
| `--save-tiffs` | desativado | Com `--build`: guarda também ficheiros TIFF calibrados por fotograma (cada um com a sua [máscara de limitação de mistura](#unmix-clamp-mask-which-pixels-a-banda-foi-limitada) quando um pixel é limitado; o resultado da compilação lista-os como `clamp_masks`). |
| `--gif` | desativado | Com `--build`: grava também GIFs animados. |

### `array-build-video` — reprocessa offline uma sequência guardada

Sincroniza temporalmente cada fotograma bruto com a leitura `.daq` guardada mais próxima e submete-o à **mesma cadeia de radiação / refletância / índice que o pipeline de importação**, gerando um ou mais vídeos.

`--products` é uma lista separada por vírgulas de itens `kind:level`, em que `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Um `level` isolado (sem `kind:`) assume por predefinição o valor de `per_cam`. O valor predefinido é `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Indicador | Padrão | Descrição |
| --- | --- | --- |
| `--burst-dir DIR` | (obrigatório) | Caminho para a pasta de burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, conforme acima. |
| `--fps F` | `10` | fps do vídeo de saída. |
| `--save-tiffs` | desativado | Guardar também ficheiros TIFF calibrados por fotograma juntamente com o(s) vídeo(s). |
| `--gif` | desativado | Grava também GIFs animados. |

**Um vídeo não contém máscara de limitação** (um fluxo de imagens, 8 bits). Em vez disso, o resultado da compilação inclui `clamp_fractions`: por produto de vídeo, a parte de píxeis de cada fotograma que a separação de cores limitou na banda que o seu canal R/G/B contém — o `Chloros:Band{R,G,B}UnmixClampFraction` com o qual o produto de radiação/refletância desse fotograma seria marcado (para o índice combinado, de cada câmara participante). A execução imprime o pior fotograma por produto numa linha `Clamp:`. O `--save-tiffs` adiciona as máscaras por pixel para os ficheiros TIFF, no `clamp_masks/<level>/`, ao lado da pasta `<level>/` dos ficheiros TIFF.**Uma compilação interrompida não deixa nenhum vídeo incompleto.** Cada vídeo é gravado como `<name>.partial.avi` e só recebe o seu nome definitivo quando estiver concluído; por isso, um vídeo mais antigo com esse nome permanece até que o seu substituto esteja completo. Execute novamente a compilação para substituir um `.partial.avi` que tenha ficado para trás.

A execução apresenta todos os vídeos que gravou, seguidos de qualquer produto `Skipped:` (uma câmara/nível solicitado — ou o índice combinado — que não produziu vídeo, com o motivo — por exemplo, ausência de pacote de calibração, ausência de leitura DLS para refletância, ou ausência de par de bandas vermelha/NIR para o índice combinado) e qualquer `Note:`. Uma câmara RGB solicitada para radianância ou refletância recebe, em vez disso, o seu vídeo de visualização, com equilíbrio de brancos quando o pacote de calibração da câmara estiver disponível; uma câmara renderizada sem ele é identificada num `Note:`. A função **retorna um valor diferente de zero** quando um produto solicitado foi ignorado ou não foi criado conforme solicitado (uma expressão de índice configurada não é avaliada offline); os vídeos que foram criados continuam a ser gravados e listados.

> **Escolha o gravador adequado.** O `array-record` é de *nível de monitorização* — grava a composição ao vivo renderizada: a vista combinada da GUI ou, sem nenhuma GUI aberta, a própria vista NDVI (ou `--expr`) do próprio backend. O `array-burst` → `array-build-video` é de *nível de análise* — guarda os dados brutos do sensor à taxa máxima e reconstrói posteriormente vídeos calibrados de radiação/refletância/índice, sem necessidade de visualização em direto.

### Câmaras monocromáticas (M3M) de banda única

A linha **M3M**é a versão monocromática da**M3C**da Bayer: um filtro de interferência de banda estreita por câmara (`M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685`), pelo que o sensor fornece uma**única banda em escala de cinzentos** sem mosaico Bayer. Não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de brancos para definir — todo o processo de cor da exibição RGB simplesmente não se aplica.

O que isso significa na CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detetam uma câmara monocromática e**ignoram com uma mensagem de uma linha**, em vez de aplicarem definições sem sentido. Continuam a funcionar normalmente com uma câmara RGB/Bayer M3C na mesma sessão.
- **Um formato de píxeis Bayer implica a sua profundidade.** Um M3M oferece apenas formatos mono; por isso, um formato Bayer proveniente de `--pixel-format` ou `--preset` define o formato mono com a mesma profundidade: `BayerRG8` e `--preset high_speed` resultam em `Mono8`; `BayerRG12`, `high_quality` e `triggered` resultam em `Mono12`; um nome compactado permanece compactado (`BayerRG12p` resulta em `Mono12p`). Os ficheiros e o nome `lattice format` referem-se ao formato Mono. Uma matriz que mistura câmaras M3C e M3M adota um formato da mesma forma: Bayer em cada M3C, a sua profundidade Mono em cada M3M. Um nome corresponde à sua profundidade de bits: o mesmo se aplica no sentido inverso, pelo que uma M3C nunca é colocada num formato Mono (`--pixel-format Mono12` é `BayerRG12` numa M3C, independentemente de qual das câmaras for a principal da matriz), e o `lattice format --set` é mapeado da mesma forma e imprime o formato que definiu.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** continuam a funcionar — a radianância e a refletância são mapas radiométricos *por banda* e estão perfeitamente bem definidos para uma única banda. Os fotogramas monocromáticos contêm uma matriz de resposta do sensor **identitária** (sem separação 3×3), pelo que o plano passa pela matemática de calibração inalterado.
- **Uma única câmara monocromática não consegue produzir um índice de vegetação.**O NDVI, NDRE e outros índices semelhantes requerem, pelo menos, duas bandas (por exemplo, Red + NIR). Para obter um índice a partir de equipamento monocromático, aponte**várias** câmaras M3M para diferentes comprimentos de onda, alinhe-as numa pilha multibanda e calcule o índice *a partir dessa pilha*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` abre as câmaras por si próprio (uma [sessão transitória](#multi-camera-transient-sessions)), por isso não ligue nada primeiro: `--serials`; caso contrário, as câmaras ativadas no perfil (ver [Índice / Matemática da Vegetação](#index--vegetation-maths)). Enquanto o Chloros mantiver qualquer câmara (a GUI, `cam-connect`, `array-connect`) a solicitação é recusada com o código HTTP 409 `CAMERA_HELD` e o programa encerra `1`; liberte-as com `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Os símbolos `--channel` devem corresponder **exatamente** aos nomes dos canais da predefinição (distingue maiúsculas de minúsculas; os NDVI são em minúsculas `red`, `nir` — ver `--list-presets`), e o nome lateral da banda deve corresponder a uma banda na pilha alinhada (o modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`).

O discriminador em toda a pilha é o token `M3M` na cadeia de caracteres do modelo (que nunca aparece numa cadeia de caracteres `M3C`), apresentado na GUI/SDK como `is_mono`.

---

## Configuração e ajuste da placa de rede do anfitrião (matrizes LATTICE)

As câmaras LATTICE transmitem GVSP através do adaptador Ethernet do anfitrião; por isso, no caso de matrizes com várias câmaras, o **driver**e o**tamanho do anel de receção** são tão importantes quanto a velocidade da ligação. Configurações incorretas manifestam-se como um `FRAMES WILL DROP` / `Reduce ROI to enable` no painel «Configurações do Conjunto» (e no `lattice network-analysis` / no SDK `analyze_array_network()`), mesmo quando as próprias câmaras estão em bom estado.

### Adaptadores USB 10GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

As definições abaixo são propriedades do controlador do Windows. No Linux, um adaptador USB também tem um limite (400 MB/s em x86_64, 200 MB/s no NVIDIA Jetson) (ver [Modelo de fps e burst do Array](#array-fps--burst-model)), e os buffers de receção de sockets do kernel são importantes para todos os adaptadores — ver [Hosts Linux](#linux-hosts--socket-receive-buffers).

| Item | Valor necessário | Por que é importante |
| --- | --- | --- |
| **Versão do controlador**|**≥ v10.67 (janeiro de 2026)**, INF `rtump64x64sta.inf` | O controlador legado de**2016**(v10.65, `rtump64x64.inf`) não lida corretamente com o desligamento e as verificações de erros com**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ao desligar/reiniciar/entrar em modo de suspensão. A transição fica bloqueada (tempo de espera de ~5 min), o utilizador desliga o equipamento à força e os encerramentos incorretos repetidos**corrompem o repositório WMI**(o PowerShell e as ferramentas começam a falhar com o erro `Invalid class`) e**bloqueiam a pilha USB** na próxima inicialização (a placa de rede não é ativada; as unidades USB deixam de ser reconhecidas). Atualize a partir de realtek.com (ou do fornecedor do dongle) antes de contar com reinicializações corretas. |
| **Buffers de receção**— palavra-chave `ReceiveBufferLen` |**256**(máximo do controlador) | O anel de receção da placa de rede. O valor predefinido do controlador de**32**deixa apenas cerca de 0,26 MB de anel utilizável — demasiado pequeno para uma rajada multiclâmera — pelo que o painel da matriz reporta o erro `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e bloqueia a ligação. Com**256**o anel é grande (**cerca de 13,5 MB medidos no anfitrião de 10 GbE do laboratório**), proporcionando ao pipeline de receção margem real para rajadas GVSP com várias câmaras. (Se uma determinada configuração *estabelece ligação* é decidido por duas verificações — a verificação de **admissão com consideração de escoamento**e a verificação de**sobre-subscrição agregada** — e não por uma comparação bruta entre a rajada e o anel; ver [Modelo de fps e rajadas do array](#array-fps--burst-model).) |
| **URBs de receção**— palavra-chave `PendingReceives` |**64** (máx.) | Blocos de pedidos USB em trânsito; aumentar em conjunto com os Buffers de Recepção para absorção de rajadas. |
| **Jumbo Frame** — palavra-chave `*JumboPacket` | **9014** | Necessário para pacotes GVSP de 9000 bytes (6 vezes menos pacotes por quadro do que com 1500). |

> ⚠️ **Uma atualização do controlador da placa de rede REINICIA estas propriedades avançadas para os valores predefinidos.**Após atualizar ou substituir o controlador da placa,**reaplique** `ReceiveBufferLen=256` e `PendingReceives=64`, ou o painel do array voltará a bloquear o acesso, mesmo que «nada tenha mudado no hardware». Esta é a principal causa de um equipamento que antes funcionava deixar subitamente de se ligar.

Aplique a partir de um PowerShell com **privilégios elevados** (substitua pelo nome do seu adaptador, por exemplo, `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **O `lattice network --fix` abrange adaptadores USB 10GbE.** Agora deteta o tipo de adaptador e ajusta a palavra-chave correta do anel de receção: `*ReceiveBuffers`→2048 para placas de rede PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 para o controlador **USB** 10GbE da Realtek (que não expõe o `*ReceiveBuffers`). Os valores-alvo são limitados ao máximo indicado por cada controlador (`NumericParameterMaxValue`), pelo que nunca é gravado um valor fora do intervalo. Execute-o a partir de um terminal com **privilégios elevados**; tal como qualquer ajuste baseado no registo, a alteração entra em vigor após o reinício do adaptador ou o reinício do sistema. Os comandos manuais `Set-NetAdapterAdvancedProperty` acima referidos continuam a ser uma boa alternativa — são aplicados em tempo real (reassociam o adaptador) sem necessidade de reinício.

### Hosts Linux — buffers de receção de sockets

No Linux, incluindo o NVIDIA Jetson, os buffers de receção de sockets do kernel são tão importantes quanto o anel de receção da placa de rede. Com a configuração predefinida do kernel (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bytes), uma sequência de pacotes GVSP da câmara provoca um transbordamento do socket de fluxo antes de o Chloros o ler. Os pacotes são descartados no kernel, embora a própria placa de rede não reporte qualquer perda, e um conjunto de câmaras perde grupos de fotogramas e reduz a sua taxa de fotogramas. Medido num conjunto de 4 câmaras ligado a um adaptador USB 10 GbE, aumentar ambos para 32 MB fez com que o conjunto passasse de 4,96 para 7,52 fps após a ligação, e uma captura bruta contínua de 4,7 para 7,6 capturas/s, sem que nenhum pacote fosse descartado no kernel.

- **O ficheiro .deb define esses valores.** A instalação do pacote grava o valor `/etc/sysctl.d/60-chloros-gige.conf` (ambos em `33554432`) e aplica-o imediatamente. Um valor já definido para um valor superior não é alterado.
- **O `lattice network` verifica-os.** Quando qualquer um dos valores for inferior, lista `socket receive buffers too small for GigE Vision (…)` em *Problemas encontrados*, e o registo do backend apresenta o mesmo aviso na primeira ligação de uma câmara ou de um conjunto de câmaras.
- **Para os aumentar manualmente** (até ao próximo reinício; o ficheiro do pacote aplica-os em cada arranque):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  O `lattice network --fix` também as aumenta e mantém, mas apenas quando o backend é executado como root. Um backend executado como o seu utilizador (iniciado com o `chloros-backend` ou [`chloros-backend@<user>`](#executar-o-backend-como-um-serviço-sem-interface-gráfica)) não pode alterar as configurações do kernel e, em vez disso, apresenta a linha `sudo sysctl`.

**O ajuste automático dos adaptadores afeta apenas os adaptadores ligados.** Na primeira ligação da câmara, um backend executado como root ajusta os adaptadores com fios cujo anel de receção é pequeno. Agora, ajusta apenas os adaptadores com ligação, pelo que uma porta integrada desligada mantém o seu MTU e os buffers de receção permanecem inalterados. O `lattice network` já não indica o MTU de um adaptador desligadocomo um problema. O `lattice network`, o `--fix`, o `lattice probe` e o ajuste de adaptadores consideram apenas portas Ethernet com fios, não o barramento CAN de um Jetson (`can0`) nem as ligações em modo de dispositivo USB (`usb0`, `usb1`) que este oferece a um PC anfitrião; um adaptador Ethernet USB para câmaras continua a ser considerado.

### Noções básicas de rede (todas as ligações LATTICE)

- **Endereçamento:** endereço local de ligação `169.254.0.0/16` (GigE Vision LLA). O anfitrião utiliza um endereço estático `169.254.x.x/16`; as câmaras e o DAQ-E atribuem-se automaticamente endereços no mesmo intervalo. Não é necessário DHCP nem gateway.
- **Tamanho do pacote:**prefira jumbo (9000), mas deixe que a detecção automática o determine — esta ignora o limite de 1500 bytes do ICMP da câmara através de um teste GVSP, pelo que opta por jumbo sempre que a ligação o suportar efetivamente.**Cada câmara é medida uma vez por sessão do backend e novamente quando a sua ligação muda (outro adaptador, outra velocidade de ligação ou uma ligação que caiu, como acontece ao trocar um switch); as reconexões reutilizam o resultado. Após uma alteração atrás de um switch que permaneceu ativo, reinicie o backend para medir novamente.**Uma primeira ligação pode, ocasionalmente, demorar alguns minutos nesta verificação, e o registo indica isso quando acontece. Utilize o pin com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` apenas quando tiver mais conhecimento do que o probe, e dê preferência a configurações por comando em vez de permanentes: um pin ignora o probe, por isso, se o caminho não conseguir realmente transmitir 9000,**todas as** capturas entram em timeout com o `SC_ERR_TIMEOUT -1011` (ver [Variáveis de Ambiente](#environment-variables)).
- **O anel RX é dimensionado com o `ReceiveBufferLen`:**no valor predefinido `32`, o anel utilizável é de ~0,26 MB (demasiado pequeno para qualquer sequência de várias câmaras); no valor máximo `256`, é grande (~13,5 MB medidos no anfitrião de 10 GbE do laboratório), proporcionando uma margem real. A decisão sobre se uma configuração se liga ou não é então determinada pela verificação de admissão sensível ao consumo**e** pela verificação de sobre-subscrição agregada abaixo — não por uma comparação bruta entre a sequência de imagens e o anel.

### fps do Array e modelo de burst

Como interpretar o painel de Configurações do Array (e o `lattice analyze-array` / o `analyze_array_network` do SDK):

- **O burst é somado por câmara no formato de píxeis real de cada câmara.**As câmaras Mono**M3M**transmitem**Mono12 (2 B/px)**, ou, com o `--pixel-format`, o formato Mono da sua profundidade (`BayerRG8` é `Mono8`, 1 B/px); as câmaras**M3C**Bayer transmitem em 8 ou 12 bits (o firmware M3C emite silenciosamente BayerRG12 mesmo quando é solicitado BayerRG8). Assim, um fotograma de resolução total com 4 câmaras tem**cerca de 12,6 MB se forem todas de 8 bits, mas cerca de 25 MB com três câmaras mono de 12 bits**. A projeção determina o formato de cada câmara a partir do seu modelo (cache de identidade), pelo que a sequência de dados corresponde ao que o cabo transmite efetivamente — e não a uma suposição genérica de BayerRG8.
- **Um adaptador Ethernet USB está limitado a 200 MB/s, independentemente da especificação indicada na etiqueta.** A tabela de eficiência que converte uma taxa de ligação num valor sustentado é derivada da PCIe; uma placa de rede USB anuncia a sua taxa de ligação *Ethernet*, mas está limitada pelo barramento USB e pelo seu controlador. Um dongle USB 10GbE costumava atingir ~1063 MB/s «sustentada» — um valor que nunca foi verificado — e o ritmo resultante corrompia 6–18 % dos quadros, apesar de continuar a reportar um fps alvo satisfatório. As placas de rede ligadas por USB estão agora limitadas de forma absoluta: **200 MB/s** no Windows e no NVIDIA Jetson, **400 MB/s** no Linux x86_64 (no Linux, um adaptador ligado a uma porta USB 2.0 tem uma largura de banda limitada a **30 MB/s**, e as especificações de capacidade indicam que se deve ligá-lo a uma porta USB 3) (o limite é determinado pelo barramento USB e pelo controlador do anfitrião, pelo que não varia em função da especificação nominal; um adaptador USB 1 GbE atinge cerca de 80 MB/s e não é afetado). No Linux, o adaptador é identificado a partir do sysfs (recorrendo a `ethtool -i` como alternativa a `bus-info`). Cada valor é medido: no Windows, o mesmo conjunto de 4 câmaras corrompeu 6–10 % dos fotogramas acima dos 200 MB/s; no Linux x86_64, o mesmo adaptador funcionou sem erros em todos os limites até aos 480 MB/s (273 MB/s fornecidos, o limite próprio das câmaras). No NVIDIA Jetson, o mesmo adaptador funcionou sem erros apenas a 200 MB/s e perdeu fotogramas de forma intermitente acima desse valor. O `wire_ceiling_source` no registo de capacidades indica isso por escrito, e o `nic_is_usb` assinala essa situação. É possível anular qualquer uma das opções com o `--wire-ceiling-mbps`.
- **A admissão tem em conta o consumo de recursos, não a comparação entre «burst» completo e «ring».** Uma rajada simultânea só tem de caber no *backlog transitório* = `max(0, Σ per-cam arrival − host drain) × emit_window`, não na rajada completa. Numa malha de anfitrião rápido / câmara lenta (um **PCIe**de 10G + 4× cams de 1 GbE: chegada ≈ 320 MB/s, drenagem ≈ 1063 MB/s), o host esvazia mais rapidamente do que as câmaras enchem, o backlog é ≈ 0, pelo que a emissão simulada em resolução total**é admitida**, apesar de a rajada de 25 MB exceder o anel de 13,5 MB. Coloque as mesmas quatro câmaras atrás de um adaptador**USB** de 10 GbE no Windows e a saída é de 200 MB/s, e não 1063 — a chegada ultrapassa-a, e a perda manifesta-se como quadros corrompidos em vez de uma taxa de quadros mais baixa. Num anfitrião de 1 GbE, o limite mínimo de DLThr de 31,25 MB/s das câmaras faz com que a chegada ultrapasse a taxa de escoamento → o sistema **bloqueia** corretamente (para *esta* classe de bloqueio, reduza a ROI ou utilize binning ≥ 2). A admissão é uma das **duas** portas de ligação — a outra é a verificação de sobre-subscrição agregada abaixo.
- **Os fps projetados correspondem ao ciclo da câmara mais lenta.**Na emissão simultânea, todas as câmaras enviam o seu fotograma ao mesmo tempo, pelo que a matriz opera ao ritmo do maior entre o tempo de leitura do sensor e o tempo de emissão do seu membro mais lento (bytes do fotograma ÷ a sua taxa de emissão: o**ligação de acesso** da câmara, 1 GbE ≈ 80 MB/s, ou a sua quota da largura de banda máxima do anfitrião, quando esta for inferior). Cada exposição sobrepõe-se à leitura do fotograma anterior, pelo que a exposição não entra no ciclo: a exposição automática preenche o período determinado pela taxa, depois aumenta o ganho, e uma cena escura já não reduz a taxa (na versão 1.2.0 isso acontecia). Apenas uma exposição manual mais longa do que o período retarda o disparo (ver [Exposição num membro da matriz](#exposure-on-an-array-member)). Adicionar câmaras não retarda a matriz até que a partilha de largura de banda destas diminua. Na emissão em série (`slip-emit-and-capture`), os tempos de emissão das câmaras somam-se, e a exposição de cada membro é considerada no escalonamento. Medido com o sensor a plena capacidade, 12 bits, numa matriz de 9 câmaras com um anfitrião PCIe 10 GbE: 10,7 fps, ou 14,0 fps com [transporte compactado](#packed-12-bit-transport---packed).
- **Após uma captura contínua, a projeção indica o valor que se manteve.** Uma captura de 30 s ou mais a uma taxa automática regista a taxa em que se estabilizou para aquele tamanho de matriz neste computador; as breves tentativas do controlador de taxa a uma velocidade mais elevada são ignoradas. A partir daí, a linha do `array-connect` e o painel «Definições da Matriz» indicam essa taxa quando esta é inferior à do modelo, juntamente com o que a limitou: `clock-bound` (a matriz funcionou à taxa em que se estabilizou, com o ciclo de captura à espera do seu relógio de disparo), `loop-bound` (o ciclo de captura ficou sem tempo), `capture-bound` (a captura ficou atrasada e saltou grupos) ou `measured` (não há indicação de qual). O valor do modelo está em `wire_fps_max`. A matriz continua a atingir o que a ligação suporta.
- **Uma matriz funciona em qualquer rede — a taxa de fotogramas é que determina o limite.**Cada câmara recebe uma parte igual do**limite máximo da ligação à prova de colisões**(`sustained × sim_emit_factor`), sincronizada com o GevSCPD, e todas elas continuam a expor com o mesmo disparador. Numa ligação lenta, a quota é pequena e a taxa de fotogramas diminui em conformidade: 8 câmaras de 12 bits em resolução total numa ligação de 1 GbE sem quadros jumbo obtêm 52 / 8 = 6,5 MB/s cada, cerca de**1 fps**(cerca de 1,3 fps compactados), enquanto um anfitrião de 10G as executa a 11-14. Se as câmaras não puderem ser sincronizadas a um ritmo tão lento — verifique o intervalo GevSCPD de cada câmara para determinar —, o conjunto muda para emissão**serial**(`slip-emit-and-capture`): as câmaras revezam-se na ligação à velocidade máxima do link, sendo acionadas uma a seguir à outra, pelo que todos os quadros continuam a chegar, mas os membros**não são expostos simultaneamente**. `analyze-array` e a saída do comando «connect» indicam o modo; defina `--force-tier sim-capture-sim-emit` para recusar o recurso alternativo. O tamanho do fotograma não influencia esta partilha, pelo que**o binning e uma ROI mais pequena aumentam a taxa de fotogramas, mas não alteram a capacidade de a matriz caber**. Apenas um cabo que não consiga transportar a matriz, nem mesmo em série, está**sobrecarregado**— por exemplo, muito mais câmaras do que um cabo consegue suportar, ou muitas outras câmaras a transmitir nele — e, nesse caso, o sintoma é a perda de pacotes GVSP, não uma taxa mais baixa: O `analyze-array` zera os valores de fps alcançáveis e apresenta o `** OVER-SUBSCRIBED**`; a ligação prossegue com um aviso sonoro e uma ligação HTTP com o `pin_resolution: true` (modo estrito) **é recusada**. As soluções consistem em reduzir o número de câmaras, utilizar quadros jumbo de ponta a ponta ou utilizar uma placa de rede mais rápida. O `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` reclassifica a recusa para um aviso sonoro para trabalhos de laboratório — consulte [Variáveis de Ambiente](#environment-variables).

### Estado do array — qual o subsistema que está a perder quadros

O `GET /api/camera/array/<array_id>/capability` de um array ligado contém um bloco
`health` em tempo real, reavaliado numa janela contínua de **10 segundos**. Este divide a perda de quadros
nas duas causas que requerem correções opostas, em vez de reportar uma taxa «incompleta»
que não identifica nenhuma delas:

| Campo | O que significa | Qual o subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por porta série) | O quadro **chegou e estava estruturalmente incorreto**— perda de pacotes GVSP. |**Rede**: largura de banda disponível, cadência, anel de receção da NIC, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nunca chegou**— a câmara não disparou ou nada saiu dela. |**Disparador / sincronização**: cabo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A pior taxa de cada câmara. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmara (ambas as causas em conjunto). | — |
| `stable_for_seconds` | Durante quanto tempo cada câmara permaneceu abaixo de 0,01 %. | — |

Acima de 5 %, o backend regista uma linha `[array-health <id>] WARN` indicando a divisão — na
primeira violação, numa mudança de faixa de gravidade, uma vez por minuto enquanto persistir e uma vez quando
for resolvida. A metade corrompida grava `[gvsp-corrupt <SN>]` no ficheiro de registo do backend na
primeira ocorrência por câmara e motivo, seguindo-se um resumo a cada 60 s; o painel de registo da aplicação mostra, em vez disso, a
linha `[array-health <id>]`. Todas as avaliações continuam a ser registadas no ficheiro de registo do backend;
os contadores avançam a cada buffer, independentemente do que é impresso.

O mesmo registo indica o número ao qual toda a alocação está associada:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de largura de banda sustentado do anfitrião em vigor, MB/s. |
| `wire_ceiling_source` | De onde veio esse número, por extenso — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (no Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando definido por `--wire-ceiling-mbps` (ou pelo campo **Orçamento de banda** da GUI). |
| `nic_is_usb` | `true` para um adaptador Ethernet USB — consulte o limite USB acima. |

**Interpretação:** um valor diferente de zero para `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` igual a 0
significa que o disparo e a sincronização do cabo estão perfeitos e que 100 % da perda ocorre no caminho da rede
— reduza o valor de `--wire-ceiling-mbps` e volte a ligar. O padrão inverso aponta, em vez disso, para o
cabo de sincronização ou para a linha de disparo.

> **O `--target-fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido
> uma vez na ligação, pelo que diminuir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na procura não produziu qualquer melhoria;
> reduzir o limite máximo da ligação de 240 para 200 MB/s fez com que o mesmo equipamento passasse de 10,4 %
> de quadros corrompidos para 0,00 %.

> **Uma matriz em funcionamento não consegue replanejar-se no firmware LATTICE (M3C / M3M).** Não
> consegue resolver isto por si próprio; desligue e volte a ligar para que o seletor de tempo de ligação possa
> replanejar com o novo limite máximo.

### Sintoma → correção

| Sintoma (Definições da matriz / ligação / `analyze_array_network`) | Causa | Correção |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` redefiniu para 32 (normalmente após uma atualização do controlador) | Defina `ReceiveBufferLen`→256, `PendingReceives`→64; reabra o painel (reinicie o backend se este tiver armazenado em cache o tamanho antigo do anel) |
| O reinício/desligamento fica bloqueado; posteriormente, erros WMI `Invalid class`, a placa de rede não ativa, faltam unidades USB | Driver antigo Realtek USB 10GbE de 2016 → BSOD `0x9F` → desligamentos forçados | Atualize o driver do adaptador para ≥ v10.67 (2026) e, em seguida, reaplique as definições do anel de receção acima |
| O array liga-se com o sensor no máximo, mas apresenta uma baixa taxa de fotogramas | O modo automático mantém o sensor no máximo e deixa a taxa cair para o que a ligação suporta | Escolha uma das opções mais rápidas da lista de saídas de ligação (`--binning 2` / `4`, ou um `--width`/`--height`), ative os quadros jumbo ou atualize a ligação |
| O Array reporta uma taxa de fotogramas alvo válida, mas entrega apenas uma fração dela; `health.gvsp_corrupt_rate_pct` diferente de zero, `never_arrived_rate_pct` 0 | O orçamento de largura de banda inferido do anfitrião exagera o que este realmente suporta (típico num adaptador Ethernet USB, numa via PCIe estreita ou numa malha partilhada) | Volte a ligar com um valor mais baixo de `--wire-ceiling-mbps` e verifique novamente o bloco de integridade. **Não** `--target-fps` — O ritmo do GevSCPD é fixado na ligação |
| Linux: uma matriz perde grupos de quadros e reduz a sua taxa de quadros, enquanto a NIC não reporta perdas; O `lattice network` lista o `socket receive buffers too small` | Os buffers de receção de sockets do kernel estão nos valores predefinidos, pelo que os pacotes da câmara são descartados no kernel | Aumente o `net.core.rmem_max` / `rmem_default` para 32 MB — consulte [hosts Linux](#linux-hosts--socket-receive-buffers). O ficheiro .deb faz isto durante a instalação |
| Câmaras em falta nos grupos publicados; `health.never_arrived_rate_pct` diferente de zero, `gvsp_corrupt_rate_pct` 0 | Acionamento / caminho de sincronização — as câmaras não estão a disparar, não é um problema de rede | Verifique o cabo de sincronização M8 e o `--line`; confirme se todos os membros estão armados (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` excedido em `analyze-array`, ou recusa de ligação no modo estrito (`pin_resolution`; `array over-subscribes the wire`) | O cabo não consegue suportar a matriz a qualquer ritmo que as câmaras atinjam, simultaneamente ou em sequência — há muito mais câmaras do que o cabo consegue partilhar, ou outras câmaras a transmitir nele | Menos câmaras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. **ROI/binning NÃO ajudará** (a partilha é independente do tamanho do quadro). O `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` substitui no banco de ensaio (aceita a perda de pacotes) |
| A matriz liga-se no `slip-emit-and-capture` (serial) numa rede lenta | As câmaras não podem ser sincronizadas a um ritmo suficientemente lento para partilharem o cabo simultaneamente, pelo que se revezam; os quadros não são expostos no mesmo instante | Quadros jumbo ou uma placa de rede mais rápida restabelecem a captura simultânea; o `--force-tier sim-capture-sim-emit` insiste nisso (e, em vez disso, perde pacotes) |

---

## `chloros-cli daq`

Comandos do sensor espectral. Os comandos `pool-*` são clientes HTTP «leves» que controlam o sensor através do conjunto persistente do backend, pelo que necessitam de um backend em execução (ver [Iniciar o Backend](#iniciar-o-backend)). O backend é responsável pelo transporte, pelo que os scripts da CLI e do SDK partilham um único identificador ativo, em vez de disputarem a porta série. O `pool-discover` é a única forma de obter o MAC BLE de um DAQ-M.

> **`chloros-cli daq --help`** (ou `-h`, ou `daq help`) lista os subcomandos do `pool-*` (no Linux, juntamente com os comandos diretos de hardware que a compilação inclui), e o `chloros-cli daq <subcommand> --help` (ou `daq help <subcommand>`) apresenta as opções de cada um. Estas abrangem a ligação, stream, gravação de ficheiros `.daq` calibrados, troca de perfis de capacitância e ligação de um DAQ a uma câmara; no caso de um DAQ-E, as suas definições de rede, PTP, reinício e atualizações de firmware. O DAQ também pode ser controlado a partir de Python com o comando `chloros_sdk.connect_daq_sensor()`, que utiliza o mesmo caminho partilhado.
>
> **Utilize os comandos `pool-*` em todas as plataformas.** Funcionam da mesma forma no Windows, no Linux e no Jetson. No Windows, o `daq discover` e o `daq list` são aliases do `pool-discover`, e qualquer outro comando `daq` que não seja o `pool-*`, termina o comando `1` com o comando `DAQ direct-hardware commands require the full daq package`. No Linux e no Jetson, os comandos `daq discover`, `daq list` e os outros comandos que não sejam `pool-*` e `daq` são os comandos diretos: abrem o sensor no próprio processo da CLI, fora do conjunto de sensores do backend (ou seja, não num sensor que o backend, a GUI ou um script mantenha), e `daq discover` aceita `--only` e `--timeout`, mas não `--json`. Esta referência documenta apenas os comandos `pool-*`.

### Fluxo de trabalho da primeira ligação do sensor DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referência do `pool-*`

| Subcomando | Finalidade |
| --- | --- |
| `daq pool-connect` (smart-detect) | Abrir um sensor no conjunto de backends. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U numa porta série específica. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC detetado automaticamente. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE num MAC conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet com um nome de anfitrião ou IP conhecido (implica `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, anfitrião detetado automaticamente (mDNS + recurso de ARP; funciona com uma cache ARP vazia no Windows e no Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Ajustar janela de integração / estado AE. |
| `daq pool-connect --no-stream` | Ligar, mas não iniciar ainda a transmissão (retomar com `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de correção de limite. O valor predefinido no backend é `sunshine_cosine`; `none` é um sensor sem tampa. Um [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignora-o (difusor fixo). **Os sensores são fornecidos sem tampa, por isso atribua um nome à tampa que instalou:** o valor predefinido é registado como (`cap_id_source=auto_default`) e o processamento emite um aviso a esse respeito, uma vez que um DAQ-U sem difusor, sob a correção de luz solar, apresenta uma leitura cerca de 30 vezes superior. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Analise cada transporte à procura de sensores aos quais possa ligar-se, sem estabelecer ligação. **É assim que se encontra o MAC BLE de um DAQ-M.** O `--timeout` é por transporte (padrão 5 s, 1-20). Os sensores já abertos no conjunto não são listados — um DAQ-M ligado deixa de anunciar —, por isso utilize o `pool-list` para esses. `usb` lista apenas dispositivos seriais USB: as portas seriais do próprio computador (`COM1`, `/dev/ttyS*` do Linux) não podem ser sensores e são excluídas. |
| `daq pool-list` | Mostra todos os sensores no conjunto de backends. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Associa uma câmara ou uma matriz a um DAQ, tal como a seleção de sensores de luz da GUI faz: as suas capturas utilizam, correspondem e guardam as leituras desse DAQ. `SENSOR` é um identificador de sensor proveniente de `pool-list` ou o endereço através do qual se acede ao DAQ. `--clear` (sem `SENSOR`) cancela a ligação, e as capturas voltam a utilizar o DAQ mais recente. A ligação mantém-se até que o backend reinicie ou até que essa câmara, matriz ou DAQ seja desligado: uma desconexão (aqui, a partir do SDK ou na aplicação) interrompe todas as ligações do hardware que desliga, independentemente de quem as tenha definido, incluindo os membros de uma matriz juntamente com a própria matriz. A câmara deve estar ligada, no projeto aberto, ou ter sido aberta anteriormente neste computador, e a matriz deve estar ligada ou no projeto aberto; qualquer outro número de série ou ID é rejeitado por nome e nada é associado. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Libertação — independentemente de quem tenha aberto o sensor, pelo que também liberta um sensor que um script que tenha falhado tenha deixado na piscina (qualquer gravação nele é concluída primeiro). Um sensor que não esteja na piscina não tem nada para libertar: imprime `ID: already released (not in the pool)` e sai com `0`; uma libertação que o backend recusou ou não confirmou sai com `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Os N quadros de espectro mais recentes (padrão 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Iniciar/parar uma gravação .daq. O `-o` assume por predefinição o valor `~/Documents/DAQ Live View` no computador do backend (um caminho relativo na pasta a partir da qual executa o comando); `--duration` (`-d`, valor predefinido `0` = até `--stop`). O `--confirm-after S` emite um aviso quando não chega nenhuma leitura S segundos após o início (valor predefinido 20; 0 desativa-o); com o `--duration`, `--heartbeat S` apresenta o número de leituras a cada S segundos e emite um aviso quando este deixa de aumentar (valor predefinido 30; 0 desativa esta funcionalidade). Uma gravação num DAQ-E que fica em silêncio continua no mesmo ficheiro assim que o backend restabelecer a ligação, e o `--stop` lista as lacunas ([Um DAQ-E que deixa de enviar dados](#a-daq-e-that-stops-delivering)). Ctrl+C encerra antecipadamente uma gravação do `--duration` e notifica o utilizador; sem o `--duration`, a gravação continua a decorrer após o Ctrl+C (sair do `130`) até ao `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de capacitância em tempo de execução. É necessário o `--cap-id`; o `--cap-id none` remove a correção de capacitância (um sensor sem tampa). Um DAQ-E-S mantém o `as_recorded` independentemente da configuração definida. |
| `daq pool-enable-ptp --sensor-id ID` | Reativa o PTP num DAQ-E cujo PTP esteja desativado. A configuração é guardada na unidade, pelo que uma unidade deixada desligada permanece desligada — os seus carimbos de data/hora não são sincronizados por PTP — até que este comando seja executado. O comando `pool-list` assinala essa unidade (`!! PTP is switched OFF`) e imprime este comando para ela. O Chloros nunca reativa uma unidade por si próprio, porque o estado desligado pode ser intencional. |
| `daq pool-info --sensor-id ID [--json]` | As próprias configurações de um DAQ-E: o seu firmware (e se esta versão do Chloros inclui uma versão mais recente), o seu endereço e como o obteve (DHCP / link-local / estático), a configuração de rede guardada até o sensor a executar (uma configuração guardada com o `--no-reboot` incluída), o fluxo de dados ao ligar e o PTP. Num sensor cujo fluxo de dados não esteja a chegar a este computador, o programa lê o sensor numa nova ligação e indica isso. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Configurar um DAQ-E para DHCP (`--auto`) ou um endereço estático (`--mask`, por predefinição `255.255.255.0`). O sensor reinicia para aplicar a configuração e é reconectado com as definições que tinha, aguardando até `--timeout` segundos (padrão 45) para tal; `--no-reboot` apenas guarda a configuração. Consulte [Definições do dispositivo DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Se um DAQ-E inicia a transmissão ao ligar-se sem nenhum software ligado (guardado no sensor). O Chloros inicia a transmissão por si próprio em qualquer dos casos. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Reiniciar um DAQ-E e voltar a ligá-lo com as suas definições anteriores (aguarda até `--timeout`, valor predefinido de 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Atualiza o firmware do DAQ-E incluído nesta versão do Chloros num sensor que esteja a executar uma versão mais antiga e, em seguida, volte a ligá-lo (cerca de um minuto; não o desligue; `--timeout`, valor predefinido de 45 s para que volte a funcionar). Um sensor que já esteja atualizado não é alterado. |

### Um DAQ-E que deixa de transmitir

Um DAQ-E no conjunto que não transmita nenhum espectro durante 20 segundos é reconectado pelo backend. Isto é feito através de um ciclo de reinicialização do interruptor que alimenta o sensor: o sensor reinicia com ele e não volta a transmitir até que lhe seja solicitado (a menos que `pool-set-auto-stream --on`).

- **Primeiro, verifica se se trata do mesmo sensor.** Solicita, no endereço que o sensor tinha — sem alterar nada —, o nome, o MAC e o número de série do sensor. Também faz a consulta utilizando o nome `daq-e-<id>.local` do sensor, mas só confia nessa resposta quando esta provém do próprio adaptador de rede (MAC) do sensor. Nenhum outro dispositivo que responda é reconectado ou registado.
- **Volta a estabelecer ligação com as definições da sessão:** tempo de integração, média de fotogramas, exposição automática e perfil de cap, tal como estavam, e não com os valores predefinidos de arranque do sensor. Um sensor que tinha o PTP ativado e voltou com ele desativado é reativado. Em seguida, reinicia o fluxo e o recetor do computador para o mesmo.
- **Reitera a tentativa em intervalos crescentes:** imediatamente, depois após 5, 10, 20 e 40 segundos, e depois a cada 60 segundos enquanto o sensor estiver indisponível. Cada tentativa limita-se a perguntar ao sensor qual é a sua identificação. Um sensor que responde, mas continua sem enviar nada após três tentativas de reconexão, é ignorado e reportado (`auto-reconnect GAVE UP`). O `pool-stream --start` tenta novamente.
- **`pool-list` indica o estado atual.** Enquanto o sensor permanece em silêncio, a sua linha `STREAM FAULT` informa o que a tentativa de reconexão está a fazer. A linha é apagada com o primeiro espectro e, em seguida, o sensor apresenta `reconnected automatically N time(s)`.
- **A gravação prossegue no mesmo `.daq`.** O ficheiro assinala cada intervalo com duas linhas: `event_type` 5 no último espectro antes do intervalo e 6 no primeiro depois dele, cada uma apenas com um registo de data e hora. `pool-record --duration` imprime `!! STREAM LOST …` enquanto o sensor está ausente e `stream resumed after …` quando este regressa. `pool-record --stop` lista cada intervalo e emite um aviso, tal como faz para um ficheiro vazio (uma gravação que não contém nenhuma leitura sai de `1`).
- **Uma gravação mantém uma calibração.** Se o sensor regressar com uma calibração diferente enquanto uma gravação está a decorrer, o resto da gravação mantém a calibração com que começou, e o aviso de paragem indica isso. `pool-disconnect` + `pool-connect` adota a nova calibração.
- **Os seus comandos prevalecem.** Um sensor que tenha sido interrompido com `pool-disconnect`, ou pausado com `pool-stream --stop`, nunca é reconectado, mesmo que uma reconexão já esteja em curso.
- **`pool-info` num sensor silencioso** lê o sensor numa nova ligação e adiciona `!! stream NOT reaching this computer`. Um sensor que não responde é imediatamente sinalizado. O comando `pool-list` já não apresenta um estado PTP para um sensor que tenha deixado de responder.
- O DAQ-U e o DAQ-M não são abrangidos. Um DAQ-M volta a ligar-se por si próprio através de Bluetooth, e um DAQ-U desligado da porta USB necessita novamente do `pool-connect`.

### DAQ-E-S — difusor fixo e sensor de inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças relevantes para o software. Tudo o resto — deteção, ligação, PTP, gravação e as definições do dispositivo abaixo — funciona exatamente como num DAQ-E.**O seu difusor é permanente, pelo que não requer correção da tampa.** O difusor do tipo Sunshine está integrado e já se encontrava na unidade quando foi medida a sua calibração de fábrica, pelo que essa calibração já indica a irradiação descendente real. O Chloros define a sua tampa para `as_recorded` (sem perfil), independentemente do que for solicitado: um limite com o seu nome (`sunshine_cosine`, `fov_*`, `none`) é ignorado com uma mensagem, e os seus registos armazenam `cap_id_source=model`, pelo que o aviso de limite presumido nunca é acionado para este. Não aplique um limite aos seus dados manualmente: um limite de `sunshine_cosine` no topo apresentaria uma leitura cerca de 11 vezes superior.

**Este dispositivo possui um acelerómetro de 3 eixos.** A resposta cosseno do difusor de luz solar só é válida enquanto este estiver nivelado; por isso, um sensor inclinado reporta uma irradiância errada — e a refletância herda o erro — sem qualquer outro sintoma. Cada leitura do DAQ-E-S inclui, portanto, a sua atitude:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ângulos em graus. `None` quando o firmware marca o ângulo como indefinido (queda livre, um eixo inativo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleração em milig. `mag_mg` longe de ~1000 significa que a unidade estava em movimento e que os ângulos não são significativos para essa amostra. |
| `cal_applied` | Definido: os eixos e ângulos estão no sistema de referência do difusor (é aplicada a calibração de montagem da unidade). Limpo: sistema de referência bruto da placa, que pode apresentar até ~2,3° de inclinação que não é real. |
| `sample_age_ms` | Antiguidade da amostra do acelerómetro no momento em que o espectro foi enviado. |
| `temp_raw` | **Tendência** da temperatura do chip não calibrada — apenas diferenças, nunca uma leitura do termómetro. |

O `daq pool-latest --json` devolve-os sob a chave `imu` de cada quadro (`null` num DAQ-U, DAQ-M ou num DAQ-E simples). Um `.daq` gravado a partir de um DAQ-E-S armazena os mesmos valores nas suas colunas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ao lado de cada espectro, para que o leitor consiga distinguir uma nuvem de um tripé que tenha sido derrubado. Em todos os outros modelos, essas colunas estão vazias.

O fluxo de atitude é reportado separadamente: o `daq pool-list` apresenta o `!! ATTITUDE STREAM FAULT (spectra unaffected)` enquanto está inoperacional e indica de quantas falhas recuperou posteriormente. Uma falha nesse fluxo nunca provoca uma lacuna no registo espectral. O `daq pool-discover` lista a unidade como `DAQ-E-S` (ou como `DAQ-E`, caso não responda à verificação de identidade no prazo de cerca de 2 s).

### Definições do dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **O `--static` sem o `--gateway` não guarda nenhum gateway** (qualquer gateway definido anteriormente é descartado), o que é adequado para uma rede de câmaras sem router.
- **Tem de existir um endereço estático na rede deste computador.** O Chloros rejeita qualquer endereço cuja sub-rede não inclua o endereço deste computador na rede do sensor: após o reinício, nada conseguiria aceder ao sensor para reverter a configuração. Aplique o `--force` apenas quando este computador for reendereçado ou encaminhado para a nova sub-rede.
- **A reinicialização é recusada enquanto o sensor estiver a gravar.** Interrompa primeiro a gravação (`pool-record --stop`). Uma gravação iniciada imediatamente antes de uma reinicialização é encerrada por esta, e os códigos `pool-set-network`, `pool-reboot` e `pool-update-firmware` terminam com uma linha `Warning:` que indica o nome do ficheiro de gravação (mesmo quando o sensor não volta a funcionar).
- **Após um reinício, o Chloros volta a ligar apenas o próprio sensor.** Aguarda no local onde o sensor se encontrava — e, após o `--auto`, também sob o seu nome `daq-e-<id>.local` — por uma resposta com o nome do sensor, nome do anfitrião e modelo, a partir do adaptador de rede (MAC) que possuía, e volta a ligá-lo no endereço que respondeu. Uma resposta com esse nome só é aceite com esse MAC; por isso, a partir de um computador que acede ao sensor através de um router, ou a partir do WSL2 na sua configuração de rede NAT predefinida, um sensor que o DHCP tenha movido para um novo endereço não é reconectado: o comando indica isso e apresenta o comando `pool-connect --eth-host <its address> …`, que o traz de volta com as definições de limite e exposição que possuía.
- **O comando `pool-update-firmware` rejeita um sensor cuja versão de firmware não consegue ler.** Um sensor que demora mais do que o tempo de espera a responder após o flash não sofreu uma falha: assim que responde, é reconectado com as definições do novo firmware preenchidas, e o comando avisa que a resposta foi lenta.
- **Uma atualização de firmware requer uma ligação TCP de entrada.** O sensor volta a ligar-se a este computador para recolher a imagem. O Chloros fica à escuta no adaptador que encaminha para o sensor (em todos os adaptadores, apenas quando não consegue identificar qual) e atende apenas o endereço do próprio sensor; qualquer outra ligação é encerrada sem ser atendida. No Windows, a regra de firewall `Chloros Backend (TCP-In)` do instalador permite a entrada (ver [Windows (.exe)](#windows-exe)); uma firewall que a bloqueie faz com que a atualização falhe com o código de erro `… device never connected back on TCP <port> …`.
- **No WSL2, utilize a rede espelhada** (`networkingMode=mirrored` em `%UserProfile%\.wslconfig`). No modo NAT predefinido, o endereço deste computador na rede do sensor é o próprio do WSL, pelo que todos os endereços estáticos são recusados (exceto `--force`), a transmissão em direto não chega ao WSL e a atualização do firmware não pode ser concluída, porque o sensor se liga novamente ao computador para receber a imagem.
- **O ID do sensor e o nome do anfitrião não se alteram.** Seja qual for o seu endereço, um DAQ-E continua a ser o `daq-e-<id>` e o `daq-e-<id>.local`, pelo que o `--eth-host daq-e-<id>.local` continua a localizá-lo.
- **A GUI mantém as mesmas definições**: separador «Sensores de luz», definições do sensor (engrenagem), «Rede». A GUI mantém a sua própria ligação a um sensor; por isso, se alterar um sensor que a GUI tenha aberto a partir da própria GUI: uma reinicialização a partir da CLI encerra a sessão da GUI com o mesmo.
- Os equivalentes no SDK são `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`.

---

## `chloros-cli project`

Abrir, ligar-se a e controlar um projeto Chloros guardado (uma pasta com `cameras.json` + `sensors.json` + `project.json`). Tudo passa pelo backend, pelo que a GUI e a CLI produzem um estado de hardware idêntico.

### Referência de subcomandos

| Subcomando | Finalidade |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Imprime o manifesto de dispositivos do projeto (câmaras, matrizes, sensores), lê a partir dos seus ficheiros e verifica os seus ficheiros de origem (ver [Um projeto movido](#a-moved-project---relink)). Não abre o projeto no backend: o projeto, já aberto nesse backend, bem como os seus dispositivos e gravações, permanecem tal como estão. Encerra o `1` quando nenhum dos ficheiros-fonte do projeto for encontrado. |
| `project devices PATH [--reconnect]` | Listar ou repetir a descoberta. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Ligar todas as câmaras / matrizes / sensores guardados. Cada matriz liga-se tal como o `lattice array-connect`: até 600 s, com os seus passos impressos em stderr (`[array <id>]    <s> s: <step>`; stdout é o relatório em JSON), e um conjunto não ligado nesse período é desfeito — as suas câmaras são repostas no estado em que foram encontradas — e reportado em `errors`, nunca ficando ligado. `--packed` define o transporte compactado de 12 bits de cada conjunto (padrão `auto`; um projeto não guarda configurações específicas por matriz; ver [Transporte compactado de 12 bits](#packed-12-bit-transport---packed)). Cada sensor de luz é aberto no conjunto DAQ do backend no seu endereço guardado, com o seu tempo de integração, média de fotogramas, exposição automática e limite guardados: um sensor sem limite de exposição selecionado recebe o valor `sunshine_cosine`, um sensor guardado sem configuração (`none`) permanece sem configuração, e um DAQ-E-S nunca tem um limite de exposição. O painel DAQ da GUI não faz parte do conjunto: feche primeiro um sensor nesse local. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Captura única de uma câmara ou matriz com nome (`--prefix` por predefinição, `capture`). `--orient` (matrizes): a matriz guarda os dados em bruto, que permanecem no formato nativo — isto indica que os produtos que o `process` cria a partir deles devem ser orientados, tal como o `array-capture --orient`. Padrão: nativo. `project burst` / `stream` e os parâmetros `capture` / `burst` / `stream` (`orient: true`) de uma receita têm a mesma opção. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Série de N fotogramas de uma câmara ou matriz especificada (`-n/--count` valor predefinido 5; `-i/--interval` intervalo entre fotogramas em segundos, valor predefinido 0). As sequências de matrizes eliminam a duplicação de grupos sincronizados repetidos (mecanismo de deteção de obsolescência), para que uma matriz em ciclo parcial não possa devolver N cópias de um fotograma; imprime os resultados por iteração. Uma captura que falhe a meio interrompe a sequência e sai do `1`, indicando quantos foram guardados antes disso. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Captura para disco em loop: o `--fps F` captura um segundo (padrão 3), um fotograma por captura de uma câmara, um grupo de fotogramas sincronizados de uma matriz (com o mesmo watchdog de obsolescência que o `project burst`), até que o `-n N` capture (por predefinição, `0` = até se premir Ctrl+C, o que apresenta o número de imagens guardadas e encerra o `0`, ou o `1` quando nenhuma imagem foi guardada ou se ocorreu uma falha no ficheiro; um Ctrl+C antes de o `-n N` capturar faz com que o `130` saia). Uma captura que falhe a meio interrompe o fluxo e sai do `1`, indicando quantos fotogramas foram guardados antes disso. O `--poll-interval` está obsoleto e é ignorado: continua a ser aceite para que os scripts mais antigos funcionem, com uma nota no stderr. |
| `project sensor read PATH NAME [--json]` | Último quadro do espectro. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Grava um ficheiro .daq para `--seconds` (padrão 10). Uma gravação que não contenha nenhuma leitura é comunicada no stderr e o `1` é encerrado. |
| `project run PATH RECIPE.yaml` | Executar uma receita de captura YAML/JSON. O `--dry-run` valida sem executar. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcula o alinhamento para uma matriz — consulte [a tabela de opções abaixo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Imprimir o perfil de alinhamento atual. |
| `project align clear PATH NAME` | Eliminar o perfil armazenado em cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Ajustar a transformação de um escravo (por predefinição, `0`, `0`, `0`, `1.0`: sem alterações). |
| `project align export PATH NAME --to FILE` | Guardar o perfil em JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carregar um perfil guardado. |

#### Opções do `project align calibrate`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alinhamento. **Estas grafias diferem das de `lattice align-calibrate`**, que aceita as formas abreviadas `orb` / `akaze` / `phase`; os dois comandos não são intercambiáveis nesta opção. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar o modelo para ajustar. |
| `--frames N` | `1` | Instantâneos de fotogramas sincronizados para média. |
| `--reference SN` | a câmara principal | Número de série da câmara de referência; todos os outros membros são distorcidos para se alinharem com ela. |
| `--max-features N` | `5000` | Limite de contagem de características ORB. |
| `--ratio-threshold F` | `0.75` | Teste da. |
| `--ransac-threshold-px F` | `3.0` | Limiar de pontos internos do RANSAC. |
| `--min-matches N` | `15` | **Limite de qualidade** — rejeitar a resolução se o número de correspondências válidas for inferior a este valor. |
| `--max-reproj-err-px F` | `4.0` | **Limite de qualidade** — rejeitar a resolução acima deste erro RMS de reprojeção. |
| `--checkerboard RxC` | — | Geometria da placa para `--method checkerboard`, por exemplo, `9x6`. |
| `--name PROFILE` | vazio | Nome do perfil incorporado no ficheiro JSON guardado. **Não é o nome da matriz** — esse é o `NAME` posicional. |

Os dois controlos de qualidade são a razão pela qual uma calibração pode ser bem-sucedida na resolução e, mesmo assim,
recusar-se a guardar: um perfil que falhe em qualquer um deles causaria silenciosamente um desalinhamento em todas as
capturas posteriores, pelo que é recusado em vez de ser guardado.

### Um projeto movido (`--relink`)

O `project.json` de um projeto nomeia cada ficheiro de origem — todas as imagens e registos do sensor de luz
— pelo seu caminho completo. Se mover a pasta do projeto, ou toda a pasta de projetos,
para outra unidade, esses caminhos continuarão a apontar para o local antigo.

Ao abrir um projeto, este corrige o que pode por si próprio: um ficheiro que se encontrava dentro da pasta do projeto
é procurado na pasta onde o projeto se encontra agora, e todos os caminhos encontrados nessa pasta
são redirecionados e guardados (o `project.json` é primeiro copiado como
`project.json.before-relink-<date>-<time>`). Isto acontece sempre que se abre — o da aplicação,
`chloros-cli project open`, `chloros-cli process` num projeto existente, os do SDK e os do `chloros-mcp`. Um caminho só é redirecionado para um ficheiro que se encontre nesse local.

Os ficheiros que estavam fora da pasta do projeto — uma sessão do Hub, uma importação através da opção «Adicionar Pasta» —
não podem ser encontrados dessa forma. Indique o nome da pasta em que se encontram agora:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` considera a pasta para onde os ficheiros foram movidos, ou qualquer pasta acima dela: a pasta de cada
ficheiro em falta é procurada nessa hierarquia com base nos nomes das pastas finais
(`Hub Projects/<session>/raw/<serial>`, depois `<session>/raw/<serial>`, ...), e cada
ficheiros dessa pasta são obtidos do local onde se encontram a maioria deles. `project open` apresenta
o que foi reorientado, quantos ficheiros de origem foram encontrados e as pastas onde os restantes se
encontravam. Ainda faltam alguns: um aviso, sair do `0` — o projeto abre com o restante. Nenhum
encontrado: um erro a indicar essas pastas e o `--relink`, sair do `1`; o backend, da mesma forma,
recusa-se a abrir tal projeto para `process` ou `project devices --reconnect` (409 HTTP `PROJECT_SOURCES_MISSING`).

### Exemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL de receitas

O `project run RECIPE.yaml` aceita um ficheiro YAML ou JSON que descreva uma sequência de ações:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ações suportadas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` utiliza as chaves `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (os nomes antigos `exposure_time`, `exposure_auto`, `gain`, `gain_auto` são aceites como aliases). Uma exposição, ganho ou valor-alvo definido desta forma é mantido como seu: o controlo automático de exposição do Chloros (incluindo a exposição medida pelo DAQ) não o altera. Uma exposição manual mantém o ganho tal como estava: no modo automático, a câmara continua a ajustar o ganho (em toda a sua gama) em direção ao brilho-alvo até definir `gain_db` ou `auto_gain: "Off"`. Um comando `auto_exposure: "Continuous"` posterior devolve a exposição ao modo automático, e o comando `target_brightness_auto: true` devolve um valor-alvo mantido ao valor-alvo automático dChloros. A ação `burst` aceita `name` (obrigatório), `count` (padrão 5), `interval` (segundos, valor predefinido 0), `output`, `format`, e `settings` (mesma configuração por câmara que `apply`); as sequências em matriz utilizam o mesmo watchdog de grupo recém-sincronizado que `project burst`.

No caso do passo `capture`, `burst` ou `stream`, o valor predefinido do `output` é o `output`. Um `output` relativo — o padrão incluído — é resolvido em relação à pasta a partir da qual executa o `chloros-cli project run`, e não à pasta do ficheiro de receita ou à pasta do projeto (na versão 1.2.0, foi colocado na pasta de instalação do backend).

Execute-o:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Cada execução é verificada primeiro em relação ao projeto.**Cada ação deve ter exatamente um dos verbos acima como chave — `- capture: {name: FrontLeft}`, e não `- action: capture` ou `- type: capture` — com um `name`, o projeto tem `cameras.json` / `sensors.json`, uma câmara ou matriz para `apply` / `capture` / `burst` / `stream` e um sensor de luz para o `sensor`; um `stream` necessita que `count` > 0 e um `wait` necessita de uma duração. Uma receita com qualquer um destes erros**não** executa nenhuma ação (`[recipe] error: …`, `[recipe] not run: …`, saída `1`), mesmo com o `stop_on_error: false`. As chaves que um passo não utiliza são um `[recipe] warning:`, tal como qualquer matriz ou sensor de luz que a receita controle e que não esteja ligado neste momento. Uma receita nunca liga dispositivos: execute primeiro o `chloros-cli project connect PATH`. O `--dry-run` imprime a receita, depois os mesmos erros e avisos, e o `[recipe] valid: …` / `NOT valid: …`, não executa nada e sai do `1` quando a receita apresenta um erro.

---

## Variáveis de Ambiente

| Variável | Efeito |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Substitui o URL do backend (padrão `http://127.0.0.1:5000`) — **respeitada apenas pelas famílias de comandos `lattice`, `project` e `daq pool-*`** (o comando `lattice network-analysis` utiliza o `--backend-url`). Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixam o `http://127.0.0.1:<port>` e ignoram esta variável (o literal IPv4 contorna a penalização de Windows `localhost`→`::1` de cerca de 2 s por pedido), pelo que visam sempre a máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Definido no ambiente do backend. `1` atenua a recusa por sobre-subscrição — uma ligação em array com `pin_resolution`, ou um [comando transitório de múltiplas câmaras](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — para um aviso sonoro, aceitando a perda de pacotes GVSP. Apenas para testes — consulte [Modelo de fps e burst da matriz](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Definido no ambiente do backend: `on` ou `off` define o valor predefinido de [transporte compactado de 12 bits](#packed-12-bit-transport---packed) para todos os arrays (padrão `auto`). Uma definição explícita de `--packed` continua a prevalecer. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (ou `true` / `yes`) desativa a [verificação automática de atualizações](#chloros-cli-update) da CLI do Linux na inicialização. |
| `CHLOROS_CLI_MODE` | Definido pela própria CLI; indica ao backend para ativar o processamento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignora o teste de fallback do GVSP (apenas resultados ICMP). **Isto desativa os pacotes jumbo, não se limita apenas a silenciar o registo** — a câmara responde a pings DF apenas até 1500 em cada caminho, pelo que este teste é a única forma de detetar pacotes jumbo. Poupa cerca de 1 s por câmara por ligação; custa cerca de 1,45× o limite da ligação se a rede *pudesse* ter transportado pacotes jumbo. O SDK avisa quando se define esta opção. |
| `CHLOROS_GVSP_SESSION_REUSE` | Definir no ambiente do backend: `0` mede o tamanho do pacote em cada ligação, em vez de uma vez por câmara por sessão do backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixa o tamanho do pacote GVSP em N bytes; ignora completamente a detecção. É preferível utilizar a configuração por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) em vez de a definir permanentemente: um tamanho fixado deixa de se adaptar à rede à sua frente, e fixar 9000 num caminho que não suporta pacotes jumbo faz com que **todas** as capturas atinjam o tempo limite com o `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Definir no ambiente do backend: o número máximo de segundos que um [comando transitório de múltiplas câmaras](#multi-camera-transient-sessions), ou um `lattice capture` de uma câmara que nada retém, demora a definir a exposição de cada câmara a partir da cena antes de guardar um fotograma (padrão `5`). `0` desativa esta funcionalidade: a exposição automática de cada câmara é então executada a partir do primeiro fotograma, começando no seu limite de exposição. |
| `CHLOROS_LOG_DIR` | Definido no ambiente do backend: a pasta para onde é enviado o seu registo detalhado. Padrão `%LOCALAPPDATA%\Chloros\logs` no Windows, `~/.cache/chloros/logs` no Linux (`$XDG_CACHE_HOME/chloros/logs` quando estiver definido). O serviço raiz do .deb define o `/var/log/chloros`, legível pelo grupo `adm` — consulte [Executar o backend como um serviço](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Local onde os ficheiros temporários são guardados. A CLI utiliza automaticamente o `/mnt/ssd/tmp`, se estiver presente. |

---

## Códigos de saída

| Código | Significado |
| --- | --- |
| `0` | Sucesso. |
| `1` | Falha genérica (a maioria dos erros de subcomandos). Um comando de captura também termina com o código `1` quando qualquer ficheiro que devia ser gravado não foi gravado, mesmo que os outros tenham sido guardados, e com o código `array-burst` / `array-record` e o `1` quando o gravador reporta um erro — consulte [Ficheiros cuja gravação falhou](#files-that-fail-to-write). O `process` também encerra o `1` quando as exportações de uma execução não puderam ser verificadas ou quando perdeu uma imagem, embora os outros produtos tenham sido gravados. |
| `2` | Erro de argumento (um sinalizador desconhecido ou em falta; também `time-sync set-priority` sem qualquer prioridade). |
| `130` | Interrompido por Ctrl+C: qualquer comando que seja interrompido — `lattice`, `project`, `daq` e `process` (`process` após o backend ter interrompido a execução, ver [Notas sobre o `process`](#notes)). Um comando cuja paragem normal é feita com Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` e `project stream` com o comando `-n 0` — encerra-se com base no que fez: `0` quando tudo foi gravado, `1` quando algo falhou. |
| `143` / `129` | Encerrado por SIGTERM / SIGHUP (128 + o sinal), após a execução da limpeza do comando: uma tarefa em execução foi interrompida e as suas câmaras foram recolocadas, uma execução de processamento foi interrompida no backend. Um `lattice continuous` ou outra gravação cuja paragem normal é feita com Ctrl+C também sai do `143`, e não do `0`. |

O `lattice network-analysis` encerra o `1` para os veredictos `needs_force_slip` e `error`, e o `selftest` encerra o `1`, a menos que todas as sete verificações sejam bem-sucedidas.

---

## Dicas para a resolução de problemas

- **«É necessário iniciar sessão»** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez neste computador.
- **«A pasta do projeto está demasiado profunda para este computador»** → Os caminhos longos do Windows estão desativados nesta máquina (configuração predefinida do Windows), pelo que nenhum caminho pode ultrapassar os 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. Nada foi processado. A mensagem indica a pasta de projeto mais longa que cabe: utilize uma mais curta (`-o` ou `chloros-cli set-project-folder`) ou ative os caminhos longos do Windows e reinicie o Chloros. O instalador do Windows apresenta esta opção como «Ativar caminhos longos do Windows», marcada por predefinição; caso contrário, utilize a Política de Grupo: *Configuração do Computador > Modelos Administrativos > Sistema > Sistema de Ficheiros > Ativar caminhos longos do Win32*.
- **«backend inacessível»** → Inicie a aplicação de ambiente de trabalho do Chloros (Windows) ou inicie o backend manualmente: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` no Windows (a pasta `backend` da sua pasta de instalação; não se encontra em `PATH`), `chloros-backend` no Linux. Ou verifique `CHLOROS_BACKEND_URL` se for remoto.
- **Os comandos `lattice` falham com a mensagem «Drivers da câmara LATTICE não encontrados»** → O backend não conseguiu carregar o runtime da câmara LATTICE com o qual está instalado. Todos os comandos `lattice` são executados no backend; a própria CLI não contém controladores de câmara. Reinstale o Chloros: o instalador do Windows e o ficheiro .deb instalam o runtime juntamente com o backend.
- **«Array connect» / «Array Settings» apresenta «FRAMES WILL DROP» ou «Reduza a ROI para ativar»** → O anel de receção da placa de rede do anfitrião é demasiado pequeno (normalmente redefini para 32 após uma atualização do controlador da placa de rede). Consulte [Configuração e ajuste da NIC do anfitrião](#host-nic-setup--tuning-lattice-arrays) — defina `ReceiveBufferLen=256`, `PendingReceives=64`.
- **A máquina bloqueia ao reiniciar/desligar; em seguida, o WMI apresenta o erro `Invalid class` / a NIC não é ativada / faltam unidades USB** → Driver desatualizado do adaptador USB 10GbE a causar o erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o controlador do adaptador — consulte [Configuração e ajuste da NIC do anfitrião](#host-nic-setup--tuning-lattice-arrays).
- **Linux: as capturas pertencem ao utilizador root, pelo que não é possível eliminá-las, e o registo do backend não se encontra no seu `~/.cache`** → O backend é executado como root através do `chloros-backend.service`. Mude para o `chloros-backend@<user>.service`, que o executa como utilizador — consulte [Executar o backend como um serviço](#running-the-backend-as-a-service-headless). Até lá, o registo do serviço root encontra-se em `/var/log/chloros` (legível pelo grupo `adm`).
- **Em Linux: uma unidade de backend é `failed` com a mensagem «Outra instância já está a ser executada» (estado de saída 98)** → Outro backend já ocupa a porta: apenas um pode ser executado de cada vez. Se duas unidades de backend estiverem ativadas, mantenha apenas uma: `sudo systemctl disable --now <unit>` para aquela que não utilizar — consulte [Executar o backend como um serviço](#running-the-backend-as-a-service-headless). Se apenas esta unidade estiver ativada, significa que um backend iniciado fora do systemd (por `chloros-backend`, `chloros-cli` ou um script do SDK) a está a ocupar: pare esse backend e, em seguida, execute `sudo systemctl start <unit>`.
- **Aviso de swap do Jetson** → Adicione swap baseado em ficheiro; a CLI apresenta os comandos exatos `fallocate` / `swapon`.
- **«backend não acessível» a partir de um comando `lattice`, `project` ou `daq`** → nenhum backend está a ser executado: esses comandos nunca iniciam um. Execute o comando `chloros-cli backend status` e consulte [Iniciar o backend](#start-the-backend).
- **«O `daq net ...` não está disponível no Chloros»** (Linux e Jetson) → Algumas das configurações próprias do DAQ-E estão reservadas para o serviço MAPIR. As configurações de rede, reinícios e atualizações de firmware são o `pool-set-network`, o `pool-reboot` e o `pool-update-firmware`; contacte o suporte da MAPIR para o restante.
- **"Os comandos diretos de hardware do DAQ requerem o pacote daq completo"** (Windows) → Utilize o `pool-*`, que controla o mesmo sensor através do backend, ou o `chloros_sdk.connect_daq_sensor()` a partir do Python.

---

## Ver também

- [Referência do SDK Python](sdk-reference.md) — equivalente programático de todos os comandos CLI.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) no manual do Chloros — os sensores, os seus limites e intervalo calibrado, gravação, rede DAQ-E.
- [Câmaras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) no manual do Chloros — ligação, definições da câmara, modos de captura, matrizes.
- Documentação online: `https://mapir.gitbook.io/chloros/cli`
