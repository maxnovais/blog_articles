# Como usar Flask (Parte II)

Até aqui, tivemos uma introdução de algumas ferramentas em Python e criamos os
ambientes virtuais para facilitar a integração das bibliotecas necessárias e não
sujar o sistema. A idéia nessa segunda parte é interagir com o Flask e o
WTForms.

## Hello Flask

Flask é um microframework bem bacana, principalmente porque em minutos você terá
uma aplicação em execução, crie um arquivo chamado hello.py e introduza esse
código:

```Python
# coding: utf-8
from flask import Flask


app = Flask(__name__)


@app.route('/')
def hello():
    return 'Hello word!'


if __name__ == '__main__':
    app.run()
```

Agora, no terminal com sua virtualenv, rode o comando `python hello.py`, isso
fará com que um servidor rode localmente o arquivo e responda na porta 5000,
abra seu navegador e digite `http://localhost:5000` e irá demonstrar um bom e
velho **Hello World!** no seu navegador.

### Entendendo como funciona

A primeira coisa que iremos fazer aqui é determinar uma aplicação por meio do
Flask, então, usamos a variável `app` e atribuímos um `Flask(__name__)` que é a
nossa aplicação.

No final do arquivo é uma indicação de um script, ou seja, quando o arquivo é
chamado através do Python, ele irá executar um método `run()` do app e iniciará
o servidor.

### Rotas

Eu nunca entendi isso muito bem, mas conforme foi passando o tempo, esse
conhecimento acabou vindo por osmose, estamos falando do decorator `@app.route`
que usamos aí. Esse, significa que a rota que usamos irá retornar a função
decorada, aconselho ler um pouco sobre como funciona os [decorators][0].

Bom, ele quem faz com que o servidor reconheça o caminho e retorne a sua
requisição. Podemos brincar um pouco com ele, vamos criar uma outra função
dentro do nosso arquivo **hello.py**:

```Python
@app.route('/hi')
def hi():
    return '<h1>Hi Human!</h1>'
```

Vamos reiniciar o servidor e acessar o o endereço `http://localhost:5000/hi`
para vermos a mensagem. Podemos também incluir outras informações na rota, como
por exemplo pegar o valor de uma variável pela rota, vamos adicionar o código
abaixo:

```Python
@app.route('/hi/<name>')
def hi_name(name):
    return '<h1>Hi Human, your name is {}?'.format(name)
```

Esse caso aqui ainda é mais bacana, pois acessando
`http://localhost:5000/hi/Max`, ele retorna uma mensagem customizada para o
usuário. :)

Bom, o arquivo está diponível no [Github][1], caso quiser olhar como é sua
versão final.

### Opcional

***Observação*** Aqui também introduzimos uma coisa nova, o `str.format()`, esse
formata a string encaixando dados dinamicos a sua string. Como é algo de Python,
vou fazer um overview somente. Primeiramente, instale o iPython no seu ambiente
com `pip install ipython`. E depois vamos rodar apenas `ipython` para ter acesso
a um python por linha de comando, só que turbinado.

Vamos fazer uma brincadeira legal:

```Python
In [1]: name = 'Max'

In [2]: print('Meu nome é {}'.format(name))
Meu nome é Max

In [3]: print('Meu nome é {name}'.format(name=name))
Meu nome é Max

In [4]: title = 'Sr.'

In [5]: print('Eu sou o {}{}'.format(title, name))
Eu sou o Sr.Max

In [6]: print('Eu sou o {title}{name}'.format(title=title, name=name))
Eu sou o Sr.Max

In [7]: print('Eu sou o {title}{name}'.format(title='Dr.', name='Mario'))
Eu sou o Dr.Mario

In [8]: print('Eu sou o {}{}'.format(name, title))
Eu sou o MaxSr.

In [9]: print('Eu sou o {title}{name}'.format(name=name, title=title))
Eu sou o Sr.Max

In [10]: print('Eu sou o {title}{name}'.format(name, title))
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-10-e6d4bf17b5a6> in <module>()
----> 1 print('Eu sou o {title}{name}'.format(name, title))

KeyError: 'title'
```

Veja que nas linhas de código introduzimos várias formas de se usar o
`str.format()`, na linha 2, 5 e 8 são os mais clássicos, ou seja, é incluso o
valor da variável pela posição dos `{}` com a posição da entrada das variáveis
na função. Isso, por exemplo, sofre alteração caso alterassemos o posicionamento
das entradas na função `str.format()` na linha 8.

Já nos casos das linhas 3, 6, e 9 não ocore a troca de posição, por que aquelas
chaves esperam receber os valores passados como argumentos da função e isso não
é posicional, ou seja, não adianta passar a variável no formato da string e não
representá-lo como um argumento, como acontece na linha 10.

[0]:https://wiki.python.org/moin/PythonDecorators
[1]:https://github.com/maxnovais/flapy_tweets/blob/master/hello.py
