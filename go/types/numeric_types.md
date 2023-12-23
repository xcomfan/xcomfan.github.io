---
layout: page
title: "Go Numeric Types"
permalink: /go/types/numeric_types
---

## Integer types

| Type Name | Min Value | Max Value |
| --------- | --------- | --------- |
| int8 | -128 | 127 |
| int16 | -32,768 | 32,767 |
| int32 | -2,147,483,648 | 2,147,483,648 |
| int64 | -9,223,372,036,854,775,808 | -9,223,372,036,854,775,808 |
| uint8 | 0 | 255 |
| uint16 | 0 | 65,535 |
| uint32 | 0 | 4,294,967,293 |
| uint64 | 0 | 18,446,744,073,709,551,615 |

## Floating point types

| Type Name | Number of Bits | Number of Decimal Digits |
| float32 | 32 | ~7 |
| float64 | 64 | ~15 |

## Special types

In Go `byte`, `int`, and `uint` are aliases for the specific types to provide a shorthand.  The 32 versus 64 is determined by your CPU architecture.

| Type Name | Same as |
| --------- | ------- |
| byte | uint8 |
| int | int32 or int64 |
| uint | uint32 or uint64 |


## Zero value for integers

The zero value for integer types is zero `0`
