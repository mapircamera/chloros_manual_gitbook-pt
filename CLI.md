# CLI: Linha de comando

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>O **Chloros CLI** fornece acesso poderoso por linha de comando ao motor de processamento de imagens Chloros, permitindo automação, criação de scripts e operação sem monitor para os seus fluxos de trabalho de imagem.

### Principais funcionalidades

* 🚀 **Automação** - Processamento em lote de vários conjuntos de dados por script
* 🔗 **Integração** - Incorporação em fluxos de trabalho e pipelines existentes
* 💻 **Operação sem interface gráfica** - Execução sem GUI
* 🌍 **Multilíngue** - Suporte para 38 idiomas
* ⚡ **Processamento paralelo** - Escala dinamicamente para a sua CPU (até 16 trabalhadores paralelos)

### Requisitos

| Requisito          | Detalhes                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operativo** | Windows 10/11 (64 bits)                                              |
| **Licença**          | Chloros+ ([plano pago necessário](https://cloud.mapir.camera/pricing)) |
| **Memória**           | Mínimo de 8 GB de RAM (recomendado 16 GB)                                  |
| **Internet**         | Necessária para ativação da licença                                     |
| **Espaço em disco**       | Varia de acordo com o tamanho do projeto                                              |

{% hint style=&quot;warning&quot; %}
**Requisito de licença**: O CLI requer uma assinatura paga do Chloros+. Os planos padrão (gratuitos) não têm acesso ao CLI. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para atualizar.
{% endhint %}

## Início rápido

### Instalação

O CLI é incluído automaticamente com o instalador Chloros:

1. Descarregue e execute o **Chloros Installer.exe**

2. Conclua o assistente de instalação
3. CLI instalado em: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema. Reinicie o seu terminal após a instalação.
{% endhint %}

### Configuração inicial

Antes de usar o CLI, ative a sua licença Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Utilização básica

Processe uma pasta com as definições predefinidas:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referência de comandos

### Sintaxe geral

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandos

### `process` - Processar imagens

Processar imagens numa pasta com calibração.

**Sintaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemplo:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opções do comando Processar

| Opção                | Tipo    | Padrão        | Descrição                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Caminho    | _Obrigatório_     | Pasta contendo imagens multiespectrais RAW/JPG                                         |
| `-o, --output`        | Caminho    | Igual à entrada  | Pasta de saída para imagens processadas                                                     |
| `-n, --project-name`  | String  | Gerado automaticamente | Nome personalizado do projeto                                                                    |
| `--vignette`          | Sinalizador    | Ativado        | Ativar correção de vinheta                                                             |
| `--no-vignette`       | Sinalizador    | -              | Desativar correção de vinheta                                                            |
| `--reflectance`       | Sinalizador    | Ativado        | Ativar calibração de refletância                                                         |
| `--no-reflectance`    | Sinalizador    | -              | Desativar calibração de refletância                                                        |
| `--ppk`               | Sinalizador    | Desativado       | Aplicar correções PPK a partir dos dados do sensor de luz .daq                                      |
| `--format`            | Escolha  | TIFF (16 bits)  | Formato de saída: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Inteiro | Automático           | Tamanho mínimo do alvo em pixels para deteção do painel de calibração                          |
| `--target-clustering` | Inteiro | Automático           | Limite de agrupamento do alvo (0-100)                                                    |
| `--exposure-pin-1`    | String  | Nenhum           | Bloquear exposição para modelo de câmara (Pino 1)                                                 |
| `--exposure-pin-2`    | String  | Nenhum           | Bloquear exposição para modelo de câmara (Pino 2)                                                 |
| `--recal-interval`    | Número inteiro | Automático           | Intervalo de recalibração em segundos                                                      |
| `--timezone-offset`   | Número inteiro | 0              | Desvio do fuso horário em horas                                                               |

***

### `login` - Autenticar conta

Inicie sessão com as suas credenciais Chloros+ para ativar o processamento CLI.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caracteres especiais**: Use aspas simples em torno de senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Limpar credenciais

Limpe as credenciais armazenadas e saia da sua conta.

**Sintaxe:**

```bash
chloros-cli logout
```

**Exemplo:**

```powershell
chloros-cli logout
```

**Saída:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**SDK Utilizadores**: O Python SDK também fornece um método programático `logout()` para limpar credenciais dentro de scripts Python. Consulte a [documentação do Python SDK](api-python-sdk.md#logout) para obter detalhes.
{% endhint %}

***

### `status` - Verificar o estado da licença

Exibe a licença atual e o estado da autenticação.

**Sintaxe:**

```bash
chloros-cli status
```

**Exemplo:**

```powershell
chloros-cli status
```

**Saída:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Verificar o progresso da exportação

Monitorizar o progresso da exportação do Thread 4 durante ou após o processamento.

**Sintaxe:**

```bash
chloros-cli export-status
```

**Exemplo:**

```powershell
chloros-cli export-status
```

**Caso de uso:** Chame este comando enquanto o processamento estiver em execução para verificar o progresso da exportação.***

### `language` - Gerir o idioma da interface

Exiba ou altere o idioma da interface CLI.

**Sintaxe:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exemplos:**

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Idiomas suportados (38 no total)

| Código    | Idioma              | Nome nativo      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglês               | Inglês          |
| `es`    | Espanhol               | Espanhol          |
| `pt`    | Português            | Português        |
| `fr`    | Francês                | Français         |
| `de`    | Alemão                | Deutsch          |
| `it`    | Italiano               | Italiano         |
| `ja`    | Japonês              | 日本語              |
| `ko`    | Coreano                | 한국어              |
| `zh`    | Chinês (simplificado)  | 简体中文             |
| `zh-TW` | Chinês (tradicional) | 繁體中文             |
| `ru`    | Russo               | Русский          |
| `nl`    | Holandês                 | Nederlands       |
| `ar`    | Árabe                | العربية          |
| `pl`    | Polaco                | Polski           |
| `tr`    | Turco               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonésio            | Bahasa Indonesia |
| `vi`    | Vietnamita            | Tiếng Việt       |
| `th`    | Tailandês                  | ไทย              |
| `sv`    | Sueco               | Svenska          |
| `da`    | Dinamarquês                | Dansk            |
| `no`    | Norueguês             | Norsk            |
| `fi`    | Finlandês               | Suomi            |
| `el`    | Grego                 | Ελληνικά         |
| `cs`    | Checo                 | Čeština          |
| `hu`    | Húngaro             | Magyar           |
| `ro`    | Romeno              | Română           |
| `uk`    | Ucraniano             | Українська       |
| `pt-BR` | Português brasileiro  | Português Brasileiro |
| `zh-HK` | Cantonês             | 粵語             |
| `ms`    | Malaio                 | Bahasa Melayu    |
| `sk`    | Eslovaco                | Slovenčina       |
| `bg`    | Búlgaro             | Български        |
| `hr`    | Croata              | Hrvatski         |
| `lt`    | Lituano            | Lietuvių         |
| `lv`    | Letão               | Latviešu         |
| `et`    | Estónio              | Eesti            |
| `sl`    | Esloveno             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Persistência automática**: A sua preferência de idioma é guardada em `~/.chloros/cli_language.json` e mantém-se em todas as sessões.
{% endhint %}

***

### `set-project-folder` - Definir pasta padrão do projeto

Altere a localização da pasta padrão do projeto (partilhada com a GUI).

**Sintaxe:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemplo:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Mostrar pasta do projeto

Exibe a localização atual da pasta padrão do projeto.

**Sintaxe:**

```bash
chloros-cli get-project-folder
```

**Exemplo:**

```powershell
chloros-cli get-project-folder
```

**Saída:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Redefinir para o padrão

Redefinir a pasta do projeto para o local padrão.

**Sintaxe:**

```bash
chloros-cli reset-project-folder
```

***

## Opções globais

Estas opções aplicam-se a todos os comandos:

| Opção          | Tipo    | Padrão       | Descrição                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Caminho    | Detetado automaticamente | Caminho para o executável do backend                       |
| `--port`        | Número inteiro | 5000          | Número da porta do backend API                          |
| `--restart`     | Sinalizador    | -             | Forçar reinício do backend (encerra processos existentes) |
| `--version`     | Sinalizador    | -             | Mostrar informações da versão e sair                |
| `--help`        | Sinalizador    | -             | Mostrar informações de ajuda e sair                   |

**Exemplo com opções globais:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guia de configurações de processamento

### Processamento paralelo

Chloros+ CLI **dimensiona automaticamente**o processamento paralelo para corresponder às capacidades do seu computador:**Como funciona:**

* Deteta os núcleos da CPU e a RAM
* Aloca trabalhadores: **2× núcleos da CPU** (usa hyperthreading)
* **Máximo: 16 trabalhadores paralelos** (para estabilidade)**Níveis do sistema:**

| Tipo de sistema   | CPU        | RAM      | Trabalhadores  | Desempenho     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Alta qualidade**  | 16+ núcleos  | 32+ GB   | Até 16 | Velocidade máxima   |
| **Médio** | 8-15 núcleos | 16-31 GB | 8-16     | Excelente velocidade |
| **Baixo**   | 4-7 núcleos  | 8-15 GB  | 4-8      | Boa velocidade      |

{% hint style=&quot;success&quot; %}
**Otimização automática**: O CLI deteta automaticamente as especificações do seu sistema e configura o processamento paralelo ideal. Não é necessária nenhuma configuração manual!
{% endhint %}

### Métodos Debayer

O CLI usa **Alta qualidade (mais rápido)** como algoritmo debayer padrão e recomendado:

| Método                      | Qualidade | Velocidade | Descrição                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Alta qualidade (mais rápido)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritmo sensível às bordas (padrão, recomendado) |

### Correção de vinheta

**O que faz:** corrige a queda de luz nas bordas da imagem (cantos mais escuros comuns em imagens de câmera).

* **Ativado por predefinição** - A maioria dos utilizadores deve manter esta opção ativada
* Use `--no-vignette` para desativar

{% hint style=&quot;success&quot; %}
**Recomendação**: Ative sempre a correção de vinheta para garantir um brilho uniforme em todo o quadro.
{% endhint %}

### Calibração da refletância

Converte os valores brutos do sensor em percentagens de refletância padronizadas usando painéis de calibração.

* **Ativado por predefinição** - Essencial para a análise da vegetação
* Requer painéis de alvo de calibração nas imagens
* Use `--no-reflectance` para desativar

{% hint style=&quot;info&quot; %}
**Requisitos**: Certifique-se de que os painéis de calibração estejam devidamente expostos e visíveis nas suas imagens para uma conversão precisa da refletância.
{% endhint %}

### Correções PPK

**O que faz:** Aplica correções cinemáticas pós-processadas usando dados de registo DAQ-A-SD para melhorar a precisão do GPS.

* **Desativado por predefinição**
* Use `--ppk` para ativar
* Requer ficheiros .daq na pasta do projeto do sensor de luz MAPIR DAQ-A-SD.

### Formatos de saída

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profundidade de bits</th><th width="116.5999755859375">Tamanho do ficheiro</th><th>Ideal para</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Inteiro de 16 bits</td><td>Grande</td><td>Análise GIS, fotogrametria (recomendado)</td></tr><tr><td><strong>TIFF (32 bits, percentagem)</strong></td><td>Flutuante de 32 bits</td><td>Muito grande</td><td>Análise científica, investigação</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Inteiro de 8 bits</td><td>Médio</td><td>Inspeção visual, partilha na web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Inteiro de 8 bits</td><td>Pequeno</td><td>Pré-visualização rápida, saída comprimida</td></tr></tbody></table>***

## Automação e scripts

### Processamento em lote do PowerShell

Processe várias pastas de conjuntos de dados automaticamente:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script em lote Windows

Loop simples para processamento em lote:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Python Script de automação

Automação avançada com tratamento de erros:

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Fluxo de trabalho de processamento

### Fluxo de trabalho padrão

1. **Entrada**: Pasta contendo pares de imagens RAW/JPG
2. **Descoberta**: CLI faz a verificação automática de ficheiros de imagem suportados
3. **Processamento**: O modo paralelo adapta-se aos núcleos da sua CPU (Chloros+)
4. **Saída**: Cria subpastas do modelo da câmara com imagens processadas

### Exemplo de estrutura de saída

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimativas de tempo de processamento

Tempos de processamento típicos para 100 imagens (12 MP cada):

| Modo              | Tempo      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Modo paralelo** | 5-10 min  | i7/Ryzen 7, 16 GB de RAM, SSD (até 16 trabalhadores) |
| **Modo paralelo** | 10-15 min | i5/Ryzen 5, 8 GB de RAM, HDD (até 8 trabalhadores)   |

{% hint style=&quot;info&quot; %}
**Dica de desempenho**: O tempo de processamento varia de acordo com a quantidade de imagens, a resolução e as especificações do computador.
{% endhint %}

***

## Resolução de problemas

### CLI não encontrado

**Erro:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluções:**

1. Verifique o local da instalação:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Use o caminho completo se não estiver no PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adicione ao PATH manualmente:
   * Abra Propriedades do sistema → Variáveis de ambiente
   * Edite a variável PATH
   * Adicione: `C:\Program Files\Chloros\resources\cli`
   * Reinicie o terminal

***

### Falha ao iniciar o backend**Erro:**

```

Backend failed to start within 30 seconds
```

**Soluções:**

1. Verifique se o backend já está em execução (feche-o primeiro)
2. Verifique se o Windows Firewall não está a bloquear
3. Tente uma porta diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Force o reinício do backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemas de licença/autenticação**Erro:**

```

Chloros+ license required for CLI access
```

**Soluções:**

1. Verifique se tem uma subscrição ativa do Chloros+
2. Inicie sessão com as suas credenciais:

```powershell
chloros-cli login user@example.com 'password'
```

3. Verifique o estado da licença:

```powershell
chloros-cli status
```

4. Contacte o suporte: info@mapir.camera

***

### Nenhuma imagem encontrada**Erro:**

```

No images found in the specified folder
```

**Soluções:**

1. Verifique se a pasta contém formatos suportados (.RAW, .TIF, .JPG)
2. Verifique se o caminho da pasta está correto (use aspas para caminhos com espaços)
3. Certifique-se de que tem permissões de leitura para a pasta
4. Verifique se as extensões dos ficheiros estão corretas

***

### Processamento trava ou fica pendente**Soluções:**

1. Verifique o espaço disponível em disco (certifique-se de que há espaço suficiente para a saída)
2. Feche outros aplicativos para liberar memória
3. Reduza a quantidade de imagens (processe em lotes)

***

### Porta já em uso**Erro:**

```

Port 5000 is already in use
```

**Solução:**

Especifique uma porta diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Perguntas frequentes

### P: Preciso de uma licença para o CLI?

**R:**Sim! O CLI requer uma licença paga**Chloros+**.

* ❌ Plano padrão (gratuito): CLI desativado
* ✅ Planos Chloros+ (pagos): CLI totalmente ativado

Inscreva-se em: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Posso usar o CLI em um servidor sem GUI?**R:** Sim! O CLI funciona completamente sem interface gráfica. Requisitos:

* Windows Server 2016 ou posterior
* Visual C++ Redistributable instalado
* RAM suficiente (mínimo de 8 GB, recomendado 16 GB)
* Ativação única da licença GUI em qualquer máquina

***

### P: Onde as imagens processadas são guardadas?**R:**Por predefinição, as imagens processadas são guardadas na**mesma pasta que a entrada** em subpastas do modelo da câmara (por exemplo, `Survey3N_RGN/`).

Use a opção `-o` para especificar uma pasta de saída diferente:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### P: Posso processar várias pastas ao mesmo tempo?**R:** Não diretamente em um único comando, mas pode usar scripts para processar pastas sequencialmente. Consulte a secção [Automação e scripts](CLI.md#automation--scripting).***

### P: Como posso guardar a saída CLI num ficheiro de registo?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Lote:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### P: O que acontece se eu pressionar Ctrl+C durante o processamento?**R:** O CLI irá:

1. Parar o processamento de forma adequada
2. Desligar o backend
3. Sair com o código 130

Imagens parcialmente processadas podem permanecer na pasta de saída.

***

### P: Posso automatizar o processamento do CLI?**R:** Com certeza! O CLI foi projetado para automação. Consulte [Automação e scripts](CLI.md#automation--scripting) para exemplos do PowerShell, Batch e Python.***

### P: Como posso verificar a versão do CLI?**R:**

```powershell
chloros-cli --version
```

**Saída:**

```

Chloros CLI 1.0.2
```

***

## Obter ajuda

### Ajuda da linha de comando

Veja as informações de ajuda diretamente no CLI:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canais de suporte

* **E-mail**: info@mapir.camera
* **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preços**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Exemplos completos

### Exemplo 1: Processamento básico

Processe com as configurações padrão (vinheta, refletância):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exemplo 2: Resultado científico de alta qualidade

32 bits flutuante TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exemplo 3: Processamento rápido de pré-visualização

8 bits PNG sem calibração para revisão rápida:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exemplo 4: Processamento corrigido por PPK

Aplicar correções PPK com refletância:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exemplo 5: Localização personalizada da saída

Processar para uma unidade diferente com formato específico:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exemplo 6: Fluxo de trabalho de autenticação

Concluir o fluxo de autenticação:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemplo 7: Utilização multilingue

Alterar o idioma da interface:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
