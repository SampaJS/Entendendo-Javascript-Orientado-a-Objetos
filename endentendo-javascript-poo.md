# Entendendo JavaScript POO

## prototipoco herança em grandiosidade!!

Postado em 9 de Outubro de 2011 

// revisar este trecho, não ficou oeso //
JavaScript é uma linguagem orientada a objetos (POO), with its roots in the Self
programming language, embora é (tristemente) projetado para parecer com Java.
Isto torna a lingagem realmente poderosa e doces carecterísticas ficam cobertas
por alguns mesmo muito feios e soluções alternativas contra-intuitivo.

Um tal recucurso afetado é a implementação de herança prototípica. Os conceitos
são simples mas flexíveis e poderosos. It makes inheritance and behaviourism first-class citizens, just like functions are first-class in functional-ish languages (JavaScript included).

Fortunately, ECMAScript 5 has gotten plenty of things to move the language in the right way, and it's on those sweet features that this article will expand. I'll also cover the drawbacks of JavaScript's design, and do a little comparison with the classical model here and there, where those would highlight the advantages or disadvantages of the language's implementation of prototypical OO.

It's important to note, though, that this article assumes you have knowledge over other basic JavaScript functionality, like functions (including the concepts of closures and first-class functions), primitive values, operators and such.

## Table of Contents

1.  Objetos  
1.1. O que são objetos?  
1.2. Criando propriedades  
1.3. Descritores  
1.4. Abandonar a verbosidade  
1.5. Acessando propriedades  
1.6. Removendo propriedades  
1.7. Getters and setters  
1.8. Listanto propriedades  
1.9. Objetos leterais  
2. Métodos  
2.1. Dinamico this  
2.2. Como o this é resolvido  
2.2.1. Chamado como um método  
2.2.2. Chamado diretamente  
2.2.3. Explicitamente aplicado  
2.3. Métodos vinculados  
3. Herança 
3.1. Protótipos   
3.2. Como [[Protótipos]] funcionam  
3.3. Sobre-escrevendo propriedades  
3.4. Mixins  
3.5. Acessando propriedades sobrescritas  
4. Construtores  
4.1. A nova magia  
4.2. Herança com construtores  
5. Considerações e compatibilidade  
5.1. Criando objetos  
5.2. Definindo propiedades  
5.3. Listando propiedades  
5.4. Métodos vinculados  
5.5. Obtendo o [[protótipo]]
5.6. Bibliotecas que fornecem fallbacks    
6. Juntando tudo  
7. Coisas que vem a pena ler no próximo  
8. Agradecimentos  

### 1. Objects  

Everything you can manipulate in JavaScript is an object. This includes
``Strings``, ``Arrays``, ``Numbers``, ``Functions``, and, obviously, the
so-called ``Object`` — there are primitives, but they're converted to an object when you need to operate upon them. An object in the language is simply a collection of key/value pairs (and some internal magic sometimes).

There are no concepts of classes anywhere, though. That is, an object with
properties ``name: Linda, age: 21`` is not an instance of any class, the
``Object`` class. Both ``Object`` and ``Linda`` are instances of themselves. They define their own behaviour, directly. There are no layers of meta-data (i.e.: classes) to dictate what given object must look like.

