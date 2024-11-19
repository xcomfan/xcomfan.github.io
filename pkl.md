---
layout: page
title: "Pkl Reference"
permalink: /pkl/
---

## Output Pkl Code to YAML or JSON or Java Property List

Basic input file.

```pkl
// intro.pkl
name = "Pkl: Configure your Systems in New Way"
attendants = 100
isInteractive = true
amountLearned = 13.37
```

Can convert to yaml or json output

```text
ubuntu@ubuntu:~/pkl_practice$ pkl eval -f yaml ./intro.pkl
name: 'Pkl: Configure your Systems in New Way'
attendants: 100
isInteractive: true
amountLearned: 13.37
ubuntu@ubuntu:~/pkl_practice$ pkl eval -f json ./intro.pkl
{
  "name": "Pkl: Configure your Systems in New Way",
  "attendants": 100,
  "isInteractive": true,
  "amountLearned": 13.37
}
ubuntu@ubuntu:~/pkl_practice$ pkl eval -f plist ./intro.pkl
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>name</key>
  <string>Pkl: Configure your Systems in New Way</string>
  <key>attendants</key>
  <integer>100</integer>
  <key>isInteractive</key>
  <true/>
  <key>amountLearned</key>
  <real>13.37</real>
</dict>
</plist>
```

## Pkl Structure: Classes, Objects and Modules

Typically a configuration needs some kind of hierarchical structure. Pkl provices *immutable* objects for this. Objects have three kinds of members: properties, elements and entries.

### Properties

```pkl
// simpleObjectWithProperties.pkl
bird {
  name = "Common wood pigeon"
  diet = "Seeds"
  taxonomy {
    species = "Columba palumbus"
  }
}
```

`bird{...` defines bird to be an **object**

For **primatives** Pkl uses the `=` syntax

`taxonomy {...` shows that objects can be nested

### Elements

You don't alwasy have names for every individual structure in your configuration. Sometimes you need "a bunch of things" without knowing how many. Pkl provides **elements** for this purpose. Elements are object members just like **properties** but can be indexed by a number. You can think of an object that only contains **elements** as an array. Similar to arrays you can use square brackets to access an element, for example `myObject[42]`.

```pkl
exampleObjectWithJustIntElements {
  100
  42
}

exampleObjectWithMixedElements {
  "Bird Breeder Conference"
  (2000 + 23)
  exampleObjectWithJustIntElements
}
```

The above Pkl code yeilds the below JAON and YAML when run though `pkl eval -f yaml|json`.

The `100` in `exampleObjectWithJustIntElements` demonstrates how when you just write a value without an name you make it an element. Elements don't have to be primatives, they can be anything as demonstrated in `(200 + 23)` and using `exampleObjectWithJustIntElements` inside `exampleObjectWithMixedElements`.

YAYML output

```yaml
exampleObjectWithJustIntElements:
- 100
- 42
exampleObjectWithMixedElements:
- Bird Breeder Conference
- 2023
- - 100
  - 42
```

JSON output
```json
{
  "exampleObjectWithJustIntElements": [
    100,
    42
  ],
  "exampleObjectWithMixedElements": [
    "Bird Breeder Conference",
    2023,
    [
      100,
      42
    ]
  ]
}
```

### Entries

Like a **property** and **entry** is named (technically keyed) but unlike a property the name does not need to be knowns at declaration time. To differentiate **entries** from **properties** the "name" for an entry is enclosed in square brackets `[]`. Thae name does not need to be a string; any value can index an entry.

```pkl
// simpleObjectsWithEntries.pkl

pigeonShelter {
  ["bird"] {
    name = "Common wood pigeon"
    diet = "Seeds"
    taxonomy {
      species = "Columba palumbus"
    }
  }
  ["address"] = "355 Bird St."
}

birdCount {
  [pigeonShelter] = 42
}
```

The difference with **properties** is the notation of the `key: [<expression>]`. As with **properties**, entries can be primitive values or objects. Any object can be used as a key for an entrty as we see when we use `pigeonShelter` as a key in `birdCount`.

### Mixed Members

