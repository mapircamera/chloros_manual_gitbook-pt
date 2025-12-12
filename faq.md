---
description: Perguntas frequentes
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Can I process images from cameras that are not MAPIR brand with Chloros?</summary>

Não, o Chloros suporta apenas o processamento de imagens de câmeras MAPIR. Consulte a lista de [modelos de câmeras compatíveis](supported-cameras.md) para obter mais informações. Oferecemos processamento de outras câmeras no MAPIR Cloud, veja a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Can I calibrate my images for reflectance without a calibration target?</summary>

Não. Sem uma imagem do alvo de precisão capturada próximo ao momento em que as imagens não são alvo de capturadas, você não será capaz de relacionar os valores de pixel da imagem a uma porcentagem de refletância conhecida. Se você também não incluir o registro de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Can I edit my images prior to processing in Chloros?</summary>

Não. A Chloros assume que os dados de entrada não foram modificados. Não altere os nomes dos arquivos.

</details>

<details>

<summary>Can I set my MAPIR Survey3 cameras to auto exposure and process the images in Chloros?</summary>

Os conjuntos de dados de imagens Survey3 devem ter uma exposição fixa/bloqueada, portanto, não há velocidade do obturador automático ou ISO automático. Todas as imagens do mesmo modelo de câmera devem ter velocidade de obturador e ISO (exposição) idênticas.

</details>

<details>

<summary>Can Chloros process or analyze orthomosaic images?</summary>

Não. Somente imagens individuais de câmeras MAPIR são suportadas, e não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>How can I speed up the target detection step of Chloros?</summary>

Na tabela do navegador de arquivos, a pré-seleção das imagens alvo na coluna da direita dirá ao Cloros para procurar apenas alvos de restrição nessas imagens, acelerando bastante o processamento.

</details>

<details>

<summary>If I will upload my images to <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> should I process in Chloros prior to uploading?</summary>

Se você planeja fazer upload para nossa plataforma de processamento on-line [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes de fazer upload. A nuvem realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>Will MAPIR ever support X feature? I really wish MAPIR offered X.</summary>

Estamos sempre interessados ​​em receber feedback sobre nossos produtos. Se você encontrar um problema com nossos produtos ou tiver uma sugestão sobre como podemos melhorá-los, [ENTRE EM CONTATO CONOSCO](https://www.mapir.camera/community/contact) para compartilhar suas idéias. A maior parte de nossa pesquisa e desenvolvimento é orientada por ouvir as maiores necessidades de nossos clientes.

</details>
