# Django REST Framework

Criei esse artigo baseado na documentação original do Django REST Framework,
aconselho, caso prefira, acesse a [documentação oficial](http://www.django-rest-framework.org/tutorial/1-serialization/).

Diferente do original, dividi esse em duas partes e tirei as comparações que o
tutorial faz para mostrar a abstração do Django REST Framework.

- - -

## 1. Virtualizando o ambiente

Antes de mais nada, aconselho dar uma olhada em outro artigo sobre virtualização
de ambiente, pois existem diversos métodos para isso, como com os pacotes
`python-virtualenv` ou até mesmo com o `pyenv`.

Nesse tutorial vamos utilizar as dependencias abaixo:

-   django
-   djangorestframework
-   pygments

Então, escolha a melhor forma, e instale essas três depêndencias. :D

- - -

## 2. Django Admin

O Django, por si só, cria a estrutura básica de projeto, segundo o site oficial
de Django, tal versão é compátivel tanto com Python 2, quanto com Python 3. Aqui
nesse ponto vamos ver que existe um pequeno problema na criação do projeto na
terceira versão do Python, pois o Django inclui o arquivo `__init__.py` nas
pastas do projeto. Isso não é nada de outro mundo, mas pode comprometer algumas
coisas, principalmente em relação as importações.

```shell
(.venv) ~/project $ django-admin startproject tutorial
(.venv) ~/project $ cd tutorial
(.venv) ~/project/tutorial $ python manage.py startapp snippets
```

Devemos comportar uma estrutura parecida com essa abaixo:

```shell
|-- manage.py
|-- snippets
|   |-- admin.py
|   |-- apps.py
|   |-- __init__.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- tests.py
|   `-- views.py
`-- tutorial
    |-- __init__.py
    |-- settings.py
    |-- urls.py
    `-- wsgi.py
```

Vamos exlcuir o arquivo `tutorial/__init__.py` para não ter problemas de
importação provido do Python 3. Após isso, abra o arquivo `tutorial/settings.py`
e adicione o app a chave `INSTALLED_APPS`:

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```

- - -

## 3. Criando o modelo

Originalmente, criaremos o modelo `Snippet` que guardará snippts de código.
Editaremos o arquivo `snippets/models.py`

```python
# coding: utf-8
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles


LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```

Não iremos sair muito do padrão do tutorial, só iremos alterar algumas partes
para facilitar a compreensão e deixar o tutorial mais enxuto. Nesse modelo, vou
explicar algumas coisas:

-   por convênção, é importante o `# coding: utf-8`, ele quer dizer em que encode foi criado o projeto.
-   A função `get_all_lexers()` retorna um conjunto de linguagens conhecidas pela lib pygments
-   A função `get_all_styles()` trás seus estilos de acordo com a linguagem.

Após fazer isso, vamos novamente a linha de comando:

```shell
(.venv) ~/project/tutorial $ python manage.py makemigrations snippets
(.venv) ~/project/tutorial $ python manage.py migrate
```

- - -

## 4. Nosso primeiro serializer

Nessa parte do tutorial do site, há uma explicação das diferenças dentre o uso
do serializer do Django REST Framework, sinceramente, não adiciona muita coisa
se você não pensa em customizar, assim como é possível com o Django forms.

Se precisar de algo mais customizável, aconselho verificar o tutorial oficial,
caso contrario, podemos continuar:

Crie um arquivo chamado `snippets\serializers.py` e inclua no arquivo:

```python
# coding: utf-8
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')
```

A idéia é que a classe que herdamos `serializers.ModelSerializer` possui os
componentes necessários para exposição dos models através de uma API REST, sem
muito segredo, só colocamos como configuração os models e os campos.

- - -

## 5. Com caminhos fica mais fácil

Vamos adicionar no arquivo `snippets\url.py` as seguintes linhas:

```python
# coding: utf-8
from django.conf.urls import url
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views


urlpatterns = [
    url(r'^snippets/$', views.snippet_list),
    url(r'^snippets/(?P<pk>[0-9]+)/$', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

-   A única coisa explicável é o `format_suffix_patterns()` que ajudará as views a ter um formato ideal de URL
-   As demais coisas são iguais o que já conhecemos no Django.

Por fim, vamos adicionar a url do `snippets` no `tutorial/urls.py` colocando o
código abaixo com o `include()`:

```python
# coding: utf-8
from django.conf.urls import url, include
from django.contrib import admin


urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^', include('snippets.urls'))
]
```

- - -

## 6. Nossas visualizações - Utilizando Mixin

Até aqui, praticamente não codificamos nada, só configuramos o modelo e como
ele terá sua entrada e saída atraves do `serializer`. Essa é a grande mágica do
Django, escrever menos código, testar mais, ser mais produtivo, facilidade de
manutenção, padronização de projetos, dentre outros.

Agora, vamos entrar um pouquinho mais nas visualizações. Abra o arquivo
`snippets\views.py` e adicione aos poucos os códigos abaixo:

```python
# coding: utf-8
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics


class SnippetList(mixins.ListModelMixin, mixins.CreateModelMixin, generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


class SnippetDetail(mixins.RetrieveModelMixin, mixins.UpdateModelMixin, mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

-   Aqui criaremos uma view para retornar toda a lista de snippets e os detalhes
-   Eles herdam uma série de Mixin que geram os métodos e processos necessários
-   Essa lista gera os métodos `GET`, `POST`, `PUT` e `DELETE`

- - -

Até aqui, já podemos rodar nossa aplicação e fazer as inclusão, exclusão e
edição que quisermos, mais pra frente, vou adicionar a parte 2 desse tutorial.
