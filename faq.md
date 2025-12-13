---
description: Perguntas frequentes
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---
# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmaras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros só suporta o processamento de imagens de câmaras MAPIR. Consulte a lista de [modelos de câmaras suportados](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmaras na MAPIR Cloud. Consulte a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Posso calibrar as minhas imagens para refletância sem um alvo de calibração?</summary>

Não. Sem uma imagem do alvo de calibração capturada por volta do momento em que as imagens não alvo são capturadas, não será possível relacionar os valores de pixel da imagem a uma percentagem de refletância conhecida. Se também não incluir o registo de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Posso editar as minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros assume que os dados de entrada não foram modificados. Não altere os nomes dos ficheiros.

</details>

<details>

<summary>Posso definir as minhas câmaras MAPIR Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagens Survey3 devem ter uma exposição fixa/bloqueada, portanto, sem velocidade do obturador automática ou ISO automático. Todas as imagens do mesmo modelo de câmara devem ter velocidade do obturador e ISO (exposição) idênticos.

</details>

<details>

<summary>O Chloros pode processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas imagens individuais da câmara MAPIR são suportadas, não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de deteção de alvos do Chloros?</summary>

Na tabela do navegador de ficheiros, pré-selecionar as imagens-alvo na coluna da direita indicará ao Chloros para procurar apenas nessas imagens os alvos de calibração, acelerando consideravelmente o processamento.

</details>

<details>

<summary>Se eu for carregar as minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">o MAPIR Cloud,</a> devo processá-las no Chloros antes de carregá-las?</summary>

Se planeia fazer o upload para a nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes de fazer o upload. A Cloud realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>O MAPIR alguma vez suportará a funcionalidade X? Gostaria muito que o MAPIR oferecesse X.</summary>

Estamos sempre interessados em receber feedback sobre os nossos produtos. Se encontrar algum problema com os nossos produtos ou tiver alguma sugestão sobre como podemos melhorá-los, [CONTACTE-NOS](https://www.mapir.camera/community/contact) para partilhar as suas ideias. A maior parte da nossa I&amp;D é orientada por ouvir as principais necessidades dos nossos clientes.

</details>
