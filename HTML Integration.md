# HTML Integration for `import.meta`

The following outlines the spec changes to the [HTML Standard](http://html.spec.whatwg.org/multipage/) that would be necessary to support the `import.meta` proposal in a web browser host environment. It is phrased as a patch ready to be lifted into the [Integration with the JavaScript module system](https://html.spec.whatwg.org/#integration-with-the-javascript-module-system) section.

## HostGetImportMetaProperties(_moduleRecord_)

JavaScript contains an implementation-defined [HostResolveImportedModule](https://domenic.github.io/proposal-import-meta/#sec-hostgetimportmetaproperties) abstract operation. User agents must use the following implementation: [JAVASCRIPT]

1. Let _module script_ be _moduleRecord_.[[HostDefined]].
1. Let _url_ be _module script_'s [base URL](https://html.spec.whatwg.org/#concept-module-script-base-url), [serialized](https://url.spec.whatwg.org/#concept-url-serializer).
1. Return « Record { [[Key]]: "`url`", [[Value]]: _url_ } ».
