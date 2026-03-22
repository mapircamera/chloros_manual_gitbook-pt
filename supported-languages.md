# Idiomas suportados

O Chloros oferece suporte completo à interface em **38 idiomas em todo o mundo**, tornando-o acessível a utilizadores de todo o mundo. Pode alternar de idioma instantaneamente em todas as interfaces: Ambiente de trabalho, Navegador, CLI e Python SDK.

O Chloros suporta os seguintes idiomas:

| # | Idioma | Nome nativo | Código CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Inglês | Inglês | `en` |
| 2 | 🇪🇸 Espanhol | Español | `es` |
| 3 | 🇵🇹 Português | Português | `pt` |
| 4 | 🇫🇷 Francês | Français | `fr` |
| 5 | 🇩🇪 Alemão | Deutsch | `de` |
| 6 | 🇮🇹 Italiano | Italiano | `it` |
| 7 | 🇯🇵 Japonês | 日本語 | `ja` |
| 8 | 🇰🇷 Coreano | 한국어 | `ko` |
| 9 | 🇨🇳 Chinês (Simplificado) | 简体中文 | `zh` |
| 10 | 🇹🇼 Chinês (Tradicional) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russo | Русский | `ru` |
| 12 | 🇳🇱 Holandês | Nederlands | `nl` |
| 13 | 🇸🇦 Árabe | العربية | `ar` |
| 14 | 🇵🇱 Polaco | Polski | `pl` |
| 15 | 🇹🇷 Turco | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonésio | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamita | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tailandês | ไทย | `th` |
| 20 | 🇸🇪 Sueco | Svenska | `sv` |
| 21 | 🇩🇰 Dinamarquês | Dansk | `da` |
| 22 | 🇳🇴 Norueguês | Norsk | `no` |
| 23 | 🇫🇮 Finlandês | Suomi | `fi` |
| 24 | 🇬🇷 Grego | Ελληνικά | `el` |
| 25 | 🇨🇿 Checo | Čeština | `cs` |
| 26 | 🇭🇺 Húngaro | Magyar | `hu` |
| 27 | 🇷🇴 Romeno | Română | `ro` |
| 28 | 🇺🇦 Ucraniano | Українська | `uk` |
| 29 | 🇧🇷 Português do Brasil | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Cantonês | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaio | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Eslovaco | Slovenčina | `sk` |
| 33 | 🇧🇬 Búlgaro | Български | `bg` |
| 34 | 🇭🇷 Croata | Hrvatski | `hr` |
| 35 | 🇱🇹 Lituano | Lietuvių | `lt` |
| 36 | 🇱🇻 Letão | Latviešu | `lv` |
| 37 | 🇪🇪 Estónio | Eesti | `et` |
| 38 | 🇸🇮 Esloveno | Slovenščina | `sl` |

## Como alterar o idioma

### No Chloros Desktop/Navegador

1. Abra as definições da aplicação
2. Aceda ao menu de seleção de idioma
3. Escolha o seu idioma preferido na lista
4. A interface será atualizada instantaneamente

### No Chloros CLI

Utilize o comando `language` para visualizar ou alterar o idioma da interface CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Para mais detalhes, consulte a [documentação CLI](CLI.md).

### No Chloros Python SDK

Defina o parâmetro de idioma ao inicializar o SDK para obter mensagens e saídas no seu idioma preferido.

## Cobertura

Todos os 38 idiomas são totalmente suportados em:

* **Chloros Desktop** - Tradução completa da GUI
* **Chloros Browser** - Interface web em todos os idiomas
* **Chloros CLI** - Interface de linha de comando e mensagens de saída
* **Chloros Python SDK** - Mensagens e documentação

O suporte a idiomas garante que utilizadores de todo o mundo possam trabalhar de forma eficiente na sua língua nativa, sem barreiras.
