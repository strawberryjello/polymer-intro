# DONE? Overview

## Componentization for the web

Software componentization

-   when you break your software system down into smaller easily identifiable pieces that have well-defined interfaces - and you do this in a specific way (i.e. by following a component model)

Decomposition vs composition

-   the traditional development process is to break down a large problem into sub-problems and solve those
-   design by composition starts by putting existing components together towards the product requirements

The goal

-   by investing in a component-based approach to build components that are well-defined and flexible we get the payoff when we re-use it - the next time around - and component evolution (including bug fixes) will payoff multiplicatively

Source: [Software Componentization](http://blogs.windriver.com/koning/2006/09/components.html)

## Web components: an emerging standard

-   previous attempts by Microsoft ([HTML Components](http://www.w3.org/TR/NOTE-HTMLComponents)) and Mozilla ([XBL](http://www.w3.org/TR/2001/NOTE-xbl-20010223/), [XBL2](http://www.w3.org/TR/xbl/)) to introduce componentization standards failed
-   Google's attempt to standardize web components (and Polymer) seems to be succeeding &#x2013; different web browsers are starting to implement web components natively
    -   instead of pushing a single monolithic standard, a handful of smaller interrelated standards were pushed, with varying levels of adoption
-   different JavaScript libraries (eg, Angular) are aligning towards componentization

## Polymer: opinionated approach to web components

-   a lightweight sugaring layer on top of the web components APIs
-   Polymer 1.0's goal: make development using web components easier and faster by providing several kinds of ready-made custom elements and its own extensions of the existing web components
-   "it's a library, not a framework"
    -   emphasis on interoperability: integrating Polymer with other frameworks should be doable

<f4a5ba7a0008046ec8b7#file-layers-of-polymer-png>

# NEARLYTHERE Intro to Web Components

## NEARLYTHERE The components

### DONE? Custom Elements

<http://w3c.github.io/webcomponents/spec/custom/>

**Custom elements** define an extension point for the HTML parser to be able to recognize a new "custom element" name and provide it with a JavaScript-backed object model automatically

Example:

    <my-avatar service="twitter" username="matz_translated" />

    var MyAvatarPrototype = Object.create(HTMLElement.prototype);
    
    MyAvatarPrototype.createdCallback = function() {
      var username = this.getAttribute('username');
      var service = this.getAttribute('service');
    
      var url = 'http://avatars.io/' + service + '/' + username;
    
      var img = document.createElement('img');
      img.setAttribute('src', url);
      this.appendChild(img);
    };
    
    document.registerElement('my-avatar', {
      prototype: MyAvatarPrototype
    });

*(Example from [Phil Leggetter's DevWeek presentation](https://www.youtube.com/watch?v=BG4KHxASs_A))*

**is attribute**

-   Hidden inside the Custom Elements spec is another significant feature &#x2013; the ability to indicate that a built-in element should be given a custom element name and API capabilities

Example:

    <button is="mega-button">

    var MegaButton = document.createElement('mega-button', {
      prototype: Object.create(HTMLButtonElement.prototype),
      extends: 'button'
    });

*(Example from [Custom Elements - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/))*

Note:

-   custom element names **must contain a dash (-)**
    -   the parser distinguishes between custom elements and regular HTML elements this way

**Lifecycle callback methods** (optional)

-   createdCallback
-   attachedCallback
-   detachedCallback
-   attributeChangedCallback(attribute, oldVal, newVal)

### DONE? Templates

<http://www.w3.org/TR/html5/scripting-1.html#the-template-element>

-   this early web components feature is now part of the [HTML5 recommendation](http://www.w3.org/TR/html5/)
-   the template element introduced the concept of inertness (template's children don't trigger downloads or respond to user input, etc.) and was the first way to declaratively create a disconnected element subtree in HTML
-   templates may be used for a variety of things from template-stamping and data-binding to conveying the content of a shadow DOM

Example:

    <template id="mytemplate">
      <img src="" alt="great image">
      <div class="comment"></div>
    </template>

    var t = document.querySelector('#mytemplate');
    // Populate the src at runtime.
    t.content.querySelector('img').src = 'logo.png';
    
    var clone = document.importNode(t.content, true);
    document.body.appendChild(clone);

*(Example from [Custom Elements - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/))*

**Gotchas**

-   no way to prerender a template (cannot preload assets, process JS, download initial CSS, etc.)
-   be careful with nested templates: nested templates require that their children also be manually activated

### DONE? Shadow DOM

<http://w3c.github.io/webcomponents/spec/shadow/>

-   provides an imperative API for creating a separate tree of elements that can be connected (only once) to a host element
-   these "shadow" children replace the "real" children when rendering the document

Example:

    <button>Hello, world!</button>
    <script>
    var host = document.querySelector('button');
    var root = host.createShadowRoot();
    root.textContent = 'こんにちは、影の世界!';
    </script>

<f4a5ba7a0008046ec8b7#file-shadow-root-png>

**Benefits**

-   DOM/CSS "scoping"/protection (prevents CSS from leaking into a custom element)
-   encapsulation

Example:

    <div id="nameTag">Matz</div>
    
    <template id="nameTagTemplate">
      <style>
      .outer {
        border: 2px solid brown;
        border-radius: 1em;
        background: red;
        font-size: 20pt;
        width: 12em;
        height: 7em;
        text-align: center;
      }
      .boilerplate {
        color: white;
        font-family: sans-serif;
        padding: 0.5em;
      }
      .name {
        color: black;
        background: white;
        font-family: "Marker Felt", cursive;
        font-size: 45pt;
        padding-top: 0.2em;
      }
      </style>
      <div class="outer">
        <div class="boilerplate">
          Hi! My name is
        </div>
        <div class="name">
          <content></content>
        </div>
      </div>
    </template>
    
    <script>
      var shadow = document.querySelector('#nameTag').createShadowRoot();
      var template = document.querySelector('#nameTagTemplate');
      var clone = document.importNode(template.content, true);
      shadow.appendChild(clone);
    </script>

*(Examples from [Shadow DOM 101 - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/))*

**Other features**

-   multiple shadow roots for a host
-   nested shadow roots

### NEARLYTHERE HTML Imports

<http://w3c.github.io/webcomponents/spec/imports/>

-   defines a declarative syntax to "import" (request, download and parse) HTML into a document
-   imports (using a link element's rel="import") execute the imported document's script in the context of the host page (thus having access to the same global object and state)
-   the HTML, JavaScript, and CSS parts of a web component can be conveniently deployed using a single import

Example:

    <link rel="import" href="bootstrap.html" />

bootstrap.html:

    <link rel="stylesheet" href="bootstrap.css">
    <link rel="stylesheet" href="fonts.css">
    <script src="jquery.js"></script>
    <script src="bootstrap.js"></script>
    <script src="bootstrap-tooltip.js"></script>
    <script src="bootstrap-dropdown.js"></script>
    ...
    
    <!-- scaffolding markup -->
    <template>
      ...
    </template>

Notes:

-   imports that reference the same URL are only retrieved once (the browser's network stack automatically checks for duplicates)
    -   dependency management
-   to load content from another domain, the URL of an import (the import location) needs to be CORS-enabled (see [Cross-origin resource sharing on Wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing))

**Using the content**

-   an imported file's contents are inert until you use them (ie, with JavaScript)

Example:

    <head>
      <link rel="import" href="warnings.html">
    </head>
    <body>
      ...
      <script>
        var link = document.querySelector('link[rel="import"]');
        var content = link.import;
    
        // Grab DOM from warning.html's document.
        var el = content.querySelector('.warning');
    
        document.body.appendChild(el.cloneNode(true));
      </script>
    </body>

warnings.html:

    <div class="warning">
      <style scoped>
        h3 {
          color: red;
        }
      </style>
      <h3>Warning!</h3>
      <p>This page is under construction</p>
    </div>
    
    <div class="outdated">
      <h3>Heads up!</h3>
      <p>This content may be out of date</p>
    </div>

*(Example from [HTML Imports - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/))*

**Scripting in imports**

-   an import can access its own DOM and/or the DOM of the page that's importing it
-   script in the import is executed in the context of the window that contains the importing document
-   scripts in an import are processed in order, but do not block the main document parsing
-   scripts execute at import time, stylesheets, markup, and other resources need to be added to the main page explicitly

## DONE? Browser support

<http://jonrimmer.github.io/are-we-componentized-yet/>

<f4a5ba7a0008046ec8b7#file-are-we-componentized-yet-png>

Note:

-   Mozilla no longer supports HTML imports because of ES6 (see [Mozilla and Web Components: Update](https://hacks.mozilla.org/2014/12/mozilla-and-web-components/), 15 Dec 2014)

## DONE? Polyfills

-   polyfill: downloadable code that implements features not yet supported natively by a browser
-   for browsers that don't support certain web components, the ff. are available via [webcomponents.js](https://github.com/WebComponents/webcomponentsjs):
    -   Custom Elements
    -   HTML Imports
    -   Shadow DOM
    -   also includes MutationObserver and WeakMap
-   [webcomponents-lite.js](https://github.com/webcomponents/webcomponentsjs/blob/master/webcomponents-lite.js) excludes Shadow DOM

# NEARLYTHERE Intro to Polymer

## DONE? Setup requirements

-   Installation instructions
    -   [Full Polymer library](https://www.polymer-project.org/1.0/docs/start/getting-the-code.html)
    -   [Individual elements](https://elements.polymer-project.org/guides/using-elements#installing-elements)
-   Recommended: install via Bower (requires Node.js/npm)
-   Or download the zip archive
    -   updating the dependencies/adding new elements requires downloading a new zip archive unless you convert to Bower

## DONE? Shady DOM

Shady DOM vs shadow DOM polyfill

-   web components require tree-scoping for proper encapsulation
-   shadow DOM is the standard that implements tree-scoping, but it's not yet universally implemented
-   polyfilling shadow DOM is hard, the robust polyfill is invasive and slow
-   shady DOM is a super-fast shim for shadow DOM that provides tree-scoping, but has drawbacks &#x2013; most importantly, one must use the shady DOM APIs to work with scoped trees
-   the annoying bits of shady DOM are exactly the reasons why shadow DOM needs to be native across platforms

Shady DOM is compatible with shadow DOM

-   the shady DOM API can optionally employ real shadow DOM where it's available
-   you can write one code base that works on all platforms, but you enjoy improved performance and robustness on platforms that implement Shadow DOM

Source: [What is shady DOM?](https://www.polymer-project.org/1.0/articles/shadydom.html)

## DONE? Vulcanize

<https://github.com/Polymer/vulcanize>

-   the more HTML imports you have, the more requests your app will make
-   Vulcanize reduces an HTML file and its dependent HTML Imports into one file
-   in the future, technologies such as [HTTP/2](http://en.wikipedia.org/wiki/HTTP/2) and [Server Push](https://http2.github.io/faq/#whats-the-benefit-of-server-push) will likely obsolete the need for a tool like Vulcanize for production uses

## STARTED Web Component Tester

<https://github.com/Polymer/web-component-tester>

-   a browser-based testing environment
-   WCT will run your tests against whatever browsers you have locally installed, or remotely via Sauce Labs
-   test suites in HTML or JS files

## NEARLYTHERE Features

Source: [dev guide feature overview](https://www.polymer-project.org/1.0/docs/devguide/feature-overview.html)

### DONE? Registration and lifecycle

-   Registering an element associates a class (prototype) with a custom element name

Example:

    MyElement = Polymer({
      is: 'my-element',
    
      created: function() {
        this.textContent = 'My element!';
      }
    });

**Extending native elements**

    MyInput = Polymer({
      is: 'my-input',
      extends: 'input',
    
      created: function() {
        this.style.border = '1px solid red';
      }
    });

**Polymer's lifecycle callbacks**

-   created
-   attached
-   detached
-   attributeChanged
-   ready
    -   invoked when Polymer has finished creating and initializing the element's local DOM

### DONE? Declared properties

-   Declared properties can be configured from markup using attributes
-   Declared properties can optionally support change observers, two-way data binding, and reflection to attributes
-   You can also declare computed properties and read-only properties

Examples:

    Polymer({
      is: 'x-custom',
    
      properties: {
        disabled: {
          type: Boolean,
          observer: '_disabledChanged'
        }
      },
    
      _disabledChanged: function(newValue, oldValue) {
        this.toggleClass('disabled', newValue);
        this.highlight = true;
      }
    });

    Polymer({
      is: 'x-custom',
    
      properties: {
        first: String,
        last: String,
    
        fullName: {
          type: String,
          // when `first` or `last` changes `computeFullName` is called once
          // and the value it returns is stored as `fullName`
          computed: 'computeFullName(first, last)'
        } 
      },
    
      computeFullName: function(first, last) {
        return first + ' ' + last;
      }
    });

### DONE? Local DOM

-   Local DOM is the DOM created and managed by the element (ie, shady DOM + shadow DOM)
-   Polymer uses shady DOM by default
-   shady DOM requires you to use the [Polymer DOM API](https://www.polymer-project.org/1.0/docs/devguide/local-dom.html#dom-api)

### DONE? Events

-   Attaching event listeners to the host object and local DOM children

Example:

    <dom-module id="x-custom">
      <template>
        <button on-click="handleClick">Kick Me</button>
      </template>
    
      <script>
        Polymer({
          is: 'x-custom',
    
          handleClick: function() {
            alert('Ow!');
          }
        });
      </script>
    </dom-module>

### NEARLYTHERE Data binding

-   Data binding binds a property or sub-property of a custom element (the host element) to a property or attribute of an element in its local DOM (the child or target element)

**Binding annotations**

-   Square brackets `[[]]` create one-way bindings. Data flow is downward, host-to-child, and the binding never modifies the host property.
-   Curly brackets `{{}}` create automatic bindings. Data flow is one-way or two-way, depending whether the target property is configured for two-way binding.

Example:

    <dom-module id="host-element">
        <template>
          <child-element name="{{myName}}"></child-element>  
        </template>
    </dom-module>

### DONE? Behaviors

-   Behaviors are reusable modules of code that can be mixed into Polymer elements
-   To add a behavior to a Polymer element definition, include it in a `behaviors` array on the prototype

Example:

    <link rel="import" href="highlight-behavior.html">
    
    <script>
      Polymer({
        is: 'my-element',
        behaviors: [HighlightBehavior]
      });
    </script>

highlight-behavior.html:

    <script>
        HighlightBehavior = {
    
          properties: {
            isHighlighted: {
              type: Boolean,
              value: false,
              notify: true,
              observer: '_highlightChanged'
            }
          },
    
          listeners: {
            click: '_toggleHighlight'
          },
    
          created: function() {
            console.log('Highlighting for ', this, 'enabled!');
          },
    
          _toggleHighlight: function() {
            this.isHighlighted = !this.isHighlighted;
          },
    
          _highlightChanged: function(value) {
            this.toggleClass('highlighted', value);
          }
    
        };
    </script>

## DONE? Polymer element categories

From the [elements guide](https://elements.polymer-project.org/guides/using-elements) and the [elements catalog](https://elements.polymer-project.org/):

-   Iron elements
    -   A set of utility elements including generic UI elements (such as icons, input and layout components), as well as non-UI elements providing features like AJAX, signaling and storage
-   Paper elements
    -   A set of UI elements that implement the [material design system](http://www.google.com/design/spec/material-design/introduction.html)
-   Gold elements
    -   Form elements for ecommerce
-   Neon elements
    -   Animation-related elements
-   Platinum elements
    -   Elements for app-like features, like push notifications, offline caching and bluetooth
-   Google Web components
    -   Components for Google's API and services
-   Molecules
    -   Wrappers for third-party libraries

# NEARLYTHERE Assessment

## Pros

-   Polymer is being pushed by Google
    -   active community, well-documented, ongoing development
-   existing suite of reusable components available
-   custom elements for Google APIs available

## Cons

-   opinionated approach to web components (YMMV)
-   automagical code
    -   part of the ramp-up for a new dev is learning how to code the Polymer way (eg, shady DOM, custom properties extensions, etc)
-   integration with frameworks: trial and error
-   currently not possible to choose whatever subset of Polymer 1.0's features you want
    -   the [experimental features guide](https://www.polymer-project.org/1.0/docs/devguide/experimental.html) describes `polymer-mini.html` and `polymer-micro.html`, which are smaller subsets of `polymer.html` (subject to change in future releases)

# DONE? Resources

## Polymer

Project sites

-   <https://www.polymer-project.org/1.0/>
-   <https://elements.polymer-project.org/>

Sep 2015 summit

-   <https://www.polymer-project.org/summit/>
    -   [Codelabs](https://codelabs.developers.google.com/polymer-summit)
    -   [Videos](https://www.youtube.com/playlist?list=PLNYkxOF6rcICdISJclfQhj2S8QZGjXV8J)

Communities

-   [Google+](https://plus.google.com/u/1/communities/115626364525706131031)

GitHub

-   <https://github.com/Polymer/polymer>
-   <https://github.com/Polymer/web-component-tester>
-   <https://github.com/PolymerElements/polymer-starter-kit>
    -   [Set-up tutorial](https://www.polymer-project.org/1.0/docs/start/psk/set-up.html)

Polycasts

-   [Youtube playlist page](https://www.youtube.com/playlist?list=PLOU2XLYxmsII5c3Mgw6fNYCzaWrsM3sMN)

Articles

-   [What is shady DOM?](https://www.polymer-project.org/1.0/articles/shadydom.html) (28 May 2015)

Presentations

-   [Componentize your app with Polymer Elements](http://webcomponents.org/presentations/componentize-your-app-with-polymer-elements/)
    -   [slide deck](https://speakerdeck.com/robdodson/componentize-your-app-with-polymer)

Related sites

-   <http://builtwithpolymer.org/>

## Web Components

Project sites

-   <http://webcomponents.org/>
-   <http://www.w3.org/wiki/WebComponents/>

Communities

-   <https://plus.google.com/+WebcomponentsOrg>

GitHub

-   <https://github.com/WebComponents/webcomponentsjs> (polyfills suite)

Specs

-   [intro to web components](http://w3c.github.io/webcomponents/explainer/)
-   [custom elements](http://w3c.github.io/webcomponents/spec/custom/)
-   [HTML imports](http://w3c.github.io/webcomponents/spec/imports/)
-   [templates](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element)
-   [shadow DOM](http://w3c.github.io/webcomponents/spec/shadow/)

Polyfills

-   [polyfills](http://webcomponents.org/polyfills/)

Other Web Component projects

-   [Basic Web Components library](https://github.com/basic-web-components/basic-web-components) (from Component Kitchen)
-   [The Gold Standard Checklist for Web Components](https://github.com/webcomponents/gold-standard/wiki) (from Component Kitchen)

Articles

-   [Bringing componentization to the web: An overview of Web Components - Microsoft Edge Dev Blog](https://blogs.windows.com/msedgedev/2015/07/14/bringing-componentization-to-the-web-an-overview-of-web-components/) (14 July 2015)
-   [Shadow DOM 101 - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) (updated 18 Dec 2013)
-   [HTML's New Template Tag - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/template/) (updated 18 Dec 2013)
-   [HTML Imports - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/) (updated 18 Dec 2013)
-   [Custom Elements - HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/) (updated 18 Dec 2013)

Presentations

-   [Why you should be using Web Components now, and how](https://www.youtube.com/watch?v=BG4KHxASs_A) - DevWeek 2015 (published 30 Oct 2015)
    -   [slide deck](http://www.slideshare.net/leggetter/why-you-should-be-using-web-components-and-how)