You can mix **properties**, **elements** and **entries** in a single object. You don't have to order them by kind, and you don't need special syntax. This can be seen in how the properties `name`, `lifespan` and `extinct` are intermixed with `"wing"`, `"claw"` and `42` along with entries `"wing"` and `false`.

```pkl
mixedObject {
  name = "Pigeon"
  lifespan = 8
  "wing"
  "claw"
  ["wing"] = "Not related to the _element_ \"wing\""
  42
  extinct = false
  [false] {
    description = "Construed object example"
  }
}
```

#### Collections

This free for all object member mixing can be confusing and target formats are not as flexible. For this reason Pkl add two sepcial types of object **listings** which exclusively contain elements, and **mappings** which contain exlusively entries.  Both **listings** and **mappings** are "just objects" so they don't require special syntax other than that of objects.

```pkl
// collections.pkl

birds {
  "Pigeon"
  "Parrot"
  "Barn owl"
  "Falcon"
}

habitats {
  ["Pigeon"] = "Streets"
  ["Parrot"] = "Parks"
  ["Barn owl"] = "Forests"
  ["Falcon"] = "Mountains"
}
```

Below we have an example of a **listing** in `birds` and an examle of a **maping** in `habitats`. This can be rendered as a JSON of YAML and you get the JSON below. Notice thatthe listing gets rendered as a JSON array. 

```json
{
  "birds": [
    "Pigeon",
    "Parrot",
    "Barn owl",
    "Falcon"
  ],
  "habitats": {
    "Pigeon": "Streets",
    "Parrot": "Parks",
    "Barn owl": "Forests",
    "Falcon": "Mountains"
  }
}
```

You can access the members of a listing by zero indexing into them as in the below exaple.

```pkl
birds {
  "Pigeon"
  "Parrot"
  "Barn owl"
  "Falcon"
}
relatedToSnowOwl = "Barn owl"
```

which would yield the JSON below.

```json
birds {
  "Pigeon"
  "Parrot"
  "Barn owl"
  "Falcon"
}
relatedToSnowOwl = "Barn owl"
```

## Composing Configurations

The central mechanism in Pkl for expressing one part of a configuration int terms of another is **amending**.

```pkl
// amendingObjects.pkl

bird {
  name = "Pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Columbiformes"
  }
}

parrot = (bird) {
  name = "Parrot"
  diet = "Berries"
  taxonomy {
    order = "Psittaciformes"
  }
}
```

In the above example Parrot and Pigeon have nearly identical properties; they only differ in their name and taxonomy. Thus you can use the `parrot = (birt)...` syntax to say "Parrot is just like `bird` except..." The above Pkl produced the below JSON.

```json
bird {
  name = "Pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Columbiformes"
  }
}
parrot {
  name = "Parrot"
  diet = "Berries"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    order = "Psittaciformes"
  }
}
```

***Note:*** So far we just worked with **Dynamic** objects. Pkl also offers **Typed** objects. Amending allows you to override, amend, and add new properties to a dynamic object. Typed objects will only let you amend or override existing properties, not add entirely new ones.

You can alos amend nested objects. This allow you to only describe the difference wit the outermost object for arbitrariy deeply nested structures.

```pkl
// nestedAmends.pkl

stockPigeon {
  name = "Stock pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Columbimorphae"
    order = "Columbiformes"
    species = "Columba oenas"
  }
}

woodPigeon = (stockPigeon) {
  name = "Common wood pigeon"
  taxonomy {
    species = "Columba palumbus"
  }
}

dodo = (woodPigeon) {
  name = "Dodo"
  extinct = true
  taxonomy {
    species = "Raphus cucullatus"
  }
}
```

In `woodPigeon` we amend species as it occurs in `stockPigeon`. As we see later in `dodo` amended objects can themswlves be amended. `extinct` in `dodo` shows us that new fields can be added to new objects when amending.

The above Pkl codew generates the below JSON.

