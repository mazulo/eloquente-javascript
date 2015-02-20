# A vida secreta dos objetos

> O problema com linguagens orientada a objetos é que elas têm todo esse ambiente implícito que elas carregam. Você queria uma banana mas o que você conseguiu foi um gorila segurando a banana e a floresta inteira.
>
> - Joe Armstrong, _entrevistado na Coders at Work_.

Quando um programador diz "objeto", isso é um termo carregado. Em minha profissão, objetos são uma maneira de vida, o assunto de guerras santas, e um jargão amado que ainda não perdeu muito seu poder.

Para um forasteiro, isso é provavelmente uma pequena confusão. Vamos começar com uma breve história de objetos como um construtor de programação.

## HISTÓRIA ##

Essa história, como a maioria das histórias de programação, começa com o problema de complexidade. Uma filosofia é que complexidade pode ser feita gerenciável separando-a em pequenos compartimentos que são isolados dos um dos outros. Esses compartimentos acabaram com o nome _objetos_.

Um objeto é um escudo duro que esconde a complexidade inoportuna no seu interior e em invés disso nos oferece alguns botões e conectores (tais como métodos) que apresentam uma _interface_ por meio do qual o objeto é utilizado. A ideia é que a interface é relativamente simples e todas as coisas complexas acontecendo dentro do objeto possam ser ignoradas quando se trabalha com ele.

![Uma interface simples pode esconder muita complexidade.](../img/object.jpg)

Como um exemplo, você pode imaginar um objeto que fornece uma interface para uma área na sua tela. Isso fornece uma maneira para desenhar formas ou texto sobre essa área, mas esconde todos os detalhes de como essas formas são convertidas para os pixels que compõem a tela. Você teria um conjunto de métodos, `drawCircle` por exemplo, e essas são as únicas coisas que você precisa saber para usar tal objeto.

Essas ideias foram inicialmente trabalhadas no ano de 1970, 1980 e nos anos de 1990, e foram realizadas por uma enorme onda de exageros — A Revolução da programação orientada a objetos. De repente, houve uma larga tribo de pessoas declarando que objetos estavam no caminho _certo_ para programar — e que qualquer coisa que não envolvesse objetos era uma tolice desatualizada.

Esse tipo de fanatismo sempre produz um monte de bobagem impraticável, e tem havido um tipo de contra-revolução desde então. Em alguns círculos, objetos tem uma reputação muito má nos dias de hoje.

Eu prefiro olhar para a questão a partir de um ângulo prática, ao invés de ideologico. Existem muitos conceitos úteis, mais importante que _encapsulamento_ (distiguindo entre complexidade interna e interface externa), que a cultura orientada a objetos têm popularizado. Estes são os que valem a pena estudar.

Esse capítulo descreve a tomada excêntrica do JavaScript em objetos, e a maneira como eles se relacionam em algumas técnicas clássicas orientadas a objeto.


## MÉTODOS ##

Métodos são simplesmente propriedades que possuem valores de função. Isso é o método simples:

````javascript
var rabbit = {};
rabbit.speak = function(line) {
	console.log("The rabbit says '" + line + "'");
};

rabbit.speak("I'm alive.");
// → The rabbit says 'I'm alive.'
````
Normalmente, um método precisa fazer alguma coisa com o objeto em que foi chamado. Quando uma função é chamada como um método — procurada como uma propriedade e imediatamente chamada, como em `object.method()` — a variável especial this no seu corpo apontará para o objeto em que foi chamado.

````javascript
function speak(line) {
  console.log("The " + this.type + " rabbit says '" + line + "'");
}
var whiteRabbit = {type: "white", speak: speak};
var fatRabbit = {type: "fat", speak: speak};

whiteRabbit.speak("Oh my ears and whiskers, " + "how late it's getting!");
// → The white rabbit says 'Oh my ears and whiskers, how
//   late it's getting!'
fatRabbit.speak("I could sure use a carrot right now.");
// → The fat rabbit says 'I could sure use a carrot
//   right now.'
````
O código usa a palavra chave `this` para a saída do tipo de coelhos que está falando.

Lembre que ambos os métodos `apply` e `bind` recebem um primeiro argumento que pode ser usado para simular chamadas de métodos. Esse primeiro argumento é de fato usado para dar um valor ao `this`.

