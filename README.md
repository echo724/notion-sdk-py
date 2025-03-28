<!-- markdownlint-disable -->
![notion-sdk-py](https://socialify.git.ci/ramnes/notion-sdk-py/image?descriptionEditable=&font=Bitter&language=1&logo=https%3A%2F%2Fwww.notion.so%2Fimage%2Fhttps%253A%252F%252Fs3-us-west-2.amazonaws.com%252Fsecure.notion-static.com%252F6de57ff9-a9eb-41d6-914c-597fe52a7294%252Fnotion-logo-no-background.png%3Ftable%3Dblock%26id%3De040febf-70a9-4950-b862-0e6f00005004%26spaceId%3De12b42ac-4e54-476f-a4f5-7d6bdb1e61e2%26width%3D200&owner=1&pattern=Circuit%20Board&theme=Light)

<div align="center">
        <a href="https://pypi.org/project/notion-client"><img src="https://img.shields.io/pypi/v/notion-client.svg" alt="PyPI"></a>
        <a href="tox.ini"><img src="https://img.shields.io/pypi/pyversions/notion-client" alt="Supported Python Versions"></a>
        <a href="LICENSE"><img src="https://img.shields.io/github/license/ramnes/notion-sdk-py" alt="License"></a>
        <a href="https://github.com/ambv/black"><img src="https://img.shields.io/badge/code%20style-black-black" alt="Code style"></a>
        <br/>
        <a href="https://github.com/ramnes/notion-sdk-py/actions/workflows/quality.yml"><img src="https://github.com/ramnes/notion-sdk-py/actions/workflows/quality.yml/badge.svg" alt="Code Quality"></a>
        <a href="https://github.com/ramnes/notion-sdk-py/actions/workflows/test.yml"><img src="https://github.com/ramnes/notion-sdk-py/actions/workflows/test.yml/badge.svg" alt="Tests"></a>
        <a href="https://github.com/ramnes/notion-sdk-py/actions/workflows/docs.yml"><img src="https://github.com/ramnes/notion-sdk-py/actions/workflows/docs.yml/badge.svg" alt="Docs"></a>
    </p>
</div>
<!-- markdownlint-enable -->

This client is meant to be a Python version of the reference [JavaScript SDK](https://github.com/makenotion/notion-sdk-js),
so usage should be pretty similar between both. 😊

> 📢 **Announcement** (14-08-2021) — 0.6.0 is now released and adds support for
> the [recent Notion API changes](https://developers.notion.com/changelog).
> Upgrading should be seamless from 0.4.0 onwards.

<!-- markdownlint-disable -->
## Installation
<!-- markdownlint-enable -->
```shell
pip install notion-client
```

## Usage

> Before getting started, [create an integration](https://www.notion.com/my-integrations)
> and find the token.
> [→ Learn more about authorization](https://developers.notion.com/docs/authorization).

Import and initialize a client using an **integration token** or an
OAuth **access token**.

```python
import os
from notion_client import Client

notion = Client(auth=os.environ["NOTION_TOKEN"])
```

In an asyncio environment, use the asynchronous client instead:

```python
from notion_client import AsyncClient

notion = AsyncClient(auth=os.environ["NOTION_TOKEN"])
```

Make a request to any Notion API endpoint.

> See the complete list of endpoints in the [API reference](https://developers.notion.com/reference).

```python
from pprint import pprint

list_users_response = notion.users.list()
pprint(list_users_response)
```

or with the asynchronous client:

```python
list_users_response = await notion.users.list()
pprint(list_users_response)
```

This would output something like:

```text
{'results': [{'avatar_url': 'https://secure.notion-static.com/e6a352a8-8381-44d0-a1dc-9ed80e62b53d.jpg',
              'id': 'd40e767c-d7af-4b18-a86d-55c61f1e39a4',
              'name': 'Avocado Lovelace',
              'object': 'user',
              'person': {'email': 'avo@example.org'},
              'type': 'person'},
             ...]}
```

All API endpoints are available in both the synchronous and asynchronous clients.

Endpoint parameters are grouped into a single object. You don't need to remember
which parameters go in the path, query, or body.

```python
my_page = notion.databases.query(
    **{
        "database_id": "897e5a76-ae52-4b48-9fdf-e71f5945d1af",
        "filter": {
            "property": "Landmark",
            "text": {
                "contains": "Bridge",
            },
        },
    }
)
```

### Handling errors

If the API returns an unsuccessful response, an `APIResponseError` will be raised.

The error contains properties from the response, and the most helpful is `code`.
You can compare `code` to the values in the `APIErrorCode` object to avoid
misspelling error codes.

```python
import logging
from notion_client import APIErrorCode, APIResponseError

try:
    my_page = notion.databases.query(
        **{
            "database_id": "897e5a76-ae52-4b48-9fdf-e71f5945d1af",
            "filter": {
                "property": "Landmark",
                "text": {
                    "contains": "Bridge",
                },
            },
        }
    )
except APIResponseError as error:
    if error.code == APIErrorCode.ObjectNotFound:
        ...  # For example: handle by asking the user to select a different database
    else:
        # Other error handling code
        logging.exception()
```

### Logging

The client emits useful information to a logger. By default, it only emits warnings
and errors.

If you're debugging an application, and would like the client to log request & response
bodies, set the `log_level` option to `logging.DEBUG`.

```python
notion = Client(
    auth=os.environ["NOTION_TOKEN"],
    log_level=logging.DEBUG,
)
```

You may also set a custom `logger` to emit logs to a destination other than `stdout`.
Have a look at [Python's logging cookbook](https://docs.python.org/3/howto/logging-cookbook.html)
if you want to create your own logger.

### Client options

`Client` and `AsyncClient` both support the following options on initialization.
These options are all keys in the single constructor parameter.

<!-- markdownlint-disable -->
| Option | Default value | Type | Description |
|--------|---------------|---------|-------------|
| `auth` | `None` | `string` | Bearer token for authentication. If left undefined, the `auth` parameter should be set on each request. |
| `log_level` | `logging.WARNING` | `int` | Verbosity of logs the instance will produce. By default, logs are written to `stdout`.
| `timeout_ms` | `60_000` | `int` | Number of milliseconds to wait before emitting a `RequestTimeoutError` |
| `base_url` | `"https://api.notion.com"` | `string` | The root URL for sending API requests. This can be changed to test with a mock server. |
| `logger` | Log to console | `logging.Logger` | A custom logger. |
<!-- markdownlint-enable -->

## Requirements

This package supports the following minimum versions:

* Python >= 3.7
* httpx >= 0.15.0

Earlier versions may still work, but we encourage people building new applications
to upgrade to the current stable.

## Getting help

If you have a question about the library, or are having difficulty using it,
chat with the community in [GitHub Discussions](https://github.com/ramnes/notion-sdk-py/discussions).

If you're experiencing issues with the Notion API, such as a service interruption
or a potential bug in the platform, reach out to [Notion help](https://www.notion.so/Help-Support-e040febf70a94950b8620e6f00005004?target=intercom).
