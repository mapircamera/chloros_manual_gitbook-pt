# GUI: Projetos

O Chloros permite-lhe criar projetos que podem ser reabertos no futuro. Um projeto é uma pasta simples (dentro da sua Pasta de Projetos) que contém:

* `project.json` — definições do projeto, lista de ficheiros e preferências de visualização
* `cameras.json` — câmaras e matrizes ligadas enquanto o projeto estava aberto, com as respetivas definições
* `sensors.json` — sensores de luz DAQ ligados enquanto o projeto estava aberto, além das ligações câmara↔sensor
* as suas capturas, gravações `.daq` e pastas de resultados processados

Não existe um formato de ficheiro de projeto proprietário — a pasta e os seus ficheiros JSON constituem o projeto, o que também facilita a cópia, o arquivo e a execução dos projetos a partir do [CLI](CLI.md) ou [Python SDK](api-python-sdk.md).

## Novo projeto

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecione «Novo projeto» no menu principal e introduza um nome único para o seu projeto.

Se tiver guardado algum modelo de projeto, aparece um menu pendente **Selecionar modelo** por baixo do campo do nome — ao escolher um, o novo projeto é iniciado com as definições desse modelo. Os modelos são guardados a partir de [Definições do projeto](project-settings/project-settings.md): introduza um nome no campo «Nome do modelo de projeto» e clique no ícone de guardar.

## Abrir projeto

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>A opção «Abrir projeto» apresenta todos os projetos na sua pasta de projetos, com a opção <strong>«Abrir pasta de projetos»</strong> na parte inferior</p></figcaption></figure>Selecione «Abrir projeto» para ver uma lista dos projetos existentes na pasta de projetos. Se não existirem projetos, o menu lateral secundário não se abrirá. Pode ver alguns projetos criados pela GUI (t1, t2, t3) listados na imagem acima. Os projetos DATE\_TIME foram criados pelo CLI utilizando o esquema de nomenclatura de projetos predefinido. Ao clicar em qualquer nome de projeto, este será aberto.

Ao clicar no botão «Abrir pasta do projeto», abre-se o explorador de ficheiros do seu computador no caminho do projeto. Pode ajustar o caminho do projeto nas [Definições do projeto](project-settings/project-settings.md).

Se algum dos ficheiros de imagem de origem do projeto tiver sido movido ou eliminado desde a última vez que foi aberto, o Chloros apresenta uma caixa de diálogo que indica exatamente quais os ficheiros que faltam, em vez de abrir uma grelha em branco.

## Duplicar projeto

Disponível assim que um projeto estiver aberto. Selecione «Duplicar projeto» para copiar o projeto atual com um novo nome — o Chloros sugere o próximo nome disponível (por exemplo, «MeuProjeto (2)») — e a cópia é aberta imediatamente.

## Adicionar ficheiros

Depois de um projeto estar aberto, selecione «Adicionar ficheiros» no menu principal para adicionar ficheiros de imagem individuais ao projeto atual. Esta funcionalidade espelha a funcionalidade de adição do navegador de ficheiros, mas é acessível diretamente a partir do menu principal por uma questão de conveniência.

## Adicionar pasta

Depois de abrir um projeto, selecione «Adicionar pasta» no menu principal para adicionar pastas de imagens ao projeto atual. Pode selecionar várias pastas de uma só vez. Os ficheiros duplicados são ignorados.

## Iniciar / Parar o processamento

Depois de os ficheiros serem adicionados a um projeto, a opção «Iniciar o processamento» fica disponível no menu principal. Esta ação equivale a clicar no botão Reproduzir/Iniciar na barra superior. Durante o processamento, a opção do menu muda para «Parar o processamento» para permitir que interrompa o fluxo de trabalho.

## Ligar à Câmara / Ligar ao Sensor de Luz

Na parte inferior do menu principal existem dois atalhos de hardware, disponíveis com ou sem um projeto aberto:

* **Ligar à Câmara** — abre o [separador «Câmaras»](lattice/) para ligar uma câmara ou matriz LATTICE.
* **Ligar ao sensor de luz** — abre o [separador «Sensores de luz»](daq/) para ligar um sensor de luz DAQ.

Ligar o hardware enquanto um projeto está aberto guarda-o no projeto (ver abaixo). Sem um projeto, as ligações são válidas apenas para a sessão.

{% hint style="info" %}
Os itens de menu «Adicionar ficheiros», «Adicionar pasta» e «Iniciar/Parar processamento» só ficam visíveis ou ativados quando um projeto está aberto e foram adicionados ficheiros. Proporcionam acesso rápido a ações também disponíveis através da barra lateral do Navegador de ficheiros e dos botões do cabeçalho.
{% endhint %}

## Os projetos memorizam o seu hardware

Novidade na versão 1.2.0: um projeto mantém o hardware que ligar enquanto estiver aberto. As câmaras e os conjuntos de câmaras (com as respetivas definições por câmara, nomes, cores e disposição em grelha) são guardados no `cameras.json`, e os sensores de luz (com nomes, cores e ligações às câmaras) no `sensors.json` — automaticamente, à medida que trabalha.

Quando **reabre** um projeto, o Chloros não acede imediatamente a nenhum hardware. Cada metade volta a ligar-se na primeira vez que aceder ao separador a que pertence:

* Ao abrir a separador **Câmaras**, as câmaras e matrizes guardadas são reconectadas e as suas definições guardadas são reaplicadas.
* Ao abrir o separador **Sensores de Luz**, os sensores DAQ guardados são reconectados.

Desta forma, abrir um projeto apenas para navegar ou exportar imagens nunca ativa o streaming das câmaras. Se um dispositivo guardado não for encontrado quando o seu separador for aberto, uma caixa de diálogo indica quais os dispositivos indisponíveis, para que possa reconectá-los ou removê-los.

## Gravações DAQ e ficheiros .daq num projeto

* As gravações `.daq` feitas enquanto o projeto está aberto (a partir da secção **Sensores de Luz**ou durante as capturas) são**automaticamente adicionadas ao projeto**.
* Os ficheiros `.daq` importados e todas as gravações do projeto são listados na secção **DAQ Light Sensor** das [Definições do Projeto](project-settings/project-settings.md), cada um com o seu perfil de correção de cap.
* Durante o processamento, os ficheiros `.daq` do projeto fornecem iluminação descendente para produtos de refletância — consulte [Formatos de Imagem de Saída](output-image-formats.md).

## Executar um projeto guardado sem interface gráfica

Um projeto guardado pode ser executado sem a interface gráfica:

* **CLI**: O `chloros-cli project open / connect / capture / sensor / align / run` opera com base no caminho da pasta do projeto — consulte a [Referência do CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` devolve um identificador do projeto; `connect_all()` coloca todas as câmaras e sensores guardados em modo ativo com as suas definições guardadas — consulte a [Referência do SDK](reference/sdk-reference.md).
