# HTML Integration for `import.meta`

We hope to eventually introduce two properties onto `import.meta` in web browsers, via the the [HTML Standard](http://html.spec.whatwg.org/multipage/).

## `import.meta.url`

This property will contain the module script's [base URL](https://html.spec.whatwg.org/#concept-module-script-base-url), [serialized](https://url.spec.whatwg.org/#concept-url-serializer).

A specification pull request exists at [whatwg/html#3141](https://github.com/whatwg/html/pull/3141) for this. (It's pretty straightforward.)

## `import.meta.scriptElement`

This property will contain the `<script>` element that initiated the fetching of the current module, if any. It is meant to be a more robust replacement for `document.currentScript`. See [whatwg/html#1013](https://github.com/whatwg/html/issues/1013) for the genesis of the idea.

However, specifying this is harder than you might imagine. For example, a given module script can be contained in multiple graphs with different root `<script>` elements, and which one caused it to instantiate or evaluate could be nondeterministic. There is also a concern that introducing such a long-lived pointer to the `<script>` element could be a memory leak.

These concerns are being discussed [in the aforementioned thread](https://github.com/whatwg/html/issues/1013#issuecomment-329344476). In the meantime, you can look at the old design we proposed for `import.meta.scriptElement` by checking out [an older draft of this document](https://github.com/tc39/proposal-import-meta/blob/f5d39bc471a5bf2791708f9a3fec943380d9e3ee/HTML%20Integration.md). In light of the above, it was not properly thought through, so this is just for historical reference.
