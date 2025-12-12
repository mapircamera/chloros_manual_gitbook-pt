---
description: Painéis medidos em laboratório usados ​​para calibrar dados capturados no pós-processamento
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Alvos de confiança

O MAPIR oferece vários alvos específicos para cobrir uma variedade de aplicações. O compacto T4-R50 visto abaixo contém 4 painéis que foram medidos quanto à refletância de luz de 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Os alvos de referência difusa T4 têm as seguintes curvas de refletância, [download de dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectance :: 400-1000nm</p></figcaption></figure>

Olhando para os gráficos de refletância, você pode ver que os valores são comprimento de onda (eixo x) versus porcentagem de refletância (eixo y). Quando capturamos uma imagem do alvo específico, criamos uma relação entre o valor do pixel e a porcentagem de refletância, dentro do espectro ao qual cada uma das bandas do sensor da câmera é sensível.

Isso significa que, com cada imagem capturada com nossas câmeras, você pode usar uma foto de nossos alvos de refletância, como o [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) para calibrar as imagens quanto à refletância. Uma vez calibrado, cada pixel da imagem é igual à porcentagem de refletância.

Se você produzir imagens calibradas em Cloros como JPG ou TIFF típico, uma porcentagem de refletância será calculada dividindo o valor do pixel pela profundidade de bits do formato da imagem. Portanto, para JPG divide por 255 e para TIFF divide por 65.535. Você também pode escolher o formato PERCENT de saída em Cloros e cada pixel variará de um valor percentual de 0,0 a 1,0 (0% a 100% de refletância). Apenas tenha em mente que alguns aplicativos de imagem não podem aceitar imagens de porcentagem (ponto flutuante) e são grandes em termos de armazenamento.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
