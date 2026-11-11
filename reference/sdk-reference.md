# Referência do Chloros Python SDK

**Versão:**

1.3.0**Gerado:**29/07/2026 19:19 ·**Revisado:** 06/11/2026**Pacote:** `chloros-sdk` (PyPI)**Público-alvo:** Otimizado para utilização por LLM; legível por humanos.**Âmbito:** Todas as classes, funções e auxiliares públicas expostas pelo `import chloros_sdk`, com exemplos que podem ser copiados e colados, abrangendo processamento de imagens, controlo de câmara única, matrizes sincronizadas, sensores DAQ e automação de projetos, além do servidor `chloros-mcp` para assistentes de IA.

Se precisar apenas dos pontos principais, avance para:
- [Instalação e início rápido](#instalação)
- [Smart-Connect para matrizes LATTICE](#smart-connect-para-câmaras-lattice)
- [Sessões de sensores DAQ](#sessões-de-sensores-daq)
- [Utilitários de câmara](#utilitários-da-câmara)
- [Matrizes Transitórias, Perfis de Alinhamento e Índices](#matrizes-transitórias-perfis-de-alinhamento--índices)
- [Pasta do Projeto e Receitas](#pasta-do-projeto--receitas)
- [Automatização de projetos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Fotogramas em tempo real no seu próprio fluxo de trabalho](#live-frames-in-your-own-pipeline)
- [Servidor MCP (Assistentes de IA)](#mcp-server-ai-assistants)

> **Alterações na versão 1.3.0** (o que um script existente da versão 1.2.0 deteta).
>
> **Alterações que um script da versão 1.2.0 poderá ter de tratar:**
>
> - **Uma versão que o backend não confirma agora gera um erro.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` num array aberto pela sessão, `DAQSensorSession.disconnect()` / `force_disconnect()` num sensor que abriu, e `ChlorosProject.disconnect_all()` geram o erro `ChlorosConnectError` quando o backend recusa a libertação ou não a confirma a tempo, porque o dispositivo pode ainda estar retido. Na versão 1.2.0, eram devolvidos silenciosamente. A saída de um bloco `with` continua a nunca gerar um erro.
> - **Um dispositivo emprestado nunca é desativado.** O `with attach_array(...)` e um `disconnect()` explícito sobre ele mantêm a matriz em funcionamento (na versão 1.2.0, sair do bloco desativava-o, mesmo quando a GUI o detinha); chame o `force_disconnect()` quando pretender encerrá-lo. Da mesma forma, um `connect_daq_sensor()` que encontrou o sensor já no conjunto, incluindo a deteção inteligente com sinalizador zero, mantém-no aberto no `disconnect()` e ao sair do seu bloco `with`.
> - **Os formatos PNG e JPEG aceitam apenas os produtos `display` e index.** `capture()` com `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) e um nível `radiance` ou `reflectance`, ou `processing="all"`, gera um `ValueError` antes de qualquer pedido; um `raw` / `debayered` de 12 bits é recusado pelo backend. Ver [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **O `connect_daq_sensor(sensor_id=...)` abre esse sensor ou gera** o `ChlorosConnectError`, listando o que encontrou; nunca recorre a outro sensor. Ver [Transporte Fixo](#pinned-transport).
> - **`frame_stream()` O  gera um erro em vez de ficar em silêncio:** `FrameFetchError` O , em caso de recusa, repetiria no tick seguinte (uma leitura de refletância sem qualquer DAQ incluída), após 60 s sem qualquer quadro e após 60 s sem um novo disparador. Consulte [Um disparador por leitura](#one-trigger-per-read--group_id).
> - **Um tempo limite de leitura ou uma ligação interrompida gera o erro `ChlorosConnectError`** em todas as chamadas smart-connect e o erro `analyze_array_network()`, em que a versão 1.2.0 gerava o erro de tempo limite ou de ligação subjacente.
> - **O `connect_array()` aguarda 600 s por predefinição (`timeout`, 180 s na versão 1.2.0), e uma ligação que desista de estabelecer é anulada.** A chamada envia o seu `timeout` para o backend, que repõe todas as câmaras no estado em que foram encontradas e não estabelece nenhuma ligação quando a ligação não é concluída a tempo; a chamada gera o erro `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). Na versão 1.2.0, uma ligação efetuada logo após as câmaras serem ligadas podia demorar mais do que a chamada e ligar a matriz depois de esta ter gerado a exceção. O `connect_all()` aguarda por cada matriz guardada da mesma forma (120 s na versão 1.2.0). Consulte [Assinatura `connect_array()`](#connect_array-signature).
> - **O `connect_array()` mantém o sensor completo.** Sem o `width` / `height` / `binning`, todas as câmaras são configuradas para o sensor completo e a taxa de fotogramas cai para o que o anfitrião e a rede fornecem; na versão 1.2.0 reduziu ou agrupou os fotogramas para manter a taxa elevada. Consulte [Matriz Sincronizada](#synchronized-array--arraysession-smart-prep).
> - **Um formato de píxeis Bayer numa câmara M3M (mono) corresponde à sua profundidade.** Uma M3M oferece apenas formatos Mono, e as predefinições designam os formatos Bayer: na versão 1.2.0, a câmara recusava o formato e mantinha o seu próprio, enquanto o `settings.pixel_format` designava o formato Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, um `CameraSettings(pixel_format=...)`, um `connect_camera(preset=...)` e um conjuntoformato partilhado definem agora o formato Mono com a mesma profundidade num M3M: `"BayerRG8"` e `high_speed` dão origem a `Mono8`; `"BayerRG12"`, `high_quality` e `triggered` resultam em `Mono12`; `cam.settings.pixel_format` nomeia-o. O mesmo se aplica no sentido inverso: um M3C nunca é colocado num formato Mono, pelo que o `"Mono12"` — a partir de qualquer um destes, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` ou o formato guardado de uma matriz — resulta em `BayerRG12` num M3C, independentemente de qual for a câmara mestre da matriz. Um `CameraSettings` que se passe mantém o seu nome, pelo que o mesmo objeto continua a solicitar a cada câmara essa profundidade na sua própria família. O `connect_camera(preset=...)` mantém o formato da predefinição (a versão 1.2.0 define a câmara para 12 bits após a aplicação da predefinição), e uma câmara aberta numa predefinição deixa de gravar o que captou no `PRESETS`. Qualquer outro formato que uma câmara recuse (um que não conste da sua lista `PixelFormat`, como o `RGB8` num M3M ou o `BayerRG16` num firmware que não o inclua) também não recebe um nome próprio: após uma ligação do `LatticeCamera`, `apply_settings()` ou `set_format()`, `cam.settings.pixel_format` e `packed_transport` indicam o formato em que a câmara transmite, e uma linha impressa indica o formato que foi rejeitado.
> - **`analyze_array_network()` indica a que ligação se aplica:** `pixel_format=None` (anteriormente `"BayerRG8"`) significa os formatos definidos pela ligação (Mono12 / BayerRG12), e o `packed="auto"` indica a decisão relativa ao transporte compactado, pelo que os fps indicados podem diferir dos da versão 1.2.0&#x27;. Um `pixel_format` que passar indica um M3M nessa profundidade. Consulte [Ajudante de Análise de Rede](#network-analysis-helper).
> - **O `CameraSession.capture()` devolve um `CaptureResult`** (ainda uma lista de dicionários de quadros) com `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. Um resultado `ArraySession.capture()` também inclui o `.alignment` (indicando se a distorção de alinhamento foi aplicada e, caso contrário, o motivo) e o `.oriented`.
> - **As capturas de matrizes são alinhadas por predefinição, tal como as da aplicação.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` e `ArrayHandle.capture()` / `capture_smart()` distorcem os ficheiros `debayered`, `display`, `radiance`, `reflectance` e ficheiros de índice no fotograma da câmara principal e recortam-nos à sobreposição dos membros (menor do que o sensor devido aos deslocamentos das câmaras, `XMP-mapir:AlignmentJSON` `"applied": true`), para que sejam os mesmos ficheiros que a captura da mesma cena pela aplicação Chloros; O `raw` mantém-se no formato nativo. Anteriormente, uma captura sem o `aligned=True` guardava todos os ficheiros no tamanho nativo, sem correção de distorção. O `aligned=False` mantém essa configuração. Os ficheiros `chloros-mcp``capture_array` e `start_capture_run` assumem o mesmo valor por predefinição. Consulte [Métodos `ArraySession`](#arraysession-methods).
> - **O `connect_all()` identifica um backend que rejeita o SDK.** Um backend em execução que esteja sem sessão ou num plano inferior ao Copper é reportado no `errors` com o seu próprio motivo (HTTP 401 / 403), e não como «nenhum backend Chloros está em execução», e cada chamada `connect_all()` verifica novamente o backend, pelo que chamá-lo após iniciar sessão funciona no mesmo objeto de projeto.
> - **Um `output_dir` relativo funciona com um backend local.** Os caminhos relativos são convertidos em absolutos em relação ao diretório de trabalho do seu script antes de serem enviados. Na versão 1.2.0, o backend resolvia-os na sua própria pasta de instalação, o que resultava na falha «Acesso negado».
> - **A função `numpy` é agora um requisito** do SDK (as chamadas live-frame devolvem matrizes numpy).
> - **Um nome de nível de exportação que não seja «one» é recusado.** `processing=` e `levels=` aceitam `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou um alias como `preview`; `processing=` e também `"all"`). Qualquer outro nome é recusado antes de a câmara ser acedida (400 HTTP `UNKNOWN_LEVEL`: `capture()` gera `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); a versão 1.2.0 guardou ou devolveu o `raw` para o mesmo.
> - **A funcionalidade «Texture Aware» é recusada para capturas LATTICE.** Num projeto que contenha capturas LATTICE, o `configure(debayer="Texture Aware (Slow, Highest Quality)")` gera o `ChlorosConfigurationError`, assim como o `process()` após um `configure()` que solicitou antes da importação. Um valor de «Texture Aware» que o projeto guardou anteriormente é definido como «Standard», com um `UserWarning`. As capturas LATTICE recebem sempre o demosaico padrão, pelo que a versão 1.2.0 aceitou a configuração e ignorou-a; a aplicação Chloros também não oferece essa opção para elas. Consulte [Valores suportados](#supported-values).
> - **As câmaras voltam ao estado em que foram encontradas — a menos que ocorra um `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, no final de cada função transitória (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) e a interrupção de `start_camera_continuous()` repõem cada câmara no estado em que a encontraram: a sua ligação do disparador e da linha de sincronização, o seu PTP e todas as definições da câmara alteradas durante a sessão (exposição e ganho, o alvo AE, limites e amortecimento, equilíbrio de brancos, nível de preto, formato de píxeis, binning e ROI, inversões e taxa de fotogramas). O comando `keep_settings=True`, por sua vez, guarda as definições de cada câmara na própria memória da câmara, para que esta inicie com essas definições; o comando `clear_settings=True` repõe essa memória para as definições de fábrica. Em qualquer dos casos, as câmaras ficam a funcionar de forma autónoma. Um projeto configura as suas câmaras a partir das suas próprias definições guardadas quando as liga. Consulte [Memória da câmara](#camera-memory--keep_settings--clear_settings).
> - **Todos os modos guardam um projeto da mesma forma.** Qualquer alteração numa câmara, num conjunto de câmaras ou num sensor de luz ligado a um projeto — a partir do SDK, da aplicação, da CLI ou do servidor MCP — é guardada no ficheiro `cameras.json` / `sensors.json`, e ao abrir o projeto em qualquer modo, os dispositivos são configurados a partir dele; um `ChlorosProject` atribui um nome à sua própria pasta no backend. Anteriormente, apenas a aplicação guardava o projeto. Consulte [Um registo de projeto, em todos os modos](#one-project-record-every-mode).
> - **Os ficheiros criados a partir de várias câmaras indicam cada uma delas.** Uma pilha multibanda, um índice em tempo real criado a partir das bandas de várias câmaras e uma imagem de captura em matriz `composite/` contêm o número de série, o modelo e o pacote de calibração de cada câmara de origem sob a forma de listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), e as listas EXIF `SerialNumber` apresentam os números de série separados por vírgulas. Anteriormente, não incluíam qualquer câmara ou apenas o número de série da primeira câmara. Ver [Ficheiros criados a partir de várias câmaras](#files-made-from-several-cameras).
> - **Apenas um ficheiro `raw` contém as etiquetas de calibração da câmara.** O conjunto compatível com MicaSense (`BlackLevel`, `RadiometricCalibration`, os conjuntos`Band{R,G,B}RadiometricCalibration` por canal, os conjuntos `MicaSenseVignetting*`) e a carga útil `CalibrationJSON` descrevem os valores brutos do sensor, pelo que são incluídos apenas no ficheiro de captura `raw`. Todos os outros ficheiros — um ficheiro de captura`debayered`, `display`, `radiance`, `reflectance` e ficheiros de índice, de gravação contínua e de— e todos os produtos `process()` — contém a identidade da câmara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, numa captura, o registo próprio do fotograma (`XMP-mapir:FrameJSON`: exposição, ganho, temperatura, formato de píxeis, hora da captura). Na versão 1.2.0, incluíam também o conjunto de dados brutos, que uma ferramenta de processamento (Pix4D, Metashape) aplicava aos píxeis que não eram valores brutos. O Processing determina a calibração desse ficheiro através do seu número de série e do `CalBundleSha256`.
>
> **Outras alterações:**
>
> - **A taxa de fotogramas automática de um conjunto de câmaras adapta-se ao que o conjunto fornece.** Uma taxa que perca mais de 1 % dos seus disparos é reduzida, avaliada ao longo de várias observações prolongadas em vez de uma única, e a taxa a partir da qual foi reduzida é tentada novamente de vez em quando. Uma taxa definida pelo utilizador permanece inalterada.
> - **A exposição automática está mais estável.** Mantém-se constante sob luz constante, estabiliza-se em segundos após uma alteração, ignora pequenos realces e restabelece o funcionamento de uma câmara que deixa de responder sob sol intenso. Consulte [Como funciona a exposição automática](#how-auto-exposure-behaves).
> - **O `import_images(recursive=True)` conta apenas as suas imagens e gravações.** Uma pasta que contenha o `camera_cal/` de um projeto deixa de ter esses ficheiros de calibração contabilizados como imagens e gravações, e um projeto que já os tenha incluído retira-os.
> - **Aplica-se o `configure(radiometric_output=...)`.** Agora define os produtos `Export radiance` / `Export reflectance` do projeto; anteriormente, as definições de produto específicas de cada projeto prevaleciam sobre esta, pelo que o `radiance`, o `sensor-response` e o `all` exportavam todos a radiação e a refletância.
> - **O `test_array_sync()` inicia a partir do estado de ligação das câmaras.** Ativa o PTP das câmaras para a execução, aguarda que os seus relógios se sincronizem e estabilizem antes do seu primeiro disparo (`ptp_lock`) — cada câmara fica sincronizada durante 120 s, cerca de 2 minutos a partir do arranque (o `ptp_settle_s=` define-o), uma vez que um relógio recém-sincronizado apresenta um desvio de até ~200 us durante cerca de 2 minutos — e repõe o PTP de cada câmara tal como o encontrou. Não se aguarda novamente por uma câmara que ainda esteja sincronizada de um teste anterior. Anteriormente, deixava o PTP desativado numa rede com um único comutador, onde nenhum DAQ-E respondia, e lia o relógio de funcionamento livre de cada câmara como um relógio PTP que não se sincronizaria («PTP não está sincronizado», em cada execução).
> - **Uma gravação `.daq` indica se a sua tampa foi selecionada ou assumida.** O DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e o Chloros não consegue detetar se está instalada uma, pelo que um sensor ligado sem uma tampa especificada recebe, por predefinição, o nome `sunshine_cosine`, uma *presunção*. Cada registo do Chloros (esquema 1.30) armazena agora o `cap_id_source` juntamente com o `cap_id`: `operator` quando se atribuiu um nome a uma tampa (incluindo o `none`), `auto_default` quando este era o valor predefinido. O processamento de um registo cujo condensador foi assumido, com o condensador DAQ do projeto definido como «Auto», gera um `[daq-dls] WARNING` que identifica o ficheiro: um DAQ-U sem filtro, sob a correção de luz solar, apresenta uma leitura cerca de 30 vezes superior. **Atribua um nome ao filtro que instalou** ao ligar (`connect_daq_sensor(cap_id=...)` ou `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, ou a lista suspensa de condensadores da GUI), ou defina o condensador DAQ do projeto, e o aviso desaparece.
> - **Já não aparecem janelas de consola pip no Windows.** Num PC com o Python instalado, o backend e cada um dos seus processadores de imagens executavam o `pip` uma vez no arranque, e cada um abria uma janela de consola. Isso já não acontece.
> - **As gravações `.daq` em tempo real sobrevivem a um corte de energia.**Uma gravação é gravada com um registo e uma sincronização completa do disco a cada commit, a partir do sensor, pelo que um disco lento atrasa o ficheiro, mas nunca as leituras. Uma gravação interrompida a meio de um commit (corte de energia, encerramento forçado) pode deixar o ficheiro `<name>.daq-journal` ao seu lado:**copie os dois em conjunto.** O Chloros reverte o diário na primeira vez que lê o ficheiro, pelo que a gravação abre com tudo o que foi submetido antes da interrupção (perdem-se cerca dos últimos 3 s); na versão 1.2.0, uma falha de energia no momento errado poderia tornar todo o ficheiro ilegível («a imagem do disco da base de dados está malformada»).
> - **O `DAQSensorSession.record_start()` num disco ocupado já não bloqueia o sensor**: as leituras são colocadas em fila e gravadas numa thread própria, e aquela que não pôde ser gravada é contabilizada no `status()` do `record_errors` (juntamente com o `record_last_error`).
> - **`LatticeCamera` direto:**um fotograma sem exposição/ganho por bloco apresenta agora a exposição e o ganho**reais** da câmara (o mecanismo de fallback lia um valor armazenado em cache que podia permanecer inalterado durante toda uma execução de exposição automática), e o `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` aplicam a mesma correção de interferências de entrada de 5 µs que os escravos por cabo do Chloros recebem (sem filtragem, um escravo disparava fotogramas sem que fosse enviado qualquer sinal de disparo).
> - **As câmaras acionadas por software já não ignoram um em cada dois sinais de disparo.** Uma câmara no modo de disparo por software coloca agora em fila um sinal de disparo que chegue durante a sua leitura (ou o Chloros aguarda que esteja pronta), tal como já acontecia com as câmaras sincronizadas por cabo.
> - **O `discover_daq_sensors()` designa um DAQ-E-S como `DAQ-E-S`.** O seu `model` era `DAQ-E` para cada sensor Ethernet; um que não responda à verificação de identidade no prazo de cerca de 2 s continua listado, como `DAQ-E`.
> - **Novo: `chloros-mcp`, um servidor MCP para assistentes de IA.** O Claude Desktop, o Claude Code, o Cursor e outros clientes MCP podem processar imagens (criar um projeto, importar capturas, escolher definições, processar, gerar relatórios sobre o que a execução exportou) e controlar câmaras LATTICE, matrizes e sensores de luz DAQ: ligar, pré-visualizar, definir a exposição e a cor, capturar, gravar. As suas ferramentas abrangem o que este SDK faz. É instalado juntamente com o SDK, e o ficheiro .deb para Linux coloca-o em `/usr/bin`. Consulte [Servidor MCP (Assistentes de IA)](#mcp-server-ai-assistants).
> - **Novo: utilitários de câmara**, os comandos `chloros-cli lattice` como chamadas do SDK: perfil de cor, saturação/contraste, realces, equilíbrio de brancos (único, automático, proporções fixas), predefinições, modelos de filtro, estado e atualização do firmware, alimentação (detecção, suspensão, ativação), calibração do alvo, adaptadores de rede (verificação, estimativa, correção), correção de vinheta em ficheiros, uma leitura DLS e a detecção do anfitrião. Consulte [Utilitários da Câmara](#camera-utilities).
> - **Novo: o restante da CLI como chamadas SDK**, pelo que a CLI, o SDK e o `chloros-mcp` oferecem as mesmas funcionalidades (a CLI mantém apenas o início de sessão, o aprovisionamento de um DAQ-E e os servidores DAQ diretos, por motivos de segurança): câmaras abertas com um único comando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), ficheiros de perfil de alinhamento (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), índices (`index_catalog()`, `compute_index()`, `capture_live_index()`), o «grandmaster» do PTP (`time_sync_status()` e afins), a pasta do projeto (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) e as receitas (`run_project_recipe()`, o executor próprio da CLI). Para sensores de luz: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` e `light_sensor_reflectance()` (refletância e índices ambientais + de objeto de dois sensores); para uma câmara: `CameraSession.status()` e `capture_repeated()`; para um conjunto: `ArraySession.capability()` e `set_member_settings()` (exposição, ganho, etc., de um membro). Consulte [Matrizes Transitórias, Perfis de Alinhamento e Índices](#transient-arrays-alignment-profiles--indices), [Pasta do Projeto e Receitas](#project-folder--recipes) e [Sessões de Sensores DAQ](#daq-sensor-sessions).
> - **Produtos de índices LATTICE melhorados** (NDVI e outros índices) a partir do `process()`, em todos os modos de processamento.
> - **Câmaras OCN do LATTICE: produtos de radianância, refletância e índices melhorados sob luz solar.** É aplicada uma correção de frota após a separação de bandas — em tempo real, nas capturas e no `process()`; as câmaras RGN permanecem inalteradas. Consulte [Saída radiométrica](#radiometric-output-lattice-multispectral-pipeline).
> - **Cada captura guarda a leitura DAQ utilizada, tal como a GUI faz.** `CameraSession.capture()` e `ArraySession.capture()` / `continuous()` gravam o `.daq` e o seu `.match.json` sempre que um DAQ é vinculado ou agrupado, incluindo uma captura apenas em formato bruto; na versão 1.2.0, uma captura em formato bruto ou sem debayering era guardada apenas com o `force_daq=True`. O `daq=False` desativa essa funcionalidade e o `daq_tolerance_ms=` define a tolerância de correspondência; ambos os resultados indicam o que foi guardado no `.daq_files` e no `.daq_matches`. Agora, uma captura aguarda a próxima leitura do DAQ para poder combinar as leituras de ambos os lados da exposição. Consulte [A leitura do DAQ que uma captura guarda](#the-daq-reading-que-uma-captura-guarda--arquivos-daq-e-correspondências-daq).
> - **O `ArraySession` tem fotogramas em tempo real e uma captura contínua do lado do servidor.** O `grab()` e o `frame_stream()` devolvem fotogramas numpy de um disparador (a mesma leitura que as APIs do projeto `ArrayHandle.grab()` / `frame_stream()`), e o `continuous()` guarda todos os grupos sincronizados no backend e devolve um [`ContinuousHandle`](#continuoushandle). Consulte [Frames em tempo real no seu próprio pipeline](#live-frames-in-your-own-pipeline).
> - **Um array `grab()` está associado a um gatilho** (`ArrayHandle.grab()` e o novo `ArraySession.grab()`). Na versão 1.2.0, 12 dos 20câmaras `ArrayHandle.grab()` abrangiam dois gatilhos. Agora, cada fotograma contém `group_id`, e uma matriz `frame_stream()` gera cada gatilho uma vez. Consulte [Um disparador por leitura](#one-trigger-per-read--group_id).
> - **Frames por banda e alinhados:** `grab()` / `frame_stream()` recebem `band=`, `align=`, `bit_depth=` e `refl_max=` em identificadores de câmara e de matriz. Consulte [Exportar uma única banda](#exporting-a-single-band--band).
> - **Novos métodos e argumentos `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (com `CaptureResult.group_seq` e `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` encadeia grupos consecutivos e pára quando a unidade está cheia.
> - **`ArraySession.record()` regista sem a GUI aberta.** Com a vista combinada da GUI fechada, o backend renderiza a própria vista: oNDVI ou o índice que passar como `expr=`. `RecorderHandle.stop()` gera uma exceção `ChlorosConnectError` para uma gravação que falhou, ou seja, que não incluiu nenhum fotograma, e mantém o resultado de paragem em `.result` (deixar um bloqueio `with` não gera exceção: verifique `rec.result`). Consulte [`RecorderHandle`](#recorderhandle).
> - **Novos métodos `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` e, para as definições próprias de um DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`. `record_stop()` devolve `warning` e `gaps`. Um reinício que encerrou uma gravação indica o seu nome na resposta ou, caso o reinício falhe, no `ChlorosConnectError` (`recording_closed`; consulte [Definições do dispositivo DAQ-E](#daq-e-device-settings)). Consulte [Métodos do `DAQSensorSession`](#daqsensorsession-methods).
> - **Um DAQ-E em pool que deixa de transmitir é reconectado pelo backend**, com as configurações que possuía, e uma gravação nele continua no mesmo `.daq` com a lacuna marcada no ficheiro; O `record_stop()` devolve as lacunas como `gaps`. Na versão 1.2.0, o sensor permanecia inativo até ser desligado e ligado novamente, o que encerrava a gravação. Consulte a referência da CLI [Um DAQ-E que deixa de transmitir](cli-reference.md#a-daq-e-that-stops-delivering).
> - **O `open_project()` funciona a partir de um SDK instalado.** O wheel (o instaladore do PyPI) inclui agora os carregadores de projeto de que necessita; anteriormente, o `open_project()` gerava o `ImportError`, referindo o `project_cameras` / `project_sensors` em qualquer SDK instalado via pip. O `connect_all()` liga as câmaras, matrizes e sensores de luz guardados através de um backend em execução (os sensores de luz através do seu conjunto DAQ), e um dispositivo guardado que não tem como localizar é listado em `errors` com a razão indicada, em vez de ser ignorado sem qualquer explicação. Uma matriz guardada que já esteja ativa no backend (nas interfaces gráficas, ou do `chloros-cli project connect`) é ligado e mantido em funcionamento, quando anteriormente era reportado como um erro («A câmara  <sn>já </sn>está <sn>na matriz<id>»). Ver [Métodos do `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` e `process_lattice_capture()` localizam o Chloros no Windows.** Procuram onde o instalador coloca o backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` por predefinição, uma pasta personalizada a partir do próprio registo do instalador) e ligam-se a um backend que já esteja a funcionar, sem precisarem de procurar nenhum. Na versão 1.2.0, apresentavam a mensagem «Backend do Chloros não encontrado» numa máquina instalada, a menos que se passasse o `backend_exe=`.
> - **Um backend que falha no arranque é imediatamente notificado.** Quando o `ChlorosLocal()` ou uma chamada de ligação inteligente inicia um backend que termina antes de responder, a chamada gera, em poucos segundos, o erro `ChlorosBackendError` a partir de `ChlorosLocal` e o erro `ChlorosConnectError` a partir das chamadas smart--connect, com o código de saída, o comando que executou e as últimas linhas que o backend apresentou. A versão 1.2.0 aguardava o tempo limite de arranque de 60 s e não indicava o motivo. Ver [Arranque Automático do Backend](#backend-auto-start).
> - **O backend continua a funcionar entre scripts.** Ao sair de um bloco `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` a regressar e o `chloros-mcp` a sair já não interrompe um backend iniciado pelo SDK; o script seguinte reutiliza-o. Este pára no `logout()` (a menos que a aplicação Chloros esteja aberta ou esteja em curso uma execução), no `shutdown_backend()`, ou por si próprio após 30 minutos sem que nada o esteja a utilizar (o `CHLOROS_BACKEND_IDLE_MIN` no ambiente do backend altera isso; `0`: nunca).
> - **Um backend iniciado pelo SDK é solicitado a encerrar, não é forçado a encerrar.** O `shutdown_backend()` permite que o backend liberte as suas câmaras e sensores de luz e conclua qualquer gravação; só é encerrado se ainda estiver a funcionar 10 s depois. A versão 1.2.0 encerrava-o imediatamente (no Windows, um encerramento forçado), o que poderia deixar um sensor de luz DAQ ligado a esse backend sem resposta. Um backend que o SDK encontrou em execução continua a ser ignorado.
> - **No Linux, o arranque automático nunca executa um `backend_server.py` a partir da pasta atual.** Sem nenhum backend instalado, um script iniciado numa pasta que contivesse um executava-o; agora, apenas o backend instalado é iniciado (ou, no caso do `ChlorosLocal`, aquele que o `backend_exe=` especifica).
> - **O `configure(indices=...)`**adiciona cada predefinição ao projeto para as bandas da respetiva câmara, incluindo um novo projeto, e emite um aviso (`UserWarning`) caso não consiga adicionar um nome.**`import_images()`** reporta as gravações `.daq` como `scan_count` e já não emite avisos sobre uma pasta de captura LATTICE.
> - **O `read_image_audit_tags()` funciona a partir de uma instalação via pip.** Ele lê os metadados da imagem por si próprio; na versão 1.2.0, devolvia o `{}` sem os componentes internos da instalação para ambiente de trabalho no caminho do Python.
> - **O `capture()`, imediatamente a seguir ao `connect_array()`**, aguarda até 20 s pelo primeiro grupo de fotogramas da matriz, em vez de apresentar uma falha com o código HTTP 503 «Matriz ainda a aquecer».
> - **`status()` reporta uma paralisação.** O novo campo `stalled` fica em True enquanto o ciclo de captura está em execução, mas não publica nada, e o `actual_fps` fica então em 0. Na versão 1.2.0, o `actual_fps` mantinha o seu último valor durante uma paragem, pelo que uma matriz em paragem apresentava o valor `actual_fps > 0`, enquanto as capturas falhavam com o erro 503 «Nenhum grupo de fotogramas sincronizado recente».
> - **Menos conjuntos de fotogramas incompletos e erros 503 de captura:** a entrada de disparo de cada câmara escrava está agora livre de interferências.
> - **Transporte compactado de 12 bits** (`connect_array(packed="auto")`) aumenta as taxas de fotogramas da matriz de sensor completo em cerca de um terço. Ver [Transporte compactado de 12 bits](#packed-12-bit-transport-packed).
> - **Exposição da matriz numa ROI:** uma ROI pequena já não limita a exposição de um membro ao período de funcionamento livre do sensor (4,9 ms a 768 × 304). Consulte [Exposição nos membros da matriz](#exposure-on-array-members).
> - **A exposição automática já não reduz a taxa de fotogramas automática de uma matriz.** Uma matriz sincronizada por cabo funciona à taxa que a sua rede e a leitura das câmaras permitem; a exposição automática preenche o período que essa taxa proporciona e, em seguida, aumenta o ganho. Uma exposição manual mais longa do que o período diminui a velocidade do disparador para se adaptar a ela, em vez de ser recusada. Consulte [Exposição nos membros da matriz](#exposure-on-array-members).
> - **`read_nodes()`**adiciona `frame_values` e `notes` enquanto a exposição automática está em execução (consulte [`read_nodes()` sob exposição automática](#read_nodes-under-auto-exposure--frame_values-and-notes)). Quando a câmara recusa uma exposição ou um ganho,**`set_settings()`** indica o intervalo que aceita e reativa o modo automático que tinha desativado para esse valor; `set_settings(target_brightness_auto=True)` é novo.
> - **`orient=`** em `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` e `ArrayHandle.capture()` / `capture_smart()` roda os produtos de acordo com a orientação da câmara. Consulte [Métodos `ArraySession`](#arraysession-methods).
> - **O `DLS()` requer o próprio `mac_address` do sensor.** Já não existe um MAC predefinido (era o endereço de uma unidade); o `DLS()` sem um gera o `DLSError`. Consulte [magnitudes da banda DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **O instalador do Windows** oferece as opções «Ativar caminhos longos do Windows» e «Excluir o backend do Chloros da verificação do Microsoft Defender», ambas marcadas por predefinição. Consulte [Passo 1](#step-1--install-the-chloros-platform-package).
> - **O .deb para Linux** instala o Python SDK para o sistema `python3` (anteriormente, apenas colocava o wheel no disco), aumenta os buffers de receção de sockets do kernel necessários para os fluxos das câmaras e adiciona um serviço backend que é executado como o seu utilizador. O pacote Jetson é o `chloros_1.3.0_arm64_jp6.deb`, para o JetPack 6, com o PyTorch CUDA do JetPack. Consulte [Passo 1](#step-1--install-the-chloros-platform-package) e [Passo 2](#linux-deb).

---

## Arquitetura em 60 segundos

O SDK é uma camada fina de Python sobre o backend do Chloros (o mesmo servidor Flask que a GUI de ambiente de trabalho e a CLI utilizam). Para automatização, importa-se o `chloros_sdk` e chamam-se métodos de alto nível; nos bastidores, cada chamada transforma-se num pedido HTTP para o backend local na porta 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente não `localhost`, que é resolvido primeiro como `::1` no Windows e demora cerca de 2 s por pedido quando se trata de um backend apenas IPv4). O backend controla o conjunto de hardware — câmaras, sensores DAQ, perfis de alinhamento, buffers de fotogramas — para que os scripts do SDK possam coexistir com a GUI sem disputarem portas seriais ou largura de banda da placa de rede.

Existem três interfaces que irá utilizar:

1. **`ChlorosLocal` + funções livres** (`process_folder`, `process_lattice_capture`) — Pipeline de processamento de imagens. Execute toda uma pasta através da calibração / remoção de pixelização / exportação de índice a partir de uma única chamada em Python.
2. **Controlos Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abra uma sessão de backend persistente para hardware em tempo real. O mesmo fluxo «smart-prep» que na GUI: sonda de rede, seleção automática de camada, PTP, inicialização AE, configuração do disparador GPIO.
3. **`ChlorosProject` / `open_project`** — Carrega um projeto guardado (pasta com `cameras.json` + `sensors.json` + `project.json`), liga as suas câmaras, matrizes e sensores de luz de uma só vez e controla as capturas com identificadores nomeados.

As superfícies 1 e 2 **iniciam automaticamente um backend local** caso ainda não haja nenhum em execução (o mesmo binário incluído que a GUI/CLI executam) — pelo que um script simples funciona a partir de um shell novo sem que seja necessário iniciar primeiro um backend: `ChlorosLocal` (e `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` e `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` nunca são iniciados: necessitam de um backend em execução. Passe `auto_start_backend=False` para desativar (por exemplo, ao apontar para um backend remoto, que nunca é iniciado). Consulte [Início automático do backend](#backend-auto-start). O Surface 3 comporta-se de forma diferente: o `open_project()` não aceita o parâmetro `auto_start_backend`, e o `connect_all()` nunca inicia um backend — ele tenta contactar o `http://127.0.0.1:5000` uma vez e, se não obtiver resposta, recorre ao controlo direto (sem backend) do dispositivo `lattice_sdk`. Isso requer o tempo de execução do SDK da Arena e o OpenCV (`cv2`) importável no seu Python, o que a roda do SDK não instala; sem eles, cada câmara e matriz guardadas são reportadas como um erro, com a respetiva razão. Os sensores de luz de um projeto não têm um caminho sem backend: sem nenhum backend em execução, cada sensor guardado é reportado como um erro, com a respetiva razão (ver [`SensorHandle`](#sensorhandle)). Apenas o `proj.process()` e o `stream(..., overlays=True)` criam de forma diferida um `ChlorosLocal()` (que inicia automaticamente).

Todos os três são auth: execute o `chloros-cli login` uma vez na máquina ou inicie sessão através da GUI do ambiente de trabalho. O que uma chamada sem uma sessão válida provoca depende da superfície (ver [Exceções](#exceptions)): o `ChlorosAuthenticationError` a partir do `ChlorosLocal` (e do `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` das chamadas smart-connect e das respetivas sessões, `status: "error"` com um `error_code` proveniente de `analyze_array_network()`, e uma entrada sob `errors` no relatório `connect_all()`.

Requisitos:
- Python 3.7+ (conforme declarado pelo pacote; desenvolvido/testado na versão 3.10)
- Chloros Desktop instalado localmente (o binário do backend vem incluído no instalador)
- Login ativo no Chloros+ em **qualquer nível pago do Chloros+ (Copper ou superior)**; o nível gratuito**Iron**não tem acesso a SDK/CLI. Isto é imposto**do lado do servidor**:** todas as solicitações com SDK ou CLI devem incluir tanto uma sessão ativa como um plano pago. Um plano inferior ao Copper recebe o código de erro `403` com `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` a partir do `ChlorosLocal`, e `ChlorosConnectError` para as chamadas smart-connect). Um chamador que tenha saído da sessão recebe, em vez disso, `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` de `ChlorosLocal`, `ChlorosConnectError` das chamadas Smart-Connect) — os dois são distintos porque a reexecução de `chloros-cli login` corrige o `401`, mas não consegue corrigir o `403`, que requer uma atualização do plano.
- A utilização offline é suportada durante o período de carência do plano: o nível é lido a partir da cache de validação do servidor (5 min) ou da cache de licenças assinadas e vinculadas ao equipamento (30 dias para planos mensais, até ao termo da subscrição para planos anuais). Quando esse período de carência expira, o plano passa a ser gratuito e o acesso ao SDK/CLI é suspenso até que a máquina consiga estabelecer ligação com o servidor pelo menos uma vez. O erro `chloros-cli status` (`GET /api/license-status`) permanece acessível no nível gratuito, pelo que o motivo fica visível.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS ou mais recente**, ou Jetson (JetPack 6). O Ubuntu 20.04**não** é suportado: as dependências do `.deb` derivam do que o backend utiliza como ligação, incluindo o `libc6 (>= 2.35)`, e o Focal vem com a glibc 2.31.

---

## Instalação

O SDK do Python é uma camada fina de Python sobre o backend Chloros. A menos que utilize um backend noutro computador (consulte [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel)), necessita do **pacote Chloros para ambiente de trabalho instalado localmente** (instalador para Windows ou Linux `.deb`), que inclui os sensores de luz DAQ. O pacote fornece o binário do backend, o tempo de execução do SDK da Arena para câmaras LATTICE e os pacotes de calibração.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Passo 1 — Instalar o pacote da plataforma Chloros

#### Windows (.exe)

1. Descarregue o ficheiro `Chloros Installer 1.3.0.exe` a partir da página de downloads.
2. Execute o instalador e siga as instruções do assistente. O caminho de instalação predefinido é `C:\Program Files\Chloros\`.
3. Inicie o Chloros pelo menos uma vez e inicie sessão com a sua conta Chloros+.

O instalador permite que o backend (`chloros-backend.exe`, nada mais) a atravessar a Firewall do Windows em todos os perfis de rede: entrada UDP, para transmissões de câmara, PTP, mDNS e a transmissão do DAQ-E, e entrada TCP apenas a partir da sub-rede local, para a ligação que um DAQ-E estabelece de volta ao computador durante uma atualização de firmware ([`update_firmware()`](#daq-e-device-settings)). O desinstalador remove ambas as regras. A API HTTP do backend permanece no `127.0.0.1` em qualquer dos casos.

O grupo **Definições do sistema** do assistente tem duas opções, ambas selecionadas por predefinição:

| Opção | O que faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** | Permite que as pastas de projeto fiquem em níveis mais profundos do que o limite de 260 caracteres do Windows. Com os caminhos longos desativados (a predefinição do Windows), uma pasta de projeto demasiado profunda para os seus produtos é rejeitada antes de qualquer processamento — consulte [Resolução de problemas](#dicas-de-resolução-de-problemas). | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender** | Impede a verificação em tempo realde cada ficheiro que uma captura de câmara grava. A taxas de captura elevadas, a verificação consome cerca de 30 % da taxa: medido em ROI 512 num conjunto de 9 câmaras, a captura RAW contínua atingiu 52,7 capturas/s com verificação e 77,7 capturas/s com o backend excluído. | `/MERGETASKS="!defenderexclusion"` |

Uma captura contínua reporta a carga do Defender como `defender_cores` (ver [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instale com o `apt install ./…`, e não com o `dpkg -i`: o apt também instala os pré-requisitos do Python SDK, que o pacote recomenda em vez de exigir (ver [Passo 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Para Jetson Orin / Orin NX / Orin Nano no JetPack 6 (L4T R36). O seu PyTorch é a compilação CUDA do JetPack 6, pelo que o processamento na GPU é executado na GPU do Orin; utiliza as bibliotecas CUDA e cuDNN que o JetPack 6 instala (`sudo apt install nvidia-jetpack` numa imagem que não as inclua).

#### Linux: buffers de receção e o serviço de backend

Ambos os ficheiros .deb aumentam para 32 MB os buffers de receção de sockets do kernel necessários para os fluxos das câmaras LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). Com a configuração padrão do kernel, os pacotes da câmara são descartados no kernel e um array perde fotogramas e taxa de fotogramas; o `chloros-cli lattice network` avisa quando os buffers estão baixos. Consulte a secção [Hosts Linux — buffers de receção de socket](cli-reference.md#linux-hosts--socket-receive-buffers) da referência da CLI.

Numa máquina sem monitor, execute o backend como um serviço **na sua conta de utilizador**, para que um script encontre um em execução (o `connect_all()` nunca inicia um backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

O backend passa então a ser executado como si: as capturas que grava são suas e utiliza o seu `chloros-cli login`. O `chloros-backend.service` do utilizador root continua instalado nas máquinas que o tenham ativado, mas todos os ficheiros que um backend do utilizador root gravar pertencem ao utilizador root. Consulte a secção [Executar o backend como um serviço](cli-reference.md#running-the-backend-as-a-service-headless) da referência da CLI.

Um backend executado pelo serviço é partilhado e nunca é interrompido por um cliente: os scripts do SDK e o `chloros-cli` utilizam-no tal como o encontram (o `ChlorosLocal.shutdown_backend()` interrompe apenas um backend que o próprio `ChlorosLocal` tenha iniciado), e o `GET /api/status` denomina a unidade como `service`. Reinicie-o com o `sudo systemctl restart chloros-backend@$USER.service`. A atualização do pacote interrompe os serviços de backend durante a instalação, pelo que um script em execução perde as suas câmaras e sensores e, em seguida, reinicia todos os que estão ativados; a remoção do pacote interrompe-os e desativa os que estão ativados. Apenas um backend funciona de cada vez, por isso ative uma unidade: uma segunda unidade recusa-se a iniciar e permanece como `failed` (estado de saída 98, não reiniciado), e o seu `systemctl status` explica o motivo. Para alternar entre unidades, desative aquela que não estiver a utilizar.</id></sn>

### Passo 2 — Instalar o Python SDK

**O instalador do Chloros inclui um pacote «wheel» do SDK correspondente.** Todos os instaladores para Windows e os ficheiros .deb para Linux colocam no disco um ficheiro `chloros_sdk-X.Y.Z-py3-none-any.whl` que corresponde exatamente à versão da GUI, da CLI e do backend. Não é necessário acompanhar o PyPI para se manter em sincronia.

#### Windows

O instalador executa automaticamente o `pip install` com o pacote «wheel» incluído, utilizando o Python do seu sistema (é preferível o executável `py.exe`; em caso de falha, recorre-se ao `python -m pip`). Não é necessária qualquer ação — o `import chloros_sdk` funciona no seu ambiente Python após uma instalação bem-sucedida. Se não houver Python instalado no sistema, o instalador ignora silenciosamente este passo e a GUI + CLI continuam a funcionar.

#### Linux (.deb)

Ambos os ficheiros .deb (x86_64 e Jetson) instalam o SDK para o sistema `python3`, pelo que o `import chloros_sdk` funciona após a instalação, sem necessidade de pip nem de ligação à rede. Os pacotes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e os carregadores de projetos) são colocados no `/usr/lib/python3/dist-packages`, e o comando `chloros-mcp` do SDK (o [servidor MCP](#mcp-server-ai-assistants)) no `/usr/bin`, todos pertencentes ao pacote: uma atualização substitui-os e a remoção do pacote elimina-os.

Os próprios pré-requisitos do SDK, `python3-numpy` e `python3-requests`, são **recomendados, não obrigatórios** (a GUI, a CLI e o backend são autónomos): O `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` num Jetson) instala-os, ao passo que o `dpkg -i` não o faz. O resultado da instalação indica a versão do SDK que o `python3` importa, quaisquer pré-requisitos em falta, e qualquer `chloros_sdk` noutro local do caminho do `python3` que substitua a cópia do pacote (como, por exemplo, um instalado anteriormente com o `pip install --user`).

Num virtualenv, que não tem acesso aos pacotes do sistema, instale o wheel incluído no mesmo:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implementações do Jetson em ambiente isolado (air-gapped), este processo é totalmente offline — o SDK e o wheel já se encontram no disco.

#### PyPI Público

Para anfitriões apenas com pip (sem pacote de ambiente de trabalho Chloros instalado; o SDK, nesse caso, controla um backend noutro computador, ver [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

O PyPI disponibiliza a versão estável mais recente. Mantenha o SDKna mesma versão que o backend do Chloros que ele controla (consulte [Versões e compatibilidade](#versioning--compatibility)). O auxiliar DAQ-M `DLS` também necessita do `bleak`: `pip install "chloros-sdk[camera]"` (ou `pip install bleak`).

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **É necessária uma subscrição do Chloros+.** Todas as chamadas ao SDK requerem um login ativo no Chloros+. Execute o `chloros-cli login user@example.com 'YourPassword'` uma vez por máquina; as credenciais são armazenadas em cache no `~/.chloros/`.

### Preciso do Pacote para Computador?

O pacote pip, por si só, **não** é suficiente para a maioria dos fluxos de trabalho. Eis o que cada interface do SDK necessita:

| Interface do SDK | Necessita do Pacote para Computador? | Porquê |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sim**(para iniciar um)**/ Não** (um backend já acessível) | Utilizar um backend em execução (o da aplicação de secretária ou outro script iniciado); caso contrário, iniciar automaticamente o que está instalado: `/usr/lib/chloros/chloros-backend` (Linux) ou `<install folder>\backend\chloros-backend.exe` (em Windows; `C:\Program Files\Chloros\backend\` por predefinição, sendo que uma pasta de instalação personalizada é lida a partir do registo do instalador). Passe `backend_exe=` para lhe atribuir um nome à sua escolha. |
| `chloros-mcp` ([servidor MCP](#mcp-server-ai-assistants)) | **Sim**(para iniciar um)**/ Não** (já existe um backend acessível) | Baseado em `ChlorosLocal`, pelo que é igual à linha acima. |
| `read_image_audit_tags` | **Não** | Lê os metadados da própria imagem processada (apenas na biblioteca padrão do Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sim**(local)**/ Não**(remoto) | Clientes HTTP puros através do backend. Backend local → é necessário um pacote para ambiente de trabalho. Backend remoto → `backend_url=`**através de um túnel** (ver Modo de Backend Remoto — os backends fornecidos ligam-se apenas ao loopback). |
| `ChlorosProject` / `open_project` | **Sim** | Acede a projetos guardados através do backend. |
| Classes LATTICE diretas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sim** | Requer o runtime do Arena SDK, que vem incluído no pacote para computador, e o OpenCV (`cv2`) no seu Python, que nem o pacote para computador nem o SDK instalam (`pip install opencv-python`). Caso contrário, o `CAMERA_AVAILABLE` é o `False` na importação. O `DLS` também necessita do `bleak`. |

### Modo de backend remoto (host apenas com pip, através de túnel)

> **O backend instalado não está acessível através da LAN.** Este escuta apenas
> no loopback (`127.0.0.1` / `::1`), pelo que o `backend_url="http://<lan-ip>:5000"`
> **não consegue aceder a um Chloros instalado**. Para controlar um backend noutro
> computador, reencaminhe você mesmo a sua porta de loopback e aponte o SDK para o
> túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Os anfitriões sem interface gráfica / CI / robótica podem manter uma máquina com a instalação completa do ambiente de trabalho como o «servidor Chloros» e o `pip install chloros-sdk` em todos os outros locais — mas o transporte entre eles é o túnel configurado pelo utilizador acima, e não um URL direto da LAN.

> **`ChlorosLocal` num host apenas com pip.** O `ChlorosLocal(backend_url=BACKEND)` liga-se a um backend ao qual consegue aceder sem uma instalação de ambiente de trabalho: procura um binário de backend local apenas quando precisa de INICIAR um (nenhum responde e `auto_start_backend=True`). Até à versão 1.2.0, procurava primeiro o binário e gerava o erro `ChlorosBackendError` («Backend Chloros não encontrado…») num anfitrião apenas com o pip, mesmo com um backend acessível. As pastas que se indicam (`process_folder`, `import_images`) são verificadas nesta máquina e lidas pelo backend; por isso, com um backend remoto, devem indicar a mesma pasta em ambos, como, por exemplo, uma unidade partilhada.

### Fluxo de trabalho apenas com DAQ (host apenas com pip)

O `pip install chloros-sdk`, por si só, não consegue controlar um sensor DAQ: um sensor DAQ é sempre controlado através de um backend do Chloros com o `connect_daq_sensor()`:

- **Sensor nesta máquina:** instale aqui o pacote de ambiente de trabalho do Chloros. O `connect_daq_sensor()` inicia o seu backend se nenhum estiver a ser executado.
- **Sensor noutro computador:** esse computador necessita do pacote para ambiente de trabalho; aceda ao seu backend através do túnel no [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel). O backend abre o sensor no seu próprio computador, pelo que um DAQ-U está ligado a essa máquina, um DAQ-M está dentro do alcance Bluetooth da mesma e um DAQ-E está numa rede à qual este consegue aceder.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

De qualquer forma, o backend aplica o plano «Chloros+»: inicie sessão na máquina que executa o backend (`chloros-cli login` ou a aplicação para computador) com qualquer nível pago (Copper ou superior). Um backend sem sessão iniciada e um plano inferior a Copper geram ambos o código de erro `ChlorosConnectError`, indicando o motivo do backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Introdução rápida

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Índice da API de nível superior

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Processamento de Imagens — `ChlorosLocal`

A classe principal do pipeline. Inicia o backend quando é instanciada (se nenhum estiver a responder e `auto_start_backend=True`), cria/configura projetos, monitoriza o progresso e devolve resumos após a execução.

### Construtor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Métodos

| Método | Descrição |
| --- | --- |
| `create_project(project_name, camera=None)` | Cria um novo projeto (opcionalmente com um modelo de câmara como o `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Abre um projeto guardado (a sua pasta ou o seu nome na pasta de projetos) e torna-o no projeto aberto. Os caminhos de um projeto movido são redirecionados para a pasta em que se encontra agora; `relink` indica o nome da pasta onde se encontram agora os restantes ficheiros em falta. Devolve a resposta do backend com `source_files`; lança a exceção `ChlorosProjectFilesMissingError` quando não é possível encontrar nenhum. Consulte **Um projeto movido** abaixo. |
| `import_images(folder_path, recursive=False)` | Importa imagens RAW/TIF/JPG/DNG **e `.daq` gravações do sensor de luz**. Devolve `count` (imagens) e `scan_count` (gravações). Avisará apenas se a pasta não contiver nenhum dos dois. |
| `export_light_sensor(daq=True, csv=True)` | Grava os ficheiros calibrados `.daq` + `.csv` para cada registo do sensor de luz do projeto, na pasta `<project>/Light Sensor/`. Consulte [Gravações do sensor de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Definir os controlos de processamento. `indices` são nomes de predefinições (`NDVI`, `GNDVI`, `OSAVI`, …), cada um adicionado ao projeto para as bandas da respetiva câmara; um nome que não seja uma predefinição, ou que as bandas não consigam formar (`NDRE` necessita de RE e NIR, que nenhuma câmara fornece sozinha), recebe um nome do tipo `UserWarning` e não tem imagem de índice. `daq_log_path` designa um registo do sensor de luz (um ficheiro `.daq` ou `.csv`, ou uma pasta com esses ficheiros) com a qual cada fotograma é comparado, tanto no LATTICE como no Survey3, em vez das gravações importadas com as imagens; se for indicado um caminho que falte ou que não contenha qualquer gravação, são utilizadas as gravações importadas. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executa o pipeline. Devolve o `{"status": "complete", "async": False}`, além de uma chave `summary` quando o backend a fornecer — consulte [Resumo pós-execução e sugestões](#post-run-summary--hints) — `export_check_failure` quando as exportações da execução não puderam ser verificadas e `image_failure` quando se perdeu uma imagem. `wait=False` devolve `{"status": "started", "async": True}` de uma só vez. |
| `get_config()` / `get_status()` / `status()` | Inspecionar o estado do backend. |
| `logout()` | Sair: limpa as credenciais armazenadas em cache e liberta o slot do dispositivo desta máquina (`device_deregistration` no resultado). Uma execução de processamento em curso não é interrompida: termina no modo em que foi iniciada (`run_in_progress: true` e `run_mode` no resultado), e a execução seguinte utiliza o processamento padrão. |
| `shutdown_backend()` | Interrompe o backend se esta instância o tiver iniciado; um backend que se encontre em execução é deixado em paz. Deixar um bloco `with` ou a instância a ser submetida à recolha de lixo não o interrompe: o backend continua a funcionar para o próximo script e pára por si próprio após 30 minutos, caso não haja nada a utilizá-lo. É solicitado ao backend que desligue, pelo que este liberta as suas câmaras e sensores de luz e conclui primeiro qualquer gravação em curso; só é encerrado se ainda estiver a funcionar 10 s depois. |
| `discover_cameras()` | Deteta câmaras LATTICE **através do backend desta instância** (`/api/camera/discover`). Devolve uma lista de dicionários (`serial`, `model`, `ip`, …) — com a mesma estrutura que a GUI/a CLI apresentam. Lista vazia se não for encontrada nenhuma ou se o backend estiver inacessível. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura um único fotograma**através do backend**(iniciado automaticamente por este identificador) para que receba a mesma preparação que a GUI/CLI (padrão de 12 bits, reutilização do pool, metadados de calibração incorporados). Resolver o alvo com `serial=` ou `device_index=`; passe `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Devolve o dicionário de metadados antigo (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Produz fotogramas de pré-visualização compostos por sobreposição a partir de uma câmara agrupada — cliente MJPEG leve através da rota `/api/camera/<serial>/stream-annotated` do backend (zebra / grelha / retículo / histograma / picos / ponto desenhado do lado do servidor). `decode=True` gera matrizes BGR e requer o OpenCV (`cv2`), que o SDK não instala; o `False` gera bytes JPEG em bruto. Também acessível por projeto como `ChlorosProject.stream(overlays=True)`. |

**Uma execução em andamento mantém o seu projeto.** Enquanto uma execução do Processing estiver em curso — seja deste script, da aplicação, do `chloros-cli process` ou do `chloros-mcp` —, o backend recusa-se a criar, abrir, importar ou alterar as definições de processamento de um projeto (409 HTTP `PROCESSING_IN_PROGRESS`): o `create_project()` e o `import_images()` geram o `ChlorosProcessingError`, e o `configure()` gera o `ChlorosConfigurationError`, e o `process()` com um modo diferente do que está em execução gera o `ChlorosProcessingError` (uma execução mantém o modo em que foi iniciada), cada uma com a mensagem do backend. Uma captura guardada no projeto junta-se a este quando a execução termina (a resposta `project_add` indica isso). Aguarde até que a execução termine (`get_status()`) e, em seguida, continue.**Um projeto movido.** O `project.json` de um projeto identifica cada ficheiro de origem (todas as imagens e registos do sensor de luz) pelo seu caminho completo; por isso, ao mover a pasta do projeto — ou toda a pasta de projetos para outra unidade —, estes continuam a referir-se ao local antigo. Cada vez que o programa é aberto, corrige o que pode por si próprio: um ficheiro que se encontrava dentro da pasta do projeto é procurado na pasta onde o projeto se encontra agora, e cada caminho encontrado nesse local é redirecionado e guardado (o `project.json` é primeiro copiado como `project.json.before-relink-<date>-<time>`). Um caminho só é redirecionado para um ficheiro que se encontre nesse local. Os ficheiros que estavam fora da pasta do projeto — uma sessão do hub, uma importação de «Adicionar Pasta» — só são encontrados com o `relink=<the folder they are in now>` (ou qualquer pasta acima desta). O `open_project()` devolve o `source_files`: `total`, `found`, `missing`, `missing_roots` (as pastas onde se encontravam os ficheiros em falta), `missing_outside_project` e `relink` (o que foi redirecionado, de onde para onde). Um projeto com alguns ficheiros em falta abre-se com o restante; um projeto em que não se encontram ficheiros em falta não é aberto, e o `open_project()` gera o erro `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Os erros `project open --relink` e `chloros-mcp` da CLI`open_project` (`relink_folder`) da CLI fazem o mesmo.

Utilize como gestor de contexto para garantir a limpeza:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

Um DAQ-U / DAQ-M / DAQ-E pode ser gravado **sem** o seu pacote de calibração. É isso
que os gravadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fazem por predefinição: gravam as contagens brutas dos sensores e marcam o
ficheiro para que o Chloros recupere a calibração de fábrica desse sensor **pelo número de série** — primeiro na cache local
primeiro, depois na nuvem MAPIR — e aplica-a na importação.

O Chloros volta a gravar o resultado como dois produtos por gravação, sob
`<project>/Light Sensor/`:

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | O arquivo reprocessável — com o mesmo esquema de uma gravação em tempo real, agora indicando o pacote que o produziu. A reimportação **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grelha de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas (potência total, lux fotópico/escotópico, PPFD e a sua divisão em azul/verde/vermelho, comprimento de onda de pico). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Apenas sensores sem pacote (DAQ-A).** Contagens espectrais brutas do sensor — *não* irradiação. Ver abaixo. |

`<name>` é o nome do próprio ficheiro da gravação. Duas gravações em pastas diferentes podem partilhar
uma — leituras por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) guardadas no
mesmo milésimo de segundo por duas câmaras — e cada uma continua a obter os seus próprios produtos: a primeira mantém
`<name>`, a seguinte acrescenta as pastas que a distinguem **após** o nome
(`<name>__<capture>_daq_calibrated.daq`), pelo que a data e a hora da gravação continuam a figurar no início.

O `process()` realiza esta exportação como uma das suas etapas. **Não** requer imagens:
um sensor de luz utilizado isoladamente constitui um fluxo de trabalho de primeira classe, e um projeto deste tipo tem zero
imagens por definição.

**As gravações DAQ-A são exportadas como contagens brutas.** A família DAQ-A é anterior ao sistema de pacotes por série
e não tem nenhum pacote para obter — em vez disso, é calibrada no terreno em relação a um
alvo de refletância, razão pela qual nunca precisou de um. Essas gravações são exportadas
com a raiz `_raw` em vez de `_calibrated`: um nome de ficheiro diferente em vez de um sinalizador
dentro do ficheiro, porque a informação tem de se manter intacta ao ser enviada por e-mail apenas como um nome simples. O
cabeçalho `.csv` indica `raw spectral sensor counts (NOT irradiance)` e avisa que os
valores são comparáveis **dentro** do ficheiro — exatamente para isso que serve a calibração baseada em alvo
para esse fim — e não entre sensores. As colunas fotométricas dependentes da potência (potência total,
lux fotópico/escotópico, PPFD) apresentam o valor **NULL** em vez de serem integradas a partir das contagens.

Um DAQ-U / DAQ-M / DAQ-E cujo pacote simplesmente não tenha sido possível obter continua a ser **ignorada**,
não sendo gravada em formato bruto: nesse caso, o pacote existe e «reconectar e reprocessar» é um conselho válido.

As gravações antigas **v1.01 / v1.02** (um DAQ-A-SD grava estas) não contêm uma época por leitura,
apenas a hora de gravação do ficheiro. O comparador de imagens↔fluxo descendente continua a rejeitá-las — comparar um
fotograma com uma data de gravação seria um erro impercetível — mas o exportador lê-as, e o
ficheiro CSV apresenta `clock=daq_created_on`, pelo que o produto indica em que relógio se encontra.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Uma gravação cujo pacote de calibração não pode ser obtido (offline ou um sensor sem
calibração no ficheiro) é reportada sob `skipped` **com o motivo**. Nunca é
gravado como um ficheiro «calibrado» contendo contagens brutas — ligue-se à Internet e
execute novamente, e a exportação será concluída.

### Chamadas de retorno de progresso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumo pós-execução e sugestões

Após a conclusão, o `process()` recupera o `GET /api/processing-summary` e anexa o corpo como `result["summary"]`. A recuperação é feita com o melhor esforço possível e nunca bloqueia um retorno bem-sucedido — se o resumo não estiver disponível, o `process()` recorre à forma simples do `{"status": "complete", "async": False}`. Cada entrada no `summary["hints"]` — frases completas com a correção sugerida, por exemplo, por que razão uma execução produziu zero resultados — é também reemitida como um `UserWarning` em Python, pelo que as execuções com zero resultados são autodiagnósticas, mesmo que nunca se inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` é a parte legível por máquina:

| Chave | O que conta |
| --- | --- |
| `models` | Grupos de câmaras na execução. |
| `images_in_groups` | Imagens de origem nesses grupos. |
| `targets_found` | Alvos de refletância detetados. |
| `images_calibrated` | Imagens para as quais a execução criou um produto de refletância. |
| `images_failed` | Imagens que falharam e não foram exportadas. |
| `exported_files` | **Ficheiros de produtos de imagem que a execução gerou**: todos os produtos (desbayering, pré-visualização, radianância, refletância, índice, renderizações de alvos), em todos os formatos. |
| `reflectance_files` | Os produtos de refletância entre eles (`Reflectance_Calibrated_Images`), quer tenham sido calibrados por um alvo, quer pela irradiação do sensor de luz (DAQ). Uma execução sem alvo que, mesmo assim, tenha gerado dados de refletância, indica isso numa nota; apenas uma execução que não tenha gerado nenhum é informada de que as suas saídas de refletância estão em falta. |
| `clamp_masks_written` | As [máscaras de limitação de desmistura](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) indicadas ao lado delas — são complementos, não resultados, pelo que não constam em `exported_files` (a CLI contabiliza-as separadamente da mesma forma). |
| `captures_left_out` | Fotogramas em bruto na pasta `captures` do projeto que esta execução não capturou. Uma execução aguarda, ao iniciar, o que foi capturado anteriormente, pelo que este valor é 0, a menos que uma captura tenha sido guardada durante a execução ou que o tempo de espera tenha expirado. O `summary["captures_left_out_note"]` indica o que fazer com elas. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Gravações do sensor de luz, contadas separadamente de forma intencional — provêm de uma fase diferente e existem para execuções sem qualquer imagem, pelo que incluí-las faria com que uma execução apenas de DAQ parecesse ter exportado imagens. |

Juntamente com estes: `summary["output_dirs"]` (todos os diretórios em que foram gravados),
`summary["light_sensor_export"]`, `summary["stopped"]` (verdiro quando o utilizador interrompeu a
execução, para que as contagens parciais não sejam interpretadas como uma execução concluída com produção insuficiente), e
`summary["groups"]` (a repartição por grupo).

O `exported_files` é registado pelo pipeline **à medida que escreve**, não sendo extraído posteriormente dos
objetos de imagem do projeto. As estratégias paralelas e de GPU criam os seus próprios objetos de imagem
(em subprocessos de trabalho para os percursos de GPU), pelo que a verificação antiga reportava
`0 file(s) written` para cada uma dessas execuções e, em seguida, emitia a indicação de zero exportações — em execuções
em que tudo tinha funcionado. Se criar um script com base neste número, uma execução paralela bem-sucedida agora
apresenta uma contagem diferente de zero.

Os saltos do sensor de luz relatam o motivo que o leitor efetivamente identificou para cada ficheiro — um
esquema ilegível, um pacote em falta, um erro de gravação — **deduplicado**, pelo que vinte ficheiros
saltados por uma única causa são interpretados como uma única causa, em vez de vinte repetições da mesma.

> **O `process()` não é acionado quando uma execução não produz imagens.** Este é um dos dois pontos em que o SDK e
> a CLI diferem deliberadamente: o `chloros-cli process` trata «foram solicitados produtos, nenhum foi
> gravado» como uma falha e termina com um valor diferente de zero, enquanto o SDK retorna normalmente e reporta a
> condição através do `summary` / dicas. Se o seu pipeline tiver de parar numa execução vazia, verifique-o
> você mesmo — inspecione o `summary` (ou conte os ficheiros na pasta do projeto) em vez de confiar na
> ausência de uma exceção. As causas habituais são uma pasta de entrada que não foi reconhecida como uma
> captura e produtos ignorados por serem inaplicáveis às câmaras presentes (por exemplo, radiação proveniente de câmaras apenas RGB
>).
>
> O `process()` também não é gerado quando as exportações de uma execução não puderam ser verificadas: emite um aviso e define
> o `result["export_check_failure"]`. Os produtos são gravados. Para interromper a execução neste caso, verifique essa chave. Uma execução que
> perdeu uma imagem funciona da mesma forma: emite um aviso e define o `result["image_failure"]`, o qual o `chloros-cli process`
> termina com um valor diferente de zero.

### Funções de conveniência

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Cada execução ocorre dentro do seu próprio `with ChlorosLocal(...)`; um backend que teve de iniciar continua a funcionar quando ela regressa, para a próxima chamada, e encerra-se automaticamente após 30 minutos sem que nada a esteja a utilizar. Nenhuma delas aceita `auto_start_backend`, `radiometric_output` ou `daq_log_path`; utilize `ChlorosLocal` e `configure()` para esses casos. O `process_lattice_capture()` não aceita `camera`, `debayer` nem `ppk`: ignora a deteção do painel e utiliza o debayer padrão.

### Valores suportados

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Saída radiométrica (pipeline multiespectral LATTICE)

O nível de exportação multiespectral LATTICE (M3C/M3M) do pipeline `process` — `reflectance` (predefinição), `radiance`, `sensor-response` ou `all` (cada modo aplicável por imagem) — define os produtos **«Exportar radianância»**e**«Exportar refletância»****: o `radiance` exporta apenas radianância, o `reflectance` apenas refletância, o `sensor-response` nenhum dos dois (apenas os produtos sem bayering e de pré-visualização), o `all` exporta ambos. Os produtos sem efeito de Bayer e de pré-visualização não podem ser alterados neste nível. O `configure()` tem uma palavra-chave dedicada para isso:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A saída de emergência avançada — definir os produtos através de `custom_settings` — continua a funcionar, mas lembre-se de que substitui todo o bloco de definições (ver o aviso abaixo). Escreva as chaves `"Export <product>"`: as definições próprias de um projeto, `"Export radiance"` / `"Export reflectance"` de um projeto prevalecem sobre a chave mais antiga `"Radiometric output"`, pelo que essa chave, por si só, não altera nada.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (a predefinição) divide a radiação da câmara pelo **fluxo descendente do DAQ com data e hora correspondentes**, resolvido automaticamente a partir de um `.daq` (DAQ-U/M/E) gravado**ou um `.csv` nativo do DAQ-M**encontrado juntamente com as imagens; qualquer pacote de calibração por câmara ou DAQ que falte localmente é**obtido automaticamente da AWS** na primeira utilização. A CLI expõe isto de acordo comopções de ativação/desativação de produtos do tipo no `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**De quem é essa leitura?**Uma leitura que uma captura `.daq` guarda é a leitura da(s) câmara(s) que descreve — identificada(s) num ficheiro sidecar `<file>.daq.cameras.json` ao seu lado (capturas únicas, sincronizadas e em matriz; uma sequência em formato raw indica a leitura de cada elemento no seu manifesto) — e é associada**apenas a imagens dessas câmaras**. Uma câmara autónoma sem leitura própria, portanto, já não utiliza o DAQ de outra câmara; associe-lhe um DAQ ou registe uma. A própria gravação de um DAQ (`chloros-cli daq pool-record`, ou uma gravação iniciada na GUI), ou qualquer `.daq` sem esse ficheiro sidecar (incluindo capturas mais antigas), continua a servir todas as câmaras.**Câmaras OCN sob luz solar.** Todos os pacotes de calibração OCN são criados sob luz de banda estreita, com um LED por banda, enquanto a luz solar mistura as bandas. O Chloros 1.3.0 aplica uma correção rápida com base na luz solar após a separação das bandas, o que melhora os produtos de radiação, refletância e índice OCN sob luz solar. Funciona em qualquer lugar onde a separação de bandas ocorra — `process()`, capturas, `grab()` / `frame_stream()` e a visualização em tempo real — para pacotes OCN que suportem esta funcionalidade; as câmaras RGN permanecem inalteradas. As imagens OCN captadas com uma versão anterior também beneficiam desta correção quando são processadas num computador que contenha a calibração com que foram captadas (o Chloros mantém a calibração de uma câmara assim que esta é ligada); caso contrário, são processadas como anteriormente. Os valores de radiação, refletância e índice do OCN diferem, portanto, dos da versão 1.2.0.

> `custom_settings` **substitui** todo o bloco de definições calculadas (ignora as outras palavras-chave e a validação do `configure()` por definição). Ao utilizá-lo, inclua todas as chaves `Project Settings` que lhe interessem, tal como no exemplo acima.

#### Leitura de píxeis de índice (NDVI e os outros índices de vegetação)

**Um raster de índice inteiro é um alongamento de contraste por imagem, não um mapa de −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — incluindo o ficheiro TIFF de 8 bits em que uma captura de 8 bits é gravada — armazenam

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

onde `IndexMin` / `IndexMax` são **os valores de índice finitos mais baixo e mais alto desse fotograma**. DN 0 não é −1, e a escala completa não é +1.

A Chloros marca ambos os extremos para que o mapeamento se mantenha invertível:

| Etiqueta | Significado |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — a convenção abaixo |
| `Chloros:IndexMin` | o valor de índice armazenado como DN 0 |
| `Chloros:IndexMax` | o valor do índice armazenado como escala total |

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

A leitura não necessita de `-config pix4d.config` — o exiftool lê estas propriedades XMP por si próprio. A configuração só é necessária para *gravá-las*.

> **`IndexMin == IndexMax`** significa que cada pixel daquele fotograma continha o mesmo valor de índice. A trama está preenchida a meio da escala e não há nada por que dividir — utilize o ponto final.

> **A ausência de etiquetas `Chloros:Index*` significa que não existe um mapeamento definido.** Uma exportação LUT é uma renderização colorida cujos píxeis são cores em gradiente associadas à sua janela de limiar LUT, e um fotograma sem píxeis de índice finito não tem intervalo a indicar. Não presuma a existência de um mapeamento em nenhum dos casos — aplica-se a mesma regra que para a ausência de `PixelScale`.**Como o alongamento é por imagem, dois fotogramas de um voo não são comparáveis em termos de DN**— cada um tem os seus próprios pontos finais, pelo que o mesmo NDVI corresponde a valores de DN diferentes em imagens diferentes. Para trabalhos quantitativos ao longo de um voo (estatísticas zonais, limiares fixos, séries temporais), exporte**`TIFF (32-bit, Percent)`**: as rasters de índice são gravadas nesse formato como float32**os próprios valores de índice**, sem necessidade de reescalonamento nem de etiquetas. Apesar do nome do formato, os produtos de índice não são ×100 — essa escala aplica-se apenas à refletância com tipo de dados inteiro.

#### Máscara de limitação da separação espectral — em que píxeis uma banda foi limitada

A separação espectral 3×3 pode tornar uma banda negativa — nos casos em que a luz mal contém essa banda, ou em que a cena fica fora do que a matriz de separação da câmara descreve. Como não se trata de um valor real, a exportação fixa-o em 0 — e a visualização em tempo real mostra o mesmo 0. Uma banda pode ser limitada a alguns píxeis ou a quase todos; a máscara regista qualquer uma das situações. O `Chloros:Band{R,G,B}UnmixClampFraction` indica em que parte de um fotograma isso ocorreu; a máscara de limitação indica **quais os píxeis**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— numa pasta `clamp_masks/`**ao lado** da pasta do produto, nunca dentro dela (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — para cada produto de radianância, refletância e índice em que pelo menos um pixel tenha sido limitado. `uint8` de banda única, na própria grelha de píxeis do produto (orientação, ROI, binning e a distorção de alinhamento da matriz incluídos). Cada máscara identifica a câmara do seu produto (`Make` / `Model` / serial), a hora de captura e as etiquetas da lente, e lista as bandas que os seus bits representam (`XMP-Camera:BandName`, bit 0 em primeiro lugar); os seus `ImageDescription` e `Software` identificam-na como uma máscara de limitação, e não contém qualquer calibração.
- **Bits**: bit 0 = a banda no canal R do produto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Um bit definido significa que o valor desse pixel nessa banda foi limitado a 0 — considere-o, bem como qualquer índice a partir dele, como de baixa confiança.
- **`Chloros:UnmixClampMask`** é gravado em todos os produtos que executaram a separação de cores: o caminho da máscara relativo à própria pasta do produto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre `/`separado — junte-o à pasta da imagem para abrir a máscara), ou `none` quando nenhum pixel foi limitado. A ausência de marcação significa que o produto nunca executou o processo de separação de cores (câmaras RGB ou monocromáticas, níveis raw/sem debayering).
- Os resultados da captura apresentam-no como **`clamp_mask`** do fotograma (o caminho da máscara, ou `None` quando nada foi limitado) — nos resultados `CameraSession.capture()`, os quadros `ArraySession.capture()`nos fotogramas `CaptureResult` e nas respostas de captura HTTP. A compilação offline lista as suas máscaras como `clamp_masks`.
- A pilha multibanda de um índice em tempo real (`/api/camera/cli-array-index` com `save_multiband`) obtém uma máscara por câmara de origem, na própria pasta de saída `clamp_masks/` — consulte a referência da CLI`lattice index`. Um vídeo não tem máscara: o resultado da compilação offline apresenta, em vez disso, a fração de limitação de cada fotograma.

Como nenhuma pasta de produto contém uma máscara, em qualquer nível de hierarquia, uma pasta de produto pode ser entregue ao software de fotogrametria tal como está — incluindo subpastas — e este aceita apenas imagens. Entregue-lhe a pasta de produto, não a pasta acima dela. O próprio processo de importação do Chloros ignora as máscaras.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect para câmaras LATTICE

Sessões de backend persistentes para hardware em tempo real. Os mesmos pontos de extremidade que a GUI utiliza, pelo que o comportamento é idêntico no SDK, na CLI e na GUI.

### Câmara única — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

A exposição, o ganho e o alvo definidos aqui são mantidos como os seus: o controlo automático de exposição do Chloros (incluindo a exposição medida pelo DAQ) não os altera. O `cam.set_settings(exposure_auto="Continuous")` (ou `gain_auto=`) devolve a exposição (ganho), e o `cam.set_settings(target_brightness_auto=True)` devolve o alvo ao sistema de alvo automático dChloros.

#### `connect_camera()` Assinatura

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Retorna com o fluxo iniciado.** O backend aguarda, com um limite (cerca de 15 s para o fluxo iniciar e, em seguida, 10 s para o seu primeiro buffer completo — não para o primeiro fotograma de pré-visualização, que surge cerca de 1,5 s depois), pelo que um `status()` surge logo a seguir ao que indica `streaming`. O `connect_stream` da sessão corresponde ao bloco `stream` da resposta: `state` `streaming`, `no_frames` (o fluxo iniciou e não transmitiu nada — o `note` indica isso e, para uma câmara a utilizar pacotes jumbo, para se reconectar com um tamanho de pacote de 1500 bytes), `starting` ou `start_failed`, com `waited_s`. `None` para uma câmara que já estava ligada. Uma pré-visualização (`get_camera_preview`, `/api/camera/jpeg/<serial>`) solicitada imediatamente após a espera, limitada a 4 s, pelo primeiro fotograma de pré-visualização da câmara, em vez de responder «Ainda não há imagem disponível»; uma câmara cujo fluxo ainda não tenha transmitido nada (`no_frames`, ou preparada para um disparador de hardware) continua a responder imediatamente.
- **O tamanho do pacote é definido por si.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` abre a câmara com pacotes de 1500 bytes, sem ser testado (`chloros-cli lattice cam-connect --packet-size standard` envia o mesmo).
- **Apenas uma ligação de cada vez mantém o conjunto de câmaras**: a abertura, a verificação do tamanho do pacote, um primeiro download de calibração (cerca de 70 s para uma câmara nova neste computador), o início da transmissão. Um pedido que necessite de uma câmara entretanto (`read_nodes`, `set_settings`, uma captura, uma segunda ligação) aguarda por ela no máximo 5 s, depois falha com o código HTTP 503 `CAMERA_POOL_BUSY`, indicando a câmara que se está a ligar (`ChlorosConnectError`); pergunte novamente daqui a pouco. As listagens nunca esperam: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` e `CameraSession.status()` respondem imediatamente. Uma câmara em processo de ligação é listada sob a resposta do conjunto `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — e `elapsed_s`), e o seu `status()` indica o `connecting: True` com o seu `phase`; as outras câmaras são lidas em tempo real quando o conjunto está livre; caso contrário, respondem com o valor da última leitura (`live: False`).

#### Métodos `CameraSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=15.0)` | O estado em tempo real da câmara, tal como o `chloros-cli lattice status` o lê: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` ou `stopped`), `device` (modelo, número de série, firmware, temperatura, `is_mono`) e as suas definições atuais. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lê os nós GenICam; devolve `{nodes, errors, enums, device, was_in_pool, in_array_id}`, além de `frame_values` e `notes` enquanto a exposição automática está em execução (ver abaixo). |
| `set_settings(**kwargs)` | Escreve nos nós pelo nome amigável (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Um `exposure_time` (`gain`) corresponde a uma exposição manual (ganho): quando escrito sem `exposure_auto` (`gain_auto`), o modo automático tem de ser desativado primeiro, para que o valor não seja rejeitado por um modo automático em funcionamento. O resultado, `applied`, mantém o que a câmara captou; uma gravação que a câmara rejeita encontra-se em `errors` e não é mantida. O erro indica o intervalo que a câmara aceita agora (um valor máximo de `ExposureTime` segue a taxa de fotogramas da câmara), e um modo automático que foi desativado apenas para o valor rejeitado é reativado; `restored` lista-o (por exemplo, `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Captura um **único** fotograma. Devolve um `CaptureResult`: uma lista de dicionários de metadados de fotogramas (um por nível guardado), com `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. (A captura em rajada/multi-fotograma foi removida — chame `capture()` num ciclo se precisar de uma série.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Capturas consecutivas (`interval_s=0`) ou com intervalo a partir desta câmara, para capturas `count` e/ou `duration_s` (pelo menos uma, para que termine), tal como o `ArraySession.capture_repeated()` faz para um array. Devolve a lista de `CaptureResult`s; pára após uma captura que indique que a unidade está cheia. O `on_capture(i, result)` é executado após cada uma; as outras palavras-chave são encaminhadas para o `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Libertação da reserva; a câmara é reposto no estado em que se encontrava no momento da ligação (ligação do disparador, PTP, definições); o `keep_settings=True` guarda as suas definições na sua própria memória; o `clear_settings=True` reinicia essa memória (ver [Memória da câmara](#camera-memory--keep_settings--clear_settings)); devolve `camera_memory` quando qualquer uma das opções for solicitada. Uma câmara que seja membro de um conjunto liberta todo o conjunto. Não realiza nenhuma operação se estivermos ligados a uma sessão já aberta. Retorna assim que o backend confirmar a libertação, ou quando não houver nada para libertar (a câmara já não existe ou não há nenhum backend em execução); uma libertação que tenha atingido o tempo limite ou que tenha sido recusada pelo backend gera o erro `ChlorosConnectError`, porque a câmara pode ainda estar retida e a próxima ligação entraria em conflito com ela. A saída do bloco `with` nunca gera um erro. |

Atributos: `serial`, `device` (o registo de descoberta da câmara) e `backend_url`.

Controlos de exportação `capture()` (mesmo modelo que a matriz + GUI):

- `processing` / `levels` — O `processing="all"` guarda todos os tipos de exportação aplicáveis; o `levels=["raw","radiance"]` guarda apenas esses (substitui o `processing`). Omita ambos para utilizar a predefinição do backend.
- `daq` — a leitura do DAQ utilizada na captura é guardada, por predefinição, como um ficheiro auxiliar `.daq`, incluindo uma captura apenas em formato bruto, para que o fotograma possa ser reprocessado posteriormente em refletância/índice. `daq=False` ignora-o quando nenhum nível o utiliza. `force_daq=True` é a grafia mais antiga do valor predefinido. Sem DAQ, nem `.daq`.
- `daq_tolerance_ms` — a que distância da exposição a leitura pode estar e ainda assim permanecer dentro da tolerância (padrão 500). Apenas o veredicto muda.

#### `read_nodes()` em exposição automática — `frame_values` e `notes`

Enquanto o `ExposureAuto` (`GainAuto`) está em execução, o nó `ExposureTime` (`Gain`) lê o **último valor gravado**, e não o que a câmara está a fazer. O `nodes` ainda mantém esse valor do nó. Quando se solicita o `ExposureTime` ou o `Gain` numa câmara ligada cujo modo automático não seja o `Off`, a resposta acrescenta:

- `frame_values` — o valor transportado pelo quadro mais recente da câmara, sob o nome do nó (`ExposureTime` em µs, `Gain` em dB);
- `notes` — uma frase por cada um desses nós, indicando ambos os valores.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Ambas as chaves estão ausentes quando o modo automático é `Off` (o nó é, nesse caso, ovalor real da câmara) e para uma câmara que não esteja ligada ao backend. No caso de um array, o `ArraySession.status()["member_ae"]` fornece os valores de fotograma de todos os elementos de uma só vez.

#### Limite máximo de exposição medido pelo DAQ (HTTP)

Com um sensor de luz DAQ ligado, o Chloros limita a exposição × ganho de uma câmara com base na luz do dia medida, para que um painel de referência branco não sofra clipping. Este limite aplica-se apenas enquanto um alvo de calibração estiver no campo de visão dessa câmara (detetado no seu enquadramento nos últimos 10 s); sem nenhum alvo no campo de visão, é ativada a exposição automática normal da câmara, com a sua própria proteção contra picos de luz, para que o resto da cena não fique mais escuro do que o necessário. Não existe um wrapper SDK; leia ou altere esta configuração por câmara com `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (o corpo da requisição POST com `{"enabled": false}` desativa-a para essa câmara). A resposta contém `enabled`, `cap_us` (o limite máximo em tempo real, `None` quando inativo, incluindo quando não há nenhum alvo de calibração no campo de visão), `k_exp_g` (a constante com que o limite máximo é calculado) e:

| Campo | Significado |
| --- | --- |
| `k_exp_g_source` | Qual a constante em uso: `camera` — a própria desta câmara, calculada a partir do ponto de laboratório no seu pacote de calibração (câmaras RGB); `fleet` — a constante da frota, utilizada quando o pacote não tem ponto de laboratório; `override` — uma substituição de campo definida no backend. |
| `k_exp_g_fleet` | A constante da frota, para comparação. |

### Matriz Sincronizada — `ArraySession` (Smart-Prep)

O `connect_array` é **o ponto de entrada recomendado** para configurações com várias câmaras. Executa, em segundo plano, todo o fluxo Smart-Prep da interface gráfica do utilizador (GUI):

1. **Análise de rede**(`/api/camera/array/recommend`) — define o**sensor completo** (sem binning, sem recorte de ROI) e permite que a taxa de fotogramas desça para o valor que este anfitrião e a rede fornecem, projetado com os formatos de píxeis e o empacotamento que o Connect irá utilizar. Os fotogramas nunca são reduzidos para manter a taxa elevada: utilize um `width` / `height` ou `binning=2` / `4` para trocar resolução por taxa de fotogramas (a lista do ponto final`alternatives` lista essas opções com os respetivos fps projetados).
2. **Seleção automática de nível** — `sim-capture-sim-emit` (todas as câmaras expõem com o mesmo disparador, com emissão ritmada para partilhar a ligação, sendo que a taxa de fotogramas diminui à medida que a partilha se reduz) em qualquer rede em que as câmaras possam ser sincronizadas para partilhar; `slip-emit-and-capture` (as câmaras revezam-se na ligação, não sendo expostas simultaneamente) apenas quando tal não for possível.
3. **PTP ativado**por predefinição — os carimbos de data/hora entre câmaras alinham-se num relógio partilhado com uma precisão de**~1 ms**, o [PTP grandmaster](#ptp-grandmaster--time_sync_status) do backend. A exposição simultânea resulta do disparador de hardware M8 (**&lt; 100 µs** entre módulos), e não do PTP: o PTP alinha *marcas temporais*, não exposições.
4. **Seleção automática do formato de pixel por câmara** — monocromático → `Mono12`, a cores → `BayerRG12`.
5. **Inicialização da AE** — captura instantânea do estado atual da AE de cada câmara para que a ligação não reinicie a exposição a meio do processo.
6. **Configuração do disparador GPIO** — `connect_array` arma todas as câmaras (`TriggerMode=On`, `TriggerSource=Line2`) para que o pulso da câmara principalimpulso do mestre acione as escravas através do cabo M8. Esta é uma etapa exclusiva para matrizes: uma câmara individual aberta com `LatticeCamera` funciona em modo livre. Ao libertar a matriz (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) repõe todas as câmaras no modo de funcionamento livre (`TriggerMode=Off`), mantendo todas as restantes configurações tal como foram detetadas pela ligação: as definições do disparador e da linha de sincronização (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; a linha de sincronização da câmara principal de volta para uma entrada), PTP e as definições da câmara alteradas durante a sessão — a menos que `keep_settings=True` / `clear_settings=True` (ver [Memória da câmara](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributos: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` e `packed_reason` (os últimos quatro são `None` num identificador a partir de `attach_array`), `connect_stream` e `backend_url`. O `connect_stream` é o bloco `stream` da resposta de ligação, a forma da ligação única mais o `members` (`{serial: {state, why?}}`): `streaming` assim que o primeiro grupo sincronizado da matriz chegar (o backend aguarda por ele, com um limite de cerca de 10 s), ou `no_frames` com a última falha de recuperação de cada membro como o seu `why` e, para pacotes jumbo, o valor de recurso de 1500 bytes no `note`. `None` num identificador `attach_array`.

#### Assinatura `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` representa o tempo que a chamada aguarda pela ligação e é enviado para o
backend como o seu prazo limite. Uma ligação não concluída até esse momento é anulada: todas as câmaras que
tenha aberto são repostas no estado em que foram encontradas e fechadas, nenhuma matriz é ligada e a
chamada gera o erro `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indicando a
etapa em que a ligação se encontrava). Assim, uma chamada que gera um erro nunca deixa uma matriz
ligada a seguir a si. Uma matriz de 9 câmaras liga-se em cerca de um minuto; logo após
as câmaras serem ligadas, pode demorar vários minutos. Uma matriz que está ativa, mas que ainda não tem
um grupo de fotogramas sincronizado, é devolvida com o código de erro `connect_stream["state"]`
`no_frames`. O registo do backend contém uma linha de tempo por etapa de ligação
(`[array-connect <id>] <step>: <s> s …`) e uma última linha que indica a mais lenta.

Valores de `force_tier`:
- `"sim-capture-sim-emit"` — verdadeiramente simultâneo (todas as câmaras disparam na mesma flanco do relógio).
- `"sim-capture-ftd-stagger"` — escalonamento flexível no domínio do tempo (as câmaras emitem em momentos ligeiramente desfasados, para que os pacotes sejam serializados na linha).
- `"slip-emit-and-capture"` — captura sequencial por câmara (sem sincronização temporal). Selecionado automaticamente apenas quando as câmaras não podem ser reguladas a um ritmo suficientemente lento para partilharem a ligação em simultâneo.

`wire_ceiling_mbps` substitui o **orçamento sustentado de banda do anfitrião** em MB/s — o único
número do qual depende toda a alocação da matriz. Deixe-o em `None` para utilizar o valor detetado automaticamente
. Reduza-o quando o conjunto de câmaras reportar quadros corrompidos pelo GVSP: o valor automático é derivado
da taxa de ligação anunciada pela NIC, que sobrestima adaptadores USB, pistas PCIe estreitas e
estruturas partilhadas ocupadas — e essa sobreestimativa manifesta-se como quadros corrompidos, em vez de como uma
ligação visivelmente lenta. O valor é guardado no bloco de captura da matriz do projeto, pelo que uma
reabertura ou um `connect_array` posterior o restaura como qualquer outra configuração da matriz.
Consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames).

#### Transporte compactado de 12 bits (`packed=`)

Com todos os sensores ativos, a taxa de fotogramas de uma matriz é determinada pela velocidade a que cada câmara consegue enviar
os seus fotogramas. Os formatos de 12 bits (`BayerRG12`, `Mono12`) são normalmente transmitidos como palavras de 16 bits;
o transporte compactado envia os mesmos 12 bits em 1,5 bytes por pixel e
descompacta-os no anfitrião, pelo que cada fotograma demora 25% menos tempo a ser enviado — cerca de
**33% mais fotogramas por segundo com o sensor a funcionar a plena capacidade** (medido: 10,7 → 14,0 fps num
conjunto de 9 câmaras). **Os dados guardados são idênticos**: mesmos valores, mesmos ficheiros de 16 bits
, mesmos metadados.

- O `packed="auto"` (predefinição) comprime apenas quando isso é vantajoso **e** o anfitrião tem
  capacidade para a descompressão: na ligação, o backend mede a sua própria velocidade de descompressão,
  o número de núcleos da CPU, a carga atual e a largura de banda disponível. Um computador de secretária com núcleos disponíveis
  compacta a partir de todos os sensores; um anfitrião ocupado ou de baixa potência (computadores de placa única,
  controladores incorporados) transmite sem compactação. Pequenas áreas de interesse (ROIs) e fotogramas agrupados beneficiam pouco.
- `packed="off"` (ou `False`) nunca compacta — para excluir essa possibilidade durante o diagnóstico,
  ou num anfitrião que precise de reservar a sua CPU para outras tarefas.
- `packed="on"` (ou `True`) compacta sempre formatos de 12 bits.
- Qualquer outro valor gera o código de erro `ValueError`. Os formatos de 8 e 16 bits nunca são
  compactados.

O `analyze_array_network(..., packed="auto")` funciona da mesma forma,
pelo que o seu fps corresponde à taxa a que a matriz irá funcionar (a resposta contém
`packed_transport` e `packed_reason`); o seu `pixel_format` assume por predefinição o que
o «connect» aplica (Mono12 / BayerRG12) em vez de 8 bits.

A decisão e a respetiva justificação encontram-se na sessão:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` aplica a mesma substituição a todas as
matrizes de um projeto (padrão `None`, o que significa `"auto"`: um projeto não guarda
nenhuma configuração compactada por matriz).
Para um valor por defeito a nível da máquina, defina `CHLOROS_ARRAY_PACKED12=off` (ou `on`) no
ambiente do backend; uma definição explícita de `packed="on"` / `"off"` continua a prevalecer.
`/api/camera/array/status` reporta `packed_transport` e `packed_decision`.

#### Taxas de fotogramas esperadas

Taxas de fotogramas da matriz medidas (sincronizadas por hardware, 12 bits; sensor completo = 2048 × 1536). Utilize-as para verificar a viabilidade do `status()["actual_fps"]`; o `analyze_array_network()` projeta a taxa para o seu próprio anfitrião e rede.

| Configuração | Taxa de fotogramas da matriz |
| --- | --- |
| Sensor completo, placa de rede PCIe 10 GbE no anfitrião, transporte compactado (`packed="auto"` num computador de secretária) | ~14 fps |
| Sensor completo, placa de rede PCIe de 10 GbE no anfitrião, sem compactação (`packed="off"`) | ~10,5 fps |
| Sensor completo, 8 câmaras num adaptador USB de 10 GbE | ~3,8 fps |
| Sensor completo, ligação de anfitrião de 1 GbE | 1,2–1,7 fps |
| Binning 2 × 2 (`binning=2`), placa de rede de anfitrião PCIe 10 GbE | ~30–33 fps |

- Numa ligação PCIe 10 GbE, as **câmaras** são o limite com o sensor completo, não a rede: 4 e 8 câmaras funcionam à mesma velocidade.
- **`target_fps` é a taxa de disparo, e a matriz nunca funciona mais depressa.** O `target_fps=1` faz com que o conjunto funcione a 1 fps; o `None` (o valor predefinido) faz com que funcione à taxa mais elevada que o anfitrião e a rede suportam. Qualquer valor acima dessa taxa é reduzido para esse valor (ver [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` de `4294` em `status()` não é uma taxa de ligação.** Isto significa que o Windows não conseguiu ler a velocidade do adaptador (o controlador reportou o maior valor de 32 bits), o que é típico dos adaptadores Ethernet USB. A taxa utilizável de um adaptador USB no Windows está limitada a 200 MB/s, independentemente das condições (ver [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames)); para ajustar um adaptador, consulte a secção «Adaptadores USB 10GbE» da referência da CLI em [Configuração e Ajuste da NIC do Anfitrião](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Também se aplica um limite USB no Linux: 400 MB/s em x86_64, 200 MB/s no NVIDIA Jetson.** O adaptador é identificado a partir do sysfs (recorrendo ao `ethtool -i`), pelo que um adaptador USB de 5 GbE ou 10 GbE é limitado a este limite máximo, em vez da velocidade nominal de 500 / ~1060 MB/s. O valor para x86_64 foi medido no Linux: o mesmo adaptador que corrompe os fotogramas acima dos 200 MB/s no Windows funcionou sem problemas com um conjunto de 4 câmaras em todos os limites até aos 480 MB/s. No NVIDIA Jetson, o mesmo adaptador funcionou sem erros apenas a 200 MB/s e perdeu quadros de forma intermitente acima desse valor. Se o seu anfitrião absorver mais sem que os quadros do `gvsp_corrupt` sejam afetados, aumente-o com o `wire_ceiling_mbps`. No Linux, um adaptador ligado a uma porta USB 2.0 tem uma largura de banda limitada a 30 MB/s; ligue-o a uma porta USB 3.

#### Sobresubscrição (o limite mínimo por câmara)

Um conjunto de câmaras funciona em qualquer rede; a taxa de fotogramas é o que determina o desempenho. O ritmo de emissão simulada atribui a cada câmara uma quota igual do orçamento de banda à prova de colisões e todas elas continuam a expor no mesmo disparador; assim, numa ligação lenta, a quota — e a taxa de fotogramas — é pequena: 8 câmarasem 12 bits em 1 GbE sem quadros jumbo obtêm 6,5 MB/s cada, cerca de **1 fps**(cerca de 1,3 fps compactados). A quota tem um limite mínimo correspondente à taxa mais lenta a que as câmaras podem ser sincronizadas (`per_cam_floor_bps`: lida a partir do intervalo GevSCPD de cada câmara no momento da ligação, e nunca abaixo de 1 MB/s). Se a ligação exigir uma velocidade inferior a essa, a ligação recorre à transmissão**serial**(`slip-emit-and-capture`): as câmaras revezam-se à velocidade máxima da ligação, pelo que todos os quadros chegam, mas os membros**não são expostos simultaneamente**— o nível de resposta e os bloqueadores (`SERIAL: ...`) indicam isso; passe `force_tier="sim-capture-sim-emit"` para recusar o recurso de fallback. O tamanho do fotograma não influencia a partilha, pelo que**o binning e a ROI aumentam a taxa de fotogramas, mas não alteram se um conjunto cabe**. Apenas uma ligação que não consiga transportar a matriz a qualquer ritmo, seja simultaneamente ou à vez, fica**sobrecarregada** — o modo de falha é a perda de pacotes GVSP, não uma taxa de fotogramas mais baixa. Soluções: menos câmaras, fotogramas jumbo de ponta a ponta ou uma placa de rede mais rápida.

- As respostas `analyze_array_network()` e `/api/camera/array/connect` transportam `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` é verdadeiro, a projeção **zera os campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) em vez de indicar uma taxa enganadora de «lenta, mas funcional».
- `POST /api/camera/array/connect` aceita um parâmetro de corpo `pin_resolution` (**apenas HTTP — não é um kwarg do SDK**; o `connect_array` não o expõe). Trata-se do modo estrito: uma ligação com excesso de subscrições em que o `pin_resolution` está definido é**rejeitada categoricamente** com um erro que indica todas as soluções possíveis. Sem ele, a ligação prossegue com um aviso de que a matriz irá perder pacotes. Seja como for, a ligação mantém o tamanho de quadro solicitado — nunca agrupa nem recorta por conta própria.
- Apenas para testes: defina `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` no ambiente do backend para rebaixar a recusa para um aviso sonoro — estabelece a ligação na mesma e aceita a perda de pacotes.

#### Integridade da matriz — qual o subsistema que está a perder quadros

O `ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) transporta um bloco `health` ativo num
array ligado, reavaliado numa janela contínua de **10 segundos** . Divide a perda de quadros
nas duas causas que requerem correções opostas, em vez de uma taxa «incompleta» que
não especifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por número de série) | O quadro **chegou e estava estruturalmente incorreto**— perda de pacotes GVSP. |**Rede**: largura de banda disponível, ritmo de transmissão, anel de receção da NIC, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nunca chegou**— a câmara não disparou, ou nada saiu dela. |**Disparador / sincronização**: cabo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Pior taxa de cada câmara cada uma. | — |
| `per_cam_rate_pct` | Taxa combinada de imagens incompletas por câmara (ambas as causas em conjunto). | — |
| `stable_for_seconds` | Durante quanto tempo cada câmara permaneceu abaixo de 0,01 %. | — |

Juntamente com o `health`, o mesmo registo indica o número em que toda a alocação fica pendente:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de largura de banda sustentado do anfitrião em vigor, MB/s. |
| `wire_ceiling_source` | De onde veio esse número, por palavras — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `wire_ceiling_mbps=` o definiu. |
| `nic_is_usb` | `true` para um adaptador USB Ethernet. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretação:** um valor diferente de zero para `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` igual a 0 significa
que o disparo e a sincronização do cabo estão perfeitos e que 100 % da perda ocorre no caminho da rede — reduza
`wire_ceiling_mbps` e volte a ligar. O padrão inverso aponta, em vez disso, para o cabo de sincronização ou para a
linha de disparo.

> **O `target_fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido uma vez na
> ligação, pelo que diminuir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na procura não produziu qualquer melhoria, enquanto
> a redução do limite máximo da ligação de 240 para 200 MB/s fez com que o mesmo equipamento passasse de 10,4 % de quadros corrompidos para
> 0,00 %.

> **Uma matriz em funcionamento não pode replanejar-se automaticamente no firmware LATTICE (M3C / M3M).** Não consegue
> resolver isto por si próprio; desligue e volte a ligar para que o seletor de tempo de ligação se replaneje de acordo com
> o novo limite máximo.

Um **adaptador Ethernet USB está limitado a 200 MB/s** pela sonda, independentemente da sua
placa de identificação: a tabela de eficiência que converte uma taxa de ligação num valor sustentado é
derivada da PCIe, e uma placa de rede USB anuncia a sua taxa de ligação Ethernet, embora esteja limitada pelo
barra USB e pelo seu controlador. O limite é absoluto, não uma fração — um adaptador USB de 1 GbE
atinge cerca de 80 MB/s e não é afetado. Aplica-se ao Windows e ao Linux (incluindo o Jetson);
consulte [Taxas de fotogramas esperadas](#expected-frame-rates) para saber onde foi medido.

#### Métodos `ArraySession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Estado em tempo real da matriz em execução: `target_fps` / `target_fps_auto` (a taxa de disparo e se o controlador de taxa a controla), `target_fps_clamped_reason` (por que razão uma taxa fixa é mantida mais baixa; caso contrário, `None`), `actual_fps` (medido; 0 assim que o ciclo de captura desistir ou ficar bloqueado), `recommended_fps`, `streaming` (False assim que o ciclo de captura desistir — verifique isto, não apenas o `last_error`; permanece True após um reinício das definições, o que o `restarting` assinala), `stalled` (True enquanto o ciclo de captura estiver em execução, mas não tiver publicado nenhum grupo de fotogramas durante cerca de dez períodos de fotogramas, 2–30 s; o `last_error` só o indica após 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` apenas quando a matriz estava ligada ao PTP **e** todos os membros tiverem lido pela última vez o `PtpEnable`; não é o mesmo que «bloqueado», que o `ptp_epoch` mede), `ptp_configured` (o que a ligação solicitou), `ptp_off_members` / `ptp_unread_members` (membros que leram pela última vez `PtpEnable` desativado / não responderam), `member_recovery` (`None`, ou um membro que perdeu alimentação ou a sua ligação: `state`, `waiting` enquanto não responde, `reopening`, depois o `recovered`, juntamente com o `members` e o `why` — a matriz reabre-o quando este volta a responder e devolve-lhe o seu PTP de tempo de ligação, a ligação do disparador e da linha de sincronização, o ritmo e as definições da câmara, e depois transmite novamente sem novos `connect_array`) e `ptp_epoch` (medidos), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` significa que o Windows não conseguiu ler a velocidade do adaptador — consulte [Taxas de fotogramas esperadas](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (se a cadeia de radiação por banda é utilizável e qual dos elementos a interrompeu) e `band_clamp` — enquanto uma visualização em tempo real renderiza a matriz: a percentagem de píxeis iluminados que a separação de cores limitou a 0, por elemento (`members`: `{serial: [band0, band1, band2]}` na ordem das bandas R, G, B da câmara) e, por banda, o que a imagem composta mostra (`composite`: `{band name: share}`), com `warn`, a percentagem que a GUI assinala como «Dados da banda de baixa confiança» (medidos aproximadamente uma vez por segundo; `None` quando não há visualização em direto a decorrer), e `member_ae` — por membro, a exposição (`exposure_us`) e o ganho (`gain_db`) que o seu último fotograma publicado continha, a partir do próprio fotograma (`read_nodes` de `ExposureTime` / `Gain` devolve o último valor gravado enquanto a exposição automática está em execução, não o que está a fazer). |
| `set_member_settings(serial, **kwargs)` | Grava as definições de uma câmara membro, tal como `CameraSession.set_settings()` as obtém (exposição, ganho, brilho alvo, ...); o membro permanece na matriz. Lança uma exceção `ChlorosConnectError` para uma série que não seja um membro. Ver [Exposição nos membros da matriz](#exposure-on-array-members). |
| `capability(timeout=10.0)` | O que o smart-prep definiu (o nível de captura, o tamanho do fotograma, `pixel_format`), o bloco `health` em tempo real e o orçamento de largura de banda do anfitrião. Consulte [Estado do Array](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Alterar a taxa de disparo enquanto o array está em funcionamento, sem necessidade de reconexão. Um número fixa-a (o controlador de taxa não a altera); `None` repõe-na para automático. Devolve `{requested, target_fps, target_fps_auto, clamped}` — `clamped` é True quando a taxa foi reduzida para o que o anfitrião e a rede conseguem suportar. Consulte [Captura por intervalos para levantamentos com drones](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Um fotograma de cada câmara, **todos a partir de um único disparador**, como matrizes numpy (sem gravação de ficheiros). Devolve um `GrabResult` (um `dict`): `{serial: array}`, ou `{serial: (array, metadata)}` com `with_metadata=True`. O `.group_id` é o disparador que os fotogramas partilham; o `.errors` mapeia cada câmara excluída para a razão (uma câmara que não pode ser atendida nesse disparador é omitida, nunca preenchida a partir de outra). O `processing` é o `raw` / `debayered` / `radiance` / `reflectance` / `display`. A mesma leitura que a [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) da API do projeto — uma implementação partilhada — pelo que todas as palavras-chave aí presentes (`band`, `align`, `bit_depth`, `refl_max`) funciona aqui. Requer `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterador que produz um resultado `grab()` por tick (apenas argumentos de palavra-chave), com ritmo definido por `fps`, limitado por `duration` (segundos) e/ou `count` (`None` = ilimitado). **Cada conjunto produzido constitui um novo gatilho:** um tick cujo gatilho mais recente seja o conjunto que já produziu aguarda o próximo gatilho em vez de o repetir, pelo que o fluxo decorre à velocidade mais lenta entre `fps` e a própria taxa de fotogramas da matriz. Lança `FrameFetchError` em vez de ficar em silêncio: imediatamente quando todas as câmaras recusarem, por qualquer motivo, que o tick seguinte se repita; após 60 s de ticks sem fotogramas; e após 60 s sem nenhum novo gatilho («nenhum novo gatilho durante 60 s»). A mesma implementação que `ArrayHandle.frame_stream()`. Consulte [Fotogramas em direto no seu próprio pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Iniciar a captura contínua própria do backend (o modo Contínuo da GUI) → [`ContinuousHandle`](#continuoushandle). Guarda **todos os** grupos sincronizados que a matriz publica, tal como o `capture()` os guardaria (com os mesmos controlos de exportação), até ser interrompido ou até que o `max_captures` / `duration_s` (`None` = até ser interrompido). Lança a exceção `ChlorosConnectError` se já estiver a ser executado um para esta matriz. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Um grupo de captura sincronizado. Devolve um `CaptureResult` (lista de dicionários de quadros + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Passe um resultado anterior`group_seq` como `after_seq` para obter o próximo grupo sincronizado a seguir a este, em vez de esperar por um novo. Controlos de exportação abaixo. |
| `capture(..., smart=True)` | **Captura inteligente** — aguarda que o AE se estabilize em todas as câmaras e, em seguida, aciona a captura. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Captura mais rápida: apenas dados brutos + a leitura DAQ atribuída (+ o índice combinado livre). Espelha o botão «Captura mais rápida» da GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Contínua / Intervalo num único ciclo delimitado. Retorna `list[CaptureResult]`.**Requer `count` e/ou `duration_s`** para que termine (o SDK não tem Ctrl+C). Também termina antecipadamente, após a passagem, quando uma passagem reporta uma unidade cheia (uma entrada `.failed` com `disk_full`); essa passagem é o último resultado. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Iniciar a gravação da visualização combinada do índice em tempo real para vídeo/GIF → `RecorderHandle`. O vídeo é um Motion-JPEG `.avi` (não é necessário o ffmpeg). Sem nenhuma vista GUI aberta, o backend renderiza-o: `expr` (uma expressão de índice nos nomes das bandas da matriz, por exemplo, `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), ou o NDVI da matriz por predefinição. Um gravador composto por cada matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Iniciar uma sequência de imagens Bayer brutas a alta taxa de fotogramas → `RecorderHandle`. Reprocessar offline com `build_video()`. `orient=True`: os produtos criados pelo `build_video()` são orientados de acordo com a orientação da câmara (os produtos de cada câmara seguem a orientação da respetiva câmara, enquanto o índice combinado segue a orientação da câmara principal); os fotogramas em formato raw mantêm-se no formato nativo e o manifesto da sequência regista a orientação de cada membro. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocessa offline uma sequência RAW guardada, convertendo-a em vídeo(s) calibrado(s). Bloqueia até estar concluído (`wait=True`) e devolve o resultado da compilação: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — consulte [o resultado da compilação](#the-build_video-result) abaixo. |
| `build_video_status(job_id, timeout=15.0)` | Verifica o estado de um trabalho de compilação offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Liberta toda a matriz — **se esta sessão a tiver aberto** (`connect_array`): passa então a ser `force_disconnect()` e lança uma exceção `ChlorosConnectError` quando a libertação não é confirmada (a saída do bloco `with` nunca gera um erro). Num identificador proveniente de `attach_array`, deixa a matriz em funcionamento e imprime uma linha no stderr a indicar isso: a matriz pode pertencer à GUI ou a outro script. Cada câmara é deixada a funcionar livremente e, de resto, tal como a ligação a encontrou; o `keep_settings=True` guarda as suas definições na própria memória da câmara, enquanto o `clear_settings=True` repõe essa memória para as definições de fábrica (ver [Memória da câmara](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Liberta todo o conjunto, independentemente de quem o tenha aberto. Utilize-o num identificador associado quando pretender encerrar o conjunto (por exemplo, para o voltar a ligar com definições diferentes). Retorna assim que o backend confirmar a libertação, ou quando não houver nada para libertar (a matriz já não existe ou nenhum backend está em execução); uma desativação que tenha atingido o tempo limite ou que tenha sido recusada pelo backend gera o erro `ChlorosConnectError`. Com qualquer um dos sinalizadores, retorna o que a memória de cada câmara recebeu (`camera_memory`), caso contrário, `None`. |

`capture()` controlos de exportação (o mesmo ponto final que a GUI/CLI utilizam):

- `processing` / `levels` — `processing="all"` (ou `levels=["raw","radiance",…]`) guarda todos os tipos de exportação aplicáveis por câmara; um único valor `processing` guarda apenas esse nível.
- `aligned` — **ativado por predefinição (`None`), uma vez que a captura da mesma matriz pela aplicação Chloros**(a sua definição de captura**Alinhada** por matriz). A exportação não bruta de cada membro é ajustada ao enquadramento da câmara principal através do [perfil de alinhamento](#array-alignment) e recortada para a sobreposição comum dos membros, de modo que os ficheiros ficam alinhados ao nível dos píxeis e apresentam o tamanho, o recorte e as etiquetas `Alignment*` que o aplicativo utiliza (numa ROI, a transformação é reajustada com base na janela); o `XMP-mapir:AlignmentJSON` indica o `"applied": true` com o seu `crop_roi`. O ficheiro Raw permanece sem distorção no tamanho nativo, mas mantém a transformação nos metadados. O `aligned=False` guarda todos os ficheiros no tamanho nativo, sem distorção; o `aligned=True` é a configuração predefinida explicitada. `continuous()`, o `capture(smart=True)`, o `capture_repeated()` e os parâmetros ao nível do projeto `ArrayHandle.capture()` / `capture_smart()` assumem o mesmo valor predefinido. Independentemente de a distorção ser aplicada ou não, cada ficheiro membro regista a transformação `Alignment*` da sua câmara assim que a matriz tiver um perfil de alinhamento — exceto no caso de uma câmara cujo perfil não esteja alinhado (omitida ou forçada à identidade devido a uma resolução falhada) e a câmara de referência quando nenhuma outra câmara está alinhada: estas não contêm etiquetas `Alignment*`. Sem perfil na matriz, a captura é guardada desalinhada, e o resultado indica isso: `result.alignment` é igual a `{"requested": True, "applied": False, "warnings": [...]}`, um aviso por cada motivo pelo qual a distorção não pôde ser aplicada (ausência de perfil, membros com resoluções diferentes, uma matriz não, um fotograma no qual o perfil não pode ser mapeado). `applied` é `True` apenas quando todas as exportações que solicitaram a correção de distorção a obtiveram. O alinhamento automático da matriz cria o perfil assim que a exposição se estabiliza; o `ArrayHandle.calibrate_alignment()` / o `align-calibrate` da CLI criam um a pedido. O `result.oriented` indica se o `orient=True` processou os produtos.
- `render_index=False` — ignora a sobreposição do índice de vegetação por câmara; por predefinição, esta é renderizada onde estiver configurada.
- `daq` / `daq_tolerance_ms` — a leitura do DAQ é guardada como um ficheiro «sidecar» `.daq` por predefinição, mesmo quando nenhum nível selecionado o necessita; o `daq=False` ignora-o nesse caso, e o `daq_tolerance_ms` define a tolerância de correspondência (padrão 500). O `force_daq=True` é a grafia antiga do valor padrão. Consulte [A leitura do DAQ que uma captura guarda](#a-leitura-de-uma-captura-pelo-DAQ-guarda--ficheiros-DAQ-e-correspondências-DAQ).
- `orient=True` — transforma os **produtos**de cada membro (`display`, `radiance`, `reflectance`, o índice por câmara e o composto) pela orientação da câmara, após a distorção de alinhamento: pela orientação da**câmara principal**quando foram distorcidos para o seu enquadramento (a predefinição `aligned` — os membros permanecem co-registados), caso contrário, pela orientação da sua própria câmara.**Predefinição: cada ficheiro de membro nativo**, tal como acontece sempre com as capturas em matriz; os `raw` e `debayered` permanecem nativos em qualquer dos casos. Cada ficheiro regista a orientação do quadro em que os seus píxeis se encontram (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — um ficheiro RAW, da sua própria câmara), e o XMP `Alignment*` mantém a transformação do quadro nativo, aplicada antes da orientação; um ficheiro RAW que contém essa transformação também regista `ChlorosAligned*` (a do quadro de referência), a orientação de um produto ajustado ao quadro de referência. O processamento roda cada produto de acordo com a orientação que pertence aos seus píxeis: a orientação alinhada quando ajusta o ficheiro RAW (alinhamento de matriz ativado), a da própria câmara quando não o faz. `capture_repeated()` e `capture_fastest()` deixam-na passar; os parâmetros ao nível do projeto `ArrayHandle.capture()` / `capture_smart()` também a aceitam.

**Compressão TIFF (opção exclusiva para HTTP):** O `ArraySession.capture()` não envia a chave `compression`, pelo que se aplica a predefinição do backend — o `POST /api/camera/array/capture` lê um parâmetro do corpo da resposta `compression`, `"none"` por predefinição: não comprimido, os píxeis do sensor exatamente como são lidos (~6,3 MB por fotograma em resolução total). O `"deflate"` (zlib L1 sem perdas + preditor horizontal, ~4,1 MB/fotograma) grava ficheiros mais pequenos de forma mais lenta — ambos são sem perdas e são lidos de forma idêntica na importação. O SDK não disponibiliza nenhum kwarg para isso; a alternativa é o `chloros-cli lattice array-capture --compression deflate` ou HTTP bruto. O DEFLATE retém o GIL do Python, pelo que as gravações comprimidas não se paralelizam entre os threads de gravação de cada câmara — a captura sustentada em resolução total à taxa do sensor requer a predefinição não comprimida. Detalhes: [Referência da CLI → array-capture](cli-reference.md).**Grupos consecutivos (protocolo HTTP):** o ponto final devolve `group_seq`; um ciclo que o reenvia como `after_seq` na sua próxima solicitação obtém o próximo grupo sincronizado — incluindo aquele publicado enquanto a captura anterior estava a ser guardada — em vez de esperar por um grupo publicado após a solicitação, e o `skipped_groups` reporta qualquer grupo que o ciclo tenha deixado de capturar. O `capture_repeated`, o `--continuous` da CLI e o modo Contínuo da GUI fazem todos isto; as passagens de intervalo mantêm-se atualizadas em cada momento agendado.**Substituições de exportação por membro (apenas HTTP):**o mesmo ponto final também aceita o `exclude_serials` (lista — excluir membros do conjunto guardado; a matriz continua a ser acionada como um único grupo sincronizado e os membros excluídos são devolvidos no `excluded`), o `serial_levels` (substituições ao nível de cada câmara `{serial: [level tokens]}`) e `serial_index` (substituições de sobreposição de índice por câmara `{serial: bool}`). Estes são parâmetros de corpo com paridade da GUI e**ainda não são argumentos-chave do SDK**; os membros ausentes dos mapas recorrem aos valores globais da matriz `levels` / `render_index`.

##### Inspecção de câmaras ignoradas — `CaptureResult.skipped`

O `ArraySession.capture()` devolve um `CaptureResult`, que é uma subclasse do `list`: itere-o, indexe-o, `len()` — todos os padrões existentes continuam a funcionar. O novo código pode inspecionar o atributo `.skipped` para verificar quais as câmaras que foram excluídas e porquê. O caso mais comum é o das câmaras RGB numa matriz de filtros mistos quando se solicita o `processing="radiance"` ou o `"reflectance"` — a radiação por pixel Bayer não faz sentido para um sensor de banda larga, pelo que o backend ignora essas câmaras em vez de produzir resultados sem sentido.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Os tokens de motivo seguem o padrão `<level>-not-applicable-to-rgb-cam` (uma entrada por nível ignorado, cada uma contendo `level`). Uma câmara cujo pacote de calibração não esteja carregado ignora `radiance` / `reflectance` com `<level>-needs-calibration (connect the camera so its cal pack loads)`, e, na ausência de uma leitura de refletância utilizável do DAQ, salta para `reflectance-skipped-no-dls`. Os outros saltos específicos de refletância são `reflectance-skipped-no-fresh-dls` (não há leitura recente de radiação descendente disponível), `reflectance-skipped-bound-daq-unavailable (…)` (não foi possível aceder ao DAQ associado) e `dls-uncalibrated-band-<nm>` — a banda situa-se maioritariamente fora da gama radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm), pelo que a divisão absoluta da refletância baseada no DAQ é recusada e o quadro reverte-se claramente para a resposta do sensor. Entre as referências de produto disponíveis, apenas a F988 o desencadeia; o percurso suportado por essa câmara é o fluxo de trabalho do painel de refletância.

##### Ficheiros que não puderam ser gravados — `CaptureResult.failed`

Uma captura que só conseguiu gravar alguns dos seus ficheiros continua a devolver: a lista contém todos os ficheiros que **foram** gravados, e o `.failed` lista os restantes, um dicionário por ficheiro com o `serial` (`None` para o composto combinado e o ficheiro auxiliar `.daq` de uma matriz), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, ou `match` — a proveniência do `.daq` do `.match.json`, quando o `.daq` foi gravado, mas o seu sidecar não) e `error`. Numa unidade cheia, a entrada também inclui o `disk_full: True`; qualquer captura posterior nessa unidade falharia da mesma forma, razão pela qual o `capture_repeated()` termina aí. Não é lançada qualquer exceção para uma captura parcial, e um ficheiro em `.failed` não ficou no disco parcialmente gravado. `.failed` fica vazio numa captura limpa.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### O DAQ que lê uma captura guarda — `.daq_files` e `.daq_matches`

Sempre que um DAQ está associado à câmara ou ao conjunto de câmaras, ou em pool, cada captura guarda a leitura descendente que utilizou: um ficheiro `.daq` na subpasta `daq/` da pasta de captura, com o seu `.match.json` e o seu `.cameras.json` (de quem é a leitura). Isto aplica-se também a uma captura em bruto ou com debayering, e é o mesmo ficheiro quer a captura tenha sido efetuada a partir da GUI, da CLI ou do SDK. O `daq=False` ignora-o no caso de uma captura cujos níveis não utilizem o DAQ (um nível `radiance`, `reflectance` ou `display` continua a gravá-lo); o `daq_tolerance_ms` desloca a linha de tolerância para essa captura e nunca a leitura utilizada. Qual DAQ: aquele associado à câmara ou ao conjunto de sensores (a seleção do sensor de luz na GUI, uma associação de um projeto ou o `chloros-cli daq pool-bind`), ou então o mais recente que o backend tiver. A CLI especifica os controlos `--no-daq` e `--daq-tolerance-ms` ([Referência da CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Uma captura guarda a leitura do sensor de luz que utilizou como um `.daq`, acompanhada de um `<file>.daq.match.json` que indica como a leitura foi associada à exposição: `method`, `gap_ms` (da exposição à leitura DAQ mais próxima) e `in_tolerance` (dentro de 500 ms, ou o `daq_tolerance_ms` da captura). O ficheiro XMP da refletância apresenta o mesmo veredicto que `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` e `Chloros:DlsInTolerance`.

Uma captura aguarda a primeira leitura do DAQ após a exposição, para poder combinar as leituras de ambos os lados (`method` `interpolated`). Um DAQ-E envia uma leitura a cada 0,4 s à luz do dia e a cada 1-2 s em ambientes interiores, pelo que o `capture()` é devolvido cerca de um intervalo e meio depois; a exposição em si não sofre atrasos. Não se aguarda um DAQ que tenha deixado de enviar dados. Uma leitura do DAQ-E corresponde à média da luminosidade ao longo do intervalo desde a sua leitura anterior, pelo que é colocada no meio desse intervalo: o `gap_ms` é medido nesse ponto médio, e o `.match.json`&#x27;`bracket_stamps_ns` fornece os próprios carimbos de data/hora das leituras. O `ArraySession.continuous()` aguarda sem abrandar a captura. As passagens de volta do `capture_repeated()`não esperam, pelo que mantêm a sua taxa e são associadas à leitura mais próxima antes da exposição.

Uma leitura nunca expira, pelo que nenhum quadro se perde devido a uma leitura tardia — em vez disso, é rotulada como:

- **O fluxo do DAQ ficou bloqueado** — é utilizada a sua leitura mais recente, e o `gap_ms` indica a sua antiguidade (`in_tolerance: false` com mais de 500 ms).
- **Não há DAQ para ler**— desligado (`disconnect()`, `chloros-cli daq pool-disconnect`) ou o DAQ associado não está acessível — a última leitura da câmara é reutilizada e descrita em relação a**esta** exposição: `method`, `retained` e `gap_ms` desta exposição, `in_tolerance` é recalculado, e `retained_from_epoch_ns` indica a exposição à qual foi inicialmente associado. Uma câmara que nunca teve uma leitura continua sem refletância, e o resultado indica o motivo (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ou a recusa da captura).

Os ficheiros que uma captura guarda dentro da pasta do projeto aberto no Chloros juntam-se à lista de ficheiros desse projeto, tal como acontece com uma captura na aplicação (os ficheiros guardados noutro local não são afetados): `CameraSession.capture()`, `ArraySession.capture()` (`smart=True` também) e `capture_cameras_transient()`. O resultado indica isso em `.project_add` (`capture_cameras_transient()`: o `project_add` do dicionário) -- `held`, `files` e o `message` do backend; o `held` é o `True` enquanto uma execução de processamento estiver em curso, e os ficheiros são reunidos assim que esta terminar. `None` quando nenhum ficheiro foi adicionado ao projeto.

`CameraSession.capture()` e `ArraySession.capture()` devolvem-nos como o resultado `.daq_files` e `.daq_matches`, como as respostas de captura HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) contêm `daq_files` e `daq_matches`, um por `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` e `stream_fault` — a falha de fluxo própria do DAQ, consultada apenas para uma leitura fora da tolerância. Antes de confiar na refletância de uma captura, verifique o `in_tolerance` no seu `.match.json` ou XMP, ou os níveis [`status()`](#daqsensorsession-methods).

Níveis de `processing`:

| Nível | Saída |
| --- | --- |
| `"raw"` | Bayer de canal único (câmaras monocromáticas: a banda única) diretamente do sensor. |
| `"debayered"` *(predefinição do SDK)* | Cor de 3 canais: o desvio de escuro (DSNU) do pacote de calibração retirado do raw, seguido do demosaico sensível a arestas do OpenCV — sem LUT de linearidade, campo plano ou separação (câmaras monocromáticas: escala de cinzentos de 1 canal). O ficheiro TIFF guardado está na ordem de canais **RGB**(RGB fotométrico, pelo que qualquer visualizador de imagens mostra as cores reais); os vetores em memória que o `grab()` / `frame_stream()` devolvem estão na ordem**BGR** do OpenCV — inverta os canais (`frame[..., ::-1]`) para comparar com um ficheiro guardado. `chloros-cli process` escreve o mesmo resultado que `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm através da cadeia radiométrica completa. Apenas multiespectral — as câmaras RGB são ignoradas. |
| `"reflectance"` | uint16 0..32768 (compatível com Pix4D); requer um emparelhamento DAQ em tempo real para referência absoluta. Apenas multiespectral. |
| `"display"` | Cadeia completa correspondente à pré-visualização da GUI (CCM + WB + gama de acordo com o perfil da câmara). |
| `"all"` | **Um ficheiro por nível aplicável** para cada câmara (correspondendo à predefinição da GUI «Capture All» / CLI). O `CaptureResult` devolvido contém então um dicionário de fotogramas por `(cam, level)`, com o nível em cada dicionário; os níveis inaplicáveis aparecem em `.skipped`. A leitura do DAQ utilizada na captura é guardada como um ficheiro auxiliar `.daq`. |

> **Nota — o valor por predefinição difere do da CLI.** O `ArraySession.capture()` tem como valor por predefinição o `processing="debayered"`; o comando `chloros-cli lattice array-capture` tem como valor por predefinição o `processing="all"`. Passe o `processing="all"` explicitamente a partir do SDK para espelhar o salvamento em vários níveis da CLI/GUI.

##### Nomes dos ficheiros de captura de matriz

Cada nível é guardado na sua própria subpasta de `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` para `display`, `index/`, `composite/`). Cada ficheiro identifica a sua câmara como `SN<serial>-<filter>`: o número de série, seguido da parte do filtro do modelo da câmara — `SN214000533-FOCN` para uma `M3C-L87-FOCN`, `SN261902142-F450` para uma `M3M-L87-F450` — juntamente com `-`, pelo que um nome tem os mesmos campos separados por `_` que anteriormente. Uma câmara cujo modelo não pode ser lido é apenas `SN<serial>`, e as capturas de versões anteriores não têm qualquer filtro no nome. O nome do ficheiro depende do número de níveis solicitados a um membro:

| Nível solicitado | Nome do ficheiro | Exemplo (`reflectance`, predefinição `format="tiff"`) |
| --- | --- | --- |
| **Um nível** (`processing="reflectance"`, ou um `levels` com um único item) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — sem sufixo de nível, tal como nas versões anteriores, para que os scripts existentes continuem a funcionar | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Vários níveis** (`processing="all"`, ou `levels` com mais de um item) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

A [máscara de limitação](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) segue o nome do produto, `clamp_masks/<product folder>/<product name>_clampmask.tif`: a captura de refletância de um úniconível de captura de refletância grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, enquanto que `processing="all"` grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. A regra do sufixo é determinada para cada membro com base nos níveis solicitados, pelo que um nível omitido (por exemplo, refletância numa câmara RGB) não a altera. Leia os caminhos a partir dos fotogramas devolvidos, em vez de os construir. Uma captura com uma única câmara (`CameraSession.capture()`) adiciona sempre o nível: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Os formatos PNG/JPEG aceitam apenas a visualização e os produtos de índice

Um ficheiro PNG ou JPEG tem 8 bits por canal. Se for solicitado um ficheiro com um nível que não possa suportar, o pedido é recusado antes de qualquer captura, com uma mensagem a indicar o nível e o formato TIFF como o que o suporta:

- **`radiance`**(float32) e**`reflectance`**(uint16, `32768` = 1,0) — e, por conseguinte, `processing="all"`, que os inclui. O SDK verifica isto nos argumentos: `capture()` gera**`ValueError`** sem uma solicitação (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` provenientes de uma câmara a transmitir mais de 8 bits** (M3C de 12 bits, Mono12 M3M) — o backend rejeita estes, conhecendo a profundidade de bits de cada câmara: 400 HTTP com `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, gerado como `ChlorosConnectError` (`CameraSession` / `ArraySession`) ou `RuntimeError` (`CameraHandle` / `ArrayHandle`). A captura de nível único `debayered` de uma câmara obtém um fotograma de 8 bits para PNG/JPEG e não é rejeitada.

Guarde os dados radiométricos como `format="tiff"` (`ext=".tiff"`); utilize PNG/JPEG para `display` e renderizações indexadas.

#### Como funciona a exposição automática

- **Primeiro a exposição, depois o ganho.** Com o ganho em «Auto», este permanece em 0 dB até que a exposição atinja o valor máximo permitido.
- **Uma exposição manual mantém o ganho em «Auto»** até que altere o ganho ou o defina para «Manual». De volta ao modo «Auto», o limite do ganho é o mesmo de antes.
- **Pequenos realces não escurecem a cena.** Um realce especular até 1 % do enquadramento é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5 % de um painel.
- **O limite máximo de exposição do DAQ aplica-se apenas enquanto um alvo de calibração estiver visível.** Impede que um painel branco sofra clipping; quando não há nenhum painel visível, a câmara expõe de acordo com a cena.
- **A luz constante mantém-se estável e a luz alterada estabiliza em segundos.** Com luz constante, a exposição e o seu limite permanecem inalterados e, após uma grande alteração na luz, a imagem estabiliza em poucos segundos. Um clipping já não limita a exposição de forma definitiva, e uma alteração na área de interesse (ROI) restabelece a exposição.
- **O alvo de brilho permanece ao alcance.** Não aumenta enquanto a câmara estiver no seu limite de exposição e o seu ganho não puder subir, pelo que uma luz que regresse não provoca um reflexo no enquadramento.
- **Uma câmara cuja exposição automática deixa de responder é mantida no nível correspondente à luz em que se encontra.** Sob sol intenso, uma câmara pode descer para a sua exposição mais curta e permanecer nesse estado com um quadro escuro, com todas as suas definições a indicarem valores normais. O Chloros observa o quadro, mantém essa câmara no nível que ela mantinha para aquela luz e liberta-a quando a luz muda. Um sensor de luz escuro ou coberto não interfere neste processo.

#### Exposição nos membros da matriz

Uma câmara LATTICE limita o `ExposureTime` a um período da sua taxa de fotogramas. Enquanto uma matriz transmite, o Chloros executa cada membro com o `AcquisitionFrameRateEnable` ativado, a uma taxa ligeiramente superior à taxa de disparo, de modo que o limite máximo de exposição de um membro fique um pouco abaixo do período de disparo, **em qualquer ROI**:

- O período de fotograma de um membro é o período de disparo menos uma margem de 15 % do período (no mínimo 3 ms, no máximo metade do período), e o seu limite máximo de exposição é esse período de fotograma menos cerca de 0,1 ms. Exemplo: um disparador de 10 fps (100 ms) resulta numa taxa de membro de cerca de 11,8 fps e num limite máximo de exposição de cerca de 85 ms.
- **Um `ExposureTime` manual com duração superior a esse limite máximo abranda o disparador para se adaptar a ele.** O Chloros abre primeiro o limite máximo desse membro para o valor e deixa a taxa mais lenta funcionar durante alguns períodos de disparo (a gravação pode demorar mais alguns segundos), depois grava-o, e o disparador não dispara mais rapidamente do que a exposição permite. Numa matriz de taxa automática, a taxa em que funcionava anteriormente é restabelecida assim que todos os membros cuja exposição manual o tiver abrandado voltarem à exposição automática. Um valor que a própria câmara não possa aceitar continua a ser recusado, com o intervalo que aceita agora indicado no erro, e a exposição automática do membro é restabelecida se tiver sido desativada apenas para essa gravação. Antes da versão 1.3.0, uma pequena ROI limitava cada membro ao próprio período de funcionamento livre do sensor, independentemente da taxa de disparo: 4,9 ms a 768 × 304, pelo que nenhum membro podia ser clareado.
- O Chloros nunca encurta uma exposição manual para se adaptar a um disparo mais rápido. Uma taxa automática mantém-se dentro do que as exposições manuais permitem; uma taxa fixada mais rápida do que a exposição permite deixa a esse membro um limite máximo suficientemente longo para si, e aceita disparos não mais rápidos do que a sua exposição permite. Reduza a exposição, ou a taxa fixada, quando todos os membros tiverem de responder a todos os disparos.
- **Leitura de retorno:** a leitura dos nós de um membro mostra `AcquisitionFrameRateEnable` True e `AcquisitionFrameRate` à taxa do membro; a ativação desativa-se quando a matriz pára.
- **A matriz é proprietária desses dois nós. Não escreva `AcquisitionFrameRateEnable` nem `AcquisitionFrameRate` num membro da matriz** — defina acom `target_fps` / `set_target_fps()`.

Escreva a exposição de um membro com `arr.set_member_settings(serial, exposure_time=20000)` (as palavras-chave de `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` faz o mesmo). O membro permanece na matriz. Não abra um membro da matriz com o comando `connect_camera()`: o `connect` recusa uma câmara que já se encontre numa matriz, e o ``CameraSession`` que seria devolvido remove a câmara da sua matriz quando esta se desliga. O ``ArraySession.status()["member_ae"]`` reporta a exposição de cada membroa partir do seu fotograma mais recente.

### Um registo de projeto, para todos os modos

Uma pasta de projeto contém as definições de todos os dispositivos: `cameras.json` — as de cada câmara (`cameras[].settings`, para uma única câmara e para cada membro da matriz) e as globais de cada matriz (`arrays[].capture`: formato, ROI, binning, taxa de disparo, orçamento de ligações) — e `sensors.json`, as de cada sensor de luz (tempo de integração, média de fotogramas, limite). A aplicação, a CLI, o SDK e o servidor MCP leem e escrevem os mesmos ficheiros, da mesma forma:

- **Um dispositivo ligado a um projeto é guardado nesse projeto e regista todas as alterações nele.** «No âmbito de um projeto» significa com um projeto aberto no backend (da aplicação) ou — através de um `ChlorosProject` (`open_project()`), que indica a sua própria pasta ao backend (`project_path`). Uma câmara, matriz ou sensor de luz que o projeto ainda não contenha é adicionado aos seus ficheiros quando se liga (`connect_camera()`, `connect_array()`, a ligação do sensor de luz, a CLI, a aplicação, `chloros-mcp`), com as respetivas definições. Uma alteração a partir de qualquer modo (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, as definições da matriz, `set_cap()`, o CLI, os controlos da aplicação) é guardada nos ficheiros desse projeto um instante depois. Uma exposição, ganho ou valor-alvo de AE definido numa câmara é guardado como mantido, pelo que, ao reabrir, este valor é mantido novamente.
- **Ao abrir um projeto, os seus dispositivos são configurados a partir dele**, em qualquer modo: uma câmara, matriz ou sensor de luz ligada sem definições próprias recebe as(`lattice cam-connect` / `array-connect` com um projeto aberto, `chloros-cli project connect`, `open_project().connect_all()`, ao reabrir a aplicação). Uma matriz ligada com um novo ID atualiza a matriz que o projeto guardou com as mesmas câmaras; uma matriz com outras câmaras é adicionada ao lado desta.
- **O próprio arquivo do aplicativo nunca sobrescreve as configurações ativas de um dispositivo** com a sua cópia mais antiga e mantém as substituições por membro guardadas por um chamador CLI ou SDK.
- **O formato de píxeis que definir é aquele que o projeto mantém.** Um formato definido nas «Definições da Matriz», por `--pixel-format` ou por `connect_array(pixel_format=)`, incluindo o de 8 bits, é guardado e restabelecido exatamente. Se não for definido, cada câmara liga-se com a sua predefinição de 12 bits (`BayerRG12`, `Mono12` num sensor monocromático). O nome do formato indica a sua profundidade de bits; a família é a do sensor, pelo que uma câmara a cores nunca é definida para um formato monocromático nem uma câmara monocromática para um formato Bayer, e a linha `[PixelFormat <serial>] ... written` indica isso quando um formato foi mapeado. As exportações de uma captura de 8 bits são de 8 bits, incluindo a refletância: redimensionadas e marcadas com o seu `PixelScale`.
- Um dispositivo ligado sem estar associado a nenhum projeto não regista nada.
- **Uma gravação segue um projeto criado depois dela; ao ser aberta, mantém o seu projeto.**Quando um projeto é criado — `create_project()`, o `chloros-mcp` do `create_project`, ou a opção**Novo Projeto**da aplicação — ou duplicado (a opção**Duplicar**da aplicação, que abre a cópia) enquanto uma gravação ainda está a gravar no projeto que estava aberto anteriormente, a gravação continua e o que é gravado acaba por ir para o novo projeto, no mesmo local dentro deste (abaixo). Ao abrir um projeto existente — `ChlorosProject.process()` (que abre o seu projeto no backend), `chloros-cli process`, a opção**Abrir Projeto** da aplicação — deixa uma gravação ainda em curso com o projeto no qual está a gravar: termina aí, e os seus ficheiros juntam-se à lista de ficheiros desse projeto (a aplicação interrompe todas as gravações antes de abrir um projeto).
  - `ArraySession.record()` e `ArraySession.burst()`: o vídeo, ou a pasta de sequência com todos os fotogramas, é movido na íntegra para lá quando a gravação termina. O resultado da paragem indica o novo caminho (`out_dir`, `video_path`), e a compilação da sequência é feita a partir desse caminho. Se a unidade recusar a transferência, a gravação permanece onde foi gravada, e o resultado indica isso (`relocation_error`). Os fotogramas em bruto de uma sequência interrompida e o ficheiro `.daq` são adicionados à lista de ficheiros do projeto em que a sequência é incorporada; um vídeo não é adicionado à lista de ficheiros de um projeto.
  - `ArraySession.continuous()`: todas as capturas após a alteração são guardadas no novo projeto; as que foram guardadas anteriormente permanecem onde estão.
  - `DAQSensorSession.record_start()` (e a gravação do sensor de luz da própria aplicação): o `.daq` é movido para a pasta `light_sensor` do novo projeto quando a gravação termina; o `record_stop()` devolve o novo caminho e o ficheiro passa a integrar a lista de ficheiros desse projeto.

  Uma gravação numa pasta fora do projeto aberto permanece onde está, e o `start_camera_continuous()` / tarefas multistream mantêm a pasta que lhes foi atribuída.
- **Na aplicação, as capturas efetuadas após a criação de um novo projeto são colocadas nesse projeto.** A criação ou duplicação de um projeto mantém todas as sessões de captura e gravações em curso, e cada captura efetuada a partir desse momento («Capturar Tudo», «Iniciar Captura») é guardada no novo projeto e adicionada à sua lista de ficheiros. Abrir um projeto existente na aplicação interrompe primeiro todas as capturas.
- **Um projeto não é movido enquanto está a gravar.**Mover o projeto aberto para outro diretório de trabalho (em**Definições → Diretório de Trabalho** da aplicação; nenhuma chamada ao SDK move um projeto, um cliente HTTP a enviar dados para `/api/move-project` sim)) é recusada com o código HTTP 409 `PROJECT_RECORDING` enquanto uma gravação em matriz ou em rajada, uma captura contínua em matriz, uma tarefa contínua/multi-stream da câmara ou uma gravação do sensor de luz estiver a escrever nele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Isto é distinto do estado em que as câmaras ficam no momento da desconexão (ver [Memória da câmara](#camera-memory--keep_settings--clear_settings)): um projeto configura as suas próprias câmaras quando é aberto.

### Memória da câmara — `keep_settings` / `clear_settings`

Quando um conjunto é ligado, as suas câmaras assumem as definições do conjunto, podendo depois alterá-las. Quando o Chloros liberta as câmaras — `ArraySession.disconnect()` / `force_disconnect()`, no final de uma função transitória, a paragem do `start_camera_continuous()` —, cada uma volta ao estado em que se encontrava antes da ligação: a sua ligação de disparador e linha de sincronização, as definições de PTP e da câmara — `ExposureAuto` / `GainAuto` (e um modo manual `ExposureTime` / `Gain`), `TargetBrightness` e os limites e amortecimento do AE, equilíbrio de brancos, nível de preto, `PixelFormat`, agrupamento de píxeis e a área de interesse (ROI), o fluxo de trabalho de cor (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, o PTP de `PtpSlaveOnly` / `PtpDomainNumber`, o tamanho do pacote (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), o ritmo (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), os dados do bloco (`ChunkModeActive`, cada `ChunkEnable` sob o seu `ChunkSelector`), a remoção de ruído da entrada de disparo — um array fornece a linha de sincronização de um escravo (`LineFilterSelector` / `LineFilterWidth`) e a taxa de fotogramas de aquisição (`AcquisitionFrameRate` apenas quando a sua ativação estava ligada: com ela desligada, a taxa indica o máximo que o ritmo permite, não uma configuração). Apenas o que difere é registado: a câmara parou primeiro, o ritmo e a taxa de fotogramas antes de uma exposição que necessita do seu período de fotograma.

Para utilizar as câmaras fora de um projeto, dois argumentos gravam na **própria memória** da câmara — aquilo com que ela arranca, em qualquer software (em `disconnect()` / `force_disconnect()`, todas as funções transitórias, `start_camera_continuous()` e `ChlorosProject.disconnect_all()`; MCP: os mesmos argumentos de `disconnect_device` e das ferramentas transitórias; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** guarda as definições de cada câmara tal como ficaram na memória no final da sessão (UserSet2, definido como conjunto de arranque). As câmaras continuam a funcionar livremente (`TriggerMode` `Off`), com a ligação da linha de sincronização tal como se encontrou. Uma câmara sem UserSet2 guarda as definições no UserSet1; este computador memoriza-as, pelo que a autocorreção da matriz dChloros — que redefine um UserSet1 que se desviou da linha de base dChloros — não a altera.
- **`clear_settings=True`** repõe a memória de cada câmara para as definições de fábrica: a linha de base do Chloros, aquela que a autocorreção da matriz incorpora — predefinições de fábrica com `TriggerMode`, `Off`, `TriggerSource`, `Line2`, exposição automática, o equilíbrio de brancos calibrado de uma câmara RGB mantido — no UserSet1, que constitui o conjunto de arranque.
- Não ambos (`ValueError`; o backend responde com 400). A resposta contém `camera_memory`, por câmara `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` ou `{"cleared": bool, "errors": [...]}` — o `force_disconnect()` devolve-o; para uma tarefa, vem com o `stop()`.

A aplicação não disponibiliza estes valores.

**Todas as saídas repõem as câmaras.**A reposição tal como encontrada é executada em cada caminho de saída: `CameraSession.disconnect()` (a câmara**X**da aplicação), o**X** do aplicativo numa matriz ou em qualquer elemento da matriz (a matriz inteira é libertada), encerrando o aplicativo (o backend tem até 20 s para restabelecer todas as câmaras, matrizes e gravações contínuas em execução; uma função transitória ou uma chamada única ainda em execução pára no seu próximo ponto seguro, restabelece primeiro as suas câmaras, durante um máximo de 8 s, e responde com HTTP 503 `SHUTTING_DOWN`; um `connect_array()` ainda em execução pára antes da sua próxima câmara e repõe as câmaras que abriu, durante um máximo de 10 s, e gera o erro `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); e um que chegue durante o encerramento não abre nada: HTTP 503 `SHUTTING_DOWN`), e um comando `chloros-cli` que é encerrado ou cujo terminal se fecha (a sua tarefa é interrompida e as suas câmaras são devolvidas em primeiro lugar). Um backend que falha ou é encerrado à força não consegue devolver nada; um reinício devolve cada câmara à sua memória.**Um projeto define as suas próprias câmaras.** Quando um conjunto se liga através de um projeto — o projeto aberto da aplicação ou um `ChlorosProject`, que se identifica junto do backend (`project_path`) — cada membro recebe as definições que o projeto guardou para ele: uma exposição, ganho ou alvo de AE mantidos para ele (mantidos como seus, pelo que a exposição automática não os altera) e o seu modo de equilíbrio de brancos (correção de Chloros; o equilíbrio de brancos da própria câmara permanece desativado), independentemente do que a memória da câmara contenha.

### Modos de Captura e Gravadores

A superfície da matriz reflete o painel de captura da GUI: modos de obturador Único / Contínuo / Intervalo / Mais rápido, além de dois gravadores (vídeo composto em direto e rajada em formato RAW → reprocessamento offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**é o ciclo Contínuo/Intervalo do SDK. Como não existe o `Ctrl+C` para interromper este ciclo a partir de um script,**deve** passar o `count` e/ou o `duration_s` (o ciclo termina quando qualquer um deles for atingido). O `interval_s` é medido a partir do início de cada passagem (em consonância com a GUI). As passagens contínuas (`interval_s=0`) encadeiam `group_seq` → `after_seq`, pelo que são grupos sincronizados consecutivos na taxa da matriz. Os restantes kwargs passam diretamente para `capture()`.
- **`continuous`**é a captura contínua própria do backend — o modo Contínuo da GUI.**Utilize-a em vez de `capture_repeated()` quando pretender que todos os eventos sejam gravados no disco a uma elevada taxa de fotogramas:** O `capture_repeated()` efetua uma solicitação HTTP por grupo a partir do seu script, enquanto o `continuous()` mantém duas capturas em andamento no backend, sem ida e volta entre grupos. Medido no ROI 512 num conjunto de 9 câmaras: 73–77 capturas/s, em comparação com um conjunto que publica cerca de 78. Mantenha o `capture_repeated()` para a captura por intervalo (`interval_s`) e para as passagens que inspecionar uma a uma. No Windows, exclua o backend do Microsoft Defender para taxas elevadas (consulte [Passo 1](#step-1--install-the-chloros-platform-package)).
- **`record`** é de *nível de monitorização*: regista o composto de índice combinado renderizado em tempo real. Com a vista combinada da GUI aberta, grava essa vista tal como é apresentada. Se nenhuma estiver aberta, o backend renderiza a própria vista: `expr` se passar uma, caso contrário, o NDVI da matriz (as bandas vermelha e NIR mais próximas de 660 nm e 850 nm, conforme a visualização em tempo real as seleciona), assim que a matriz estiver alinhada. Nomeie as bandas em `expr` conforme a matriz as denomina: `<band>_<filter>` para uma câmara M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` para uma M3M (`Mono_850`); um nome curto de cor (`NIR`, `Red`) só funciona quando uma banda da matriz o possui. Uma vista da GUI aberta durante a gravação é gravada enquanto permanecer aberta. Uma gravação que não escreve nenhum fotograma falha com `error` e `"no-frames: …"` (a matriz pode ainda estar a alinhar-se ou a estabilizar a sua exposição), e o `stop()` é acionado (ver [`RecorderHandle`](#recorderhandle)). Um gravador composto por matriz (aciona-se se já houver um em execução).
- **`burst` → `build_video`** é de *nível de análise*: `burst` grava fotogramas em formato raw + um manifesto por fotograma + um `.daq` por cada leitura DLS distinta sob `<output>/bursts/<base>/` à taxa máxima do ciclo de captura (sem cadeia, sem exiftool, sem visualização em tempo real). O `build_video` sincroniza temporalmente cada fotograma com o `.daq` mais próximo e reexecuta a cadeia de radiancia/refletância/índice do pipeline de importação. O `products` é uma lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (por predefinição: o índice combinado). O `burst().stop()` também inicia automaticamente a construção de um, devolvida como `build_job` no resultado de paragem.

##### O resultado `build_video`

| Chave | Significado |
| --- | --- |
| `success` | `True` apenas quando todos os produtos solicitados foram criados conforme solicitado: pelo menos uma saída, nada em `skipped`, e nada em `degraded`. |
| `outputs` | Caminhos dos vídeos (e GIFs) gravados. |
| `errors` | Problemas, em texto. |
| `skipped` | Um `{"serial", "level", "reason"}` por produto solicitado que **não** produziu nenhum vídeo — seja devido a um problema da câmara (ausência de pacote de calibração para essa câmara, ausência de leitura DLS para refletância, falha na cadeia de processamento) ou do índice combinado, que inclui `"kind": "combined"`, `serial` e `None` (sem par de bandas vermelha/NIR entre as câmaras, sem grupo com radiancia para ambas as bandas). Qualquer entrada gera o `success` e o `False`; os outros vídeos continuam a ser gravados e listados. |
| `substitutions` | Câmaras renderizadas a um nível diferente do solicitado, e porquê — uma câmara RGB que solicitou radianância/refletância obtém o seu vídeo de visualização, com equilíbrio de brancos, quando o pacote de calibração da câmara está disponível. Não se trata de uma falha. |
| `notes` | Texto informativo, não problemas — por exemplo, uma câmara sem expressão de índice específica não tem vídeo de índice, ou o vídeo de exibição de uma câmara RGB não tem equilíbrio de brancos (sem pacote de calibração ou com um sem equilíbrio de brancos de fábrica), indicando o nome da câmara. Não é uma falha. |
| `degraded` | `True` quando um produto foi criado, mas não é o solicitado: o índice combinado offline é a diferença normalizada vermelho/NIR incorporada, pelo que uma expressão de índice configurada não é avaliada. Gera `success` `False`. |
| `combined` | Resumo do índice combinado: as bandas utilizadas, o intervalo da LUT, os fotogramas criados. |
| `clamp_masks` | Caminhos das [máscaras de limitação de desmistura](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) dosTIFFs calibrados por fotograma (`save_tiffs=True`), em `clamp_masks/<level>/` ao lado da pasta `<level>/` dos TIFFs, uma por TIFF em que um pixel foi limitado. Um vídeo não contém nenhuma. |
| `clamp_fractions` | O que um vídeo indica em vez de uma máscara: um `{"kind", "serial", "level", "frames"}` por produto de vídeo (`kind` `"combined"` para as câmaras que integram o índice combinado), `frames` uma lista de `[frame, R, G, B]` — a percentagem de píxeis desse fotograma que o processo de separação limitou na banda que o seu canal R / G / B contém (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Devolvido por `ArraySession.record()` e `ArraySession.burst()`. Utilize-o como um gestor de contexto para parar automaticamente ao sair do escopo, ou controle-o manualmente.

| Membro | Descrição |
| --- | --- |
| `job_id` | ID da tarefa de backend (str). |
| `kind` | `"composite"` (de `record`) ou `"raw"` (de `burst`). |
| `start_stats` | O dicionário devolvido pela chamada `start`. |
| `result` | `None` durante a execução; o dicionário final do resultado da paragem, uma vez parado. |
| `stats(timeout=10.0)` | Estatísticas da tarefa em tempo real (fotogramas gravados, fps alcançados, tempo decorrido). |
| `stop(timeout=60.0)` | Interrompe o gravador; devolve e armazena em cache o resultado final (`.result` permanece como `None` se o próprio pedido de interrupção falhar). No caso de uma gravação `record()` que tenha falhado — uma que não tenha gravado nenhum fotograma (`error` `"no-frames: …"`) — o resultado é armazenado em cache em `.result` e, em seguida, é gerado o `ChlorosConnectError`. Idempotente (uma segunda chamada devolve o resultado armazenado em cache, sem gerar exceção). Sair de um bloco `with` interrompe o gravador sem gerar um erro, por isso verifique o `rec.result.get("error")` a seguir. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Devolvido por `ArraySession.continuous()`. Utilize-o como gestor de contexto — sair do bloco `with` interrompe a captura se esta ainda estiver em execução — ou execute-o com `status()` / `wait()` / `stop()`. É executada uma captura contínua por matriz de cada vez.

| Membro | Descrição |
| --- | --- |
| `start_status` | O dicionário de estado devolvido pela chamada de início. |
| `result` | `None` durante a execução; o dicionário de estado final assim que a captura terminar (definido por `stop()`, ou por um `status()` / `wait()` que tenha detetado o seu fim). |
| `status(timeout=10.0)` | Dicionário de estado em tempo real (campos abaixo). |
| `wait(timeout=None, poll_s=0.5)` | Bloquear até que a captura termine por si própria — quando `max_captures` ou `duration_s` for atingido, a matriz desaparecer, a unidade ficar cheia ou ocorrer um erro — e devolver o estado final. Com `timeout`, devolve o estado mais recente após esse número de segundos, mesmo que ainda esteja em execução: verifique `running`. |
| `stop(timeout=30.0)` | Interrompe a captura e devolve o seu estado final. Idempotente (uma segunda chamada devolve o resultado armazenado em cache). |

Campos de estado:

| Campo | Significado |
| --- | --- |
| `running` | Verdadeiro enquanto a captura estiver a decorrer. |
| `captures` | Grupos sincronizados guardados. |
| `files` | Ficheiros gravados. |
| `failed_files` | Ficheiros que não foi possível gravar (conforme em [`CaptureResult.failed`](#ficheiros-que-não-foram-gravados--resultado-da-captura-falhou)). |
| `failed_captures` | Tentativas de captura que falharam na totalidade, incluindo aquelas em que a captura foi repetida (um erro HTTP 503 enquanto o conjunto não tinha nenhum novo grupo a tempo). |
| `skipped_groups` | Grupos publicados pela matriz que não foram guardados (a captura ficou atrasada). |
| `captures_per_s` | Taxa média de gravação desde o início. |
| `elapsed_s` / `started_at` | Segundos de execução até ao momento; hora de início (segundos da época Unix). |
| `last_group_seq` | O grupo sincronizado mais recente que foi guardado. |
| `last_error` | O erro mais recente, ou `None`. |
| `stop_reason` | Uma vez concluído: `count` (`max_captures` atingido), `duration` (alcançado o `duration_s`), `stopped` (chamado o `stop()`), `array_gone` (a matriz foi desligada), `error` (20 falhas consecutivas ou um pedido rejeitado pelo backend), `disk_full` (um ficheiro para o qual a unidade não tinha espaço). `None` durante a execução. |
| `max_captures` / `duration_s` | Os limites com que foi iniciado (`0` = nenhum). |
| `defender_cores` | Número de núcleos da CPU utilizados pelo Microsoft Defender nos últimos 10 segundos; `None` até que seja medido um, e sempre `None` num anfitrião sem o Defender. |
| `defender_warned` | Verdadeiro assim que a captura registar que o Defender a está a tornar mais lenta (com 1 núcleo ou mais). Exclua o backend da verificação do Defender — consulte [Passo 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Quadros em tempo real no seu próprio pipeline

Para enviar fotogramas sincronizados para o seu próprio código — aprendizagem automática, um visualizador personalizado, o seu próprio gravador — utilize o `ArraySession.frame_stream()`. Cada tick gera um `GrabResult`: `{serial: numpy array}` para cada câmara, **tudo a partir de um único disparador**, com o ID desse disparador no `.group_id`. Trata-se da mesma implementação que o `ArrayHandle.frame_stream()` da API do projeto, pelo que tudo o que está em [Um disparador por leitura](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) e [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) aplica-se a ambos.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Cada conjunto gerado constitui um novo gatilho.** O `frames.group_id` nunca se repete dentro de um fluxo; quando se solicita mais `fps` do que a velocidade a que a matriz está a funcionar, o fluxo aguarda pelo próximo gatilho, pelo que funciona à velocidade mais lenta das duas.
- **As câmaras em falta são identificadas, não preenchidas retroativamente.** Uma câmara que não pôde ser atendida nesse gatilho está ausente do dicionário e é listada em `frames.errors`.
- **Termina de forma evidente.** O `FrameFetchError`, em caso de recusa, repetir-se-ia no tick seguinte, após 60 s sem qualquer fotograma ou após 60 s sem um novo gatilho.**O que proporciona.**O `frame_stream()` transfere fotogramas numpy completos através da ligação HTTP local e atinge um pico próximo dos**170 MB/s**. Isso é suficiente para o processamento em tempo real de fotogramas em bruto, ou de uma matriz agrupada ou de ROI, mas não para cada gatilho na resolução máxima — para manter**todos os** gatilhos, guarde no disco com [`continuous()`](#continuoushandle). Medido num conjunto de quatro câmaras (FOCN, FRGN, FRGB, F450) com transporte compactado; cada tick correspondia a um conjunto completo de um disparador:

| | Sensor completo (matriz 14,2–14,4 fps) | Agrupamento 2 × 2 (matriz ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 conjuntos/s | 22,2 conjuntos/s |
| `frame_stream(processing="debayered")` | 2,3 conjuntos/s | 9,3 conjuntos/s |
| `frame_stream(processing="display")` | 0,8 conjuntos/s | 3,0 conjuntos/s |
| MJPEG por câmara, 4 fluxos a 720p | 13,5 fps cada; cada fotograma correspondente a um disparador | ~25 fps cada; ~45 % dos disparadores presentes nos 4 |

Utilize o `processing="raw"` para obter maior débito e efetuar o demosaico do seu lado; o nível `debayered` também elimina o desvio de escuro do pacote de calibração, o que não acontece com a desmosaicização do `raw`.

#### Fluxos MJPEG por câmara (sem RTSP)

**O RTSP não é suportado.** A interface de transmissão é MJPEG sobre HTTP (`multipart/x-mixed-replace`, um JPEG por parte, cada parte com um `Content-Length`), um URL por câmara:

| URL | O que fornece |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Pré-visualização de 8 bits, até ~30 fps. Para um membro do conjunto, o `max_h` (padrão 720, 64–2160) e `jpeg_q` (padrão 80) definem o tamanho e a qualidade. |
| `GET /api/camera/<serial>/stream-annotated` | A mesma pré-visualização com as sobreposições da GUI desenhadas sobre ela (zebra, grelha, retículo, histograma, picos, medidor pontual). O `ChlorosLocal.camera_stream()` lê-a por si. |
| `GET /api/camera/<serial>/stream-reflectance` | Refletância de 8 bits por banda — consulte [Fluxo de refletância em tempo real](#live-reflectance-stream-mjpeg). |

- **Cabeçalhos de disparo.** Para um elemento da matriz, cada parte de todos os três fluxos contém `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Partes de câmaras diferentes com o mesmo `X-Chloros-Group-Id` constituem o mesmo gatilho: reúna as partes e agrupe-as com base nesse cabeçalho para recompor um conjunto. Uma câmara autónoma não tem grupo: as suas partes de pré-visualização não contêm nenhum destes cabeçalhos (as suas peças de refletância apresentam um `X-Chloros-Group-Id` vazio).
- **O fluxo de pré-visualização perde fotogramas sob carga.** Apresenta o fotograma mais recente e ignora qualquer um que não tenha conseguido enviar a tempo; por isso, a taxas de matriz elevadas, nem todos os gatilhos chegam ao fluxo de todas as câmaras (ver a tabela acima). Para cada disparador, utilize `frame_stream()` ou `continuous()`.
- **O `/api/camera/stream/<serial>` fica em pausa enquanto o fluxo ao vivo combinado da matriz estiver aberto** (`/api/camera/array/stream/<array_id>` — a vista da matriz na GUI abre-o). A transmissão de cada membro envia então um fotograma de preenchimento «Pausado (transmissão da matriz ativa)», sem cabeçalhos `X-Chloros-*`, e nada mais até que o fluxo combinado seja encerrado; em seguida, retoma por si próprio. O `stream-annotated` e o `stream-reflectance` não entram em pausa. Feche a vista de matriz da GUI quando um script ler os fluxos de pré-visualização de cada câmara.
- **Para números, e não imagens, utilize o `frame_stream()`.**O formato JPEG é com perdas e de 8 bits; o `frame_stream()` devolve os mesmos resultados sem qualquer codec no caminho.

### Captura por intervalos para levantamentos com drones

Um voo de levantamento necessita de um grupo de imagens sincronizadas a cada poucos metros de percurso, e não de todos os fotogramas que a matriz consegue produzir. Duas definições controlam isto:**`target_fps`**define a frequência com que as câmaras captam um fotograma, e**`capture_repeated(interval_s=…)`** define a frequência com que um fotograma é guardado.**As câmaras continuam a transmitir entre os guardados.** Cada intervalo guarda o grupo sincronizado mais recente; os fotogramas intermédios não são gravados. A exposição automática ajusta-se em cada fotograma, pelo que a exposição já está estabilizada quando a gravação ocorre. Não existe nenhum modo em que as câmaras parem entre as capturas e sejam ativadas para cada uma delas: cada captura teria então de esperar que a transmissão começasse e que a exposição se estabilizasse.**Defina a taxa de fotogramas de acordo com as necessidades do intervalo.**Se mantida em modo automático, uma matriz funciona à taxa mais elevada que o anfitrião e a rede suportam. Definir uma taxa mais baixa reduz o aquecimento da câmara, a carga da rede e a utilização da CPU do anfitrião, sem alterar o que é guardado. Escolha uma taxa que proporcione**2–4 fotogramas por intervalo**:

- Cada gravação aguarda o fotograma seguinte após ser solicitada, pelo que um grupo gravado chega até um período de um fotograma após o momento previsto. Uma taxa mais elevada mantém essa diferença reduzida.
- A exposição automática ajusta-se uma vez por fotograma, pelo que uma taxa muito baixa diminui a rapidez com que a exposição acompanha uma nuvem que passa.

**Calcule o intervalo a partir do plano de voo:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Por exemplo, uma área de cobertura de 60 m com 80 % de sobreposição para a frente corresponde a um espaçamento de 12 m; a 10 m/s, isso corresponde a `interval_s=1.2`, e um `target_fps=3` fixo resulta em 3 a 4 fotogramas por intervalo. A captura é baseada no tempo: o SDK não dispõe de modo acionado por distância ou GPS, pelo que deve manter uma velocidade ao solo constante nas linhas de levantamento.

**Guarde os dados brutos na aeronave e calibre mais tarde.** O `interval_s` é medido a partir do início de cada gravação; por isso, uma gravação que demore mais do que o intervalo faz com que a seguinte comece mais tarde e o espaçamento se desvie. Guardar os dados brutos juntamente com a leitura do sensor de luz (`processing="raw"`; a leitura é guardada por predefinição) mantém cada gravação curta e garante que todos os fotogramas possam ser convertidos em refletância posteriormente.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fixa a taxa desde o início;**`arr.set_target_fps(fps)`**altera-a a meio do voo sem necessidade de reconexão, e**`arr.set_target_fps(None)`** volta a colocá-la em modo automático. Uma taxa superior à capacidade do anfitrião e da rede é reduzida para esse limite máximo — verifique `clamped` no resultado — e a taxa mínima é de 0,1 fps.
- **Não utilize o `burst()` para reduzir o número de fotogramas.** Este comando grava todos os fotogramas que a matriz produz; segue-se a um `target_fps` fixado, pelo que é a ferramenta adequada apenas quando se pretende todos os fotogramas.
- **O `target_fps` não corrige fotogramas corrompidos.** Reduzir este valor diminui o número de fotogramas produzidos, não a velocidade a que cada um é enviado; para fotogramas corrompidos pelo GVSP, reduza antes o valor de `wire_ceiling_mbps` (consulte [Estado da Matriz](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Ligar-se a um conjunto já ligado — `attach_array`

Se o conjunto já estiver ativo (a GUI abriu-a ou uma sessão anterior do SDK chamou o `connect_array`), utilize o `attach_array` para obter um identificador da mesma, em vez de se voltar a ligar. O `connect_array` apresenta sempre o erro «A câmara  <sn>já está na matriz<id>», nessa situação, porque o envio de um POST com o código `/array/connect` para um membro do conjunto não é idempotente; `attach_array` lê `/api/camera/array/list` e faz a correspondência com base no array_id ou nos números de série.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Padrão: os scripts do SDK que partilham o ambiente com a GUI do ambiente de trabalho devem tentar primeiro o `attach_array` e recorrer ao `connect_array` se ainda não houver nenhuma matriz no conjunto.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Uma matriz anexada é emprestada, não pertence ao utilizador.** Um identificador de `attach_array` nunca desativa a matriz: nem a saída do bloco `with` nem uma chamada explícita de `disconnect()` a libertam (a chamada explícita imprime uma linha no stderr a indicar que deixou a matriz em execução). A mesma regra que se aplica a `CameraSession` / `DAQSensorSession`. Portanto, a partilha de espaço com a GUI é segura, com ou sem `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Quando pretender realmente encerrá-la — para se reconectar com um `wire_ceiling_mbps` diferente, por exemplo — chame `arr.force_disconnect()`. A reconexão após um simples `disconnect()` num identificador associado falha com «A câmara <sn>já </sn>está <sn>na matriz<id>», porque nada foi libertado.

### Auxiliar de Análise de Rede

Útil antes de abrir a matriz — permite prever se as configurações propostas serão adequadas:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nunca inicia um backend: um que não responde gera o erro `ChlorosConnectError`, enquanto um pedido recusado é devolvido como resultado. Um `packed` inválido gera o erro `ValueError`.

O `status` é um dos seguintes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (caso contrário, `error` — incluindo um pedido recusado: sem sessão iniciada, `error_code: AUTH_REQUIRED`, ou um plano abaixo do limite mínimo do SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` significa que a resolução solicitada só se adapta ao anel RX a uma taxa de disparo limitada — mantenha a resolução e passe de `target_fps=result["recommended"]["recommended_target_fps"]` para `connect_array` (ver [Exemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` aparece apenas quando o fotograma solicitado não pode ser transmitido a qualquer taxa de fotogramas; o connect nunca reduz os fotogramas por si próprio, pelo que deve aplicar o `recommended` manualmente.

**Como interpretar a projeção** (mesmo modelo que o painel «Configurações da Matriz» da GUI):

- **Burst (`frame_bytes_total`) é somado por câmara no formato de píxeis real de cada câmara.**As câmaras**M3M**transmitem Mono12 (2 B/px) ou, com um `pixel_format`, o formato Mono da sua profundidade (`"BayerRG8"` é `Mono8`, 1 B/px), pelo que, sem nenhum, um fotograma de resolução total de 4 câmaras tem**~25 MB** com três câmaras mono, e não os ~12,6 MB que a hipótese de tudo a 8 bits daria. O backend determina o formato de cada câmara a partir do seu modelo.
- **A admissão (`burst_fits_nic_ring`) tem em conta a capacidade de escoamento**, e não se baseia na comparação entre «burst completo» e «anel»: o «sim-emit» é adequado quando o anfitrião esvazia o anel de receção mais rapidamente do que as câmaras o enchem. Um anfitrião de 10 G + câmaras de 1 GbE**admite** resolução total mesmo quando o burst excede o anel; um anfitrião de 1 GbE bloqueia (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` é um limite máximo conservador para a recuperação em série** — `max(readout+emit, N×emit)` com a emissão por câmara limitada à ligação de câmara de 1 GbE, independentemente da exposição. Por exemplo, ~2,8 fps para um conjunto de 4 câmaras a resolução total de 12 bits (corresponde aos ~2,7–3,0 medidos no tempo de execução). Modelo completo: [Referência CLI → Modelo de fps e rajadas do conjunto de câmaras](cli-reference.md#array-fps--burst-model).
- **Após uma captura contínua, a projeção indica o valor que se manteve.** Uma captura de 30 s ou mais a uma taxa automática regista a taxa em que se estabilizou para aquele tamanho de matriz neste computador; os breves testes do controlador de taxa a uma velocidade mais elevada são ignorados. A partir daí, a linha do `array-connect`, a análise de `connect_array()` e o painel «Definições da Matriz» indicam essa taxa quando esta é inferior à do modelo, com o que a limitou em `cycle_limiter`: `"clock"` (a matriz funcionou à taxa em que se estabilizou, com o ciclo de captura à espera do seu relógio de disparo), `"loop"` (o ciclo de captura ficou sem tempo), `"capture"` (a captura ficou atrasada e saltou grupos) ou `"measured"` (não há indicação de qual). O valor do modelo encontra-se em `wire_fps_max`. A matriz continua a atingir o valor que a ligação transmite.
- **A sobre-subscrição (`oversubscribed: true`) significa que a ligação não consegue transmitir a matriz a qualquer ritmo que as câmaras atinjam** — os campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) apresentam o valor 0, e o binning não consegue resolver o problema (pois reduz os bytes por fotograma, e não os bytes por segundo). Uma ligação lenta, por si só, não causa isto: em vez disso, a taxa de fotogramas diminui e, se as câmaras não puderem ser reguladas a um ritmo suficientemente lento, o nível passa a ser `slip-emit-and-capture` (serial). As soluções consistem em utilizar menos câmaras, quadros jumbo ou uma placa de rede mais rápida; o `max_cams_collision_safe` indica o limite máximo da sua ligação e das câmaras. A resposta também inclui `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps`. Consulte [Sobresubscrição](#over-subscription-the-per-cam-floor).

### Descoberta e listagem

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Cada uma aceita apenas palavras-chave — `backend_url=` e `timeout=30.0` — e devolve uma lista de dicionários. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) também aceita `auto_start_backend=True`; as chamadas `list_*` nunca iniciam um backend.

---

## Smart-AE / Smart-Capture

As matrizes LATTICE executam AE contínuo em segundo plano assim que são ligadas, mas uma cena recém-focalizada demora um momento a convergir. **O Smart-Capture** é a solução prática integrada: verifica a exposição de cada câmara, aguarda até que a matriz esteja estável numa janela e, em seguida, aciona a captura. É equivalente à interface gráfica: o botão de captura «inteligente» da aplicação para computador chama o mesmo ponto final do backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Ao utilizar o `ChlorosProject` (próxima secção), tem à sua disposição mais opções de ajuste:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

A política de AE inteligente é conservadora por predefinição. Torne o `exposure_tolerance_pct` mais restritivo para trabalhos radiométricos exigentes; torne-o mais flexível para cenas em rápida mudança, nas quais apenas se pretende um resultado «suficientemente próximo».

---

## Sessões de sensores DAQ

Conjunto de backends persistentes para sensores espectrais (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Espelha a superfície da câmara: deteção inteligente, reutilização do conjunto, ligação idempotente.

### Detecção Inteligente (Configuração Zero)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioridade: Ethernet → BLE → USB. Passe qualquer indicação explícita para fixar o transporte.

### Transporte Fixado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` identifica um sensor, e apenas esse sensor é aberto. É comparado com os sensores já abertos; em seguida, solicita-se ao nome de host ou IP de um DAQ-E (um DAQ-E que a varredura de rede deteta apenas pelo IP é consultado, em modo de leitura, quanto ao seu nome, MAC e número de série, pelo que qualquer um deles também o deteta), o MAC BLE de um DAQ-M e o ID que um DAQ-U ou DAQ-M reporta assim que é aberto (qualquer outro que seja aberto para verificação é fechado novamente). Se não for encontrado, o `connect_daq_sensor` gera o `ChlorosConnectError`, listando o que foi encontrado; incluindo também um endereço (`port=` / `mac=` / `eth_host=`) que contenha um sensor diferente, também é gerado. Nunca recorre ao sensor que estiver visível — é para isso que serve a chamada com o sinalizador zero.</id></sn></id></sn>

### `connect_daq_sensor()` Assinatura

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Um sensor que já se encontra no conjunto é devolvido tal como está (emprestado): os códigos `integration_time`, `frame_avg` e `enable_ae` não lhe são aplicados. Desligue-o primeiro para o abrir com as suas definições. O `cap_id` constitui a exceção: num sensor emprestado, é definido com o `set_cap()`. Se for omitido, aplica-se o valor predefinido do backend (ver a nota abaixo da tabela de métodos).

### Métodos `DAQSensorSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Resumo da entrada no pool (estado de transmissão/gravação, intervalo de comprimento de onda, sha de calibração, tempo de integração, frame_avg, estado de AE). Para um DAQ-E cuja transmissão ficou em silêncio: `reconnect` (onde se aplica a reconexão automática do backend: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` enquanto transmite), `stream_lost_s`, `reconnects` e `last_reconnect`, e `record_gaps` para a gravação em curso; `rx_stream_fault` indica o estado de reconexão até à chegada do primeiro espectro. |
| `latest(*, n=1, timeout=10.0)` | Devolve até N quadros de espectro mais recentes (cada um `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Retomar / pausar a transmissão (o identificador permanece aberto). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Iniciar a gravação de um ficheiro .daq. Devolve o caminho do ficheiro. Não é permitido para o DAQ-U/M sem um pacote de calibração AWS (o DAQ-E está isento). |
| `record_stop()` | Parar a gravação. Devolve `{path, rows, warning, gaps}`; `warning` corresponde a `None` no caso de uma gravação sem erros, caso contrário, o diagnóstico do backend (por exemplo, um ficheiro vazio ou uma lacuna no fluxo), que também é impresso em stderr. `gaps` lista cada intervalo em que o fluxo não forneceu nada: `start_utc`, `end_utc` (`None`: ainda perdido na paragem), `duration_s`, `reconnected`, `marked_in_file`. Fica vazio no caso de uma gravação contínua. |
| `set_cap(cap_id, timeout=15.0)` | Define o perfil de correção de limite aplicado às leituras a partir da próxima: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, ou `"none"` (ou `None`) para um sensor sem revestimento. Devolve o, cujo valor é lido de volta por `cap_id`. Um DAQ-E-S mantém o valor de `as_recorded` independentemente do que for solicitado (o seu difusor está integrado). Aumenta o valor de `ChlorosConnectError` para um perfil que não carrega. CLIm: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Apenas DAQ-E: reativa o PTP numa unidade cujo PTP esteja desativado (`status()["rx_ptp_enabled"]` é `False`; a configuração é guardada na unidade, pelo que permanece desativado até ser reativado). Retorna o estado atualizado. Lança o código de erro `ChlorosConnectError` para um DAQ-U / DAQ-M, ou quando a unidade não confirma. O Chloros nunca reativa uma unidade por si próprio, porque o estado desativado pode ser intencional. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Apenas DAQ-E: as próprias configurações da unidade. `firmware` (`running`, `bundled`, `update_available`), `ip` e `ip_mode` (`dhcp` / `linklocal` / `static`: o que está a ser executado neste momento), `network` (a configuração guardada: `mode`, mais `ip` / `mask` / `gateway` quando estático), `network_change_pending` (válido até a unidade executar a configuração guardada, uma configuração que inclui o `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` é igual a `None` quando a versão da unidadenão foi possível ler a versão da unidade. Enquanto o fluxo da unidade não chega a este computador, é lido numa nova ligação e o dicionário adiciona `stream_silent` (`True`), `stream_fault` e `reconnect`; uma unidade que não responde gera o erro CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E apenas: DHCP (`"auto"`) ou um endereço estático. Reinicia a unidade para aplicar a configuração e volta a ligá-la; esta sessão continua a funcionar e o `address` acompanha a unidade. O `reboot=False` apenas guarda a configuração. Consulte [Definições do dispositivo DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Apenas DAQ-E: transmite ao ligar, sem software ligado (guardado na unidade). Devolve a configuração confirmada. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Apenas DAQ-E: reinicia a unidade e volta a ligá-la com as definições que tinha. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Apenas DAQ-E: atualize o firmware incluído nesta versão do Chloros numa unidade que esteja a executar uma versão mais antiga e, em seguida, volte a ligá-la (demora cerca de um minuto; não a desligue). Retorna `updated`, `fw_before`, `fw_after`, `bundled` e um `warning` quando a unidade demorou a responder ou voltou a funcionar com outra versão; `updated` corresponde a `False` para uma unidade que já está atualizada. Gera o erro `ChlorosConnectError` para uma unidade cuja versão não pode ser lida. CLI: `pool-update-firmware`. |
| `disconnect()` | Libertação da reserva. Não-op para identificadores ligados que não são propriedade do utilizador. Num sensor aberto por esta sessão, é `force_disconnect()`, e gera o erro `ChlorosConnectError` quando a libertação não é confirmada (a saída do bloco `with` nunca gera uma exceção). |
| `force_disconnect(timeout=30.0)` | Liberta o sensor independentemente de quem o abriu — por exemplo, um sensor que um script que falhou deixou no conjunto, ao qual um novo script acede apenas como um identificador associado cujo `disconnect()` não faz nada. Encerrar o sensor de outro cliente interrompe qualquer registo que este esteja a efetuar. Retorna assim que o backend confirmar a libertação, ou quando não há nada para libertar (o sensor não está no conjunto ou nenhum backend está a ser executado); uma libertação que atingiu o tempo limite ou que foi recusada pelo backend gera o erro `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributos: `sensor_id`, `model`, `transport`, `address` (segue-se a um DAQ-E que o `set_network()` reendereçou), `info` e `backend_url`.

> **Identifique a tampa.** Os modelos DAQ-U, DAQ-M e DAQ-E são fornecidos sem condensador, e o Chloros não consegue detetar se está instalado algum: um sensor ligado sem o `cap_id` recebe o `sunshine_cosine`, uma suposição que os seus registos indicam terem feito (`cap_id_source=auto_default`). Um DAQ-U sem tampa, com essa correção, apresenta uma leitura cerca de 30 vezes superior ao normal; por isso, passe `cap_id="none"` para um sensor sem tampa (ou `set_cap("none")`), e o perfil da tampa que instalou de outra forma.

### DAQ-E-S — difusor fixo e sensor de inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças relevantes para o software. Tudo o resto — deteção, ligação, PTP, gravação e as definições do dispositivo abaixo — funciona exatamente como num DAQ-E.**O seu difusor é permanente, pelo que não requer correção da tampa.** O difusor do tipo Sunshine está integrado e já se encontrava na unidade quando foi realizada a sua calibração de fábrica, pelo que essa calibração já indica a irradiação descendente real. O Chloros define o seu limite para `as_recorded` (sem perfil), independentemente do que for solicitado: um limite com o seu nome (`sunshine_cosine`, `fov_*`, `none`) é ignorado com uma mensagem, e os seus registos armazenam `cap_id_source=model`, pelo que o aviso de limite presumido nunca é acionado para este dispositivo. Também não aplique manualmente um limite aos seus dados: o `sunshine_cosine`, por exemplo, apresentaria uma leitura cerca de 11 vezes superior ao normal.

**Este dispositivo possui um acelerómetro de 3 eixos.** A resposta cosseno do difusor de luz solar só é válida enquanto este estiver nivelado; por isso, um sensor inclinado reporta uma irradiância errada — e a refletância herda o erro — sem qualquer outro sintoma. Cada leitura do DAQ-E-S inclui, portanto, a sua atitude:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ângulos em graus. `None` quando o firmware marca o ângulo como indefinido (queda livre, um eixo inoperante). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleração em mili-g. `mag_mg` longe de ~1000 significa que a unidade estava em movimento e que os ângulos não são significativos para essa amostra. |
| `cal_applied` | Definido: os eixos e ângulos estão no sistema de referência do difusor (é aplicada a calibração de montagem da unidade). Desmarcado: sistema de referência bruto da placa, que pode apresentar até ~2,3° de inclinação que não é real. |
| `sample_age_ms` | Antiguidade da amostra do acelerómetro no momento em que o espectro foi enviado. |
| `temp_raw` | **Tendência** da temperatura do chip não calibrada — apenas diferenças, nunca uma leitura do termómetro. |

O `DAQSensorSession.latest()` devolve-as sob a chave `imu` de cada quadro (`None` num DAQ-U, DAQ-M ou DAQ-E simples). Um `.daq` registado a partir de um DAQ-E-S armazena os mesmos valores nas suas colunas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ao lado de cada espectro, para que o leitor consiga distinguir uma nuvem de um tripé que se tenha deslocado. Em todos os outros modelos, essas colunas estão vazias.

O fluxo de atitude apresenta os seus próprios campos em `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), separados dos campos espectrais para que uma falha do acelerómetro nunca seja interpretada como uma lacuna no registo. O `discover_daq_sensors()` e o `model` da sessão indicam o `DAQ-E-S`.

### Definições do dispositivo DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Deve existir um endereço estático na rede deste computador.** O `set_network` gera o erro `ChlorosConnectError` para quem a sub-rede não inclua o endereço deste computadorna rede do sensor: após o reinício, nada consegue aceder ao sensor para reverter a alteração. Aplique o `force=True` apenas quando este computador for reendereçado ou encaminhado para a nova sub-rede.
- **`set_network`, `reboot` e `update_firmware` reiniciam o sensor** e geram o `ChlorosConnectError` enquanto este grava, e quando não regressar no prazo de `comeback_timeout` segundos. O ID do sensor e o nome de anfitrião `daq-e-<id>.local` nunca se alteram.
- **Uma gravação iniciada imediatamente antes do reinício é encerrada pelo próprio reinício e recebe um nome.** Quando o sensor volta a funcionar, o dicionário devolvido contém `recording_closed` (o caminho do ficheiro) e um `warning` a indicar isso. Quando não volta, ou se a transferência do firmware falhar, o `ChlorosConnectError` indica isso em vez disso: a sua mensagem termina com o aviso, e os valores `e.recording_closed` / `e.warning` são definidos (ambos o `None` quando nenhuma gravação foi encerrada). Reinicie a gravação assim que o sensor estiver de volta.
- **Apenas o próprio sensor é reconectado.** Após o reinício, a unidade deve responder onde se encontrava — e, após o `set_network("auto")`, também sob o seu nome `.local` — com o seu nome, nome de anfitrião e modelo, a partir do adaptador de rede (MAC) que possuía; `address` torna-se o IP que respondeu. Uma resposta sob o nome só conta com esse MAC; assim, a partir de um computador que acede ao sensor através de um router, ou a partir da rede NAT predefinida do WSL2, um sensor que tenha sido movido pelo DHCP não é reconectado (`ChlorosConnectError`; volte a ligá-lo no seu novo endereço).
- **Uma atualização de firmware requer que o sensor volte a ligar-se.** `update_firmware()` convida a unidade, e esta abre uma ligação TCP de volta a este computador para a imagem, numa porta escolhida pelo SO. O backend escuta no adaptador que acede à unidade (em todos os adaptadores, apenas quando não consegue identificar qual) e atende apenas essa unidade: qualquer outra ligação é recusada (`[DAQ-FW] … refused a firmware connection from <ip>` na saída do backend) enquanto este continua à espera. Uma firewall que bloqueie a ligação faz com que a atualização falhe com o código `device never connected back on TCP <port>`. O instalador do Windows adiciona a regra necessária (ver [Passo 1](#windows-exe)); noutro firewall, permita o tráfego TCP de entrada para o backend a partir da sub-rede do sensor.
- **No WSL2, utilize a rede espelhada** (`networkingMode=mirrored` em `.wslconfig`): no modo NAT predefinido, todos os endereços estáticos são recusados (o próprio endereço do WSL não está na rede do sensor), o fluxo não chega ao WSL e uma atualização de firmware não pode ser concluída.
- A GUI mantém a sua própria ligação a um sensor: um reinício a partir de um script encerra a sessão da GUI com o sensor. A GUI tem as mesmas definições (separador «Sensores de Luz», definições do sensor, «Rede»).

### Descoberta — encontrar um endereço para estabelecer ligação

O `discover_daq_sensors()` procura em USB / BLE / ETH por sensores que *possa* abrir. É a contraparte DAQ do `discover_lattice_cameras()` e a única forma de obter o **MAC BLE de um DAQ-M** — um DAQ-E tem um nome de anfitrião e um DAQ-U uma porta COM, mas o MAC não está impresso no dispositivo nem é listado pelo sistema operativo.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrição |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome do anfitrião — passar para `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Etiqueta legível por humanos. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, ou `None`. Um sensor Ethernet é o `DAQ-E-S` ou o `DAQ-E`, tal como se identifica quando solicitado, em modo de leitura apenas, durante a verificação (`DAQ-E` quando não responde a tempo). Uma porta série USB apresenta o valor `DAQ-U` quando o seu chip de ponte é um dos utilizados pelo DAQ-U (FTDI, CH340, CP210x, Prolific) — uma indicação da descrição da porta de que não uma sonda — e `None` para qualquer outro dispositivo USB, que continua listado, pelo que pode ser passado como `port=`. Uma porta série sem ID de fornecedor USB e sem esse chip de ponte (um UART da placa-mãe, como o `/dev/ttyS*` do Linux, uma ligação série Bluetooth, Intel AMT) não pode ser um DAQ-U e não está listada. |
| `extra` | Detalhes por transporte (nome anunciado do BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Os valores vazios são omitidos. |

| Parâmetro | Padrão | Descrição |
| --- | --- | --- |
| `transports` | os três | Sequência (ou cadeia de caracteres CSV) que limita a varredura. Vale a pena especificar quando souber o que pretende — o BLE é a parte mais lenta. |
| `scan_timeout` | 5 | Janela de varredura por transporte em segundos; o backend limita o valor a 1–20. |
| `timeout` | 60,0 | Limite máximo de HTTP para toda a chamada (tal como noutras partes do SDK). |
| `auto_start_backend` | `True` | Inicia um backend local se nenhum estiver a ser executado. Nunca é iniciado para um `backend_url` remoto. |

> **Os sensores já abertos na piscina não aparecem.** Um periférico BLE ligado deixa de anunciar e uma porta COM aberta não pode ser detetada, pelo que a descoberta lista o que está *disponível para ligação*. É esperado um resultado vazio logo após ter ligado algo — utilize o `list_daq_sensors()` para o que já possui. Os transportes cuja verificação não pode ser executada (sem o bleak / zeroconf instalado) são ignorados em vez de gerarem um erro, pelo que um computador sem Bluetooth continua a obter as respostas USB e ETH.

### Lista

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Qual DAQ alimenta qual câmara — `daq_bindings()`

A refletância de uma captura e o `.daq` que esta guarda utilizam o DAQ associado a essa câmara ou matriz; caso contrário, utilizam o DAQ mais recente que o backend tiver. A seleção do sensor de luz na GUI e o `chloros-cli daq pool-bind` definem-no; assim como:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` devolve `{"cameras": {serial: b}, "arrays": {array_id: b}}`: cada `b` contém `binding` (o valor definido), `source` (`project`, do `sensors.json` do projeto, ou `runtime`), `known` / `known_as` (se a chave da câmara ou da matriz designa algo que um Chloros conhece, e de que forma: `connected`, `project` — o `cameras.json` do projeto aberto — ou, no caso de uma câmara, `seen`: já aberto neste computador anteriormente; `known: False` para uma chave que não designa nada) e `resolved` (o sensor de pool ao qual está agora associado: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Uma ligação em tempo de execução prevalece sobre as do projetodurante a sessão; o ficheiro do projeto não é alterado. Desligar uma câmara, um conjunto ou um DAQ anula todas as ligações desse hardware, independentemente de quem as tenha definido (as dos membros de um conjunto são ligadas ao próprio conjunto).

`set_daq_bindings()` vincula apenas uma câmara que esteja ligada, no projeto aberto ou que tenha sido aberta anteriormente neste computador, e uma matriz que esteja ligada ou no projeto aberto (ou uma tecla já vinculada). Qualquer outra tecla gera o erro `ChlorosConnectError`, indicando (`400`, `code`, `UNKNOWN_BINDING_TARGET`, com as teclas `unknown`), e **nada** nessa chamada é associado. `None` limpa qualquer chave.

### Refletância ambiente + objeto — `light_sensor_reflectance()`

Dois DAQs agrupados, um a olhar para o céu (ambiente) e outro para o alvo (objeto), fornecem a refletância do alvo e os índices de vegetação a partir das suas últimas leituras: o par Ambiente + Objeto da GUI e o `chloros-cli daq reflectance` sem abrir os sensores uma segunda vez. Ambos têm de estar ligados e a transmitir dados.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bandas.** A refletância de cada banda é a irradiância integrada da bandasobre a do ambiente. O `daq_band_presets(filter_code=None)` lista os conjuntos de bandas: um por filtro LATTICE M3C (`RGN`, `OCN`, `NGB`, o modelo de bandas próprio da câmara, pelo que um NDVI do DAQ e um NDVI da câmara correspondem à mesma grandeza) e a banda larga `GENERIC`. O seu `suggested` é a predefinição correspondente a `filter_code`; caso contrário, é o filtro da primeira câmara ligada (`suggested_from` indica qual).
- **Retorna** `bands` (por banda `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (os integrados), `custom` (cada um com `value`, ou `valid: False` e o `error`), `invalid_bands`, `reflectance_type`, `preset` e as duas leituras de `timestamps` (`ts_ns`). Um intervalo que o sensor ambiental não conseguiu medir (fora do seu intervalo de medição, no seu nível de ruído de fundo) é considerado inválido, com indicação do motivo, e todos os índices que dele dependem são excluídos, nunca sendo calculados com base no valor zero.
- **Aparente ou relativo.** Os índices dependentes da escala (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) necessitam de uma relação absoluta, pelo que só são calculados quando ambos os sensores estão calibrados em potência (cada um tem um `calibration_bundle_sha`): `"Apparent Reflectance"`. Caso contrário, o resultado é `"Relative Reflectance"`, apenas com os índices invariantes em relação à escala.
- **Duas grelhas de comprimentos de onda diferentes** são comparadas quanto à sua sobreposição: os pontos do sensor ambiente dentro do alcance do objeto, com o espectro do objeto interpolado sobre eles.
- **Dois sensores, nunca um só.** Atribuir o mesmo nome a um sensor (o mesmo ID, em qualquer grafia que o pool aceite) gera o erro `ChlorosConnectError`: a leitura de um sensor sobre si próprio é 1,0 em todas as bandas, o que não mede nada. O backend também recusa isso (`POST /api/daq/indices` responde a `400`, `code` e `SAME_SENSOR`, quando o `ambient_sensor_id` e o `object_sensor_id` enviados pelo SDK correspondem a um único sensor agrupado, por exemplo, o seu ID e a sua porta).

### Partilha com a CLI e outros scripts

O conjunto de sensores DAQ do backend é partilhado por todos os clientes CLI e SDK na máquina. Se um sensor já estiver no conjunto — aberto por `chloros-cli daq pool-connect` ou por outro script —, chamar `connect_daq_sensor(port="COM3")` a partir de Python devolve um identificador emprestado: o seu `disconnect()` é uma operação nula, pelo que o seu script não retira o sensor ao cliente que o abriu ao sair do âmbito. O mesmo se aplica a `sensor_id=` (digitado com `:` ou `-`, qualquer um dos dois serve), para um DAQ-E identificado pelo seu IP quando foi aberto pelo nome de host (ou vice-versa), e para a deteção inteligente com o sinalizador zero quando o backend responde com um sensor que já detinha — um bloco `with` só liberta um sensor que ele próprio tenha aberto.

A GUI do Chloros **não** faz parte deste conjunto: o seu painel DAQ estabelece a sua própria ligação ao sensor, pelo que o `connect_daq_sensor` nunca utiliza um sensor que a GUI tenha aberto. Não utilize o mesmo sensor a partir da GUI e de um script ao mesmo tempo. Um DAQ-U ou DAQ-M que a GUI mantém normalmente simplesmente não consegue abrir, mas um DAQ-E aceita uma segunda sessão: a ligação do script altera as definições de aquisição da unidade na GUI, e o seu `disconnect()` interrompe o fluxo da unidade, o que encerra a visualização em direto da GUI e qualquer gravação que esta esteja a efetuar. Feche primeiro o sensor na GUI.

### PTP Grandmaster — `time_sync_status()`

O backend executa o PTP Grandmaster do qual as câmaras LATTICE e as unidades DAQ-E obtêm os seus carimbos de data/hora. Inicia-se a pedido: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` e `connect_all()` iniciam-no caso de não estar a funcionar — incluindo uma ligação que falhe posteriormente, como o `connect_all()` ao tentar ligar-se a um DAQ-E que não existe — e o mesmo acontece com o `test_array_sync()` e todas as chamadas que abrem câmaras para um comando (`capture_cameras_transient()`, `start_multi_camera_stream()`, as chamadas de alinhamento, `capture_live_index()`).

- **Onde é anunciado.** Nos adaptadores de rede com fios que estão ligados às câmaras (incluindo um endereço link-local `169.254.x.x`), ou então no adaptador da rota predefinida, caso este seja com fios. Nunca em Wi-Fi ou num adaptador virtual (Tailscale, WSL): não há nenhuma câmara nem DAQ-E ligados a um desses. Se não houver nenhum adaptador desse tipo ativo, o programa continua a anunciar em nada (`health` `NO_SOCKETS`) e vincula-se ao adaptador assim que a próxima câmara ou DAQ se ligar.
- **Após uma queda de ligação.** Quando o adaptador da câmara perde a ligação e a recupera (ao puxar um cabo, substituir um switch ou reenumerar um adaptador USB), o «grandmaster» reconstrói os seus sockets automaticamente, sem necessidade de reinício, e verifica novamente todos os DAQ-E ligados, reativando aquele que ficou bloqueado devido à falha de ligação. Também se reconstrói quando todas as câmaras e DAQ-E que estava a servir ficam inativos durante 15 s.
- **Está a chegar ao cabo?** Leia `time_sync_status()` (`GET /api/time-sync/status`). O `state` apenas indica que o thread do «grandmaster» está a funcionar; o `health` indica se os seus pacotes são enviados: `OK` (nenhuma falha detetada), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (algumas placas com falha) ou `PEERS_SILENT` (as câmaras e os DAQ-Es que estava a servir deixaram de responder), com uma mensagem em `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` contam apenas as mensagens que foram enviadas; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` e `last_delay_req_age_s` descrevem as que não foram enviadas, e `rebinds` / `last_rebind` as reconstruções. O `sync_sends_slow` conta as sincronizações cujo envio demorou mais de 200 us e o `sync_send_max_us` é o mais demorado; o tempo de origem de cada Follow_Up conta, no máximo, 200 us do seu envio. O grandmaster é executado num processo próprio (`own_process`, `process_pid`), com prioridade elevada no Windows (`process_priority`), pelo que um backend ocupado não pode atrasar os tempos que marca nem ajustar o relógio de todas as câmaras de uma só vez; se esse processo não puder ser iniciado, é executado dentro do backend como anteriormente (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Função | O que faz |
| --- | --- |
| `time_sync_status()` | O estado do «grandmaster», a sua integridade, a identidade do relógio, as prioridades e os contadores de mensagens (acima). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Os escravos PTP que deteta (câmaras, DAQ-Es): `peers`, cada um dos `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Cada câmara de matriz ligada: `PtpStatus`, `PtpOffsetFromMaster` e `PtpMeanPathDelay` (uma câmara cujo firmware não tenha nós PTP apresenta apenas o número de série). `time-sync cameras`. |
| `restart_time_sync()` | Parar e reiniciar o grandmaster; o PTP de cada DAQ-E agrupado é verificado novamente após (`daq_ptp_rechecked`). Lança `ChlorosConnectError` quando não voltar a funcionar. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | As suas prioridades PTP (0-255; o valor mais baixo vence a eleição do melhor mestre), para uma rede com outro mestre PTP. Requer que este esteja em execução. `time-sync set-priority`. |

Cada um recebe o `backend_url=` e o `timeout=` e inicia um backend local que não está a funcionar.

---

## Utilitários da câmara

O que o `chloros-cli lattice` faz para além de ligar e capturar, como funções simples no `chloros_sdk` (módulo `chloros_sdk.camera_utils`). Cada um chama a rota do backend que o comando CLI utiliza, com as regras da CLI, pelo que as duas se comportam de forma semelhante. Aceitam um **número de série** da câmara, e não uma sessão, e as mesmas palavras-chave `backend_url=` / `timeout=` que as chamadas smart-connect; um backend que não esteja a funcionar é iniciado, tal como o `connect_camera()` faz. Uma recusa gera o `ChlorosConnectError` com o motivo indicado pelo backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Função | O que faz |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | O perfil de cor do ecrã: `raw`, `linear`, `natural`, `enhanced` ou `custom_temp`, e a sua temperatura de cor (limitada a 2000-10000 K; lida quando o perfil é `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Saturação e contraste do ecrã, 0-200 % cada (100 = inalterado); `reset=True` repõe ambos para 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Reduz a luminosidade dos píxeis mais brilhantes e mais brancos da imagem apresentada: -100..0 (truncado para zero e, em seguida, limitado; 0 = desligado). |
| `camera_white_balance_once(serial, settle_s=None)` | Equilíbrio de brancos de uma única medição: a câmara transmite brevemente, converge e devolve `red_ratio` / `blue_ratio` (`ratios_read` False quando não consegue reportá-los). Mantém o equilíbrio de brancos que calculou; todas as outras definições que o seu «open» escreveu são devolvidas tal como foram encontradas. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (contínuo) ou `"off"`; ou relações de equilíbrio fixas `red` / `blue`, que desativam primeiro o modo automático. Uma relação que a câmara recusa é designada por `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | As predefinições da câmara e a aplicação de uma delas: a câmara mantém as definições indicadas nos nomes das predefinições (a resposta é `settings`, tal como a câmara os lê posteriormente, por exemplo, a taxa de fotogramas a que funciona; `None` para um campo que a predefinição não identifica), e todas as outras definições que o ficheiro de configuração abriu voltam ao estado em que foram encontradas. `"default"` repõe a câmara para os valores predefinidos de fábrica (as suas definições de fábrica com exposição automática livre, o limite de débito da ligação e a correção de defeitos desativados, mantendo-se o equilíbrio de brancos calibrado de uma câmara RGB), sem que sejam guardadas na sua memória: a resposta contém `factory_defaults: True` e todos os campos de `settings`. Em `camera_presets()`, `default` é igual a `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Os modelos de filtro e os detalhes de um modelo (bandas, número de condição da resposta espectral). O modelo de filtro de uma câmara é lido a partir da sua cadeia de caracteres do modelo; não é definido manualmente. |
| `camera_firmware_status(serial=None)` | O firmware de cada câmara: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` e `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Instale o firmware desta versão do Chloros numa câmara: reposição de fábrica, instalação, reinicialização; as definições de base do Chloros são reescritas; a calibração da câmara é mantida. Demora vários minutos; a câmara deve permanecer ligada. O `force=True` volta a instalar o firmware numa câmara já existente. |
| `camera_power_probe(serial)` | Os nós de alimentação e térmicos da câmara e a sua temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | O modo de inatividade de baixo consumo da câmara (permanece em modo de suspensão até ser ativada: o limite de largura de banda da ligação ativado (`DeviceLinkThroughputLimitMode` `On`) no seu valor mínimo, o modo de poupança de energia ativado, caso a câmara o possua), com a temperatura antes e depois; o despertar restabelece a ligação tal como estava quando entrou em modo de suspensão (o limite, o seu modo e o atraso de pacotes `GevSCPD`, que a câmara mantém em 0 enquanto o limite está ativado; o registo é mantido neste computador, pelo que um despertar por parte de um backend posterior também o encontra), ou, no caso de uma câmara que este computador não colocou em modo de suspensão, mas que parece estar em suspensão (com o limite ativado no valor mínimo), com o limite desativado e um `GevSCPD` mantido em 0 a 80 (as definições de fábrica); um limite definido acima do valor mínimo é mantido tal como está. Apenas o estado de alimentação é mantido; todas as outras definições cujos valores foram alterados são repostas tal como se encontravam. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Capturar fotogramas até que um alvo de calibração MAPIR esteja visível e ajustar os coeficientes por canal a partir dos seus painéis: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Adaptadores Ethernet deste computador para GigE Vision (MTU, velocidade de ligação, quadros jumbo) e quaisquer `issues`; `estimate=True` adiciona a taxa de fotogramas e a largura de banda que as câmaras `cameras` obteriam. |
| `fix_network_settings()` | Aplica as definições do adaptador GigE Vision ao. Altera as definições do adaptador e só funciona quando o Chloros é executado como Administrador; caso contrário, apresenta um erro com o registo do que encontrou. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Corrija a vinheta das imagens numa pasta, cada uma com o mapa de campo plano da sua própria lente: `processed`, `lens_models` (a lente para a qual cada ficheiro foi corrigido), `skipped` (com justificações). Sem o `lens_model`, a lente de cada ficheiro é determinada a partir do seu Modelo EXIF; um ficheiro `lens_model` aplica-se a ficheiros cujo Modelo não especifica nenhuma câmara, e um ficheiro cujo Modelo o contradiga é ignorado. Apenas ficheiros. |
| `read_dls(mac, filter=None, camera_serial=None)` | Uma leitura de um DLS (sensor de luz descendente Bluetooth): espectro, irradiação por banda e magnitudes de banda através da própria resposta da câmara quando é fornecido o `camera_serial`. O `discover_daq_sensors(transports=["ble"])` identifica o MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | O hardware do anfitrião para câmaras e a configuração que este recomenda (`report_text`), bem como a CPU / GPU / tempo de execução do PyTorch do sistema. |

- **Algumas precisam da câmara só para si.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` e `update_camera_firmware()` abrem a câmara. Enquanto o Chloros a mantém ocupada (ligada na aplicação, com o `connect_camera()`, como membro de um conjunto, ou a ser utilizada por outro comando neste momento, como um teste de matriz ou uma captura que abre as suas câmaras), o backend recusa com o código HTTP 409 `CAMERA_HELD` (tanto no `code` como no `error_code`), indicando como o libertar. As definições de cor, realces, `camera_power_probe()` e as leituras funcionam numa câmara ligada.
- **A cor, os realces e o equilíbrio de brancos destinam-se a câmaras RGB.** Numa câmara monocromática de banda única (M3M), os setter não alteram nada e devolvem `{"skipped": True, "reason": ...}`, tal como o CLI. Trata-se de definições de visualização: a visualização em direto e as capturas ao nível do ecrã, nunca a radiação ou a refletância.

---

## Matrizes transitórias, perfis de alinhamento e índices

O que o `chloros-cli lattice` faz sem uma ligação persistente, tal como as funções no `chloros_sdk` (módulo `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` e `index`. Cada um envia o pedido que o comando da CLI envia, com os valores predefinidos da CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Eles próprios abrem as câmaras.** Cada comando abre as suas câmaras, executa a sua tarefa e fecha-as, pelo que necessita de câmaras que o Chloros não esteja a controlar (ligadas na aplicação, com o `connect_camera()` ou como membro de uma matriz); o backend recusa com o código HTTP 409 `CAMERA_HELD` e indica como libertar uma. Uma abertura transitória demora algum tempo (cerca de 40 s para algumas câmaras). Para capturas repetidas, `connect_array()` uma única vez é a opção mais rápida e poupa em todos os tipos de exportação, enquanto uma captura transitória guarda uma imagem sem debayering. `calibrate_alignment_profile()` é a exceção: numa matriz ligada dessas câmaras, resolve a partir dos próprios fotogramas da matriz (`used_live_array`).
- **`serials`** são as câmaras a abrir, sendo a primeira a acionar o disparador; `None` abre todas as câmaras na rede.
- **`settings`** é um dicionário de substituições de configurações da câmara, os campos definidos pelos sinalizadores da CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` e `preset` (uma predefinição de câmara com nome). Um valor de exposição ou ganho especificado desativa esse controlo automático, tal como acontece com os sinalizadores.
- **`gpu`**: `True` força o warp na GPU, `False` na CPU (`--gpu` / `--no-gpu`); o `None` permite que o backend escolha.
- **`keep_settings`**/**`clear_settings`** (todas as funções que abrem câmaras, por predefinição `False`): no final, cada câmara é reposto no estado em que foi encontrada; `keep_settings=True` guarda as definições da execução na própria memória da câmara (funcionamento autónomo), `clear_settings=True` repõe essa memória para as definições de fábrica — ver [Memória da câmara](#camera-memory--keep_settings--clear_settings). A resposta (para uma tarefa, a do `stop()`) contém o `camera_memory`.
- **Os caminhos são os do backend**: os relativos tornam-se absolutos em relação ao seu diretório de trabalho quando o backend se encontra nesta máquina. As funções do ficheiro de perfil de alinhamento (`alignment_profile_info()`, `reorder_alignment_profile()`) lêem o ficheiro neste computador, tal como a CLI.

| Função | O que faz |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Grava uma câmara continuamente até ser interrompida. Devolve um `CliJob`. Os fotogramas de uma execução `tiff` constituem o ficheiro `raw`, uma matriz de capturas dessa câmara (um plano de mosaico com ospróprios valores do sensor, não comprimidos); `png` e `jpg` contêm 8 bits, pelo que essas execuções são fotogramas desmosaicados para visualização. Um DAQ vinculado ou agrupado grava um `.daq` por fotograma napasta `daq/` da série (`daq_files` nas suas estatísticas). Cada fotograma é gravado uma vez com os seus metadados completos (modelo, hora da captura, lente, bandas, calibração num fotograma bruto); o `stop()` é devolvido assim que todos os fotogramas já capturados forem gravados (até 25 s), e só então a câmara é desligada. Uma câmara deixada em modo de espera para um disparador de hardware (`TriggerMode` `On`) é configurada para funcionamento livre durante a sessão (a resposta de início, `trigger`, indica o que foi encontrado). Na paragem, a câmara é reposto no estado em que foi encontrada — as suas definições de disparo e todas as definições alteradas pela execução —, a menos que se utilize o `keep_settings=True`, que guarda as definições da execução na própria memória da câmara (funcionamento livre), ou o `clear_settings=True`, que reinicia essa memória. Uma execução que não registe nenhum fotograma termina automaticamente após 10 s com o `stop_reason` e o `no_frames`, e o motivo é indicado no `last_error`, e o seu `stop()` devolve esse motivo como `error`. Os fotogramas perdidos porque os gravadores não conseguiram acompanhar o ritmo são contabilizados, com o `drop_reason` a indicar o motivo. Para uma câmara que a Chloros mantém, utilize o `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Transmite grupos de fotogramas sincronizados no `fps`, para grupos `count` ou até ser interrompido. Devolve um `CliJob` (o seu `start_reply` contém `n_cams` e o fluxo `pacing`); as suas estatísticas finais contam os grupos capturados, gravados e descartados, os ficheiros que o gravador não conseguiu gravar e os grupos para os quais uma câmara não enviou nada. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Uma captura sincronizada, uma imagem sem efeito «debayering» por câmara: `results` (`filepath`, `serial`, `frame_id`) e `pacing`. Um grupo em que falta uma câmara é destacado, indicando primeiro quantos ficheiros foram guardados. `lattice multi-capture` sem matriz ligada. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` capturas sincronizadas com uma exposição fixa: `sync_groups`, o `master`, uma linha `group_timestamps` por disparo (uma câmara que falhou um disparo está ausente da sua linha) e `sync`: o desfasamento de cada duas câmaras em relação à câmara principal, disparo a disparo (`offsets_us`: por câmara `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), num ÚNICO relógio (`clock_domain`: `ptp`, ou `device` — PTP desativado, o relógio de tempo de funcionamento de cada câmara, pelo que o seu desvio corresponde ao momento em que foi inicializada e apenas o jitter mede a sincronização). Um disparador cujos carimbos misturam o relógio do dispositivo e o PTP (PTP desativado ou ainda a sincronizar) é excluído e contabilizado (`groups_mixed_domain`), tal como aquele em que um relógio PTP ainda está a mais de `agree_within_us` do relógio mestre (`groups_clocks_disagree`); o `notes` confirma isso. O `clock_offset_mean_us` / `clock_offset_jitter_us` representam a média e o jitter máximos por câmara. `success` é falso, tal como `error`, quando um disparador foi perdido, quando menos de dois disparadores têm todas as câmaras num único relógio, ou quando os relógios nunca coincidem (`sync.problems`); as linhas são devolvidas em qualquer dos casos. Inicia o grandmaster PTP se este não estiver a funcionar (`time_sync` indica que `grandmaster_running` e `grandmaster_started_by_this_test`), ativa o PTP de todas as câmaras (tal como faz o `connect_array()`) e, antes do primeiro disparador, aguarda que o relógio PTP de todas as câmaras se sincronize com o «grandmaster» e estabilize: `PtpStatus` «Slave» num «grandmaster», com um desvio de até 100 us em relação a este e estável durante 3 s, e cada câmara sincronizada dessa forma, sem interrupção, durante 120 s — cerca de 2 minutos após o arranque. Um relógio que acabou de se sincronizar apresenta um desvio de até ~200 us durante cerca de 2 minutos após indicar «Slave», e o próprio `PtpOffsetFromMaster` da câmara não o mostra; 120 s fazem com que as câmaras fiquem a cerca de ~10 us de um relógio estabilizado. Uma câmara que ainda esteja sincronizada desde um teste anterior que deixou o PTP ativado, tal como este backend o detetou, conta a partir dessa sincronização, pelo que um teste realizado imediatamente a seguir a outro não tem de esperar novamente; uma câmara que o teste liga, ou uma encontrada com o PTP ativado que este backend ainda não tenha visto sincronizada (ligada noutro local, ou antes de um reinício do backend), aguarda os seus 120 s, e num conjunto de ambas as câmaras que acabaram de se sincronizar, estas continuam a aguardar o seu tempo. Uma câmara que sai do modo Escravo, ultrapassa os 100 us ou segue outro «grandmaster» durante 3 leituras consecutivas (1,5 s) durante a espera, reinicia a sua contagem; uma ou duas dessas leituras são registadas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e a contagem prossegue, sendo que um ID de «grandmaster» que não possa ser lido é considerado desconhecido, e não como uma alteração. O reinício simultâneo da contagem de todas as câmaras (no espaço de um segundo) deve-se à variação do tempo do «grandmaster», e não ao das câmaras: o registo do backendregisto indica `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, o `ptp_lock` lista-o no `grandmaster_steps`, e uma espera que se esgota a seguir inicia o seu erro com `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. O limite não é alargado para este caso. `ptp_settle_s` (0-600; `0` para ausência de espera de estabilização) define o tempo de estabilização para a execução; qualquer outro valor é recusado (código HTTP 400). A espera está limitada ao tempo de estabilização mais 90 s (210 s por predefinição, no mínimo 90 s), e a chamada aguarda o backend `timeout` s: por predefinição, 240 mais o tempo de estabilização (360 s), tal como acontece com o `test_array_sync` do `chloros-mcp`. O `ptp_lock` tem o `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (há quanto tempo cada câmara está bloqueada e estável em conjunto), `found_settled` (as câmaras já estavam bloqueadas durante o tempo de estabilização quando foram detetadas) e, por câmara, `status`, `offsets_ns`, `slave_s` (há quanto tempo está sincronizada; «Nenhuma»: não está sincronizada) e `remaining_s` («Nenhuma» quando o PTP de nenhuma câmara se ativou). O registo do backend apresenta uma linha a cada 5 s com o estado de cada câmara, há quanto tempo está sincronizada e quanto tempo ainda falta. Todas as câmaras ligam-se com o PTP desativado; a partir daí, os relógios sincronizam-se em cerca de 10 s. Posteriormente, cada câmara é reposto no estado em que o teste a encontrou — o seu PTP, a exposição automática e o ganho que o teste mantém desativados durante a sua execução (`ExposureAuto` / `GainAuto` e um modo manual `ExposureTime` / `Gain`), e as restantes, a menos que seja o caso de `keep_settings=True` — ou seja, uma câmara que tenha ficado com o PTP ativado (`PtpEnable=True`, ou uma execução com `keep_settings=True`) permanece bloqueada, e assim que este backend verificar que está bloqueada durante o tempo de estabilização, um teste posterior não espera por ela. Se os relógios não se estabilizarem, o `success` é falso antes de qualquer acionamento, e o `error` indica o que fazer: uma câmara que nunca ficou bloqueada (`not_slave`) significa que o «grandmaster» não está a alcançá-la (`time_sync_status()`); uma que continuou a perder a ligação é identificada pelo tempo durante o qual esteve bloqueada (`remaining_s`); relógios ainda a convergir (`far`): verifique `time_sync_status()` e, para que fiquem sincronizados antes do teste, ative o PTP e mantenha-o ativado (`PtpEnable=True` em cada câmara, ou `keep_settings=True`; o teste repõe o que encontrou, pelo que permanece ativado), depois execute-o. `lattice multi-test`. |
| `detect_array_master(line=None)` | As câmaras na linha de sincronização e um mestre predefinido: `master_serial`, `slave_serials`, o `line` foi testado. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Que variedade de modelos destas câmaras (nomes de produtos, `"LATT-M3C-L87-FOCN"`) que apareceria na rede deste computador: `tier`, `achievable_fps_max`, o orçamento de cabos (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` e `fix_suggestions`. Por predefinição, os modelos das câmaras detetadas; nenhuma câmara está aberta. Para as câmaras de que dispõe, com as definições que uma ligação utilizaria, consulte [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Resolver um perfil de alinhamento a partir de fotogramas em tempo real e guardá-lo. `method`: `orb`, `akaze`, `phase`, `checkerboard` (com o `checkerboard="9x6"`, os cantos internos) ou o `manual` (com o `control_points={serial: [[x, y], ...]}`, as câmaras de referência também). `model`: `translation`, `rigid`, `affine` ou `homography`. As opções mais específicas são os sinalizadores da CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` é igual a `{serial: {option: value}}`. `rms_threshold_px` recusa-se a guardar uma solução pior. Retorna `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (por câmara `reprojection_error`, `num_matches`) e `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Uma imagem multibanda alinhada através do perfil: `files` (o ficheiro TIFF multibanda com o seu `bands` e tamanho, mais um por banda com `per_band` e um por câmara com `per_camera`) e `vignette_corrected`. O `bands` seleciona e o `order` ordena as bandas por nome (uma lista ou uma sequência separada por vírgulas). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Imagens multibanda alinhadas por fluxo (as mesmas opções de warping que o `capture_aligned_multiband()`), para o `count` ou até ser interrompido. Devolve um `CliJob`; as suas estatísticas finais incluem o `frames_written` e qualquer `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Um ficheiro de perfil resumido como `align-info` imprime-o: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (por câmara `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (por câmara: `band_names`, `wavelengths_nm`, `enabled`); o `profile` é o ficheiro completo. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Alterar o ficheiro de perfil no local: `order` são nomes de bandas (cada uma move a camada da sua câmara para cima, nessa ordem), `enable` / `disable` são números de série. Devolve o novo `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | As predefinições de índice (`presets`: cada `name`, `channels`, `formula`) e de cor `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Um índice sobre um ficheiro TIFF multibanda alinhado: um `preset` (`NDVI`, `NDRE`, ...) ou um `formula` sobre os símbolos `channel_map` mapeia para bandas (`{"NIR": "NIR_850", "Red": 0}`, por nome ou índice). Uma predefinição&#x27;símbolo `channel_map` exclui a única banda do ficheiro cujo nome começa por ele (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), nomeada pelo próprio ficheiro (a sua lista de bandas ou o `Chloros:Band{R,G,B}Name` de uma captura ou produto Chloros), pelo que o NDVI numa captura de refletância do Chloros não necessita de `channel_map`; duas bandas correspondentes a um símbolo são rejeitadas, não inferidas. O `capture_level` indica o que as bandas contêm: o `reflectance` ou o `radiance` tornam os valores quantitativos; uma pilha não calibrada (`align-apply`) fornece valores qualitativos; `"unknown"` lê-o a partir do ficheiro, incluindo o `Chloros:ProcessingLevel` de um ficheiro Chloros. Devolve o `stats` (forma, o intervalo do 1.º ao 99.º percentil, o intervalo da LUT, o `capture_level` utilizado) e o `files` escrito ao lado da entrada (`<input stem>_<preset>.tif`, `..._LUT.png`) ou em `output`, um caminho: com uma extensão de imagem, os ficheiros são nomeados a partir do seu radical (`output="D:/out/ndvi_FRGN.png"` escreve `ndvi_FRGN.tif` e `ndvi_FRGN_LUT.png`); sem ela, trata-se de um prefixo (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); leia os caminhos a partir de `files`. Estes contêm os registos da câmara de entrada (marca/modelo/número de série, GPS, lente, hora de captura em UTC, a identidade de calibração, mas nunca a sua calibração raw-DN), `Chloros:ProcessingLevel=index`, e o índice como a sua única banda (`XMP-Camera:BandName`, o TIFF e o PNG da LUT da mesma forma). Apenas ficheiros; sem câmara. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Captura, calibra (`reflectance` ou `radiance`; o `raw` ignora esta etapa), alinhar e calcular um índice numa única etapa; o `save_multiband=True` mantém a pilha calibrada, que o `compute_index()` pode utilizar posteriormente. Devolve o `stats` (com o `serials` cujas bandas utilizou) e o `files`. Abre o `serials` (o primeiro a disparar o gatilho), caso contrário, as câmaras do perfil cuja entrada `layer_order` esteja ativada, começando pela de referência (aberta mesmo com as suas bandas desativadas); uma câmara cujas bandas estejam desativadas nunca é calibrada. O `channel_map` é necessário apenas para um símbolo ao qual a pilha não consegue responder, como em `compute_index()`: o símbolo de uma predefinição utiliza a única banda ativada cujo nome comece por esse símbolo, verificada antes de qualquer câmara ser aberta (se nenhuma ou várias responderem, a resposta é recusada, indicando os seus nomes); um `formula` personalizado é necessário para cada símbolo. O `index.tif` e a LUT contêm a câmara da pilha, a hora da captura (a exposição da câmara de referência, UTC) e a lente, e ambos designam o índice como a sua única banda (`XMP-Camera:BandName`, por exemplo, `NDVI`: o ficheiro PNG a cores indica o índice, não os seus canais de cor). `lattice index --live`. |

### `CliJob`

O identificador devolvido pelos três fluxos. A tarefa é executada no backend, que mantém o controlo das suas câmaras até ser interrompida.

| Membro | Descrição |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (fotogramas guardados / gravados, descartados, `fps`, `elapsed`, falhas de gravação, `stop_reason`; numa tarefa contínua também `frames_grabbed` e `frames_queued` — cada fotograma capturado é gravado, colocado em fila, descartado ou falha — e `drop_reason`, `writers` e `write_ms`, o tempo médio que demorou a gravação de um fotograma) e `output_dir`. `state` é `running`, `ended` (parou por si próprio e ainda não foi parado), `stopping` (um processo de paragem está a gravar a sua fila: estatísticas em tempo real) e `stopped_by`) ou `finished` (as suas estatísticas finais, o `error` e o `stopped_by` da paragem). O registo de um trabalho interrompido é mantido durante 10 minutos (no máximo os últimos 32), para cada cliente: outro script ou o `chloros-mcp` lê o mesmo estado e as estatísticas finais. Um trabalho `chloros-cli` iniciado (`lattice continuous`, `multi-stream`, `align-stream`) é interrompido pelo backend quando a CLI deixa de enviar verificações durante 30 s — encerrado, com a sua máquina em modo de suspensão — com `stop_reason`, `client_gone` e `stopped_by` no backend; a leitura das suas estatísticas a partir daqui não o mantém ativo. Um trabalho iniciado a partir do SDK funciona até ser interrompido. Cada início envia o seu `timeout` como prazo do backend (`client_deadline_s`): uma câmara que se abre depois disso (uma que ainda está a arrancar) é reposto e fechado e nenhum trabalho é iniciado, e a chamada gera o `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Uma câmara retida por um trabalho é recusada a todas as outras abertas (`connect_camera()`, outra abertura, uma captura transitória) com o código HTTP 409 `CAMERA_HELD`, indicando o trabalho e quem o iniciou. |
| `stop(timeout=30.0)` | Interrompe-o e fecha as suas câmaras: `final_stats`, `output_dir` e `error` quando a paragem não ocorreu corretamente. Uma segunda chamada devolve a mesma resposta, assim como uma interrupção a partir de outro cliente: uma enviada enquanto a primeira ainda está a gravar a sua fila aguarda por ela (até 25 s; passado esse tempo, gera um erro, e as estatísticas finais seguem-se em `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Aguardar até que termine por si próprio (que o seu `count` seja atingido, que a unidade fique cheia ou que ocorra um erro) e, em seguida, `stop()` para recolher as contagens finais. O `timeout` gera um erro `TimeoutError` enquanto ainda está a ser executado; um trabalho sem `count` só termina quando for interrompido. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | O seu ID e tipo (`continuous`, `multi-stream`, `align-stream`), onde escreve, a resposta da rota inicial e se o `stop()` foi executado. |

Utilize-o como um gestor de contexto para o interromper ao sair do âmbito.

---

## Pasta do projeto e receitas

**A pasta do projeto** é a pasta onde são criados novos projetos, partilhada pela aplicação Chloros, pela CLI e pelo SDK (módulo `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Uma pasta configurada que já não exista recorre à predefinição (`Documents/MAPIR/Chloros Projects`), tal como a aplicação. Um caminho relativo indicado para `set_project_folder()` é obtido a partir do seu diretório de trabalho quando o backend se encontra nesta máquina.

**Uma receita** é a lista de ações em YAML/JSON que o `chloros-cli project run PATH RECIPE` executa num projeto guardado. O `run_project_recipe()` executa-a com o próprio executor da CLIpróprio executor da CLI (instalado com o SDK), pelo que uma receita comporta-se da mesma forma em qualquer um dos seguintes:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** é um ficheiro `.yaml` / `.yml` / `.json` ou a própria receita: um dicionário com `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) e, opcionalmente, `overrides` e `stop_on_error` (valor predefinido «true»: a execução pára na primeira ação que falhar). As ações correspondem à [Recipe DSL](cli-reference.md#recipe-dsl) da referência da CLI; um `output` relativo é resolvido em relação ao diretório de trabalho do seu script.
- **Retorna** `ok` (todas as ações foram bem-sucedidas), `actions` (quantas), `log` (a saída do executor; o `echo=True` também a imprime à medida que é executado) e `validation`. Um ficheiro de receita que esteja em falta ou que não seja analisado corretamente gera o `ChlorosConnectError`.
- **Verificado antes da execução.**`validation` é `{"ok", "errors", "warnings", "steps"}`: cada ação deve ter exatamente um verbo conhecido como chave (`{"capture": {"name": ...}}`, não `{"action": "capture"}` ou `{"type": "capture"}`) que identifique uma câmara, matriz ou sensor existente no projeto, do tipo correto. Uma receita com um erro**não** executa nenhuma ação (`ok` false, os erros em `log`), independentemente do que `stop_on_error` indique. As chaves não utilizadas, e todos os painéis ou sensores de luz controlados pela receita que não estejam ligados neste momento, são `warnings`. Uma receita não liga dispositivos: ligue o primeiro do projeto (`ChlorosProject.connect_all()`, `chloros-cli project connect`, ou a ferramenta MCP `connect_project_devices`). `dry_run=True` não executa nada e devolve `{"recipe": ..., "validation": ...}`.
- Numa processo, só é executada uma receita de cada vez. Uma receita YAML necessita do PyYAML.

---

## Automatização de Projetos — `ChlorosProject`

Um projeto Chloros guardado é uma pasta que contém `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` carrega o manifesto (`relink`: a pasta onde se encontram agora os ficheiros de origem em falta de um projeto movido, utilizada quando o backend o carrega — `process()` — ver **Um projeto movido** em `ChlorosLocal`), e o `connect_all` coloca online todas as câmaras, matrizes e sensores de luz guardados com as suas definições guardadas — o mesmo estado de hardware que a GUI produziria. Os sensores de luz ligam-se através do [pool de sensores DAQ](#daq-sensor-sessions), pelo que necessitam de um backend em execução (ver [`SensorHandle`](#sensorhandle)).

### Exemplo mínimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Ou como gestor de contexto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Métodos `ChlorosProject`

| Método | Descrição |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Deteta e liga todas as câmaras, matrizes e sensores de luz guardados. Devolve um relatório de ligação por classe (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, cada um com `connected`, `missing` e `errors`; o `arrays` também inclui o `alignment`). Com um backend a escutar no `127.0.0.1:5000`, as câmaras e matrizes guardadas são localizadas e ligadas através dele. Na ausência de um backend, este processo localiza-as e controla-as por si próprio através do `lattice_sdk`, que necessita do ambiente de execução do SDK da Arena e do OpenCV (`cv2`) — o pacote SDK não instala nenhum dos dois; quando tal não é possível, todas as câmaras e matrizes guardadas são listadas em `errors` com a respetiva razão, e `missing` permanece vazio. Um backend que está em execução mas rejeita o SDK (desconectado: HTTP 401; um plano abaixo do Copper: 403) não é considerado «sem backend»: todas as câmaras, matrizes e sensores de luz guardados são listados em `errors` com o motivo do backend, e nada é aberto neste processo. Cada chamada verifica novamente o backend, pelo que o `connect_all()`, após o início de sessão ou após o arranque do Chloros, estabelece ligação ao mesmo objeto de projeto. Uma matriz guardada que já esteja ativa no backend com as mesmas câmaras (as da GUI ou as de `chloros-cli project connect`) é anexada tal como está, sem reaplicar as suas definições guardadas, e o `disconnect_all()` deixa-o em execução. Os sensores de luz passam sempre pelo backend (ver [`SensorHandle`](#sensorhandle)). Nunca cria um backend. |
| `disconnect_all()` | Desmontar tudo. Todos os dispositivos são primeiro libertados; depois, se o backend não tiver confirmado a libertação de uma câmara, matriz ou sensor de luz (o tempo de espera da desmontagem expirou ou foi recusado), um `ChlorosConnectError` identifica cada um. Um dispositivo já desativado, ou sem backend em execução, permanece em silêncio. Uma matriz ou um sensor de luz que outro cliente tivesse aberto permanece em execução. A saída do projeto `with` nunca é acionada. |
| `capture_all(output_dir=".")` | Um fotograma de cada câmara + matriz ligada, e um espectro (`SensorHandle.read()`) de cada sensor de luz ligado, captados sucessivamente (câmaras, depois matrizes, depois sensores), e não num único instante. |
| `stream(camera, overlays=False, fps=10.0)` | Gerador que produz fotogramas BGR `numpy` a partir de uma câmara (ou matriz) nomeada. O `overlays=False` lê fotogramas com o identificador`grab()` no nível `debayered`: através do backend quando o projeto está ligado a um, caso contrário, um ciclo de captura direto `lattice_sdk` (as matrizes produzem dicionários `{serial: frame}`). `overlays=True` encaminha através de `ChlorosLocal.camera_stream()` → o feed MJPEG do backend, com o bloco `ui.overlay` guardado pela câmara a ser passado como parâmetros de consulta. Requer o modo backend, OpenCV (`cv2`) e uma **câmara autónoma**: uma câmara em modo direto gera o erro `RuntimeError` (o backend não consegue aceder a uma câmara pertencente a este processo) e um array gera o erro `NotImplementedError` (sobrepõe a composição por câmara — transmite um membro pelo nome). Equivalente de execução única: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Executar o alinhamento em todas as matrizes atualmente ligadas. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executar o pipeline de calibração/indexação nas imagens do projeto (envolve `ChlorosLocal.process`; estes quatro são os **únicos** argumentos-chave aceites — `indices=` etc. provoca um erro `TypeError`; define índices através de `ChlorosLocal.configure()`). Constrói de forma diferida um `ChlorosLocal()`, que inicia automaticamente um backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` indexado por nome E número de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexado por nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexado por nome E slot_id.
- `proj.config` — `project.json["config"]` dicionário.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Níveis de processamento.** `capture()`, `grab()` e `frame_stream()` utilizam todos o mesmo token `processing`
, e a cadeia é cumulativa — cada nível executa tudo o que está acima dele:

| Nível | Saída | Notas |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo do sensor | Sem demosaico. As sobreposições não estão disponíveis neste nível. |
| `debayered` | Cor de 3 canais (**padrão** para `CameraHandle` e todos os `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` e `stream()` têm como predefinição o `raw`): BGR na memória (`grab()` / `frame_stream()`), guardado como RGB pelo `capture()` | No modo backend, como captura: o desvio de escuro (DSNU) do pacote de calibração aplicado ao raw, seguido do demosaico sensível a arestas do OpenCV — sem LUT de linearidade, campo plano ou desmistura. O único nível que funciona sem o modo backend, onde é utilizado o demosaico bilinear simples da biblioteca da câmara (sem desvio de escuro). O ficheiro TIFF guardado é RGB fotométrico, pelo que se invertem os canais de um fotograma na memória (`frame[..., ::-1]`) para o comparar com um ficheiro. |
| `radiance` | float32, W/m²/sr/nm, **um plano por banda** | Cadeia radiométrica completa: demosaico + separação 3×3 (multiespectral) + DSNU + campo plano + escala NIST, com a exposição × ganho eliminados para que os valores sejam absolutos. |
| `reflectance` | uint16, 32768 = 1,0 (ou uint8, 255 = 1,0 — ver `bit_depth`), **um plano por banda** | Radianância dividida pela irradiância descendente (ρ = π·L/E). Requer uma leitura DLS/DAQ — ver a nota abaixo. |
| `display` | 8 bits, semelhante a sRGB | Renderização equivalente à GUI: CCM + equilíbrio de brancos + gama através do perfil de cor ativo da câmara. |

Apenas no modo direto (sem backend) está disponível o `debayered`: o `grab()`, noutro nível, ignora
`NotImplementedError`, o `capture()` ignora o `processing` e o `frame_stream()` eleva
`RuntimeError` em todos os níveis. O `reflectance` necessita de uma leitura descendente utilizável — o ponto final do fotograma puxa
o DAQ agrupado para a ranhura DLS da câmara automaticamente, mas sem nenhum DAQ associado, a cadeia recusa a
saída de refletância e regista honestamente a desclassificação nos metadados devolvidos, em vez de, silenciosamente,
devolver um produto de qualidade inferior.

> **Escala DN de refletância — não a codifique de forma rígida.** A refletância LATTICE utiliza `32768` = ρ 1,0 e regista
> XMP `Chloros:PixelScale=32768`; a refletância Survey3 utiliza `65535` = ρ 1,0 e não contém
> `Chloros:*`. Leia a etiqueta e divida por ela. Está definida no domínio uint16, pelo que permanece
> `32768` para todos os formatos que redimensionam (TIFF de 16 bits, TIFF de 8 bits, PNG/JPG de 8 bits, percentagem de 32 bits)
> — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float). A única
> exceção é um ficheiro de uma compilação anterior à versão 1.3.0: um TIFF de 8 bits era *recortado* nessa altura, não redimensionado, pelo que
> nenhuma escala o descreve, e não contém qualquer tupla `PixelScale` ou MicaSense. Trate uma etiqueta em falta num
> ficheiro de refletância LATTICE como «sem escala válida», e não como um valor por predefinição.

**Etiquetas de calibração: apenas no `raw`.** O ficheiro `raw` de uma captura contém o pacote de calibração da câmara: a carga útil `CalibrationJSON` e as etiquetas compatíveis com a MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que descrevem os valores brutos do sensor. Todos os ficheiros dos restantes níveis (`debayered`, `display`, `radiance`, `reflectance`, a renderização `index/`) e cada fotograma de gravação contínua/fluxo multicâmara contém a identidade da câmara (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e os próprios dados de gravação do fotograma, `XMP-mapir:FrameJSON` (exposição, ganho, temperatura, formato de píxel, hora de captura, correspondência DAQ), uma vez que todos os produtos `process()` contêm essa identificação. O processamento localiza a calibração desse ficheiro através do seu número de série e do `CalBundleSha256` no cache de calibração do Chloros, descarregando o pacote uma única vez caso este computador não o possua.

#### Ficheiros criados a partir de várias câmaras

Um ficheiro criado a partir das bandas de mais do que uma câmara — uma pilha multibanda (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), um índice em tempo real criado a partir de várias câmaras (`index.tif`, `index_LUT.png`), uma imagem `composite/` de uma captura em matriz — indica todas as câmaras que contribuíram para a mesma:

- **Por-câmara, por ordem de banda** (a composição: camada de fundo, seguida da camada de primeiro plano; uma camada de índice, as câmaras de onde provêm as suas bandas): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (o pacote de calibração pelo qual as bandas de cada câmara passaram; caso contrário — uma pilha `capture_aligned_multiband()` / `start_aligned_stream()`, cujas bandas não passam por nenhum — o pacote da câmara que o Chloros armazenou em cache, aquele cujo nome corresponde ao dos seus ficheiros de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, em que o item *n* de cada uma descreve a mesma câmara; `-`, onde uma câmara não tem valor (sem pacote de calibração, uma lente desconhecida). Cada câmara é **um** item: quando o seu pacote tem vários valores para um campo — os certificados de lâmpadas halógenas de um pacote — são unidas com `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca com uma vírgula, pelo que uma lista apresenta um item por câmara, independentemente da forma como for impressa.
- **EXIF `SerialNumber`** contém os mesmos números de série, separados por vírgulas (`214000535,214000551`).
- Não contém **nenhuma** identificação de câmara individual (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): esses identificam uma câmara e o seu conjunto, e o processamento determinaria a calibração do ficheiro a partir deles. Um índice calculado a partir das bandas de várias câmaras (`index.tif`, `index_LUT.png`) contém os dados EXIF `Model` `LATTICE` (o conjunto de câmaras), e não o modelo `LATT-…` de uma única câmara.

Um ficheiro das bandas de **uma** câmara — um ficheiro TIFF por banda, um fotograma `capture_aligned_multiband(per_camera=True)`, uma pilha ou um índice em tempo real de uma única câmara — contém as etiquetas de identidade próprias dessa câmara, tal como acontece com qualquer outro ficheiro de captura. A pilha do `capture_aligned_multiband()`, os ficheiros por banda e por câmara, bem como cada pilha do `start_aligned_stream()` e os seus ficheiros por banda, contêm a hora de captura do seu grupo (a exposição da câmara de referência, UTC), e um fotograma `per_camera` de uma câmara monocromática é um plano, tal como a pilha `bit_depth`.**Orientação da câmara (inversão / rotação).** Os fotogramas de `grab()` / `frame_stream()` estão sempre na
orientação nativa da câmara, em todos os níveis. As capturas guardadas seguem a configuração de orientação
da câmara (as opções «Espelhar H» / «Espelhar V» / rotação da GUI):

- **Os ficheiros `raw` e `debayered` correspondem exatamente ao que o sensor leu**, independentemente da configuração — um mosaico Bayer
  girado ou invertido seria um padrão de filtro de cor diferente, e os mapas de calibração estão
  na própria grelha do sensor.
- **Os ficheiros `display`, `radiance`, `reflectance` e os ficheiros de índice estão orientados** — rodados uma vez, após todos os
  cálculos de calibração. As suas máscaras de limitação encontram-se na mesma grelha.
- **Cada ficheiro de captura regista a configuração** em XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, no sentido horário, após
  as inversões) e `ChlorosOrientationApplied` — `false` em imagens em bruto e sem debayering, `true` nos produtos orientados
  . `ChlorosProcessing` lista as operações aplicadas aos píxeis desse ficheiro (`none` quando
  nativo). Isso inclui todos os fotogramas de uma gravação contínua (a própria configuração da câmara,
  não aplicada) e de um fluxo multicâmara (a identidade: os seus fotogramas constituintes são nativos, tal como numa
  captura em matriz).
- **O processamento de um projeto aplica a orientação registada** aos produtos que cria a partir de um
  (primeiro a calibração nas coordenadas do sensor, depois a distorção de alinhamento da matriz e, por último, a orientação),
  para que correspondam aos produtos da captura em direto — incluindo os produtos de recurso não calibrados com resposta do sensor e com correção de vinheta;
  o seu produto sem debayering permanece nativo.
- **O visualizador de imagens do Chloros mostra um ficheiro bruto ou sem bayering virado de acordo com o seu registo** — apenas para visualização,
  o ficheiro não é alterado; um ficheiro sem o registo é apresentado tal como está armazenado.
- **As capturas sem o registo são processadas como nativas.** Antes da versão 1.3.0, uma captura com uma única câmara
  convertia ela própria os seus ficheiros RAW e sem debayering; o Chloros não consegue distinguir um ficheiro RAW desse tipo de um ficheiro nativo e
  não faz suposições.
- **As capturas em matriz mantêm todos os ficheiros membros na sua forma nativa por predefinição** e registam a identidade;
  O `ArraySession.capture(orient=True)` converte os seus produtos (ver os controlos de captura em matriz).

#### Saída por banda e matrizes mistas

Nos níveis `radiance` e `reflectance`, uma câmara devolve **um plano por banda dessa
câmara** — três para uma M3C (as suas três bandas não misturadas, ordem dos canais BGR), um para uma M3M (o seu
único filtro de banda estreita). Os planos replicados da câmara mono são agrupados, pelo que uma **matriz mista
M3C/M3M produz formas de matriz diferentes por série**. Leia a chave `bands` nos metadados do fotograma
para saber o que é cada plano; não o deduza a partir da contagem de canais:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Verifique sempre o `hdr["processing_level"] == "reflectance"` antes de utilizar os valores. Uma desclassificação
(sem leitura DAQ, sem pacote de calibração, disposição de bandas desconhecida) é reportada em `processing_level` e
`radiometric_refused` em vez de ser sinalizada — a matriz que recebe continua a ser uma quadro válida, apenas
não é o produto que solicitou. Uma captura guardada que foi rebaixada contém o mesmo código no seu XMP
que o `Chloros:RadiometricRefused` (por exemplo, `no-band-layout`, `no-dls-snapshot`).

#### Um disparador por leitura — `group_id`

**Os membros da matriz são servidos a partir do grupo de fotogramas sincronizados da matriz, e não a partir do próprio
buffer de captura de cada câmara.** As câmaras de uma matriz sincronizada por hardware expõem-se em conjunto, mas N leituras por câmara
não *amostram* em conjunto — a leitura do último fotograma armazenado em buffer de cada câmara ultrapassa os limites do disparador
, e um único fotograma perdido dessincroniza o conjunto sem que ninguém repare.

A disponibilização a partir do grupo é necessária, mas não suficiente por si só: cada pedido continua a
captura *o mais recente* grupo no seu próprio instante, e o ciclo de captura pode publicar entre esses instantes.
Medido num conjunto de 4 câmaras em tempo real, **12 de 20 leituras paralelas abrangiam dois grupos** — sempre
exatamente adjacentes. Assim, a leitura é **fixada**:

1. Cada fotograma contém `group_id`, o contador de publicações monotónico do conjunto.
2. `grab()` lê a câmara principal (ou, se esta não puder ser atendida, a próxima câmara disponível),
   e, em seguida, fixa as câmaras restantes ao ID do grupo em que se situou (`?group_id=N` no ponto final HTTP
  ).
3. O backend mantém os grupos recentes durante **pelo menos meio segundo**, pelo que um elemento já
   ultrapassado por publicações posteriores continua a ser atendido por esse disparador — tempo suficiente para que a
   exportação de refletância por câmara, mesmo num conjunto de 9 câmaras com sensores completos. Uma atribuição que o backend já não
   mantenha é recusada com `409` e `group_aged_out: true`, em vez de ser respondida silenciosamente
   com um gatilho diferente — o `grab()` volta a ler o conjunto.

Remensurado com fixação: **20 de 20 leituras devolveram um gatilho.** O `grab()` faz isto por si;
só precisa do `group_id=` quando acede diretamente ao ponto final HTTP.

- Uma câmara que não pode ser atendida no grupo fixado — ausente do mesmo ou recusando o pedido
  (um `band` que não possui, um nível que não consegue produzir) — é **omitida em vez de preenchida retroativamente**
  a partir de um gatilho diferente, e as outras câmaras continuam a ser devolvidas.
- O dicionário que o `grab()` devolve é um `GrabResult` (uma subclasse simples de `dict`;
  `from chloros_sdk import GrabResult`): `.errors` mapeia cada
  número de série omitido para a exceção que a sua leitura levantou, e `.group_id` é o grupo partilhado. Um
  `.errors` vazio significa que todas as câmaras compõem este grupo.
- Uma câmara autónoma não pertence a nenhum grupo, reporta `group_id: None` e **rejeita** um `group_id=`
  pin em vez de o ignorar.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

O mesmo contador é publicado em **cada fluxo MJPEG por câmara** de um membro do conjunto — a
pré-visualização (`/api/camera/stream/<serial>`), a pré-visualização anotada (`/stream-annotated`) e o
fluxo de refletância (`/stream-reflectance`) — como `X-Chloros-Group-Id` em cada parte, além de
`X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Os fluxos são contínuos e não
pontuais, pelo que são emparelhados através da recolha de partes e do agrupamento com base nesse cabeçalho — não há
nenhum pino para configurar. Uma câmara autónoma não tem grupo. Consulte
[Fluxos MJPEG por câmara](#per-camera-mjpeg-streams-no-rtsp).

**O `frame_stream()` nunca fica em silêncio.** Um tick que falha por uma razão que se pode resolver por si só
é ignorado — um fluxo de câmara não produz nada para ele, um fluxo de matriz apresenta as câmaras que
responderam (possivelmente nenhuma). Esses motivos são: ainda não há fotograma; o segmento de captura da câmara ficou em pausa durante um
segundo ou dois (`404 not streaming`, ao qual o backend responde sempre que outra câmara se liga
ou desliga, ou quando uma alteração de configuração reequilibra o conjunto); um reinício do backend ou `5xx`; o alinhamento
ainda a estabilizar-se (`409 … retry shortly`); e uma recusa de refletância porque o DAQ ainda não tem nenhuma leitura
(`no-dls-snapshot`, uma leitura desatualizada ou vazia). Uma recusa no tick seguinte apenas se repetiria — um
`band` desconhecido, `bit_depth=8` na radianância, um pacote de calibração sem escala de radianância, uma recusa de refletância
com **ausência total de DAQ** (nenhum associado à câmara ou ao seu conjunto de sensores, nenhum ligado ao
backend: o `dls_source` da recusa é `"none"`), qualquer outro `4xx` —
gera o `chloros_sdk.FrameFetchError` (um `RuntimeError`, que transporta `.status` e `.body`) de
uma só vez. O mesmo acontece com **60 segundos** de ticks saltados sem nenhum fotograma entre eles, seja qual for o `fps` — tempo
suficiente para superar a estabilização do alinhamento e um reinício do backend, mas curto o suficiente para que uma câmara que esteja
realmente deixou de funcionar termine a transmissão. Esse erro de desistência é também um `FrameFetchError`, tanto para uma câmara como para
um conjunto, com a própria exceção da última leitura em `.errors` (e, para uma câmara, como `__cause__`). Para
um conjunto, «recusado» significa que todas as câmaras recusaram; a exceção `.errors` contém o
motivo de cada câmara.

**Um conjunto `frame_stream()` gera cada gatilho uma vez** (tanto o `ArrayHandle` como o `ArraySession`). Um
tick cujo conjunto mais recente é o gatilho que já produziu aguarda o próximo gatilho em vez de
produzir o mesmo conjunto novamente, pelo que o fluxo decorre à taxa mais lenta entre `fps` e a própria taxa de fotogramas da matriz
, e os conjuntos consecutivos nunca partilham um `group_id`. Uma matriz que deixa de publicar gera
`FrameFetchError` («nenhum novo gatilho durante 60 s») em vez de repetir o seu último conjunto. Sem isto, um
fluxo que solicitasse mais do que o conjunto publicasse — ou que começasse logo após a ligação — apresentaria
repetições: numa execução medida, 20 conjuntos continham 11 IDs de grupo distintos.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportação de uma única banda — `band`

`grab()` / `frame_stream()` utilizam o `band="<name>"` para exportar apenas uma das bandas de uma câmara como um
matriz `(H, W, 1)` de canal único — com a mesma forma que uma câmara monocromática produz, pelo que a saída de uma única banda é
tratada de forma idêntica, quer provenha de um M3M, quer da divisão de um M3C. A banda vermelha de um FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

As grafias aceites são o nome completo do layout (`Red_660`, `NIR_850`, `Green_550`) ou o seu
prefixo (`red`, `nir`, `green`), sem distinção entre maiúsculas e minúsculas. Um nome desconhecido é rejeitado com a lista de
nomes disponíveis.

> **A seleção é feita pelo nome, não pelo número do canal — deliberadamente.** «Canal 1» é `Green_550` na
> ordem dos canais BGR da matriz, mas *vermelho* na ordem alfabética do código do filtro(`RGN`), e os dois
> não coincidem. Escolher o errado resulta num quadro perfeitamente plausível da banda errada, que
> nada a jusante consegue detetar. Não existe um índice numérico por esse motivo.

#### Alinhamento dos fluxos — `align`

O `align=True` transforma o fotograma para o fotograma de referência comum da sua matriz. Cada câmara é transformada
através da **mesma** resolução de perfil e área de interesse (ROI) de recorte, pelo que os fluxos por câmara abertos independentemente
ficam **mutuamente registados** — não é necessário solicitá-los em conjunto:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Requisitos e comportamento:

- A câmara deve estar num conjunto **sincronizado por hardware** (cabo de sincronização M8) que possua um perfil de alinhamento
  — o co-registo pressupõe que os membros sejam expostos simultaneamente, o que um conjunto de software ou
  matriz em funcionamento livre não proporciona. Caso contrário, é recusado com o código `409`.
- **Recusado, nunca desalinhado silenciosamente.** O caminho de captura pode guardar imagens desalinhadas com um aviso
  porque um ficheiro mantém as suas etiquetas de proveniência e pode ser re-registado mais tarde; uma transmissão em direto não tem
  essa segunda oportunidade, pelo que a falta de um perfil é um erro e não uma alternativa.
- **Solicitar o alinhamento inicia o processo.** Solicitar o `align` num conjunto sem perfil inicia a
  calibração e devolve o `409` com o que está a aguardar (por exemplo, *«à espera que a exposição
  se estabilize em 214000551 (1/3)»*) — tente novamente e terá sucesso, normalmente em poucas dezenas de segundos.
  Isto é importante no modo sem interface gráfica: o alinhamento automático no momento da ligação desiste após 30 s, e todas as tentativas
  dentro desse intervalo é ignorada enquanto o smart-AE ainda está a estabilizar; assim, um conjunto cuja exposição
  converge lentamente costumava ficar sem perfil para toda a sessão e sem forma de obter
  um sem abrir a vista combinada do ambiente de trabalho. O `alignment_status` também reporta
  `ae_align_holdouts`, indicando que as câmaras ainda estão a estabilizar-se e o seu progresso.
- Atribua a cada fluxo de uma matriz o **mesmo `max_h`**, ou os seus recortes ficarão em grelhas de píxeis
  diferentes.
- O alinhamento é aplicado *antes* da codificação da profundidade de bits, pelo que a distorção interpola valores de precisão total
  .

#### `bit_depth` — trocar precisão por largura de banda

`grab()` e `frame_stream()` aceitam `bit_depth=16` (predefinição) ou `bit_depth=8`:

| `bit_depth` | Codificação de refletância | Custo |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, margem até ρ 2,0 | Precisão total. |
| `8` | uint8 abrangendo `0..refl_max` (ρ 1,0 por predefinição) | Metade dos bytes; quantização de ~0,004 ρ no limite superior por predefinição. Os valores acima do limite superior são cortados — ver abaixo. |

##### O limite máximo de 8 bits — `refl_max`

`refl_max` define o ρ que o intervalo de 8 bits abrange, pelo que o `pixel_scale = 255 / refl_max` e o ρ continuam a ser
`pixel / pixel_scale`. O `grab()` e o `frame_stream()` aceitam-no como palavra-chave tanto na câmara como
manipuladores de matriz (`?refl_max=` no ponto final HTTP). Aumente-o para obter margem para realces especulares,
em passos proporcionalmente mais grosseiros:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | passo |
| --- | --- | --- |
| `1.0` (padrão) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **A exposição automática não consegue corrigir um recorte de 8 bits e, ao utilizá-la para tentar fazê-lo, os dados ficam piores.**
> ρ = π·L/E, com a exposição e o ganho eliminados do cálculo; por isso, ρ é uma propriedade da cena, não da
> configuração da câmara — medido num painel ativo, reduzir a exposição para metade alterou o valor médio de ρ em 1,3%. Aumentar
> a exposição para «ajustar» a gama leva, pelo contrário, os píxeis à saturação do **sensor**, o que
> subestima L e distorce ρ para **valores baixos** sem que seja sinalizado qualquer erro (medido: ρ máximo 1,203 → 0,985 com
> exposição a 2×, média −23%). Escolha a gama com o `refl_max`; manter o sensor fora da
> saturação é precisamente para isso que serve o AE, e isso protege a *validade* de ρ em vez da sua gama.

**Leia o `reflectance_clipped_fraction`, não o sinalizador.** O `reflectance_clipped_at_1_0` dispara com um
único pixel — numa cena real, reportou `True` enquanto a fração era `0.000001`, um pixel
em um milhão. A fração é o que indica se se deve aumentar o limite máximo.

**O `bit_depth=8` refere-se apenas à refletância.** A solicitação do `radiance` é rejeitada em vez de
ser atendida: a refletância tem um intervalo fixo de 0 a 1 que se mantém numa codificação de 8 bits, enquanto a radianância não tem
um intervalo fixo, pelo que a sua adaptação a 8 bits requer um alongamento percentil por fotograma cujo mapeamento
muda a cada fotograma — uma imagem, não uma medição.

#### Fluxo de refletância em tempo real (MJPEG)

O `GET /api/camera/<serial>/stream-reflectance` fornece refletância de 8 bits por banda como
`multipart/x-mixed-replace` — um M3C como JPEG de 3 canais, um M3M em escala de cinzentos. Parâmetros de consulta:
`fps` (padrão 10), `jpeg_q` (padrão 95), `max_h` (0 = nativo), `band` (uma banda nomeada, tal como
acima), `align` (`1` para co-registo — as mesmas regras que o `align` do SDK), `refl_max` (o limite máximo de codificação de 8 bits,
como acima). Cada parte contém
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` e `X-Chloros-Reflectance-Clipped`.
**As partes de câmaras diferentes que contêm o mesmo `X-Chloros-Group-Id` correspondem ao mesmo gatilho** —
é assim que vários fluxos por câmara de um conjunto são reagrupados, e isso é garantido
do lado do servidor, uma vez que os membros do conjunto são fornecidos a partir do grupo de fotogramas sincronizado (ver
[Um disparador por leitura](#um-gatilho-por-leitura--group_id)). Devolve `409` quando a refletância está
indisponível no momento da ligação e encerra o fluxo se esta se tornar indisponível a meio da execução; assim, um fluxo
paralisado significa «já não há refletância», em vez de «ainda está tudo bem, apenas estático».

> **O JPEG é com perdas — estes píxeis são uma medição.** O erro de compressão reflete-se diretamente nos
> valores de refletância e é mais grave nas bordas, que é onde se encontram os limites dos materiais. A qualidade
> é predefinida para 95 por esse motivo. Se precisar que os números permaneçam intactos, utilize
> `grab(processing="reflectance")` / `frame_stream(...)`, que devolvem o mesmo produto sem
> codec no caminho (e `bit_depth=8` se o custo da ligação, e não o codec, for a restrição).

> **Os dados EXIF são transferidos para a exportação.** `process()` copia o bloco GPS da captura de origem
> **e o seu ExifIFD** para cada produto, pelo que as exportações incluem `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, bem como a
> georreferenciamento. O `FocalLength` é a base a partir da qual a Pix4D calcula a distância entre amostras no solo — sem ele,
> a reconstrução recorre a uma escala extremamente errada (num caso medido, um local de 411 m
> transformou-se num de 47,8 km). A cópia não é deliberadamente o `-all:all`: as etiquetas estruturais do IFD0 quebram
> a saída do LATTICE, e o `ExifImageWidth`/`Height` são excluídos porque descrevem a captura de origem
> em vez da imagem raster exportada.

Sub-sinalizadores da fase de captura (aplicáveis aos níveis radiométricos — `radiance`, `reflectance`, `display`):

| Sinalizador | Padrão | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + separação 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de equilíbrio de brancos. Compatível com DLS quando um DAQ está associado à câmara. |
| `apply_index` | `False` | Avaliação do índice de vegetação. |
| `index_expression` | `None` | Substituir fórmula. Sevazio → ativa automaticamente o índice. |
| `annotated` | `False` | Sobreposição de decorações da GUI (zebra/grelha/pico). Não disponível para `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **O valor de retorno é um `chloros_sdk.CapturePaths`, do tipo `CapturePathMap`, e não um `Dict[str, str]`.**
> O `chloros_sdk.CapturePathMap` é um `Dict[str, Union[str, List[str]]]`: um caminho de nível único
> o `processing` atribui a cada número de série um caminho, enquanto o `processing="all"` atribui-lhe a
> **lista ordenada** de todos os produtos guardados para essa câmara. O `ArrayHandle.capture()` não aceita
> nenhum `levels=` e ignora os argumentos-chave que não reconhece (`levels`, `daq`,
> `render_index`, `force_daq`), por isso passe apenas os que aqui são apresentados (`aligned=False`
> guarda ficheiros nativos, tal como no `ArraySession.capture()`); utilize
> `ArraySession.capture()` para esses controlos. Um composto combinado em tempo real, caso esteja a ser transmitido, chega sob a chave extra
> chave `"combined"`, em vez de sob um número de série. O código que assume o `str` falha na
> forma de lista sem que qualquer verificador de tipos levante objeções — a anotação indicava o `Dict[str, str]`
> durante algum tempo após o lançamento da forma de lista, razão pela qual o alias existe. Normalize
> quando pretender a forma plana:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **O que não poderia ser escrito: `.failed`.** O mapa que o `capture()` devolve é um
> `chloros_sdk.CapturePaths` — uma subclasse simples de `dict`, pelo que tudo o que foi referido acima funciona
> sem alterações — que também inclui `.failed`: os ficheiros que o backend não conseguiu gravar,
> as mesmas entradas que [`CaptureResult.failed`](#ficheiros-que-não-puderam-ser-gravados--captureresultfailed)
> (`serial`, `level`, `error` e `disk_full: True` numa unidade cheia). Uma
> captura parcial continua a devolver todos os caminhos que foram gravados; num disco cheio, o mapa fica
> vazio e o `.failed` explica o motivo. `.skipped` lista os saltos por câmara e
> `.group_seq` o grupo sincronizado, e o `.alignment` / `.oriented` são os
> do backend, tal como no `CaptureResult`. O `capture_smart()` devolve-os como
> `result["frames"].failed` e `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Alinhamento de matrizes

O `ArrayHandle` expõe toda a superfície de alinhamento. Por predefinição, os perfis são válidos apenas para a sessão — chame explicitamente o `export_alignment()` para os tornar persistentes.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alinhamento no momento da ligação

O `connect_all(align=...)` pode alinhar automaticamente todas as matrizes no momento da ligação:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Recorre ao `project.json["config"]["auto_align_on_connect"]` quando não especificado.

### `SensorHandle`

Um sensor de luz guardado (DAQ-U, DAQ-M ou DAQ-E), aberto pelo `connect_all()` no [pool de sensores DAQ](#daq-sensor-sessions) — o mesmo pool que o `connect_daq_sensor()` e o `chloros-cli daq pool-connect` utilizam.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- O `read()` aguarda o **próximo** espectro do sensor — um que já esteja armazenado em buffer no momento da chamada não conta — e devolve o `(spectrum, is_saturated, integration_time, x, y, z)`, ou o `None` se nenhum chegar no prazo de 5 s. Um fluxo em pausa é retomado. O `spectrum` está em W/m²/nm quando o sensor está calibrado.
- O `.sensor` é o [`DAQSensorSession`](#sessões-de-sensor-de-aquisição de dados); para tudo o resto: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` e `.settings` são as entradas guardadas a partir de `sensors.json`.

**O que o `connect_all` restaura.** Cada sensor é aberto no seu endereço guardado — a sua porta COM, endereço Bluetooth ou anfitrião DAQ-E — e, quando o projeto registou o ID do sensor, tem de ser esse sensor: uma unidade diferente nesse endereço é rejeitada, não está ligada. São aplicados o seu tempo de integração guardado, a média de fotogramas e a exposição automática, bem como a sua correção de tampa:

| Limite guardado | Ligado com |
| --- | --- |
| Um perfil de limite (por exemplo, `fov_30`) | Esse perfil. |
| Nenhuma selecionada | `sunshine_cosine`, a predefinição de ligação, registada como presumida (`cap_id_source=auto_default`), pelo que o processamento emite um aviso. Os sensores são fornecidos sem tampa: selecione a tampa que está instalada. |
| `none` — selecionado como **Nenhum (sensor sem tampa)** na interface gráfica | Sem correção da tampa. |

Um DAQ-E-S não aceita qualquer perfil de tampa, independentemente do que tenha sido guardado: o seu difusor faz parte da sua calibração (ver [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Um projeto guardado por um Chloros anterior registou um sensor sem tampa como opção por defeito, pelo que se reconecta com o `sunshine_cosine`; selecione **Nenhum (sensor sem tampa)** na interface gráfica mais uma vez e guarde o projeto.

Um sensor que outro cliente já tenha aberto no conjunto (na CLI ou noutro script) é ligado tal como está, sem reaplicar as definições, e o `disconnect_all()` deixa-o a funcionar; um sensor aberto pelo `connect_all()` é libertado. Tal como acontece com o `connect_daq_sensor()`, a GUI do Chloros **não** faz parte do conjunto: feche um sensor na GUI antes de ligar o projeto a partir de um script.**Requisitos e o relatório.** Os sensores de luz necessitam de um backend do Chloros em execução e com sessão iniciada num plano pago do Chloros+ (Copper ou superior), e o `connect_all()` nunca inicia um. Sem nenhum em execução, todos os sensores guardados são listados no `report["sensors"]["errors"]`, indicando essa situação. Um sensor que o backend não consiga abrir também é listado aí, com o motivo indicado pelo backend; o `report["sensors"]["missing"]` permanece vazio para sensores de luz.

---

## Hardware direto (sem backend)

Quando pretender zerar a dependência do backend para câmaras LATTICE (CI, robôs sem periféricos, incorporadas), importe o `lattice_sdk` diretamente — este é reexportado pelo `chloros_sdk`. Atenção ao `CAMERA_AVAILABLE`: O `lattice_sdk` está incluído no pacote «wheel» do SDK, mas requer a presença do runtime do Arena SDK e do OpenCV (`cv2`), e o pacote «wheel» não instala nenhum deles. Os sensores de luz passam pelo backend — [`connect_daq_sensor()`](#daq-sensor-sessions) ou o `proj.sensors` de um projeto — com uma exceção, o auxiliar de magnitude de banda do DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), que também necessita do `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Predefinições e o disparador

Três das quatro predefinições funcionam em **modo livre**: a câmara expõe continuamente e um
`capture()` devolve o quadro seguinte. O `triggered` é a exceção — prepara a
câmara para uma flanco de hardware na Linha 2, pelo que não captura nada até que este ocorra.

| Predefinição | Gatilho | Utilizar quando |
| --- | --- | --- |
| `default` | funcionamento livre | uso geral |
| `high_speed` | funcionamento livre | 8 bits, limite de 60 fps, exposição curta |
| `high_quality` | funcionamento livre | 12 bits, sem limite de fps — a escolha habitual para fotografias |
| `triggered` | **preparada, Linha 2** | a câmara está ligada a um cabo de sincronização M8 e é acionada por outro dispositivo |

Se escolher `triggered` (ou configurar `trigger_mode="On"` manualmente) sem nada
a acionar a Linha 2, todos os `capture()` entrarão em timeout — corretamente, uma vez que pediu
à câmara para aguardar. O SDK explica isto quando acontece; consulte
[SC_ERR_TIMEOUT durante a captura](#direct-hardware-backend-free).

> **Nota — As mensagens «GVSP probe» / `SC_ERR_TIMEOUT -1011` na ligação não são erros.**
> Na ligação, o SDK tenta negociar**quadros jumbo** (pacotes GVSP de 9000 bytes) para obter um maior débito. Numa ligação direta ponto a ponto via NIC (por exemplo, um endereço `169.254.x.x` local da ligação), a rede normalmente não consegue transportar quadros jumbo, pelo que esta sonda atinge o tempo limite e regista linhas como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Este é o **plano de contingência previsto**: o SDK reverte automaticamente para pacotes padrão de 1500 bytes e a câmara continua a ligar-se normalmente (as linhas `[chunk-enable …]` que se seguem fazem parte da sequência normal de ligação). A captura continua a funcionar.
>
> Pode ignorar esta sonda, mas **não se trata apenas de um silenciador de registos — desativa os quadros jumbo.** A câmara responde a pings «Don&#x27;t-Fragment» apenas até 1500 bytes, independentemente da qualidade da sua rede; por isso, o teste de ping por si só nunca consegue detetar quadros jumbo; esta sonda é a única que o consegue fazer. Desative-a e a câmara passará a utilizar pacotes padrão de 1500 bytes para sempre, em qualquer rede:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Só vale a pena numa rede que *saiba* que não suporta quadros jumbo, onde poupa cerca de um segundo de tempo de ligação por câmara. Uma vez que se trata de uma troca real e não apenas cosmética, o SDK agora indica isso quando a utiliza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Não altere esta configuração, a menos que tenha um motivo.**Se permanecer ativado, cada câmara é medida uma vez por sessão do backend e novamente quando a sua ligação muda (outro adaptador, outra velocidade de ligação ou uma ligação que caiu, como acontece ao trocar um switch); as reconexões reutilizam o resultado.**Após uma alteração atrás de um switch que permaneceu ativo, reinicie o backend para medir novamente.** Uma primeira ligação pode, ocasionalmente, demorar alguns minutos nesta verificação, e o registo indica isso quando acontece.
>
> Se *pretender* o débito jumbo, ative o jumbo de ponta a ponta (MTU da placa de rede de 9000 + um switch que os transmita), ou fixe-o com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando souber que a ligação o suporta — embora seja preferível utilizar o comando `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando em vez de o definir permanentemente, uma vez que um tamanho fixado ignora o teste de detecção e deixa de se adaptar à rede à sua frente. **Todos os** dispositivos no caminho têm de transmitir pacotes jumbo — incluindo qualquer divisor ou injetor PoE, o que é a razão habitual pela qual uma configuração, que de outra forma seria compatível com pacotes jumbo, não consegue transmiti-los.

> **O `SC_ERR_TIMEOUT -1011` durante o `capture()` / `grab*()` é um problema diferente — esse é um erro real.**
> A nota acima refere-se apenas ao `-1011` registado pela**sonda de tempo de ligação**. O mesmo erro gerado a partir de uma**captura** significa que a câmara se ligou corretamente, mas não está a enviar imagens:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> O indício revelador é uma câmara cujo canal de *controlo* está em bom estado — a deteção funciona, as definições e as gravações `[chunk-enable …]` são todas bem-sucedidas — enquanto *todos* os fotogramas atingem o tempo limite.
>
> **A causa habitual é a câmara estar configurada para um disparador de hardware.** Com os códigos `trigger_mode="On"` e `trigger_source="Line2"`, a câmara não emite absolutamente nada até que chegue uma flutuação elétrica ao cabo de sincronização M8. Se não houver nenhum cabo a alimentar essa linha, todas as capturas ficam à espera indefinidamente. A câmara não está avariada e a rede está a funcionar bem — está a fazer exatamente o que lhe foi pedido.
>
> O `CameraSettings()` e o `default` / `high_speed` / `high_quality` definem o modo de funcionamento livre, e uma captura que atinge o tempo limite enquanto está armada explica a situação em vez de apresentar apenas um `-1011`. O `PRESETS["triggered"]` arma a Linha 2, conforme previsto.
>
> Para forçar qualquer câmara a funcionar em modo livre:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se continuar a expirar com o código `trigger_mode="Off"`, a câmara não está, de facto, a enviar dados — envie-nos o registo e o código `ip link show`.

#### Perfis de cor (pré-visualização em tempo real RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleciona o perfil de cor do ecrã para a **pré-visualização em tempo real** em câmaras RGB (as câmaras multiespectrais ignoram esta configuração):

| Perfil | Significado |
| --- | --- |
| `raw` | Ignora completamente a cadeia radiométrica. |
| `linear` | DSNU + flat + WB, sem CCM, sem gama. |
| `natural` | Linear + CCM medido + gama sRGB, apenas com o acabamento básico (suavização de croma + dessaturação de realces) — a predefinição realista. |
| `enhanced` | `natural` mais o acabamento completo com paridade de hub (remoção de franjas, vibração, contraste local CLAHE). Aspecto mais rico com aproximadamente **o dobro do custo de acabamento por fotograma**, o que resulta numa taxa de fotogramas LIVE mais baixa. |
| `custom_temp` | `natural`, mas com o equilíbrio de brancos fixado em `custom_cct_k` Kelvin (DLS ignorado; limitado a 2000–10000 K no backend). |

O perfil é um regulador de velocidade/aspecto **apenas para pré-visualização em tempo real**: as capturas guardadas obtêm sempre o acabamento rico completo, independentemente do perfil selecionado; por isso, escolher o `natural` para recuperar tempo de fotograma não reduz a qualidade do que é gravado no disco. Um perfil desconhecido ativa o `ValueError`; quando um backend chloros está acessível, a alteração é também enviada para ele via POST, para que o próximo fotograma de pré-visualização a reflita (os utilizadores do SDK direto sem um backend continuam a receber a alteração das definições).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Destaques (ecrã RGB) — `set_highlights`

O `LatticeCamera.set_highlights(value)` retira os píxeis mais brilhantes e mais brancos do **ecrã** nas câmaras RGB — a correção para uma superfície branca brilhante que é renderizada como uma mancha branca plana. O `value` é o `-100`..`0` (limitado); o `0` está desativado, e a renderização é, então, exatamente como seria sem essa configuração. Configurações mais intensas reduzem o branco do ecrã (para cerca de 60% no `-100`) e alargam a curva de tons, pelo que uma superfície que a pré-visualização automática de brilho tenha empurrado para o branco puro recupera os seus detalhes.

Trata-se de um controlo de visualização, mas, ao contrário da saturação e do contraste (que apenas alteram a pré-visualização em tempo real), também altera o que se guarda: a pré-visualização em tempo real e as capturas **ao nível do ecrã** seguem esta configuração, pelo que esses ficheiros também ficam mais escuros (apresentam o valor `Chloros:DisplayHighlights`); as exportações de radiação e refletância nunca o fazem, e não é possível recuperar dados do sensor que tenham sofrido clipping — reduza a exposição para esse efeito. As câmaras multiespectrais e monocromáticas ignoram-no. O valor é armazenado em `settings.highlights` e enviado via POST para um backend chloros acessível, exatamente como o `set_color_profile`; o equivalente na CLI é o `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Câmaras monocromáticas (M3M) e `Calibration`

Uma câmara monocromática **M3M** (`M3M-<lens>-F<wavelength>`) é de banda única: um plano em escala de cinzentos, sem mosaico de Bayer, sem matriz de interferência espectral 3×3. O `Calibration` reconhece-a e expõe um sinalizador `is_mono`. A refletância continua a aplicar-se como um mapa radiométrico por banda (a separação de cores é a matriz identidade), mas os cálculos multibanda numa única câmara produzem resultados válidos em vez de valores sem sentido:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Para construir um índice de vegetação a partir de hardware monocromático, combine várias câmaras M3M em diferentes comprimentos de onda numa pilha multibanda alinhada (ver [Alinhamento de Matriz](#array-alignment)) e calcule o índice nessa pilha, em vez de o fazer numa única câmara.

#### Magnitudes de banda do DAQ-M — `DLS.band_magnitudes`

O `DLS(mac_address, filter_model="RGN-IMX265")` liga-se a um DAQ-M via Bluetooth através do seu próprio MAC BLE. É a única classe de sensor de luz que comunica diretamente com o sensor, em vez de o fazer através do backend, pelo que necessita do que as classes de câmara direta necessitam (`CAMERA_AVAILABLE`: o runtime do SDK da Arena e o OpenCV), além do `bleak` para Bluetooth (`pip install "chloros-sdk[camera]"` ou `pip install bleak`; sem ele, o `DLS` gera o `DLSError`). Um DAQ-M ligado noutro local (no conjunto do backend, num projeto, na GUI) deixa de anunciar, pelo que deve libertá-lo primeiro nesse local. **O `mac_address` é obrigatório**: não existe um valor predefinido, uma vez que cada unidade tem o seu próprio, e o `DLS()`, sem um, gera o `DLSError` antes de qualquer tentativa de ligação. Encontre o MAC com o `discover_daq_sensors(transports=["ble"])` (ver [Descoberta](#discovery--finding-an-address-to-connect-with)) ou `chloros-cli daq pool-discover --only ble`.

O `DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integra um espectro DAQ-M (a leitura mais recente quando o `spectrum` é `None`; `DLSError`, caso ainda não exista nenhum) através das respostas dos três canais de uma câmara e devolve `{"red": …, "green": …, "nir": …}` — as chaves nomeiam os canais nessa ordem; com as curvas de uma câmara RGB, o terceiro (`nir`) é o seu canal azul.

- Sem o `spectral_response`, utiliza a curva padrão do modelo de filtro (aquela com a qual o `filter_model` e o `DLS` foram construídos), que foi medida numa unidade.
- O `spectral_response` utiliza, em vez disso, as curvas medidas **pela própria** câmara: um dicionário `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, tal como armazenado no pacote de calibração dessa câmara (`color_correction.spectral_response_smoothed` — câmaras RGB; os pacotes multiespectrais não incluem essa curva). Se não for possível utilizar esse dicionário, recorre-se à curva da frota.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

O `chloros-cli lattice dls --camera SERIAL` da CLI faz o mesmo através do backend, encontrando a curva da câmara no seu pacote de calibração carregado ou armazenado em cache, e indica qual a curva utilizada. Via HTTP, isso corresponde a `POST /api/dls/cli-read` com `mac` (obrigatório: sem ele, a resposta é HTTP 400, `code` `MAC_REQUIRED`) e `camera_serial` no corpo; o `band_magnitudes_source` da resposta é `camera <serial>` quando foi utilizada a curva própria dessa câmara, ou `model <filter>` para a curva da frota (uma câmara sem curva: não é RGB ou não tem nenhum pacote carregado ou armazenado em cache).

---

## Exceções

Intercepte a classe base para tratar «qualquer coisa que tenha corrido mal no Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` são atributos de `chloros_sdk` (e de `chloros_sdk.exceptions`), mas não em `__all__`, pelo que `from chloros_sdk import *` não os inclui: importe-os pelo nome. A família `ChlorosError` é derivada de `ChlorosLocal` e das funções nela baseadas (`process_folder()`, `process_lattice_capture()`); a superfície «smart-connect» herda de `ChlorosConnectError`.

Hierarquia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemplos de ponta a ponta

### 1. Processar uma pasta com uma barra de progresso personalizada

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matriz LATTICE em tempo real → Refletância + Referência DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanha de captura orientada por projeto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Fluxo de fotogramas de várias câmaras → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captura direta no hardware (sem backend) sem interface gráfica

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verificação de capacidades antes de ligar uma matriz de 4 câmaras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente à receita de captura (Python puro)

O `run_project_recipe()` executa uma receita CLI tal como está (ver [Pasta do projeto e receitas](#project-folder--recipes)). Escritas em Python, as mesmas etapas são:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Servidor MCP (assistentes de IA)

O `chloros-mcp` permite que um assistente de IA que utilize o [Model Context Protocol](https://modelcontextprotocol.io) (MCP), como o Claude Desktop, o Claude Code ou o Cursor, controle o Chloros: processar imagens (criar um projeto, importar uma pasta de capturas, escolher definições, processar, relatar o que a execução exportou) e controlar câmaras LATTICE, matrizes e sensores de luz DAQ (ligar, pré-visualizar, definir exposição e cor, capturar, gravar, atualizar firmware). As suas ferramentas abrangem o que este SDK faz. Basta pedir em linguagem simples e o assistente aciona as ferramentas do servidor:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

O servidor está construído com base neste SDK: as ferramentas de processamento chamam [`ChlorosLocal`](#image-processing--chloroslocal) e as ferramentas de hardware que as [chamadas smart-connect](#smart-connect-for-lattice-cameras), [sessões DAQ](#daq-sensor-sessions), `ChlorosProject` e os [utilitários da câmara](#camera-utilities), todos a comunicar com o mesmo backend que a aplicação Chloros utiliza. As suas solicitações incluem `X-Chloros-Client: sdk`, pelo que o backend aplica as verificações de início de sessão e de plano do SDK, e as regras do SDK são aplicadas em conformidade (por exemplo, o Texture Aware é recusado para capturas LATTICE).

### Instalação

O servidor vem incluído no pacote `chloros-sdk`; não há mais nada para instalar. O SDK instala um comando `chloros-mcp`, que é idêntico ao `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** o instalador do Chloros instala o SDK através do pip no seu Python, o que coloca o `chloros-mcp.exe` na pasta `Scripts` desse Python.
- **Linux (.deb):** o pacote instala o SDK no sistema `python3` e coloca o comando `chloros-mcp` em `/usr/bin`, pelo que este se encontra no `PATH`. Num virtualenv, o `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` adiciona o `chloros-mcp` ao `bin` do virtualenv.

O servidor utiliza apenas a biblioteca padrão, além do SDK (Python 3.7+). Um cliente MCP inicia-o e comunica com ele através de stdin/stdout; não é necessário executá-lo manualmente, exceto com o `--help`.

### Configuração do cliente

**Claude Desktop:**adicione o servidor ao `claude_desktop_config.json` (no Windows, `%APPDATA%\Claude\claude_desktop_config.json`; aceda a**Definições &gt; Desenvolvedor &gt; Editar Configuração** para o abrir) e, em seguida, reinicie o Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** a mesma entrada `mcpServers` do Claude Desktop, em `~/.cursor/mcp.json` (em todos os projetos; `%USERPROFILE%\.cursor\mcp.json` no Windows) ou `.cursor/mcp.json` num projeto.

As opções devem ser inseridas em `args` ou após o comando para o Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Para utilizar um SDK específico do Python em vez do `chloros-mcp` no `PATH`, execute o módulo com esse Python (o seu caminho completo como `command`, caso não seja o primeiro `python3` no `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Um cliente inicia o servidor sem o seu shell; por isso, se o cliente indicar que o servidor não conseguiu iniciar, é provável que o `chloros-mcp` não esteja no `PATH` que o cliente vê. Indique, em vez disso, o caminho completo (`where chloros-mcp` no Windows, `which chloros-mcp` no Linux, onde o ficheiro .deb é `/usr/bin/chloros-mcp`), ou execute o módulo com o Python que inclua o SDK.

### Opções

| Opção | Padrão | O que faz |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | O backend do Chloros a utilizar. |
| `--no-auto-start` | desativado | Nunca iniciar um backend: utilizar um já em execução, como o da aplicação Chloros, o serviço de backend do Linux ou um túnel para a (ver [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel); os caminhos das pastas devem, nesse caso, indicar a mesma pasta em ambas as máquinas). |
| `--backend-exe PATH` | onde o Chloros o instala | O backend a iniciar quando nenhum estiver em execução, para um Chloros instalado num local onde o SDK não procura. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` ou `ERROR`. Os registos são enviados para o stderr, que a maioria dos clientes apresenta no seu registo MCP; o stdout transporta apenas mensagens de protocolo. |

### Ferramentas

Os argumentos de cada ferramenta são os da chamada do SDK que esta encapsula; o cliente lista cada um deles com o seu esquema completo. Os caminhos são absolutos.

**Processamento**

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Cria um projeto e torna-o o projeto aberto, sobre o qual as ferramentas de processamento atuam. Um nome simples cria a pasta na pasta de projetos predefinida; um caminho absoluto cria-a nesse local. |
| `open_project` | `ChlorosLocal.open_project` | Abre um projeto guardado e define-o como o projeto aberto. Os caminhos de um projeto movido são redirecionados para a pasta onde se encontra agora; `relink_folder` (absoluto) indica a pasta onde se encontram agora os restantes ficheiros em falta. Devolve `source_files` e `image_count`. Não foi encontrado nada: um erro a indicar onde estavam e a nomear `relink_folder`; nada está aberto. |
| `import_images` | `import_images` | Importa pares Survey3 `.RAW`+`.JPG`, LATTICE `.tif` capturas e DAQ `.daq` gravações; devolve `count` (imagens) e `scan_count` (gravações). |
| `configure_processing` | `configure` | Altera as definições passadas e nenhuma outra (valores: [Valores suportados](#supported-values)); `custom_settings` envia toda a configuração tal como foi fornecida. |
| `get_project_settings` | `get_config` | Lê as definições guardadas de processamento, exportação e índice do projeto aberto. |
| `start_processing` | `process(wait=False)` | Inicia o processamento do projeto aberto e retorna imediatamente. |
| `get_processing_status` | `get_status` | Comunica `state` (`idle`, `starting`, `processing` ou `finished`) e o progresso; uma vez concluída, a execução apresenta os códigos `summary`, `export_check_failure` quando as exportações não puderam ser verificadas e `image_failure` quando as imagens se perderam: os restantes produtos são gravados; registar a execução como falhada. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Cria um projeto para uma pasta, importa-o, configura e inicia a execução numa única chamada (`lattice: true` para capturas LATTICE). Enquanto uma execução está em curso, o sistema recusa a operação antes de criar qualquer coisa, e o backend também recusa uma alteração de projeto a partir de qualquer ferramenta nessa altura. |
| `export_light_sensor` | `export_light_sensor` | Grava ficheiros calibrados `.daq` e `.csv` para as gravações do sensor de luz do projeto sem processar imagens, obtendo a calibração de fábrica a partir da MAPIR Cloud quando esta não se encontra em cache (ver [Registos do Sensor de Luz](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Lê os metadados de auditoria do Chloros de uma imagem processada (fonte, calibração, etapas aplicadas). |

**Câmaras e matrizes**

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | O que está ligado neste momento, em qualquer cliente, cada um com `connected_here`; além dos trabalhos em execução no servidor. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Câmaras e sensores de luz que poderiam estar ligados. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Estado em tempo real de uma câmara, de um conjunto de câmaras (com a respetiva capacidade e estado de funcionamento) ou de um sensor de luz. |
| `analyze_array_network` | `analyze_array_network` | O que este anfitrião e esta rede podem suportar para um conjunto proposto. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Ligar uma câmara ou um conjunto sincronizado (começando pelo mestre). |
| `connect_project_devices` | `open_project(...).connect_all` | Ligue todos os dispositivos que um projeto guardado recorda, com as respetivas configurações. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Desligar uma câmara, um conjunto ou um sensor de luz (ver Comportamento). O `keep_settings: true` numa matriz guarda as configurações de cada câmara na sua própria memória; o `clear_settings: true` reinicia essa memória (o seu resultado inclui o `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Os dispositivos do projeto, em conjunto. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Ler ou escrever definições da câmara, por nome amigável ou nome de nó GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Definir a taxa de um array ou restabelecer a taxa automática. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Uma captura (`fastest` e `wait_for_exposure` são `capture_fastest` e a captura com exposição ajustada). Na ausência de `output_dir`, guarda-se na pasta `captures` do projeto abertopasta `captures` do projeto aberto, e os ficheiros são incorporados no projeto (retidos até que uma execução de processamento termine); a resposta `project_add` confirma isso. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Capturas contínuas ou em intervalos, como uma tarefa. |
| `start_array_recording` | `ArraySession.record` / `burst` | Um vídeo com índice combinado ou uma sequência em bruto, como uma tarefa. |
| `build_burst_video` | `ArraySession.build_video` | Reprocessar uma sequência em vídeo calibrado, como uma tarefa. |
| `get_job_status` / `stop_job` | | Acompanhar ou interromper uma tarefa. |
| `get_camera_preview` | | Pré-visualização em tempo real de cada câmara, devolvida como uma imagem. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Um fotograma (ou um conjunto sincronizado) num nível de processamento, sem guardar: estatísticas por plano e um ficheiro PNG de cada um. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Alinhamento de uma matriz. |

**Sensores de luz**

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Liga um DAQ-U, DAQ-M ou DAQ-E (`cap_id` define a correção do limite). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | As leituras mais recentes: carimbo de data/hora, saturação, intervalo e pico do espectro, e a média do espectro em intervalos de comprimento de onda (`full_spectrum` para cada ponto). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Iniciar ou parar o fluxo. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Gravar um ficheiro `.daq`, como uma tarefa; `duration_s` interrompe-o automaticamente. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | A correção da tampa (`none` para um sensor sem cobertura). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Qual o sensor que fornece a refletância de cada câmara e matriz. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Os conjuntos de bandas e a refletância e índices «Ambiente + Objeto» de dois sensores de transmissão contínua. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Configurações próprias de um DAQ-E, guardadas na unidade. |

**Utilitários da câmara** ([Utilitários da câmara](#camera-utilities))

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` e os seus setter | Cor de exibição de uma câmara RGB: perfil e temperatura de cor, saturação/contraste, realces. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Equilíbrio de brancos de disparo único, ou automático / desligado / proporções fixas. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Predefinições da câmara. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Os modelos de filtro, ou um por nome. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versões de firmware e um flash (uma tarefa por predefinição). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Estado de alimentação e térmico; em modo de suspensão ou ativo. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coeficientes de um alvo de calibração na vista. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Os adaptadores de rede da câmara, uma estimativa e a correção (apenas para o administrador). |
| `apply_vignette_correction` | `apply_vignette_correction` | Corrigir a vinheta de uma pasta de imagens. |
| `read_dls` | `read_dls` | Uma leitura DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | O hardware e o tempo de execução do anfitrião. |

**Câmaras transitórias, perfis de alinhamento e índices** ([Matrizes transitórias, perfis de alinhamento e índices](#transient-arrays-alignment-profiles--indices)). Todas as ferramentas aqui que abrem câmaras utilizam `keep_settings` / `clear_settings` (padrão: false): consulte [Memória da câmara](#camera-memory--keep_settings--clear_settings).

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Grava uma câmara ou transmite várias em sincronia, como uma tarefa (o `duration_s` interrompe-a automaticamente). |
| `capture_cameras_transient` | `capture_cameras_transient` | Uma captura sincronizada sem ligar uma matriz. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Verificação da sincronização do disparador e quais as câmaras que estão na linha de sincronização. |
| `analyze_array_models` | `analyze_array_models` | O que um conjunto destes modelos obteria nesta rede. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Resolver e guardar um ficheiro de perfil de alinhamento (o `options` contém as opções mais detalhadas do solucionador). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Uma imagem multibanda alinhada ou um fluxo dessas imagens como uma tarefa. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | O conteúdo de um ficheiro de perfil; a ordem das bandas e as câmaras ativadas. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Predefinições e gradientes de índice; um índice sobre um ficheiro TIFF multibanda; captura para índice num único passo. |

**Sincronização de tempo, pasta de projeto e receitas** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Pasta de projeto e receitas](#project-folder--recipes))

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | O estado do «grandmaster» PTP, dos seus pares e do estado PTP de cada câmara de matriz. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Reiniciá-lo; definir as suas prioridades. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | A pasta onde são criados os novos projetos. |
| `run_project_recipe` | `run_project_recipe` | Executar a receita de um projeto guardado, como uma tarefa com o respetivo registo. Verificação inicial: uma receita com um erro é rejeitada e nada é executado; são emitidos avisos sobre matrizes/sensores não ligados (não os liga: primeiro o `connect_project_devices`). O `dry_run` retorna a verificação e nada é executado. |

Não existe nenhuma ferramenta para abrir um projeto guardado para processamento: cada tarefa de processamento começa com o `create_project` (ou `process_folder`).

### Requisitos

- **Chloros instalado no mesmo computador.** O backend do Chloros realiza o processamento. Em alternativa, o servidor pode utilizar um backend acedido através de um túnel (`--backend-url` com `--no-auto-start`, ver [Opções](#options)).
- **Ter iniciado sessão nesse computador**, na aplicação Chloros ou com o `chloros-cli login`. O servidor não pode iniciar sessão por si: uma ferramenta chamada enquanto estiver sem sessão responde «Não está com sessão no Chloros» com o motivo indicado pelo backend, e o assistente informa-o.
- **Qualquer nível pago do Chloros+ (Copper ou superior).** O servidor requer acesso ao SDK, e o nível gratuito Iron não o possui: uma ferramenta chamada no nível gratuito responde «Não disponível neste plano do Chloros» com o motivo indicado pelo backend.
- Python 3.7 ou mais recente com o SDK e um cliente MCP que inicie servidores locais (stdio).

### Comportamento

- **O backend.** Iniciar o servidor ou listar as suas ferramentas não inicia nada; a primeira chamada à ferramenta estabelece a ligação. Se a aplicação Chloros (ou o serviço de backend Linux) estiver a funcionar, o servidor utiliza o seu backend e mantém-no em execução. Se nenhum backend estiver a funcionar, o servidor inicia o que está instalado, sem janela. Quando o servidor termina (se encerrar o cliente ou se este reiniciar o servidor), interrompe os trabalhos de captura e as gravações do sensor de luz que iniciou e liberta os dispositivos aos quais se ligou; o backend continua a funcionar, de modo que uma execução de processamento inacabada prossegue até ao fim e o próprio backend encerra-se após 30 minutos sem que nada o esteja a utilizar. Com o `--no-auto-start`, o servidor nunca inicia um backend: uma ferramenta chamada sem nenhum backend em execução solicita que abra a aplicação Chloros e a chamada seguinte, depois de o fazer, estabelece a ligação.
- **Um projeto aberto.** O backend tem um projeto aberto, partilhado com a aplicação Chloros quando esta está aberta; por isso, não trabalhe noutro projeto na aplicação enquanto o assistente estiver a funcionar.
- **Apenas as definições passadas são alteradas.** O `configure_processing` mantém todas as definições que não lhe forem fornecidas nos seus valores guardados (o SDK do `configure()` redefine o debayer, a correção de vinheta, a calibração de refletância, o PPK e o formato de exportação para os seus valores predefinidos quando não são passados; o servidor preenche-as a partir do projeto). Se não conseguir ler as definições guardadas do projeto, não altera nada e indica isso, a menos que todas as cinco sejam especificadas. O `indices` substitui a lista de índices do projeto. O resultado lista o que foi enviado como `settings_sent`, e nomeia, sob `warnings`, qualquer índice que as bandas da câmara não consigam criar.
- **O processamento é do tipo «iniciar e depois consultar».** `start_processing` retorna assim que a execução começa, o que demora entre minutos e horas, e o assistente segue-se com `get_processing_status`. `state` é `idle` (sem processamento), o `starting` (o backend ainda não comunicou a execução, por um período de até 20 s), o `processing` ou o `finished`, com o [resumo](#post-run-summary--hints): o que foi exportado e indicações caso tenha sido exportado menos do que o esperado (sem imagens, nenhum alvo de calibração encontrado). Os produtos são gravados na pasta do projeto, tal como na aplicação. O `start_processing` é recusado enquanto uma execução estiver em curso, e um backend que deixe de responder é reportado como um erro, não como uma execução concluída.
- **O que o assistente vê.** Os avisos do SDK aparecem no resultado da ferramenta sob o código `warnings`, e as falhas (sessão encerrada, plano, caminho incorreto) são apresentadas como texto de erro que o assistente pode ler e transmitir ao utilizador. Uma lista com mais de 20 itens é reduzida aos primeiros 20, com a sua extensão completa indicada como `<key>_total`. Os caminhos devem ser absolutos.
- **Os dispositivos são partilhados.** Câmaras, matrizes e sensores de luz residem no backend, pelo que o assistente pode controlar um dispositivo ligado à aplicação Chloros, e as alterações que efetua são apresentadas na aplicação. O `disconnect_device` divulga o que o servidor ligou; um dispositivo ligado por outro cliente (a aplicação, o `chloros-cli`, um script, outro `chloros-mcp`) só é libertado com o `force: true`, que é o `force_disconnect` do SDK, e a recusa indica quem o ligou quando o backend tem conhecimento; um dispositivo que o backend não possui de todo é reportado como não ligado (`released: false`, `connected: false`, nada para libertar). Uma gravação ou transmissão executada pelo backend gera a sua visualização `get_job_status` com `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupos de fotogramas para uma transmissão multicâmara) — e uma linha `summary` («220 fotogramas gravados, 55 descartados, 0 com falha: ...»), enquanto está a ser executado e, ao terminar, os seus números finais; a lista de outro cliente sobre o mesmo (`other_clients_jobs`) apresenta os mesmos dados (`list_cameras()`, `list_arrays()` e `list_daq_sensors()` apresentam, cada um, os dados do seu dispositivo`connected_by`: `client`, `label`, `name`, `at`). Cada `chloros-mcp` identifica-se junto do backend como `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, enviado como `X-Chloros-Client-Label`; qualquer script SDK pode defini-lo). Interromper a gravação do sensor de luz de outro cliente também envolve o `force`.
- **Um trabalho demorado é uma tarefa.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, e qualquer captura, resolução ou índice solicitado para o `background`, devolve imediatamente um `job_id`; o `get_job_status` segue-se a este e o `stop_job` encerra a mesma. Uma execução continua até ser interrompida, a menos que lhe seja atribuído um `max_captures`, `duration_s` (ou, para uma sequência, `max_frames`); o `min_free_gb` recusa-se a iniciar quando a unidade tem menos espaço livre. Não é aplicado qualquer limite, a menos que se defina um. Uma gravação ou transmissão executada pelo backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) mantém o ID da tarefa do backend, pelo que outro `chloros-mcp` — iniciado posteriormente — o localiza através desse ID: o `get_job_status` segue-o (e lista tarefas como o `other_clients_jobs`, a partir do `GET /api/camera/cli-jobs`, cada uma com o `started_by`), e o `stop_job` só o interrompe com o `force: true`. Qualquer outro trabalho (uma execução de captura, um registo em matriz ou em rajada, uma captura em segundo plano) é executado no `chloros-mcp` que o iniciou, e apenas esse servidor tem conhecimento disso: outro que o tenha solicitado indica isso. Um processo interrompido lê o `stopping` enquanto escreve a sua fila e, em seguida, o `finished` com as suas contagens finais, durante 10 minutos e a partir de qualquer `chloros-mcp`: aquele que iniciou o processo vê a paragem forçada de outro da mesma forma. O `get_job_status` nunca aguarda uma paragem: durante uma (a sua própria `stop_job`, a de outro clienteou a paragem que liberta as câmaras de um trabalho que terminou por si próprio), responde imediatamente com o `stopping` e as contagens em tempo real, e o `stop_job` de um trabalho já em paragem indica isso mesmo. Um trabalho que terminou indica o motivo no `stop_reason`: o seu próprio motivo (`no_frames`, `disk_full`, `member_missing`, ou, no caso de uma execução contínua em matriz, `count`, `duration`, `array_gone`), `limit reached` apenas quando as suas contagens indicam o seu `count` (ou o `max_frames` de uma sequência, o`duration_s`) for atingido, `error`, `stopped` (com o `stopped_by`) quando um cliente a interrompeu; caso contrário, o `finished`.
- **Para onde vão as capturas.** Para a pasta `captures` do projeto aberto, onde a aplicação as guarda, a menos que seja especificado `output_dir`.
- **Imagens.** `get_camera_preview` e `grab_frames` devolvem imagens que o assistente pode visualizar, além do seu JSON. As imagens `grab_frames` têm o contraste ajustado para visualização; as suas estatísticas correspondem aos valores reais.
- **Quando o servidor termina**, as tarefas que iniciou são interrompidas e qualquer gravação do sensor de luz que tenha iniciado é encerrada. Se o servidor tiver iniciado o backend, também liberta os dispositivos aos quais se ligou antes de parar o backend; no backend da aplicação, estes permanecem ligados.
- **Protocolo.** Apenas stdio (sem ponto de extremidade HTTP), apenas ferramentas (sem recursos ou solicitações), resultados em texto e imagem, versões do protocolo MCP de 2024-11-05 a 2025-11-25.**O que está excluído.** Iniciar ou terminar sessão (a conta é sua; o servidor não pode iniciar sessão) e parar um backend que o servidor não tenha iniciado. Tudo o resto que este SDK faz em termos de processamento e hardware tem uma ferramenta. (O aprovisionamento de um DAQ-E — o seu nome, token de autenticação, palavra-passe OTA, requisito de assinatura, reposição de fábrica — e os servidores DAQ diretos são exclusivos do `chloros-cli`, também no SDK: um nome ou token incorreto bloqueia todas as instalações do Chloros na unidade.)

---

## Arranque automático do backend

Os pontos de entrada do smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` e `discover_daq_sensors` — são clientes HTTP «leves» que pressupõem que um backend está a escutar no `127.0.0.1:5000` (a URL predefinida da superfície de ligação inteligente). Quando a GUI ou a CLI já estão a funcionar, um deles já está em execução. A partir de um script simples, pode não haver nenhum — por isso, estas funções **iniciam automaticamente o binário do backend incluído** (sem janela, da mesma forma que o `ChlorosLocal` faz) antes da sua primeira chamada e, em seguida, aguardam até 60 s até que este inicie. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` nunca iniciam nenhum.

Regras:

- **Apenas um URL local é criado.** Um `backend_url` que aponte para `localhost` / `127.0.0.1` / `[::1]` é elegível; qualquer outro anfitrião é considerado como sendo a máquina de outra pessoa e nunca é iniciado.
- **Um backend que termine antes de responder encerra a espera.** A chamada é interrompida imediatamente com o código de saída, o comando que executou e as últimas linhas impressas pelo backend, em vez de aguardar os 60 s. O código de saída 98 constitui a exceção: outro backend já detém a porta ou o ficheiro de bloqueio (da aplicação Chloros, do serviço backend ou de outro script que ainda esteja a iniciar), pelo que a chamada continua à espera que esse responda. O `ChlorosLocal` faz o mesmo.
- **O backend é deixado em execução para reutilização** (tal como na CLI) — não há um encerramento implícito quando o seu script termina. Ao executar novamente o script, o backend ativo é reutilizado.
- **Desative esta funcionalidade com `auto_start_backend=False`** em qualquer uma dessas chamadas (por exemplo, quando apontar para um backend remoto ou se gerir o ciclo de vida do backend por conta própria).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se o binário incluído não puder ser localizado ou iniciado, a chamada HTTP subsequente gera um `ChlorosConnectError` **sensível à plataforma** e passível de ação, em vez de um simples rastreio de recusa de ligação — no Windows, remete-o para a aplicação de ambiente de trabalho ou para um comando `chloros-cli`; no Linux (sem GUI), remete-o para um comando `chloros-cli` ou para o `.deb`.

---

## Ambiente e cabeçalhos

O SDK marca todas as chamadas HTTP do backend com `X-Chloros-Client: sdk`. O backend aplica as regras de licenciamento do SDK/CLI (é necessário efetuar o início de sessão **e** ter um plano Chloros+ pago), em vez do caminho da camada gratuita da GUI. Isto é definido automaticamente no momento da importação — não é necessário fazer nada.

O cabeçalho é adicionado a todas as chamadas `requests` / `urllib` cujo URL contenha `localhost` ou `127.0.0.1`, em qualquer porta — incluindo os seus próprios serviços locais — e substitui um valor que você mesmo definiu. As chamadas para outros anfitriões não são alteradas.

Substitua a URL do backend passando `backend_url=` (`ChlorosLocal` aceita `backend_url=` ou `api_url=`; o `process_folder()` e o `process_lattice_capture()` aceitam apenas o `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Um backend instalado escuta apenas no loopback, pelo que um `backend_url` da LAN não consegue aceder-lhe; consulte [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel) para o padrão de túnel.)

---

## Versões e compatibilidade

- A versão do SDK é indicada como `chloros_sdk.__version__`.
- O SDK vincula o comportamento à versão do backend incluído no pacote. Combinar um SDK mais antigo com um backend mais recente geralmente funciona (pontos de extremidade compatíveis com versões futuras), mas combinar um SDK mais recente com um backend mais antigo pode provocar erros `404` em novos pontos de extremidade — atualize a aplicação para computador para que corresponda.
- A superfície de ligação inteligente (`connect_camera` / `connect_array` / `connect_daq_sensor`) e o ponto final de análise de rede devolvem esquemas JSON estáveis; os novos campos são adicionais.

---

## Dicas para a resolução de problemas

- **`ChlorosAuthenticationError: Login required`** (proveniente de `ChlorosLocal`; as chamadas do Smart-Connect geram o erro `ChlorosConnectError: Login required. …` pela mesma causa) → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina ou inicie sessão através da aplicação de ambiente de trabalho Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (ou `Failed to create project: …` proveniente de `create_project()`) → Os caminhos longos do Windows estão desativados neste computador (configuração predefinida do Windows), pelo que nenhum caminho pode ultrapassar os 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. Nada foi processado. A mensagem indica a pasta de projeto mais longa que cabe: utilize uma mais curta ou ative os caminhos longos do Windows e reinicie o Chloros. O instalador do Windows apresenta esta opção como «Ativar caminhos longos do Windows», marcada por predefinição; caso contrário, utilize a Política de Grupo: *Configuração do Computador &gt; Modelos Administrativos &gt; Sistema &gt; Sistema de Ficheiros &gt; Ativar caminhos longos do Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → O Smart-Connect invoca o arranque automático de um backend local, pelo que isto só aparece quando o binário incluído não pode ser encontrado/iniciado (por exemplo, um hostsem pacote de ambiente de trabalho). A mensagem varia consoante a plataforma: no Windows, abra a aplicação de ambiente de trabalho ou execute qualquer comando `chloros-cli`; no Linux, execute um comando `chloros-cli` (não existe GUI) ou instale o `.deb`. Para um backend remoto, passe `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** na importação → Falha ao carregar `lattice_sdk`: o tempo de execução do SDK da Arena, ou o OpenCV (`cv2`), não é importável neste Python. A superfície orientada pelo backend (`connect_*`, projetada através de um backend, `ChlorosLocal`) continua a funcionar.
- **O Array liga-se com o sensor completo, mas com uma baixa taxa de fotogramas**→ Por definição: o modo automático mantém o sensor completo e permite que a taxa desça para o que a ligação suporta. Utilize o `binning=2` / `4` ou um valor menor, como `width` / `height`, para obter mais fotogramas por segundo, ou ative os fotogramas jumbo / atualize a ligação. O binning e a ROI**não** resolvem a sobre-subscrição agregada (`oversubscribed: true`, campos fps 0): o excesso de câmaras para a ligação não pode ser resolvido através do binning/ROI — reduza o número de câmaras, ative os quadros jumbo ou mude para uma placa de rede mais rápida (consulte [Sobresubscrição](#over-subscription-the-per-cam-floor)).
- **O `analyze_array_network()` indica que o anel de receção da placa de rede (NIC) está muito pequeno (~0,26 MB) / portas de ligação com «FRAMES WILL DROP»** → O anel de receção da placa de rede do anfitrião está na sua configuração predefinida (muitas vezes redefinido para 32 após uma atualização do controlador da placa de rede). Num adaptador Realtek USB 10 GbE, defina `ReceiveBufferLen=256` e `PendingReceives=64` (com privilégios elevados) e, em seguida, reinicie o backend para que este volte a ler o anel. Procedimento completo: [Referência da CLI → Configuração e ajuste da placa de rede do anfitrião](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **O anfitrião bloqueia ao reiniciar/desligar; posteriormente, erros WMI `Invalid class` / a placa de rede não ativa** → Controlador USB 10GbE desatualizado a causar o erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o controlador do adaptador para uma versão atual (≥ 2026) e volte a aplicar as definições do anel de receção. Consulte [Referência da CLI → Configuração e ajuste da NIC do anfitrião](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **A captura do array falha com o erro HTTP 503 «No fresh synced frame group»** → O array não publicou nenhum novo grupo de quadros durante o período de espera. Verifique se `arr.status()`: `stalled: True` (ou `streaming: False`) com `actual_fps` 0, o que significa que o ciclo de captura deixou de publicar — desligue e volte a ligar o conjunto. Se `stalled` for False e `actual_fps` for superior a 0, tente novamente. Imediatamente após o `connect_array()`, uma captura já aguarda até 20 s pelo primeiro grupo.
- **O `status()` apresenta o `link_speed_mbps: 4294`** → O Windows não conseguiu ler a velocidade do adaptador; isto é típico dos adaptadores Ethernet USB e não constitui uma falha. Consulte [Taxas de quadros esperadas](#expected-frame-rates).
- **`update_firmware()` falha com `device never connected back on TCP <port>`** → Uma firewall bloqueou a ligação do DAQ-E a este computador. O instalador do Windows adiciona a regra (consulte [Definições do dispositivo DAQ-E](#daq-e-device-settings)); com outra firewall, permita o tráfego TCP de entrada para o backend a partir da sub-rede do sensor.
- **Refletância recusada** → É necessário associar um DAQ ativo à câmara (ou matriz) para obter a refletância em escala absoluta. Efetue a associação através da GUI ou utilize o `processing="radiance"` (W/m²/sr/nm), que não requer um sensor emparelhado.
- **A captura do `smart=True` demora mais do que o esperado** → A convergência do AE depende da dinâmica da cena. Com o `ArrayHandle.capture_smart()`, aumente o intervalo do `exposure_tolerance_pct`, reduza o `stability_window_s` ou diminua o `settle_timeout_s` para obter um disparo mais rápido e menos estabilizado.

---

## Ver também

- [Referência da CLI](cli-reference.md) — cada subcomando da CLI corresponde a uma chamada do SDK.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) no manual do Chloros — os sensores, os seus limites e intervalo calibrado, gravação, ligação em rede DAQ-E.
- [Câmaras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) no manual do Chloros — ligação, definições da câmara, modos de captura, matrizes.
- Documentação online: `https://mapir.gitbook.io/chloros/api-python-sdk`