You might ask: "how?"; more so if you come from a highly classically Object Orientated language (like Java or C#). "Wouldn't having each object defining their own behaviour, instead of a common class mean that if I have 100 objects, I will have 100 different methods? Also, isn't it dangerous? How would one know if an object is really an Array, for example?"

Well, to answer all those questions, we'll first need to unlearn everything about the classical OO approach and start from the ground up. But, trust me, it's worth it.

The prototypical OO model brings in some new ways of solving old problems, in an
more dynamic and expressive way. It also presents new and more powerful models
for extensibility and code-reuse, which is what most people are interested about
when they talk about Object Orientation. It does not, however, give you
contracts. Thus, there are no static guarantees that an object ``X`` will always have a given set of properties, but to understand the trade-offs here, we'll need to know what we're talking about first.

### 1.1. What are objects?  

As mentioned previously, objects are simple pairs of unique keys that correspond
to a value — we'll call this pair a ``property``. So, suppose you'd want to
describe a few aspects of an old friend (say ``Mikhail``), like age, name and gender:

![GitHub Logo](http://dl.dropbox.com/u/4429200/blog/oop-obj-mikhail.png)

Objects are created in JavaScript using the ``Object.create`` function. It takes a parent and an optional set of property descriptors and makes a brand new instance. We'll not worry much about the parameters now.

An empty object is an object with no parent, and no properties. The syntax to create such object in JavaScript is the following:

```javascript
var mikhail = Object.create(null)
```

### 1.2. Creating properties

So, now we have an object, but no properties — we've got to fix that if we want
to describe ``Mikhail``.

Properties in JavaScript are dynamic. That means that they can be created or removed at any time. Properties are also unique, in the sense that a property key inside an object correspond to exactly one value.

Creating new properties is done through the ``Object.defineProperty`` function, which takes a reference to an object, the name of the property to create and a descriptor that defines the semantics of the property.

```javascript
Object.defineProperty(mikhail, 'name', { value:        'Mikhail'
                                       , writable:     true
                                       , configurable: true
                                       , enumerable:   true })

Object.defineProperty(mikhail, 'age', { value:        19
                                      , writable:     true
                                      , configurable: true
                                      , enumerable:   true })

Object.defineProperty(mikhail, 'gender', { value:        'Male'
                                         , writable:     true
                                         , configurable: true
                                         , enumerable:   true })
```

``Object.defineProperty`` will create a new property if a property with the given key does not exist in the object, otherwise it'll update the semantics and value of the existing property.

**-By the way**
> You can also use the `Object.defineProperties` when you need to add more than one property to an object:

```javascript
Object.defineProperties(mikhail, { name:   { value:        'Mikhail'
                                           , writable:     true
                                           , configurable: true
                                           , enumerable:   true }

                                 , age:    { value:        19
                                           , writable:     true
                                           , configurable: true
                                           , enumerable:   true }

                                 , gender: { value:        'Male'
                                           , writable:     true
                                           , configurable: true
                                           , enumerable:   true }})
```

> Obviously, both calls are overtly verbose — albeit also quite configurable —, thus not really meant for end-user code. It's better to create an abstraction layer on top of them.

### 1.3. Descriptors

The little objects that carry the semantics of a property are called descriptors
(we used them in the previous `Object.defineProperty` calls). Descriptors can be one of two types - data descriptors or accessor descriptors.

Both types of descriptor contain flags, which define how a property is treated
in the language. If a flag is not set, it's assumed to be `false` — unfortunately this is usually not a good default value for them, which adds to the verbosity of these descriptors.

**-writable**
> Whether the concrete value of the property may be changed. Only applies to data descriptors.

**-configurable**
> Whether the type of descriptor may be changed, or if the property can be removed.

**-enumerable**
> Whether the property is listed in a loop through the properties of the object.

Data descriptors are those that hold concrete values, and therefore have an
additional `value` parameter, describing the concrete data bound to the property:

**-value**
> The value of a property.

Accessor descriptors, on the other hand, proxy access to the concrete value
through getter and setter functions. When not set, they'll default to `undefined`.

**-get ()**
> A function called with no arguments when the property value is requested. 

**-set (new_value)**
> A function called with the new value for the property when the user tries to modify the value of the property.

### 1.4. Ditching the verbosity

Luckily, property descriptors are not the only way of working with properties in JavaScript, they can also be handled in a sane and concise way.

JavaScript also understands references to a property using what we call bracket notation. The general rule is:

```javascript
<bracket-access> ::= <identifier> "[" <expression> "]"
```

Where `identifier` is the variable that holds the object containing the
properties we want to access, and expression is any valid JavaScript
`expression` that defines the name of the property. There are no constraints in which name a property can have1, everything is fair game.

Thus, we could just as well rewrite our previous example as:

```javascript
mikhail['name']   = 'Mikhail'
mikhail['age']    = 19
mikhail['gender'] = 'Male'
```

**-Note**
> All property names are ultimately converted to a String, such that
> `object[1]`,`object[⁣[1]⁣]`, `object['1']` and `object[variable]` (when the
> variable resolves to `1`) are all equivalent.

There is another way of referring to a property called dot notation, which
usually looks less cluttered and is easier to read than the bracket alternative.
However, it only works when the property name is a [valid JavaScript IdentifierName2](http://github.com), and doesn't allow for arbitrary expressions (so, variables here are a no-go).

The rule for _dot notation_ is:

```javascript
<dot-access> ::= <identifier> "." <identifier-name>
```

This would give us an even sweeter way of defining properties:

```javasctipt
mikhail.name   = 'Mikhail'
mikhail.age    = 19
mikhail.gender = 'Male'
```

Both of these syntaxes are equivalent to creating a data property, with all
semantic flags set to `true`.

1.5. Accessing properties

Retrieving the values stored in a given property is as easy as creating new ones, and the syntax is mostly similar as well — the only difference being there isn't an assignment.

So, if we want to check on Mikhail's age:

```javascript
mikhail['age']
// => 19
```

Trying to access a property that does not exist in the object simply returns
`undefined` 3:

```javascript
mikhail['address']
// => undefined
```

1.6. Removing properties

To remove entire properties from an object, JavaScript provides the `delete`
operator. So, if you wanted to remove the `gender` property from the `mikhail` object:

```javascript
delete mikhail['gender']
// => true

mikhail['gender']
// => undefined
```

The `delete` operator returns `true` if the property was removed, `false`
otherwise. I won't delve into details of the workings of this operator, since
[@kangax](http://twitter.com/kangax) has already written a [most awesome article
on how delete works.](http://perfectionkills.com/understanding-delete/)
