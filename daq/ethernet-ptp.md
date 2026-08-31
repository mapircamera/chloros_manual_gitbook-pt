# Rede e sincronização de tempo do DAQ-E

> A configuração física da rede para o sensor — cablagem, PoE, atribuição de IP e as próprias definições de rede do dispositivo — encontra-se no **[manual do utilizador do DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página aborda a parte relativa ao Chloros: ligação, sincronização de tempo e o que fazer quando a deteção não produz resultados.

O DAQ-E é o membro da família DAQ que utiliza Ethernet: alimentado por PoE, detetado via mDNS (serviço `_daq-e._tcp`) e acedível através de um nome de anfitrião derivado do seu ID de sensor — `daq-e-<6 hex>.local`, por exemplo, `daq-e-def330.local`. Esta página aborda a forma como transfere dados na rede e como participa na sincronização de tempo PTP.

## Modos de transporte

| Modo | Ponto final | Consumidores | Notas |
| --- | --- | --- | --- |
| **Multicast** (predefinição) | UDP `239.10.10.10:5002` | Qualquer número na mesma LAN recebe o mesmo fluxo | Cada datagrama é validado por CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Exatamente um cliente (exclusivo) | Compatível a nível de bytes com o DAQ-U |

O Chloros utiliza multicast por predefinição, o que permite que a GUI, o CLI e o SDK monitorizem todos um sensor em simultâneo.

## Requisitos de rede

* **Mesmo domínio de difusão.** A máquina que executa o Chloros deve estar no mesmo segmento de rede L2 que o sensor — a descoberta mDNS não atravessa routers.
* **Aviso do firewall do Windows: aceite-o.** Na primeira vez que o Chloros estabelece ligação aos sockets de multicast, o Windows Defender apresenta um aviso único. Permitir isso abrange os dados do DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320). No Linux, isto ocorre silenciosamente.
* **Alimentação PoE, sem LED de estado.** O DAQ-E não possui LED próprio — verifique a alimentação através do indicador de ligação/PoE no switch ou na porta do injetor e aguarde alguns segundos após o arranque para que o dispositivo inicie e se ligue à rede.

## Ligação

**GUI:** Separador «Sensores de luz» → «Ligar sensor» → Tipo de dispositivo «DAQ-E (Ethernet)». A deteção só ocorre enquanto a caixa de diálogo de ligação estiver no ecrã (pesquisa mDNS e varredura ARP no Windows), repetindo-se a cada 15 segundos; o botão «Atualizar» realiza uma nova pesquisa imediatamente. Os sensores detetados aparecem no menu suspenso; o primeiro sensor detetado é selecionado automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend em execução):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosts com várias placas de rede e a primeira ligação após o arranque

Em anfitriões com mais do que uma interface de rede ativa, o **primeiro** `pool-connect --eth` após o arranque pode ficar vazio, mesmo quando o sensor está em bom estado — a pesquisa de deteção pode não identificar a interface em que o sensor se encontra enquanto a cache ARP ainda estiver fria. A solução fiável consiste em ignorar a descoberta e passar o endereço explicitamente:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aceita o nome de anfitrião mDNS ou o IP, direciona-se sempre para o sensor correto e é a forma recomendada para scripts e instalações sem interface gráfica. Na GUI, utilize o botão «Atualizar» da caixa de diálogo de ligação e aguarde um ciclo de nova pesquisa.

## Configurações do dispositivo e firmware

O próprio sensor contém as configurações de rede — IP estático vs. DHCP + endereçamento local de ligação, nome do dispositivo, transmissão automática ao arrancar e palavra-passe OTA. Estas definições do lado do dispositivo não estão disponíveis como comandos no CLI fornecido; são geridas através da GUI do Chloros, onde são apresentadas, ou com o suporte do MAPIR.

**As atualizações de firmware estão integradas na interface gráfica.**Quando um DAQ-E ligado executa um firmware mais antigo do que a imagem incluída na sua compilação do Chloros, a linha do seu sensor apresenta um ícone âmbar**Atualização disponível** e a janela modal de definições (engrenagem) oferece um<version>

