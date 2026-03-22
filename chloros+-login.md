# Chloros+ Iniciar sessão

## Chloros e Chloros (Navegador) Iniciar sessão

O <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite-lhe iniciar sessão na sua conta Chloros+ e desbloquear funcionalidades adicionais.

Quando iniciar sessão, os detalhes da sua conta serão apresentados:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Login

Inicie sessão com as suas credenciais Chloros+ para ativar o processamento CLI. No Linux (sem GUI), esta é a única forma de ativar a sua licença.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Utilizadores do SDK**: O Python SDK também fornece um método programático `logout()` para limpar credenciais armazenadas em cache. Consulte a [documentação do Python SDK](api-python-sdk.md#logout) para obter mais detalhes.
{% endhint %}

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiais**: Utilize aspas simples em torno de palavras-passe que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Armazenamento de credenciais

As credenciais em cache são armazenadas num local específico da plataforma:

| Plataforma | Caminho do cache de credenciais |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Expiração do plano

A data de expiração do plano na GUI indica quando a sua licença deixará de ser válida. Para assinaturas mensais recorrentes, a expiração ocorre no final do mês. Para assinaturas anuais, ocorre um ano após o início da assinatura. A verificação da licença requer uma ligação à Internet mensal para ser efetuada, com um período de carência de 30 dias.

### Limite de dispositivos

Cada plano Chloros+ oferece um número diferente de dispositivos registados. Cada dispositivo em que iniciar sessão com uma conta Chloros+ contará para o seu número de dispositivos registados. Pode renomear e remover um dispositivo na página da sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PRATA</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
