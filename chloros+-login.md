# Login do Cloros+

## Login de Cloros e Cloros (navegador)

The user <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebar menu allows you to log into your Chloros+ account and unlock additional features.

Quando logado, os detalhes da sua conta serão mostrados:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI de login

Faça login com suas credenciais da Chloros+ para ativar o processamento CLI.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% dica estilo = "aviso" %}
**Caracteres especiais**: use aspas simples em senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Expiração do plano

A expiração do plano na GUI mostra quando sua licença se torna inválida. Para assinaturas mensais recorrentes, o vencimento é no final do mês. Para assinaturas anuais, é um ano após o início da assinatura. A verificação da licença requer uma conexão mensal com a Internet para verificação, com período de carência de 30 dias.

### Limite de dispositivos

Cada plano Cloros+ oferece uma quantidade diferente de dispositivos cadastrados. Cada dispositivo em que você fizer login com uma conta Chloros+ contará para o seu número de dispositivos registrados. Você pode renomear e remover um dispositivo na página de sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Devices Supported</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
