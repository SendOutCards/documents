# SendOutCards Public Card Print API Documentation

## Introduction

This document provides comprehensive information about the SendOutCards Public Card Print GraphQL API, which allows third-party applications to create and manage card sending operations programmatically.

## Endpoint

The API endpoint is:

```
https://sendoutcards.com/public_api/graphql
```

## Authentication

All API requests require OAuth2 authentication. Please refer to the [Authentication Documentation](./authentication.md) for detailed information on how to authenticate your requests.

Your application must be registered with SendOutCards to receive credentials. Authentication details should be included in the request headers.

## Card Batch Operations

The API allows you to create batches of card orders and manage them through their lifecycle.

### Types

#### Enums

| Enum              | Values                                                                                                      |
| ----------------- | ----------------------------------------------------------------------------------------------------------- |
| `CardType`        | `POSTCARD`, `FLATCARD`, `TWO_PANEL`, `THREE_PANEL`, `TWO_PANEL_BIG`                                         |
| `CardOrientation` | `LANDSCAPE`, `PORTRAIT`                                                                                     |
| `CardPaperType`   | `STANDARD`, `SATIN`, `PEARL`                                                                                |
| `CardBatchStatus` | `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`                                                              |
| `OrderInfoStatus` | `PENDING`, `FULFILLED`, `REJECTED`, `HELD`, `PAYMENT_ERROR`, `AWAITING_FULFILLMENT`, `REFUNDED`, `CANCELED` |
| `CardPaperType`   | `SATIN`, `STANDARD`, `PEARL`                                                                                |

#### Input Types

##### `AddressInput`

Represents address information for return addresses.

| Field         | Type      | Description                                                    |
| ------------- | --------- | -------------------------------------------------------------- |
| `firstName`   | `String!` | First name. Limited to 100 characters.                         |
| `lastName`    | `String!` | Last name. Limited to 100 characters.                          |
| `companyName` | `String!` | Company name. Limited to 255 characters.                       |
| `address1`    | `String!` | Primary address line. Limited to 100 characters.               |
| `address2`    | `String!` | Secondary address line. Limited to 100 characters.             |
| `city`        | `String!` | City name. Limited to 50 characters.                           |
| `state`       | `String!` | State or province. Limited to 100 characters.                  |
| `postalCode`  | `String!` | Postal or ZIP code. Limited to 20 characters.                  |
| `countryCode` | `String!` | ISO3166-1 alpha2 country code. Must have exactly 2 characters. |

##### `ContactInput`

Represents a recipient's contact information.

| Field               | Type      | Description                                                              |
| ------------------- | --------- | ------------------------------------------------------------------------ |
| `firstName`         | `String!` | First name of the recipient. Limited to 100 characters.                  |
| `lastName`          | `String!` | Last name of the recipient. Limited to 100 characters.                   |
| `companyName`       | `String!` | Company name. Limited to 255 characters.                                 |
| `address1`          | `String!` | Primary address line. Limited to 100 characters.                         |
| `address2`          | `String!` | Secondary address line. Limited to 100 characters.                       |
| `city`              | `String!` | City name. Limited to 50 characters.                                     |
| `state`             | `String!` | State or province. Limited to 100 characters.                            |
| `postalCode`        | `String!` | Postal or ZIP code. Limited to 20 characters.                            |
| `countryCode`       | `String!` | ISO3166-1 alpha2 country code. Must have exactly 2 characters.           |
| `externalReference` | `ID!`     | ID of the contact in your system. Required and limited to 50 characters. |

##### `CardLinePanelInput`

Represents one panel of a card.

| Field | Type      | Description                                 |
| ----- | --------- | ------------------------------------------- |
| `url` | `String!` | URL to the image to be used for this panel. |

##### `CardLineBatchCancelInput`

| Field | Type  | Description                        |
| ----- | ----- | ---------------------------------- |
| `id`  | `ID!` | ID of the batch you want to cancel |

##### `CardLineBatchRetryInput`

| Field | Type  | Description                       |
| ----- | ----- | --------------------------------- |
| `id`  | `ID!` | ID of the batch you want to retry |

##### `CardInput`

Represents the design of a card with all its panels.

