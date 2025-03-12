# Clams API Docs (BETA)

## Request an API Key

To get an API key and start using the Clams API, please sign up for the beta using the form at the link below:

<a href="https://clams.tech/api" target="_blank" rel="noopener noreferrer">Sign Up for Beta</a>

## Security

Please note that the Clams API is intended to be used securely on the server side. API keys must be kept confidential and should never be exposed to the client (e.g., in front-end JavaScript code or mobile applications).

## Authentication

All endpoints are secured and require an API key to be included in the request header.
Header: `x-api-key: <API_KEY>`
If the `x-api-key` header is missing or invalid, the server returns a 401 Unauthorized status.

## Base URL

Staging (test env): `https://staging.api.clams.tech`

Prod (Not yet live): `https://api.clams.tech`

## Endpoints

`POST /v0/transactions`

**Description**

Adds a list of transactions for a specific user under the authenticated account. Transactions represent Bitcoin payments (Onchain, Lightning) or Fiat payments.

**Method**

`POST`

**Request**

- **Header**:
  `x-api-key: <API_KEY>` (Required)

- Content-Type: `application/json`

- **Body**:

```jsonc
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "transactions": [
    {
      "transactionId": "tx123",
      "timestamp": "2023-01-01T12:00:00Z",
      "direction": "send",
      "amount": "1.5",
      "currency": "BTC",
      "fee": "0.001",
      "destinationWalletId": "550e8400-e29b-41d4-a716-446655440000",
      "note": "Payment for services",
      "walletId": "550e8400-e29b-41d4-a716-446655440000"
    }
    // ... More transactions for this user_id
  ]
}
```

- **Fields**:

  - `userId: string` (UUID) - The user’s unique identifier.
  - `transactions: array` - List of transaction objects, where each Transaction has:
    - `transactionId: string` - Unique transaction ID (e.g., txid for onchain, payment_hash for Lightning).
    - `timestamp: string` - ISO 8601 UTC datetime (e.g., "2023-01-01T12:00:00Z").
    - `direction: string` - "send" or "receive" (lowercase).
    - `amount: string` - Decimal amount of the transaction (e.g., "1.5").
    - `currency: string` - "BTC" or "USD" (uppercase).
    - `fee: string` (optional) - Decimal fee amount (e.g., "0.001").
    - `destinationWalletId`: string (optional) - UUID of the destination wallet if this transaction is a transfer to another wallet the user owns.
    - `note: string` (optional) - Descriptive note.
    - `walletId: string` (optional) - UUID of the source wallet to group this transaction with others from the same wallet.

- **Constraints**: The pair (`walletId`, `transactionId`) must be unique.

**Response**

- **Success**:

  - **Status**: `200 OK`
  - **Body**: None
  - **Description**: Transactions were successfully written to storage.

