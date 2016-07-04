# Flask / SQLAlchemy - Queries Tips
## O que é?
SQL Alchemy é um ORM utilizado em Python, geralmente utilizado com Flask, se torna mais fácil criar e gerenciar rotinas que envolvam linguagem SQL, uma vez que o ORM irá tratar o código gerado para o seu SBD-SQL, não se importando qual a plataforma utilizada (Postgres, MySQL, Oracle, SQL Server).
- - -
## Transições Rápidas
### Inserção
Para inserção e edição basta usar o método `.add()` para a inclusão de um novo dado.

    objeto = new objeto
    db.session.add(objeto)
    db.session.commit()

### Edição e Exclusão
Para excluir, inicialmente teremos de localizar o objeto desejado, para isso executaremos a instrução abaixo:

    objeto = Objeto.query.filter_by(id=id).first_or_404()
 
O método `.first_or_404()` assim como diz, trás o primeiro resultado ou nada para a variável **objeto**. Após isso podemos executar o comando de edição:

    db.session.add(objeto)
    db.session.commit()

**Nota:** O comando de edição é o mesmo de inclusão de um novo objeto, a única diferença fica por conta que selecionamos préviamente o objeto a ser editado.

Para excluir, não tem segredo, utilizamos o comando abaixo:

    db.session.delete(objeto)
    db.session.commit()
- - -
## Consultas
Talvez a parte mais complexa disso são as consultas, quando nas instruções anteriores apenas tratamos de forma mais simples, essa trás mais funções e métodos além dos vistos `.filter_by()` e `first_or_404()`.
### Todos os valores
    Objeto.query.all()
### Filtro único
    Objeto.query.filter_by(*condição*)
### Ordenar
    Objeto.query.order_by(Objeto.coluna.asc())
    Objeto.query.order_by(Objeto.coluna.desc())
### Contador
    Objeto.query.count()
- - -
## Filtro múltiplo
Além da função `.filter_by()`, podemos utilizar a função `.filter()`.

    Objeto.query.filter(*condição*)

Digamos que tenhamos um objeto e precisamos filtrar por duas variáveis, um deles **cor** e outra **tamanho**, como se trata de duas variáveis, essas podem não ter valores alocados, conforme código abaixo:

    consulta = Objeto.query
    if variavel_cor:
        consulta = consulta.filter(Objeto.cor.ilike('%'+variavel_cor+'%')
    if variavel_tamanho:
        consulta = consulta.filter(Objeto.tamanho = variavel_tamanho)
    return consulta.all()

No exemplo acima ainda utilizamos um método dentro do filter, o `.ilike()`, esse trás todo conteúdo igual ao valor e não é "case sensitive". Nota ainda que utilizamos o valor `'%' + variavel_cor + '%'`, essa foi uma forma de fazer com que traga todos os objetos que contenham o valor mencionado, não utilizamos o `.contains()`, pois esse é "case sensitive".

No caso do tamanho, geralmente utilizamos medidas padrões, sejam elas, métricas, imperiais ou até mesmo nominais (Pequeno, Médio e Grande), então, usei uma condição comum.