---
layout: post
title: Restricted Objects
---

ES5 provides three static methods on `Object` to prevent various types of changes to your objects. They are, in order of strength:

1. `Object.preventExtensions` [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/preventExtensions)
2. `Object.seal` [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)
3. `Object.freeze` [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)

<!--more-->

Each provides a different level of restriction, and I'll explore them below. One thing to keep in mind: attempting forbidden changes _does not throw an error unless `'use strict'` is in effect_. This is important! The bugs caused by silent failed property changes can be incredibly hard to debug. These methods work on arrays as well, and throw similar errors. Keep in mind they may prevent you from using array methods that mutate an array in-place (`push`, `pop`, `splice`, etc.).

Here's some test code I'm going to refer to throughout this post:

```javascript
// pass 'preventExtensions', 'seal', or 'freeze' to this function
function testObjRestrictions ( method ) {
  'use strict';

  var obj = { key: 'value' };
  Object[method]( obj );

  // Case 1: adding a proprety
  try {
    obj.newKey = "newValue";
  } catch ( err ) {
    console.error( err );
    // => "Can't add property otherKey, object is not extensible"
  }

  // Case: 2 deleting a property
  try {
    delete obj.key;
  } catch ( err ) {
    console.error( err );
    // => "Cannot delete property 'key' of #<Object>"
  }

  // Case 3: reassigning a property
  try {
    obj.key = "changedValue";
  } catch ( err ) {
    console.error( err );
    // => "Cannot assign to read only property 'key' of #<Object>"
  }
}
```

This function will allow us to catch and inspect the various errors thrown by `preventExtensions`, `seal`, and `freeze`. Passing `preventExtensions` to this function will only throw in case 1, when attempting to add another property. Using `seal` will throw in cases 1 and 2, so the own properties of the object can't be changed at all. Finally, `freeze` will throw in all three cases; no properties can be added, removed, or reassigned. Remember, these errors only get thrown when `'use strict'` is active.

<table class="plain">
  <tr>
    <td></td>
    <td><code>preventExtensions</code></td>
    <td><code>seal</code></td>
    <td><code>freeze</code></td>
  </tr>
  <tr>
    <td>add property</td>
    <td class="cell-negative">fails/throws</td>
    <td class="cell-negative">fails/throws</td>
    <td class="cell-negative">fails/throws</td>
  </tr>
  <tr>
    <td>delete property</td>
    <td class="cell-positive">OK</td>
    <td class="cell-negative">fails/throws</td>
    <td class="cell-negative">fails/throws</td>
  </tr>
  <tr>
    <td>reassign property</td>
    <td class="cell-positive">OK</td>
    <td class="cell-positive">OK</td>
    <td class="cell-negative">fails/throws</td>
  </tr>
</table>

### Inspecting Objects
We also have three inspection methods to determine what, if any, restrictions an object has.

- `Object.isFrozen`: returns `true` only if object has been put through `Object.freeze`, else `false`.

- `Object.isSealed`: returns `true` only if object has been passed to `Object.seal` _or_ `Object.freeze`, else `false`.

- `Object.isExtensible`: returns `true` if none of `preventExtensions`, `seal`, or `freeze` have been called on the object.

### Caveats
One massive caveat comes to mind: these methods only restrict the top-level properties of an object; they don't recursively operate on nested objects.

```javascript
(function () {
  'use strict';
  var obj = { prop: "val", subObj: {} };
  Object.freeze( obj );

  // doesn't throw!
  obj.subObj.newProp = "newVal";

})();
```
[MDN has an example](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) of recursively "deep freezing" an object so that it's completely immutable. Alternately, there's [a tiny Node module by Substack](https://github.com/substack/deep-freeze) which does the same thing, but is perhaps a little more robust.

### Use Cases

### Support
ES5 awesomeness is not supported everywhere, and these methods aren't <a href="https://github.com/es-shims/es5-shim">shim-able</a>. They're supported on all modern and modern-ish browsers, including:

- IE 9+
- FF 4+
- Safari 5.1+
- Chrome 6+
- Opera 12+
