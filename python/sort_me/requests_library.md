---
layout: page
title: "Requests Library"
permalink: /sort_me/requests_library
---

## Requests Library is not have Async support.

If you need async support check out [AIOHTTP](https://docs.aiohttp.org/en/stable/) or [HTTPX](https://www.python-httpx.org/async/)

HTTPX is broadly compatible with Requests syntax.

[comment]: <> (TODO: Use HTTPX docs to reproduce everything you are doing in this tutorial in a HTTPX doc.)

## Getting Started

Because requests is a third party library you need to install it.

`python -m pip install requests`

## Response Object

### Status code checking and handling

If you use a Response instance in a conditional expression, then it will evaluate to `True` if the status code was smaller than 400.

```python
if response:
    print("Success!")
else:
    raise Exception(f"Non-success status code: {response.status_code})
```

Keep in mind that this approach does not check for a `200` status code. `204` NO CONTENT and `304` NOT MODIFIED are also considered successful.

You can also use Requests built in `raise_for_status()` To raise an exception if a request was unsuccessful.

```python
import requests
from requests.exceptions import HTTPError

URLS = ["https://api.github.com", "https://api.github.com/invalid"]

for url in URLS:
    try:
        response = requests.get(url)
        response.raise_for_status()
    except HTTPError as http_err:
        print(f"HTTP error occurred: {http_err}")
    except Exception as err:
        print(f"Other error occurred: {err}")
    else:
        print("Success!")
```

### Content

To see the content of a response in raw bytes use `.content`.

To get the text version use `.text`. Because the decoding of bytes to a string requires an encoding scheme; Requests will try to guess the encoding based on the headers in the response. You can provide an explicit encoding by setting `.encoding` before accessing `.text`.

If the response is in JSON format you can load it to a dictionary with the `.json()` method.

```python
>>> import requests
response = requests.get("https://api.github.com")
>>> response.content
b'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url":...'
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url...'
>>> response.econding = "utf-8"
>>> response.text
'{"current_user_url":"https://api.github.com/user","current_user_authorizations_html_url...'
>>> response.json()
{'current_user_url': 'https://api.github.com/user', 'current_user_authorizations_html_url': ...'}
>>> type(response.json())
<class 'dict'>
```

### Response headers

You can access the headers of a response with `.headers` which returns a dictionary like object. Because the HTTP specification defines headers as case insensitive this dictionary object is not case sensitive for indexing by key.

```python
>>> import requests

>>> response = requests.get("https://api.github.com")
>>> response.headers
{'Server': 'GitHub.com',
...
'X-GitHub-Request-Id': 'AE83:3F40:2151C46:438A840:65C38178'}
>>>
>>> response.headers["Content-Type"]
'application/json; charset=utf-8'
>>>
>>> response.headers["content-type"]
'application/json; charset=utf-8'
>>>
```

## Query String Parameters

Query string parameters are the arguments passed inside a URL. You set those for your request by passing a `data` parameter.

The example below we use query string parameters in calling GitHub search API.

```python
import requests

response = requests.get(
    "https://api.github.com/search/repositories",
    params={"q": "language:python", "sort": "stars", "order": "desc"}
    )

json_resonse = response.json()
popular_repositories = json_resonse["items"]
for repo in popular_repositories[:3]:
    print(f"Name: {repo['name']}")
    print(f"Description: {repo['description']}")
    print(f"Stars: {repo['stargazers_count']}")
    print()
```

You can even pass the `data` values as bytes

```python
>>> requests.get(
...     "https://api.github.com/search/repositories",
...     params=b"q=language:python&sort=stars&order=desc",
... )
<Response [200]>
```

## Request Headers

To customize the headers sent with your request use the `headers` parameter.

```python
import requests

response = requests.get(
    "https://api.github.com/search/repositories",
    params={"q": '"real python"'},
    headers={"Accept": "application/vnd.github.text-match+json"},
)

# View the new `text-matches` list which provides information
# about your search term within the results
json_response = response.json()
first_repository = json_response["items"][0]
print(first_repository["text_matches"][0]["matches"])
```

## Other HTTP Methods

The authors of the requests library created [httpbin.org](https://httpbin.org) which is a service that accepts request and provides responses for practicing/demos. We will be using it here for our examples.

```python
>>> import requests

>>> requests.get("https://httpbin.org/get")
<Response [200]>
>>> requests.post("https://httpbin.org/post", data={"key": "value"})
<Response [200]>
>>> requests.put("https://httpbin.org/put", data={"key": "value"})
<Response [200]>
>>> requests.delete("https://httpbin.org/delete")
<Response [200]>
>>> requests.head("https://httpbin.org/get")
<Response [200]>
>>> requests.patch("https://httpbin.org/patch", data={"key": "value"})
<Response [200]>
>>> requests.options("https://httpbin.org/get")
<Response [200]>
```

***Note:*** All of the above calls are high level shortcuts to `requests.request()` passing the name of the relevant HTTP method.

```python
>>> requests.request("GET", "https://httpbin.org/get")
<Response [200]>
```

## The Message Body

According to the HTTP specification, POST, PUT, and the less common PATCH requests pass their data through the message body rather than through parameters in the query string. Use the `data` parameter of requests to pass the payload. `data` takes a dictionary, a list of tuples, bytes, or a file like object. You will need to adapt the `data` being sent by your request to the service you are dealing with.

```python
>>> import requests

>>> requests.post("https://httpbin.org/post", data={"key": "value"})
<Response [200]>
```

You can also send the same data as a list of tuples

```python
>>> requests.post("https://httpbin.org/post", data=[("key", "value")])
<Response [200]>
```

If you need to send JSON data, then you can use the `json` parameter. Requests will serialize your data and add the Content-Type header for you.

```python
>>> response = requests.post("https://httpbin.org/post", json={"key": "value"})
>>> json_response = response.json()
>>> json_response["data"]
'{"key": "value"}'
>>> json_response["headers"]["Content-Type"]
'application/json'
```

## Request Inspection

When you a request, the Requests library prepares the request before actually sending it to the destination server. Request preparation includes things such as serializing JSON content and validating headers. You can view the `PreparedRequest` by accessing the `.request` on a Response Object.

```python
>>> import requests

>>> response = requests.post("https://httpbin.org/post", json={"key":"value"})

>>> response.request.headers["Content-Type"]
'application/json'
>>> response.request.url
'https://httpbin.org/post'
>>> response.request.body
b'{"key": "value"}'
```

## Authentication

Typically you authenticate by passing the `Authorization` header or a custom header defined by the service. All the functions of Requests provide a parameter `auth` for passing credentials.

```python
>>> import requests

>>> response = requests.get(
...     "https://httpbin.org/basic-auth/user/passwd",
...     auth=("user", "passwd")
... )

>>> response.status_code
200
>>> response.request.headers["Authorization"]
'Basic dXNlcjpwYXNzd2Q='
```

Requests does the work of serializing the authentication to the HTTP basic authentication scheme. You could make the equivalent call by using `HTTPBasicAuth`

```python
>>> from requests.auth import HTTPBasicAuth
>>> requests.get(
...     "https://httpbin.org/basic-auth/user/passwd",
...     auth=HTTPBasicAuth("user", "passwd")
... )
<Response [200]>
```

Requests provides other authentication options such as `HTTPDigestAuth` and `HTTPProxyAuth`

### Authenticating with a Bearer Token

Below is a bad way of authenticating with a token that works. The reasons why its bad is 1. Your token is being sent with HTTP basic encryption which is not secure, and 2. using an empty string is input looks a bit awkward.

```python
>>> import requests

>>> token = "<YOUR_GITHUB_PA_TOKEN>"
>>> response = requests.get(
...     "https://api.github.com/user",
...     auth=("", token)
... )
>>> response.status_code
200
```

A better way is to create a subclass of `AuthBase` and implement `.__call__()`.

```python
from requests.auth import AuthBase

class TokenAuth(AuthBase):
    """Implements a token authentication scheme."""

    def __init__(self, token):
        self.token = token

    def __call__(self, request):
        """Attach an API token to the Authorization header."""
        request.headers["Authorization"] = f"Bearer {self.token}"
        return request
```

```python
>>> import requests
>>> from custom_token_auth import TokenAuth

>>> token = "<YOUR_GITHUB_PA_TOKEN>"
>>> response = requests.get(
...     "https://api.github.com/user",
...     auth=TokenAuth(token)
... )

>>> response.status_code
200
>>> response.request.headers["Authorization"]
'Bearer ghp_b...Tx'
```

For OAuth authentication look into [Oauth](https://docs.python-requests.org/en/latest/user/authentication/#oauth-1-authentication) and [Requests-OAuthlib](https://requests-oauthlib.readthedocs.io/en/latest/)

## SSL Certificate Verification

If you want to disable SSL certificate verification pass `False` to `verify` parameter.

Requests uses a package called [certifi](https://requests.readthedocs.io/en/latest/user/advanced/#ca-certificates) to provide certificate authorities.

```python
>>> import requests

>>> requests.get("https://api.github.com", verify=False)
InsecureRequestWarning: Unverified HTTPS request is being made to host
⮑ 'api.github.com'. Adding certificate verification is strongly advised.
⮑ See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#tls-warnings
⮑  warnings.warn(
<Response [200]>
```

## Performance

### Timeouts

By default Requests will wait indefinitely on the response, so you should almost always specify a timeout duration to prevent these issues from happening. 

```python
>>> requests.get("https://api.github.com", timeout=1)
<Response [200]>
>>> requests.get("https://api.github.com", timeout=3.05)
<Response [200]>
```

You can also pass a tuple to `timeout` with 1. a connection timeout (the time it allows to establish a connection), and 2. a read timeout (the time it will wait on a response once your client has established a connection).

```python
>>> requests.get("https://api.github.com", timeout=(3.05, 5))
<Response [200]>
```

### The `Session` Object

High level `requests` APIs such as `get()` and `post()` are based on a class called `Session`. If you need to fine-tune your control over how your requests are being made you may need to use the `Session` object.

Sessions are used to persist parameter across requests. For example, if you want to use the same authentication across multiple requests, then you can use a session. The primary performance improvement here is using the same session you don't have to re-establish a session with each called instead it is kep in a connection pool and re-used.

```python
import requests
from custom_token_auth import TokenAuth

TOKEN = "<YOUR_GITHUB_PA_TOKEN>"

with requests.Session() as session:
    session.auth = TokenAuth(TOKEN)

    first_response = session.get("https://api.github.com/user")
    second_response = session.get("https://api.github.com/user")

print(first_response.headers)
print(second_response.json())
```

### Max Retries

Requests won't do retries by default when a request fails. To apply this functionality you need to implement a custom [`transport adapter`](https://requests.readthedocs.io/en/latest/user/advanced/#transport-adapters). Transport adapters let you define a set of configurations for each service that you are working with. For example if you want requests to `https://api.github.com` to retry two tmes before finally raising a `RetryError` you would build a transport adapter, set its `max_retries` parameter, and mount it to an existing `Session`:

```python
import requests
from requests.adapters import HTTPAdapter
from requests.exceptions import RetryError

github_adapter = HTTPAdapter(max_retries=2)

session = requests.Session()

session.mount("https://api.github.com", github_adapter)

try:
    response = session.get("https://api.github.com/")
except RetryError as err:
    print(f"Error: {err}")
finally:
    session.close()
```

A better version of the example above below uses the `urllib3.util.Retry` library to further customize the retry functionality and adds logging so you can monitor when retries happen.

```python
import logging

import requests
from requests.adapters import HTTPAdapter
from requests.exceptions import RetryError
from urllib3.util import Retry

logging.basicConfig(level=logging.DEBUG)

retries = Retry(total=3, status_forcelist=[404])

github_adapter = HTTPAdapter(max_retries=retries)

session = requests.Session()

session.mount("https://api.github.com", github_adapter)

try:
    response = session.get("https://api.github.com/")
    print(response.status_code)
    response = session.get("https://api.github.com/invalid")
    print(response.status_code)
except RetryError as err:
    print(f"Error: {err}")
finally:
    session.close()
```
