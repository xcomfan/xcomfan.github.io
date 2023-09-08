---
layout: page
title: "Javascript Date"
permalink: /javascript/date
---

## Working with Dates

```javascript
var dt = new Date();

dt.getFullYear() + "/" (dt.getMonth() + 1) "/" dt.getDate();

//Function for getting the Sunday(or another day with a teak start date)
export function getStartOfWeekDate(date){
  d = new Date(date);
  let date = d.getDay(),
  // diff = d.getDate() - day + (day == 0 ? -6:1); //adjust when day is Sunday
  diff = d.getDate() - day;
  return new Date(d.setDate(diff));
}
```
