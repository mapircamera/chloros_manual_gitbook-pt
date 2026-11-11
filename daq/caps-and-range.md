# Perfis das tampas e intervalo calibrado

> As próprias tampas — quais as que vêm com cada sensor, como são montadas e o seu comportamento ótico — estão documentadas no **[manual do utilizador do DAQ](https://mapir.gitbook.io/daq)**. Esta página aborda a *declaração* da tampa instalada ao Chloros, o que é essencial para que a correção seja correta.

A calibração radiométrica de fábrica de cada sensor de luz DAQ descreve o sensor *nu*. A tampa física colocada sobre o difusor altera a luz que o sensor capta, pelo que o Chloros aplica um **perfil de correção da tampa** medido de fábrica sobre o conjunto de calibração. Declarar a tampa correta faz parte do processo de obtenção de dados calibrados — esta página aborda quais as tampas existentes por modelo, como declará-las e qual é, na realidade, a gama espectral calibrada do sensor.

## Disponibilidade de tampas por modelo

| Perfil da tampa (`cap_id`) | Tampa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tampa corretora de cosseno para luz solar (**padrão em todos os modelos**) | Sim | Sim | Sim |
| `fov_15` / `fov_45` / `fov_90` | Cones de restrição do campo de visão (15° / 45° / 90°) | Sim | — | Sim |
| `fov_30` / `fov_60` | Cones de restrição do campo de visão (30° / 60°) | Sim | — | — |
| `none` | Sem tampa instalada | — | — | Sim |

Notas específicas do modelo:

* **O DAQ-M tem um único perfil de tampa: `sunshine_cosine`.** «Bare-plus-Sunshine-cap» é a sua definição de produto, e um DAQ-M «bare» não necessita de perfil geométrico.
* **Um DAQ-U «bare» é verdadeiramente «bare»** — não necessita de qualquer perfil geométrico, razão pela qual não existe um perfil `none` para ele.
* **O `none` num DAQ-E NÃO é uma operação nula.** O difusor recuado e coberto de vidro do DAQ-E possui uma correção geométrica real própria, pelo que «sem tampa» é, por si só, um perfil medido neste modelo.
* Um **DAQ-E sem acessórios não consegue medir a luz solar direta a qualquer elevação** — a tampa Sunshine é a configuração de campo. Não planeie trabalhos ao ar livre com um DAQ-E sem acessórios.

Nas definições por sensor da GUI (ícone de engrenagem no separador «Sensores de Luz»), o menu suspenso **Tampa** também oferece a opção «Nenhuma (sensor sem tampa)» no DAQ-U e no DAQ-M — nesses dois modelos, «sem tampa» significa simplesmente que não é aplicada qualquer correção de tampa, conforme as notas acima. Selecione esta opção apenas quando a tampa estiver fisicamente removida.

## Declaração da tampa — e por que é importante

**O código `cap_id` declarado deve corresponder à tampa que está fisicamente colocada no sensor.** Nem o sensor nem o software conseguem detetar a tampa instalada. A declaração determina duas coisas:

1. A **correção em tempo real** aplicada a cada espectro.
2. A **marca da tampa gravada em cada registo `.daq`**, na qual o processamento de refletância a jusante se baseia.

A tampa Sunshine atenua em cerca de **12×, por design**; por isso, registar com a tampa errada declarada distorce a escala dos espectros aproximadamente nesse fator. Declare imediatamente as alterações de tampa.

### Definir a tampa

GUI: separador «Light Sensors» → ícone de engrenagem na linha do sensor → menu suspenso **Cap**. O valor predefinido para todos os modelos é `sunshine_cosine`, uma suposição (os modelos DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e uma gravação indica que a tampa foi assumida até que se escolha uma), e a seleção mantém-se no projeto.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(o backend tem de estar a funcionar):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

A CLI aceita sintaticamente a lista completa `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil é validado em relação ao modelo do sensor no momento da ligação, pelo que um ID de limite indisponível (por exemplo, um ID exclusivo do E num DAQ-U) resulta num erro claro, em vez de uma correção incorreta. O valor predefinido do backend, quando nada é passado, é `sunshine_cosine`.

Python SDK: nomeie o cap no momento da ligação com `connect_daq_sensor(cap_id="sunshine_cosine")`, ou altere-o em tempo de execução com `DAQSensorSession.set_cap("fov_45")`; consulte a [Referência do SDK](../reference/sdk-reference.md#daq-sensor-sessions).

Avançado: os perfis são fornecidos na instalação do Chloros em `daq/cap_profiles/<u|m|e>/<cap_id>.json` e podem ser substituídos por cada utilizador em `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Independentemente dos limites, os sensores que nunca foram recalibrados recebem automaticamente um pequeno ajuste de desvio de escuro derivado da frota — sem qualquer intervenção do utilizador.

## DAQ-E-S: difusor fixo e inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças que são relevantes para o Chloros. Tudo o resto (detecção, ligação, PTP, gravação, definições do dispositivo) funciona exatamente como num DAQ-E.

* **O seu difusor é permanente, pelo que não requer correção de tampa.** O difusor já se encontrava na unidade quando foi realizada a sua calibração de fábrica, pelo que essa calibração já indica a irradiação descendente real. O Chloros não lhe aplica qualquer perfil de limite, independentemente do que seja solicitado: um limite com o seu nome é ignorado com uma mensagem, os seus registos indicam que o limite provém do modelo e o aviso de limite presumido nunca é acionado para este dispositivo. Em Definições do Projeto → Sensor de Luz DAQ, a sua linha não apresenta qualquer limite, e a substituição do limite a nível do projeto não o afeta. Não aplique manualmente um limite aos seus dados: a correção Sunshine resultante apresentaria um valor cerca de 11 vezes superior ao real.
* **Possui um acelerómetro de 3 eixos.** A resposta cosseno do difusor só é válida enquanto este estiver nivelado; por isso, um sensor inclinado reporta uma irradiância errada sem apresentar qualquer outro sintoma. Cada leitura do DAQ-E-S inclui, portanto, a sua inclinação, rotação e inclinação longitudinal, e um `.daq` gravado a partir de um desses sensores armazena esses valores junto de cada espectro (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), permitindo distinguir uma nuvem de uma montagem desequilibrada.

Para cada campo de atitude e para saber como é comunicada uma falha no fluxo de atitude, consulte [DAQ-E-S na Referência da CLI](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Desempenho do Sunshine Cap (configuração ao ar livre)

Valores em que pode basear os seus procedimentos:

| Propriedade | Valor |
| --- | --- |
| Campo de visão | Hemisférico de 180° |
| Erro de resposta cosinusoidal | ≤ ±4 % até 60° de incidência; ≤ ±4,5 % até 70° |
| Limite de sol baixo | Não recomendado abaixo de ~15° de elevação solar |
| Atenuação | ~12× (por conceção) |
| Repetibilidade da remontagem da tampa | ≈ 1,5 % |
| Irradiação quantitativa | Média de **≥ 15 s** de leituras (característica do instrumento, não constitui um defeito) |

Para qualquer valor de irradiação quantitativa — incluindo referências de refletância — utilize uma média de, pelo menos, 15 segundos de leituras, em vez de um único fotograma.

## Gama espectral calibrada

| Propriedade | Valor |
| --- | --- |
| Amostragem espectral | 340–1010 nm em intervalos de 5 nm (135 pontos) |
| Intervalo calibrado radiometricamente | **~374–974 nm** (imposto pelo software) |

O sensor reporta a grelha completa de 340–1010 nm, mas o ganho radiométrico rastreável ao NIST abrange ~374–974 nm. O Chloros **recusa a divisão de refletância absoluta** para qualquer banda da câmara com menos de metade do seu peso espectral dentro desse intervalo, indicando o motivo de omissão `dls-uncalibrated-band-<nm>` em vez de produzir um produto não calibrado. Entre os modelos de câmara disponíveis no mercado, apenas o filtro F988 fica fora deste intervalo; este utiliza, em vez disso, o fluxo de trabalho do painel de refletância — consulte [Fluxos de Trabalho de Refletância](reflectance.md).

Para modelos de sensores, transportes e IDs de sensores, consulte a [visão geral do DAQ](README.md). Para saber como o carimbo de limite máximo é utilizado durante o processamento, consulte [Gravação e o formato .daq](recording.md).
