---
layout: page
title: "Working With Files And Directories"
permalink: /python/files_and_dirs
---

[comment]: <> (TODO: Make this a part of standard library reference)

## Traversing directories

### Custom traversal

```python
import os

SKIP = {".git"}

def recursively_print_files(path: str, sep=0):
    try:
        files = os.listdir(path)
        for file in files:
            if file in SKIP:
                continue
            file_path = os.path.join(path,file)
            spacing = ' ' * sep
            if os.path.islink(file_path):
                print(f"{spacing}l: {file}")
            elif os.path.isdir(file_path):
                print(f"{' ' * sep}d: {file}")
                recursively_print_files(file_path, sep+2)
            else:
                print(f"{' '* sep }f: {file}")
    except FileNotFoundError:
        print(f"Could not find: {path}")

if __name__ == "__main__":
    recursively_print_files("/home/ubuntu/practice_")
```

### Using os.walk

```python
def os_walk_demo(path: str):
    for root, dirs, files in os.walk(path):
        for file in files:
            print(os.path.join(root, file))

'''
for each directory under path argument root will be a directory and dirs and files will be the names of the dirs and files in that path.
'''
```
