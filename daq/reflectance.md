# Fluxos de trabalho de refletância

Um sensor de luz DAQ transforma imagens radiométricas em refletância. Existem dois fluxos de trabalho distintos:

1. **Sensor único** — um DAQ mede a irradiância descendente enquanto uma câmara captura a imagem, e o Chloros divide a radianância da câmara por essa referência.
2. **Sensor duplo** — dois sensores DAQ, um a monitorizar o céu e outro a monitorizar um objeto, produzem uma curva de refletância espectral em tempo real sem o envolvimento de uma câmara.

## Sensor único + câmara (referência descendente)

O DAQ funciona como sensor de luz descendente (DLS): a câmara mede a radianância ascendente **L**(W/m²/sr/nm), o DAQ mede a irradiância descendente**E** (W/m²/nm) e o Chloros calcula a refletância por banda da seguinte forma:

> ρ = π · L / E

A leitura do DAQ está sempre **sincronizada com a exposição** — é por isso que o DAQ e as câmaras partilham um relógio sincronizado por PTP (ver [Rede e Sincronização de Tempo do DAQ-E](ethernet-ptp.md)). Ajusta a capa cosseno Sunshine para trabalhos ao ar livre e declara-a corretamente; a declaração da capa dimensiona diretamente E (ver [Perfis de Capa e Intervalo Calibrado](caps-and-range.md)). Para trabalhos quantitativos, lembre-se da característica do instrumento: a irradiância quantitativa resulta de uma média de, pelo menos, 15 s de leituras.

### Captura em tempo real

Associe o DAQ a uma câmara no separador «Câmaras»: o painel de definições de cada câmara possui um menu suspenso **Sensor de Luz** que lista todos os DAQ ligados (DAQ-U/M/E) a partir do separador «Sensores de Luz»; para um conjunto sincronizado, a seleção de um «Sensor de Luz» para todo o conjunto propaga-se a todos os membros (as câmaras individuais ainda podem substituir essa configuração). Uma vez associado, os espectros do sensor alimentam a ranhura DLS da câmara e as exportações de refletância são divididas pela leitura correspondente. Numa matriz, um membro com o seu próprio sensor utiliza a sua própria leitura — para as suas bandas na composição em tempo real, as suas exportações e a sua exposição automática — e os outros membros utilizam a do conjunto.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dois comportamentos que vale a pena conhecer:

* **Sem ligação ao DAQ → a refletância é recusada, não simulada.** O Chloros rejeita o produto de refletância e regista o motivo da omissão, em vez de devolver silenciosamente um produto inferior.
* **A leitura utilizada é preservada.** Para cada fotograma de refletância, a leitura do DAQ efetivamente aplicada é gravada como um ficheiro `.daq` na pasta `daq/` das capturas, para que a captura possa ser reprocessada posteriormente ([Gravação e o formato .daq](recording.md)). Um ficheiro `.daq.cameras.json` ao lado identifica as câmaras a que se refere, e o reprocessamento associa-o apenas às imagens dessas câmaras.

### Processamento de imagens gravadas

Para o processamento pós-voo, grave um ficheiro `.daq` durante a sessão e guarde-o juntamente com as imagens — o pipeline resolve automaticamente a luz descendente com correspondência de data e hora, obtendo qualquer calibração de fábrica em falta a partir da nuvem dMAPIR
na primeira utilização. As gravações da GUI são adicionadas automaticamente ao projeto aberto quando terminam.

A referência de refletância é selecionável no momento do processamento — `--reflectance-source` em `chloros-cli process`, ou a definição da fonte de refletância nas «Definições do Projeto» da GUI:

| Valor | Comportamento |
| --- | --- |
| `auto` (predefinição) | Um alvo de calibração dentro do quadro que tenha passado no controlo de qualidade (QA) é a referência absoluta; a calibração descendente do DAQ (ρ = π·L/E) é a alternativa. Um alvo que passe no controlo de qualidade com um sinalizador (um painel em falta ou uma banda fora de ordem) não é automaticamente preferido: as cadeias «alvo», «alvo+DAQ» e «apenas DAQ» são testadas por ordem da pontuação medida em painéis não utilizados. |
| `daq` | DAQ como autoridade |
| `target` | Alvo estritamente dentro do quadro; sem substituição por DAQ |

