# Como usar Flask (Parte I)

Nessa semana, tive uma pequena tarefa, que consiste em pegar dados do Twitter e
colocar em um banco de dados para posterior consulta - isso ocorre pois existe
um certo limite de consultas simultaneas na API do Twitter. Bom, em relação a
isso, aproveitei pra criar um tutorial básico de como usar alguns componentes
bacanas de Flask.

Não é nada muito avançado, muito pelo contrário, é básico e vou tentar abordar
de maneira prática esse tutorial as seguintes ferramentas:

-   Boas práticas de Python
-   Flask
-   Jinja2
-   WTForms
-   Python-Twitter
-   Pymongo
-   Virtualenv
-   Docker

## Sumário

### Flask

Framework pequeno para criação de aplicações web, essa é a principal prioridade
dos ensinamentos aqui, ou seja, aprender um pouco de como o Flask e suas funções
funcionam na prática.

### Jinja2

Linguagem de template, geralmente abordada com os marcadores `{% %}` ou `{{ }}`
dependendo de seu uso, vamos abordar algumas funções dele e como usar.

### WTForms

Talvez uma das coisas mais legais que irei apresentar, se trata de um
gerenciador de formulário via backend, esse daqui é bastante poderoso nessa
função.

### Python-Twitter

Lib de Python para conectar no Twitter e utilizar os dados da API, não tem muito
segredo, a não ser a criação e autenticação via Twitter.

### Pymongo

Cliente oficial de Mongo para Python, ou seja, aqui ele abstrai os métodos do
Mongo para ficar mais Pythonico. :D

### Virtualenv

Imagine criar um ambiente virtual, onde as coisas não dependem mais do seu SO e
das coisas que estão instalada nele, essa é a idéia do virtualenv, criar
ambientes isolados conforme a necessidade.

### Docker

Assim como virtualenv, Docker serve, principalmente, para isolar uma aplicação
e não instalar ela em nosso SO, essa parte é opcional, e não iremos abordar
assuntos como criação de dockerfile ou até mesmo colocar a aplicação em um
container. :(

## Começando

Primeira coisa, crie uma pasta de trabalho, pode ser com qualquer nome, iremos
iniciar com uma pasta chamada `flapy_tweets`.

### Virtualenv e PIP

Bom, se você já conhece sobre virtualenv e já sabe manejar uma, nem precisa
seguir, essa parte. Se você é novato em Python, aconselho buscar formas de
gerenciar seus ambientes virtuais, um conselho, é o uso da [pyenv][0] com o
[plugin virtuaenv][1]. Segue no nome os repositórios oficiais, e esses tem como
instalar localmente *(em inglês)*.

Vamos seguir usando *pyenv*, iremos criar uma virtualenv:

```shell
pyenv virtualenv 3.5.2 flapy_tweets
pyenv activate flapy_tweets
```

Se tudo tiver dado certo, o console irá mostrar
`(flapy_tweets) [nome_da_maquina] $`, com isso, teremos um ambiente isolado na
sua máquina chamado **flapy_tweets**. Agora prosseguiremos com a instalação das
bibliotecas necessárias usando pip:

```shell
pip install flask flask-wtf pymongo python-twitter
```

Quando for executado esse comando, ele irá instalar essas bibliotecas mais as
suas dependencias, se você executar um comando como `pip freeze`, ele irá
retornar todos os pacotes instalados dentro daquele ambiente, aqui vamos criar
o primeiro arquivo:

```shell
pip freeze > requirements.txt
```

Deve ser criado um arquivo com as dependencias usadas no seu sistema, e se você
quiser e/ou precisar instalar em um novo ambiente / sistema, é só usar o comando
`pip install -r requirements.txt` para instalar os requisitos no novo ambiente.

***Opcional:*** Se quiser, pode criar e ativar um outro ambiente e tentar rodar
o comando acima para testar. :)

[0]:https://github.com/yyuu/pyenv
[1]:https://github.com/yyuu/pyenv-virtualenv