```json
stockPigeon {
  name = "Stock pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Columbimorphae"
    order = "Columbiformes"
    species = "Columba oenas"
  }
}
woodPigeon {
  name = "Common wood pigeon"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Columbimorphae"
    order = "Columbiformes"
    species = "Columba palumbus"
  }
}
dodo {
  name = "Dodo"
  diet = "Seeds"
  taxonomy {
    kingdom = "Animalia"
    clade = "Columbimorphae"
    order = "Columbiformes"
    species = "Raphus cucullatus"
  }
  extinct = true
}
```

So far we have only amended **properties** which are simple since you refer to them by name. 

```pkl
favoriteFoods {
  "red berries"
  "blue berries"
  ["Barn owl"] {
    "mice"
  }
}

adultBirdFoods = (favoriteFoods) {
  [1] = "pebbles" 
  "worms" // Without explicit indices, Pkl can't know which element to overwrite, so, instead, it adds an element to the object you're amending.
  ["Falcon"] {
    "insects"
    "amphibians"
  }
  ["Barn owl"] {
    "fish"
  }
}
```

As we see in `[1] = "pebbles"` explicitly amending by index replaces the elements at that index. Without an explicity index Pkl can't know which elment to overwrite so instead ita adds the element to the object being amended as we see with `worms`.  When you write "new" entries with a key that is not in object being amended such as `["Falcon"]` Pkl also adds them. When you write an entry using a key that exists, this notation amends its value as we see with `["Bgarn own"] { "fish" }`.

## Modules

A `.pkl` file describes a **module**. Modules are objects that can be referred to from other modules. Thus you can do something like.

```pkl
// pigeon.pkl

name = "Common wood pigeon"
diet = "Seeds"
taxonomy {
  kingdom = "Animalia"
  clade = "Dinosauria"
  species = "Columba palumbus"
}
```

and in a different file (module)

```pkl
// parrot.pkl

import "pigeon.pkl"

parrot = (pigeon) {
  name = "Great green macaw"
  diet = "Berries"
  taxonomy {
    species = "Ara ambiguus"
  }
}
```

evaluating each you get the following.

```bash
$ pkl eval /Users/me/tutorial/pigeon.pkl
name = "Common wood pigeon"
diet = "Seeds"
taxonomy {
  kingdom = "Animalia"
  clade = "Dinosauria"
  species = "Columba palumbus"
}

$ pkl eval /Users/me/tutorial/parrot.pkl
parrot {
  name = "Great green macaw"
  diet = "Berries"
  taxonomy {
    kingdom = "Animalia"
    clade = "Dinosauria"
    species = "Ara ambiguus"
  }
}
```

Notice that `pigeon` is "spread" in the top level, while `parrot` is a nested and named object. This is because writing `parrot {...}` defines an object property in the "current" modules.

In order to say that "this module is an object, amended from `pigeon` module" you use the `amends` keyword.

```pkl
amends "pigeon.pkl" // "This" moulde is the same as "pigeon.pkl", except for what is in the remainder of the file.

name = "Great green macaw"
```

Think of "amending a module" as "filling out a form"

## Amending Templates

A Pkl file can be a **template** or a "normal" **module**. This terminology describes the intended use of the module. You can't tell from looking at the Pkl code, if somethign is a normal **modules** or a **template**.

```pkl
// AcmeCICD.pkl

module AcmeCICD

class Pipeline {
  name: String(nameRequiresBranchName)?

  hidden nameRequiresBranchName = (_) ->
      if (branchName == null)
        throw("Pipelines that set a 'name' must also set a 'branchName'.")
      else true

  branchName: String?
}

timeout: Int(this >= 3)

pipelines: Listing<Pipeline>

output {
  renderer = new YamlRenderer {}
}
```

Thus to use the above pipeline your "filling out of the form" would look like...

```pkl
amends "AcmeCICD.pkl"

amends "AcmeCICD.pkl"

timeout = 3
pipelines {
  new {
    name = "prb"
    branchName = "main"
  }
}
```

Because there is no pipeline object to amend; the `new` keyword gives you an object to amend. There is no object to amend. Writing `myNewPipeline { ... }` defines a property, but listings may only include elements. This is where you use the keyword `new`. `new` gives you an object to amend. Pkl derives from context which `new` is used and what the object to amend should look like. This is called default value for the context. 