Consulte [Alvos de Calibração](../calibration-targets.md) para fluxos de trabalho de alvos e o [capítulo LATTICE](../lattice/README.md), bem como a [Referência CLI](../reference/cli-reference.md) para o pipeline de processamento completo. Ao ler os píxeis de refletância exportados, utilize a escala indicada (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — consulte [Formatos de imagem de saída](../output-image-formats.md).

### Bandas fora da gama calibrada do DAQ

A faixa radiometricamente calibrada do DAQ é de ~374–974 nm. O Chloros rejeita a refletância baseada no DAQ para qualquer banda da câmara com menos de metade do seu peso espectral dentro desse intervalo, indicando o motivo de omissão `dls-uncalibrated-band-<nm>`. Entre os SKUs disponíveis, isto afeta apenas a F988: a refletância da F988 é calibrada utilizando um painel de refletância na cena; como a banda se situa fora da gama calibrada do sensor de luz do DAQ, o Chloros aplica a sua captura mais recente do painel e mantém-na entre as observações do painel. Se uma câmara F988 for utilizada apenas com o DAQ, o Chloros rejeita a refletância baseada no DAQ para essa banda com o motivo de omissão `dls-uncalibrated-band-988` — o fluxo de trabalho com o painel é a via suportada.

## Sensor duplo (ambiente + objeto)

Dois sensores DAQ — qualquer par, em qualquer tipo de transporte — fornecem um espectro de refletância em tempo real sem necessidade de câmara: um sensor está virado para o céu (**Fonte de Luz Ambiente**), outro está virado para o objeto (**Scanner de Objeto**), e o Chloros calcula por comprimento de onda:

> R(λ) = objeto(λ) / ambiente(λ)

(zero quando ambiente ≤ 0).

### Na interface gráfica

Com ambos os sensores ligados no separador «Sensores de Luz», abra a sobreposição para adicionar sensores (o botão «+» num bloco de gráfico na vista em grelha) e selecione **Combinar Ambiente + Objeto**. Selecione os dois sensores nos menus suspensos**Fonte de Luz Ambiente**e**Scanner de Objetos**e clique em**Criar**. O grupo aparece como um gráfico próprio e como uma linha na barra lateral com um emblema verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Abaixo do gráfico de refletância (visualização em lista), uma **tabela de índices de vegetação** em tempo real calcula os índices a partir da curva utilizando os centros das bandas em azul 450 / verde 550 / vermelho 670 / NIR 800 nm. Os índices baseados em rácios que anulam a escala absoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) são sempre apresentados; os índices que requerem refletância absoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) só aparecem quando ambos os sensores são modelos calibrados em potência.

### Aparente vs. Relativa — a regra de rotulagem O

Chloros rotula a saída do sensor duplo de acordo com o que o par de sensores pode efetivamente afirmar:

| Par de sensores | Rotulagem |
| --- | --- |
| Ambos os sensores calibrados — pacote de fábrica carregado | **Refletância Aparente** |
| Qualquer um dos sensores não calibrado | **Refletância Relativa** |

Todos os três modelos são radiométricos: assim que o pacote de calibração de fábrica de um sensor é carregado, os seus espectros são absolutos em W/m²/nm; por isso, um par de sensores calibrados resulta numa refletância aparente absoluta — o transporte não determina isso. Um sensor que continue a transmitir contagens brutas (sem pacote acessível) reclassifica o resultado para uma curva relativa (a forma espectral continua válida). Ambos os sensores devem possuir limites declarados corretamente ([Perfis de Limite e Intervalo Calibrado](caps-and-range.md)).

### A partir de Python


Não existe uma chamada dedicada para dois sensores na interface do SDK agrupado: abra duas sessões com o `chloros_sdk.connect_daq_sensor()` e faça você mesmo a relação entre os seus espectros `latest()`, aplicando a mesma convenção de rotulagem. (Existe também uma ferramenta de gravação para dois sensores na interface interna de acesso direto ao hardware da MAPIR, listada na [Referência da CLI](../reference/cli-reference.md) por uma questão de exaustividade — não faz parte da CLI fornecida; o fluxo de trabalho da GUI acima é o caminho ativo suportado.)
