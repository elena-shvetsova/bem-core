# *i-bem.js*: User Guide #

# Overview #

## *i-bem.js*: Javascript framework for BEM ##

*i-bem.js* is a JavaScript framework for web-development using [BEM methodology](http://en.bem.info/method/). With
*i-bem.js* one can:

 * develop web interface in terms of blocks, elements, modifiers;
 * describe a block's behaviour in a declarative style as a set of states;
 * integrate easily JavaScript code with CSS in BEM style and with BEMHTML templates;
 * flexibly redefine behaviour of library blocks.

*i-bem.js* is not suitable for:

 * replacing the general purpose framework, such as jQuery.


**Brief content of the document**:

* [Framework overview](#intro): relation to the BEM domain area, a brief description of the modular structure of the framework, project stub, the build tools, that was written using *i-bem.js*. 

* [Binding JS blocks to HTML](#html) — syntax to define JS blocks on HTML page, how HTML elements and JS blocks relate.

* [Block declarations](#decl) — JS blocks definition syntax.

* [Working with DOM tree](#dom) — API for working with DOM representation of blocks:
  elements, DOM tree dynamic modification (AJAX).

* [Events](#events) —  *i-bem.js* event model: DOM events,
  BEM events, event delegation.

* [Block states](#states) — modifiers, triggers for state change  (modifiers setting), initialization of block instances.

* [Blocks interaction ](#ibc) — API for block-to-block communication.

* [What's next?](#docs) — links to documentation and additional materials.
* 
##BEM methodology and JavaScript##
As far as BEM methodology is concerned, web interface is built of independent **blocks** (in which **elements** are allocated). Both blocks and elements may have states, described by **modifiers**.

Web interface work is provided by multiple  **technologies**
(HTML, CSS, JS...). In BEM methodology all the web interface implementation is arranged into blocks. The block description in this case consists of several files — one per each technology:

 * `my-block.css` — describes block's style;
 * `my-block.bemhtml` — templates for generating block's HTML view;
 * `my-block.js` — describes block's **dynamic behaviour** in browser.

*i-bem.js* framework allows to decompound client JavaScript into BEM components:

 + **Block** is a JS piece that describes the way the same-type interface components work.
 For example, all buttons may be implemented as a `button` block. Then, in accordance with the BEM methodology, `button.css` determines the appearance of the buttons, `button.js` — determines the way they behave.
 + Each page can contain more than one **instance** of a block (for example, buttons on a page). Each block instance corresponds to the JS object that is created dynamically in the browser's memory and stores the state of the given instance. JS object has a link to the DOM node that is bound to the block instance. 
 + **Elements** are DOM nodes nested in DOM node block with `class` attribute, indicating their role in BEM domain area (the names of blocks and elements). Elements of a block are available through the [JS-API](#elem-api) of the instance of the block.
 + **Modifiers** store information on a block state and its elements. Modifier state is written in `class` attribute of block's DOM nodes and elements. ! исправить
   Modifiers are operated via [JS-API](#mods-api) of the block instance.



##How to use i-bem.js##

*i-bem.js* framework is included in [bem-core library](http://github.com/bem/bem-core/) .

Implementation consists of two modules:

* [`i-bem`][] module.<br/> 
  Basic implementation of i-bem JS block, which all the blocks in *i-bem.js* inherit from. i-bem block is written to be used in any JS-environment, both on client and server sides (for example, in Node.js).
* [`i-bem__dom`][] module.<br/> Basic implementation of a block bound to DOM node. Is intended for using on the client side, based on browsers' work with DOM. Depends on jQuery. 

Dependencies:

 * jQuery (only for `i-bem__dom` module). When using the bem-core library, the separate including jQuery is not required / as it will be installed automatically.
 * [ymaps/modules][ymaps] modular system. When using [bem-tools][] along with `.browser.js` technology (and derrivatives based on it), this dependancy is resolved automatically.

One can use *i-bem.js* as a part of full stack of BEM-tools. In this case it is convinient to create a project based on [project-stub](http://github.com/bem/project-stub/) template repository, which automatically installs the dependent libraries.

If one doesn't plan on using other technologies of BEM platform, one can copy the bem-core library code to the current project.


##Build##

According to BEM methodology web development is modular: each block is programmed separately. The final sourcecode of web pages is formed from the separate blocks code by use of **build** proceedures.


In a file system it is convenient to represent a block as a catalog / catalog -> derictory [проверить у Ширшина), and the implementation of a block in each of the technologies as a separate file:

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

The code of the used blocks on each web page can be built in bundles: 

    desktop.bundles/
        index/
            index.html
            index.css
            index.js
            ...

There are two tools for building code bundles from separate blocks definition:

* [bem-tools](http://github.com/bem/bem-tools/);
* [enb](https://github.com/enb-make/enb) along with [enb-modules](https://github.com/enb-make/enb-modules).


##Why i-bem.js is called that way##

According to BEM methodology, basic JS library of BEM platform was originally developed as a special helper block. 
This approach allows to work with base libraries in the same way as with common blocks. Particularly, it allows to structure code in terms of elements and modifiers, and flexibly set up the library behaviour at different block levels.

It was common for BEM to give names to helper blocks with `i-` prefixes. So, the name `*i-bem.js*` read as "implementation of the `i-bem` block in `js` technology".


#Binding JS blocks to HTML#

JavaScript components in *i-bem.js* are used to make HTML elements of a page dynamic. A typical task for JS block is to bind certain events handling to the specific HTML fragment.

In developing a web interface with *i-bem.js* framework there is a primary structure — the HTML document tree. 
In the HTML tree there are nodes, bound with JS blocks — interactive interface components . The point of a JS block binding is an HTML element.  The name of the block is indicated in its `class` attribute , [block parameters](#html-syntax) are indicated in `data-bem` attribute.

When loading а page, the browser runs [blocks initialization](#init). During the initialization process block instances are generated – JS objects for all the blocks, mentioned in HTML elements on the page. 
JS object, bound to the HTML element handles its [DOM events](#dom-events) and stores the state of the given instance of a block.

*i-bem.js* allows to implement a JS component, not bound to HTML — [block without DOM representation ](#i-blocks). Such block gives API, similar to common JS blocks. 

This approach of binding JavaScript components to HTML has the following advantages:

 * graceful interface degradation on the client-side with JavaScript disabled; 
 * _progressive rendering_ — the opportunity to start drawing interface components before the download of all data pages is over ( eg images ).

<a name="html-syntax"></a>

##Blocks binding syntax##

To bind a block to HTML element (for example, `<div>...</div>`), one should:

 * **Mark a block in HTML tree**.<br/>
 Include the block name in the list of classes of HTML-element (`class` attribute).

```HTML
<div class="my-block">...</div>
```

 * **Initialize the block instance**.<br/>
 Include the `i-bem` class in the list of classes of HTML element. The presence of this class allows the framework to initialize the block.

```HTML
<div class="my-block i-bem">...</div>
```

 * **Pass parameters to the block instance**.<br/> Write parameters of a block in `data-bem` attribute. Write the block parameters in JSON format so hash parameters correspond to the block name. Parameters are transferred to the block instance at the point of initialization.                 
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

###One HTML element — several JS blocks###

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

###One JS block on several HTML elements###

This design allows you to transparently implement blocks consisting of several components, which state must be coordinated. 
For example, the "tab" widget, whereon clicking on the header of the tab (one HTML element) changes the contents of the tab (the other HTML element). For example, the "tabbed panel" widget, whereon clicking on the tab (one HTML- element) changes the contents of the panel (the other HTML element).
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

As a result, when initializing the blocks a single JS object is created, 
 which [`{jQuery} domElem`](#domElem) field contains links to both DOM nodes.

`id` Modifier is only used *during initialization* of a block instance. `id` value must be unique within one block instances inside of one [initialization wave](#init-wave).


<a name="i-blocks"></a>

###Blocks without DOM representation###

Infrastructure code that solves common interface tasks: communication with the backend, general computing, etc. When working with *i-bem.js* code can take the form of block, like all the rest of JS code.
In order not to bind these blocks to HTML tree manually, *i-bem.js*  provides the opportunity to create blocks without DOM representation.

Blocks without DOM representation:

 * They are not written in HTML code of a page.
 * Declared as [module extension`i-bem`](#bem-decl), not the `i-bem__dom`, as in the case of blocks with DOM representation.
 * They shoul be [initialized explicitly](#init-bem).


##Parameters transfer syntax##

Block parameter is an arbitrary JavaScript object, that is transferred to the block at the point of initialization.
Parameters allow to modify the behaviour of the block instance bound to the given HTML element.

The value of `data-bem` attribute contain the parameters of *all the JS blocks in this node*.
Parameters are transferred as a hash in JSON format:

 + key — `{String}` block name;
 + value — `{Object}` the parameters of the given block. If this block instance has no parameters, declare empty hash `{}`.

This parameters format proceeds from the following:

 * Indication of the block name in  parameters  allows to avoid having to parse value of the `class` attribute, which simplifies and accelerates blocks initialization.
 *This same solution allows to place multiple blocks in one HTML element without the need to multiply the block attributes.

Value of `data-bem` attribute  should contain valid JSON.

#Block declaration#

JS implementation of a block describes the functionality of a certain class of web interface components. In a specific interface, each block can have several instances. 
Each instance of a block implements functionality of the whole class and has its own state, independent of the others.

In terms of object oriented programming paradigm:

 * block — класс;
 * block instance — class instance.

In terms of OOP, the entire functionality of the block is implemented in modules in class (= block)_methods_.
Block methods are subdivided into: 

 * block instance methods;
 * static methods.

Block code in *i-bem.js* is commonly known as **declaration**, to point out used in BEM terminology declarative programming style. 
In accordance with the declarative style the block behaviour is programmed as the following statements _set of conditions — block reaction_.   


## Declaration syntax ##

In order to declare a new JS block **with DOM representation**
(bound to HTML element), one has got to modify? `i-bem__dom` [ymaps][] module.

To declare blocks use `decl`method, that can have three parameters:

1. Block name `{String}` or [block description ](#decl-selector) `{Object}`.
2. Block instance methods — `{Object}`.
3. Static methods — `{Object}`.

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl(/* block name or block definition */,
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

**NB** In terms of [ymaps][] modular system, declarations of multiple blocks represent redefining of one and the same module
  `i-bem__dom`. And yet in terms of *i-bem.js* this is the way *different instances * for each block.

-------------------------------------------------------------------------------

<a name="bem-decl"></a>

Blocks, that do not have DOM representation are declared as extension of the `i-bem` [ymaps][]-module .
For declaring use `decl` method, receiving the same parameters, as`decl` method of `i-bem__dom` module:
 
```js
modules.define('i-bem', function(provide, BEM) {

BEM.decl(/* block name or block definition */,
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
BEM blocks (states, expressed by modifiers, BEM events etc.). Without using BEM stack infrastructure code can be designed as  [ymaps][] module. For example:

```js
modules.define('router', function(provide) {

provide({
    route : function() { /* ... */ }
});

});
```

-------------------------------------------------------------------------------

<a name="decl-selector"></a>


## Block definition in a declaration ##

The first parameter of the `decl` method is a block definition. The methods announced in the declaration will be applied.
The block definition must contain the block name and may additionally contain:

* limiting the scope of the declaration by the certain modification of the block;
* list of parent blocks, methods of which would inherit the given block. Блок отнаследует методы родительских блоков из данного списка.

Block definition may be given in one of two forms:

1. Block name - string.<br/>
   Declared methods will be applied in the block instances, regardless of their states (modifiers) 
For example: Declaration of methods for the `button` block.

    ```js
DOM.decl('button',
    { /* instance methods */ },
    { /* static methods  */ }
);
    ```

2. Block definition  - hash.<br/> 
Contains modifier name (and values). Declared methods will be applied only  to the block instances, that have the given modifier (with the specified value).
For example: declaring method for the `button` block with the `type` modifier with value
   `link` (describes behaviour of pseudo buttons):

    ```js
DOM.decl({ block: 'button', modName: 'type', modVal: 'link' },
    { /* instance methods */ },
    { /* static methods */ }
);
    ```

-------------------------------------------------------------------------------

**NB** If static methods are defined in a declaration for the block with specific modifiers, they will be available in all instances of this block *regardless of the modifiers values*.    
Modifiers are the properties of block instances. Static methods belong to the class of the block, and so can not consider the limitations of the modifier.

-------------------------------------------------------------------------------


## Сontext ##

**Block instance methods** are implemented in the JS object context of a block instance.  Accordingly, the keyword `this` in instance methods of the block refers to JS object of a **block instance**. 

**Static methods** are applied in the context of JS object, that refers to the class of the block.
Accordingly, `this` keyword  in static methods of the block refers to **block class**, not to an instance.

Context has the reserved fields:

 + `this.__self`: Refers to the static methods of the class, which the instance belongs to. It is defined in the instance methods of a block. It does not make sense for static methods and is not defined.

    For example: Static method `staticMethod` calling in `onEvent` method of `my-block` block instance. 

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

 + `this.__base`: Refers to the implementation of the method in the base class, which it is inhereted from. 
    Allows to make super call. It is defined in the block instance methods and in static methods of the block.

    For example: calling (and modification) of `_onClick` method of parent class (the basic method implementation in the `button` class).

```js
DOM.decl({ block: 'my-button', baseBlock: 'button' }, {
    _onClick: function() {
        this.__base();
        this.doMore();
    }
);
```

-------------------------------------------------------------------------------

**NB** In blocks development using  *i-bem.js*  with block private methods, not intended for external use, it is common to give names starting with an underscore. For example, `_onClick`.

-------------------------------------------------------------------------------


#Working with DOM tree#

<a name="domElem"></a>

##DOM node of a block instance##

All the block instances, bound to the DOM tree, contain jQuery object
in the `{jQuery} this.domElem` field, referring to one or more DOM nodes, bound to the given block instance.

<a name="elem-api"></a>

##Elements##

BEM elements of blocks  are represented in *i-bem.js* as DOM nodes, nested into the DOM node of the block instance.
To access DOM nodes of elements and work with their modifiers use API, provided by the block instance.

The block instance provides two methods to access the elements of a given instance:

* Caching ccess`elem(elems, [modName], [modVal])`. There is no need to store as a variable the element obtained this way.

```js
DOM.decl('b-link', {
    setInnerText: function() {
        this.elem('inner').text('Link text');
        /* ... */
        this.elem('inner').text('Another text');
    }
);
```

* Non caching access `findElem(elems, [modName], [modVal])`.

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

In the process of [dynamic adding and removing block elements](#dynamic) it can  be necessary to drop cache of elements. This is what the `dropElemCache('elements')` method is used for. 
The space-separated list of names is indicated as a parameter for those elements, cache reset is needed for:  

```js
DOM.decl('attach', {
    clear: function() {
        DOM.destruct(this.elem('control'));
        DOM.destruct(this.elem('file'));
        return this.dropElemCache('control file');
    }
});
```


Complete API description for working with elements is contained in the source code of the [`i-bem__dom`][] module.

<a name="dynamic"></a>

##Dynamic update of blocks and elements in the DOM tree##

In modern interfaces one often needs to create new pieces of DOM tree and replace the old ones in the process of work (AJAX). 
There are functions provided for adding and replacing fragments DOM tree in *i-bem.js*:
* append a DOM fragment:

* `append` — into to the end of specified context;
* `prepend` — into to the end of specified context;
* `before` — before the specified context;
* `after` — after the specified context;
* replace a DOM fragment:

* `update` — inside the specified context;
* `replace` — to replace the specified context with a new DOM fragment.

All functions automatically run [initialization in the updated DOM tree fragment](#init-ajax).  

To simplify the creation of BEM entities in a renewable fragments of DOM-tree, it is possible to use the template engine
[BEMHTML](http://ru.bem.info/articles/bemhtml-reference/), and define it as a [ymaps][] module. 
BEM in [BEMJSON](http://ru.bem.info/articles/bemhtml-reference#bemjson) format.
BEM entities are defined in BEMJSON format directly in the block code. BEMHTML.apply function generates HTML elements by BEMJSON declaration according to the BEM naming rules.

**For example**: `_updateFileElem` method of the `attach` block removes the `file` element, if it existed, and generates a new element by using `BEMHTML.apply`: 

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

* **DOM event** is JavaScript event, triggered in DOM node, bound to the block. It reflects a user's interaction with interface (clickштп, mouse pointing, text input, etc.). 
  DOM event is usually handled  by the block instance, in the DOM node of which it was triggered.
* **BEM event** is the custom event, generated by block. It allows to organize API for [blocks](#ibc). BEM events usually handle the block instance, that tracks the states of other blocks generating events. 

When planning interface architecture, consider that DOM events should only be used in *private* block methods. 
Use BEM events for the block interaction with other blocks.


-------------------------------------------------------------------------------

**NB** Working with DOM events is fully realized by means of jQuery framework.  

-------------------------------------------------------------------------------


<a name="delegated-events"></a>

## Event delegation ##

BEM and DOM events handling can be **delegated** to a container
(either the entire document or the specific DOM node). In this case container listens for any event in the child nodes, even if the they do not exist yet. 
without adding or removing event listeners for added or deleted items
For example, menu block can contain nested blocks — menu items (or elements, it depends on the specific block implementation). 
It makes sense to delegate click event handling on menu items to the menu block itself. 
First of all, this allows to save the resources of adding events listeners (it's cheaper to add one event listener to a container, than many events listeners to the elements). 
Secondly, it makes it possible to change the menu contents: add and remove menu items without
adding event listeners for the added items or remove event listeners from the deleted items.

* [**DOM event delegation**](#dom-events-delegated) can be used for handling DOM events, triggered in the DOM node of the block instance or in the DOM nodes of its elements.
DOM events delegation can be performed either for all the block instances within the document, or for the block instances within the specified fragment of the HTML tree.
  

  `window.document` always performs as a container to which DOM events handling is delegated.


* [**BEM events delegation**](#bem-events-delegated) should be used to handle events in *block instances*, contained in the specific DOM node.

    **arbitrary DOM node** can perform as a container, that BEM events handling is delegated to. The entire (`window.document`) document performs as a container by default. 
    Often a block handles BEM events of the nested blocks, then DOM node of the handler block [`this.domElem`](#domElem) should be passed.


The complete list of helpers for adding event listeners to delegated events can be found in the source code of the [`i-bem__dom`][] module.


## DOM events ##

To add event listeners to DOM nodes, bound to a block or an element, use instance block method `bindTo([elem], event, handler)`.

**For example**: At the point of [block instance initialization](#init)
`my-block` event listener to `click` event is added, when the event is triggered, the block sets [modifier](#modifier) `size` with value `big`.

```js
DOM.decl('my-block', {
    onSetMod : {
        'js' : {
            'inited': function() {
                this.bindTo('click', function(e) {
                    var domElem = $(e.currentTarget); // DOM element, for which the event listener is registered
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

**Removing of event listener** to DOM events will be run automatically when the block instance is removed. If it is necessary to remove the event listener manually while the block is being used, use `unbindFrom([elem], event, handler)` method. 


<a name="dom-events-delegated"></a>

### DOM event delegation ###

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
A DOM event in the block instance, to which a delegated event listener is added, can be such a moment
, or referring to the block instance [из другого блока/from another block](#ibc).
If the block initialization can not be deferred ([automatic initialization](#init-auto) is required), следует вернуть `false` в
результате выполнения функции в значении `live` property.  

-------------------------------------------------------------------------------

**NB**  Handler function is executed in the context of the nearest block of this type in the path of DOM event (from the bottom up the DOM tree).

-------------------------------------------------------------------------------

**Removal of the delegated event listener** never is performed automatically. To remove the delegated event listener use `liveUnbindFrom([elem], event, [handler])` method. 


### DOM event object ###

jQuery object is transferred as a parameter, describing DOM event — [`{jQuery.Event}`](http://api.jquery.com/category/events/event-object/) to the handler function.

If a DOM event was generated manually, all the parameters, transferred
to the `trigger` function at the point of event creation при создании события, will be transferred to the handler function in the same order after the event object после объекта события.



<a name="bem-events"></a>

## BEM events ##

Unlike DOM events, BEM events are generated not in DOM elements, but in **block instances**. Block elements can not generate BEM events.

To generate BEM event, `emit(event)` block instance method  is used.

**For example**: When a user clicks on DOM element of the `submit`button
(DOM event `click` is emitted),`_onClick()` method of `submit`block instance is executed, in which **BEM event** `click` is generated
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

* `{jQuery} [ctx]` — DOM node, that contain BEM-events /в пределах которого отслеживаются
(container). If it is not specified, the entire document is used as a container.
* `{String} event` — BEM event name.
* `{Object} [data]` — Arbitrary data transferrred to the function handler.
* `{Function} handler` — Function event handler/ Функция-обработчик события.
* `{Object} [handlerCtx]` — Function of event handler context.
  Usually it is the block instance, that added a BEM event listener, not the one, that emittted the event. Обычно в качестве контекста должен выступать тот экземпляр блока, который подписывается на BEM-событие, а не тот, в котором BEM-событие произошло.


**For example**: At the point of initialization of the `menu` block instances, the `click` BEM event listener is added to all the links (of the `link` block instances) within the DOM node, that menu (`this.domElem`) is bound to. 
При инициализации экземпляров блока `menu` выполняется подписка на BEM-событие `click` всех ссылок (экземпляров блока `link`) в пределах DOM-узла, к которому привязано меню
  (`this.domElem`). The block instance, that will handle the event (`this`) is transferred as a context of a handler function. В качестве контекста функции-обработчика передается экземпляр блока, в котором событие будет обрабатываться (`this`). When [removing the block instances/уничтожении экземпляров блока](#destruct) `menu`

```js
DOM.decl('menu', {
    onSetMod : {
        'js' : {
            'inited' : function() {
                DOM.blocks['link'].on( // adding BEM event listener
                    this.domElem, // container — DOM node of the menu block instance
                    'click', // BEM event
                    this._onLinkClick, // handler
                    this); // контекст обработчика context of a handler — instance of the menu block
            },

            '' : function() {
                DOM.blocks['link'].un( // removing BEM event listener
                    this.domElem,
                    'click',
                    this._onLinkClick,
                    this);
            }
        }
    },

    _onLinkClick : function(e) {
        var clickedLink = e.target; // 'link' block instance, that emitted 'click' BEM event
    }
});
```

-------------------------------------------------------------------------------

**NB**  If the `[handlerCtx]`parameter of `on` method is not specified,
  then the context for handler function is the block, that *emitted* BEM event.

-------------------------------------------------------------------------------

**Удаление подписки/Event listeners removal** from BEM events can never be performed automatically. One should always remove event listener manually using the `un([ctx], event, [handler],
  [handlerCtx])` static block method. 


Complete API description for working with BEM events is contained in source code of [`i-bem`][] and [`i-bem__dom`][] modules.


<a name="api"></a>

## BEM event  object ##

An object, describing BEM event is transferred to a handler function as a parameter. BEM event object `events.Event` определен/ defined in the [ymaps][] module in
[`events`](https://github.com/bem/bem-core/blob/v1/common.blocks/events/events.vanilla.js)
of the bem-core library. Contains the following fields:

* `target` — The block instance, emitted the BEM event.
* `data` — Arbitrary additional data. Is transferred as a parameter of/ в качестве
  параметра `data` at the point of adding a BEM event listener or creating или при создании BEM event by a block.
* `result` — Last value, возвращенное обработчиком/ returned by handler of the given event. Similar to [jQuery.Event.result](http://api.jquery.com/event.result/).
* `type` — Event type. Similar to
[jQuery.Event.type](http://api.jquery.com/event.type/).


<a name="ibc"></a>

# Block states #

Проектируя When creating a dynamic block in BEM style, it is great to visualize всю логику/all the changes that take place in it, as a set of block **states**. Then block behaviour is defined by **triggers** — the callback functions, which run/которые выполняются when a block changes one state to another/ при переходе блока из одного состояния в другое.

This approach allows to wtite code of a block in a declarative style as a set of assertions /как набор утверждений вида:

* State descriptions — actions taken to set the certain state. 

<a name="modifiers"></a>

## Modifiers ##

In BEM methodology, the state of block and its elements are described with **modifiers**.

* Modifier — is **name** and **value**. For example, `size`: `m`.

* **Простой модификатор/ Simple modifier**. Частный случай/ specific instance, when a block either has a modifier or doesn't have one/ когда модификатор либо присутствует у блока, либо отсутствует. 
  For example, `disabled`. ??? In *i-bem.js* are implemented as modifiers with zero value. For example: `disabled`: `true`. При выставлении модификатора/ When setting a modifier with unspecified value *i-bem.js* will assign `true` value  automatically.

* Each block can have one or several modifiers.

* A block can go without any modifiers/ Блок может не иметь модификаторов.

In *i-bem.js* modifiers устанавливаются/ are set at the point of [block instance initialization/инициализации экземпляра блока](#init) (if modifiers and their values are specified in the `class` attribute of the certain HTML element).

-------------------------------------------------------------------------------

**NB** During initialization block with modifiers triggers for setting given modifiers are not /триггеры на установку данных модификаторов *не выполняются*, as the block instance in this case receives the initial state, but does not change it.

-------------------------------------------------------------------------------

Modifiers can be added, removed, can change values/ Модификаторы могут добавляться, удаляться и менять значения:

* В ходе выполнения кода блока/ In the process of implementing block code (for instance, in response to /в качестве реакции на [DOM events](#dom-events)).
* Upon request from another block/ По запросу из другого блока. For further info please see [Blocks interaction](#ibc).

While adding, removing, and changing values of modifiers triggers are implemented.


<a name="mods-api"></a>

###Operating modifiers###

A block instance provides methods for installation, value checking
and removing modifiers of the given instance.

-------------------------------------------------------------------------------

**NB**: Modifiers can not be installed by directly changing CSS classes in the corresponding DOM node.
For changing modifiers values one should use API, предоставляемое/ provided by *i-bem.js* (see below).

-------------------------------------------------------------------------------

**For example**: The `square` block instance can change `green` and `red` values of the `color` modifier by clicking on DOM element of the block, if the `disabled` modifier is not set:

```js
DOM.decl('square', {
    onSquareClick: function(e) {
        if(!this.hasMod('disabled')) {
            this.toggleMod('color', 'green', 'red');
        }
    }
});
```

These same methods are used to operate modifiers of block elements. For that the link to the element object (not the element name) is designated as the first parameter (which is optional).

**For example**: By clicking on the `searchbox` block can set a simple modifier `clean` to its `input` element (the implied value — `true`):

```js
DOM.decl('searchbox', {
    _onClick: function() {
        this.setMod(this.elem('input'), 'clean');
    }
});
```

-------------------------------------------------------------------------------

**NB** In operating modifiers of elementsv provide a link to the **DOM node of element** as the first parameter, not the element name. Otherwise it would cause ambiguity:  that is setting the `input` *modifier* in the value `clean` or setting the  *simple modifier* `clean`to the `input` element.

-------------------------------------------------------------------------------

Complete API description for operating modifiers please find in the source code of [`i-bem`][] and [`i-bem__dom`][] modules.


## Триггеры на установку модификаторов / Triggers on modifiers setting ##

Выполнение триггеров на установку модификаторов/triggers on modifiers setting is divided into two stages:

1. **До установки модификатора/ before setting the modifier**. At this stage it is still possible to **отменить/ call off** the modifier setting. Если хотя бы один if any one or more of the triggers, implemented in this stage, 
    вернет / recalls `false`, the modifier will not be set.
2. **after setting the modifier**. Triggers, implemented at this stage, can't recall the modifier setting. 

Триггеры могут быть привязаны к следующим типам изменений значений модификаторов/ Triggers can be bound to the following value change of modifiers:

1. setting *any* modifier to *any* value;
2. setting the *specific* modifier `modName` to *any* value (which includes
   setting simple modifier to the value `true`);
3. setting the *specific* modifier `modName` to the в *specific* value `modVal`;
4. setting the modifier to value `''` (empty string), that is equivalent to
    deleting the  modifier or setting simple modifier to the value `false`).


When setting the modifier `modName` to the value `modVal` triggers in each stage (if defined) are called in the order in which they are listed in the above list of events (from general to specific).

Therefore, in defining /при определении trigger a user points /указывает:

* stage of implementation/ фазу выполнения (before or after modifier setting);
* type of event (name and chosen value of modifier и устанавливаемое значение модификатора).

### Декларация триггеров/ Trigger declaration ###

Triggers, executed /выполняемые at the modifiers setting, are described in a block declaration. For this purpose  there are the properties reserved in hash of block instance methods:

* `beforeSetMod` — triggers, called before
  **block modifiers** setting.
* `beforeElemSetMod` — triggers, called before setting
  **element modifiers**.
* `onSetMod` — triggers, called after setting
  **block modifiers**.
* `onElemSetMod` — triggers, called after setting
  **modifiers of block elements**.

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl(/* block selector */,
    {
        /* instance methods */
        beforeSetMod: { /* triggers before block modifiers setting */}
        beforeElemSetMod: { /* triggers before element modifiers setting */}
        onSetMod: { /* triggers after block modifiers setting */ }
        onElemSetMod: { /* triggers after element modifiers setting */ }
    },
    {
        /* static methods */
    }
);

provide(DOM);

});
```

Property values `beforeSetMod` and `onSetMod` — hash, binding modifiers changes to triggers / хэш, связывающий
изменения модификаторов с триггерами. The following are passed to triggers as parameters:  В качестве параметров триггерам передаются:

* modifier name;
* выставляемое / set modifier value;
* preceding (for `beforeElemSetMod`) or current (for `onElemSetMod`) modifier value.

```js
{
    'mod1': function(modName, modVal, prevModVal) { /* ... */ }, // setting mod1 to any value
    'mod2': {
        'val1': function(modName, modVal, prevModVal) { /* ... */ }, // trigger на установку/ on setting mod2 in value val1
        'val2': function(modName, modVal, prevModVal) { /* ... */ }, // trigger on setting mod2 в значение val2
        '': function(modName, modVal, prevModVal) { /* ... */ } // триггер на удаление модификатора mod2
    'mod3': {
        'true': function(modName, modVal, prevModVal) { /* ... */ }, // trigger on setting simple modifier mod3
        '': function(modName, modVal, prevModVal) { /* ... */ }, // trigger on deleting simple modifier mod3
    },
    '*': function(modName, modVal, prevModVal) { /* ... */ } // trigger on setting any simple modifier to any value
}
```

There is a shorthand notation for a trigger on setting any block modifier to any value / Для триггера на установку любого модификатора блока в любое значение существует :

```js
beforeSetMod: function(modName, modVal, prevModVal) { /* ... */ }
onSetMod: function(modName, modVal, prevModVal) { /* ... */ }
```

For properties `beforeElemSetMod` and `onElemSetMod` in value hash / в хэш значений
additional nesting level is added, задающий/ assigning **element**,
on modifiers setting of which the triggers are set. 
В качестве параметров триггеру передаются / The following are passed to triggers as parameters:

* element name;
* modifier name;
* выставляемое/ set modifier value;
* preceding (for `beforeElemSetMod`) or current (for `onElemSetMod`) modifier value.


```js
{
    'elem1': {
        'mod1': function(elem, modName, modVal, prevModVal) { /* ... */ }, // trigger on setting mod1 of element elem 1 to any value 
        'mod2': {
            'val1': function(elem, modName, modVal, prevModVal) { /* ... */ }, // trigger on setting mod2 of element elem1 to value val1
            'val2': function(elem, modName, modVal, prevModVal) { /* ... */ } // trigger on setting mod2 of element elem1 to value val2
            }
        },
    'elem2': function(elem, modName, modVal, prevModVal) { /* ... */ } // trigger on setting  on setting any modifier of element elem2 to any value
}
```

A shorthand notation for a trigger on setting any element modifier `elem1` to any value:

```js
beforeElemSetMod: { 'elem1': function(elem, modName, modVal, prevModVal) { /* ... */ } }
onElemSetMod: { 'elem1': function(elem, modName, modVal, prevModVal) { /* ... */ } }
```

### Triggers examples ###

A common tasks for triggers, called after setting the modifiers or
changing its value (property `onSetMod`) — make the necessary for the state transition changes with the DOM node of a block.

**For example**: The `input` block instance at the setting `focused` simple modifier (in value `true`) clears input /очищает поле ввода — replace text of the DOM node block with an empty string / пустой строкой текст DOM-узла блока.

```js
DOM.decl('input', {
    onSetMod : {
        'focused' : {
            'true' : function() {
                this.domElem.val(''); // clear input/очистить поле ввода
            }
        }
    }
});
```

Triggers, executed before modifier setting (property
`beforeSetMod`), are necessary for checking current state of a block instance and a possibility to recall transition to another state /и возможности отменить переход в другое состояние.

**For example**: Block instance `input` before setting simple modifier `focused` checks, whether it has a modifier`disabled`. If it does have `disabled`, `false` will be recalled after checking and the modifier `focused` will not be set.

```js
DOM.decl('input', {
    beforeSetMod : {
        'focused' : {
            'true' : function() {
                return !this.hasMod('disabled'); // вернет false/ will recall, if disabled
            }
        }
    }
});
```


<a name="init"></a>

## Initialization ##

Block initialization — creating a JS object in browser memory,
that corresponds to the block instance. Initialization of the block instances is called by the `init()` method and the `i-bem__dom` module in the specific fragment of the DOM tree /на заданном фрагменте DOM-дерева.

Each block instance can be ascribed to three states:

* block instance not initialized (JS object is not created);
* block instance initialized (JS object is created in browser memory);
* block instance deleted /уничтожен (all the links to the JS object of the block instance удалены/ are removed and it can be removed by the garbage collector).

In *i-bem.js* the states of the block instance are described by the `js` service /служебного
modifier.

* A block instance does not have the `js` modifier before initialization.

```HTML
<div class="my-block i-bem" data-bem="..." >...</div>
```

* A block instance gets the `js` modifier set in the `inited` value at the moment of initialization.

```HTML
<div class="my-block i-bem my-block_js_inited" data-bem="...">...</div>
```

* If a fragment the DOM tree is deleted in the process (using the `destruct` method of the `i-bem__dom` module), then block instances are deleted along with it, the HTML elements of which are contained in this fragment. The `js` modifier gets deleted before deleting the block instance.

-------------------------------------------------------------------------------

**NB** If a block instance was
  [bound to several HTML elements](#distrib-block), the block would be available,
  while there is at least one element in the HTML tree bound to it.

-------------------------------------------------------------------------------


If there are instances of different blocks in the HTML element /экземпляров других блоков на HTML-элементе, initialization of one of them (появление/ with the `my-block_js_inited` modifief)
has no influence on the initialization of the others.

**For example**: In HTML element only the block instance `my-block` ?? is initialized,
the block instance`lazy-block` is not initialized:

```HTML
<div class="my-block my-block_js_inited lazy-block i-bem"
    data-bem='{ "my-block": {}, "lazy-block": {} }' >
    ...
</div>
```

-------------------------------------------------------------------------------

**NB** The presence of the `js` modifier allows to write various /разные CSS styles for a block, depending on whether if it is initialized or not.

-------------------------------------------------------------------------------


### Сonstructor of block instance  ###

На изменение значений модификатора `js` можно назначать триггеры так
же, как и для любых других модификаторов блока. The change in values can be assigned to the modifier `js` triggers as well as any other modifier block.

Триггер на установку модификатора `js` в значение `inited` выполняется
при создании блока. Этот триггер можно считать **конструктором
экземпляра блока**: Trigger for setting the `js` modifier to the `inited` value is executed when the block is created. This trigger can be regarded as the constructor of the block instance:

```js
onSetMod: {
    'js': {
        'inited': function() { /* ... */ } // constructor of block instance
    }
}
```


<a name="destruct"></a>

### A descructor of the block instance ###

Моментом удаления блока является момент уничтожения всех ссылок на
JS-объект блока, после чего он может быть удален из памяти браузера
The block is removed at the moment, when all the links to the JS object of the block are deleted, whereupon it can be removed by the garbage collector from the browser memory.

Триггер на удаление модификатора `js` (установку в пустое значение
`''`) выполняется перед удалением блока. Такой триггер можно считать
**деструктором экземпляра блока**. Trigger for deleting the `js` modifier (setting to the empty value `''`) is executed before the removal of the block. This trigger can be considered as **destructor of the block instance**.

```js
onSetMod: {
    'js': {
        '': function() { /* ... */ } // destructor of block instance
    }
}
```


<a name="init-wave"></a>

### Волны инициализации/ Waves of initialization ###

Initialization of block instances on the page, does not necessarily occur simultaneously. Blocks can be added in the process of work (for example, through dynamic generation of HTML based on data coming from the server) or initialized only upon request. Initialization of the next group of blocks is called a wave initialization.
Инициализация очередной группы блоков называется **волной
инициализации**. Initialization of the subsequent group of blocks is called a wave initialization.

Новая волна инициализации создается в следующих случаях: A new wave initialization created in the following cases:

* [Автоматическая инициализация всех блоков в документе по событию `domReady`](#init-auto); Automatic initialization of all blocks in the document ?? `domReady`
* [Инициализация блока по событию на DOM-узле](#init-live) (ленивая инициализация); Initialization of a block? the event in DOM node (lazy initialization)
* [Явный вызов инициализации блоков на указанном фрагменте DOM-дерева](#init-ajax). Manual call of the block initializing in the specific fragment of the DOM tree.


<a name="init-auto"></a>

### Automatic initialization ###

*i-bem.js* framework allows to automatically initialize all the blocks with DOM representation (bound to DOM elements on the page) at the point of emittance of the `domReady` DOM event. To do so one declare on the page the `i-bem` block with the `init` modifier in the `auto` value. Here is an example of the `.deps.js` file:

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
[`i-bem__dom`][]. In the case of automatic initialization there will be created JS objects for all the DOM nodes, еру `class` attributes of which contain `i-bem`.

<a name="init-live"></a>

### Initialization on event/ Инициализация по событию (lazy initialization) ###

Автоматическая инициализация всех блоков в момент загрузки страницы
может быть нежелательной, так как при большом количестве экземпляров
блоков на странице увеличивается время загрузки и объем затраченной
памяти браузера. Automatic initialization of all the blocks when the page is loading may be undesired, since a large number of block instances on the page increase the load time and browser memory footprint.

So, in this case it makes sense to initialize JS objects only at he moment, when their functionality is required by user, for example, by clicking on the block. This kind of initialization is called **lazy initialization** or
**live initialization**.

Для описания условий ленивой инициализации зарезервировано свойство
`live` в разделе статических методов декларации блока. Свойство `live`
может принимать два типа значений: To describe the conditions for lazy initialization the `live` property is reserved in the static methods of block declaration section. `live` property can take two types of values:

* `Boolean`.<br/> Если `live` установлено в значение `true`, экземпляры
  блоков данного класса будут инициализированы только при попытке
  получить соответствующий экземпляр. Подробнее
  см. раздел [Взаимодействие блоков](#ibc). If `live` is set to the `true`, the block instances of this class will be initialized only when trying to get the appropriate instance. For details, see [Interaction blocks](#ibc).

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // this code will be run
                                                   // at the first call /при первом обращении to the block instance
            }
        }
    },
    { live: 'true' } // static methods and properties
);

provide(DOM);

});
```

* `Function`.<br/> Функция, указанная в качестве значения/ ???  `live`: 

    * Выполняется один раз — при попытке инициализации **первого
    экземпляра** блока заданного класса. * It is performed once — at the moment of initializing / при попытке инициализации ** the first block instance** of a certain class.
    * If function returns / возвращает значение `false` value, the block instances will be initialized [automatically](#init-auto).

С помощью этой функции можно организовать инициализацию экземпляров
блока по наступлению DOM-событий на DOM-узле блока и вложенных элементах
или BEM-событий на вложенных блоках. Для этого в коде
функции следует выполнить подписку на
[делегированные события](#delegated-events). With this function one can arrange initialization of block instances on the emittance of DOM events in DOM node of the block and on nested elements or BEM events in nested blocks. For that add event listener on [delegated events](#delegated-events) in the function code.


**For example**: Экземпляры блока/ Instances of the `my-block` block will be initialized on/ `click` DOM events in the block DOM node. По каждому DOM-событию `click` будет вызываться метод экземпляра блока `_onClick`: `_onClick` block instance method  will be called on each `click` DOM event:

```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // выполняется/ performed at the first DOM event 'click'
            }
        },

        _onClick: function() { /* ... */ } // performed at each 'click' DOM event 
    },
    {
        live: function() {
            this.liveBindTo('click', function() {
                this._onClick(); // в момент клика будет создан экземпляр блока и вызван его метод/ at the click the block instance is created and its _onClick method is called
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
значение `false`: If it is necessary to use delegated events in a block, but block initialization can not be postponed (block instances must be initialized immediately after the page is loaded), the `false` value should be returned:


```js
modules.define('i-bem__dom', function(provide, DOM) {

DOM.decl('my-block',
    {
        onSetMod: {
            'js': {
                'inited': function() { /* ... */ } // будет выполнена по наступлении / will be performed before domReady is emitted
            }
        },

        _onClick: function() { /* ... */ } // будет выполняться каждый
                                           // раз при наступлении DOM-события 'click'/ will be performed every time the DOM event is triggered 
    },
    {
        live: function() {
            this.liveBindTo('click', function() { this._onClick() });
            return false; // block instances will be initialized automatically
        }
    }
);

provide(DOM);

});
```

The full list of helpers for adding event listeners to the delegated events
can be found in the source code of the [`i-bem__dom`][] module.

-------------------------------------------------------------------------------

**NB** The `live` property sets lazy initialization for *all the block instances* of the corresponding block, as it технически относится к статическим методам класса/ technically belongs to static methods of the block class. So, even if the `live` property is declared for a block modifier with a certain modifier value, it will be applied to all the blocks of the given class regardless of the modifiers.

-------------------------------------------------------------------------------


<a name="init-ajax"></a>

### Инициализация и удаление блоков на фрагменте DOM-дерева / Initialization and removing blocks in a fragment of DOM tree ###

Procedure of initialization or removal of JS-объектов/ может быть вызвана
явно для указанного фрагмента DOM-дерева can be called manually for the specific fragment of DOM tree. Often this is required in the AJAX interfaces development, when one needs to [dynamically build in new block instances into a page ](#dynamic) or update the current ones. 

The following functions perform dynamic initialization of blocks in *i-bem.js*:

* Initialization/removing blocks in the specific fragment of DOM tree (`init`, `destruct`);
* Adding/replacing a fragment of DOM tree with simultaneous blocks initialization in the renewed fragment (`update`, `replace`, `append`, `prepend`, `before`, `after`).


<a name="init-bem"></a>

### Initialization and removing blocks without DOM representation ###

Чтобы создать JS-объект для блока, не имеющего DOM-представления (не
привязанного к HTML-элементу), необходимо вызвать метод `create`,
который вернет экземпляр блока указанного класса. To create a JS object for the block without DOM representation (not bound to the HTML element), call the `create` method, which will return the block instance of the specific class.

**For example**: At the moment of initialization of block instance with DOM representation
  `container` the `router` block instance without DOM-representation is created. Экземпляр блока
  The `container` block instance will then request/ the created block instance
  `router` at the calling of the method `onRequest`:

```js
modules.define('i-bem__dom', 'i-bem', function(provide, BEM, DOM) {

DOM.decl('container', {
    onSetMod: {
        'js': {
            'inited': function() {
                this._router = BEM.create('router'); // создание экземпляра блока router/ creating the router block instance
            }
        }
    },

    onRequest: function() {
        this._router.route(/* ... */) // вызов метода экземпляра блока router/ call of the router block instance method
    }
});

provide(DOM);

});
```

**Пример**: The block without DOM representation is implemented as a simple 
  [ymaps module][ymaps], without using the `i-bem` module. The block of this kind is used as a common ymaps module (one doesn't need to create a block instance):

```js
modules.define('i-bem__dom', 'router', function(provide, DOM, router) {

DOM.decl('container', {
    onRequest: function() {
        router.route(/* ... */); //  the router block method calling
    }
});

provide(DOM, router);

});
```

**Removing** the block instances without DOM-representation can not be done automatically and so it is the responsibility of the developer. Deleting instances blocks without DOM-representation can not be done automatically. Blocks without DOM representation are common JS objects and get removed at the moment of removal all links to the block object.


**For example**: When deleting the `container` block instance, the `router` block instance without DOM representation, that was created earlier, is deleted.

```js
modules.define('i-bem__dom', 'i-bem', function(provide, BEM, DOM) {

DOM.decl('container', {
    onSetMod : {
        'js' : {
            '' : function() {
                delete this._router; // removing the router block instance
            }
        }
    }
});

provide(DOM);

});
```

# Blocks interaction #

In BEM methodology предполагает / it is asssumed, that blocks should be developed independent, to exclude the dependency of block states / по возможности исключить зависимость состояний одних
блоков от других. However, experience has proven that complete blocks independence 
is unattainable.

Blocks interaction may be implemented in two ways:

* By adding event listeners to [BEM events](#bem-events) of other block instances
  or adding event listeners to [delegated BEM events](#bem-events-delegated).
* Via непосредственного/ direct calling other block instances methods or static class methods of another block. 

-------------------------------------------------------------------------------

**NB** Do not use [DOM events](#dom-events) for for interaction between the blocks.
  DOM-events are meant for dealing with block internal procedures implementation.

-------------------------------------------------------------------------------

*i-bem.js* provides API to implement block interaction:

* [Searching block instances in the DOM tree](#);
* [Access to block instances without DOM-representation](#);
* [Access to block classes.](#).


## Поиск экземпляров блоков в DOM-дереве / Searching block instances in the DOM tree ##

Referring to another in *i-bem.js* is performed from the current block, placed in a particular node of DOM tree.
размещенного на определенном узле DOM-дерева. Поиск других блоков в
DOM-дереве может вестись по трем направлениям (осям) относительно
DOM-узла текущего блока: Search for other blocks in the DOM tree can be carried out in three directions (axes) relating to the DOM node of a current block :

* «Inside the block» — in DOM nodes, nested in the DOM node of a current block.
* «Outside the block» — in DOM nodes, the DOM node of a current block is a child of. Необходимость в таком поиске может свидетельствовать о неудачной архитектуре интерфейса. The necessity of such search may indicate interface architecture failures.
* «На себе ???» — in the same DOM node, where the currrent block is located. That would be in the case of [placing several JS blocks in the same DOM node](#mixes) (mix).

**For example**: При переключении модификатора `disabled` экземпляр блока
  `attach` находит вложенный в него блок `button` и переключает его
  модификатор `disabled` в то же значение, которое получил сам: : When switching the `disabled` modifier, `attach` block instance finds `button` nested i it and toggles its modifier `disabled` to the same value that it received:

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
в исходном коде модуля [`i-bem__dom`][]. The complete list of methods for blocks search is contained in the source code of the [`i-bem__dom`][] module.


-------------------------------------------------------------------------------

**NB** Не используйте jQuery-селекторы для поиска блоков и элементов.
*i-bem.js* предоставляет высокоуровневое API для доступа к DOM-узлам
блоков и элементов. Обращение к DOM-дереву в обход этого API менее
устойчивым к изменениям БЭМ-библиотек и может привести к возникновению
сложно обнаруживаемых ошибок. Do not use jQuery selectors for seaching blocks and elements. *i-bem.js* provides a high-level API to access the DOM nodes of blocks and elements. Referring to DOM tree to bypass the API makes the code less resistant to the BEM libraries changes and can cause errors hard to detect.

-------------------------------------------------------------------------------

## Доступ к экземплярам блоков без DOM-представления / Access to block instances without DOM representation ##

При создании экземпляра блока без DOM-представления необходимо
позаботиться о том, чтобы ссылка на этот экземпляр была доступна
блокам, которым потребуется взаимодействовать с ним. Подробности и
пример см. в разделе [Инициализация и удаление блоков без DOM-представления](#init-bem). 
When creating a block instance without DOM representation make sure that the link to this instance is available to the blocks that need to interact with it. Details and an example please see in [Initialization and removing blocks without DOM representation](#init-bem).


## Доступ к классам блоков / Access to the block classes ##

JS-компоненты, соответствующие всем блокам («классы» блоков), хранятся
в структуре данных `BEM.blocks`. Классы блоков,
[не привязанных к DOM-дереву](#i-blocks), также размещены в этой
структуре данных. При необходимости доступа к таким блокам следует
использовать конструкцию: JS-components corresponding to all blocks (blocks "classes" ) are stored in `BEM.blocks` data structure. The block classes that are [not bound to the DOM tree](#i-blocks), are also placed in this data structure. To access such blocks use the following construction:
```js
BEM.blocks['name']
```
where `name` — block name.

Доступ к классам блоков необходим для решения двух основных задач: Access to the block classes is necessary to address two major problems:

* [BEM events delegation](#bem-events-delegated).
* Calling the class static method.

**For example**: Вызов статического метода `close` блока `popup` — закрыть
  все попапы на странице: Example: Calling the `close` static method of the `popup` block - close all the pop-ups on the page:

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

# Что дальше?/ What's next? #

General info on BEM methodology, tools, news in the world of BEM can be found at [bem.info](http://bem.info/).

Полную информацию обо всех методах API *i-bem.js* можно найти в
исходном коде, который сопровождается структурированными комментариями
в формате JSDoc: Complete information about all the API methods *i-bem.js* can be found in the source code, accompanied by structured comments in JSDoc format:

* [`i-bem`][];
* [`i-bem__dom`][].

Задать вопрос опытным пользователям и разработчикам *i-bem.js* и
следить за текущими обсуждениями можно в социальных сетях: Ask a question to experienced users and developers of *i-bem.js* and monitor ongoing discussions in social networks:

* [Ya.ru club](http://clubs.ya.ru/bem/);
* [Facebook group](http://www.facebook.com/#!/groups/209713935765634/);
* [Twitter](https://twitter.com/bem_ru).

Прочитать о принципах работы *i-bem.js* в другом изложении, найти
образцы его применения и пошаговые инструкции на примере простых
проектов можно в статьях: Read about the principles of *i-bem.js* in another presentation, find samples of its application and step-by-step instructions on the example of simple projects in these articles:

* [JavaScript по БЭМ:/ JavaScript according to BEM: basic terms](http://ru.bem.info/articles/bem-js-main-terms/);
* [Tutorial on JavaScript in BEM terms](https://github.com/varya/bem-js-tutorial);
* [Попробуй БЭМ на вкус! Try and taste BEM!](http://habrahabr.ru/post/162385/);
* [БЭМ-приложение на Leaflet и API 2GIS / BEM application to Leaflet and API 2GIS](http://ru.bem.info/articles/firm-card-story/).


-------------------------------------------------------------------------------

**NB** Обратите внимание, что в перечисленных статьях может
использоваться устаревший синтаксис, не соответствующий текущей версии
*i-bem.js*, включенной в bem-core. Please note that these articles can contain outdated syntax that does not match the current version of the *i-bem.js*, included in the bem-core library.

-------------------------------------------------------------------------------


[ymaps]: https://github.com/ymaps/modules

[bem-tools]: http://ru.bem.info/tools/bem/

[`i-bem`]: https://github.com/bem/bem-core/blob/v1/common.blocks/i-bem/i-bem.vanilla.js

[`i-bem__dom`]: https://github.com/bem/bem-core/blob/v1/common.blocks/i-bem/__dom/i-bem__dom.js
