---
layout: page
title: "Kubernetes Naming Constraints"
permalink: /kubernetes/naming_constraints
---

The key names for data items inside of a Secret or ConfigMap are defined to map to valid environment variable names. They may begin with a dot followed by a letter or number. Following characters include dots, dashes, and underscores. Dots cannot be repeated and dots and underscores or dashes cannot be adjacent to each other. The below regular expression checks for valid names.

`^[.[?[a-zAZ0-9[([.[?[a-zA-Z0-9[+[-_a-zA-Z0-9[?)*$`

Below are some examples of valid and invalid names.

| Valid key name | Invalid key name |
| -------------- | ---------------- |
| `.auth_token` | `Token..properties`|
| `Key.pem` | `auth file.json` |
| `config_file` | `_password.txt` |

When selecting a key name, consider that these keys can be exposed to Pods via a volume mount. Pick a name that is going to make sense when specified on a command line or in a config file. Storing a TLS key as key.pem is more clear than `tls-key` when configuring applications to access secrets.
