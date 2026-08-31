# Chloros+ Iniciar sessão

## Iniciar sessão na GUI

O menu lateral do utilizador <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite-lhe iniciar sessão na sua conta Chloros+ e desbloquear funcionalidades adicionais.

**Basta iniciar sessão uma vez por computador.** A GUI, o CLI e o Python SDK partilham a mesma sessão em cache — o início de sessão através da GUI do ambiente de trabalho também ativa o CLI e o SDK nessa máquina (e vice-versa através do `chloros-cli login`).

Quando estiveres com sessão iniciada, os detalhes da tua conta serão apresentados:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Níveis de plano

| Plano | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Grátis |
| Copper | `1` | Pago (Chloros+) |
| Bronze | `2` | Pago (Chloros+) |
| Prata | `3` | Pago (Chloros+) |
| Ouro | `4` | Pago (Chloros+) |

Consulte [planos e preços](https://cloud.mapir.camera/pricing) para saber o que cada nível pago inclui.

### O acesso a CLI / SDK requer um nível pago

O acesso a CLI e Python SDK requer **qualquer nível pago Chloros+ (Copper ou superior)**. Isto é aplicado**do lado do servidor** — cada pedido CLI/SDK deve incluir tanto uma sessão ativa como um plano pago:

| Estado HTTP | `error_code` | Significado | Solução |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Não está com sessão iniciada neste computador | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Iniciou sessão, mas o nível do plano é demasiado baixo (nível Iron gratuito) | Atualize para qualquer plano Chloros+ pago |

O `chloros-cli status` continua acessível no plano gratuito, pelo que pode sempre ver o seu plano atual e o motivo pelo qual o acesso foi recusado.

### Limites de hardware ligado por plano

Cada plano estabelece um limite máximo para o número de câmaras LATTICE e sensores de luz DAQ que podem estar ligados em tempo real ao mesmo tempo:

| Plano | Câmaras LATTICE | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito / sem iniciar sessão) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Início de sessão no CLI

Inicie sessão com as suas credenciais do Chloros+ para ativar o processamento do CLI. No Linux (sem interface gráfica), esta é a única forma de ativar a sua licença.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilizadores do SDK**: O Python SDK também disponibiliza um método programático `logout()` para limpar as credenciais armazenadas em cache. Consulte a [Referência do SDK](reference/sdk-reference.md) para obter mais detalhes.
{% endhint %}

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiais**: Utilize aspas simples em torno de palavras-passe que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Armazenamento de credenciais

As credenciais e a configuração em cache são armazenadas na pasta `.chloros` do diretório pessoal do utilizador em **todas as plataformas**:

| Plataforma | Caminho do cache de credenciais |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Expiração do plano e período de tolerância offline

A data de expiração do plano apresentada na interface gráfica indica quando a sua licença deixará de ser válida. Para subscrições mensais recorrentes, a expiração ocorre no final do mês; para subscrições anuais, ocorre um ano após o início da subscrição.

O Chloros valida a sua licença online, mas o funcionamento offline é suportado durante um período de tolerância:

* As validações bem-sucedidas no servidor são armazenadas em cache durante **5 minutos**, pelo que a utilização normal implica muito poucas chamadas de licença.
* Um cache de licenças assinado e vinculado ao equipamento cobre períodos de inatividade mais longos: **30 dias para planos mensais**e**até à data de expiração da sua subscrição (no máximo 365 dias) para planos anuais**.
* Quando o período de tolerância termina, o plano passa para o nível gratuito «Iron» até que o computador consiga aceder ao servidor de licenças uma vez; o acesso é retomado na próxima verificação bem-sucedida.

### Limite de dispositivos

Cada plano Chloros+ oferece um número diferente de dispositivos registados. Cada dispositivo em que iniciar sessão com uma conta Chloros+ conta para o seu número de dispositivos registados. Pode renomear e remover um dispositivo na página da sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PRATA</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>O número exato de dispositivos permitidos na sua conta é apresentado na página da sua conta MAPIR Cloud. Ao terminar a sessão num dispositivo, o respectivo lugar fica efetivamente livre, e um dispositivo já registado pode sempre iniciar sessão novamente, mesmo quando a conta atingir o limite de dispositivos.
