# python-quickbooks

A Python 3 library for accessing the QuickBooks API.

`python-quickbooks` provides Python objects and helpers for working with QuickBooks Online data through the QuickBooks API. It supports common workflows such as querying objects, filtering results, creating and updating records, batch operations, change data capture, attachments, JSON conversion, date formatting, and QuickBooks-specific exception handling.

## Overview

This project is a rework of `quickbooks-python` and is designed for applications that need to interact with QuickBooks Online from Python.

The library uses object names and property names that match their QuickBooks counterparts. These names may not follow PEP 8 because they are aligned with the QuickBooks API model.

The original usage examples were written with a Django application in mind, but the library can be adapted to other Python application frameworks or integration approaches.

## Installation

```bash
pip install python-quickbooks
```

## QuickBooks OAuth

The library requires `intuit-oauth` for OAuth authentication.

To connect to QuickBooks Online, follow Intuit’s OAuth 2.0 documentation and create an `AuthClient` with your QuickBooks app credentials, access token, environment, and redirect URI.

```python
from intuitlib.client import AuthClient

auth_client = AuthClient(
    client_id="CLIENT_ID",
    client_secret="CLIENT_SECRET",
    access_token="ACCESS_TOKEN",
    environment="sandbox",
    redirect_uri="http://localhost:8000/callback",
)
```

Then create a QuickBooks client with the auth client, refresh token, and company ID.

```python
from quickbooks import QuickBooks

client = QuickBooks(
    auth_client=auth_client,
    refresh_token="REFRESH_TOKEN",
    company_id="COMPANY_ID",
)
```

A QuickBooks API minor version can also be supplied when creating the client.

```python
client = QuickBooks(
    auth_client=auth_client,
    refresh_token="REFRESH_TOKEN",
    company_id="COMPANY_ID",
    minorversion=69,
)
```

Note: Intuit has announced deprecation of support for minor versions 1–74 beginning August 1, 2025.

## Object Operations

The library supports common object operations such as listing, filtering, querying, counting, retrieving, creating, and updating QuickBooks entities.

```python
from quickbooks.objects.customer import Customer

customers = Customer.all(qb=client)
```

Filtered queries are supported:

```python
customers = Customer.filter(Active=True, FamilyName="Smith", qb=client)
```

Ordering and paging can be supplied where supported:

```python
customers = Customer.filter(
    start_position=1,
    max_results=25,
    Active=True,
    FamilyName="Smith",
    qb=client,
)
```

Custom query strings can also be used:

```python
customers = Customer.query(
    "SELECT * FROM Customer WHERE Active = True",
    qb=client,
)
```

Warning: user input should not be passed directly into custom queries without sanitization. This library does not sanitize user input.

## Batch Operations

Batch helpers allow multiple QuickBooks operations to be submitted in one request.

Supported batch examples include:

- creating multiple objects
- updating multiple objects
- deleting entities that support delete operations
- reviewing successful and failed batch results

```python
from quickbooks.batch import batch_create

results = batch_create(customers, qb=client)
```

Batch responses include successes and faults so callers can inspect which operations completed and which returned errors.

## Change Data Capture

Change Data Capture can be used to retrieve objects changed since a given timestamp.

```python
from quickbooks.cdc import change_data_capture
from quickbooks.objects import Invoice

cdc_response = change_data_capture(
    [Invoice],
    "2017-01-01T00:00:00",
    qb=client,
)
```

Multiple entity types can be queried together, and `datetime` values are converted automatically.

## Attachments

The library supports QuickBooks attachable objects, including notes and files.

Supported attachment patterns include:

- attaching a note to an entity
- attaching a file by path
- attaching file bytes directly

A file attachment can use either `_FilePath` or `_FileBytes`, but not both at the same time.

## Optional Parameters

Some QuickBooks objects require query-string parameters for specific operations. These parameters can be passed when saving an object.

```python
purchase.save(
    qb=client,
    params={"include": "allowduplicatedocnum"},
)
```

## Invoice Link and Void Support

Invoices can be used with QuickBooks invoice-link behavior when the required invoice fields and query parameters are supplied. The README notes that `minorversion` must be set to 36 or greater for this use case.

Invoices with an ID can also be voided:

```python
invoice.void(qb=client)
```

## JSON Support

Objects include `to_json` and `from_json` helpers.

```python
json_data = account.to_json()
```

```python
account = Account.from_json(
    {
        "AccountType": "Accounts Receivable",
        "AcctNum": "123123",
        "Name": "MyJobs",
    }
)
```

## Date Formatting

QuickBooks requires specific formats for date and datetime values. The library includes formatting helpers such as:

- `qb_date_format`
- `qb_datetime_format`
- `qb_datetime_utc_offset_format`

## Exception Handling

QuickBooks API errors can be handled with `QuickbooksException`, which exposes additional error information returned by QuickBooks.

```python
from quickbooks.exceptions import QuickbooksException

try:
    pass
except QuickbooksException as e:
    e.message
    e.error_code
    e.detail
```

## Notes

- The package is intended for Python 3.
- The library is public work-in-progress software for developers integrating with the QuickBooks API.
- Object and property names intentionally match QuickBooks API naming.