Existe um método similar ao `apply`, chamado `call`. Ele também chama a função que é um método mas pega seus argumentos normalmente, em vez de como um array. Como `apply` e `bind`, `call` pode ser passado um valor `this` específico.

````javascript
speak.apply(fatRabbit, ["Burp!"]);
// → The fat rabbit says 'Burp!'
speak.call({type: "old"}, "Oh my.");
// → The old rabbit says 'Oh my.'
````

## PROTÓTIPOS ##

Assista de perto.

````javascript
var empty = {};
console.log(empty.toString);
// → function toString(){...}
console.log(empty.toString());
// → [object Object]
````

Eu apenas removi para fora uma propriedade de um objeto vazio. Mágica!

Bem, na verdade não. Eu tenho simplesmente ocultado informações sobre a maneira que objetos JavaScript trabalham. Além do seu conjunto de propriedades, quase todos os objetos também tem um _prototype_. Um prototype é outro objeto que é usado como uma fonte de retorno de propriedades. Quando um objeto recebe uma requisição para uma propriedade que ele não tem, seu prototype será pesquisado pela propriedade, então o prototype do prototype, e assim por diante.

Então quem é o prototype daquele objeto vazio? É o grande prototype ancestral, a entidade por trás de quase todos os objetos, `Object.prototype`.

````javascript
console.log(Object.getPrototypeOf({}) == Object.prototype);
// → true
console.log(Object.getPrototypeOf(Object.prototype));
// → null
````

Como você pode esperar, a função `Object.getPrototypeOf` retorna o prototype de um objeto.

As relações de prototype de objetos JavaScript formam uma estrutura em forma de árvore, e na raíz dessa estrutura está o `Object.prototype`. Ele fornece alguns métodos que aparecem em todos os objetos, tais como `toString`, que converte um objeto para uma representação de string.

Muitos objetos não tem diretamente `Object.prototype` como seu prototype, mas ao invés tem outro objeto, que fornece suas próprias propriedades padrão. Funções derivam de `Function.prototype`, e arrays derivam de `Array.prototype`.

````javascript
console.log(Object.getPrototypeOf(isNaN) == Function.prototype);
// → true
console.log(Object.getPrototypeOf([]) == Array.prototype);
// → true
````

Tal objeto prototype terá em si um prototype, geralmente `Object.prototype`, de modo que ele ainda indiretamente fornece métodos como `toString`.

A função `Object.getPrototypeOf` obviamente retorna o prototype de um objeto. Você pode usar `Object.create` para criar um objeto com um prototype específico.

````javascript
var protoRabbit = {
  speak: function(line) {
    console.log("The " + this.type + " rabbit says '" + line + "'");
  }
};
var killerRabbit = Object.create(protoRabbit);
killerRabbit.type = "killer";
killerRabbit.speak("SKREEEE!");
// → The killer rabbit says 'SKREEEE!'
````
O "proto" rabbit atua como um container para as propriedades que são compartilhadas por todos os coelhos. Um objeto rabbit individual, como o killer rabbit, contém propriedades que aplicam apenas em sim - neste caso seu tipo - e deriva as propriedades compartilhadas de seu prototype.

## CONSTRUTOR ##

Uma maneira mais conveniente de criar objetos que derivam de algum prototype compartilhado é usar um _construtor_. Em JavaScript, chamar uma função com a palavra chave `new` em frente faz com que ele seja tratado como um construtor. O construtor terá sua variável `this` ligada ao novo objeto, e a menos que explicitamente retorne um outro valor de objeto, esse novo objeto será retornado a partir da chamada.

Um objeto criado com `new` é dito ser uma _instância_ de seu construtor.

Aqui é um simples construtor para coelhos. É uma convenção capitalizar os nomes dos construtores para que eles seja facilmente distinguidos de outras funções.

````javascript
function Rabbit(type) {
	this.type = type;
}
var killerRabbit = new Rabbit("killer");
var blackRabbit = new Rabbit("black");
console.log(blackRabbit.type);
// → black
````

