# import.meta

This repository contains a proposal for adding an `import.meta` metaproperty to JavaScript, for holding host-specific metadata about the current module. It is currently in stage 4 of [the TC39 process](https://tc39.github.io/process-document/).

Previously this problem space was discussed with the module-loading community in [whatwg/html#1013](https://github.com/whatwg/html/issues/1013). The result was a presentation at the May 2017 TC39 meeting of [these slides](https://docs.google.com/presentation/d/1p1BGFY05-iCiop8yV0hNyWU41_wlwwfv6HIDkRNNIBQ/edit?usp=sharing). The committee's feedback about the options presented there led to the particular form chosen in this proposal; these discussions are summarized in the rest of this README.

You can view the in-progress [spec draft](https://tc39.github.io/proposal-import-meta/) and take part in the discussions on the [issue tracker](https://github.com/tc39/proposal-import-meta/issues).

## Motivation and use cases

It is often the case that a host environment can provide useful module-specific information, to code evaluating within a module. Several examples follow:

### The module's URL or filename

This is provided in Node.js's CommonJS module system via the in-scope `__filename` variable (and its counterpart `__dirname`). This allows easy resolution of resources relative to the module file, via code such as

```js
const fs = require("fs");
const path = require("path");
const bytes = fs.readFileSync(path.resolve(__dirname, "data.bin"));
```

Without `__dirname` available, code like `fs.readFileSync("data.bin")` would resolve `data.bin` relative to the current working directory. This is generally not useful for library authors, which bundle their resources with their modules which could be located anywhere relative to the CWD.

Very similar use cases exist in browsers, where the URL would stand in place of the filename.

### The initiating `<script>`

This is provided in browsers for classic scripts (i.e. non-module scripts) via the global `document.currentScript` property. It is often used for configuration of an included library. The page writes code such as:

```html
<script data-option="value" src="library.js"></script>
```

and the library then looks up the value of `data-option` via code like

```js
const theOption = document.currentScript.dataset.option;
```

As an aside, the mechanism of using an (effectively) global variable for this, instead of a lexically-scoped value, is problematic, as it means the value is only set at top-level, and must be saved there if it is to be used in any asynchronous code.

### Am I the "main" module?

In Node.js, it is common practice to branch on whether you are the "main" or "entry" module for a program, via code such as:

```js
if (module === process.mainModule) {
  // run tests for this library, or provide a CLI interface, or similar
}
```

That is, it allows a single file to serve as a library when imported, or as a side-effecting program when run directly.

Note how this particular formulation relies on comparing the host-provided scoped value `module` with the host-provided global value `process.mainModule`.

### Other miscellaneous use cases

Other cases that can be envisioned include:

- Other Node.js utilities, such as `module.children` or `require.resolve()`
- Information on the "package" a module belongs to, either via Node.js's package.json or a [web package](https://github.com/dimich-g/webpackage)
- A pointer to the containing DocumentFragment for JavaScript modules embedded inside [HTML modules](https://docs.google.com/presentation/d/1ksnC9Qr3c8RwbDyo1G8ZZSVOEfXpnfQsTHhR5ny9Wk4/edit#slide=id.g1c508fcb31_0_17).

## Constraints

Given that this information is so often host-specific, we want JavaScript to provide a general extensibility mechanism that hosts can use, instead of requiring standardization in JavaScript for every piece of metadata.

Also, as alluded to above, this is information is best provided lexically, instead of via e.g. a temporarily-set global variable.

## Proposed solution

This proposal adds an `import.meta` meta-property, which is itself an object. It is created by the ECMAScript implementation, with a null prototype. The host environment can return a set of properties (as key/value pairs) that will be added to the object. Finally, as an escape hatch, the object can be modified arbitrarily by the host if necessary.

The `import.meta` meta-property is only syntactically valid in modules, as it is meant for meta-information about the currently running module, and should not be repurposed for information about the currently-running classic script.

## Example

The following code uses a couple properties that we anticipate adding to `import.meta` in browsers:

```js
(async () => {
  const response = await fetch(new URL("../hamsters.jpg", import.meta.url));
  const blob = await response.blob();

  const size = import.meta.scriptElement.dataset.size || 300;

  const image = new Image();
  image.src = URL.createObjectURL(blob);
  image.width = image.height = size;

  document.body.appendChild(image);
})();
```

When this module is loaded, no matter its location, it will load a sibling file `hamsters.jpg`, and display the image. The size of the image can be configured using the script element used to import it, such as with

```html
<script type="module" src="path/to/hamster-displayer.mjs" data-size="500"></script>
```

## Alternative solutions explored

There are a number of other ways of potentially supplying host-specific metadata. The main other avenues explored were:

### Using a particular module specifier

The idea here is that we already have a mechanism for obtaining context-specific values from the module system: importing them! In this case the syntax for obtaining these properties would be via top-level importing of some particular module specifier, such as

```js
import { url, scriptElement } from "js:context";
```

This fits well with the existing framework, and can be taken care of entirely on the host side, requiring no new ECMAScript proposals (since hosts have control over how module specifiers are interpreted). However, on the web, it saw [opposition from WebKit](https://github.com/whatwg/html/issues/1013#issuecomment-281863721), and given TC39 was not unamimously united behind this idea, it was deemed not worthwhile trying to overcome WebKit's objection. Node.js may still implement this, especially if `import.meta` is not available soon enough for their needs.

### Introducing lexical variables in another way

This possibility would introduce the appropriate information as variables somehow injected into the scope of the module, similar to how Node.js currently injects `__dirname`, `__filename`, and `module`. Then they would just be used as normal in-scope variables:

```js
console.log(moduleURL);
console.log(moduleScriptElement);
```

From an implementation and spec perspective, this could be done with no ECMAScript spec changes by, for example, the host prepending appropriate variable declarations to every module source text before handing it off to the ECMAScript spec mechanisms. Alternately, the host could introduce a new type of module record that has a slightly different scope for its [[Environment]] field. (Or ECMASCript could be modified to allow hosts to intervene in setting up the [[Environment]] of a source text module record.)

The committee was generally against handling this idea in ECMAScript as several members were not a fan of "polluting" module scopes with new variables. It remains a fallback option for web hosts if `import.meta` fails to advance quickly enough.

## FAQs

### Will this object be locked down in any way?

The committee's tentative decision was no: by default the `import.meta` object will be extensible, and its properties will be writable, configurable, and enumerable.

There is no real benefit to locking down the object, as it is local to the module, and can only be shared explicitly by passing it around. It does not represent "global state" or anything of the sort.

Additionally, leaving it mutable allows module-to-module transpilers to "polyfill" future features by inserting a few extra lines at the top of each module, adding or modifying `import.meta` properties. For example, if in a few years HTML adds an additional property, it would be possible to write a transpiler that adds that property, when targeting older browsers.

Finally, the escape hatch provided by HostFinalizeImportMeta allows hosts which prefer a more locked-down `import.meta` object to take that route.
