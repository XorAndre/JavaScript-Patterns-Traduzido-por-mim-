### Capítulo-2
---------
## Essentials

Este capítulo discute as melhores práticas, padrões e hábitos essenciais para escrever o código JavaScript Highquality, como evitar globais, usando declarações simples de var, precaching comprimento em loops, seguindo convenções de codificação e muito mais. O capítulo também inclui alguns hábitos não necessariamente relacionados ao código propriamente dito, mas mais sobre o processo de criação de código geral, incluindo a escrita da documentação da API, a realização de revisões por pares e a execução do JSLint. Esses hábitos e práticas recomendadas podem ajudá-lo a escrever um código melhor, mais compreensível e passível de manutenção — código para se orgulhar (e ser capaz de descobrir) ao revisitá-lo meses e anos abaixo da estrada.

### Escrevendo código de fácil manutenção

Bugs de software são dispendiosos de corrigir. E seu custo aumenta ao longo do tempo, especialmente se os bugs rastejar para o produto liberado publicamente. Neste caso a solução ideal seria correção do bug imediatamente, logo que você encontrá-lo; Isto é, quando o problema é seu código resolva em quanto ainda está fresco em sua cabeça.
Caso contrário, você vai passar para outras tarefas e esquecer tudo sobre esse código. Rever o código depois de algum tempo que passou requer:<br> 
* tempo para reaprender e entender o problema<br>
* tempo para entender o código que é posto para resolver o problema de outro problema, específico para maiores projetos ou empresas, pois, a pessoa que irá corrigir o bug não é a mesma pessoa que criou o bug (e também não é a mesma pessoa que encontrou o bug).<br> 
É fundamental reduzir o tempo que leva para entender o código, seja escrito por si mesmo há algum tempo ou escrito por outro desenvolvedor na equipe. É fundamental para a linha de fundo (receita comercial) e a felicidade do desenvolvedor.
Outro fato da vida relacionado ao desenvolvimento de software em geral é que geralmente gasta mais tempo lendo código do que escrevendo. Tem épocas em que você fica focado profundamente em um problema, e você pode se sentar e em uma tarde criar uma quantidade considerável de código.

O código provavelmente irá funcionar, mas como a aplicação amadurece, e isto irá requerer que o seu código seja revisto, várias coisas podem acontecer durante o processo. Por exemplo:
* os bugs são descobertos.<br>
* Novos recursos são adicionados na aplicação.<br>
* A aplicação precisa trabalhar em novos ambientes (por exemplo: se adaptar para novos navegadores).<br>
* O código é revisado.<br>
* O código é completamente reescrito do zero ou portado para outra arquitetura ou até mesmo outra linguagem.<br>
Como resultado das mudanças, as poucos horas gastas escrevendo o código acabam em Man-Weeks (Várias semanas de trabalho) gastas lendo o código. É por isso que criar código de fácil manutenção é fundamental para o sucesso de uma aplicação.
Código de fácil manutenção significa que o código: <br>
* é legível <br>
* é consistente <br>
* é previsível <br>
* parece que foi escrito pela mesma pessoa <br>
* está documentado<br>
O resto deste capítulo irá aborda esses pontos quando se trata de escrever JavaScript.<br>

### Minimizing Globals

JavaScript usa funções para gerenciar escopo. Uma variável declarada dentro de uma função é local para essa função e não está disponível fora da função. Por outro lado, as variáveis globais são aquelas declaradas fora de qualquer função ou simplesmente usadas sem serem declaradas.
Cada ambiente JavaScript tem um objeto global acessível e quando você usa isso fora de qualquer função. Cada variável global que você criar se torna uma propriedade do objeto global. Em navegadores, por conveniência, há uma propriedade adicional do objeto global chamado window que (geralmente) aponta para o objeto global propriamente dito. O trecho do código a seguir mostra como criar e acessar uma variável global em um ambiente do navegador:
```js
myglobal = "hello"; // antipattern
console.log(myglobal); //"hello"
console.log(window.myglobal); //"hello"
console.log(window["myglobal"]); //"hello"
console.log(this.myglobal); //"hello"
```

### O problema com as variáveis a Globais

O problema com variáveis globais é que elas são compartilhados entre todo o código na aplicação. Elas vivem no mesmo namespace global e há sempre uma chance de nomeclaturas colidirem — quando duas partes separadas de uma aplicação definem variáveis globais com o mesmo nome, mas com finalidades diferentes.
Também é comum que páginas da Web incluam código não escrito pelos desenvolvedores da página, por exemplo:

* Uma biblioteca JavaScript de terceiros
* scripts de parceiros
* código de um script de rastreamento e análise de usuários de terceiros 
* diferentes tipos de widgets, badges, e botões

