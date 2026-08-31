# Fluxos de trabalho de refletância

Um sensor de luz DAQ converte imagens radiométricas em refletância. Existem dois fluxos de trabalho distintos:

1. **Sensor único** — um DAQ mede a irradiância descendente enquanto uma câmara captura a imagem, e o Chloros divide a radiancia da câmara por essa referência.
2. **Sensor duplo** — dois sensores DAQ, um a observar o céu e outro a observar um objeto, produzem uma curva de refletância espectral em tempo real sem o envolvimento de uma câmara.

## Sensor único + câmara (referência descendente)

O DAQ atua como sensor de luz descendente (DLS): a câmara mede a radianância ascendente **L**(W/m²/sr/nm), o DAQ mede a irradiância descendente**E** (W/m²/nm) e o Chloros calcula a refletância por banda da seguinte forma:

> ρ = π · L / E

A leitura do DAQ está sempre **sincronizada com o carimbo de data/hora da exposição** — é por isso que o DAQ e as câmaras partilham um relógio regulado por PTP (ver [Rede e Sincronização de Tempo do DAQ-E](ethernet-ptp.md)). Aplique o limite cosseno Sunshine para trabalhos ao ar livre e declare-o corretamente; a declaração do limite escala diretamente E (ver [Perfis de Limite e Intervalo Calibrado](caps-and-range.md)). Para trabalhos quantitativos, tenha em conta a característica do instrumento: a irradiância quantitativa resulta de uma média de, pelo menos, 15 s de leituras.

### Captura em tempo real

Associe o DAQ a uma câmara no separador «Câmaras»: o painel de definições de cada câmara possui um menu suspenso **Sensor de Luz** que lista todos os DAQ ligados (DAQ-U/M/E) a partir do separador «Sensores de Luz»; para um conjunto sincronizado, a seleção de um Sensor de Luz para todo o conjunto propaga-se a todos os membros (as câmaras individuais ainda podem substituir esta configuração). Uma vez associado, os espectros do sensor alimentam a ranhura DLS da câmara e as exportações de refletância são divididas pela leitura correspondente.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dois comportamentos que vale a pena conhecer:

* **Sem DAQ associado → a refletância é recusada, não simulada.** O Chloros rejeita o produto de refletância e regista o motivo da omissão, em vez de devolver silenciosamente um produto inferior.
* **A leitura utilizada é preservada.** Para cada fotograma de refletância, a leitura do DAQ efetivamente aplicada é gravada como um ficheiro auxiliar `.daq` junto às imagens, para que a captura possa ser reprocessada posteriormente ([Gravação e o formato .daq](recording.md)).

### Processamento de imagens gravadas

Para o processamento pós-voo, grave um `.daq` durante a sessão e mantenha-o juntamente com as imagens — o pipeline resolve automaticamente a luz descendente com correspondência de data e hora, obtendo qualquer calibração de fábrica em falta a partir da nuvem do MAPIR na primeira utilização. As gravações da GUI são adicionadas automaticamente ao projeto aberto quando terminam.

A referência de refletância pode ser selecionada no momento do processamento — `--reflectance-source` no `chloros-cli process`, ou a definição da fonte de refletância nas «Definições do Projeto» da GUI:

| Valor | Comportamento |
| --- | --- |
| `auto` (predefinição) | Um alvo de calibração dentro do quadro que passe no controlo de qualidade (QA) é a referência absoluta; a radiação descendente do DAQ (ρ = π·L/E) é a alternativa |
| `daq` | Autoridade do DAQ |
| `target` | Alvo estritamente dentro do enquadramento; sem substituição pelo DAQ |

