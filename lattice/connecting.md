# Ligar as câmaras

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>O separador «Câmaras» antes de qualquer dispositivo estar ligado</p></figcaption></figure>O Chloros deteta automaticamente as câmaras LATTICE na ligação — a partir do separador «Câmaras» da interface gráfica, a partir do `chloros-cli lattice`, ou a partir do Python SDK. A string do modelo da câmara determina todos os passos seguintes: O Chloros determina o perfil do sensor, a configuração das bandas e a calibração de fábrica a partir dos parâmetros `DeviceUserID` + `DeviceSerialNumber` da câmara, pelo que **não há nada a configurar em cada câmara**.

Antes de ligar, certifique-se de que a rede anfitriã está configurada — endereçamento link-local, quadros jumbo e, no caso de matrizes, as definições do buffer de receção da placa de rede. Essa é a configuração do lado do hardware e encontra-se no manual da LATTICE: [**Configuração de Rede**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Ligar a partir da GUI

Abra o separador **Câmaras**na barra lateral do Chloros (os separadores de hardware aparecem assim que o backend terminar de iniciar) ou utilize o menu principal →**Ligar à Câmara**. Ambas as opções abrem a caixa de diálogo**Ligar Câmara(s)**.

### A caixa de diálogo **Ligar Câmara(s)**A caixa de diálogo analisa a rede assim que é aberta («A analisar a rede...») e apresenta todas as câmaras que encontra. Cada linha mostra o**modelo**da câmara (por exemplo, `LATT-M3M-L41-F550`), o**número de série**e o**endereço IP**.

* **Clique numa linha para a selecionar**(destaque a verde). Pode selecionar**várias câmaras** e ligá-las de uma só vez — a Chloros liga-as sequencialmente.
* As linhas com o ícone **«Conectado»** já estão conectadas e não podem ser selecionadas novamente.
* As linhas com o ícone **«Em conjunto»** pertencem a um conjunto de câmaras atualmente conectado. Desconecte primeiro o conjunto para utilizar essa câmara de forma independente.
* **Ligar** — liga a(s) câmara(s) selecionada(s); o botão mostra uma contagem, por exemplo, «Ligar (3)», quando mais do que uma está selecionada.
* **Reexecutar pesquisa** — executa novamente a pesquisa.
* **Fechar** — fecha a caixa de diálogo.
* Se a pesquisa terminar sem resultados, a caixa de diálogo exibe **«Nenhuma câmara encontrada na rede»** — consulte [Resolução de problemas](connecting.md#troubleshooting) abaixo.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>A caixa de diálogo «Ligar câmara(s)» — aqui apresentada sem câmaras na rede</p></figcaption></figure>### Primeira ligação: transferência do pacote de calibração

Na **primeira vez**que uma determinada câmara é ligada a um equipamento, o Chloros obtém o pacote de calibração de fábrica da câmara (\~3,8 MB) diretamente da própria câmara através de GigE. Enquanto este processo decorre, a caixa de diálogo apresenta um painel verde com a mensagem**«A descarregar dados de calibração da câmara»**, acompanhado de uma barra de progresso por número de série — o tempo estimado é de cerca de**70 segundos** por câmara. O pacote é armazenado em cache no anfitrião, pelo que as ligações posteriores da mesma câmara ignoram completamente a descarga (e nunca apresentam o painel).

### Analisar Sistema

O botão **Analisar Sistema** da caixa de diálogo examina o anfitrião e a rede (a legenda indica «A analisar...» enquanto o processo decorre) e gera um relatório de diagnóstico:

* **Anfitrião** — núcleos da CPU e RAM; nome e memória da GPU, ou «GPU: Nenhuma detetada».
* **Interfaces de rede** — nome de cada placa de rede, velocidade de ligação, MTU (com a indicação «jumbo» quando ativa), estado de ativação/desativação e se está ligada a um barramento USB.
* **Câmaras**— número de série, modelo, IP e**em que placa de rede cada câmara está ligada**.
* **Desempenho** — fps atuais vs. ideais por câmara para o formato de píxeis, com uma linha verde «Potencial: melhoria de N× possível» quando o ideal excede o valor atual.
* **Avisos e recomendações numeradas** — ou «O sistema parece estar bem para o número atual de câmaras», quando não há nada a corrigir.

Execute-o sempre que a deteção ou a transmissão se comportarem de forma inesperada — identifica a maioria dos problemas do lado da placa de rede (MTU errada, câmara na interface errada, limites do adaptador USB) sem sair da caixa de diálogo.

### Ligar um conjunto

Para ligar duas ou mais câmaras como um **conjunto sincronizado**, utilize o assistente de ligação de conjuntos (**Ligar Conjunto de Câmaras**): este guia-o pela seleção de mestre/escravo (pré-preenchida por uma sonda de ligação GPIO), pela escolha do modo de exibição (mosaicos separados vs. combinados) e por uma cena de configurações do conjunto com uma projeção em tempo real dos fps alcançáveis e da largura de banda da ligação antes de confirmar. O assistente e os fluxos de trabalho de matrizes são abordados na secção sobre matrizes de múltiplas câmaras deste manual; o equivalente no CLI é o «Fluxo de Trabalho de Primeira Ligação da Câmara LATTICE» na [Referência do CLI](../reference/cli-reference.md).

## Ligação a partir do CLI e do SDK

O acesso ao CLI e ao SDK requer um plano Chloros+ pago e estar com sessão iniciada; isto é aplicado do lado do servidor (`401 AUTH_REQUIRED` quando não se está com a sessão iniciada, `403 PLAN_UPGRADE_REQUIRED` no plano gratuito).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Assinaturas completas, opções e fluxos de trabalho de captura: [Referência CLI](../reference/cli-reference.md) § `chloros-cli lattice`, [Referência SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Como a calibração é resolvida na ligação

Todas as câmaras LATTICE incluem o seu pacote de calibração de fábrica **na própria câmara**, e a Chloros também verifica a nuvem da MAPIR quando a câmara se liga:

| Situação   | O que a Chloros utiliza                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| A**calibração mais recente publicada para esse número de série** — a cópia na nuvem prevalece sobre a cópia na câmara. Uma câmara que tenha sido recalibrada ou atualizada pelo MAPIR atualiza-se, portanto, automaticamente; não é necessária qualquer ação do utilizador. |
| **Offline**| O**pacote na câmara**, tal como está. Os fluxos de trabalho totalmente offline continuam a funcionar; simplesmente não incorporam as calibrações mais recentes até que a câmara fique online pelo menos uma vez (ou seja reatualizada de fábrica).                                                  |

No momento da captura, os coeficientes efetivamente aplicados são **congelados nos metadados XMP de cada imagem**. Uma atualização de calibração posterior nunca altera silenciosamente as imagens que já capturou — o reprocessamento de uma captura antiga utiliza os coeficientes gravados nos seus metadados XMP, e não os mais recentes do momento.

## Resolução de problemas

* **«Não foram encontradas câmaras na rede&quot;**— verifique a configuração de ligação local em [Configuração de Rede](https://mapir.gitbook.io/lattice-camera/setup/network-setup): NIC do anfitrião estática `169.254.x.x/16`, câmaras na mesma ligação, não é esperado DHCP/gateway. Em seguida, utilize**Analisar Sistema**na caixa de diálogo de ligação para verificar em que placa de rede cada câmara está (ou não está) visível.**Volte a procurar** após qualquer alteração no cabeamento ou na placa de rede.
* **Um equipamento que antes funcionava recusa-se a ligar-se** (portas do painel de matriz com `FRAMES WILL DROP` / `Reduce ROI to enable`) — uma atualização do controlador da placa de rede redefiniu silenciosamente as definições do anel de receção. Reaplique-as ou execute o comando `chloros-cli lattice network --fix` a partir de um terminal com privilégios elevados; consulte [Configuração de Rede](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Uma câmara apresenta a mensagem «In Array»** — pertence a uma sessão de matriz ligada. Desligue a matriz para utilizar a câmara de forma autónoma.
