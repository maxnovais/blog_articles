# Alterando Propriedade CSS via Javascript

## Conceito
Alterar propriedades de CSS de qualquer elemento da página.
- - -
## HTML

    <div id="box"></div>

    <button type="button" onclick="button_click('red');">Vermelho</button>
    <button type="button" onclick="button_click('green');">Verde</button>
    <button type="button" onclick="button_click('blue');">Azul</button>
- - -

## Javascript
    function button_click(color){
      document.getElementById("box").style.backgroundColor=color;
    }
- - -

## Observações
Note que o objeto citado para mudança é o box, chamado pela função `getElementById` sequenciado pela propriedade a ser trabalhada `backgroundColor`, essa recebe o valor da variável passada quando chamamos o .`onclick="button_click('red');`, que se obtém a cor vermelha.
- - -
## Exemplo Prático
[Flapy Control - Material Design]
[Flapy Control - Material Design]:http://maxnovais.github.io/themes/flapy_control_md/
