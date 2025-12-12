# CLI: Linha de Comando

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

O **Chloros CLI** fornece acesso poderoso de linha de comando ao mecanismo de processamento de imagens do Chloros, permitindo automação, scripts e operação sem comando para seus fluxos de trabalho de imagens.

### Principais recursos

* 🚀 **Automação** – Processamento em lote de scripts de vários conjuntos de dados
* 🔗 **Integração** – Incorporação em fluxos de trabalho e pipelines existentes
* 💻 **Operação sem cabeça** - Execute sem GUI
* 🌍 **Multilíngue** - Suporte para 38 idiomas
* ⚡ **Processamento Paralelo** - Escala dinamicamente para sua CPU (até 16 trabalhadores paralelos)

### Requisitos

| Requisito | Detalhes |
| -------------------- | ---------------------------------------------------------------------------------- |
| **Sistema operacional** | Windows 10/11 (64 bits) |
| **Licença** | Cloros+ ([é necessário plano pago](https://cloud.mapir.camera/pricing)) |
| **Memória** | Mínimo de 8 GB de RAM (recomendado 16 GB) |
| **Internet** | Necessário para ativação da licença |
| **Espaço em discoteca** | Varia de acordo com o tamanho do projeto |

{% dica estilo = "aviso" %}
**Requisito de licença**: A CLI requer uma assinatura paga do Chloros+. Os planos padrão (gratuitos) não têm acesso CLI. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para atualizar.
{% endhint %}

## Início rápido

### Instalação

A CLI é automaticamente incluída no instalador do Chloros:

1. Baixe e execute **Chloros Installer.exe**
2. Conclua o assistente de instalação
3. CLI installed to: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% dica estilo = "sucesso" %}
O instalador adiciona automaticamente `chloros-cli` ao PATH do seu sistema. Reinicie seu terminal após a instalação.
{% endhint %}

### Configuração inicial

Antes de usar a CLI, ative sua licença Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Uso Básico

Processe uma pasta com configurações padrão:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referência de comando

### Sintaxe Geral

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandos

### `process` - Processar imagens

Processe imagens em uma pasta com experiência.

**Sintaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemplo:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opções de comando de processo

| Opção | Tipo | Padrão | Descrição |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<pasta de entrada>` | Caminho | _Obrigatório_ | Pasta contendo imagens multiespectrais RAW/JPG |
| `-o, --saída` | Caminho | O mesmo que entrada | Pasta de saída para imagens processadas |
| `-n, --nome do projeto` | Corda | Gerado automaticamente | Nome do projeto personalizado |
| `--vinheta` | Bandeira | Habilitado | Ativar correção de vinheta |
| `--sem-vinheta` | Bandeira | - | Desativar correção de vinheta |
| `--refletância` | Bandeira | Habilitado | Ativar calibração de refletância |
| `--no-reflectance` | Bandeira | - | Desativar calibração de refletância |
| `--ppk` | Bandeira | Desativado | Aplicar correções PPK a partir de dados do sensor de luz .daq |
| `--formato` | Escolha | TIFF (16 bits) | Formato de saída: `TIFF (16 bits)`, `TIFF (32 bits, porcentagem)`, `PNG (8 bits)`, `JPG (8 bits)` |
| `--min-tamanho-alvo` | Inteiro | Automóvel | Tamanho mínimo do alvo em pixels para detecção do painel de calibração |
| `--target-clustering` | Inteiro | Automóvel | Limite de agrupamento alvo (0-100) |
| `--exposição-pin-1` | Corda | Nenhum | Bloquear exposição para modelo de câmera (Pino 1) |
| `--exposição-pin-2` | Corda | Nenhum | Bloquear exposição para modelo de câmera (Pino 2) |
| `--recal-intervalo` | Inteiro | Automóvel | Intervalo de recalibração em segundos |
| `--timezone-offset` | Inteiro | 0 | Deslocamento de fuso horário em horas |

***

### `login` - Autenticar conta

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

***

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

***

### `status` - Verifique o status da licença

Exibir licença atual e status de autenticação.

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

### `export-status` - Verifica o progresso da exportação

Monitore o progresso da exportação da Linha 4 durante ou após o processamento.

**Sintaxe:**

```bash
chloros-cli export-status
```

**Exemplo:**

```powershell
chloros-cli export-status
```

**Caso de uso:** Chame este comando enquanto o processamento está em execução para verificar o progresso da exportação.

***

### `linguagem` - Gerenciar idioma da interface

Visualize ou altere o idioma da interface CLI.

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

| Código | Idioma | Nome nativo |
| ------- | --------------------- | ---------------- |
| `pt` | Inglês | Inglês |
| `é` | Espanhol | Espanhol |
| `pt` | Português | Português |
| `fr` | Francês | Français |
| `de` | Alemão | Alemão |
| `isso` | Italiano | Italiano |
| `já` | Japonês | 日本語 |
| `ko` | Coreano | 한국어 |
| `zh` | Chinês (simplificado) | 简体中文 |
| `zh-TW` | Chinês (tradicional) | 繁體中文 |
| `ru` | Russo | Russo |
| `nl` | Holandês | Holanda |
| `ar` | Árabe | العربية |
| `pl` | Polonês | Polaco |
| `tr` | Turco | Turquia |
| `oi` | Hindi | हिंदी |
| `id` | Indonésio | Bahasa Indonésia |
| `vi` | Vietnamita | Tiếng Việt |
| `th` | Tailandês | ไทย |
| `sv` | Sueco | Sueca |
| `da` | Dinamarquês | Dinamarquês |
| `não` | Norueguês | Norueguês |
| `fi` | Finlandês | Suomi |
| `el` | Grego | Eλληνικά |
| `cs` | Tcheco | Čeština |
| `hu` | Húngaro | Magiar |
| `ro` | Romeno | Română |
| `Reino Unido` | Ucraniano | Українська |
| `pt-BR` | Português Brasileiro | Português Brasileiro |
| `zh-HK` | Cantonês | 粵語 |
| `ms` | Malaio | Bahasa Melayu |
| `sk` | Eslovaco | Eslovena |
| `bg` | Búlgaro | Búlgaro |
| `hr` | Croata | Hrvatsky |
| `lt` | Lituano | Lietuvių |
| `lv` | Letão | Letão |
| `et` | Estoniano | Este |
| `sl` | Esloveno | Eslovênia |

{% dica estilo = "sucesso" %}
**Persistência automática**: sua preferência de idioma é salva em `~/.chloros/cli_language.json` e persiste em todas as sessões.
{% endhint %}

***

### `set-project-folder` - Definir pasta de projeto padrão

Altere o padrão local da pasta do projeto (compartilhada com uma GUI).

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

Exibe o padrão local atual da pasta do projeto.

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

Redefinir a pasta do projeto para o padrão local.

**Sintaxe:**

```bash
chloros-cli reset-project-folder
```

***

## Opções globais

Estas opções se aplicam a todos os comandos:

| Opção | Tipo | Padrão | Descrição |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Caminho | Detectado automaticamente | Caminho para o executável de back-end |
| `--porta` | Inteiro | 5000 | Número da porta da API de back-end |
| `--reiniciar` | Bandeira | - | Forçar reinicialização de back-end (elimina processos existentes) |
| `--versão` | Bandeira | - | Mostrar informações da versão e sair |
| `--ajuda` | Bandeira | - | Mostrar informações de ajuda e sair |

**Exemplo com opções globais:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guia de configurações de processamento

### Processamento Paralelo

Cloros+ CLI **escalona automaticamente** o processamento paralelo para especificações das capacidades do seu computador:

**Como funciona:**

* Detecta seus núcleos de CPU e RAM
* Aloca trabalhadores: **2× núcleos de CPU** (usa hyperthreading)
* **Máximo: 16 trabalhadores paralelos** (para estabilidade)

**Níveis do sistema:**

| Tipo de sistema | CPU | memória RAM | Trabalhadores | Desempenho |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Alta qualidade** | Mais de 16 núcleos | Mais de 32 GB | Até 16 | Velocidade máxima |
| **Intermediário** | 8-15 núcleos | 16-31 GB | 8-16 | Excelente velocidade |
| **Básico** | 4-7 núcleos | 8-15 GB | 4-8 | Boa velocidade |

{% dica estilo = "sucesso" %}
**Otimização automática**: a CLI detecta automaticamente as especificações do sistema e configura o processamento paralelo ideal. Nenhuma configuração manual necessária!
{% endhint %}

### Métodos Debayer

A CLI usa **Alta qualidade (mais rápida)** como algoritmo debayer padrão e recomendado:

| Método | Qualidade | Velocidade | Descrição |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Alta qualidade (mais rápido)** ⭐ | ⭐⭐⭐⭐ | ⚡⚡⚡ | Algoritmo com reconhecimento de borda (padrão, recomendado) |

### Correção de vinheta

**O que faz:** corrige a queda de luz nas bordas da imagem (cantos mais escuros, comuns em imagens de câmeras).

* **Ativado por padrão** - A maioria dos usuários deve manter esta opção ativada
* Use `--no-vignette` to disable

{% dica estilo = "sucesso" %}
**Recomendação**: Sempre ativa a correção de vinheta para garantir brilho uniforme em todo o quadro.
{% endhint %}

### Calibração de Refletância

Converta valores brutos do sensor em porcentagens de refletância padronizadas usando painéis de deficiência.

* **Ativado por padrão** - Essencial para análise de vegetação
* Solicitar painéis de destino de desaparecidos em imagens
* Use `--no-reflectance` to disable

{% dica estilo = "info" %}
**Requisitos**: Certifique-se de que os painéis de fiação estejam devidamente expostos e visíveis em suas imagens para uma conversão de refletância precisa.
{% endhint %}

### Correções PPK

**O que faz:** Aplique correções cinemáticas pós-processadas usando dados de registro DAQ-A-SD para melhorar as resultados do GPS.

* **Desativado por padrão**
* Use `--ppk` to enable
* Solicite arquivos .daq na pasta do projeto do sensor de luz MAPIR DAQ-A-SD.

### Formatos de saída

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bit Depth</th><th width="116.5999755859375">File Size</th><th>Best For</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit integer</td><td>Large</td><td>GIS analysis, photogrammetry (recommended)</td></tr><tr><td><strong>TIFF (32-bit, Percent)</strong></td><td>32-bit float</td><td>Very Large</td><td>Scientific analysis, research</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit integer</td><td>Medium</td><td>Visual inspection, web sharing</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit integer</td><td>Small</td><td>Quick preview, compressed output</td></tr></tbody></table>

***

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

### Script em lote do Windows

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

### Script de automação Python

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

1. **Entrada**: pasta contendo pares de imagens RAW/JPG
2. **Descoberta**: a CLI verifica automaticamente arquivos de imagem compatíveis
3. **Processamento**: o modo paralelo é dimensionado de acordo com os núcleos da CPU (Chloros+)
4. **Saída**: Cria subpastas de modelo de câmera com imagens processadas

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

Tempos típicos de processamento para 100 imagens (12 MP cada):

| Modo | Tempo | Ferragens |
| ----------------- | --------- | -------------------------------------------- |
| **Modo Paralelo** | 5-10 minutos | i7/Ryzen 7, 16 GB de RAM, SSD (até 16 trabalhadores) |
| **Modo Paralelo** | 10-15 minutos | i5/Ryzen 5, 8 GB de RAM, HDD (até 8 trabalhadores) |

{% dica estilo = "info" %}
**Dica de desempenho**: o tempo de processamento varia de acordo com a contagem de imagens, resolução e especificações do computador.
{% endhint %}

***

## Solução de problemas

### CLI não encontrada

**Erro:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluções:**

1. Verifique o local de instalação:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Use o caminho completo se não estiver em PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adicione o PATH manualmente:
   * Abra Propriedades do Sistema → Variáveis ​​de Ambiente
   * Editar variável PATH
   * Add: `C:\Program Files\Chloros\resources\cli`
   * Reinicie o terminal

***

### Falha ao iniciar o back-end

**Erro:**

```
Backend failed to start within 30 seconds
```

**Soluções:**

1. Verifique se o backend já está em execução (feche-o primeiro)
2. Verifique se o Firewall do Windows não está bloqueando
3. Experimente uma porta diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forçar renovação do back-end:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemas de licença/autenticação

**Erro:**

```
Chloros+ license required for CLI access
```

**Soluções:**

1. Verifique se você tem uma assinatura ativa do Chloros+
2. Faça login com suas credenciais:

```powershell
chloros-cli login user@example.com 'password'
```

3. Verifique o status da licença:

```powershell
chloros-cli status
```

4. Entre em contato com o suporte: info@mapir.camera

***

### Nenhuma imagem encontrada

**Erro:**

```
No images found in the specified folder
```

**Soluções:**

1. Verifique se a pasta contém formatos suportados (.RAW, .TIF, .JPG)
2. Verifique se o caminho da pasta está correto (use aspas para caminhos com espaços)
3. -se de ter permissões de leitura certifique-se para a pasta
4. Verifique se as extensões dos arquivos estão corretas

***

### Processamento paralisado ou travado

**Soluções:**

1. Verifique o espaço em disco disponível (garanta espaço suficiente para saída)
2. Baixe outros aplicativos para liberar memória
3. Reduza a contagem de imagens (processe em lotes)

***

### Porta já em uso

**Erro:**

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

### P: Preciso de uma licença para CLI?

**R:** Sim! A CLI requer uma licença paga **Chloros+**.

* ❌ Plano padrão (gratuito): CLI desativado
* ✅ Planos Cloros+ (pagos): CLI totalmente habilitado

Inscreva-se em: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Posso usar uma CLI em um servidor sem GUI?

**R:** Sim! A CLI funciona completamente sem comando. Requisitos:

* Windows Server 2016 ou posterior
* Visual C++ Redistribuível instalado
* RAM suficiente (mínimo de 8 GB, recomendado 16 GB)
* Ativação única da licença GUI em qualquer máquina

***

### P: Onde as imagens processadas são salvas?

**R:** Por padrão, as imagens processadas são salvas na **mesma pasta da entrada** nas subpastas do modelo da câmera (por exemplo, `Survey3N_RGN/`).

Use a opção `-o` para especificar uma pasta de saída diferente:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### P: Posso processar várias massas de uma vez?

**R:** Não diretamente em um comando, mas você pode usar scripts para processar pastas sequencialmente. Consulte a seção [Automação e scripts](CLI.md#automation--scripting).

***

### P: Como salvou a saída da CLI em um arquivo de log?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Lote:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### P: O que acontece se eu pressionar Ctrl+C durante o processamento?

**R:** A CLI irá:

1. Pare de processar normalmente
2. Desligue o back-end
3. Saia com o código 130

Imagens parcialmente processadas podem permanecer na massa de saída.

***

### P: Posso automatizar o processamento CLI?

**R:** Com certeza! A CLI foi projetada para automação. Consulte [Automação e scripts](CLI.md#automation--scripting) para obter exemplos de PowerShell, Lote e Python.

***

### P: Como verificar a versão CLI?

**HUM:**

```powershell
chloros-cli --version
```

**Saída:**

```
Chloros CLI 1.0.2
```

***

## Obtendo ajuda

### Ajuda da linha de comando

Visualize informações de ajuda diretamente na CLI:

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
* **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preços**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Exemplos completos

### Exemplo 1: Processamento Básico

Processo com configurações padrão (vinheta, refletância):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Exemplo 2: Produção Científica de Alta Qualidade

TIFF flutuante de 32 bits:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Exemplo 3: Processamento rápido de visualização

PNG de 8 bits sem exceção para revisão rápida:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Exemplo 4: Processamento corrigido por PPK

Aplique correções PPK com refletância:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Exemplo 5: Local de saída personalizado

Processo para uma unidade diferente com formato específico:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Exemplo 6: Fluxo de trabalho de autenticação

Fluxo de autenticação completo:

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

### Exemplo 7: Uso de vários idiomas

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
