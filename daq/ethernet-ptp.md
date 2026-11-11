# Rede e sincronização de tempo do DAQ-E

> A configuração física da rede para o sensor — cablagem, PoE, atribuição de IP e as próprias definições de rede do dispositivo — encontra-se no **[manual do utilizador do DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página aborda a parte do Chloros: ligação, sincronização de tempo e o que fazer quando a deteção não produz resultados.

O DAQ-E é o membro da família DAQ que utiliza Ethernet: alimentado por PoE, detetado através de mDNS (serviço `_daq-e._tcp`) e endereçável por um nome de anfitrião derivado do seu ID de sensor — `daq-e-<6 hex>.local`, por exemplo, `daq-e-def330.local`. Esta página aborda a forma como o dispositivo transfere dados na rede e como participa na sincronização de tempo PTP.

## Modos de transporte

| Modo | Ponto final | Consumidores | Notas |
| --- | --- | --- | --- |
| **Multicast** (predefinição) | UDP `239.10.10.10:5002` | Qualquer número na mesma LAN recebe o mesmo fluxo | Cada datagrama é validado por CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Exatamente um cliente (exclusivo) | Compatível ao nível do byte com o DAQ-U |

O Chloros utiliza multicast por predefinição, o que permite que a GUI, a CLI e o SDK observem todos um único sensor ao mesmo tempo.

## Requisitos de rede

* **Mesmo domínio de difusão.** A máquina que executa o Chloros deve estar no mesmo segmento de rede L2 que o sensor — a descoberta mDNS não atravessa routers.
* **Firewall.** No Windows, o instalador adiciona a regra de firewall **Chloros Backend (UDP-In)**para o backend, em todos os perfis de rede, que abrange os dados do DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320), bem como uma regra**«Chloros Backend (TCP-In)»**, limitada à sub-rede local, para a ligação que um DAQ-E estabelece de volta ao computador durante uma atualização de firmware. No Linux, o `.deb` abre as portas UDP no `ufw` ou no `firewalld` quando essa firewall está ativa no momento da instalação. Um firewall de terceiros, ou um ativado posteriormente, necessita que as mesmas portas sejam permitidas.
* **Alimentação PoE, sem LED de estado.** O DAQ-E não possui LED próprio — verifique a alimentação através do indicador de ligação/PoE no switch ou na porta do injetor e aguarde alguns segundos após a ligação para que o dispositivo inicie e se junte à rede.

## Ligação

**GUI:** Separador «Sensores de luz» → «Ligar sensor» → Tipo de dispositivo «DAQ-E (Ethernet)». A deteção só decorre enquanto a caixa de diálogo de ligação estiver no ecrã (navegação mDNS mais uma varredura ARP no Windows), repetindo-se a cada 15 segundos; o botão «Atualizar» volta a procurar imediatamente. Os sensores detetados aparecem no menu suspenso; o primeiro sensor detetado é selecionado automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend em execução):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Um DAQ-E que já se encontre no conjunto é reconhecido tanto pelo seu nome de anfitrião como pelo seu endereço IP: ligá-lo novamente com o outro endereço devolve a entrada existente no conjunto, em vez de abrir a unidade pela segunda vez.

### Anfitriões com várias placas de rede e a primeira ligação após o arranque

Em anfitriões com mais do que uma interface de rede ativa, o **primeiro** `pool-connect --eth` após o arranque pode ficar vazio, mesmo quando o sensor está em bom estado — a pesquisa de deteção pode não encontrar a interface em que o sensor se encontra enquanto a cache ARP ainda estiver inativa. A solução fiável consiste em ignorar a descoberta e passar o endereço explicitamente:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aceita o nome de anfitrião mDNS ou o IP, direciona-se sempre para o sensor correto e é a forma recomendada para scripts e instalações sem interface gráfica. Na GUI, utilize o botão «Atualizar» da caixa de diálogo de ligação e aguarde um ciclo de nova pesquisa.

## Configurações do dispositivo e firmware

O próprio sensor armazena a sua configuração de rede: DHCP (a configuração de fábrica, recorrendo a um endereço local de ligação 169.254.x.x quando nenhum servidor DHCP responde) ou um endereço estático. Também armazena se inicia a transmissão ao ligar-se. Altere estas definições na GUI (nas definições do sensor, **Rede**:**Alterar…**,**Transmitir ao ligar**,**Reiniciar sensor**; consulte [O separador DAQ](gui.md)), ou a partir da CLI e do SDK com `pool-set-network`, `pool-set-auto-stream` e `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; consulte a [Referência da CLI](../reference/cli-reference.md#daq-e-device-settings)). O sensor reinicia para aplicar uma alteração na rede, e o Chloros volta a ligá-lo. Deve existir um endereço estático na rede do computador onde o Chloros está a ser executado: o Chloros rejeita qualquer endereço ao qual não consiga aceder após o reinício. O ID do sensor e o seu nome de anfitrião `daq-e-<id>.local` nunca se alteram. Novidade na versão 1.3.0.

**As atualizações de firmware estão integradas na GUI.** Quando um DAQ-E ligado executa um firmware mais antigo do que a imagem incluída na sua compilação do Chloros, a linha do seu sensor apresenta um ícone âmbar com a mensagem **Atualização disponível**, e a janela modal de definições (engrenagem) apresenta um <version>botão</version>

«Atualizar para <version>». A atualização é transferida pela rede em cerca de 30 segundos; o sensor reinicia e volta a ligar-se automaticamente, e uma transferência interrompida mantém o firmware atual intacto. A partir da CLI ou do SDK, e no Linux — que não possui interface gráfica —, execute o comando `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronização de tempo PTP

