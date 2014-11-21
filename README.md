# CoinSimple HTTP API

The CoinSimple API allows you to create invoices in your businesses through JSON requests. The Base URL for requests is: `https://app.coinsimple.com/api/v1/`

## Authentication

Every business in CoinSimple has a unique `Business ID` and `API Key`. To create an authenticated request you have to send the Business's ID under the key `business_id`, and the SHA256 hash of the API Key along with the current timestamp.

Example:

```ruby
# API credentials
API_KEY     = "2ee7c9d2d5bcde0a8b0ad45956c4332904f7b940"
BUSINESS_ID = 44
TIMESTAMP   = 1416595468

# SHA256_HMAC(TIMESTAMP, API_KEY)
HASH = "fd9ceac7abbe73d907c52332309d246269216f850130aa5fefdbcfe1b8ccd1b2"

# REQUEST
{
    business_id: 44,
    timestamp: 1416595468,
    hash: "fd9ceac7abbe73d907c52332309d246269216f850130aa5fefdbcfe1b8ccd1b2"
}
```

## Creating a new Invoice

To create a new invoice, you need to send a JSON POST request to: `/api/v1/invoice` with the following fields.

| Field Name    | Required | Type         | Description                                   |
|---------------|----------|--------------|-----------------------------------------------|
|name           |Yes       |String        |Recipients Name                                |
|email          |Yes       |String        |Recipients Email                               |
|items          |Yes       |Array         |An array of items                              |
|processor      |Yes *     |String        |The Processor or Wallet to handle the payment  |
|rate           |Yes *     |String        |The Service name to get the rate from          |
|currency       |Yes *     |String        |The Currency for the invoice                   |
|notes          |No        |String        |Terms or Notes                                 |
|invoice_type   |No        |String        |The invoice's type (`regular`, `days`, `date`) |
|interval       |No        |Integer       |The interval for recurring invoices            |
|recurring_times|No        |Integer       |The number of times to recur an invoice        |
|percent        |No        |Float         |An integer representing a percent discount     |
|custom         |No        |String / Hash |Any cusom data which gets sent in the callback |
|callback_url   |No        |String        |A URL to send a notification on payment        |
|redirect_url   |No        |String        |A URL to redirect the user to after payment    |

Processor, Rate and Currency can be left out if you set the defaults in the Business's settings page.

Each item in the `items` array needs to contain three fields:

| Field Name | Type   | Description          |
|------------|--------|----------------------|
|description |String  |The items description |
|quantity    |Integer |The number of items   |
|price       |Float   |The unit price        |

#### Recurring Invoices

To create a recurring invoice you need to set the `invoice_type` and `interval` fields. The two types of recurring invoices are 'by date' or 'by interval of days'. If you want to create an invoice which will be billed on the sixth of every month you can set `invoice_type` to `"date"` and then set `interval` to `6`. On the other hand if you want an invoice to be sent every seven days you can set `invoice_type` to `days` and then set `interval` to `7`.

By default invoices will recur forever, but if you want it to only recur a certain number of times you can set `recurring_times` to the number of times after this invoice to recur.

### Example

A full JSON request body could look something like the following:

```json
{
    "business_id": 44,
    "timestamp": 1416595468,
    "hash": "fd9ceac7abbe73d907c52332309d246269216f850130aa5fefdbcfe1b8ccd1b2",
    "name": "Coin Simple",
    "email": "hi@coinsimple.com",
    "items": [
                { "description": "blue shirt", "price": 15.80, "quantity": 2 },
                { "description": "nice hat", "price": 12, "quantity": 1 },
    ],
    "processor": "armory",
    "rate": "coinbase",
    "currency": "usd",
    "notes": "Will be gift wrapped :)",
    "invoice_type": "regular",
    "percent":  10,
    "custom": { "customer_id": 34, "order_number": 12 },
    "callback_url": "https://example.com/process_callback",
    "redirect_url": "https://example.com/thanks_for_buying"
}

```

The corresponding HTTP request would be something like:

```http
POST /api/v1/invoice HTTP/1.1
Host: app.coinsimple.com
Content-Type: application/json
Cache-Control: no-cache

{ "business_id": 44, "timestamp": 1416595468, "hash": "fd9ceac7abbe73d907c52332309d246269216f850130aa5fefdbcfe1b8ccd1b2", "name": "Coin Simple", "email": "hi@coinsimple.com", "items": [ { "description": "blue shirt", "price": 15.80, "quantity": 2 }, { "description": "nice hat", "price": 12, "quantity": 1 }, ], "processor": "armory", "rate": "coinbase", "currency": "usd", "notes": "Will be gift wrapped :)", "invoice_type": "regular", "percent": 10, "custom": { "customer_id": 34, "order_number": 12 }, "callback_url": "https://example.com/process_callback", "redirect_url": "https://example.com/thanks_for_buying" }
```

### Callbacks

If you set up a callback url when creating an invoice, then that address will be posted with a notification once the invoice is paid. The notification for the example above would look something like this:

```json
{
    "status": "paid",
    "custom": { "customer_id": 34, "order_number": 12 },
    "timestamp": 1416600035,
    "hash": "e03bcda1016075d13052a2081b80a6a2cc1b54bf7c4276ed806addc624c67620"
}
```

The hash that is sent along in the notification is generated in the same way that authentication hash is created, except here the timestamp is the time when the notification was sent. To verify it's authenticity you can hash your API key with the given timestamp and see if it matches up. You can also add custom validations by passing something through the `custom` parameter, and of course combining these is even better.
