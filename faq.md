---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmaras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros suporta apenas o processamento de imagens de câmaras MAPIR — das famílias Survey3 e LATTICE. Consulte a lista de [modelos de câmaras suportados](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmaras na MAPIR Cloud; consulte a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>O Chloros suporta câmaras LATTICE?</summary>

Sim. O Chloros 1.2.0 suporta os módulos de câmara LATTICE M3C e M3M de ponta a ponta: **controlo em tempo real**— detetar, ligar, pré-visualizar e capturar a partir do separador «Câmaras» da interface gráfica do utilizador, `chloros-cli lattice` ou o Python SDK, incluindo conjuntos sincronizados de várias câmaras com sincronização temporal PTP — e**processamento radiométrico completo** das capturas (bruto → sem efeito bayes → radianância → refletância → índice). Consulte [Câmaras compatíveis](supported-cameras.md) e o [guia do LATTICE](lattice/README.md).

</details>

<details>

<summary>Posso calibrar as minhas imagens para refletância sem um alvo de calibração?</summary>

**Survey3:** Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo que as imagens que não são do alvo, não será possível relacionar os valores dos píxeis da imagem com uma percentagem de refletância conhecida. Se também não incluir o registo de um sensor de luz MAPIR, o espectro da luz ambiente não será medido e os resultados de refletância não serão precisos.**LATTICE:** Sim. A refletância pode ser referenciada à irradiância descendente medida por um sensor de luz DAQ em vez de um painel (ρ = π·L/E). Quando *está* presente um alvo no quadro que tenha passado no controlo de qualidade, este torna-se a referência absoluta por predefinição (`--reflectance-source auto`). Uma exceção: «A refletância F988 é calibrada utilizando um painel de refletância na cena: a banda situa-se fora da gama calibrada do sensor de luz DAQ, pelo que o Chloros aplica a sua captura de painel mais recente e mantém-na entre as observações do painel.» Consulte [Alvos de calibração](calibration-targets.md).

</details>

<details>

<summary>Preciso de um sensor de luz DAQ?</summary>

Não para a radianância: as exportações de radianância do LATTICE provêm da calibração radiométrica de fábrica de cada câmara e não necessitam nem de um sensor de luz DAQ nem de um alvo. Para a **refletância**, é necessária uma referência para a luz ambiente — quer seja uma medição descendente de um sensor de luz DAQ, quer seja um alvo de calibração no enquadramento. Um sensor DAQ permite-lhe produzir refletância calibrada**sem colocar quaisquer painéis na cena**. Os ficheiros `.daq` gravados são automaticamente associados às suas imagens através do carimbo de data/hora. Consulte [Alvos de Calibração](calibration-targets.md) e a [Referência CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso utilizar o Chloros com um assistente de IA (Claude, ChatGPT, etc.)?</summary>

Sim — este manual e os ficheiros CLI/SDK foram concebidos para esse fim:

* O índice completo do manual está disponível em `https://mapir.gitbook.io/chloros/llms.txt` para que os assistentes de IA possam encontrar todas as páginas.
* O código Markdown bruto de cada página está disponível na respetiva página em minúsculas URL, com `.md` acrescentado (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência CLI](reference/cli-reference.md) e [Referência SDK](reference/sdk-reference.md) foram redigidas para utilização por LLM: sinalizadores exatos, valores predefinidos, semântica de saída e comandos que podem ser copiados e colados.

Consulte [Assistentes de IA](ai-assistants.md) para saber como configurar o seu assistente para o Chloros.

</details>

<details>

<summary>Para onde vão os meus ficheiros de saída processados?</summary>

Os produtos são guardados na pasta do projeto, agrupados por câmara e, em seguida, por formato de ficheiro:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **pasta-da-câmara** — `LATT-<sensor>-<lens>-F<filter>` para LATTICE, `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para Survey3
* **pasta-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32`
* **pastas de produtos** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (sempre na pasta `tiff32`), `<INDEX>_Index_Images/`**Os ficheiros exportados mantêm o nome do ficheiro de origem — a pasta identifica o produto, não um sufixo do nome do ficheiro.**Com o CLI, a pasta do projeto é criada ao lado da pasta de entrada, a menos que se passe o `-o`. Note que uma execução do `chloros-cli process` que tenha solicitado produtos, mas não tenha gravado nenhum, imprime `Processing finished but wrote no image products.` e**termina com um valor diferente de zero**, para que os scripts possam detetá-la. Consulte [Formatos de imagem de saída](output-image-formats.md) e a [Referência do CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso editar as minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros pressupõe que os dados de entrada não foram modificados. Não altere os nomes dos ficheiros.

</details>

<details>

<summary>Posso configurar as minhas câmaras MAPIR e Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagem do Survey3 têm de ter uma exposição fixa/bloqueada, pelo que não é possível utilizar a velocidade de obturação automática nem o ISO automático. Todas as imagens do mesmo modelo de câmara têm de ter velocidade de obturação e ISO (exposição) idênticos.

As câmaras LATTICE não têm esta restrição: o Chloros controla a exposição em tempo real (Smart AE) e cada captura regista a exposição e o ganho efetivamente utilizados, o que é tido em conta pelo pipeline radiométrico.

</details>

<details>

<summary>O Chloros consegue processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas são suportadas imagens individuais da câmara MAPIR, e não imagens unidas, como um mapa ortomosaico.

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

Estamos sempre interessados em receber feedback sobre os nossos produtos. Se detetar algum problema com os nossos produtos ou tiver alguma sugestão sobre como podemos melhorá-los, por favor [CONTACTE-NOS](https://www.mapir.camera/community/contact) para partilhar as suas ideias. A maior parte da nossa I&amp;D é orientada pela atenção às principais necessidades dos nossos clientes.

</details>

<details>

<summary>O Chloros está disponível para o Linux?</summary>

Sim! O Chloros 1.2.0 suporta o Linux amd64 (x86_64) e o arm64 (NVIDIA Jetson JetPack 6) através dos pacotes do `.deb`. O CLI e o Python SDK são totalmente suportados no Linux, incluindo o controlo em tempo real de câmaras LATTICE e sensores DAQ. Não existe uma interface gráfica (GUI) para o Linux — toda a interação é feita através do [CLI](CLI.md) ou do [Python SDK](api-python-sdk.md). Consulte a [Visão geral do Linux](linux/linux-overview.md) para obter mais detalhes.

</details>

<details>

<summary>Posso executar o Chloros no NVIDIA Jetson?</summary>

Sim! O Chloros é compatível com as plataformas NVIDIA Jetson, incluindo o Jetson Nano, o Orin Nano, o Orin NX e o AGX Orin com o JetPack 6. O Chloros deteta automaticamente o seu modelo Jetson e otimiza a sua estratégia de processamento. Consulte o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obter instruções de configuração e implementação.

</details>

<details>

<summary>O Chloros otimiza-se automaticamente para o meu hardware?</summary>

Sim! O Chloros inclui a [Adaptação Dinâmica de Computação](processing-architecture/dynamic-compute-adaptation.md), que deteta automaticamente a sua CPU, GPU, RAM e (no Jetson) sensores térmicos. Em seguida, seleciona a estratégia de processamento ideal — desde o `GPU_PARALLEL` em sistemas com muita memória até ao `GPU_SINGLE` em dispositivos com recursos limitados e ao `CPU_PARALLEL` em sistemas sem uma GPU NVIDIA. Não é necessária qualquer configuração manual.

</details>

<details>

<summary>O que é o pipeline de processamento de 4 threads?</summary>

O Chloros utiliza uma arquitetura em pipeline de 4 threads para utilizadores do Chloros+: A thread 1 (Detecção) carrega imagens e deteta alvos de calibração, a thread 2 (Calibração) calcula a calibração de refletância, a thread 3 (Processamento) realiza o debayering acelerado por GPU e o cálculo do índice, e a thread 4 (Exportação) grava os ficheiros de saída. É possível ter várias imagens em diferentes threads simultaneamente para obter o máximo rendimento. Consulte [Pipeline de Processamento](processing-architecture/processing-pipeline.md) para mais detalhes.

</details>

<details>

<summary>Como posso executar diagnósticos na minha instalação do Chloros?</summary>

Utilize o comando `selftest` para executar um teste de funcionamento em 7 etapas: versão, disponibilidade de portas, arranque do backend, conectividade do API (`/api/test`), informações do sistema (`/api/system-info` — GPU/CUDA/PyTorch), presença do modelo de redução de ruído e prontidão da CUDA + redutor de ruído:

```bash
chloros-cli selftest
```

Isto é especialmente útil em sistemas Linux/Jetson para verificar a configuração da GPU e do CUDA.

</details>
