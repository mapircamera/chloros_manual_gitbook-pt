---
description: Perguntas frequentes
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Can I process images from cameras that are not MAPIR brand with Chloros?</summary>

No, Chloros only supports processing MAPIR camera images. Please see the list of [supported camera models](supported-cameras.md) for more information. We do offer processing of other cameras on MAPIR Cloud, see full list [here](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Can I calibrate my images for reflectance without a calibration target?</summary>

Não. Sem uma imagem do alvo de calibração capturada próximo ao momento em que as imagens não alvo são capturadas, você não será capaz de relacionar os valores de pixel da imagem a uma porcentagem de refletância conhecida. Se você também não incluir o registro de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Can I edit my images prior to processing in Chloros?</summary>

Não. A Chloros assume que os dados de entrada não foram modificados. Não altere os nomes dos arquivos.

</details>

<details>

<summary>Can I set my MAPIR Survey3 cameras to auto exposure and process the images in Chloros?</summary>

Os conjuntos de dados de imagens Survey3 devem ter uma exposição fixa/bloqueada, portanto, não há velocidade do obturador automático ou ISO automático. Todas as imagens do mesmo modelo de câmera devem ter velocidade de obturador e ISO (exposição) idênticos.

</details>

<details>

<summary>Can Chloros process or analyze orthomosaic images?</summary>

Não. Somente imagens individuais de câmeras MAPIR são suportadas, e não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>How can I speed up the target detection step of Chloros?</summary>

Na tabela do navegador de arquivos, a pré-seleção das imagens alvo na coluna da direita dirá ao Chloros para procurar apenas alvos de calibração nessas imagens, acelerando bastante o processamento.

</details>

<details>

<summary>If I will upload my images to <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> should I process in Chloros prior to uploading?</summary>

If you plan to upload to our online processing platform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) do not edit the images prior to uploading. Cloud will perform all the same processing and more.

</details>

<details>

<summary>Will MAPIR ever support X feature? I really wish MAPIR offered X.</summary>

We are always interested in receiving feedback on our products. If you find an issue with our products, or have a suggestion on how we can improve our products please [CONTACT US](https://www.mapir.camera/community/contact) to share your thoughts. Most of our R\&D is guided by listening to our customer's biggest needs.

</details>