Construtores (na verdade, todas as funções) automaticamente recebem uma propriedade chamada `prototype`, a qual por padrão possui um simples e vazio objeto que deriva de `Object.prototype`. Cada instância criada com este construtor terá esse objeto como seu prototype. Então para adicionar um método `speak` para os rabbits criado com o construtor `Rabbit`, nós podemos simplesmente fazer isso:

````javascript
Rabbit.prototype.speak = function(line) {
  console.log("The " + this.type + " rabbit says '" +
              line + "'");
};
blackRabbit.speak("Doom...");
// → The black rabbit says 'Doom...'
````

É importante notar a distinção entre a maneira que um prototype é associado com um construtor (através de sua propriedade `prototype`) e a maneira que objetos _tem_ um prototype (que podem ser recuperados com `Object.getPrototypeOf`). O prototype real de um construtor é o `Function.prototype` desde que construtores são funções. Sua _propriedade_ `prototype` será o prototype de instâncias criadas por ele, mas não é o seu _próprio_ prototype.

## Sobrescrevendo propriedades derivadas ##

Quando você adiciona uma propriedade a um objeto, se ele está presente no prototype ou não, a propriedade é adicionada ao _próprio_ objeto, que a partir de agora a tem como sua própria propriedade. Se _existe_ uma propriedade com o mesmo nome no prototype, essa propriedade não irá afetar o objeto. O próprio prototype não é alterado.

````javascript
Rabbit.prototype.teeth = "small";
console.log(killerRabbit.teeth);
// → small
killerRabbit.teeth = "long, sharp, and bloody";
console.log(killerRabbit.teeth);
// → long, sharp, and bloody
console.log(blackRabbit.teeth);
// → small
console.log(Rabbit.prototype.teeth);
// → small
````

O diagrama a seguir esboça a situação depois desse código rodar. O prototype `Rabbit` e `Object` estão por trás de `killerRabbit` como um tipo de pano de fundo, onde as propriedades que não são encontradas no próprio objeto possam ser encontradas neles.

![Rabbit object prototype schema](../img/rabbits.svg)

Sobrescrever propriedades que existem em um prototype é geralmente uma coisa útil para fazer. Como no exemplo do dente de coelho (rabbit teeth) mostra, pode ser usado para expressar propriedades excepcionais de instância de uma classe mais genérica de objetos, enquanto permite que os objetos não-excepcionais simplesmente assumir um valor padrão do seu prototypes.

Também é usado para dar a função padrão e prototypes de array um método toString diferente do que o do prototype do objeto básico.

````javascript
console.log(Array.prototype.toString ==
            Object.prototype.toString);
// → false
console.log([1, 2].toString());
// → 1,2
````

Chamar `toString` em uma array dá um resultado similar ao chamar `.join(",")` nele - coloca vírgulas entre os valores no array. Chamar diretamente `Object.prototype.toString` com um array produz uma string diferente. Essa função não sabe nada sobre arrays, então ela simplesmente coloca a palavra "object" e o nome do tipo entre colchetes.

````javascript
console.log(Object.prototype.toString.call([1, 2]));
// → [object Array]
````

## Prototype interference ##

A prototype can be used at any time to add new properties and methods to all objects based on it. For example, it might become necessary for our rabbits to dance.

````javascript
Rabbit.prototype.dance = function() {
  console.log("The " + this.type + " rabbit dances a jig.");
};
killerRabbit.dance();
// → The killer rabbit dances a jig.
````

That’s convenient. But there are situations where it causes problems. In previous chapters, we used an object as a way to associate values with names by creating properties for the names and giving them the corresponding value as their value. Here’s an example from [Chapter 4]():

````javascript
var map = {};
function storePhi(event, phi) {
  map[event] = phi;
}
storePhi("pizza", 0.069);
storePhi("touched tree", -0.081);
````

We can iterate over all phi values in the object using a `for/in` loop and test whether a name is in there using the regular in operator. But unfortunately, the object’s prototype gets in the way.

````javascript
Object.prototype.nonsense = "hi";
for (var name in map)
  console.log(name);
// → pizza
// → touched tree
// → nonsense
console.log("nonsense" in map);
// → true
console.log("toString" in map);
// → true

// Delete the problematic property again
delete Object.prototype.nonsense;
````

That’s all wrong. There is no event called “nonsense” in our data set. And there _definitely_ is no event called “toString”.

