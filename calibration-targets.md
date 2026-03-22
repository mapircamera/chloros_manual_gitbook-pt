---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Alvos de calibração

A MAPIR oferece vários alvos de calibração para cobrir uma vasta gama de aplicações. O compacto T4-R50, apresentado abaixo, contém 4 painéis cuja refletância da luz foi medida no intervalo de 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Os alvos de referência difusa T4 apresentam as seguintes curvas de refletância, [descarregue os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 400-1000 nm</p></figcaption></figure>Os alvos de referência difusos T4P apresentam as seguintes curvas de refletância, [descarregue os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 400-1000 nm</p></figcaption></figure>Ao observar o gráfico de refletância, pode ver-se que os valores representam o comprimento de onda (eixo x) em função da percentagem de refletância (eixo y). Quando capturamos uma imagem do alvo de calibração, criamos uma relação entre o valor do pixel e a percentagem de refletância, dentro do espectro ao qual cada uma das bandas do sensor da câmara é sensível.

Isto significa que, com cada imagem que capturar com as nossas câmaras, pode utilizar uma fotografia dos nossos alvos de refletância, tais como o [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou o [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), para calibrar as imagens em termos de refletância. Uma vez calibrada, cada pixel da imagem corresponde a uma percentagem de refletância.

Se exportar as imagens calibradas no Chloros como o típico JPG ou no TIFF, a percentagem de refletância é calculada dividindo o valor do pixel pela profundidade de bits do formato da imagem. Assim, para JPG, divida por 255, e para o TIFF, divida por 65 535. Também pode escolher a saída no formato PERCENT no Chloros, e então cada pixel variará entre um valor percentual de 0,0 a 1,0 (0% a 100% de refletância). Tenha apenas em mente que algumas aplicações de imagem não aceitam imagens em percentagem (ponto flutuante), e estas são de grande dimensão em termos de armazenamento.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
