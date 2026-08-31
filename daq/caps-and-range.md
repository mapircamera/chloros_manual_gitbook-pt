# Perfis das tampas e intervalo calibrado

> As próprias tampas — quais as tampas fornecidas com cada sensor, como são montadas e o seu comportamento ótico — estão documentadas no **[manual do utilizador do DAQ](https://mapir.gitbook.io/daq)**. Esta página aborda a *declaração* da tampa instalada no Chloros, o que é essencial para que a correção seja correta.

A calibração radiométrica de fábrica de cada sensor de luz do DAQ descreve o sensor *nu*. A tampa física instalada sobre o difusor altera a luz que o sensor capta, pelo que o Chloros aplica um **perfil de correção da tampa** medido de fábrica, além do pacote de calibração. A declaração da tampa correta faz parte da obtenção de dados calibrados — esta página aborda quais as tampas existentes por modelo, como declará-las e qual é, na realidade, a gama espectral calibrada do sensor.

## Disponibilidade de tampas por modelo

| Perfil da tampa (`cap_id`) | Tampa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tampa corretora de cosseno para luz solar (**padrão em todos os modelos**) | Sim | Sim | Sim |
| `fov_15` / `fov_45` / `fov_90` | Cones de restrição do campo de visão (FOV) (15° / 45° / 90°) | Sim | — | Sim |
| `fov_30` / `fov_60` | Cones de restrição do campo de visão (30° / 60°) | Sim | — | — |
| `none` | Sem tampa instalada | — | — | Sim |

Notas específicas do modelo:

* **O DAQ-M tem um único perfil de tampa: `sunshine_cosine`.** «Bare-plus-Sunshine-cap» é a sua definição de produto, e um DAQ-M sem tampa não necessita de perfil geométrico.
* **Um DAQ-U sem tampa é verdadeiramente «bare»** — não necessita de qualquer perfil geométrico, razão pela qual não existe um perfil `none` para ele.
* **O `none` num DAQ-E NÃO é uma operação nula.** O difusor recuado e coberto de vidro do DAQ-E possui uma correção geométrica real própria, pelo que «sem tampa» é, por si só, um perfil medido neste modelo.
* Um **DAQ-E sem acessórios não consegue medir a luz solar direta a qualquer elevação** — a tampa Sunshine é a configuração de campo. Não planeie trabalhos ao ar livre com um DAQ-E sem acessórios.

Nas definições por sensor da GUI (ícone de engrenagem no separador «Sensores de Luz»), o menu pendente **Tampa** também oferece a opção «Nenhuma (sensor sem tampa)» no DAQ-U e no DAQ-M — nesses dois modelos, «sem tampa» significa simplesmente que não é aplicada qualquer correção de tampa, conforme as notas acima. Selecione esta opção apenas quando a tampa estiver fisicamente removida.

## Declaração da tampa — e por que é importante

**O código `cap_id` declarado deve corresponder à tampa que está fisicamente colocada no sensor.** Nem o sensor nem o software conseguem detetar a tampa instalada. A declaração determina duas coisas:

1. A **correção em tempo real** aplicada a cada espectro.
2. O **código da tampa gravado em cada registo `.daq`**, no qual o processamento de refletância a jusante se baseia.

A tampa Sunshine atenua em cerca de **12×, por conceção**; por isso, gravar com a tampa errada declarada distorce a escala dos espectros aproximadamente nesse fator. Declare imediatamente as alterações de tampa.

### Definir a tampa

GUI: separador «Sensores de luz» → ícone de engrenagem na linha do sensor → menu suspenso **Tampa**. O valor predefinido para todos os modelos é `sunshine_cosine` (todos os sensores DAQ são fornecidos com o corretor de cosseno instalado), e a seleção mantém-se no projeto.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (o backend tem de estar em execução):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

O CLI aceita sintaticamente a lista completa de `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil é validado em relação ao modelo do sensor no momento da ligação, pelo que um ID de cap não disponível (por exemplo, um ID «E-only» num DAQ-U) resulta num erro claro, em vez de uma correção incorreta. O valor predefinido do backend, quando nada é passado, é o `sunshine_cosine`.

Python SDK nota: `cap_id` **não** é um regulador SDK — `connect_daq_sensor()` / `DAQSensorSession` não expõem nenhum parâmetro de cap. Selecione o limite através dos comandos CLI acima ou do menu suspenso da GUI; consulte a [Referência do SDK](../reference/sdk-reference.md).

Avançado: os perfis são fornecidos na instalação do Chloros em `daq/cap_profiles/<u|m|e>/<cap_id>.json` e podem ser substituídos por cada utilizador em `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Independentemente dos limites, os sensores que nunca foram recalibrados recebem automaticamente um pequeno ajuste do desvio de escuro derivado da frota — sem qualquer intervenção do utilizador.

## Desempenho do limite de luz solar (a configuração para o exterior)

Valores em que pode basear os seus procedimentos:

| Propriedade | Valor |
| --- | --- |
| Campo de visão | Hemisférico de 180° |
| Erro de resposta cosseno | ≤ ±4 % até 60° de incidência; ≤ ±4,5 % até 70° |
| Limite de sol baixo | Não recomendado abaixo de ~15° de elevação solar |
| Atenuação | ~12× (por conceção) |
| Repetibilidade da remontagem da capa | ≈ 1,5 % |
| Irradiação quantitativa | Média de **≥ 15 s** de leituras (característica do instrumento, não constitui um defeito) |

Para qualquer valor de irradiação quantitativa — incluindo referências de refletância — utilize uma média de, pelo menos, 15 segundos de leituras, em vez de um único quadro.

## Intervalo espectral calibrado

| Propriedade | Valor |
| --- | --- |
| Amostragem espectral | 340–1010 nm em intervalos de 5 nm (135 pontos) |
| Intervalo calibrado radiometricamente | **~374–974 nm** (imposto pelo software) |

O sensor reporta a grelha completa de 340–1010 nm, mas o ganho radiométrico rastreável pelo NIST abrange ~374–974 nm. O Chloros **recusa a divisão por refletância absoluta** para qualquer banda da câmara com menos de metade do seu peso espectral dentro desse intervalo, indicando o motivo de omissão `dls-uncalibrated-band-<nm>` em vez de produzir um produto não calibrado. Entre os SKUs de câmaras disponíveis no mercado, apenas o filtro F988 fica fora deste intervalo; este utiliza, em vez disso, o fluxo de trabalho com painéis de refletância — consulte [Fluxos de Trabalho de Refletância](reflectance.md).

Para modelos de sensores, transportes e IDs de sensores, consulte a [visão geral do DAQ](README.md). Para saber como o carimbo de limite máximo é consumido durante o processamento, consulte [Gravação e o formato .daq](recording.md).