Oddly, `toString` did not show up in the `for/in` loop, but the `in` operator did return true for it. This is because JavaScript distinguishes between _enumerable_ and _nonenumerable_ properties.

All properties that we create by simply assigning to them are enumerable. The standard properties in `Object.prototype` are all nonenumerable, which is why they do not show up in such a `for/in` loop.

It is possible to define our own nonenumerable properties by using the `Object.defineProperty` function, which allows us to control the type of property we are creating.

````javascript
Object.defineProperty(Object.prototype, "hiddenNonsense",
                      {enumerable: false, value: "hi"});
for (var name in map)
  console.log(name);
// → pizza
// → touched tree
console.log(map.hiddenNonsense);
// → hi
````

So now the property is there, but it won’t show up in a loop. That’s good. But we still have the problem with the regular `in` operator claiming that the Object.prototype properties exist in our object. For that, we can use the object’s `hasOwnProperty` method.

````javascript
console.log(map.hasOwnProperty("toString"));
// → false
````

This method tells us whether the object _itself_ has the property, without looking at its prototypes. This is often a more useful piece of information than what the `in` operator gives us.

When you are worried that someone (some other code you loaded into your program) might have messed with the base object prototype, I recommend you write your `for/in` loops like this:

````javascript
for (var name in map) {
  if (map.hasOwnProperty(name)) {
    // ... this is an own property
  }
}
````

### Prototype-less objects ##

But the rabbit hole doesn’t end there. What if someone registered the name `hasOwnProperty` in our `map` object and set it to the value 42? Now the call to `map.hasOwnProperty` will try to call the local property, which holds a number, not a function.

In such a case, prototypes just get in the way, and we would actually prefer to have objects without prototypes. We saw the `Object.create` function, which allows us to create an object with a specific prototype. You are allowed to pass `null` as the prototype to create a fresh object with no prototype. For objects like `map`, where the properties could be anything, this is exactly what we want.

````javascript
var map = Object.create(null);
map["pizza"] = 0.069;
console.log("toString" in map);
// → false
console.log("pizza" in map);
// → true
````

Much better! We no longer need the `hasOwnProperty` kludge because all the properties the object has are its own properties. Now we can safely use `for/in` loops, no matter what people have been doing to `Object.prototype`.

## Polymorphism ##

When you call the `String` function, which converts a value to a string, on an object, it will call the `toString` method on that object to try to create a meaningful string to return. I mentioned that some of the standard prototypes define their own version of `toString` in order to be able to create a string that contains more useful information than `"[object Object]"`.

This is a simple instance of a powerful idea. When a piece of code is written to work with objects that have a certain interface—in this case, a `toString` method—any kind of object that happens to support this interface can be plugged into the code, and it will just work.

This technique is called _polymorphism_—though no actual shape-shifting is involved. Polymorphic code can work with values of different shapes, as long as they support the interface it expects.

## Laying out a table ##

I am going to work through a slightly more involved example in an attempt to give you a better idea what polymorphism, as well as object-oriented programming in general, looks like. The project is this: we will write a program that, given an array of arrays of table cells, builds up a string that contains a nicely laid out table—meaning that the columns are straight and the rows are aligned. Something like this:
````
name         height country
------------ ------ -------------
Kilimanjaro    5895 Tanzania
Everest        8848 Nepal
Mount Fuji     3776 Japan
Mont Blanc     4808 Italy/France
Vaalserberg     323 Netherlands
Denali         6168 United States
Popocatepetl   5465 Mexico
````

The way our table-building system will work is that the builder function will ask each cell how wide and high it wants to be and then use this information to determine the width of the columns and the height of the rows. The builder function will then ask the cells to draw themselves at the correct size and assemble the results into a single string.

The layout program will communicate with the cell objects through a well-defined interface. That way, the types of cells that the program supports is not fixed in advance. We can add new cell styles later—for example, underlined cells for table headers—and if they support our interface, they will just work, without requiring changes to the layout program.

This is the interface:
  * `minHeigth()` returns a number indicating the minimum height this cell requires (in lines).
  * `minWidth()` returns a number indicating this cell’s minimum width (in characters).
  * `draw(width, height)` returns an array of length height, which contains a series of strings that are each width characters wide. This represents the content of the cell.

