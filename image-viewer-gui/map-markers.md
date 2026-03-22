# Marcadores do mapa

O separador «Mapa» apresenta as suas imagens num mapa 2D interativo com base nas suas coordenadas GPS. Isto proporciona uma visão geral geográfica da sua sessão de captura e ajuda-o a visualizar a cobertura espacial. É também útil aquando da importação inicial das suas imagens para eliminar rapidamente quaisquer imagens que não precise de processar.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Aceder ao separador Mapa

1. Abra ou crie um projeto no Chloros
2. Importe imagens que contenham metadados GPS
3. Clique no separador **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na barra lateral esquerda
4. O mapa exibirá marcadores na localização GPS de cada imagem

{% hint style="info" %}
**GPS necessário**: Apenas as imagens com coordenadas GPS incorporadas nos seus metadados EXIF aparecerão no mapa. Certifique-se de que a sua câmara tem o GPS ativado durante a captura.
{% endhint %}

***

## Ajustar imagens a partir do separador Mapa

O separador **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> tem o mesmo botão  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  e remoção  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que a guia [**Navegador de Ficheiros**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Também mostra a mesma lista de ficheiros do projeto, mas com cabeçalhos de coluna diferentes:

### Nome do ficheiro

* Nome original do ficheiro da câmara
* Mantém a convenção de nomenclatura da câmara (por exemplo, IMG\_0001.RAW)

### Latitude

* A latitude da imagem

### Longitude

* A longitude da imagem

### Altitude

* A altitude da imagem

{% hint style="info" %}
Clicar nos cabeçalhos das colunas da tabela também ordena os dados das linhas
{% endhint %}

***

## Marcadores de imagem

Cada imagem com dados GPS é representada por um marcador no mapa:

### Exibição de marcadores

* Os marcadores indicam as coordenadas GPS exatas onde cada imagem foi capturada
* Marcadores agrupados podem ficar juntos quando se diminui o zoom
* Aumente o zoom para ver as localizações individuais das imagens

{% hint style="success" %}
SUPER-ZOOM: Quando atinge o nível máximo de zoom do fornecedor de mosaicos do mapa, o mosaico é então ampliado ao aumentar ainda mais o zoom, permitindo-lhe ver marcadores que estão próximos uns dos outros.
{% endhint %}

### Pré-visualização ao passar o cursor

* **Passe o cursor do rato** sobre qualquer marcador para ver uma pré-visualização em miniatura dessa imagem
* Isto permite uma identificação visual rápida sem sair da visualização do mapa
* Útil para localizar imagens específicas dentro de uma sessão de captura de grandes dimensões

***

## Fornecedores de mosaicos de mapa

{% hint style="success" %}
**Seleção automática**: o Chloros escolhe automaticamente o serviço de mosaicos que fornece o melhor nível de zoom para a sua localização atual no mapa. Pode alternar manualmente entre fornecedores, se desejar.
{% endhint %}

O separador «Mapa» suporta dois fornecedores de mosaicos para as imagens de fundo do mapa:

### Google Maps

* Imagens padrão de satélite e de mapa do Google
* Ideal para cobertura geral a nível mundial

### ESRI

* Imagens de satélite e aéreas do ESRI ArcGIS
* Frequentemente fornece imagens de maior resolução em determinadas regiões

***

## Tipos de mosaicos de mapa

Pode escolher o tipo de camada de mapa (da esquerda para a direita):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreno

Mostra perfis de elevação e mosaicos de mapa com detalhes (estradas, etc.)

### Mapa

Mostra mosaicos de mapa padrão (baixa largura de banda) com detalhes (estradas, etc.)

### Satélite

Mostra mosaicos de mapa de satélite detalhados (alta largura de banda)

### Híbrido

Mostra mosaicos de mapa de satélite com detalhes adicionais (estradas, etc.)

***

## Navegação no mapa

### Controlos de zoom

* **Aumentar/Diminuir zoom**: Utilize a roda do rato ou os botões de zoom
* **Ecrã inteiro**: Coloca o mapa em ecrã inteiro

### Controlos de panorâmica

* **Panorâmica**: Clique e arraste para se deslocar pelo mapa***

## Casos de utilização

### Visualização da trajetória de voo

* Visualize a área de cobertura das sessões de captura por drone
* Identifique lacunas na cobertura de imagens
* Verifique a execução da trajetória de voo

### Revisão do levantamento terrestre

* Veja a distribuição espacial das capturas terrestres
* Localize imagens de alvos de calibração em relação à área de levantamento
* Planeie locais de captura adicionais

### Controlo de qualidade

* Identifique rapidamente imagens capturadas em locais inesperados
* Verifique a precisão do GPS em todo o conjunto de dados
* Faça uma referência cruzada entre as localizações das imagens e as notas de campo

***

## Resolução de problemas

### Não aparecem marcadores

**Possíveis causas:**

* As imagens não contêm metadados de GPS
* O GPS estava desativado na câmara durante a captura
* Os dados EXIF foram removidos por software externo

**Solução**: Verifique se o GPS está ativado na sua câmara e reimporte os ficheiros originais

### Marcadores na localização errada

**Possíveis causas:**

* O GPS da câmara teve uma fixação de satélite fraca
* Desvio do GPS durante a captura

**Solução**: Trata-se normalmente de um problema relacionado com o momento da captura; considere utilizar GPS PPK/RTK para aplicações de precisão
