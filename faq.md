---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmaras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros só suporta o processamento de imagens de câmaras MAPIR — as famílias Survey3 e LATTICE. Consulte a lista de [modelos de câmaras suportados](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmaras na MAPIR Cloud; consulte a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>O Chloros é compatível com câmaras LATTICE?</summary>

Sim. O Chloros 1.3.0 suporta os módulos de câmara LATTICE M3C e M3M de ponta a ponta: **controlo em tempo real**— detetar, ligar, pré-visualizar e capturar a partir do separador «Câmaras» da interface gráfica do utilizador (GUI), `chloros-cli lattice`, ou do SDK SDK, incluindo matrizes sincronizadas de várias câmaras com sincronização temporal PTP — e**processamento radiométrico completo** das capturas (bruto → sem bayering → radianância → refletância → índice). Consulte [Câmaras suportadas](supported-cameras.md) e o [guia da LATTICE](lattice/README.md).

</details>

<details>

<summary>Posso calibrar as minhas imagens para refletância sem um alvo de calibração?</summary>

**Survey3:** Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo que as imagens que não são do alvo, não será possível relacionar os valores dos píxeis da imagem com uma percentagem de refletância conhecida. Se também não incluir o registo de um sensor de luz MAPIR, o espectro da luz ambiente não será medido e os resultados de refletância não serão precisos.

**LATTICE:** Sim. A refletância pode ser referenciada à irradiância descendente medida por um sensor de luz DAQ em vez de um painel (ρ = π·L/E). Quando um alvo dentro do enquadramento que tenha sido aprovado no controlo de qualidade *está* presente, torna-se a referência absoluta por predefinição (`--reflectance-source auto`), a menos que seja aprovado com um sinalizador, caso em que o alvo e as cadeias do DAQ competem com base na pontuação medida. Uma exceção: «A refletância F988 é calibrada utilizando um painel de refletância na cena: a banda situa-se fora da gama calibrada do sensor de luz do DAQ, pelo que o Chloros aplica a sua captura de painel mais recente e mantém-na entre as observações do painel.» Consulte [Alvos de Calibração](calibration-targets.md).

</details>

<details>

<summary>Preciso de um sensor de luz DAQ?</summary>

Não para a radianância: as exportações de radianância do LATTICE provêm da calibração radiométrica de fábrica de cada câmara e não necessitam nem de um sensor DAQ nem de um alvo. Para a **refletância**, é necessária uma referência para a luz ambiente — quer seja uma medição descendente de um sensor de luz DAQ, quer seja um alvo de calibração no enquadramento. Um sensor DAQ permite-lhe produzir refletância calibrada**sem colocar quaisquer painéis na cena**. Os ficheiros `.daq` gravados são automaticamente associados às suas imagens através do carimbo de data/hora. Consulte [Alvos de Calibração](calibration-targets.md) e a [Referência da CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso utilizar o Chloros com um assistente de IA (Claude, ChatGPT, etc.)?</summary>

Sim — este manual e a CLI/SDK foram concebidos para isso:

* O índice completo do manual está disponível em `https://mapir.gitbook.io/chloros/llms.txt` para que os assistentes de IA possam descobrir todas as páginas.
* O código Markdown bruto de cada página está disponível no URL da página em minúsculas, com `.md` acrescentado (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência da CLI](reference/cli-reference.md) e a [Referência do SDK](reference/sdk-reference.md) foram redigidas para utilização por LLM: sinalizadores exatos, valores predefinidos, semântica de saída e comandos que podem ser copiados e colados.

Consulte [Assistentes de IA](ai-assistants.md) para saber como configurar o seu assistente para utilizar o Chloros.

</details>

<details>

<summary>Para onde vão os meus ficheiros de saída processados?</summary>

Os produtos são guardados na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **pasta-da-câmara** — `LATT-<sensor>-<lens>-F<filter>` para o LATTICE, `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para o Survey3
* **pasta-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32`
* **pastas de produtos** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (sempre na pasta `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — Apenas multiespectral LATTICE: máscaras por pixel ao lado das pastas de produtos, ver abaixo**Os ficheiros exportados mantêm o nome do ficheiro de origem — a pasta identifica o produto, não um sufixo do nome do ficheiro.**Com a CLI, a pasta do projeto é criada na sua pasta de projetos predefinida (`Documents/MAPIR/Chloros Projects`, a menos que a tenha alterado), a menos que indique `-o`; um `-o` relativo é interpretado em relação à pasta a partir da qual executa o comando. Note que uma execução de `chloros-cli process` que tenha solicitado produtos, mas não tenha gravado nenhum, imprime `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, para que os scripts possam detetá-la. Consulte [Formatos de imagem de saída](output-image-formats.md) e a [Referência da CLI](reference/cli-reference.md).

</details>

<details>

<summary>O que são as pastas «clamp_masks» e os ficheiros *_clampmask.tif?</summary>

Novidade na versão 1.3.0. Os produtos de radiação e refletância de uma câmara multiespectral LATTICE M3C realizam uma separação de bandas 3×3 e, sempre que isso produz um valor negativo numa banda, o Chlorosa limita esse valor a zero. Um produto com pelo menos um pixel cortado recebe uma máscara `uint8` de banda única que indica quais os píxeis e bandas que foram cortados (bit 0 = canal R do produto, bit 1 = G, bit 2 = B). As máscaras são guardadas numa pasta `clamp_masks/` **ao lado** das pastas dos produtos — por exemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — e a etiqueta XMP do produto, `Chloros:UnmixClampMask`, identifica a sua máscara (ou indica `none`).

Não se trata de imagens da cena: o próprio processo de importação do Chloros ignora-as e, como nunca se encontram dentro de uma pasta de produto, pode enviar uma pasta de produto para o Pix4D ou o Metashape tal como está. Consulte [Máscaras de fixação](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Por que razão o Chloros indica «A pasta do projeto é demasiado profunda para este computador»?</summary>

Isto aplica-se apenas ao Windows. O Windows vem com os caminhos longos desativados, pelo que nenhum caminho de ficheiro pode ter mais de 260 caracteres. Os produtos Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto, pelo que o Chloros recusa uma pasta de projeto mais profunda antes do processamento — em vez de executar e não gravar nada, como podia acontecer antes da versão 1.3.0. A mensagem indica a pasta de projeto mais longa que cabe.

Resolva o problema de uma das seguintes formas:

* Utilize uma pasta de projeto mais curta (o **Diretório de Trabalho** nas Definições do Projeto, ou `-o` / `chloros-cli set-project-folder` para a CLI).
* Ative os caminhos longos do Windows e reinicie o Chloros. A opção **Ativar caminhos longos do Windows** do instalador do Chloros (marcada por predefinição) faz isso; volte a executar o instalador se a tiver desmarcado.

Consulte [Iniciar o processamento](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Posso editar as minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros parte do princípio de que os dados de entrada não foram modificados. Não altere os nomes dos ficheiros.

</details>

<details>

<summary>Posso configurar as minhas câmaras MAPIR Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagens do Survey3 têm de ter uma exposição fixa/bloqueada, pelo que não é permitida a velocidade de obturação automática nem o ISO automático. Todas as imagens do mesmo modelo de câmara têm de ter velocidade de obturação e ISO (exposição) idênticos.

As câmaras LATTICE não têm esta restrição: o Chloros controla a sua exposição em tempo real (Smart AE) e cada captura regista a exposição e o ganho efetivamente utilizados, o que é tido em conta pelo pipeline radiométrico.

</details>

<details>

<summary>O Chloros consegue processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas são suportadas imagens individuais de câmaras MAPIR, não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de deteção de alvos do Chloros?</summary>

Na tabela do navegador de ficheiros, a pré-seleção das imagens-alvo na coluna da direita indicará ao Chloros para procurar alvos de calibração apenas nessas imagens, acelerando significativamente o processamento.

</details>

<details>

<summary>Se pretender carregar as minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">a MAPIR Cloud,</a> devo processá-las no Chloros antes do carregamento?</summary>

Se pretender carregar as imagens para a nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes do carregamento. A Cloud realizará todo o processamento e muito mais.

</details>

<details>

<summary>O MAPIR virá a suportar a funcionalidade X? Gostaria muito que o MAPIR oferecesse a funcionalidade X.</summary>

Estamos sempre interessados em receber feedback sobre os nossos produtos. Se detetar algum problema com os nossos produtos ou tiver alguma sugestão sobre como podemos melhorá-los, por favor [CONTACTE-NOS](https://www.mapir.camera/community/contact) para partilhar as suas ideias. A maior parte da nossa investigação e desenvolvimento é orientada pela atenção às principais necessidades dos nossos clientes.

</details>

<details>

<summary>O Chloros está disponível para Linux?</summary>

Sim! O Chloros 1.3.0 suporta Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) através dos pacotes `.deb`. A CLI e o SDK Python são totalmente suportados no Linux, incluindo o controlo em tempo real de câmaras LATTICE e sensores DAQ. Não existe uma GUI para o Linux — toda a interação é feita através da [CLI](CLI.md) ou do [SDK Python](api-python-sdk.md). Consulte a [Visão Geral do Linux](linux/linux-overview.md) para obter mais detalhes.

</details>

<details>

<summary>Posso executar o Chloros no NVIDIA Jetson?</summary>

Sim! O Chloros é compatível com a **família Orin** da NVIDIA Jetson — Orin Nano, Orin NX e AGX Orin — que executam o JetPack 6. Os modelos mais antigos (Jetson Nano, TX2, Xavier NX) não podem executar o JetPack 6 e não são suportados pelo pacote atual. O Chloros deteta automaticamente o seu modelo de Jetson e otimiza a sua estratégia de processamento. O pacote Jetson utiliza a compilação CUDA do PyTorch do JetPack 6, pelo que o processamento é executado na GPU do Jetson. Consulte o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obter instruções de configuração e implementação.

</details>

<details>

<summary>O Chloros otimiza automaticamente para o meu hardware?</summary>

Sim! O Chloros inclui a [Adaptação Dinâmica de Computação](processing-architecture/dynamic-compute-adaptation.md) que deteta automaticamente a sua CPU, GPU, RAM e (no Jetson) sensores térmicos. Em seguida, seleciona a estratégia de processamento ideal — desde o `GPU_PARALLEL` em GPUs com grande capacidade de memória até ao `GPU_SINGLE` em GPUs mais pequenas e ao `CPU_PARALLEL` em sistemas sem uma GPU CUDA que o PyTorch possa utilizar. Não é necessária qualquer configuração manual.

</details>

<details>

<summary>O que é o pipeline de processamento de 4 threads?</summary>
O

Chloros utiliza uma arquitetura em pipeline de 4 threads para utilizadores do Chloros+: A thread 1 (Detecção) carrega imagens e deteta alvos de calibração; a thread 2 (Calibração) calcula a calibração de refletância; a thread 3 (Processamento) realiza o debayering acelerado por GPU e o cálculo do índice; e a thread 4 (Exportação) grava os ficheiros de saída. É possível ter várias imagens em diferentes threads simultaneamente para obter o máximo rendimento. Consulte [Pipeline de Processamento](processing-architecture/processing-pipeline.md) para mais detalhes.

</details>

<details>

<summary>Como posso executar diagnósticos na minha instalação do Chloros?</summary>

Utilize o comando `selftest` para executar um teste de funcionamento de 7 etapas: versão, disponibilidade de portas, arranque do backend, conectividade da API (`/api/test`), informações do sistema (`/api/system-info` — GPU/CUDA/PyTorch), presença do modelo de redução de ruído e prontidão da CUDA + modelo de redução de ruído:

```bash
chloros-cli selftest
```

Isto é especialmente útil em sistemas Linux para verificar a configuração da GPU e do CUDA. Num Jetson, o código `CUDA: False` significa normalmente que faltam as bibliotecas CUDA e cuDNN do JetPack 6: instale-as com o comando `sudo apt install nvidia-jetpack`.

</details>
