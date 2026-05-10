# Exchange Rate API - PHP SDK

Official PHP SDK for the [Exchange Rate API](https://exchange-rateapi.com) exchange rate service.

Real-time mid-market exchange rates for 160+ currencies, sourced from Reuters (Refinitiv) and interbank market feeds.

## Installation

```bash
composer require exchangerateapi/sdk
```

## Quick Start

Get your free API key at [exchange-rateapi.com/register](https://exchange-rateapi.com/register).

```php
use ExchangeRateAPI\ExchangeRateAPI;

$client = new ExchangeRateAPI('era_live_your_key_here');

// Get exchange rate
$rate = $client->getRate('USD', 'EUR');
echo "1 USD = {$rate[0]['rate']} EUR\n";

// Convert amount
$result = $client->convert('USD', 'EUR', 100);
echo "\$100 = €{$result['result']}\n";

// Get historical rates
$history = $client->getHistoricalRates('USD', 'EUR', '30d');
foreach ($history['rates'] as $point) {
    echo "{$point['time']}: {$point['rate']}\n";
}
```

## API Reference

### `new ExchangeRateAPI($apiKey, $baseUrl, $timeout)`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `$apiKey` | `string` | **required** | Your API key ([register free](https://exchange-rateapi.com/register)) |
| `$baseUrl` | `string` | `https://exchange-rateapi.com` | API base URL |
| `$timeout` | `int` | `10` | Request timeout in seconds |

### Methods

| Method | Description |
|--------|-------------|
| `getRate($from, $to, $amount)` | Get exchange rate between two currencies |
| `convert($from, $to, $amount)` | Convert amount between currencies |
| `getRates($source, $target)` | Get rate data with metadata |
| `getHistoricalRates($source, $target, $period)` | Historical rates (1d/7d/30d/1y) |

All methods require an API key.

### Error Handling

```php
use ExchangeRateAPI\ExchangeRateAPI;
use ExchangeRateAPI\ExchangeRateAPIException;

try {
    $rate = $client->getRate('USD', 'INVALID');
} catch (ExchangeRateAPIException $e) {
    echo $e->getMessage();      // Error message
    echo $e->getStatusCode();   // HTTP status code
}
```

## Requirements

- PHP >= 7.4
- ext-curl
- ext-json

## Links

- [API Documentation](https://exchange-rateapi.com/docs)
- [Register (Free)](https://exchange-rateapi.com/register)
- [Dashboard](https://exchange-rateapi.com/profile)

## License

MIT