botão</version> «Atualizar para<version>

». A atualização é transferida através da rede em cerca de 30 segundos; o sensor reinicia e volta a ligar-se automaticamente, e uma transferência interrompida mantém o firmware atual intacto.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronização de tempo PTP

O firmware v1.2.0+ do DAQ-E participa no IEEE 1588 PTPv2 como um relógio normal (apenas escravo). **O backend do anfitrião Chloros é o «grandmaster» PTP** — todos os DAQ-E e todas as câmaras LATTICE na LAN funcionam como escravos deste no domínio 0, mantendo todos os carimbos de data/hora dos dispositivos dentro de uma tolerância de ~1 ms. É esse relógio partilhado que permite que as leituras do DAQ tenham carimbos de data/hora correspondentes às exposições da câmara (ver [Gravação e o formato .daq](recording.md)).

Verifique a sincronização a partir do CLI:

| Comando | Mostra |
| --- | --- |
| `chloros-cli time-sync status` | Estado do «grandmaster» do anfitrião, prioridades BMCA, identidade do relógio |
| `chloros-cli time-sync peers` | Todos os escravos detetados (sensores DAQ-E + câmaras LATTICE) |
| `chloros-cli time-sync cameras` | Estado de integridade PTP por câmara (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar o processo «grandmaster» |

Na GUI, a janela modal de configurações do DAQ-E apresenta uma linha **PTP Sync** em tempo real com o estado PTP atual do sensor.

Detalhes para consumidores que exigem alinhamento rigoroso:

* Cada datagrama transmitido contém um campo de sinalizadores; **o bit 2 é ativado em quadros cujo carimbo de data/hora está sincronizado com o PTP**. Os pipelines que exigem um alinhamento rigoroso entre a câmara e o DAQ devem ser acionados com base nesse bit.
* Antes de uma captura sincronizada, confirme se o sensor aparece no `chloros-cli time-sync peers`. (As ferramentas internas de hardware direto do MAPIR também podem condicionar a gravação ao bloqueio PTP com um sinalizador do `--wait-ptp` que aguarda até 15 s até que o sensor atinja o estado SLAVE; essa funcionalidade não faz parte do CLI fornecido.)
* Enquanto o PTP estiver ativamente em modo escravo, o sensor recusa envios manuais de relógio («O PTP está a fornecer o relógio»). Isso faz parte do comportamento previsto — confie no PTP.

## Notas sobre o Linux

* **O PTP necessita do `libcap2-bin` no momento da instalação.** O script de pós-instalação `.deb` concede permissão ao `cap_net_bind_service=+ep` no `/usr/lib/chloros/chloros-backend` para que este possa ligar as portas PTP 319/320 sem direitos de root. Se o `libcap2-bin` estiver em falta, essa etapa é ignorada e o PTP não conseguirá iniciar. Solução:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi sem interface gráfica:** na primeira instalação, a unidade systemd `chloros-backend.service` é gerada, mas não está ativada. Para manter o PTP sempre ativo (e a disponibilidade do DAQ) sem a GUI:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Sem ela, o PTP só funciona enquanto a GUI Chloros estiver aberta.

## Resolução de problemas: «Não foram encontrados dispositivos DAQ-E»

| Verificação | Detalhe |
| --- | --- |
| Alimentação | Nenhum LED aceso no sensor — verifique os indicadores de PoE e de ligação da porta do comutador/injetor; aguarde alguns segundos após o arranque |
| Domínio de difusão | Anfitrião e sensor no mesmo segmento L2; o mDNS não encaminha |
| Firewall do Windows | Aceite o aviso do Defender na primeira execução (UDP 5002, 5353, 319/320) |
| Anfitrião com várias placas de rede | A primeira deteção após o arranque pode não identificar o sensor — ligue-se com o `--eth-host <ip-or-hostname>` |
| Nova pesquisa na GUI | A deteção só decorre enquanto a caixa de diálogo de ligação estiver aberta; utilize «Atualizar» |</version>
