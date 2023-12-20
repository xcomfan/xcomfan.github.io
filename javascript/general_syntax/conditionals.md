---
layout: page
title: "JavaScript Conditionals"
permalink: /javascript/data_types
---

### if

```javascript
if (username === null) username = "John Doe";

if (!address){
    address = "";
    message = "Please specify a mailing address.";
}
```

### if else

```javascript
if (hour < 18) {
  greeting = "Good day";
} else {
  greeting = "Good evening";
}
```

### ternary operator

```javascript
const age = 18;
const message = age >= 18 ? "You are an adult" : "You are not an adult";
console.log(message);
```

### switch

```javascript
switch (new Date().getDay()) {
  case 0:
    day = "Sunday";
    break;
  case 1:
    day = "Monday";
    break;
  case 2:
     day = "Tuesday";
    break;
  case 3:
    day = "Wednesday";
    break;
  case 4:
    day = "Thursday";
    break;
  case 5:
    day = "Friday";
    break;
  case 6:
    day = "Saturday";
}
```
