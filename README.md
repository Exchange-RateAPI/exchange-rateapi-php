# exchangerateapi/sdk

[![Packagist version](https://img.shields.io/packagist/v/exchangerateapi/sdk.svg)](https://packagist.org/packages/exchangerateapi/sdk)
[![PHP](https://img.shields.io/packagist/php-v/exchangerateapi/sdk.svg)](https://packagist.org/packages/exchangerateapi/sdk)
[![license](https://img.shields.io/packagist/l/exchangerateapi/sdk.svg)](https://github.com/Exchange-RateAPI/exchange-rateapi-php/blob/main/LICENSE)
[![zero dependencies](https://img.shields.io/badge/dependencies-0-brightgreen.svg)](https://packagist.org/packages/exchangerateapi/sdk)

**Official PHP SDK for real-time mid-market exchange rates. 160+ currencies, zero external dependencies.**

## Why Choose This SDK?

- **Lightweight** -- Only requires `ext-curl` and `ext-json`, both bundled with PHP by default
- **Real-Time Data** -- Rates updated every 60 seconds from Reuters (Refinitiv) and interbank feeds
- **Mid-Market Rates** -- The true interbank rate -- no hidden spread or markup
- **160+ Currencies** -- Major, minor, and exotic currency pairs
- **PHP 7.4+** -- Works on any modern PHP version, including PHP 8.x
- **Zero Dependencies** -- No Composer packages to audit, no supply chain risk

## Get Your API Key

Ready to start? Get your free API key from [exchange-rateapi.com/register](https://exchange-rateapi.com/register).

## Installation

```bash
composer require exchangerateapi/sdk
```

## Quick Start

Get up and running in seconds:

```php
use ExchangeRateAPI\ExchangeRateAPI;

$client = new ExchangeRateAPI('era_live_your_key_here');

// Get exchange rate
$rate = $client->getRate('USD', 'EUR');
echo "1 USD = {$rate['rate']} EUR\n";

// Convert an amount
$result = $client->convert('USD', 'EUR', 1000);
echo "\$1,000 = EUR {$result['result']}\n";

// Get historical rates for the last 30 days
$history = $client->getHistoricalRates('USD', 'EUR', '30d');
echo "Current rate: {$history['current']['rate']}\n";
foreach ($history['rates'] as $point) {
    echo "{$point['time']}: {$point['rate']}\n";
}
```

## API Reference

- [Single Rate](#single-rate) -- Get an exchange rate between two currencies
- [Currency Conversion](#currency-conversion) -- Convert an amount between currencies
- [Rates with Metadata](#rates-with-metadata) -- Get rate data with full metadata
- [Historical Rates by Period](#historical-rates-by-period) -- Preset period lookups (1d/7d/30d/1y)

---

### Single Rate

Get an exchange rate between two currencies with a single call:

```php
// Basic rate lookup
$data = $client->getRate('USD', 'EUR');
echo "1 USD = {$data['rate']} EUR\n";

// With an amount
$data = $client->getRate('USD', 'EUR', 500);
echo "\$500 = EUR {$data['to']['amount']}\n";
```

**Response:**

```php
[
    'from' => ['currency' => 'USD', 'amount' => 1],
    'to'   => ['currency' => 'EUR', 'amount' => 0.9234],
    'rate'   => 0.9234,
    'source' => 'mid-market',
]
```

---

### Currency Conversion

Convert any amount between currencies -- returns a clean, flat result:

```php
$result = $client->convert('USD', 'EUR', 1000);
echo "\$1,000 = EUR {$result['result']}\n";
echo "Rate used: {$result['rate']}\n";
```

**Response:**

```php
[
    'from'   => 'USD',
    'to'     => 'EUR',
    'amount' => 1000,
    'result' => 923.4,
    'rate'   => 0.9234,
]
```

---

### Rates with Metadata

Get exchange rates with full metadata for a currency pair:

```php
$data = $client->getRates('USD', 'EUR');
print_r($data);
```

**Response:**

```php
[
    [
        'source' => 'USD',
        'target' => 'EUR',
        'rate'   => 0.9234,
        'time'   => '2026-05-25T14:30:00Z',
    ],
]
```

---

### Historical Rates by Period

Get historical rates using preset periods -- no date math needed:

```php
$history = $client->getHistoricalRates('USD', 'EUR', '30d');

echo "Current rate: {$history['current']['rate']}\n";
echo "Period: {$history['period']}\n";

foreach ($history['rates'] as $point) {
    echo "{$point['time']}: {$point['rate']}\n";
}
```

**Available periods:** `1d`, `7d`, `30d`, `1y` (default: `7d`)

**Response:**

```php
[
    'source'  => 'USD',
    'target'  => 'EUR',
    'period'  => '30d',
    'current' => ['rate' => 0.9234, 'time' => '2026-05-25T14:30:00Z'],
    'rates'   => [
        ['rate' => 0.9187, 'time' => '2026-04-25T14:30:00Z'],
        ['rate' => 0.9195, 'time' => '2026-04-26T14:30:00Z'],
        // ...
    ],
]
```

---

## Configuration

```php
$client = new ExchangeRateAPI(
    apiKey:  'era_live_your_key_here',           // Required
    baseUrl: 'https://exchange-rateapi.com',     // Optional
    timeout: 10,                                  // Optional (seconds)
);
```

| Parameter  | Type     | Default                        | Description                |
| ---------- | -------- | ------------------------------ | -------------------------- |
| `$apiKey`  | `string` | --                             | Your API key               |
| `$baseUrl` | `string` | `https://exchange-rateapi.com` | API base URL               |
| `$timeout` | `int`    | `10`                           | Request timeout in seconds |

---

## Error Handling

All errors are thrown as `ExchangeRateAPIException` with an optional HTTP status code:

```php
use ExchangeRateAPI\ExchangeRateAPI;
use ExchangeRateAPI\ExchangeRateAPIException;

$client = new ExchangeRateAPI('era_live_your_key_here');

try {
    $rate = $client->getRate('USD', 'INVALID');
} catch (ExchangeRateAPIException $e) {
    echo $e->getMessage();      // "Currency not found"
    echo $e->getStatusCode();   // 404
}
```

| Status | Meaning                                 |
| ------ | --------------------------------------- |
| --     | Missing API key (thrown before request) |
| `401`  | Invalid API key                         |
| `404`  | Currency code not found                 |
| `429`  | Rate limit exceeded                     |
| `500`  | Server error                            |

---

## Methods Reference

| Method                                          | Description                                 |
| ----------------------------------------------- | ------------------------------------------- |
| `getRate($from, $to, $amount)`                  | Get exchange rate between two currencies    |
| `convert($from, $to, $amount)`                  | Convert amount and get a flat result array  |
| `getRates($source, $target)`                    | Get rates with full metadata                |
| `getHistoricalRates($source, $target, $period)` | Historical rates by period (1d/7d/30d/1y)   |

---

## Zero Dependencies

This SDK uses only PHP extensions that ship with every standard PHP installation:

- **ext-curl** -- HTTP requests
- **ext-json** -- JSON encoding and decoding

No third-party Composer packages. Nothing to audit, nothing to break.

---

## Requirements

- PHP >= 7.4
- ext-curl
- ext-json

---

## Links

- [API Documentation](https://exchange-rateapi.com/docs/)
- [Register (Free)](https://exchange-rateapi.com/register/)
- [Dashboard](https://exchange-rateapi.com/dashboard/)
- [Status](https://exchange-rateapi.com/status/)
- [GitHub](https://github.com/Exchange-RateAPI/exchange-rateapi-php)
- [Packagist](https://packagist.org/packages/exchangerateapi/sdk)

## License

MIT