I’m going to make heavy use of higher-order array methods in this example since it lends itself well to that approach.

The first part of the program computes arrays of minimum column widths and row heights for a grid of cells. The `rows` variable will hold an array of arrays, with each inner array representing a row of cells.

````javascript
function rowHeights(rows) {
  return rows.map(function(row) {
    return row.reduce(function(max, cell) {
      return Math.max(max, cell.minHeight());
    }, 0);
  });
}

function colWidths(rows) {
  return rows[0].map(function(_, i) {
    return rows.reduce(function(max, row) {
      return Math.max(max, row[i].minWidth());
    }, 0);
  });
}
````

Using a variable name starting with an underscore (_) or consisting entirely of a single underscore is a way to indicate (to human readers) that this argument is not going to be used.

The `rowHeights` function shouldn’t be too hard to follow. It uses `reduce` to compute the maximum height of an array of cells and wraps that in `map` in order to do it for all rows in the `rows` array.

Things are slightly harder for the `colWidths` function because the outer array is an array of rows, not of columns. I have failed to mention so far that `map` (as well as `forEach`, `filter`, and similar array methods) passes a second argument to the function it is given: the index of the current element. By mapping over the elements of the first row and only using the mapping function’s second argument, `colWidths` builds up an array with one element for every column index. The call to `reduce` runs over the outer `rows` array for each index and picks out the width of the widest cell at that index.

Here’s the code to draw a table:

````javascript
function drawTable(rows) {
  var heights = rowHeights(rows);
  var widths = colWidths(rows);

  function drawLine(blocks, lineNo) {
    return blocks.map(function(block) {
      return block[lineNo];
    }).join(" ");
  }

  function drawRow(row, rowNum) {
    var blocks = row.map(function(cell, colNum) {
      return cell.draw(widths[colNum], heights[rowNum]);
    });
    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
  }

  return rows.map(drawRow).join("\n");
}
````

The drawTable function uses the internal helper function `drawRow` to draw all rows and then joins them together with newline characters.

The `drawRow` function itself first converts the cell objects in the row to *blocks*, which are arrays of strings representing the content of the cells, split by line. A single cell containing simply the number 3776 might be represented by a single-element array like `["3776"]`, whereas an underlined cell might take up two lines and be represented by the array `["name", "----"]`.

The blocks for a row, which all have the same height, should appear next to each other in the final output. The second call to `map` in `drawRow` builds up this output line by line by mapping over the lines in the leftmost block and, for each of those, collecting a line that spans the full width of the table. These lines are then joined with newline characters to provide the whole row as `drawRow`’s return value.

The function `drawLine` extracts lines that should appear next to each other from an array of blocks and joins them with a space character to create a one-character gap between the table’s columns.

Now let’s write a constructor for cells that contain text, which implements the interface for table cells. The constructor splits a string into an array of lines using the string method `split`, which cuts up a string at every occurrence of its argument and returns an array of the pieces. The `minWidth` method finds the maximum line width in this array.

````javascript
function repeat(string, times) {
  var result = "";
  for (var i = 0; i < times; i++)
    result += string;
  return result;
}

function TextCell(text) {
  this.text = text.split("\n");
}
TextCell.prototype.minWidth = function() {
  return this.text.reduce(function(width, line) {
    return Math.max(width, line.length);
  }, 0);
};
TextCell.prototype.minHeight = function() {
  return this.text.length;
};
TextCell.prototype.draw = function(width, height) {
  var result = [];
  for (var i = 0; i < height; i++) {
    var line = this.text[i] || "";
    result.push(line + repeat(" ", width - line.length));
  }
  return result;
};
````

The code uses a helper function called `repeat`, which builds a string whose value is the `string` argument repeated `times` number of times. The `draw` method uses it to add “padding” to lines so that they all have the required length.

Let’s try everything we’ve written so far by building up a 5 × 5 checkerboard.

````javascript
var rows = [];
for (var i = 0; i < 5; i++) {
   var row = [];
   for (var j = 0; j < 5; j++) {
     if ((j + i) % 2 == 0)
       row.push(new TextCell("##"));
     else
       row.push(new TextCell("  "));
   }
   rows.push(row);
}
console.log(drawTable(rows));
// → ##    ##    ##
//      ##    ##
//   ##    ##    ##
//      ##    ##
//   ##    ##    ##
````

