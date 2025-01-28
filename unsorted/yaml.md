---
layout: page
title: "YAML Reference"
permalink: /yaml/
---

## What is YAML

YAML is a [data serialization format](https://en.wikipedia.org/wiki/Comparison_of_data-serialization_formats). YAML is also a superset of JSON meaning that every valid JSON document also happens to be a YAML document. YAML puts more focus on human readability making it more applicable to configuration files edited by hand rather than as a transport layer.

YAML uses Python style block indentation (leading white space on each line defines the scope of a block)to define structure. This makes it easier to read, but you can't collapse whitespace to reduce size when transferring messages over a network.

The official YAML site is [yaml.org](https://yaml.org/)

## YAML Syntax

There are three fundamental data structures in YAML.

1. **Scalars** - Simple values like numbers, strings or booleans.

2. **Arrays** - Sequences of scalars or other collections

3. **Hashes** - Associated arrays, also known as maps, dictionaries, objects, or records comprised of key-value pairs.

| Data Type | YAML |
| --------- | ---- |
| Null | `null`, `~` |
| Boolean | `true`, `false` |
| Integer | `10`, `0b10`, `0x10`, `0o10` |
| Floating-Point | `3.14`, `12.5e-9`, `.inf`, `.nan` |
| String | `Lorem`, 'ipsum` |
| Date and Time | `2024-10-31`, `23:59`, `2022-01-16 23:59:59` |

Reserved words in YAML can be written in lowercase (`null`), uppercase (`NULL`) or title case (`Null`) any other case variant of such words will be treated as plain text.

Sequences in YAML are just like Python lists. They use the standard square bracket syntax `[]` if in the inline-block mode or leading `-` at the start of each line when they are block indented. You can keep your list items at the same indentation level as their property or add more indentation if that improves readability.

```yaml
fruits: [apple, banana, orange]
veggies:
  - tomato
  - cucumber
  - onion
mushrooms:
- champignon
- truffle
```

YAML has hashes analogous to Python dicts. They are made of keys (property names) followed by a colon (`:`) and a value.

Notice in the below example how the list of children contains **anonymous objects**. Unlike for example the spouse defined under a property named `spouse`. When anonymous or unnamed objects appear as list items, you can recognize them by their properties, which are aligned with an opening dash character (`-`)

```yaml
person:
  firstName: John
  lastName: Doe
  dateOfBirth: 1969-12-31
  married: true
  spouse:
    firstName: Jane
    lastName: Smith
  children:
    - fistName: Bobby
      dateOfBirth: 2001-05-14
    - firstName: Molly
      dateOfBirth: 1995-01-17
```

Property names in YAML are flexible. They can contain whitespace characters and span multiple lines. YAML hashes allow you to use almost any data type for a key.

## Unique Features

YAML understands various date and time formats, including the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard, and can work with optional time zones. Timestamps such as 23:59:59 get deserialized into the number of seconds elapsed since midnight.

To solve ambiguities you can cast values to specific data types by using **YAML tags** which start with a double exclamation point `!!`. There are a few [language-independent tags](https://yaml.org/type/index.html), but different parsers might provide additional extensions only relevant to your programming language.

```yaml
text: !!str 2022-01-16

numbers: !!set
    ? 5
    ? 8
    ? 13

image: !!binary
    R0lGODdhCAAIAPAAAAIGAfr4+SwAA
    AAACAAIAAACDIyPeWCsClxDMsZ3CgA7

```

`?` denotes a mapping key in YAML. They are usually unnecessary but can help you define a compound key from another collection or a key that contains reserved characters.

Another YAML feature is **anchors and aliases** which let you define an element once and then refer to it many times within the same document.  Possible use cases for this are

* Reusing the shipping address for invoicing
* Rotating meal in a meal plan
* Referencing exercises in a training program.

To declare an anchor which you can think of as a named variable you use the ampersand `&` symbol and you use the asterisk `*` to derefernce it.

```yaml
recursive: &cycle [*cycle]

exercises:
  - muscles: &push-up
      - pectoral
      - triceps
      - biceps
  - muscles: &squat
      - glutes
      - quadriceps
      - hamstrings
  - muscles: &plank
      - abs
      - core
      - shoulders

schedule:
  monday:
    - *push-up
    - *squat
  tuesday:
    - *plank
  wednesday:
    - *push-up
    - *plank
```

The `recursive` above demonstrates a circular reference. This property is a sequence whose only element is the sequence itself. In other words `recursive[0]` is the same as `recursive` This feature allows you to represent directed graphs in YAML.

You can also **merge** or override attributes defined elsewhere by combining two or more objects.

In the example below the `rectangle` object inherits properties of share and square while adding a new attribute `b`, and changing the value of `color`

```yaml
shape: &shape
  color: blue

square: &square
  a: 5

rectangle:
  << : *shape
  << : *square
  b: 3
  color: green
```

Scalars in YAML support either a **flow styel** or a **block style**, which give you different levels of control over the newline handling in multiline strings. Flow scalars can start on the same line as their property name and may span multiple lines. In such a case, each line's leading and trailing whitespace will always be folded into a single space, turning paragraphs into lines.

```yaml
text: Lorem ipsum
   dolor sit amet

   Lorem ipsum
   dolor sit amet
```

The above YAML code would produce the below text:

```text
Lorem ipsum dolor sit amet
Lorem ipsum dolor sit amet
```

In contrast to flow calars, block scalars allow for changing how to deal with newlines, trailing newlines, or indentation. Fore example, the pipe (`|`) indicator placed right after the property name preserves the newlines literally, which can be handy for embedding shell scripts in your YAML file.

```yaml
script: |
   #!/usr/bin/env python

   def main():
       print("Hello world")

   if __name__ == "__main__":
       main()
```

The YAML document above defines a property named `script`, which holds a short Python script. Without the pipe indicator, a YAML parser would've treated the following lines as nested elements rather than as a whole. Ansible frequently takes advantage of this feature.

If you want to only fold lines with indentation determined by the first line in a paragraph, then use the greater than sign (`>`) indicator:

```yaml
 text: >
   Lorem
     ipsum
   dolor
   sit
   amet

   Lorem ipsum
   dolor sit amet
```

The above YAML would produce text that looks like.

```text
Lorem
  ipsum
dolor sit amet
Lorem ipsum dolor sit amet
```

Finally you can have multiple YAML documents stored in a single file separated with the triple dash (`---`). You can optionally use the triple dot (`...`) to mark the end of each document.