# Login no Chloros+

## Login na GUI

O menu lateral do utilizador <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite-lhe iniciar sessão na sua conta Chloros+ e desbloquear funcionalidades adicionais.

**Só precisa de iniciar sessão uma vez por computador.** A GUI, a CLI e o SDK do Python partilham a mesma sessão em cache — o início de sessão através da GUI do ambiente de trabalho também ativa a CLI e o SDK nessa máquina (e vice-versa através do `chloros-cli login`).

Quando estiver com a sessão iniciada, os detalhes da sua conta serão apresentados:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Níveis de plano

| Plano | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Gratuito |
| Copper | `1` | Pago (Chloros+) |
| Bronze | `2` | Pago (Chloros+) |
| Silver | `3` | Pago (Chloros+) |
| Ouro | `4` | Pago (Chloros+) |

Consulte [planos e preços](https://cloud.mapir.camera/pricing) para saber o que cada nível pago inclui.

### O acesso à CLI/SDK requer um nível pago

O acesso à CLI e ao Python SDK requer **qualquer nível pago do Chloros+ (Cobre ou superior)**. Esta regra é aplicada**do lado do servidor** — cada pedido de CLI/SDK deve incluir tanto uma sessão ativa como um plano pago:

| Estado HTTP | `error_code` | Significado | Solução |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Não está com sessão iniciada nesta máquina | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Iniciou sessão, mas o nível do plano é demasiado baixo (nível gratuito «Iron») | Atualize para qualquer plano pago «Chloros+» |

O `chloros-cli status` continua acessível no nível gratuito, pelo que pode sempre ver o seu plano atual e o motivo pelo qual o acesso foi recusado.

### Limites de hardware ligado por plano

Cada plano estabelece um limite para o número de câmaras LATTICE e sensores de luz DAQ que podem estar ligados em tempo real ao mesmo tempo:

| Plano | Câmaras LATTICE | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito / sem iniciar sessão) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Início de sessão na CLI



Inicie sessão com as suas credenciais do Chloros+ para ativar o processamento via CLI. No Linux (sem GUI), esta é a única forma de ativar a sua licença.

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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Armazenamento de credenciais

As credenciais e a configuração em cache são armazenadas na pasta `.chloros` do diretório pessoal do utilizador em **todas as plataformas**:

| Plataforma | Caminho do cache de credenciais |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

A instalação de uma versão mais recente sobre uma já existente mantém os seus dados de início de sessão, no Windows (o instalador) e no Linux (o `.deb`, incluindo o `chloros-cli update`). A desinstalação do Chloros — ou a remoção do pacote Linux — limpa o login armazenado em cache, pelo que terá de iniciar sessão novamente após uma nova instalação.

### Expiração do plano e período de tolerância offline

A data de expiração do plano na interface gráfica indica quando a sua licença deixará de ser válida. Para subscrições mensais recorrentes, a expiração ocorre no final do mês; para subscrições anuais, ocorre um ano após o início da subscrição.

O Chloros valida a sua licença online, mas o funcionamento offline é suportado durante um período de tolerância:

* As validações bem-sucedidas no servidor são armazenadas em cache durante **5 minutos**, pelo que a utilização normal implica muito poucas chamadas de licença.
* Um cache de licenças assinadas e vinculadas ao equipamento cobre períodos offline mais longos: **30 dias para planos mensais**e**até à data de expiração da sua subscrição (no máximo 365 dias) para planos anuais**.
* Quando o período de tolerância expira, o plano passa para o nível gratuito «Iron» até que o computador consiga aceder ao servidor de licenças uma vez; o acesso é retomado na próxima verificação bem-sucedida.

### Limite de dispositivos

Cada plano Chloros+ oferece um número diferente de dispositivos registados. Cada dispositivo em que iniciar sessão com uma conta Chloros+ conta para o seu número de dispositivos registados. Pode renomear e remover um dispositivo na página da sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

O número exato de dispositivos permitidos na sua conta é apresentado na página da sua conta MAPIR Cloud. Ao sair de um dispositivo, a vaga fica efetivamente livre, e um dispositivo já registado pode sempre voltar a iniciar sessão, mesmo que a conta tenha atingido o limite de dispositivos.
