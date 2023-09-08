---
layout: page
title: "ES6 Importing and Exporting Code Blocks"
permalink: /javascript/imports
---

[comment]: <> (TODO: Need to run though this section and make corrections if needed based on testing it myself.)

## Exporting

In order to use a function defined in one file to be used in another you need to export it.  To export a single function...

```javascript
export const add = (x, y) => {
  return x + y;
}
```

If you need to export multiple functions...

```javascript
const add = (x,y) => {
  return x + y;
}

const subtract = (x,y) => {
  return x - y;
}

export{ add, subtract };
```

You can create an export fallback with export default.  This is usually done if one value is being exported from a file or to create a fallback value for file or module.  Your are not limited to functions with exporting.  You an export variables declared via `var`, `let` or `const` as well.

```javascript
// default export function
export default function(x, y){
  return x + y
}
```

When using export default on the import side you just specify the file not what you are importing.  In the example below you are naming the function being imported add.

```javascript
import add from "./math_functions.js"
```

## Importing

You can import specific exports.

```javascript
import { add } from './math_functions.js';
import { add, subtract } from './math_functions.js';
```

You can use `import * as` syntax to import all contents of a file.

```javascript
import * as myMathModule from "./math_functions.js";

myMathModule.add(2,3);
myMathModule.subtract(5,3);
```