Digamos que um dos scripts de terceiros defina uma variável global, chamada, por exemplo, Result. E depois, em uma de suas funções, você define outra variável global chamada Result. O resultado disso é que a última variável de resultado substitui as anteriores, e o script de terceiros pode simplesmente parar de funcionar.
Portanto, é importante ser um bom vizinho para os outros scripts que podem estar na mesma página e usar o máximo de variáveis globais possível. Mais adiante no livro você irá aprender sobre estratégias para minimizar o número de globais, como o padrão namespacing ou as funções imediatas de execução própria, mas o padrão mais importante para ter menas variáves globals é usar sempre o var para declarar variáveis.
É surpreendentemente fácil criar globais involuntariamente por causa de dois recursos JavaScript.
Primeiro, você pode usar variáveis sem mesmo declará-las. E em segundo lugar, o JavaScript tem a noção de globais implícitas, o que significa que qualquer variável que você não declare se torna uma propriedade do objeto global (e está acessível como uma variável global declarada corretamente).
Considere o exemplo a seguir:

```js
function sum(x, y) {
    // antipattern: implied global
    result = x + y;
    return result;
}
```

No código acima, o resultado é usado sem está declarado. O código funciona bem, mas depois de chamar a função você acaba com mais um resultado variável no namespace global e que pode ser uma fonte de problemas.
A regra empírica é sempre declarar variáveis com var, como demonstrado na versão melhorada da função SUM():
```js
function sum(x, y) {
    var result = x + y;
    return result;
}
```
Outro antipadrão que cria variáveis globais implícitas é a cadeia de atribuições como parte de uma declaração var. No trecho a seguir a é local, mas b torna-se global que provavelmente não é o que você pretendia fazer:
```js
// antipattern, Não utilize
function foo() {
    var a = b = 0;
    // ...
}
```
Se você está se perguntando "por que isso acontece?" é por causa da avaliação da direita para a esquerda. Primeiro, a expressão b = 0 é avaliada e neste caso b não é declarada. O valor de retorno dessa expressão é 0 e é atribuída à nova variável local e declarada com var a. Em outras palavras é como se você digita-se:
```js
var a = (b = 0);
```
Se você já declarou as variáveis, encadeamento em atribuições é bom e não cria variáveis globais inesperadas. veja o Exemplo:
```js
function foo() {
    var a, b;
    // ...
    a = b = 0; 
}
```

### Efeitos colaterais ao esquecer var
Há uma pequena diferença entre variáveis globais implícitas e explicitamente definidas — a diferença está na capacidade de desdefinir essas variáveis usando o operador Delete: 
* As variáveis globais criadas com var (aquelas criadas no programa fora de qualquer função) não podem ser excluídas.
* Globalidades implícitas criadas sem var (independentemente se as funções internas criadas) podem ser excluídas.
Isso mostra que globalidades implícitas são tecnicamente não variáveis reais, mas são propriedades do objeto global. As propriedades podem ser excluídas com o operador Delete enquanto variáveis não podem:
```js
var global_var = 1;
global_novar = 2; // antipattern
(function () {
    global_fromfunc = 3; // antipattern
}());
// attempt to delete
delete global_var; // false
delete global_novar; // true
delete global_fromfunc; // true
// test the deletion
typeof global_var; // "number"
typeof global_novar; // "undefined"
typeof global_fromfunc; // "undefined"
```
No modo estrito ES5, as atribuições a variáveis não declaradas (como os dois antipadrões no trecho anterior) lançarão um erro.

### Acesso ao objeto global

Nos navegadores, o objeto global é acessível a partir de qualquer parte do código através da Propriedade window(a menos que você tenha feito algo especial e inesperado, como declarar uma variável local chamada window). Mas em outros ambientes está propriedade da conveniência pode ser chamada algo mais (ou mesmo não disponível ao programador). Se você precisar acessar o objeto global sem embutir a janela do identificador, você pode fazer o seguinte a partir de qualquer nível de escopo de função aninhada:

```js
var global = (function () {
    return this;
}());
```
Dessa forma, você sempre pode obter o objeto global, porque dentro de funções que foram chamadas como funções (ou seja, não como construtores com um novo) isso deve sempre apontar para o objeto global. Isso na verdade não é mais o caso no ECMAScript5 em modo estrito, então você tem que adotar um padrão diferente quando seu código está em modo estrito. Por exemplo, se você estiver desenvolvendo uma biblioteca, você pode encapsular seu código de biblioteca em uma função imediata (discutiremos no capítulo 4) e, em seguida, do escopo global, passar uma referência a isso como um parâmetro para sua função imediata.

### Único padrão de var