| Field    | Type                  | Description                               |
| -------- | --------------------- | ----------------------------------------- |
| `front`  | `CardLinePanelInput!` | Front panel of the card. Required.        |
| `back`   | `CardLinePanelInput`  | Back panel of the card. Optional.         |
| `left`   | `CardLinePanelInput`  | Left inside panel of the card. Optional.  |
| `right`  | `CardLinePanelInput`  | Right inside panel of the card. Optional. |
| `center` | `CardLinePanelInput`  | Center panel of the card. Optional.       |
| `spread` | `CardLinePanelInput`  | Spread panel of the card. Optional.       |

##### `CardLineInput`

Represents a complete card to be sent to one or more recipients.

| Field           | Type               | Description                                                                                             |
| --------------- | ------------------ | ------------------------------------------------------------------------------------------------------- |
| `type`          | `CardType!`        | Type of the card.                                                                                       |
| `paperType`     | `CardPaperType`    | Paper type, default to standard                                                                         |
| `orientation`   | `CardOrientation!` | Orientation of the card.                                                                                |
| `recipients`    | `[ContactInput!]!` | List of recipients for this card.                                                                       |
| `card`          | `CardInput!`       | Design information for the card.                                                                        |
| Field           | Type               | Description                                                                                             |
| --------------- | ------------------ | ------------------------------------------------------------------------------------------------------- |
| `type`          | `CardType!`        | Type of the card.                                                                                       |
| `orientation`   | `CardOrientation!` | Orientation of the card.                                                                                |
| `recipients`    | `[ContactInput!]!` | List of recipients for this card.                                                                       |
| `card`          | `CardInput!`       | Design information for the card.                                                                        |
| `returnAddress` | `AddressInput`     | Optional return address for the card. If not provided, the default account return address will be used. |

#### Output Types

##### `ReturnAddress`

Contains return address information.

| Field         | Type      | Description                    |
| ------------- | --------- | ------------------------------ |
| `firstName`   | `String!` | First name.                    |
| `lastName`    | `String!` | Last name.                     |
| `companyName` | `String!` | Company name.                  |
| `address1`    | `String!` | Primary address line.          |
| `address2`    | `String!` | Secondary address line.        |
| `city`        | `String!` | City name.                     |
| `state`       | `String!` | State or province.             |
| `postalCode`  | `String!` | Postal or ZIP code.            |
| `countryCode` | `String!` | ISO3166-1 alpha2 country code. |

##### `Contact`

Contains recipient information.

| Field               | Type      | Description                        |
| ------------------- | --------- | ---------------------------------- |
| `id`                | `ID!`     | Unique identifier for the contact. |
| `firstName`         | `String!` | First name of the recipient.       |
| `lastName`          | `String!` | Last name of the recipient.        |
| `companyName`       | `String!` | Company name.                      |
| `address1`          | `String!` | Primary address line.              |
| `address2`          | `String!` | Secondary address line.            |
| `city`              | `String!` | City name.                         |
| `state`             | `String!` | State or province.                 |
| `postalCode`        | `String!` | Postal or ZIP code.                |
| `countryCode`       | `String!` | ISO3166-1 alpha2 country code.     |
| `externalReference` | `ID!`     | ID of the contact in your system.  |

##### `CardLineOrderInfo`

Contains information about the status of an individual card being sent to a specific recipient. Each card-recipient pair has its own order status.

