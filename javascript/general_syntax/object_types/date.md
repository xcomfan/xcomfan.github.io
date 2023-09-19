---
layout: page
title: "Javascript Date"
permalink: /javascript/date
---



## Working with Dates

```javascript
let now = new Date();
let then = new Date(2010,0,1); // the first day of the first month of 2010
let later = new Date(2010,0,1,17,10,30); // same day at 5:30pm
elapsed = now - then;
later.getFullYear() // 2010
later.getMonth() // 0: 0 based months
later.getDate() // 1: 1 based dates
later.getHours() // // 17:15 local time
later.getUTCHours() // hours in UTC will depend on timezone
later.toString() // "Fri Jan 01 2010 17:10:30 GMT-0900 (PST)"
later.toLocaleDateString() // "01/01/2020"
later.toLocaleTime() // 05:10:30"
```


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