Usando uma única instrução var na parte superior de suas funções é um padrão útil para adotar. Ela tem os seguintes benefícios: 
* fornece um único local para procurar todas as variáveis locais necessárias para a função <br>
* evita erros lógicos quando uma variável é usada antes de ser definida (consulte "levantamento: um problema com VARs dispersos")<br> 
* ajuda a lembrar-se de declarar variáveis e, portanto, minimizar as variáveis globals <br>
* é menos código (para digitar e transferir)<br>
O único padrão com var irá se parecer com isso:
```js
function func() {
    var a = 1,
        b = 2,
        sum = a + b,
        myobject = {},
        i,
        j;
// corpo da função...
}
```
Você usar uma instrução var e declarar várias variáveis delimitadas por vírgulas é uma boa prática e também inicializar a variável com um valor inicial no momento em que você declara. Isso pode impedir erros lógicos (todas as variáveis não inicializadas e declaradas são inicializadas com o valor indefinido) e também melhora a legibilidade do código. Quando você olha para o código mais tarde, você pode ter uma idéia sobre o uso pretendido de uma variável com base em seu valor inicial — por exemplo, era suposto ser um objeto ou um inteiro?
Você também pode fazer algum trabalho real no momento da declaração, como o caso com a Sum = a + b no código anterior. Outro exemplo é quando se trabalha com referências DOM (Document Object Model). Você pode atribuir referências dom a variáveis locais junto com a única declaração, como demonstra o código a seguir:
```js
function updateElement() {
    var el = document.getElementById("result"),
        style = el.style;
        // Faça algo com el e style
}
```

### Hoisting: um problema com VARs dispersas
JavaScript permite que você tenha várias instruções var em qualquer lugar em uma função, e todos elas agem como se as variáveis fosse declaradas na parte superior da função. Esse comportamento é conhecido como hoisting. Isso pode levar a erros lógicos quando você usa uma variável e, em seguida, declará-lo ainda mais na função. Para JavaScript, desde que uma variável esteja no mesmo escopo (mesma função), ela é considerada declarada, mesmo quando é usada antes da declaração var. Dê uma olhada neste exemplo:

```js
// antipattern
myname = "global"; // global variable
function func() {
    alert(myname); // "undefined"
    var myname = "local";
    alert(myname); // "local"
}
func();
```
Neste exemplo, você pode esperar que o primeiro alert() prompt será "global" e o segundo será prompt "local". É uma expectativa razoável porque, no momento do primeiro alerta, myname não foi declarado e, portanto, a função provavelmente deve "ver" o myname global. Mas não é assim que funciona. O primeiro alerta dirá "indefinido" porque myname é considerado declarado como uma variável local para a função. (embora a declaração vem depois.) Todas as declarações de variáveis são içadas para a parte superior da função. Portanto, para evitar esse tipo de confusão, é melhor declarar upfront todas as variáveis que você pretende usar.
O trecho de código anterior se comportará como se fosse assim:
```js
myname = "global"; // variável global
function func() {
    var myname; // mesmo -> var myname = indefinido;
    alert(myname); // "indefnido"
    myname = "local";
    alert(myname); // "local"
}
func();
```
Em loops você itera sobre matrizes ou objetos como arrays, como argumentos e objetos htmlcollection. O usual para o padrão de loop se parece com o seguinte:
```js
for (var i = 0; i < myarray.length; i++) {
// fazer algo com myArray[i]
}
```
Um problema com esse padrão é que o comprimento da matriz é acessado em cada iteração de loop. Isso pode retardar seu código, especialmente quando myArray não é uma matriz, mas um objeto htmlcollection.
HTMLCollections são objetos retornados por métodos DOM, como:

* document.getElementsByName()<br>
* document.getElementsByClassName()<br>
* document.getElementsByTagName()<br>

Há também uma série de outros HTMLCollections, que foram introduzidos antes do padrão dom e ainda estão em uso hoje. Não incluem (entre outros):
```js
document.images
    //Todos os elementos img na página
document.links
    //Todos os links
document.forms
    //Todos os forms
document.forms[0].elements
    //Todos os campos no primeiro formulário na página
```

O problema com coleções é que eles são consultas ao vivo contra o documento subjacente (a página HTML),e isso significa que toda vez que você acessa o comprimento de qualquer coleção, você está consultando o DOM Live, e as operações DOM são caras em geral.
É por isso que um padrão melhor para loops é armazenar em cache o comprimento da matriz (ou coleção) que você está Iterando, conforme mostrado no exemplo a seguir:

```js
for (var i = 0, max = myarray.length; i < max; i++) {
// Faça algo com myarray[i]
}
```

