---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Faça o download da versão mais recente do Chloros para começar a processar imagens multiespectrais.

### Requisitos do sistema

| Requisito          | Mínimo                         | Recomendado                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Sistema operativo** | Windows 10 (64 bits)             | Windows 11 (64 bits)             |
| **Processador**        | Intel Core i5 ou equivalente     | Intel Core i7 ou superior         |
| **Memória (RAM)**     | 8 GB                             | 16 GB ou mais                    |
| **Placa gráfica**    | Compatível com DirectX 11           | GPU NVIDIA com 4 GB+ de VRAM       |
| **Armazenamento**          | 6 GB de espaço livre                  | SSD com 10 GB+ de espaço livre       |
| **Ecrã**          | 1920x1080                       | 2560x1440 ou superior             |
| **Internet**         | Necessária para ativação da licença | Necessária para ativação da licença |

{% hint style=&quot;info&quot; %}
**Aceleração GPU**: Os utilizadores do Chloros+ com GPUs NVIDIA (4 GB+ VRAM) podem usar a aceleração CUDA para um processamento significativamente mais rápido. Os utilizadores do Chloros+ também ganham processamento multithread para velocidade máxima.
{% endhint %}

***

## Descarregar Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Descarregar Chloros aqui</a>

### Última versão estável

**Chloros Instalador para Windows*** **Versão**: 1.0.4
* **Data de lançamento**: 5 de janeiro de 2026
* **Tamanho do ficheiro (download)**: 1,8 GB
* **Tamanho do ficheiro (instalado)**: 5,7 GB
* **Tipo de ficheiro**: .exe (Instalador Windows)

#### **Etapas de instalação:**

1. Descarregue o ficheiro `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\[USER]\Chloros\`)
5. Conclua a instalação e inicie o Chloros, o Chloros (navegador) ou o Chloros CLI
6. Inicie sessão com a sua conta [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

{% hint style=&quot;success&quot; %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema para acesso à linha de comando.
{% endhint %}

***

## Recursos adicionais

### Python SDK

Para programadores e fluxos de trabalho de automação, instale o Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)**Requisitos**: Chloros Desktop deve estar instalado, Chloros+ login de licença necessário***

## O que está incluído

A instalação do Chloros inclui:

* ✅ **Chloros** - Interface gráfica completa
* ✅ **Chloros (Navegador)** - Interface baseada na Web para sistemas com especificações mais baixas
* ✅ **Chloros CLI** - Interface de linha de comando (requer licença Chloros+)
* ✅ **Chloros SDK** - Python API (requer licença Chloros+)
* ✅ **Perfis de câmara** - Modelos de câmara MAPIR pré-configurados***

## Atualize para Chloros+

Desbloqueie funcionalidades avançadas com uma subscrição Chloros+:

* 🚀 **Processamento multithread** - Processe imagens em paralelo
* ⚡ **Aceleração GPU (CUDA)** - Aproveite o poder da GPU NVIDIA
* 💻 **Acesso CLI** - Automatize com ferramentas de linha de comando
* 🐍 **Python SDK** - Acesso programático API
* 📱 **Vários dispositivos** - Use em 2 a 10+ dispositivos (dependendo do plano)
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Ver planos e preços do Chloros</a></p>***

## Ajuda com a instalação

### Resolução de problemas

**A instalação falha com a mensagem de erro:**

* Certifique-se de que tem direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se cumpre os requisitos mínimos do sistema

**A aplicação não inicia:**

* Experimente a versão Chloros (navegador)
* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os controladores gráficos
* Verifique o Windows Event Viewer para obter detalhes sobre o erro
* Contacte o suporte com os registos de erros

**Problemas de ativação da licença:**

* Certifique-se de que a ligação à Internet está ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se a firewall não está a bloquear o Chloros
* Consulte [Chloros+ Login](chloros+-login.md) para obter instruções detalhadas

### Obter suporte

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)***

## Registo de alterações

<details>

<summary>Versão 1.0.4</summary>

#### **Data de lançamento**: 5 de janeiro de 2026**Novos recursos*** **Alternar imagem/metadados**: Adicionado alternador no Navegador de arquivos para visualizar os metadados da imagem selecionada em uma tabela em vez da grade de imagens
* **Controle deslizante de zoom da grade de imagens**: Novo controle deslizante da interface do usuário para ajustar o tamanho das miniaturas (também suporta CTRL + roda do rato)
* **Botões de exportação da grelha de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (alvos, refletância, índice, LUT)
* **Guia Mapa**: Novo mapa 2D interativo que mostra marcadores de localização GPS da imagem
  * Suporta Google Maps e blocos de mapa ESRI (seleciona automaticamente o melhor serviço de blocos com base na disponibilidade do nível de zoom)
  * Pré-visualização da miniatura ao passar o mouse sobre os marcadores do mapa

**Correções de bugs*** Suporte aprimorado para a instalação do Chloros em computadores que não estão em inglês

</details>

<details>

<summary>Versão 1.0.3</summary>

#### **Data de lançamento**: 20 de dezembro de 2025**Novos recursos*** Lançamento inicial

**Melhorias*** Lançamento inicial

**Correções de bugs*** Lançamento inicial

**Problemas conhecidos*** Lançamento inicial

</details>***

## Contrato de licença**Software proprietário** - Copyright (c) 2025 MAPIR Inc.

É proibido o uso, distribuição ou modificação não autorizados.

**Versão gratuita**: disponível para uso pessoal e comercial com limitações de funcionalidades**Chloros+**: licença baseada em assinatura para funcionalidades avançadas e implementações comerciais
