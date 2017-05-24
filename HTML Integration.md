# HTML Integration for `import.meta`

The following outlines the spec changes to the [HTML Standard](http://html.spec.whatwg.org/multipage/) that would be necessary to support the `import.meta` proposal in a web browser host environment.

## Scripting processing model definitions

_In the [definitions section](https://html.spec.whatwg.org/#module-script), add a new field to module scripts:_

**script element**: A `HTMLScriptElement` that initiated the graph, or null if it was not initiated by a `script` element.

## Fetching scripts

_In the [fetching scripts](https://html.spec.whatwg.org/#fetching-scripts) section, add an additional optional argument *script element* to the **fetch a module script graph** algorithm._

_This then needs to be threaded through various algorithms, starting with **[fetch a module script graph](https://html.spec.whatwg.org/#fetch-a-module-script-tree)** and **[fetch the descendants of and instantiate a module script](https://html.spec.whatwg.org/#fetch-the-descendants-of-and-instantiate-a-module-script)**. Eventually it is passed to **[create a module script](https://html.spec.whatwg.org/#creating-a-module-script)** which sets the value in the newly-created module script's **script element** field._

_The **[fetch a module worker script graph](https://html.spec.whatwg.org/#fetch-a-module-worker-script-tree)** algorithm should, in contrast, pass through null as the script element to all the algorithms it calls._

## Prepare a script

_In the **[prepare a script](https://html.spec.whatwg.org/#prepare-a-script)** algorithm, pass through the `script` element to the invocations of **fetch a module script graph**, **create a module script**, and **fetch the descendants of and instantiate a module scirpt**._

## HostGetImportMetaProperties(_moduleRecord_)

_To the [Integration with the JavaScript module system](https://html.spec.whatwg.org/#integration-with-the-javascript-module-system) section, add this new subsection._

JavaScript contains an implementation-defined [HostResolveImportedModule](https://domenic.github.io/proposal-import-meta/#sec-hostgetimportmetaproperties) abstract operation. User agents must use the following implementation: [JAVASCRIPT]

1. Let _module script_ be _moduleRecord_.[[HostDefined]].
1. Let _url_ be _module script_'s [base URL](https://html.spec.whatwg.org/#concept-module-script-base-url), [serialized](https://url.spec.whatwg.org/#concept-url-serializer).
1. Let _script element_ be _module script_'s **script element**.
1. Return « Record { [[Key]]: "`url`", [[Value]]: _url_ }, Record { [[Key]]: "`scriptElement`", [[Value]]: _script element_ } ».
