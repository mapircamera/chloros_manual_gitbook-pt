# Marcadores do mapa

O separador «Mapa» representa graficamente as suas imagens num mapa 2D interativo com base nas suas coordenadas GPS. Proporciona-lhe uma visão geral geográfica de uma sessão de captura e é a forma mais rápida, logo após a importação, de eliminar imagens que não pretende processar.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Aceder ao separador «Mapa»

1. Abra ou crie um projeto no Chloros
2. Importe imagens que contenham metadados GPS
3. Clique no separador **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> na barra lateral esquerda
4. O mapa apresenta um marcador na localização GPS de cada imagem

{% hint style="info" %}
**GPS obrigatório**: apenas as imagens com coordenadas GPS nos seus metadados EXIF aparecem no mapa. Uma imagem sem coordenadas continua no projeto e é processada normalmente — simplesmente não tem marcador.
{% endhint %}

***

## Ajustar imagens a partir do separador «Mapa»

O separador **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> dispõe dos mesmos botões de adição <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> e remoção <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> de ficheiros que o separador [**Explorador de ficheiros**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Apresenta a mesma lista de ficheiros do projeto, com colunas geográficas:

| Coluna        | Conteúdo                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nome**      | O nome do ficheiro tal como saiu da câmara                             |
| **Latitude**  | Graus decimais, seis casas decimais                                |
| **Longitude** | Graus decimais, seis casas decimais                                |
| **Altitude**  | Metros, uma casa decimal — `-` quando a imagem não contém altitude |

{% hint style="info" %}
Clique em qualquer cabeçalho de coluna para ordenar por essa coluna; clique novamente para inverter a ordem.
{% endhint %}

{% hint style="warning" %}
**A altitude é a altura acima do nível do mar, não a altura acima do solo.** O valor provém da etiqueta EXIF `GPSAltitude` da imagem, que se refere ao nível médio do mar. Não se trata da altura de voo acima do terreno, e o Chloros não calculará a distância de amostragem ao solo a partir deste valor — sobre um campo a 300 m acima do nível do mar, um dron a 100 m AGL regista aqui cerca de 400 m. Utilize esta coluna para identificar valores atípicos e confirmar uma altitude de voo consistente, e não como uma medição AGL.
{% endhint %}

***

## Marcadores de imagem

Cada imagem com dados GPS recebe um marcador nas suas coordenadas.

### Exibição dos marcadores

* Os marcadores situam-se nas coordenadas exatas registadas para cada captura
* Os marcadores muito próximos uns dos outros podem sobrepor-se visualmente quando se afasta o zoom — aproxime o zoom para os separar
* Os marcadores selecionados e realçados são exibidos acima dos restantes

### Pré-visualização ao passar o cursor

* **Passe o cursor** sobre qualquer marcador para exibir uma miniatura dessa imagem com o nome do ficheiro
* **Clique**num marcador para selecionar a imagem e**fixar** a janela pop-up — esta permanece aberta até clicar noutro local. Enquanto a janela pop-up estiver fixada, passar o cursor sobre outros marcadores não a fecha
* Esta é a forma mais rápida de encontrar um fotograma específico numa sessão extensa sem sair do mapa

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>O separador «Mapa» apresenta todas as imagens com geotags do projeto</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: quando atinge o zoom máximo para o qual o fornecedor de mosaicos dispõe de imagens, continuar a ampliar aumenta os mosaicos em vez de parar, permitindo-lhe distinguir marcadores que se encontram quase uns em cima dos outros.
{% endhint %}

* O super-zoom só é ativado quando se está **no** zoom máximo do fornecedor para essa localização e os mosaicos já terminaram de carregar. Abaixo desse nível, o zoom funciona normalmente
* O intervalo é de **1× a 32×**, além do máximo do próprio fornecedor
* Um indicador no canto mostra o super-zoom atual como uma percentagem, e um botão **×** ao lado dele permite-lhe voltar ao zoom normal com um clique
* Ao diminuir o zoom, a ação é sempre transmitida ao próprio mapa, pelo que nunca fica preso no super-zoom
* O zoom e a panorâmica enquanto se está no super-zoom transferem o deslocamento resultante de volta para o mapa, pelo que a área descentrada para a qual se deslocou continua a solicitar mosaicos em vez de ficar em branco
* Os marcadores são desenhados como elementos vetoriais em vez de serem rasterizados, pelo que se mantêm nítidos em todos os níveis de super-zoom

***

## Fornecedores de blocos de mapa

{% hint style="success" %}
**Seleção automática**: o Chloros escolhe o serviço de blocos que oferece o melhor nível de zoom para a localização das suas imagens. Pode mudar manualmente a qualquer momento.
{% endhint %}

| Fornecedor        | Notas                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Ampla cobertura mundial; suporta os quatro tipos de mosaicos                                                                                                            |
| **Esri ArcGIS**| Imagens aéreas frequentemente de maior resolução em determinadas regiões. O tipo de mosaico**Terrain** não está disponível para o Esri e o respetivo botão fica desativado enquanto o Esri estiver selecionado |***

## Tipos de mosaicos de mapa

Escolha o tipo de camada de mapa com os botões (da esquerda para a direita):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tipo                 | Mostra                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terreno**          | Sombreado de elevação com detalhes do mapa (estradas, rótulos). Apenas Google       |
| **Mapa**              | Mosaicos padrão de mapas de ruas — a opção que requer menos largura de banda              |
| **Satélite**        | Imagens de satélite detalhadas, sem legendas — a opção que consome mais largura de banda |
| **Híbrido** (predefinição) | Imagens de satélite com estradas e legendas sobrepostas                |

O separador «Mapa» abre-se na opção **Híbrido**. A sua escolha é aplicada à mudança de fornecedor, sempre que este o suporte.***

## Navegação no mapa

* **Zoom**: roda de deslocamento do rato ou botões de zoom no mapa
* **Deslocamento**: clicar e arrastar
* **Ecrã inteiro**: o controlo de ecrã inteiro expande o mapa para ocupar toda a janela***

## Casos de utilização

### Revisão da trajetória de voo

* Veja de relance a área de cobertura de uma sessão com drone
* Identifique lacunas onde faltou uma passagem
* Confirme se o voo seguiu o padrão planeado

### Análise de levantamento terrestre

* Veja como as capturas terrestres estão distribuídas
* Localize os quadros-alvo de calibração em relação à área de levantamento
* Decida onde são necessárias capturas adicionais

### Controlo de qualidade

* Encontre imagens captadas em locais inesperados e remova-as antes do processamento
* Ordene por altitude para identificar um quadro captado à altura errada ou em que a localização GPS foi deficiente
* Compare as localizações das imagens com as notas de campo

***

## Resolução de problemas

### Não aparecem marcadores

**Possíveis causas**

* As imagens não contêm metadados de GPS
* O GPS estava desativado na câmara durante a captura
* Os dados EXIF foram removidos por outro software antes da importação

**O que fazer**: confirme se o GPS está ativado na câmara e volte a importar os ficheiros originais. Pode verificar se um ficheiro específico tem coordenadas procurando-o na tabela de ficheiros do separador «Mapa» — uma imagem sem coordenadas não tem nenhuma linha nessa tabela.

### Os marcadores estão no local errado

**Possíveis causas**: uma localização por satélite de má qualidade no momento da captura ou desvio do GPS durante a sessão.**O que fazer**: trata-se de um problema relacionado com o momento da captura e não de algo que o Chloros possa corrigir posteriormente. Para trabalhos de precisão, utilize um fluxo de trabalho com GPS PPK/RTK — consulte a definição**Aplicar correções PPK** em [Definições do projeto](../project-settings/project-settings.md).

### O mapa está em branco ou os mosaicos deixam de carregar

Os fornecedores de mosaicos são serviços online. Se os mosaicos deixarem de chegar, verifique a ligação de rede do equipamento e, em seguida, tente mudar de fornecedor. Se estiver com um zoom muito elevado, prima o botão de reinício **×** para regressar a um nível de zoom normal e permitir que o mapa volte a solicitar os mosaicos.***

## Páginas relacionadas

* [**Grelha de imagens**](image-grid.md) — o mesmo conjunto de imagens que as miniaturas
* [**Abrir uma imagem em ecrã inteiro**](opening-an-image-full-screen.md) — inspecionar uma imagem em detalhe
* [**Adicionar ficheiros a um projeto**](../processing-images-gui/adding-files-to-a-project.md) — os botões para adicionar/remover ficheiros que esta separador partilha