| Field    | Type               | Description                                                                                                                                                                 |
| -------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`     | `ID!`              | Unique identifier for the order information.                                                                                                                                |
| `status` | `OrderInfoStatus!` | Current status of the individual card order. Can be one of: `PENDING`, `FULFILLED`, `REJECTED`, `HELD`, `PAYMENT_ERROR`, `AWAITING_FULFILLMENT`, `REFUNDED`, or `CANCELED`. |

##### `Recipient`

Contains recipient and order information.

| Field       | Type                | Description                           |
| ----------- | ------------------- | ------------------------------------- |
| `contact`   | `Contact!`          | Contact information of the recipient. |
| `orderInfo` | `CardLineOrderInfo` | Information about the order status.   |

##### `CardPanel`

Contains information about a card panel.

| Field | Type     | Description                               |
| ----- | -------- | ----------------------------------------- |
| `url` | `String` | URL to the rendered image for this panel. |

##### `Card`

Contains information about a card.

| Field       | Type            | Description                     |
| ----------- | --------------- | ------------------------------- | --- |
| `type`      | `CardType!`     | Type of the card.               |
| `paperType` | `CardPaperType` | Paper type, default to standard |     |
| `front`     | `CardPanel`     | Front panel of the card.        |
| `inside`    | `CardPanel!`    | Inside panel of the card.       |
| `back`      | `CardPanel!`    | Back panel of the card.         |

##### `CardLine`

Represents a card order for one or more recipients.

| Field           | Type             | Description                          |
| --------------- | ---------------- | ------------------------------------ |
| `id`            | `ID!`            | Unique identifier for the card line. |
| `card`          | `Card`           | Card information.                    |
| `recipients`    | `[Recipient!]!`  | List of recipients for this card.    |
| `returnAddress` | `ReturnAddress!` | Return address for the card.         |

##### `CardLineBatch`

Represents a batch of card orders. A Card Line Batch is the process of creating multiple cards in a batch and optionally creating orders for the whole batch.

| Field                     | Type               | Description                                                                                                             |
| ------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| `id`                      | `ID!`              | Unique identifier for the batch.                                                                                        |
| `status`                  | `CardBatchStatus!` | Current status of the batch. Reflects only the status of the card creation process, not the status of the entire order. |
| `totalCardLinesToProcess` | `Int!`             | Total number of card lines in the batch.                                                                                |
| `totalCardLinesProcessed` | `Int!`             | Number of card lines that have been processed.                                                                          |
| `lines`                   | `[CardLine!]!`     | List of card lines in the batch.                                                                                        |
| `statusMessage`           | `String`           | Additional context for the current status. Shows failure reasons when status is in `FAILED` state                       |

The status of the CardLineBatch progresses as follows:

- `PENDING`: Batch has been created but processing has not yet started
- `PROCESSING`: System is downloading images, creating cards, etc.
- `COMPLETED`: All cards in the batch were successfully created
- `FAILED`: One or more cards in the batch failed to be created

Note: `orderInfo` for recipients will remain `null` until the batch status moves to `COMPLETED`. Additionally, orders are batched at end of day (EOD), so the `orderInfo` status for each card will most likely return `PENDING` until the following day, even after the batch is `COMPLETED`.

### Queries

#### `cardLineBatch`

Retrieve a specific card line batch by ID.

```graphql
query GetCardLineBatch($id: ID!) {
  cardLineBatch(id: $id) {
    id
    status
    totalCardLinesToProcess
    totalCardLinesProcessed
    lines {
      edges {
        node {
          id
          card {
            type
            front {
              url
            }
            inside {
              url
            }
            back {
              url
            }
          }
          returnAddress {
            firstName
            lastName
            companyName
            address1
            address2
            city
            state
            postalCode
            countryCode
          }
          recipients {
            contact {
              id
              firstName
              lastName
              address1
              city
              state
              postalCode
              countryCode
              externalReference
            }
            orderInfo {
              id
              status
            }
          }
        }
      }
    }
  }
}
```

#### `cardLineBatches`

Retrieve a paginated list of card line batches.

```graphql
query GetCardLineBatches($first: Int, $after: String) {
  cardLineBatches(first: $first, after: $after) {
    edges {
      node {
        id
        status
        totalCardLinesToProcess
        totalCardLinesProcessed
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Mutations

#### `cardLineBatchCreate`

Create a new batch of card orders. This mutation initiates the process of downloading your image URLs, uploading them to our system, and constructing the cards.

```graphql
mutation CreateCardLineBatch(
  $cards: [CardLineInput!]!
  $shouldFinalizeOrders: Boolean
) {
  cardLineBatchCreate(
    cards: $cards
    shouldFinalizeOrders: $shouldFinalizeOrders
  ) {
    id
    status
    totalCardLinesToProcess
    totalCardLinesProcessed
  }
}
```

**Parameters:**

| Parameter              | Type                | Description                                                                                                                        |
| ---------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `cards`                | `[CardLineInput!]!` | List of card orders to create.                                                                                                     |
| `shouldFinalizeOrders` | `Boolean`           | Optional. If true, orders will be automatically finalized after all cards in the batch are successfully created. Default is false. |

**Note:** When `shouldFinalizeOrders` is true, orders will only be created if ALL cards in the batch are successfully created (batch status becomes `COMPLETED`). If any card fails, no orders will be created.

#### `cardLineBatchFinalize`

Finalize a batch of card orders, triggering them to be sent. This mutation should be used when `shouldFinalizeOrders` was set to false during batch creation and you now want to create orders for the cards.

```graphql
mutation FinalizeCardLineBatch($batchUuid: ID, $batchId: ID) {
  cardLineBatchFinalize(batchUuid: $batchUuid, batchId: $batchId) {
    id
    status
    totalCardLinesToProcess
    totalCardLinesProcessed
  }
}
```

**Parameters:**

| Parameter   | Type | Description                              |
| ----------- | ---- | ---------------------------------------- |
| `batchUuid` | `ID` | Optional. UUID of the batch to finalize. |
| `batchId`   | `ID` | Optional. ID of the batch to finalize.   |

**Note:**

- At least one of `batchUuid` or `batchId` must be provided. An error will be returned if neither is provided.
- This mutation should only be called after confirming that the batch status is `COMPLETED`. Attempting to finalize a batch that is not in the `COMPLETED` state will not create any orders.

## Examples

### Creating a New Batch with Return Address

```graphql
mutation CreateCardLineBatch {
  cardLineBatchCreate(
    cards: [
      {
        type: POSTCARD
        orientation: LANDSCAPE
        returnAddress: {
          firstName: "Jane"
          lastName: "Smith"
          companyName: "My Business"
          address1: "456 Business Ave"
          address2: "Floor 2"
          city: "Business City"
          state: "NY"
          postalCode: "54321"
          countryCode: "US"
        }
        recipients: [
          {
            firstName: "John"
            lastName: "Doe"
            companyName: "ACME Inc."
            address1: "123 Main St"
            address2: "Suite 101"
            city: "Anytown"
            state: "CA"
            postalCode: "12345"
            countryCode: "US"
            externalReference: "contact-123"
          }
        ]
        card: {
          front: { url: "https://example.com/images/front.jpg" }
          back: { url: "https://example.com/images/back.jpg" }
        }
      }
    ]
    shouldFinalizeOrders: false
  ) {
    id
    status
  }
}
```

### Creating a New Batch without Return Address (uses default)

```graphql
mutation CreateCardLineBatch {
  cardLineBatchCreate(
    cards: [
      {
        type: POSTCARD
        orientation: LANDSCAPE
        recipients: [
          {
            firstName: "John"
            lastName: "Doe"
            companyName: "ACME Inc."
            address1: "123 Main St"
            address2: "Suite 101"
            city: "Anytown"
            state: "CA"
            postalCode: "12345"
            countryCode: "US"
            externalReference: "contact-123"
          }
        ]
        card: {
          front: { url: "https://example.com/images/front.jpg" }
          back: { url: "https://example.com/images/back.jpg" }
        }
      }
    ]
    shouldFinalizeOrders: false
  ) {
    id
    status
  }
}
```

#### `cardLineBatchCancel`

Cancels an existing batch that has been finalized with `shouldFinalizeOrders`. Note this will cancel the orders created with this batch that have NOT been fulfilled.

```graphql
mutation CancelCardLineBatch($input: CardLineBatchCancelInput!) {
  cardLineBatchCancel(input: $input) {
    success
    message
  }
}
```

**Parameters:**

| Parameter | Type                        | Description                                      |
| --------- | --------------------------- | ------------------------------------------------ |
| `input`   | `CardLineBatchCancelInput!` | Includes the ID of the batch you want to cancel. |

### Finalizing a Batch

```graphql
mutation FinalizeCardLineBatch {
  cardLineBatchFinalize(batchUuid: "550e8400-e29b-41d4-a716-446655440000") {
    id
    status
  }
}
```

### Retry a Failed Batch

This will change the `status` back to `PENDING` until the asynchronous processes finish the retrying to create the batch. Once they finish the status will either be `COMPLETED` or `FAILED` again
Returns the full CardLineBatch

```graphql
mutation RetryFailedBatch($input: CardLineBatchRetryInput!) {
  cardLineBatchRetry(input: $input) {
    id
    status
    statusMessage
    ...CardLineBatchFields ## get any fields needed from the card line batch type
  }
}
```

### Checking Batch Status

```graphql
query GetBatchStatus {
  cardLineBatch(id: "550e8400-e29b-41d4-a716-446655440000") {
    status
    totalCardLinesToProcess
    totalCardLinesProcessed
    lines {
      edges {
        node {
          returnAddress {
            firstName
            lastName
            companyName
            address1
            city
            state
            postalCode
            countryCode
          }
          recipients {
            contact {
              firstName
              lastName
            }
            orderInfo {
              status
            }
          }
        }
      }
    }
  }
}
```

## Workflow and Process

### Card Line Batch Process

A Card Line Batch represents the process of creating multiple cards in a batch and optionally creating orders for the whole batch. Here's how the process works:

1. You create a batch by calling the `cardLineBatchCreate` mutation with your card data and images.
2. Optionally specify a `returnAddress` for each card line. If not provided, the default account return address will be used.
3. The system processes each card in the batch by downloading client images, reuploading them to our system, and constructing the cards.
4. The status of the `CardLineBatch` reflects only the status of this card creation process, not the status of the entire order.
5. If `shouldFinalizeOrders` is set to `true`, orders will be created only after all cards in the batch are successfully created.
6. You can check the batch status at any time using the `cardLineBatch` query.
7. The `orderInfo` field will remain `null` until the cards are all created and the batch status moves to `COMPLETED`.
8. Orders are batched at the end of the day (EOD), so the `orderInfo` status for each card will most likely return `PENDING` until the following day, even after the batch is `COMPLETED`.

### Card Line Batch Status Flow

```
PENDING → PROCESSING → COMPLETED or FAILED
```

- `PENDING`: Batch has been created but processing has not yet started
- `PROCESSING`: System is downloading images, creating cards, etc.
- `COMPLETED`: All cards in the batch were successfully created
- `FAILED`: One or more cards in the batch failed to be created

### Order Status Flow

After a batch is `COMPLETED` and orders are created (either automatically with `shouldFinalizeOrders: true` or manually with `cardLineBatchFinalize`), each individual card-recipient pair gets its own order status:

```
PENDING → AWAITING_FULFILLMENT → FULFILLED or REJECTED or HELD or PAYMENT_ERROR or REFUNDED or CANCELED
```

## Card Type Requirements

Different card types require different panels:

| Card Type       | Required Panels                    | Optional Panels |
| --------------- | ---------------------------------- | --------------- |
| `POSTCARD`      | `front`, `back`                    | None            |
| `FLATCARD`      | `front`, `back`                    | None            |
| `TWO_PANEL`     | `front`, `left`, `right`           | `back`          |
| `THREE_PANEL`   | `front`, `left`, `center`, `right` | `back`          |
| `TWO_PANEL_BIG` | `front`, `spread`                  | `back`          |

**Important Notes on Panel Configuration:**

- For landscape oriented cards, the `left` field represents the top panel and the `right` field represents the bottom panel.
- For portrait oriented cards, the `left` and `right` fields represent their respective sides.
- The `spread` panel can be used in place of `left`, `right`, and `center` panels to represent all inside panels across as a single image.
- You cannot use `spread` in combination with any of the `left`, `right`, or `center` panels - it must be used exclusively.
- If you provide invalid panel configurations for any card in the batch, the entire batch process will fail.

## Error Handling

The API returns standard GraphQL errors with descriptive messages. Common error scenarios include:

- Authentication failures
- Invalid input data
- Resource not found
- Permission denied

## Rate Limiting

API requests are subject to rate limiting. Current limits are:

- 100 requests per minute
- 1000 requests per hour
- 5000 requests per day

Exceeding these limits will result in a 429 Too Many Requests response.

## Using the API

The GraphQL API is accessed through standard HTTP POST requests to:

```
https://sendoutcards.com/public_api/graphql
```

All requests must include your OAuth2 token in the Authorization header. Here are examples using cURL:

```bash
curl -X POST \
  https://api.sendoutcards.com/graphql \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{"query": "YOUR_GRAPHQL_QUERY"}'
```

### Creating a Card Batch with Return Address

Request:

```bash
curl -X POST \
  https://api.sendoutcards.com/graphql \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "mutation CreateCardLineBatch($cards: [CardLineInput!]!, $shouldFinalizeOrders: Boolean) { cardLineBatchCreate(cards: $cards, shouldFinalizeOrders: $shouldFinalizeOrders) { id status totalCardLinesToProcess totalCardLinesProcessed } }",
  "variables": {
    "cards": [
      {
        "type": "POSTCARD",
        "orientation": "LANDSCAPE",
        "returnAddress": {
          "firstName": "Jane",
          "lastName": "Smith",
          "companyName": "My Business",
          "address1": "456 Business Ave",
          "address2": "Floor 2",
          "city": "Business City",
          "state": "NY",
          "postalCode": "54321",
          "countryCode": "US"
        },
        "recipients": [
          {
            "firstName": "John",
            "lastName": "Doe",
            "companyName": "ACME Inc.",
            "address1": "123 Main St",
            "address2": "Suite 101",
            "city": "Anytown",
            "state": "CA",
            "postalCode": "12345",
            "countryCode": "US",
            "externalReference": "contact-123"
          }
        ],
        "card": {
          "front": {
            "url": "https://example.com/images/front.jpg"
          },
          "back": {
            "url": "https://example.com/images/back.jpg"
          }
        }
      }
    ],
    "shouldFinalizeOrders": false
  }
}'
```

Response:

```json
{
  "data": {
    "cardLineBatchCreate": {
      "id": "Q2FyZExpbmVCYXRjaDoxMjM=",
      "status": "PENDING",
      "totalCardLinesToProcess": 1,
      "totalCardLinesProcessed": 0
    }
  }
}
```

### Finalizing a Card Batch

Request:

```bash
curl -X POST \
  https://api.sendoutcards.com/graphql \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "mutation FinalizeCardLineBatch($batchUuid: ID, $batchId: ID) { cardLineBatchFinalize(batchUuid: $batchUuid, batchId: $batchId) { id status totalCardLinesToProcess totalCardLinesProcessed } }",
  "variables": {
    "batchId": "123"
  }
}'
```

Response:

```json
{
  "data": {
    "cardLineBatchFinalize": {
      "id": "Q2FyZExpbmVCYXRjaDoxMjM=",
      "status": "PROCESSING",
      "totalCardLinesToProcess": 1,
      "totalCardLinesProcessed": 1
    }
  }
}
```

### Querying Batch Status with Return Address

Request:

```bash
curl -X POST \
  https://api.sendoutcards.com/graphql \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "query GetBatchStatus($id: ID!) { cardLineBatch(id: $id) { status totalCardLinesToProcess totalCardLinesProcessed lines { edges { node { returnAddress { firstName lastName companyName address1 city state postalCode countryCode } recipients { contact { firstName lastName } orderInfo { status } } } } } } }",
  "variables": {
    "id": "Q2FyZExpbmVCYXRjaDoxMjM="
  }
}'
```

Response:

```json
{
  "data": {
    "cardLineBatch": {
      "status": "COMPLETED",
      "totalCardLinesToProcess": 1,
      "totalCardLinesProcessed": 1,
      "lines": {
        "edges": [
          {
            "node": {
              "returnAddress": {
                "firstName": "Jane",
                "lastName": "Smith",
                "companyName": "My Business",
                "address1": "456 Business Ave",
                "city": "Business City",
                "state": "NY",
                "postalCode": "54321",
                "countryCode": "US"
              },
              "recipients": [
                {
                  "contact": {
                    "firstName": "John",
                    "lastName": "Doe"
                  },
                  "orderInfo": {
                    "status": "FULFILLED"
                  }
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```

### Paginated Query for Multiple Batches

Request:

```bash
curl -X POST \
  https://api.sendoutcards.com/graphql \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "query": "query GetCardLineBatches($first: Int, $after: String) { cardLineBatches(first: $first, after: $after) { edges { node { id status totalCardLinesToProcess totalCardLinesProcessed } cursor } pageInfo { hasNextPage endCursor } } }",
  "variables": {
    "first": 10,
    "after": null
  }
}'
```

Response:

```json
{
  "data": {
    "cardLineBatches": {
      "edges": [
        {
          "node": {
            "id": "Q2FyZExpbmVCYXRjaDoxMjM=",
            "status": "COMPLETED",
            "totalCardLinesToProcess": 1,
            "totalCardLinesProcessed": 1
          },
          "cursor": "YXJyYXljb25uZWN0aW9uOjA="
        },
        {
          "node": {
            "id": "Q2FyZExpbmVCYXRjaDoxMjQ=",
            "status": "PENDING",
            "totalCardLinesToProcess": 2,
            "totalCardLinesProcessed": 0
          },
          "cursor": "YXJyYXljb25uZWN0aW9uOjE="
        }
      ],
      "pageInfo": {
        "hasNextPage": false,
        "endCursor": "YXJyYXljb25uZWN0aW9uOjE="
      }
    }
  }
}
```
