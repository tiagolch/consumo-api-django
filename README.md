# Tutorial: consumindo uma API com Django

O objetivo deste tutorial é compartilhar o conhecimento obtido por mim após dois dias de pesquisa sobre como consumir uma API usando Django. Precisei disso para o meu projeto integrador na faculdade e estava batendo cabeça porque, às vezes, um arquivo JSON é muito grande e é difícil de manipulá-lo, já que Python não oferece isso de forma nativa. Depois de muita pesquisa, finalmente consegui construir um front-end, ainda que arcaico, mas consumindo de uma API em produção.

## O que isto é
- **Compartilhamento de conhecimentos**: este projeto pretende passar o conhecimento que o autor obteve através de pesquisas

## O que isto não é
Este tutorial não pretende ensinar, completamente
- **Django**: a comunidade do Django tem uma documentação ótima! Você pode acessá-la [clicando aqui](https://www.djangoproject.com/). Pretendo fazer um outro projeto compartilhando meu conhecimento sobre Django também, mas não é o caso deste repositório
- **Python**: a [W3Schools](https://www.w3schools.com) oferece um [tutorial ótimo](https://www.w3schools.com/python/default.asp) sobre Python
- **HTML**: não vamos usar muitos elementos HTML aqui, mas você também pode acessar um [tutorial completo na W3Schools](https://www.w3schools.com/html/default.asp)
- **APIs REST**: definitivamente, não! Sequer usaremos muitos verbos HTTP. Só usaremos o GET. Mas não vamos aprender os princípios REST.
- **Manipulação de JSONs**: o nosso trabalho com JSON será decodificá-lo para uma estrutura que seja compreensível ao Python. Mas é um assunto interessante de se aprender, se você está querendo se aprofundar em consumo de APIs. A W3Schools tem uma [subseção sobre isso](https://www.w3schools.com/js/js_json_intro.asp)

É bastante recomendável que você tenha tido um pré-contato com os itens acima, embora, ocasionalmente, alguns desses assuntos possam vir a ser discutidos brevemente em algum momento

## Funcionamento
Para construir o conhecimento de forma mais concisa, você poderá acompanhar a explicação deste arquivo e todo o código gerado estará disponível na pasta [/codigo](/codigo)

---

# Tutorial

## API: IBGE
Inicialmente, precisaremos escolher uma API existente que nos retorne dados sem necessidade de autenticação, isto é, dados aos quais não precisamos nos identificar para ter acesso. O IBGE, em seu [site de APIs](https://servicodados.ibge.gov.br/api/docs), disponibiliza mais ou menos 12 serviços do tipo. Vamos escolher a API de **localidades**, que é referente às divisões político-administrativas do Brasil. Você pode ler a documentação referente: [API de localidades](https://servicodados.ibge.gov.br/api/docs/localidades?versao=1). Quando fizermos uma requisição do tipo GET para o endereço do serviço, a API nos retornará um JSON carregado de informações referentes.

### Escolha dos recursos
APIs geralmente são baseadas em recursos. Na documentação, conseguimos encontrar a descrição de cada um deles. Mas, a título do exemplo, escolheremos o que nos retorna um JSON carregado de municípios de determinada unidade federativa:

![barra lateral da API do IBGE](/image.png)

Mas para isso, precisamos antes ter em mãos o id da unidade federativa. Em [outra seção do site](https://servicodados.ibge.gov.br/api/docs/localidades?versao=1#api-UFs-estadosGet), conseguimos o id 24, referente a unidade federativa do **Rio Grande do Norte**, inspecionando o [JSON](https://servicodados.ibge.gov.br/api/v1/localidades/estados/) que nos é retornado no navegador:

![JSON visto do navegador Firefox](/image2.png)

**Observação**: Para ter essa visualização, precisei acessar o [endereço](https://servicodados.ibge.gov.br/api/v1/localidades/estados/) pelo Mozilla Firefox. Caso você utilize outro navegador, o retorno talvez possa não ser formatado. Nesse caso, o que você verá é um texto em formato JSON.

De posse do ID, acessaremos: https://servicodados.ibge.gov.br/api/v1/localidades/estados/24/municipios

Esse é o endereço que será efetivamente utilizado. Ele nos retorna todos os municípios do Rio Grande do Norte, bem como informações de sua micro e mesorregião.

## Projeto Django
Para consumir esses dados, precisaremos apenas das views e dos templates do Django, portanto, podemos ignorar toda a parte dos models e da ORM.

Assumindo que você tenha o módulo `django` instalado no Python, poderemos iniciar o projeto em termos de código:

![Terminal após o início do projeto Django](/image3.png)

**Observação:** após iniciar o projeto, renomeei a pasta gerada para _codigo_.

E depois, uma aplicação chamada `ibge`:

![Terminal após o inicio da aplicação](/image4.png)

### Configuração básica
Na pasta da aplicação, criaremos um diretório chamado _templates_, contendo um arquivo _index.html_. Subindo um nível, criaremos um arquivo chamado _urls.py_:

![printscreen da árvore de diretórios](/image10.png)

Também adicionei a aplicação recém-criada ao projeto:

![printscreen do arquivo de configuração do projeto](/image6.png)

Após isso, teremos que configurar as rotas:

![em cima, arquivo de urls do projeto, em baixo, da aplicação](/image7.png)

Na imagem acima, temos dois arquivos de URLs abertos. O de cima é o o projeto. O de baixo, é o recém-criado, da aplicação.

Depois de fazer essas configurações básicas do Django, está na hora de views e templates

## Views do Django
No código das [views](/codigo/ibge/views.py) da aplicação, temos a seguinte ordem:
- Na linha 1, importamos o módulo que vai facilitar o trabalho de encaminhamento dos templates ao usuário
- Na linha 2, importamos o módulo que trabalha com requisições HTTP (talvez você tenha que instalá-lo pelo pip: `pip install requests`)
- Na linha 4, abrimos a função que atenderá ao usuário quando ele digitar a URL [localhost:8000](localhost:8000)
- Na linha 5, colocamos a URL da API anteriormente selecionada
- Na linha 6, acionamos o método `get()` do módulo requests, passando a string `api` como parâmetro
  - A requisição GET viaja até a API
- Na linha 7, pegamos uma lista de retorno de `requisicao`
  - O método `json()` só sera bem sucedido se a string de resposta de `requisicao` se esta for escrita em formato JSON
  - Como isso é uma demonstração, assumiremos que vai dar certo e não trataremos possíveis exceções
- Na linha 8, declaramos um dict vazio
  - `dicionario` vai armazenar os valores do list
  - Template não trabalha com lists, só com dicts
- Na linha 10, estamos preenchendo nosso dict
- Na linha 12, colocamos esse dict em uma variável que será encaminhada para o template
- Na linha 16, a view retorna o template index.html para o usuário e a variável de contexto para o desenvolvedor

## Templates do Django
No [código de template](/codigo/ibge/templates/index.html), fazemos uso de um HTML simples, misturado com alguns padrões da _linguagem de templates Django_
- Na linha 10, fazemos um loop dentro do dicionário que foi passada dentro da variável de contexto na view
  - O dicionário vem carregado de int, str e outros dicts:

  ![printscreen de parte do JSON retornado visto do Firefox](/image8.png)

  - Temos, para cada item do dicionário um id (int), um nome (str) e uma microrregião (dict).
- Com esses dados, podemos mostrar ao usuário:
  - O nome de cada municipio
  - O id de cada municipio
  - O nome de cada microrregião
  - O nome de cada mesorregião
