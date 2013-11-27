# *i-bem.js*: User Guide #

# Overview #

## *i-bem.js*: Javascript framework for BEM ##

*i-bem.js* is a JavaScript framework for web-development within [BEM-methodology](http://en.bem.info/method/). Using
*i-bem.js* one can:

 * develop web interface in terms of blocks, elements, modifiers;
 * describe a block's behaviour in a declarative style as a set of states;
 * integrate easily JavaScript code with CSS in BEM style and with BEMHTML-templates;
 * flexibly override behaviour of library blocks.

*i-bem.js* is not suitable for:

 * replacing the general purpose framework, such as jQuery.


** Brief content of the document**:

* [Framework overview](#intro): relation to the BEM data domain terms, a brief description of the modular structure of the framework, project stub, tools for buiding the code, that was written using *i-bem.js*. 

* [Binding JS blocks to HTML](#html) — JS blocks on
  HTML page binding syntax, how HTML elements и JS blocks relate.

* [Block declarations](#decl) — JS blocks description syntax.

* [Working with DOM tree](#dom) — API for working with DOM nodes of blocks:
  elements, DOM tree modification (AJAX).

* [Events](#events) — event model *i-bem.js*: DOM events,
  BEM events, event delegation .

* [Block states](#states) — modifiers, triggers of state changes (modifiers setting), initialization of block instances.

* [Blocks interaction ](#ibc) — API for block to block communication.

* [What's next?](#docs) — links to documentation and additional materials.
* 
## BEM-methodology and JavaScript ##
As far as BEM-methodology is concerned, web interface is built  of independent **blocks** (in which **elements** are allocated). Both blocks and elements may have states, described by **modifiers**.

Web interface work is provided by multiple  **technologies**
(HTML, CSS, JS...), a description of a block consists of the implementations of these technologies. Usually implementation consists of a few files, for example:

 * `my-block.css` — describes block's appearance;
 * `my-block.bemhtml` — templates for generating block's HTML-view;
 * `my-block.js` — describes block's **dynamic behavior** in browser.

*i-bem.js* framework allows to decompose client JavaScript into components in BEM-terms:

 + **Block** is a JS component that describes the way the same interface elements work.
 For example, all buttons may be implemented as a `button` block. Then, according to the BEM-methodology, `button.css` determines the appearance of the buttons, `button.js` — determines the way they work.
 + Each page can contain more than one **instance** of a block (for example, a button). Each instance of the block corresponds to the JS object that is created dynamically in the browser's memory and stores the state of the instance. JS-object stores a link to the DOM-node that is bound to the instance of the block. 
 + **Elements** are DOM-nodes nested on DOM-node block with `class` attribute, indicating their role in BEM-subject domain (the names of blocks and elements). Elements of a block are available through the [JS-API](#elem-api) of the instance of the block.
 + **Modifiers** store information on a block state and its elements. Modifier state is written in `class` attribute on a block's DOM-nodes and elements. 
   Modifiers are operated via [JS-API](#mods-api) of the instance of the block.



## How to use i-bem.js ##

*i-bem.js* framework is included in [bem-core](http://github.com/bem/bem-core/).

Implementation consists of two modules:

* [`i-bem`][] module.<br/> 
  Basic implementation of i-bem JS-block, which all the blocks in *i-bem.js* inherit from. i-bem block is written to be used in any JS-environment, both on client and server sides (for example, in Node.js).
* [`i-bem__dom`][] module.<br/> Basic implementation of a block linked to DOM-node. Basic implementation of a block linked to DOM-node. Is intended for using on the client side, based on browsers' work with DOM. Depends on jQuery. 

Dependencies:

 * jQuery (only for `i-bem__dom` module). When using bem-core, a separate installation of jQuery is not required.
 * [ymaps/modules][ymaps] modular system. When using [bem-tools][] along with `.browser.js` technology (and derrivatives based on it), this dependancy is resolved automatically.

One can use *i-bem.js* as a part of full stack of BEM-tools. In this case it is convinient to create a project based on [project-stub](http://github.com/bem/project-stub/) template repository, which automatically installs the dependent libraries.

If one doesn't plan on using other technologies of BEM-platform, one can copy the bem-core library code to the current project.


## Build ##

According to BEM methodology web development is modular: each block is programmed separately. The final sourcecode of web pages is formed from the separate blocks code by use of **build** proceedures.


In a file system it is convenient to represent a block as a catalog, and the implementation of a block in each of the technologies as a separate file:
    desktop.blocks/
        my-block/
            my-block.css
            my-block.js
            my-block.bemhtml
            ...

    desktop.blocks/
        other-block/
            other-block.css
            other-block.js
            other-block.bemhtml
            ...

The code of the used blocks on each web page can be built in single files: 

    desktop.bundles/
        index/
            index.html
            index.css
            index.js
            ...

There are two instruments for building code of the resultant pages from separate blocks description: 

* [bem-tools](http://github.com/bem/bem-tools/);
* [enb](https://github.com/enb-make/enb) along with [enb-modules](https://github.com/enb-make/enb-modules).


## Why i-bem.js is called that way ##

According to BEM-methodology, basic JS-library of BEM platform was originally developed as a special helper block. 
This approach allows us to work with base libraries in the same way as with common blocks. Particularly, it allows to structure the code in terms of elements and modifiers, and flexibly set up the library behavior at different levels.

It was common for BEM to give names to helper blocks with `i-` prefixes . So, the name `*i-bem.js*` read as "implementation of the `i-bem` block in `js` technology".


# Binding JS blocks to HTML #

JavaScript components in *i-bem.js*  are used for making HTML-elements of a page dynamic. A typical task for JS block is to bind certain events handling to the specific HTML fragment.

In developing a web interface with *i-bem.js* framework there is a primary structure — the HTML document tree. 
In the HTML tree there are nodes, bound with JS blocks — interactive interface elements . The point of a JS block binding is a HTML element.  The name of the block is indicated in its `class` attribute , [block parameters](#html-syntax) are indicated in `data-bem` attribute.

When loading а page, the browser runs [blocks initialization](#init). During the initialization process blocks instances are generated – JS objects for all the blocks, mentioned in HTML elements on the page. 
JS object, bound to the HTML element handles its [DOM events](#dom-events) and stores the state of the given instance of a block.

*i-bem.js* allows to implement a JS component, not bound to HTML — [block without DOM representation ](#i-blocks). Such block gives API, similar to common JS blocks. 

This approach of binding JavaScript components to HTML has the following advantages:

 * graceful interface degradation on the client-side with JavaScript disabled; 
 * _progressive rendering_ — the opportunity to start drawing interface elements before the download of all data pages is over ( eg images ).

<a name="html-syntax"></a>

## Blocks binding syntax ##

To bind a block to HTML-элементу (for example, `<div>...</div>`), one should:

 * **Mark a block in HTML tree**.<br/>
 Include the block name in the list of classes of HTML-element (`class` attribute).

```HTML
<div class="my-block">...</div>
```

 * **Initialize the block instance**.<br/>
 Include the `i-bem` class in the list of classes of HTML-element. The presence of this class allows the framework to initialize the block.

```HTML
<div class="my-block i-bem">...</div>
```

 * **Pass parameters to the block instance**.<br/> Write parameters of a block in `data-bem` attribute. Write the block parameters
 in JSON format as a structure hash __block name—parameters hash__. Parameters are transferred пустой списокto the block instance at the point of initialization.
 ([read more...](#data-bem)).

```HTML
<div class="my-block i-bem" data-bem='{ "my-block": { "name": "ya" } }'>...</div>
```

One HTML element does not necessarily correspond to one block instance. The are following types of binding of blocks and HTML elements:

### One HTML element — one JS block ###


It is easiest and most common way to bind blocks to HTML.

For example: `div` HTML element on which `my-block` is placed, block parameters — empty list `{}`.

```HTML
<div class="my-block i-bem" data-bem='{ "my-block": {} }'>
    ...
</div>
```


<a name="mixes"></a>

### One HTML element — several JS blocks ###

A technique in BEM methodology for placing multiple blocks on one HTML element is called **mix**.

For example: `div` HTML element, where the `user` block with the `name` parameter: `pushkin` and `avatar` block with the `img`: `http://...` parameter.

```HTML
<div class="user avatar i-bem"
    data-bem='{
        "user": { "name": "pushkin" },
        "avatar": { "img": "http://..." }
     }'>
     ...
</div>
```

<a name="distrib-block"></a>

### One JS block on several HTML elements ###

This design allows you to transparently implement blocks consisting of several components , which state must be agreed. 
For example, the " tab " widget, whereon clicking on the header of the tab (one HTML- element) changes the contents of the tab (the other HTML- element).
Another example - a marker indicating a point on the map (the first element) and the bound description of the point in the list next to the map (the second element).

In order to bind the instance of a block to multiple HTML elements, one needs to write the same `id` value in block parameters for all of the bound HTML elements.
The `id` value can be a random string.

For example: The instance of a block `notebook` bound to HTML elements  `div` and `span`, in the block parameters written common `id` — `maintab`.

```HTML
<div class="notebook i-bem" data-bem='{ "notebook": { "id": "maintab" }}'>
</div>
...
<span class="notebook i-bem" data-bem='{ "notebook": { "id": "maintab" }}'>
</span>
```

As a result, when initializing the blocks a single JS-object is created, 
 which [`{jQuery} domElem`](#domElem) field contains links to both DOM-nodes.

`id` Modifier is only used *during initialization* of a block instance. `id` value must be unique within one block instances inside of one [initialization wave](#init-wave).


<a name="i-blocks"></a>

### Blocks without DOM representation ###

Infrastructure code that solves common interface tasks: communication with the backend, general computing, etc. When working with *i-bem.js* code can take the form of block, like all the rest of JS-code.
In order not to bind these blocks to HTML-tree manually, *i-bem.js*  provides the opportunity to create blocks without DOM-representation.

Blocks without DOM-representation:

 * They are not written in HTML code of a page.
 * Declared as [module extension`i-bem`](#bem-decl), not the `i-bem__dom`, as in the case of blocks with DOM-representation.
 * They shoul be [initialized explicitly](#init-bem).


## Parameters transfer syntax##

Block parameter — is an arbitrary JavaScript object, that is transferred to the block at the point of initialization.
Parameters allow to modify the behavior of the block instance bound to the given HTML element.

The value of `data-bem` attribute contain the parameters of *all the JS blocks on this node*.
Parameters are transferred as a hash in JSON format:

 + key — `{String}` block name;
 + value — `{Object}` the parameters of the given block. If this block instance has no parameters, declare empty hash `{}`.

This parameters format proceeds from the following:

 * Indication of the block name in  parameters  allows to avoid having to parse value of the `class` attribute, which simplifies and accelerates blocks initialization.
 *This same solution allows to place multiple blocks in one HTML-element without the need to multiply the block attributes.

Value of `data-bem` attribute  should contain valid JSON.

# Block declaration #

JS implementation of a block describes the functionality of a certain class of web interface elements. In a specific interface, each block can have several instances. 
Each instance of a block implements functionality of the whole class and has its own state, independent of the others.

In terms of object oriented programming paradigm:

 * block — класс;
 * block instance — class instance.

In terms of OOP, the entire functionality of the block is implemented in modules in class (= block)_methods_.
Block methods are subdivided into: 

 * block instance methods;
 * static methods.

Block code in *i-bem.js* is commonly known as **declaration**, to point out used in BEM terminology declarative programming style. 
In accordance with the declarative style the block behavior is programmed as the following statements _set of conditions — block reaction_.   


## Declaration syntax ##

In order to declare a new JS block **with DOM representation**
(bound to HTML element), one has got to extend `i-bem__dom` [ymaps][] module.

To declare blocks use `decl`method, that has three parameters:

1. Block name `{String}` or [block description ](#decl-selector) `{Object}`.
2. Block instance methods — `{Object}`.
3. Static methods — `{Object}`.

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl(/* block name or block description */,
    {
        /* instance methods */
    },
    {
        /* static methods */
    }
);

provide(DOM);

});
```

-------------------------------------------------------------------------------

**NB** In terms of [ymaps][] modular system, declarations of multiple blocks represents  overriding of the same module
  `i-bem__dom`. And yet in terms of *i-bem.js* this is the way *разные объекты different objects* for building block instances are created.

-------------------------------------------------------------------------------

<a name="bem-decl"></a>

Blocks, that do not have DOM representation are declared as extension of `i-bem` [ymaps][]-module .
For declaring use `decl` method, receiving the same parameters, as`decl` method of `i-bem__dom` module:

```js
modules.define('i-bem', function(provide, BEM) {

BEM.decl(/* block name or block description */,
    {
        /* instance methods */
    },
    {
        /* static methods */
    }
);

provide(BEM);

});
```


-------------------------------------------------------------------------------

**NB**: It is convenient to design infrastructure code as a block without DOM representation, if it is planned to use API of
BEM blocks (states, expressed by modifiers, BEM events etc.). Without using BEM terms/BEM stack infrastructure code can be designed as  [ymaps][] module. For example:

```js
modules.define('router', function(provide) {

provide({
    route : function() { /* ... */ }
});

});
```

-------------------------------------------------------------------------------

<a name="decl-selector"></a>


## Block description in a declaration ##

Первый параметр метода  представляет собой описание блока, в
котором будут применяться объявленные в декларации методы. The first parameter of the `decl` method is a block description, which will be announced in the declaration implemented methods.
Описание обязательно содержит имя блока и может дополнительно содержать: Description must contain the block name and may additionally contain:

* ограничение сферы действия декларации определенной модификацией
  блока; limiting the scope of the declaration certain modification of the block;
* список родительских блоков, методы которых должен наследовать данный
  блок. list of parent blocks, methods which should inherit the block.

Описание может быть задано в одной из двух форм: Description may be given in one of two forms:

1. Block name - string.<br/>
   Объявленные методы будут применяться во всех экземплярах блока независимо от их состояний (модификаторов). Declared methods will be applied in all instances of the block , regardless of their states ( modifiers ) 
Пример: декларация методов для блока `button`. Example: Declaration of methods for the block

    ```js
DOM.decl('button',
    { /* instance methods */ },
    { /* static methods  */ }
);
    ```

2. Block description  - hash.<br/> This is an example of a methods declaration for the `button` block with the `type` modifier with value
   `link` (describes behavior of pseudo buttons):

    ```js
DOM.decl({ block: 'button', modName: 'type', modVal: 'link' },
    { /* instance methods */ },
    { /* static methods */ }
);
    ```

-------------------------------------------------------------------------------

**NB** If static methods are described in a declaration for the block with specific modifiers, they will be available in all instances of this block *regardless of the modifiers values*.    
  Модификаторы являются свойствами экземпляров блоков, а статические методы принадлежат классу блока и поэтому не могут
  учитывать ограничения по модификатору. Modifiers are the properties of block instances, and static methods belong to the class of the block can not consider the limitations of the modifier.

-------------------------------------------------------------------------------


## Сontext ##

**Block instance methods** are executed in the JS object context of the block instance.  Accordingly, the key word `this` in instance methods of the block refers to JS object of a **block instance**. 

**Static methods** are executed in the context of JS-object, that corresponds to the class of the block.
  Accordingly, `this` keyword  in static methods of the block refers to the **block class**, not to an instance.

Контекст содержит зарезервированные поля: Context contains the reserved fields:

 + `this.__self`: Refers to the static methods of the class, which the instance belongs to. It is defined in the instance methods of a block. Для
   статических методов не имеет смысла и не определен. It does not make sense for static methods and is not defined.

    For example: Static method calling `staticMethod` in `onEvent` method of `my-block` block instance. 

```js
DOM.decl('my-block', {
    onEvent: function() {
        this.__self.staticMethod(); // calling of static method
        this.doMore();
    },
    {
        staticMethod: function() { /* ... */ }; // static method defining 
    }
});
```

 + `this.__base`: Refers to the implementation of the method in the base class, which it is inherets from. 
    Allows a super call. It is defined in the block instance methods and in static methods of the block.

    For example: call (and modification) of `_onClick` method of parent class (the base implementation of the method in the `button` class).

```js
DOM.decl({ block: 'my-button', baseBlock: 'button' }, {
    _onClick: function() {
        this.__base();
        this.doMore();
    }
);
```

-------------------------------------------------------------------------------

**NB** In blocks development using  *i-bem.js* внутреннимметодам блока private methods of a block, not intended for external use, commonly given names starting with an underscore. For example, `_onClick`.

-------------------------------------------------------------------------------


# Работа с DOM-деревом Working with DOM tree#

<a name="domElem"></a>

## DOM node of a block instance ##

All the block instances, bound to the DOM, contain jQuery object
in the `{jQuery} this.domElem` field, that refers to one or more DOM nodes, bound to this instance of the block.

<a name="elem-api"></a>

## Elements ##

BEM elements of blocks  are represented in the *i-bem.js* as DOM nodes, nested into the DOM node of the block instance.
To access DOM nodes of elements and work with their modifiers, one should use API, of the block instance.

The block instance provides two methods to access the elements of a given instance:

* Кеширующий доступ: Access with cache`elem(elems, [modName], [modVal])`. There is no need to store as a variable the element obtained this way.

```js
DOM.decl('b-link', {
    setInnerText: function() {
        this.elem('inner').text('Link text');
        /* ... */
        this.elem('inner').text('Another text');
    }
);
```

* Некеширующий доступ: Access without cache `findElem(elems, [modName], [modVal])`.

```js
DOM.decl('b-link', {
    setInnerText: function() {
        var inner = this.findElem('inner');
        inner.text('Link text');
        /* ... */
        inner.text('Another text');
    }
});
```

In the process of [dynamic adding and removing block elements](#dynamic) it can  be necessary to reset cache of elements. This is what the `dropElemCache('elements')` method is used for. 
The space-separated list of names as a parameter for the elements, cache reset is needed for:  

```js
DOM.decl('attach', {
    clear: function() {
        DOM.destruct(this.elem('control'));
        DOM.destruct(this.elem('file'));
        return this.dropElemCache('control file');
    }
});
```


Complete API description for working with elements is contained in the source code of [`i-bem__dom`][] module.

<a name="dynamic"></a>

## Dynamic updating of blocks and elements in the DOM- tree ##

In modern interfaces one often needs to create new pieces of DOM tree and replace the old ones in the process of work (AJAX). 
В  предусмотрены следующие функции для добавления и замены
фрагментов DOM-дерева: There are functions provided for adding and replacing fragments DOM- tree in *i-bem.js*:

* `append` — add a DOM fragment into to the end of specified context в конец указанного контекста;
* `prepend` — add a DOM fragment into to the end of specified context в начало указанного контекста;
* `before` — add a DOM fragment before the specified contextперед указанным контекстом;
* `after` — add a DOM fragment after the specified context после указанного контекста;
* `update` — to replace a DOM fragment the specified context внутри указанного контекста;
* `replace` — to replace the specified context with a new DOM fragment.

All function blocks automatically perform [initialization on the updated DOM tree fragment](#init-ajax).  

To simplify the creation of BEM entities on renewable fragments of DOM-tree, it is possible to use the template engine
[BEMHTML](http://ru.bem.info/articles/bemhtml-reference/), and connect it as a [ymaps][] module. 
BEMHTML позволяет по описанию БЭМ-сущностей allows to generate DOM nodes in accordance with the rules of BEM naming directly in JS code block in [BEMJSON](http://ru.bem.info/articles/bemhtml-reference#bemjson) format.


**For example**: `_updateFileElem` method of `attach` block removes `file` element, if it existed, and generates of a new element by using `BEMHTML.apply`: 

```js
modules.define(
    'i-bem__dom',
    ['BEMHTML', 'strings__escape'],
    function(provide, BEMHTML, escape, DOM) {

DOM.decl('attach', {
    _updateFileElem : function() {
        var fileName = extractFileNameFromPath(this.getVal());
        this.elem('file').length && DOM.destruct(this.elem('file'));
        DOM.append(
            this.domElem,
            BEMHTML.apply({
                block : 'attach',
                elem : 'file',
                content : [
                    {
                        elem : 'icon',
                        mods : { file : extractExtensionFromFileName(fileName) }
                    },
                    { elem : 'text', content : escape.html(fileName) },
                    { elem : 'clear' }
                ]
            }));
        return this.dropElemCache('file');
    }
});

provide(DOM);

});
```


# Events #

*i-bem.js* supports two kinds of events:

<a name="dom-events"></a>

* **DOM events** — JavaScript events, triggered in DOM nodes of the corresponding blocks. These are the events that reflect a user's interaction with the interface (click, mouse pointing, text input, etc.). 
  DOM events are usually handled the block instance, in the DOM node of which they were triggered.
* **BEM events** — собственные events, generated by blocks. They allow to organize API [interaction with block](#ibc). BEM events usually handle the block instance, tracking the states of other blocks, which generate events. 

When planning interface architecture, one needs to consider, that DOM events should only be used in block instance methods / *внутренних* процедурах блока. 
In order for a block to interact with other blocks с *внешней* средой (другими блоками), one should use BEM events.


-------------------------------------------------------------------------------

**NB** Working with DOM-events is fully realized by means of jQuery framework.  

-------------------------------------------------------------------------------


<a name="delegated-events"></a>

## Event delegation ##

BEM and DOM events handling can be **delegated** to the container/контейнеру
(either to the entire document or to the specific DOM-node). В этом случае контейнер
служит точкой обработки событий, возникающих на любом из
дочерних узлов контейнера, даже если в момент подписки на события
некоторые из дочерних узлов еще не существовали. In this case container is a point of events handling, triggered in any of the child nodes, so it will hanlde events for all child nodes, even if they do not exist yet and will be loaded later. 

For example, menu block can contain nested blocks (or elements, this depends on the specific block implementation) — menu items. 
It makes sence to delegate обработку кликов click event handling on menu items to the menu block itself. 
This, first of all, allows to save the resources/ overhead of adding events затраты ресурсов на
подписку на события (it's cheaper to add one event listener to a container, than many events listeners to the elements). 
Secondly, you can change menu contents: add and remove menu items without
adding event listener for added items or remove event listeners from deleted items.

* [**DOM events delegation**](#dom-events-delegated) can be used for handling DOM events, triggered in the DOM node of the block instance or in the DOM nodes of its elements.
  DOM events delegation can be performed either for all the block instances within the document, or for the block instances within a specific fragment of the HTML tree.
  внутри указанного контекста? (фрагмента HTML-дерева).

  `window.document` always performs as a container to which DOM events handling is delegated.


* [**BEM events delegation**](#bem-events-delegated) should be used to handle events in *block instances*, contained in the specific DOM node.

    **arbitrary DOM node** can perform as a container, that BEM events handling is delegated to. The entire (`window.document`) document performs as a container by default. 
    Often a block handles BEM events of the nested blocks, then DOM node of the handler block [`this.domElem`](#domElem) would be a container.


The complete list of helpers for adding event listeners to delegated events can be found in the source code of the [`i-bem__dom`][] module.


## DOM events ##

To add event listeners to DOM nodes Для подписки на DOM-события на узлах, bound to a block or an element, use instance block method `bindTo([elem], event,
handler)`.

**For example**: At the point of [block instance initialization](#init)
`my-block` event listener to `click` event is added, when the event is triggered, the block выставляет себе gets [modifier](#modifier) `size` with the value `big`.

```js
DOM.decl('my-block', {
    onSetMod : {
        'js' : {
            'inited': function() {
                this.bindTo('click', function(e) {
                    var domElem = $(e.currentTarget); // DOM element, на котором the event слушается событие
                                                      // in this case, it is the same as this.domElem
                    this.setMod('size', 'big');
                });
            }
        }
    }
});
```

**For example**: At the point of [block instance initialization](#init) `my-form`
  the event listener `click` event of `submit` element is added, when it is triggered, method of a block instance `_onSubmit` will be called.

```js
DOM.decl('my-block', {
    onSetMod : {
        'js' : {
            'inited': function() {
                this.bindTo('submit', 'click', function(e) {
                    var domElem = $(e.currentTarget); // DOM element with an event listener
                                                      //   in this case, it is the same as this.elem('submit')
                    this._onSubmit();
                });
            }
        }
    },

    _onSubmit : function() { /* ... */ }
});
```

-------------------------------------------------------------------------------

**NB** Handler function is executed in the context of the block instance in which the event emitted.
   

-------------------------------------------------------------------------------

**Removal of event listener** to DOM events will be run automatically when the block instance is removed. If it is necessary to unsubscribe manually
в процессе работы блока while the block is being used, one should use `unbindFrom([elem], event, handler)` method. 


<a name="dom-events-delegated"></a>

### DOM events delegation ###

Delegating of DOM events handling is performed by the `liveBindTo([elem], event, handler)` method. In a block declaration the point,
reserved for adding delegated DOM events listeners для подписки на делегированные DOM-события, serves `live` property in the  static block methods segment.

**For example**: All instances of  the `menu` block add to подписываются на
  delegated DOM event listener `click` своих элементов of its elements `item`. Method
  `_onItemClick` of the `menu` block instance will be run  when clicking on any item (element `item`) in this menu,
  regardless of whether this item existed at the point of the block instance initialization.
   
```js
DOM.decl('menu', {
    _onItemClick : function(e) {
        var clickedItem = $(e.currentTarget); // 'item' element of the 'menu' block, на котором слушается 'click' DOM event
    }
}, {
    live : function() {
        this.liveBindTo('item', 'click', function() {
            this._onItemClick();
        });
        return false; // if the block initialization can not be deferred
    }
});
```

With `live`property in a block declaration block instances by default inititalization will be *отложена/deferred* till the moment, when
the block instance will be needed ([lazy initialization](#init-live)). 
A DOM event on the block instance, to which a delegated event listener is added, can be such a moment
, or referring to the block instance [из другого блока/from another block](#ibc).
If the block initialization can not be deferred ([automatic initialization](#init-auto) is required), следует вернуть `false` в
результате выполнения функции в значении `live` property.  

-------------------------------------------------------------------------------

**NB**  Handler function is executed in the context of the nearest block of this type in the path of DOM event (from the bottom up the DOM tree).

-------------------------------------------------------------------------------

**Removal of the delegated event listener** never is performed automatically. To remove the delegated event listener use `liveUnbindFrom([elem], event, [handler])` method. 


### DOM event object ###

jQuery object is transferred as a parameter , describing DOM event — [`{jQuery.Event}`](http://api.jquery.com/category/events/event-object/) to the handler function.

If a DOM event was generated manually, all the parameters, transferred
to the `trigger` function at the point of event creation при создании события, will be transferred to the handler function in the same order after the event object после объекта события.



<a name="bem-events"></a>

## BEM events ##

Unlike DOM events, BEM events are generated not in DOM elements, but in **block instances**. Block elements can not generate BEM events.

To generate BEM event, `emit(event)` block instance method  is used.

**For example**: When a user clicks on DOM element of the `submit`button
(DOM event `click` is emitted),`_onClick()` method of `submit`block instance is ecexecuted, in which **BEM event** `click` is generated
in the case, if the block doesn't have the `disabled` modifier не выставлен
модификатор at the moment:

```js
DOM.decl('submit', {
    onSetMod: {
        'js': {
            'inited': function() {
                this.bindTo('click', this._onClick); // adding DOM event listener "click"
            }
        }
    },

    _onClick: function() {
        if(!this.hasMod('disabled')) {
            this.emit('click'); // creating BEM event "click" 
        }
    }
});
```

For aading BEM events listeners the block instance methods `on(event, [data], handler, [handlerCtx])` are used.

**For example**: At the point of initialization HTML form (of `my-form` block instance)
выполняется поиск the `submit` nested into the form button is found and `click` BEM event listener of the button is added.  
As a result, when the button is clicked on (instance of `submit` block) `_onSubmit` method  формы??
of (`my-form` block instance) form will be executed.

```js
DOM.decl('my-form', {
    onSetMod: {
        'js': {
            'inited': function() {
                this.findBlockInside('submit').on(
                    'click', // BEM event name
                    this._onSubmit, // block instance method my-form
                    this); // context for running _onSubmit — my-form block
            }
        }
    },

    _onSubmit: function() { /* ... */ }
});
```

-------------------------------------------------------------------------------

**NB** If the last parameter of`on` —
  `[handlerCtx]` method is not provided, the context for handler function execution контекстом для выполнения функции-обработчика will be the block, in which
  the BEM event has been emitted. (In the example above — it is the `submit` block.)

-------------------------------------------------------------------------------


**Removing the event listener/ Удаление подписки** to BEM events is performed automatically when the block instance is removed. To remove it manually while the block is at work, 
use the `un(event, [handler], [handlerCtx])` block instance method/ метод экземпляра блока.


<a name="bem-events-delegated"></a>

### BEM event delegation ###

BEM events delegation means, that a block adds a event listener to the specific BEM event блок подписывается на
определенное BEM-событие of **all the block instances** with the name specified/ с заданным именем
**в пределах заданного контекста/ within the specified context**. Adding the delegated BEM event listener/ Подписка на делегированные
BEM-события is performed with a static method *класса блока/ block class*
`on([ctx], event, [data], handler, [handlerCtx])`.

Parameters:

* `{jQuery} [ctx]` — DOM-узел, в пределах которого отслеживаются
BEM-события (контейнер). Если не указан, в качестве контейнера
используется весь документ.
* `{String} event` — Имя BEM-события.
* `{Object} [data]` — Произвольные данные, передаваемые
  функции-обработчику.
* `{Function} handler` — Функция-обработчик события.
* `{Object} [handlerCtx]` — Контекст функции-обработчика
  события. Обычно в качестве контекста должен выступать тот экземпляр
  блока, который подписывается на BEM-событие, а не тот, в котором BEM-событие
  произошло.


**Пример**: При инициализации экземпляров блока `menu` выполняется
  подписка на BEM-событие `click` всех ссылок (экземпляров блока
  `link`) в пределах DOM-узла, к которому привязано меню
  (`this.domElem`). В качестве контекста функции-обработчика
  передается экземпляр блока, в котором событие будет обрабатываться
  (`this`). При [уничтожении экземпляров блока](#destruct) `menu`

```js
DOM.decl('menu', {
    onSetMod : {
        'js' : {
            'inited' : function() {
                DOM.blocks['link'].on( // подписка на BEM-событие
                    this.domElem, // контейнер — DOM-узел экземпляра блока menu
                    'click', // BEM-событие
                    this._onLinkClick, // обработчик
                    this); // контекст обработчика — экземпляр блока menu
            },

            '' : function() {
                DOM.blocks['link'].un( // удаление подписки на BEM-событие
                    this.domElem,
                    'click',
                    this._onLinkClick,
                    this);
            }
        }
    },

    _onLinkClick : function(e) {
        var clickedLink = e.target; // экземпляр блока 'link', на котором произошло BEM-событие 'click'
    }
});
```

-------------------------------------------------------------------------------

**NB** Если не указывать параметр `[handlerCtx]` метода `on`,
  контекстом для функции-обработчика будет тот блок, в котором
  *возникло* BEM-событие.

-------------------------------------------------------------------------------

**Удаление подписки** на делегированные BEM-события никогда не
  происходит автоматически. Всегда следует явно удалять подписку при
  помощи статического метода блока `un([ctx], event, [handler],
  [handlerCtx])`.


Полное описание API для работы с BEM-событиями содержится в исходном
коде модулей [`i-bem`][] и [`i-bem__dom`][].


<a name="api"></a>

## Объект BEM-события ##

В качестве параметра функции-обработчику передается объект,
описывающий BEM-событие. Объект BEM-события `events.Event` определен
в [ymaps][]-модуле
[`events`](https://github.com/bem/bem-core/blob/v1/common.blocks/events/events.vanilla.js)
библиотеки bem-core. Содержит поля:

* `target` — Экземпляр блока, в котором произошло BEM-событие.
* `data` — Произвольные дополнительные данные. Передается в качестве
  параметра `data` в момент подписки на BEM-событие или при создании
  BEM-события блоком.
* `result` — Последнее значение, возвращенное обработчиком данного
  события. Аналогично [jQuery.Event.result](http://api.jquery.com/event.result/).
* `type` — Тип события. Аналогично
[jQuery.Event.type](http://api.jquery.com/event.type/).


<a name="ibc"></a>

# Состояния блока #

Проектируя динамический блок в стиле БЭМ, нужно представить всю логику
изменений, происходящих в нем, как набор **состояний** блока. Тогда
поведение блока определяется **триггерами** — callback-функциями, которые
выполняются при переходе блока из одного состояния в другое.

Такой подход позволяет писать код блока в декларативном стиле как
набор утверждений вида:

* Описание состояния — действия, выполняемые при переходе в данное состояние.

<a name="modifiers"></a>

## Модификаторы ##

Согласно БЭМ-методологии, состояние блока и его элементов описывается
**модификаторами**.

* Модификатор — это **имя** и **значение**. Например, `size`: `m`.

* **Простой модификатор**. Частный случай, когда модификатор либо
  присутствует у блока, либо отсутствует. Например, `disabled`. В
  *i-bem.js* представлены как модификаторы с булевым
  значением. Например: `disabled`: `true`. При выставлении
  модификатора с неуказанным значением *i-bem.js* автоматически
  присваивает ему значение `true`.

* Каждому блоку можно установить один или несколько модификаторов.

* Блок может не иметь модификаторов.

В *i-bem.js* модификаторы устанавливаются при
  [инициализации экземпляра блока](#init) (если модификаторы и их
  значения указаны в атрибуте `class` соответствующего HTML-элемента).

-------------------------------------------------------------------------------

**NB** При инициализации блока с модификаторами триггеры на установку
  данных модификаторов *не выполняются*, так как экземпляр блока в
  этом случае получает начальное состояние, а не меняет его.

-------------------------------------------------------------------------------

Модификаторы могут добавляться, удаляться и менять значения:

* В ходе выполнения кода блока (например, в качестве реакции на [DOM-события](#dom-events)).
* По запросу из другого блока. Подробнее см. раздел [Взаимодействие блоков](#ibc).

При добавлении, удалении и изменении значений модификаторов выполняются триггеры.


<a name="mods-api"></a>

### Управление модификаторами ###

Экземпляр блока предоставляет методы для установки, проверки значений
и удаления модификаторов данного экземпляра.

-------------------------------------------------------------------------------

**NB**: Модификаторы нельзя устанавливать, напрямую меняя CSS-классы на
соответствующем DOM-узле. Для изменения значений модификаторов следует
использовать описанное ниже API, предоставляемое *i-bem.js*.

-------------------------------------------------------------------------------

**Пример**: Экземпляр блока `square` может по клику на DOM-элементе
блока переключаться между значениями `green` и `red` модификатора
`color`, если не выставлен модификатор `disabled`:

```js
DOM.decl('square', {
    onSquareClick: function(e) {
        if(!this.hasMod('disabled')) {
            this.toggleMod('color', 'green', 'red');
        }
    }
});
```

Эти же методы используются для управления модификаторами элементов
блока. Для этого в качестве первого (необязательного) параметра
указывается ссылка на объект элемента (а не имя элемента).

**Пример**: Блок `searchbox` по клику может выставлять своему элементу
`input` простой модификатор `clean` (подразумеваемое значение —
`true`):

```js
DOM.decl('searchbox', {
    _onClick: function() {
        this.setMod(this.elem('input'), 'clean');
    }
});
```

-------------------------------------------------------------------------------

**NB** При управлении модификаторами элементов в качестве первого
  параметра необходимо указывать ссылку на **DOM-узел элемента**, а не
  имя элемента. В противном случае возникла бы неоднозначность:
  имеется в виду установка блоку *модификатора* `input` в значение
  `clean` или установка элементу `input` *простого модификатора* `clean`.

-------------------------------------------------------------------------------

Полное описание API для управления модификаторами приведено в
исходном коде модулей [`i-bem`][] и [`i-bem__dom`][].


## Триггеры на установку модификаторов ##

Выполнение триггеров на установку модификаторов разбито на две фазы:

1. **До установки модификатора**. Эта фаза зарезервирована для
   возможности **отменить** установку модификатора. Если хотя бы один
   из триггеров, выполняемых в этой фазе, вернет `false`, установки
   модификатора не произойдет.
2. **После установки модификатора**. Триггеры, выполняемые в этой
   фазе, уже не могут отменить установку модификаторов.


Триггеры могут быть привязаны к следующим типам изменений значений модификаторов:

1. установка *любого* модификатора в *любое* значение;
2. установка *конкретного* модификатора `modName` в *любое* значение (в том числе
   установка простого модификатора в значение `true`);
3. установка *конкретного* модификатора `modName` в *конкретное* значение `modVal`;
4. установка модификатора в значение `''` (пустая строка), что
   эквивалентно удалению модификатора или установке простого
   модификатора в значение `false`).


При установке модификатора `modName` в значение `modVal` триггеры
каждой фазы (если они определены) вызываются в том порядке, в котором они
перечислены в приведенном выше списке событий (от общего к частному).

Таким образом, при определении триггера пользователь указывает:

* фазу выполнения (до или после установки модификатора);
* тип события (имя и устанавливаемое значение модификатора).

### Декларация триггеров ###

Триггеры, выполняемые при установке модификаторов, описываются в
декларации блока. Для этого в хэше методов экземпляра блока
зарезервированы свойства:

* `beforeSetMod` — триггеры, вызываемые до установки
  **модификаторов блока**.
* `beforeElemSetMod` — триггеры, вызываемые до установки
  **модификаторов элементов**.
* `onSetMod` — триггеры, вызываемые после установки
  **модификаторов блока**.
* `onElemSetMod` — триггеры, вызываемые после установки
  **модификаторов элементов** блока.

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl(/* селектор блока */,
    {
        /* методы экземпляра */
        beforeSetMod: { /* триггеры до установки модификаторов блока*/}
        beforeElemSetMod: { /* триггеры до установки модификаторов элементов*/}
        onSetMod: { /* триггеры после установки модификаторов блока */ }
        onElemSetMod: { /* триггеры после установки модификаторов элементов */ }
    },
    {
        /* статические методы */
    }
);

provide(DOM);

});
```

Значение свойств `beforeSetMod` и `onSetMod` — хэш, связывающий
изменения модификаторов с триггерами. В качестве параметров триггерам
передаются:

* имя модификатора;
* выставляемое значение модификатора;
* предшествующее (для `beforeElemSetMod`) или текущее (для `onElemSetMod`) значение модификатора.

```js
{
    'mod1': function(modName, modVal, prevModVal) { /* ... */ }, // установка mod1 в любое значение
    'mod2': {
        'val1': function(modName, modVal, prevModVal) { /* ... */ }, // триггер на установку mod2 в значение val1
        'val2': function(modName, modVal, prevModVal) { /* ... */ }, // триггер на установку mod2 в значение val2
        '': function(modName, modVal, prevModVal) { /* ... */ } // триггер на удаление модификатора mod2
    'mod3': {
        'true': function(modName, modVal, prevModVal) { /* ... */ }, // триггер на установку простого модификатора mod3
        '': function(modName, modVal, prevModVal) { /* ... */ }, // триггер на удаление простого модификатора mod3
    },
    '*': function(modName, modVal, prevModVal) { /* ... */ } // триггер на установку любого модификатора в любое значение
}
```

Для триггера на установку любого модификатора блока в любое значение
существует сокращенная форма записи:

```js
beforeSetMod: function(modName, modVal, prevModVal) { /* ... */ }
onSetMod: function(modName, modVal, prevModVal) { /* ... */ }
```

Для свойств `beforeElemSetMod` и `onElemSetMod` в хэш значений
добавляется дополнительный уровень вложенности, задающий **элемент**,
на установку модификаторов которого устанавливаются триггеры. В
качестве параметров триггеру передаются:

* имя элемента;
* имя модификатора;
* выставляемое значение модификатора;
* предшествующее (для `beforeElemSetMod`) или текущее (для `onElemSetMod`) значение модификатора.


```js
{
    'elem1': {
        'mod1': function(elem, modName, modVal, prevModVal) { /* ... */ }, // триггер на установку mod1 элемента elem 1 в любое значение
        'mod2': {
            'val1': function(elem, modName, modVal, prevModVal) { /* ... */ }, // триггер на установку mod2 элемента elem1 в значение val1
            'val2': function(elem, modName, modVal, prevModVal) { /* ... */ } // триггер на установку mod2 элемента elem1 в значение val2
            }
        },
    'elem2': function(elem, modName, modVal, prevModVal) { /* ... */ } // триггер на установку любого модификатора элемента elem2 в любое значение
}
```

Сокращенная запись для триггера на установку любого модификатора элемента
`elem1` в любое значение:

```js
beforeElemSetMod: { 'elem1': function(elem, modName, modVal, prevModVal) { /* ... */ } }
onElemSetMod: { 'elem1': function(elem, modName, modVal, prevModVal) { /* ... */ } }
```

### Примеры триггеров ###

Типовая задача триггеров, вызываемых после установки модификатора или
изменения его значения (свойство `onSetMod`) — выполнить операции над
DOM-узлом блока, необходимые при переходе в новое состояние.

**Пример**: Экземпляр блока `input` при установке простого
  модификатора `focused` (в значение `true`) очищает поле ввода —
  заменяет пустой строкой текст DOM-узла блока.

```js
DOM.decl('input', {
    onSetMod : {
        'focused' : {
            'true' : function() {
                this.domElem.val(''); // очистить поле ввода
            }
        }
    }
});
```

Триггеры, выполняемые перед установкой модификатора (свойство
`beforeSetMod`), необходимы для проверки текущего состояния экземпляра
блока и возможности отменить переход в другое состояние.

**Пример**: Экземпляр блока `input` перед установкой простого
  модификатора `focused` проверяет, не выставлен ли у него модификатор
  `disabled`. Если `disabled` выставлен, проверка вернет `false` и
  установки модификатора `focused` не произойдет.

```js
DOM.decl('input', {
    beforeSetMod : {
        'focused' : {
            'true' : function() {
                return !this.hasMod('disabled'); // вернет false, если disabled
            }
        }
    }
});
```


<a name="init"></a>

## Инициализация ##

Инициализация блока — это создание в памяти браузера JS-объекта,
соответствующего экземпляру блока. Инициализация экземпляров блоков выполняется
методом `init()` модуля `i-bem__dom` на заданном фрагменте DOM-дерева.

Каждому экземпляру блока можно приписать три состояния:

* экземпляр блока не инициализирован (JS-объект не создан);
* экземпляр блока инициализирован (JS-объект создан в памяти браузера);
* экземпляр блока уничтожен (удалены все ссылки на JS-объект экземпляра
  блока и он может быть удален сборщиком мусора).

В *i-bem.js* эти состояния экземпляра блока описываются с помощью служебного
модификатора `js`.

* До инициализации экземпляр блока не имеет модификатора `js`.

```HTML
<div class="my-block i-bem" data-bem="..." >...</div>
```

* В момент инициализации экземпляру блока устанавливается модификатор
  `js` в значении `inited`.

```HTML
<div class="my-block i-bem my-block_js_inited" data-bem="...">...</div>
```

* Если в процессе работы удаляется фрагмент DOM-дерева (при помощи
  метода `destruct` модуля `i-bem__dom`) , то вместе с
  ним удаляются экземпляры блоков, все HTML-элементы которых находятся
  в этом фрагменте. Перед удалением экземпляра блока модификатор `js`
  удаляется.

-------------------------------------------------------------------------------

**NB** Если экземпляр блока был
  [привязан к нескольким HTML-элементам](#distrib-block), блок будет существовать,
  пока в HTML-дереве сохраняется хотя бы один элемент, с которым он
  связан.

-------------------------------------------------------------------------------


Если на HTML-элементе размещено несколько экземпляров других блоков, то
инициализация одного из них (появление модификатора `my-block_js_inited`)
не влияет на инициализацию остальных.

**Пример**: На HTML-элементе инициализирован только экземпляр блока `my-block`,
экземпляр блока `lazy-block` не инициализирован:

```HTML
<div class="my-block my-block_js_inited lazy-block i-bem"
    data-bem='{ "my-block": {}, "lazy-block": {} }' >
    ...
</div>
```

-------------------------------------------------------------------------------

**NB** Наличие модификатора `js` позволяет писать разные CSS-стили для
  блока в зависимости от того, инициализирован он или нет.

-------------------------------------------------------------------------------


### Конструктор экземпляра блока ###

На изменение значений модификатора `js` можно назначать триггеры так
же, как и для любых других модификаторов блока.

Триггер на установку модификатора `js` в значение `inited` выполняется
при создании блока. Этот триггер можно считать **конструктором
экземпляра блока**:

```js
onSetMod: {
    'js': {
        'inited': function() { /* ... */ } // конструктор экземпляра блока
    }
}
```


<a name="destruct"></a>

### Деструктор экземпляра блока ###

Моментом удаления блока является момент уничтожения всех ссылок на
JS-объект блока, после чего он может быть удален из памяти браузера
сборщиком мусора.

Триггер на удаление модификатора `js` (установку в пустое значение
`''`) выполняется перед удалением блока. Такой триггер можно считать
**деструктором экземпляра блока**.

```js
onSetMod: {
    'js': {
        '': function() { /* ... */ } // деструктор экземпляра блока
    }
}
```


<a name="init-wave"></a>

### Волны инициализации ###

Инициализация экземпляров блоков, присутствующих на странице, не
обязательно происходит одновременно. Блоки могут добавляться в ходе
работы (например, за счет динамической генерации HTML на основе
данных, пришедших с сервера) или инициализироваться только по запросу.
Инициализация очередной группы блоков называется **волной
инициализации**.

Новая волна инициализации создается в следующих случаях:

* [Автоматическая инициализация всех блоков в документе по событию `domReady`](#init-auto);
* [Инициализация блока по событию на DOM-узле](#init-live) (ленивая инициализация);
* [Явный вызов инициализации блоков на указанном фрагменте DOM-дерева](#init-ajax).


<a name="init-auto"></a>

### Автоматическая инициализация ###

Фреймворк *i-bem.js* позволяет автоматически инициализировать все блоки,
имеющие DOM-представление (привязанные к DOM-элементам на странице) в
момент наступления DOM-события `domReady`. Для этого необходимо
задекларировать на странице блок `i-bem` с модификатором `init` в
значении `auto`. Пример файла `.deps.js`:

```js
({
    shouldDeps: [
        {
            block: 'i-bem',
            elem: 'dom',
            mods: { 'init': 'auto' }
        }
    ]
})
```

При автоматической инициализации в памяти браузера будут созданы
JS-объекты для всех DOM-узлов, в атрибуте `class` которых указан
`i-bem`. Инициализация выполняется функцией `init` модуля
[`i-bem__dom`][].

<a name="init-live"></a>

### Инициализация по событию (ленивая инициализация) ###

Автоматическая инициализация всех блоков в момент загрузки страницы
может быть нежелательной, так как при большом количестве экземпляров
блоков на странице увеличивается время загрузки и объем затраченной
памяти браузера.

В этом случае имеет смысл инициализировать JS-объекты только в тот
момент, когда их функциональность потребуется пользователю, например,
по клику на блоке. Такая инициализация называется **ленивой** или
**live-инициализацией**.

Для описания условий ленивой инициализации зарезервировано свойство
`live` в разделе статических методов декларации блока. Свойство `live`
может принимать два типа значений:

* `Boolean`.<br/> Если `live` установлено в значение `true`, экземпляры
  блоков данного класса будут инициализированы только при попытке
  получить соответствующий экземпляр. Подробнее
  см. раздел [Взаимодействие блоков](#ibc).

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // этот код будет выполняться
                                                   // при первом обращении к экземпляру блока
            }
        }
    },
    { live: 'true' } // статические методы и свойства
);

provide(DOM);

});
```

* `Function`.<br/> Функция, указанная в качестве значения `live`:

    * Выполняется один раз — при попытке инициализации **первого
    экземпляра** блока заданного класса.
    * Если функция возвращает значение `false`, экземпляры блоков
      будут инициализироваться [автоматически](#init-auto).

С помощью этой функции можно организовать инициализацию экземпляров
блока по наступлению DOM-событий на DOM-узле блока и вложенных элементах
или BEM-событий на вложенных блоках. Для этого в коде
функции следует выполнить подписку на
[делегированные события](#delegated-events).


**Пример**: Экземпляры блока `my-block` будут инициализироваться по
  DOM-событию `click` на DOM-узле блока. По каждому DOM-событию
  `click` будет вызываться метод экземпляра блока `_onClick`:

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // выполняется при первом DOM-событии 'click'
            }
        },

        _onClick: function() { /* ... */ } // выполняется при каждом DOM-событии 'click'
    },
    {
        live: function() {
            this.liveBindTo('click', function() {
                this._onClick(); // в момент клика будет создан экземпляр блока и вызван его метод _onClick
            });
        }
    }
);

provide(DOM);

});
```

Если необходимо воспользоваться делегированными событиями в блоке, но
инициализацию блока нельзя отложить (экземпляры блока должны быть
инициализированы немедленно после загрузки страницы), следует вернуть
значение `false`:


```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // будет выполнена по наступлении domReady
            }
        },

        _onClick: function() { /* ... */ } // будет выполняться каждый
                                           // раз при наступлении DOM-события 'click'
    },
    {
        live: function() {
            this.liveBindTo('click', function() { this._onClick() });
            return false; // экземпляры блоков будут инициализированы автоматически
        }
    }
);

provide(DOM);

});
```

Полный список хелперов для подписки на делегированные события
приведен исходном коде модуля [`i-bem__dom`][].

-------------------------------------------------------------------------------

**NB** Свойство `live` задает ленивую инициализацию для *всех
  экземпляров* соответствующего блока, так как технически относится к
  статическим методам класса блока. Поэтому даже если свойство `live`
  задекларировано для блока с определенным значением модификатора, оно
  будет применено ко всем блокам данного класса вне зависимости от
  модификаторов.

-------------------------------------------------------------------------------


<a name="init-ajax"></a>

### Инициализация и удаление блоков на фрагменте DOM-дерева ###

Процедура инициализации или уничтожения JS-объектов может быть вызвана
явно для указанного фрагмента DOM-дерева. Часто такая необходимость
возникает при разработке AJAX-интерфейсов, когда нужно [динамически
встроить в страницу новые экземпляры блоков](#dynamic) либо обновить существующие.

В *i-bem.js* следующие функции выполняют динамическую инициализацию блоков:

* Инициализация/уничтожение блоков на указанном фрагменте DOM-дерева
  (`init`, `destruct`);
* Добавление/замена фрагмента DOM-дерева с одновременной
  инициализацией блоков на обновленном фрагменте (`update`, `replace`,
  `append`, `prepend`, `before`, `after`).


<a name="init-bem"></a>

### Инициализация и удаление блоков без DOM-представления ###

Чтобы создать JS-объект для блока, не имеющего DOM-представления (не
привязанного к HTML-элементу), необходимо вызвать метод `create`,
который вернет экземпляр блока указанного класса.

**Пример**: В момент инициализации экземпляра блока с DOM-представлением
  `container` создается экземпляр блока без DOM-представления `router`. Экземпляр блока
  `container` затем будет обращаться к созданному им экземпляру блока
  `router` при вызове метода `onRequest`:

```js
modules.define('i-bem__dom', 'i-bem', function(provide, BEM, DOM) {

DOM.decl('container', {
    onSetMod: {
        'js': {
            'inited': function() {
                this._router = BEM.create('router'); // создание экземпляра блока router
            }
        }
    },

    onRequest: function() {
        this._router.route(/* ... */) // вызов метода экземпляра блока router
    }
});

provide(DOM);

});
```

**Пример**: Блок без DOM-представления реализован в виде простого
  [ymaps-модуля][ymaps], без использования модуля `i-bem`. Такой блок
  используется как обычный ymaps-модуль (нет необходимости создавать
  экземпляр блока):

```js
modules.define('i-bem__dom', 'router', function(provide, DOM, router) {

DOM.decl('container', {
    onRequest: function() {
        router.route(/* ... */); // вызов метода блока router
    }
});

provide(DOM, router);

});
```

**Удаление** экземпляров блоков без DOM-представления не может быть
выполнено автоматически и является ответственностью
разработчика. Блоки без DOM-представления представляют собой обычные
JS-объекты и удаляются в момент удаления всех ссылок на объект блока.


**Пример**: При удалении экземпляра блока `container` удаляется созданный им в
процессе работы экземпляр блока без DOM-представления `router`.

```js
modules.define('i-bem__dom', 'i-bem', function(provide, BEM, DOM) {

DOM.decl('container', {
    onSetMod : {
        'js' : {
            '' : function() {
                delete this._router; // удаление экземпляра блока router
            }
        }
    }
});

provide(DOM);

});
```

# Взаимодействие блоков #

БЭМ-методология предполагает, что блоки должны разрабатываться таким
образом, чтобы по возможности исключить зависимость состояний одних
блоков от других. Однако на практике идеал полной независимости блоков
недостижим.

Взаимодействие блоков может быть реализовано двумя способами:

* С помощью подписки на [BEM-события](#bem-events) других экземпляров
  блоков или подписки на [делегированные BEM-события](#bem-events-delegated).
* С помощью непосредственного вызова методов других экземпляров
  блоков или статических методов класса другого блока.

-------------------------------------------------------------------------------

**NB** Не используйте [DOM-события](#dom-events) для
  организации взаимодействия между блоками. DOM-события предназначены
  только для реализации внутренних процедур блока.

-------------------------------------------------------------------------------

Для реализации взаимодействия блоков *i-bem.js* предоставляет API для
доступа к JS-объектам экземпляров блоков и к JS-объектам классов блоков.

## Поиск экземпляров блоков в DOM-дереве ##

Обращение к другому блоку в *i-bem.js* выполняется из текущего блока,
размещенного на определенном узле DOM-дерева. Поиск других блоков в
DOM-дереве может вестись по трем направлениям (осям) относительно
DOM-узла текущего блока:

* «Внутри блока» — на DOM-узлах, вложенных в DOM-узел текущего блока.
* «Снаружи блока» — на DOM-узлах, потомком которых является DOM-узел
  текущего блока. Необходимость в таком поиске может свидетельствовать
  о неудачной архитектуре интерфейса.
* «На себе» — на том же DOM-узле, на котором размещен текущий
  блок. Это актуально в случае [размещения нескольких JS-блоков на
  одном DOM-узле](#mixes) (микс).

**Пример**: При переключении модификатора `disabled` экземпляр блока
  `attach` находит вложенный в него блок `button` и переключает его
  модификатор `disabled` в то же значение, которое получил сам:

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('attach', {
    onSetMod: {
        'disabled': function(modName, modVal) {
            this.findBlockInside('button').setMod(modName, modVal);
        }
    }
});

provide(DOM);

});
```

Полный список методов для поиска блоков блоков приведен
в исходном коде модуля [`i-bem__dom`][].


-------------------------------------------------------------------------------

**NB** Не используйте jQuery-селекторы для поиска блоков и элементов.
*i-bem.js* предоставляет высокоуровневое API для доступа к DOM-узлам
блоков и элементов. Обращение к DOM-дереву в обход этого API менее
устойчивым к изменениям БЭМ-библиотек и может привести к возникновению
сложно обнаруживаемых ошибок.

-------------------------------------------------------------------------------

## Доступ к экземплярам блоков без DOM-представления ##

При создании экземпляра блока без DOM-представления необходимо
позаботиться о том, чтобы ссылка на этот экземпляр была доступна
блокам, которым потребуется взаимодействовать с ним. Подробности и
пример см. в разделе [Инициализация и удаление блоков без DOM-представления](#init-bem).


## Доступ к классам блоков ##

JS-компоненты, соответствующие всем блокам («классы» блоков), хранятся
в структуре данных `BEM.blocks`. Классы блоков,
[не привязанных к DOM-дереву](#i-blocks), также размещены в этой
структуре данных. При необходимости доступа к таким блокам следует
использовать конструкцию:
```js
BEM.blocks['name']
```
где `name` — имя блока.

Доступ к классам блоков необходим для решения двух основных задач:

* [Делегирование БЭМ-событий](#bem-events-delegated).
* Вызов статического метода класса.

**Пример**: Вызов статического метода `close` блока `popup` — закрыть
  все попапы на странице:

```js
DOM.decl('switcher', {
    onSetMod : {
        'popup' : {
            'disabled' : function() {
                BEM.blocks['popup'].close();
            }
        }
    }
});
```

<a name="docs"></a>

# Что дальше? #

Общую информацию о БЭМ-методологии, инструментарии, новостях в мире
БЭМ можно найти на сайте [bem.info](http://ru.bem.info/).

Полную информацию обо всех методах API *i-bem.js* можно найти в
исходном коде, который сопровождается структурированными комментариями
в формате JSDoc:

* [`i-bem`][];
* [`i-bem__dom`][].

Задать вопрос опытным пользователям и разработчикам *i-bem.js* и
следить за текущими обсуждениями можно в социальных сетях:

* [Клуб в Я.ру](http://clubs.ya.ru/bem/);
* [Группа в Facebook](http://www.facebook.com/#!/groups/209713935765634/);
* [Twitter](https://twitter.com/bem_ru).

Прочитать о принципах работы *i-bem.js* в другом изложении, найти
образцы его применения и пошаговые инструкции на примере простых
проектов можно в статьях:

* [JavaScript по БЭМ: основные понятия](http://ru.bem.info/articles/bem-js-main-terms/);
* [Tutorial on JavaScript in BEM terms](https://github.com/varya/bem-js-tutorial);
* [Попробуй БЭМ на вкус!](http://habrahabr.ru/post/162385/);
* [БЭМ-приложение на Leaflet и API 2GIS](http://ru.bem.info/articles/firm-card-story/).


-------------------------------------------------------------------------------

**NB** Обратите внимание, что в перечисленных статьях может
использоваться устаревший синтаксис, не соответствующий текущей версии
*i-bem.js*, включенной в bem-core.

-------------------------------------------------------------------------------


[ymaps]: https://github.com/ymaps/modules

[bem-tools]: http://ru.bem.info/tools/bem/

[`i-bem`]: https://github.com/bem/bem-core/blob/v1/common.blocks/i-bem/i-bem.vanilla.js

[`i-bem__dom`]: https://github.com/bem/bem-core/blob/v1/common.blocks/i-bem/__dom/i-bem__dom.js