Desta forma, você recuperar o valor de length apenas uma vez e usá-lo durante o loop inteiro.
Cache o comprimento quando a iteração sobre HTMLCollections é mais rápido em todos os navegadores — em qualquer lugar entre duas vezes mais rápido (Safari 3) e 190 vezes (IE7). (para mais detalhes, consulte JavaScript de alto desempenho por Nicholas Zakas [o'Reilly].)
Observe que quando você explicitamente pretende modificar a coleção no loop (por exemplo, adicionando mais elementos DOM), você provavelmente gostaria que o comprimento a ser atualizado e não constante.
Seguindo o único padrão de var, você também pode tirar o var do loop e fazer o loop como:

```js
function looper() {
    var i = 0,
    max,
    myarray = [];
    // ...
    for (i = 0, max = myarray.length; i < max; i++) {
    // Faça algo com myarray[i]
    }
}
```

Este padrão tem o benefício da consistência porque você irá trabalhar no padrão único de var.
Uma desvantagem é que ele torna um pouco mais difícil de copiar e colar loops. Por exemplo, se você estiver copiando o loop de uma função para outra, você tem que ter certeza que você também irá levar mais de i e Max para a nova função (e provavelmente excluí-los da função original, caso eles não sejam mais necessários lá).
Um último ajuste para o loop seria substituir i + + com qualquer uma dessas expressões:
```js
i = i + 1
i += 1
```

JSLint solicita que você faça isso; a razão é que + + e-promover "excesso de artimanhas."
Se você discordar com isso? você pode! O conjunto do JSLint tem como opção o plusplus para false. (é verdade, isso é padrão.) Mais tarde no livro, o último padrão que será usado: i + = 1.
Duas variações do padrão para introduzir algumas micro-otimizações porque eles: 
• Use uma variável a menos (sem Max) <br>
• contagem regressiva para 0, que é geralmente mais rápido porque é mais eficiente para comparar a 0 do que o comprimento da matriz ou qualquer outra coisa que não seja 0 primeiro.
O padrão desafiado é:<br>

```js
var i, myarray = [];
    for (i = myarray.length; i--;) {
    // Faça algo com myarray[i]
}
```
E o segundo usa um loop While:
```js
var myarray = [],
i = myarray.length;
while (i--) {
// Faça algo com  myarray[i]
}
```

Estas são condições micro-optimizations e só serão as operações que a performance-critical percebeu. Além disso, Jslint vai se queixar sobre o uso de i--.

### for-in Loops

For-in em loops devem ser usados para iterar sobre objetos nonarray. Looping com for-in também é chamado de enumeração.
Tecnicamente, você também pode usar for-in para loop em arrays (porque em JavaScript arrays são objetos), mas não é recomendável. Ele pode levar a erros lógicos se o objeto array já tiver sido aumentado com a funcionalidade personalizada. Além disso, a ordem (a seqüência) de listagem de propriedades não é garantida em um for-in. Portanto, é preferível usar normal para loops com matrizes e for-in para objetos.
É importante usar o método hasOwnProperty() ao iterar sobre propriedades do objeto para filtrar as propriedades que descem a cadeia de protótipos.

Considere o exemplo:

```js
// Objeto
var man = {
    hands: 2,
    legs: 2,
    heads: 1
};
// em algum outro lugar do código
// um método foi adicionado a todos os objetos
if (typeof Object.prototype.clone === "undefined") {
    Object.prototype.clone = function(){};
}
```
Neste exemplo, temos um objeto simples chamado homem definido com um literal de objeto. Em algum lugar antes ou depois que o homem foi definido, o protótipo de objeto foi aumentado com um método útil chamado clone(). A cadeia de protótipos é ao vivo, o que significa que todos os objetos têm acesso automaticamente ao novo método. Para evitar que o método clone() apareça ao enumerar o homem, você precisará chamar hasOwnProperty() para filtrar as propriedades do protótipo. Falha ao fazer a filtragem pode resultar na função Clone() aparecendo, que é comportamento indesejado na maior parte de todos os cenários:

```js
for (var i in man) {
if (man.hasOwnProperty(i)) { // filter
console.log(i, ":", man[i]);
}
}
/*
resultado nas mãos do console: 2 
pernas: 2 
cabeças: 1
*/
// 2.
// antipattern:
// for-in loop without checking hasOwnProperty()
for (var i in man) {
console.log(i, ":", man[i]);
}
/* resultado nas mãos do console: 2 
pernas: 2 
cabeças: 1 
clone: função () 
*/
```

Outro padrão para usar hasOwnProperty() chame esse método fora do Object.Prototype, desse jeito:

```js
for (var i in man) {
    if (Object.prototype.hasOwnProperty.call(man, i)) { // filter
        console.log(i, ":", man[i]);
    }
}
```