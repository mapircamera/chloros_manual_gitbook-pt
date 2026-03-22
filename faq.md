---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmaras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros suporta apenas o processamento de imagens de câmaras MAPIR. Consulte a lista de [modelos de câmaras suportados](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmaras na MAPIR Cloud; consulte a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Posso calibrar as minhas imagens para refletância sem um alvo de calibração?</summary>

Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo que as imagens sem alvo, não será possível relacionar os valores de píxeis da imagem com uma percentagem de refletância conhecida. Se também não incluir o registo de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Posso editar as minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros pressupõe que os dados de entrada não foram modificados. Não altere os nomes dos ficheiros.

</details>

<details>

<summary>Posso configurar as minhas câmaras MAPIR e Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagem do Survey3 devem ter uma exposição fixa/bloqueada, pelo que não é possível utilizar a velocidade do obturador automática nem o ISO automático. Todas as imagens do mesmo modelo de câmara devem ter velocidade do obturador e ISO (exposição) idênticos.

</details>

<details>

<summary>O Chloros pode processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas são suportadas imagens individuais de câmara do MAPIR, não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de deteção de alvos do Chloros?</summary>

Na tabela do navegador de ficheiros, a pré-seleção das imagens-alvo na coluna da direita indicará ao Chloros para procurar alvos de calibração apenas nessas imagens, acelerando significativamente o processamento.

</details>

<details>

<summary>Se eu for carregar as minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">o MAPIR Cloud,</a> devo processá-las no Chloros antes do carregamento?</summary>

Se planeia carregar para a nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes do carregamento. A Cloud realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>O MAPIR irá alguma vez suportar a funcionalidade X? Gostaria muito que o MAPIR oferecesse X.</summary>

Estamos sempre interessados em receber feedback sobre os nossos produtos. Se encontrar algum problema com os nossos produtos ou tiver uma sugestão sobre como podemos melhorá-los, por favor [CONTACTE-NOS](https://www.mapir.camera/community/contact) para partilhar as suas ideias. A maior parte da nossa I&amp;D é orientada pela escuta das principais necessidades dos nossos clientes.

</details>

<details>

<summary>O Chloros está disponível para o Linux?</summary>

Sim! O Chloros 1.1.0 suporta o Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) através dos pacotes `.deb`. O CLI e o Python SDK são totalmente suportados no Linux. Não existe GUI para o Linux — toda a interação é feita através do [CLI](CLI.md) ou do [Python SDK](api-python-sdk.md). Consulte a [Visão geral do Linux](linux/linux-overview.md) para obter mais detalhes.

</details>

<details>

<summary>Posso executar o Chloros no NVIDIA Jetson?</summary>

Sim! O Chloros 1.1.0 suporta plataformas NVIDIA Jetson, incluindo Jetson Nano, Orin Nano, Orin NX e AGX Orin com JetPack 6. O Chloros deteta automaticamente o seu modelo Jetson e otimiza a sua estratégia de processamento. Consulte o [Guia NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obter instruções de configuração e implementação.

</details>

<details>

<summary>O Chloros otimiza automaticamente para o meu hardware?</summary>

Sim! O Chloros 1.1.0 inclui [Adaptação Dinâmica de Computação](processing-architecture/dynamic-compute-adaptation.md) que deteta automaticamente a sua CPU, GPU, RAM e (no Jetson) sensores térmicos. Em seguida, seleciona a estratégia de processamento ideal — desde o `GPU_PARALLEL` em sistemas com muita memória até ao `GPU_SINGLE` em dispositivos com recursos limitados e ao `CPU_PARALLEL` em sistemas sem uma GPU NVIDIA. Não é necessária qualquer configuração manual.

</details>

<details>

<summary>O que é o pipeline de processamento de 4 threads?</summary>

O Chloros 1.1.0 utiliza uma arquitetura em pipeline de 4 threads para utilizadores do Chloros+: O Thread 1 (Detecção) carrega imagens e deteta alvos de calibração, o Thread 2 (Calibração) calcula a calibração de refletância, o Thread 3 (Processamento) realiza o debayering acelerado por GPU e o cálculo do índice, e o Thread 4 (Exportação) grava os ficheiros de saída. Várias imagens podem estar em diferentes threads simultaneamente para obter o máximo rendimento. Consulte [Pipeline de Processamento](processing-architecture/processing-pipeline.md) para obter detalhes.

</details>

<details>

<summary>Como posso executar diagnósticos na minha instalação do Chloros?</summary>

Utilize o comando `selftest` para executar 7 diagnósticos do sistema, incluindo verificação da versão, disponibilidade de portas, arranque do backend, conectividade API, informações do sistema, modelos de denoiser e disponibilidade de CUDA:

```bash
chloros-cli selftest
```

Isto é especialmente útil em sistemas Linux/Jetson para verificar a configuração da GPU e da CUDA.

</details>