O firmware v1.2.0+ do DAQ-E participa no IEEE 1588 PTPv2 como um relógio comum (apenas escravo). **O backend do anfitrião Chloros é o «grandmaster» PTP** — todos os DAQ-E e todas as câmaras LATTICE na LAN funcionam como escravos deste no domínio 0, mantendo todos os carimbos de data/hora dos dispositivos dentro de uma tolerância de ~1 ms. É esse relógio partilhado que permite que as leituras do DAQ tenham carimbos de data/hora correspondentes às exposições da câmara (ver [Gravação e o formato .daq](recording.md)).

Verifique a sincronização a partir da CLI:

| Comando | Mostra |
| --- | --- |
| `chloros-cli time-sync status` | Estado do «grandmaster» do anfitrião, prioridades BMCA, identidade do relógio e `health`: `OK`, ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` com um `health_detail` a indicar por que razão os seus pacotes não estão a chegar à rede |
| `chloros-cli time-sync peers` | Todos os escravos detetados (sensores DAQ-E + câmaras LATTICE) |
| `chloros-cli time-sync cameras` | Estado do PTP por câmara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar o processo do grandmaster |

Na GUI, a janela modal de definições do DAQ-E apresenta uma linha **PTP Sync** em tempo real com o estado PTP atual do sensor.**Um DAQ-E com o PTP desativado permanece desativado.** A unidade mantém a sua configuração de ativação/desativação do PTP na sua própria memória; por isso, uma unidade cujo PTP tenha sido desativado permanece desativado após uma reconexão ou reinicialização, e os seus carimbos de data/hora não são sincronizados via PTP. O `chloros-cli daq pool-list` sinaliza essa unidade (`!! PTP is switched OFF on this unit …`) e apresenta o comando que corrige o problema:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
``` O

Chloros nunca volta a ativar uma unidade por si próprio, porque o estado desativado pode ser intencional.

Detalhes para utilizadores que exigem um alinhamento rigoroso:

* Cada datagrama transmitido contém um campo de sinalizadores; **o bit 2 é definido em quadros cujo carimbo de data/hora está sincronizado com o PTP**. Os pipelines que exigem um alinhamento rigoroso entre a câmara e o DAQ devem basear-se nesse bit.
* Antes de uma captura sincronizada, confirme se o sensor aparece em `chloros-cli time-sync peers` e se `chloros-cli daq pool-list` não indica que o seu PTP está desligado. (As ferramentas internas de hardware direto do MAPIR também podem condicionar a gravação ao bloqueio PTP com um sinalizador `--wait-ptp` que aguarda até 15 s até que o sensor atinja o estado SLAVE; essas ferramentas não fazem parte da CLI fornecida.)
* Enquanto o PTP estiver ativamente em modo escravo, o sensor recusa envios manuais de relógio («O PTP está a fornecer o relógio»). Isso faz parte do design — confie no PTP.

## Notas sobre o Linux

* **Portas PTP sem direitos de root.** O PTP utiliza as portas UDP 319/320. O `.deb` escreve o `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), pelo que o backend se liga a essas portas, executando-se como o seu utilizador. A partir da versão 1.3.0, o pacote não define qualquer capacidade de ficheiro (`setcap`) no backend e não necessita de `libcap2-bin`; não adicione uma capacidade por conta própria, pois o backend descompactado não arrancará. Se o PTP não conseguir ligar-se à porta 319, consulte [Instalação no Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson sem monitor / host Linux:** o pacote instala os serviços do backend, mas não ativa nenhum. Para que o PTP (e a disponibilidade do DAQ) permaneçam sempre ativos sem que ninguém execute um comando, ative o serviço como o seu utilizador:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Sem isso, o PTP só funciona enquanto algo tiver iniciado o backend — um comando `chloros-cli` ou um script do SDK. Consulte [Instalação no Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) para obter informações sobre o utilizador root `chloros-backend.service` e como sair desse modo.

## Resolução de problemas: «Não foram encontrados dispositivos DAQ-E»

| Verificação | Detalhe |
| --- | --- |
| Alimentação | Não há LED aceso no sensor — verifique os indicadores de PoE e de ligação da porta do comutador/injetor; aguarde alguns segundos após a ligação |
| Domínio de difusão | O anfitrião e o sensor estão no mesmo segmento L2; o mDNS não encaminha |
| Firewall | Windows: a regra **Chloros Backend (UDP-In)** do instalador permite a entrada do fluxo DAQ-E (UDP 5002), mDNS (5353) e PTP (319/320); um firewall de terceiros necessita da mesma configuração. Linux: o `.deb` abre estas portas no `ufw` / `firewalld` apenas se o firewall estiver ativo no momento da instalação — autorize-as manualmente caso o firewall seja ativado posteriormente |
| Host com várias placas de rede | A descoberta pesquisa o mDNS com um recurso de fallback ARP; o `--eth-host <ip-or-hostname>` ignora a descoberta e liga-se diretamente |
| Nova pesquisa na GUI | A descoberta só é executada enquanto a caixa de diálogo de ligação estiver aberta; utilize «Atualizar» |</version>