It works! But since all cells have the same size, the table-layout code doesn’t really do anything interesting.

The source data for the table of mountains that we are trying to build is available in the `MOUNTAINS` variable in the sandbox and also [downloadable](http://eloquentjavascript.net/code/mountains.js) from the website.

We will want to highlight the top row, which contains the column names, by underlining the cells with a series of dash characters. No problem—we simply write a cell type that handles underlining.

````javascript
function UnderlinedCell(inner) {
  this.inner = inner;
};
UnderlinedCell.prototype.minWidth = function() {
  return this.inner.minWidth();
};
UnderlinedCell.prototype.minHeight = function() {
  return this.inner.minHeight() + 1;
};
UnderlinedCell.prototype.draw = function(width, height) {
  return this.inner.draw(width, height - 1)
    .concat([repeat("-", width)]);
};
````

An underlined cell *contains* another cell. It reports its minimum size as being the same as that of its inner cell (by calling through to that cell’s `minWidth` and `minHeight` methods) but adds one to the height to account for the space taken up by the underline.

Drawing such a cell is quite simple—we take the content of the inner cell and concatenate a single line full of dashes to it.

Having an underlining mechanism, we can now write a function that builds up a grid of cells from our data set.

````javascript
function dataTable(data) {
  var keys = Object.keys(data[0]);
  var headers = keys.map(function(name) {
    return new UnderlinedCell(new TextCell(name));
  });
  var body = data.map(function(row) {
    return keys.map(function(name) {
      return new TextCell(String(row[name]));
    });
  });
  return [headers].concat(body);
}

console.log(drawTable(dataTable(MOUNTAINS)));
// → name         height country
//   ------------ ------ -------------
//   Kilimanjaro  5895   Tanzania
//   … etcetera
````

The standard `Object.keys` function returns an array of property names in an object. The top row of the table must contain underlined cells that give the names of the columns. Below that, the values of all the objects in the data set appear as normal cells—we extract them by mapping over the `keys` array so that we are sure that the order of the cells is the same in every row.

The resulting table resembles the example shown before, except that it does not right-align the numbers in the `height` column. We will get to that in a moment.

## Getters and setters ##

When specifying an interface, it is possible to include properties that are not methods. We could have defined `minHeight` and `minWidth` to simply hold numbers. But that’d have required us to compute them in the constructor, which adds code there that isn’t strictly relevant to *constructing* the object. It would cause problems if, for example, the inner cell of an underlined cell was changed, at which point the size of the underlined cell should also change.

This has led some people to adopt a principle of never including nonmethod properties in interfaces. Rather than directly access a simple value property, they’d use `getSomething` and `setSomething` methods to read and write the property. This approach has the downside that you will end up writing—and reading—a lot of additional methods.

Fortunately, JavaScript provides a technique that gets us the best of both worlds. We can specify properties that, from the outside, look like normal properties but secretly have methods associated with them.

````javascript
var pile = {
  elements: ["eggshell", "orange peel", "worm"],
  get height() {
    return this.elements.length;
  },
  set height(value) {
    console.log("Ignoring attempt to set height to", value);
  }
};

console.log(pile.height);
// → 3
pile.height = 100;
// → Ignoring attempt to set height to 100
````

In object literal, the `get` or `set` notation for properties allows you to specify a function to be run when the property is read or written. You can also add such a property to an existing object, for example a prototype, using the `Object.defineProperty` function (which we previously used to create nonenumerable properties).

````javascript
Object.defineProperty(TextCell.prototype, "heightProp", {
  get: function() { return this.text.length; }
});

var cell = new TextCell("no\nway");
console.log(cell.heightProp);
// → 2
cell.heightProp = 100;
console.log(cell.heightProp);
// → 2
````

You can use a similar `set` property, in the object passed to `defineProperty`, to specify a setter method. When a getter but no setter is defined, writing to the property is simply ignored.

## INHERITANCE ##

We are not quite done yet with our table layout exercise. It helps readability to right-align columns of numbers. We should create another cell type that is like `TextCell` but rather than padding the lines on the right side it pads them on the left side so that they align to the right.

We could simply write a whole new constructor with all three methods in its prototype. But prototypes may themselves have prototypes, and this allows us to do something clever.

````javascript
function RTextCell(text) {
  TextCell.call(this, text);
}
RTextCell.prototype = Object.create(TextCell.prototype);
RTextCell.prototype.draw = function(width, height) {
  var result = [];
  for (var i = 0; i < height; i++) {
    var line = this.text[i] || "";
    result.push(repeat(" ", width - line.length) + line);
  }
  return result;
};
````

We reuse the constructor and the `minHeight` and `minWidth` methods from the regular `TextCell`. An `RTextCell` is now basically equivalent to a `TextCell`, except that its `draw` method contains a different function.

This pattern is called *inheritance*. It allows us to build slightly different data types from existing data types with relatively little work. Typically, the new constructor will call the old constructor (using the `call` method in order to be able to give it the new object as its `this` value). Once this constructor has been called, we can assume that all the fields that the old object type is supposed to contain have been added. We arrange for the constructor’s prototype to derive from the old prototype so that instances of this type will also have access to the properties in that prototype. Finally, we can override some of these properties by adding them to our new prototype.

Now if we slightly adjust the dataTable function to use RTextCells for cells whose value is a number, we get the table we were aiming for.

````javascript
function dataTable(data) {
  var keys = Object.keys(data[0]);
  var headers = keys.map(function(name) {
    return new UnderlinedCell(new TextCell(name));
  });
  var body = data.map(function(row) {
    return keys.map(function(name) {
      var value = row[name];
      // This was changed:
      if (typeof value == "number")
        return new RTextCell(String(value));
      else
        return new TextCell(String(value));
    });
  });
  return [headers].concat(body);
}

console.log(drawTable(dataTable(MOUNTAINS)));
// → … beautifully aligned table
````

Inheritance is a fundamental part of the object-oriented tradition, alongside encapsulation and polymorphism. But while the latter two are now generally regarded as wonderful ideas, inheritance is somewhat controversial.

The main reason for this is that it is often confused with polymorphism, sold as a more powerful tool than it really is, and subsequently overused in all kinds of ugly ways. Whereas encapsulation and polymorphism can be used to *separate* pieces of code from each other, reducing the tangledness of the overall program, inheritance fundamentally ties types together, creating *more* tangle.

You can have polymorphism without inheritance, as we saw. I am not going to tell you to avoid inheritance entirely—I use it regularly in my own programs. But you should see it as a slightly dodgy trick that can help you define new types with little code, not as a grand principle of code organization. A preferable way to extend types is through composition, such as how `UnderlinedCell` builds on another cell object by simply storing it in a property and forwarding method calls to it in its own methods.

## The instanceof operator ##

It is occasionally useful to know whether an object was derived from a specific constructor. For this, JavaScript provides a binary operator called `instanceof`.

````javascript
console.log(new RTextCell("A") instanceof RTextCell);
// → true
console.log(new RTextCell("A") instanceof TextCell);
// → true
console.log(new TextCell("A") instanceof RTextCell);
// → false
console.log([1] instanceof Array);
// → true
````

The operator will see through inherited types. An `RTextCell` is an instance of `TextCell` because `RTextCell.prototype` derives from `TextCell.prototype`. The operator can be applied to standard constructors like `Array`. Almost every object is an instance of `Object`.

## SUMMARY ##

So objects are more complicated than I initially portrayed them. They have prototypes, which are other objects, and will act as if they have properties they don’t have as long as the prototype has that property. Simple objects have `Object.prototype` as their prototype.

Constructors, which are functions whose names usually start with a capital letter, can be used with the `new` operator to create new objects. The new object’s prototype will be the object found in the `prototype` property of the constructor function. You can make good use of this by putting the properties that all values of a given type share into their prototype. The `instanceof` operator can, given an object and a constructor, tell you whether that object is an instance of that constructor.

One useful thing to do with objects is to specify an interface for them and tell everybody that they are supposed to talk to your object only through that interface. The rest of the details that make up your object are now *encapsulated*, hidden behind the interface.

Once you are talking in terms of interfaces, who says that only one kind of object may implement this interface? Having different objects expose the same interface and then writing code that works on any object with the interface is called *polymorphism*. It is very useful.

When implementing multiple types that differ in only some details, it can be helpful to simply make the prototype of your new type derive from the prototype of your old type and have your new constructor call the old one. This gives you an object type similar to the old type but for which you can add and override properties as you see fit.

## EXERCISES ##

### A VECTOR TYPE ##

Write a constructor `Vector` that represents a vector in two-dimensional space. It takes `x` and `y` parameters (numbers), which it should save to properties of the same name.

Give the `Vector` prototype two methods, `plus` and `minus`, that take another vector as a parameter and return a new vector that has the sum or difference of the two vectors’ (the one in `this` and the parameter) x and y values.

Add a getter property `length` to the prototype that computes the length of the vector—that is, the distance of the point (x, y) from the origin (0, 0).

````javascript
// Your code here.

console.log(new Vector(1, 2).plus(new Vector(2, 3)));
// → Vector{x: 3, y: 5}
console.log(new Vector(1, 2).minus(new Vector(2, 3)));
// → Vector{x: -1, y: -1}
console.log(new Vector(3, 4).length);
// → 5
````

> Display hints

> Your solution can follow the pattern of the `Rabbit` constructor from this chapter quite closely.

> Adding a getter property to the constructor can be done with the `Object.defineProperty` function. To compute the distance from (0, 0) to (x, y), you can use the Pythagorean theorem, which says that the square of the distance we are looking for is equal to the square of the x coordinate plus the square of the y coordinate. Thus, √(x² + y²) is the number you want, and `Math.sqrt` is the way you compute a square root in JavaScript.

### ANOTHER CELL ###

Implement a cell type named `StretchCell(inner, width, height)` that conforms [to the table cell interface](http://eloquentjavascript.net/06_object.html#table_interface) described earlier in the chapter. It should wrap another cell (like `UnderlinedCell` does) and ensure that the resulting cell has at least the given `width` and `height`, even if the inner cell would naturally be smaller.

````javascript
// Your code here.

var sc = new StretchCell(new TextCell("abc"), 1, 2);
console.log(sc.minWidth());
// → 3
console.log(sc.minHeight());
// → 2
console.log(sc.draw(3, 2));
// → ["abc", "   "]
````

> Display hints

> You’ll have to store all three constructor arguments in the instance object. The minWidth and minHeight methods should call through to the corresponding methods in the inner cell but ensure that no number less than the given size is returned (possibly using Math.max).

> Don’t forget to add a draw method that simply forwards the call to the inner cell.

### SEQUENCE INTERFACE ###

Design an *interface* that abstracts iteration over a collection of values. An object that provides this interface represents a sequence, and the interface must somehow make it possible for code that uses such an object to iterate over the sequence, looking at the element values it is made up of and having some way to find out when the end of the sequence is reached.

When you have specified your interface, try to write a function `logFive` that takes a sequence object and calls `console.log` on its first five elements—or fewer, if the sequence has fewer than five elements.

Then implement an object type `ArraySeq` that wraps an array and allows iteration over the array using the interface you designed. Implement another object type `RangeSeq` that iterates over a range of integers (taking `from` and `to` arguments to its constructor) instead.

````javascript
// Your code here.

logFive(new ArraySeq([1, 2]));
// → 1
// → 2
logFive(new RangeSeq(100, 1000));
// → 100
// → 101
// → 102
// → 103
// → 104
````

> Display hints

> One way to solve this is to give the sequence objects *state*, meaning their properties are changed in the process of using them. You could store a counter that indicates how far the sequence object has advanced.

> Your interface will need to expose at least a way to get the next element and to find out whether the iteration has reached the end of the sequence yet. It is tempting to roll these into one method, `next`, which returns `null` or `undefined` when the sequence is at its end. But now you have a problem when a sequence actually contains `null`. So a separate method (or getter property) to find out whether the end has been reached is probably preferable.

> Another solution is to avoid changing state in the object. You can expose a method for getting the current element (without advancing any counter) and another for getting a new sequence that represents the remaining elements after the current one (or a special value if the end of the sequence is reached). This is quite elegant—a sequence value will “stay itself” even after it is used and can thus be shared with other code without worrying about what might happen to it. It is, unfortunately, also somewhat inefficient in a language like JavaScript because it involves creating a lot of objects during iteration.