Consulte [Alvos de calibração](../calibration-targets.md) para conhecer os fluxos de trabalho dos alvos e o [capítulo LATTICE](../lattice/README.md), bem como a [Referência CLI](../reference/cli-reference.md) para o pipeline de processamento completo. Ao ler os píxeis de refletância exportados, utilize a escala indicada (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — consulte [Formatos de imagem de saída](../output-image-formats.md).

### Bandas fora da gama calibrada do DAQ

A gama radiometricamente calibrada do DAQ é de ~374–974 nm. O Chloros rejeita a refletância baseada no DAQ para qualquer banda da câmara com menos de metade do seu peso espectral dentro desse intervalo, indicando o motivo de omissão `dls-uncalibrated-band-<nm>`. Entre os SKUs disponíveis, isto afeta apenas a F988: a refletância da F988 é calibrada utilizando um painel de refletância na cena; a banda situa-se fora da faixa calibrada do sensor de luz do DAQ, pelo que o Chloros aplica a sua captura mais recente do painel e mantém-na entre as observações do painel. Se uma câmara F988 for utilizada apenas com o DAQ, o código Chloros rejeita a refletância baseada no DAQ para essa banda com o motivo de omissão `dls-uncalibrated-band-988` — o fluxo de trabalho com o painel é o método recomendado.

## Sensor duplo (ambiente + objeto)

Dois sensores de aquisição de dados — qualquer par, em qualquer meio de transporte — fornecem um espectro de refletância em tempo real sem necessidade de câmara: um sensor está virado para o céu (**Fonte de Luz Ambiente**), outro está virado para o objeto (**Scanner de Objetos**), e o Chloros calcula por comprimento de onda:

> R(λ) = objeto(λ) / ambiente(λ)

(zero quando ambiente ≤ 0).

### Na interface gráfica

Com ambos os sensores ligados no separador «Sensores de luz», abra a sobreposição para adicionar sensores (o botão «+» num bloco de gráfico na vista em grelha) e selecione **Combinar ambiente + objeto**. Selecione os dois sensores nos menus suspensos «Fonte de Luz Ambiente» e «Scanner de Objetos» e clique em «Criar». O grupo aparece como um gráfico próprio e como uma linha na barra lateral com um emblema verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Abaixo do gráfico de refletância (visualização em lista), uma **tabela de índices de vegetação** em tempo real calcula os índices a partir da curva, utilizando os centros das bandas em azul 450 / verde 550 / vermelho 670 / NIR 800 nm. Índices baseados em rácios que anulam a escala absoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) são sempre apresentados; os índices que requerem refletância absoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) aparecem apenas quando ambos os sensores são modelos com calibração de potência.

### Aparente vs. Relativa — a regra de rotulagem

O Chloros rotula a saída do sensor duplo de acordo com o que o par de sensores pode efetivamente indicar:

| Par de sensores | Rotulagem |
| --- | --- |
| Ambos os sensores calibrados — pacote de fábrica carregado | **Refletância aparente** |
| Qualquer um dos sensores não calibrado | **Refletância Relativa** |

Todos os três modelos são radiométricos: assim que o pacote de calibração de fábrica de um sensor é carregado, os seus espectros são absolutos em W/m²/nm; por isso, um par de sensores calibrados resulta numa refletância aparente absoluta — o transporte não determina isso. Um sensor que continue a transmitir contagens brutas (sem pacote acessível) reclassifica o resultado para uma curva relativa (a forma espectral continua válida). Ambos os sensores devem possuir limites declarados corretamente ([Perfis de Limites e Gama Calibrada](caps-and-range.md)).

### A partir de Python

Não existe uma chamada dedicada para dois sensores na superfície agrupada SDK: abra duas sessões com o `chloros_sdk.connect_daq_sensor()` e compare os respetivos espectros `latest()`, aplicando a mesma convenção de rotulagem. (Existe também uma ferramenta de gravação com dois sensores na superfície interna de hardware direto do MAPIR, listada na [Referência do CLI](../reference/cli-reference.md) para completar — não faz parte do CLI fornecido; o fluxo de trabalho da GUI acima é o caminho ativo suportado.)