- **Errors**:

  - **Status**: `400 Bad Request`
    - **Body**: `{"error": "Invalid transactions payload: <details>"}`
    - **Description**: Invalid JSON or type mismatch.
  - **Status**: `401 Unauthorized`
    - **Body**: { `"error": "API key must be included in the x-api-key header" }`, `{ "error": "Invalid API key" }`
    - **Description**: Missing or invalid API key.
  - **Status**: `500 Internal Server Error`
    - **Body**: `{ "error": "Something went wrong, please try again or contact support." }`
    - **Description**: Internal failure, contact for support.

`POST /v0/trades`

**Description**
Adds a list of trades (Bitcoin/Fiat exchanges) for a specific user under the authenticated account.

**Method**
`POST`

**Request**

- **Header**:
  `x-api-key: <API_KEY>` (Required)
- Content-Type: `application/json`
- **Body**:

```jsonc
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "trades": [
    {
      "tradeId": "trade456",
      "timestamp": "2023-01-02T12:00:00Z",
      "fromCurrency": "BTC",
      "toCurrency": "USD",
      "fromAmount": "0.5",
      "toAmount": "10000",
      "feeAmount": "10",
      "feeCurrency": "USD",
      "note": "Sold BTC for USD",
      "walletId": "550e8400-e29b-41d4-a716-446655440000"
    }
    // ... More trades for this user_id
  ]
}
```

- **Fields**:
  - `userId: string` (UUID) - The user's unique identifier.
  - `trades: array` - List of trade objects, where each Trade has:
    - `tradeId: string` - Unique trade identifier.
    - `timestamp: string` - ISO 8601 UTC datetime (e.g., "2023-01-02T12:00:00Z").
    - `fromCurrency: string` - "BTC" or "USD" (uppercase).
    - `toCurrency: string` - "BTC" or "USD" (uppercase).
    - `fromAmount: string` - Decimal amount sold (e.g., "0.5").
    - `toAmount: string` - Decimal amount received (e.g., "10000").
    - `feeAmount: string` (optional) - Decimal fee amount (e.g., "10").
    - `feeCurrency: string` (optional) - "BTC" or "USD" (uppercase).
    - `note: string` (optional) - Descriptive note.
    - `walletId: string` (optional) - UUID of the wallet involved.

**Response**

- **Success**:
  - **Status**: `200 OK`
  - **Body**: None
  - **Description**: Trades were successfully written to storage.
- **Errors**:

  - **Status**: `400 Bad Request`
    - **Body**: `{"error": "Invalid trades payload: <details>"}`
    - **Description**: Invalid JSON or type mismatch.
  - **Status**: `401 Unauthorized`

    - **Body**: { `"error": "API key must be included in the x-api-key header" }`, `{ "error": "Invalid API key" }`
    - **Description**: Missing or invalid API key.

  - **Status**: `500 Internal Server Error`
    - **Body**: `{ "error": "Something went wrong, please try again or contact support." }`
    - **Description**: Internal failure, contact for support.

`POST /v0/reports/capital-gains`

**Description**
Initiates an asynchronous capital gains report for a user, returning a `reportKey` immediately.

**Method**
`POST`

**Request**

- **Header**:
  `x-api-key: <API_KEY>` (Required)
- Content-Type: `application/json`
- **Body**:

```json
{
  "userId": "550e8400-e29b-41d4-a716-446655440000",
  "algorithm": "FIFO",
  "from": "2023-01-01T00:00:00Z",
  "to": "2023-12-31T23:59:59Z",
  "webhookUrl": "https://example.com/webhook"
}
```

- **Fields**:
  - `userId: string` (UUID) - The user's unique identifier.
  - `algorithm: string` - "FIFO", "LIFO", or "HIFO" (uppercase).
  - `from: string` (optional) - ISO 8601 UTC start datetime (e.g., "2023-01-01T00:00:00Z").
  - `to: string` (optional) - ISO 8601 UTC end datetime (e.g., "2023-12-31T23:59:59Z").
  - `webhookUrl: string` (optional) - URL for completion notification.

**Response**

- **Success**:
  - **Status**: `200 OK`
  - **Body**:
    ```json
    {
      "reportKey": "<accountId>_<userId>_capital-gains"
    }
    ```
  - **Description**: Report generation started; use reportKey to track status.
- **Errors**:

  - **Status**: `400 Bad Request`
    - **Body**: `{"error": "Invalid capital gains report payload: <details>"}`
    - **Description**: Invalid JSON or type mismatch.
  - **Status**: `401 Unauthorized`

    - **Body**: { `"error": "API key must be included in the x-api-key header" }`, `{ "error": "Invalid API key" }`
    - **Description**: Missing or invalid API key.

  - **Status**: `500 Internal Server Error`
    - **Body**: `{ "error": "Something went wrong, please try again or contact support." }`
    - **Description**: Internal failure, contact for support.

`GET /v0/reports/{report_key}`

**Description**
Retrieves the status or result of a report and can be polled as an alternative to using a webhook to receive the generated report.

**Method**
`GET`

**Request**

- **Header**:
  `x-api-key: <API_KEY>` (Required)
- **Path Parameter**:
  - `report_key: string` - Unique report identifier (e.g., "<accountId>\_<userId>\_capital-gains").

**Response**

- **Success**:
  - **Status**: `200 OK`
    - **Body**: `{status: "complete", result: "<report_string>"}`
    - **Description**: Report is complete.
  - **Status**: `200 OK`
    - **Body**: `{"status": "pending"}`
    - **Description**: Report is still processing.
- **Errors**:
  - **Status**: `401 Unauthorized`
    - **Body**: { `"error": "API key must be included in the x-api-key header" }`, `{ "error": "Invalid API key" }`
    - **Description**: Missing or invalid API key.
  - **Status**: `403 Forbidden`
    - **Body**: `{ "error": "Invalid account_id in reportKey for this API key" }`
    - **Description**: The provided API key is not valid to get a report for the account_id in the `reportKey`.
  - **Status**: `404 Not Found`
    - **Body**: { `"error": "Report not found" }`
    - **Description**: A generated report was not found for the given `reportKey`.
  - **Status**: `500 Internal Server Error`
    - **Body**: `{ "error": "Something went wrong, please try again or contact support." }`
    - **Description**: Internal failure, contact for support.

**Types and Enums**

- **Direction**:
  - **Values**: "send", "receive" (lowercase)
  - **Used in**: Transaction.direction
- **Currency**:
  - **Values**: "BTC", "USD" (uppercase)
  - **Used in**: Transaction.currency, Trade.fromCurrency, Trade.toCurrency, Trade.feeCurrency
- **CapitalGainsAlgorithm**:
  - **Values**: "FIFO", "LIFO", "HIFO" (uppercase)
  - **Used in**: POST /v0/reports/capital-gains request body
