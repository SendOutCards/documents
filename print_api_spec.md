# Print API Specification

## Overview

We are extending our existing Public API (which uses OAuth authentication) to support card creation and order processing. **This document is a work in progress and should not be treated as a finalized implementation.**

## Approach

The proposed solution simplifies the client experience by abstracting away our internal card structure. Clients will:

1. Treat each card panel as a single image
2. Upload images for each panel (or full-bleed panel) they wish to customize
3. Process orders through our standard SendOutCards workflow

This approach allows third-party integrations to create cards without needing to understand our internal element structure. From our system's perspective, these externally-created cards will be processed identically to cards created within our platform.

## Benefits

- Simplified integration for clients
- Consistent order processing
- Maintains compatibility with existing workflows
- Reduces API complexity by hiding implementation details

## Proposed GraphQL Spec

```gql
enum CardType {
  POSTCARD
  FLATCARD
  TWO_PANEL
  THREE_PANEL
  TWO_PANEL_BIG
}

enum CardOrientation {
  HORIZONTAL
  VERTICAL
}

enum CardPaperType {
  STANDARD
  SATIN
  PEARL
}

enum CardPanelLocation {
  FRONT
  BACK
  INSIDE_LEFT
  INSIDE_CENTER
  INSIDE_RIGHT
  INSIDE_SPREAD
}

input CardLinePanelInput {
  url: String!
}

input ContactInput {
  firstName: String!
  lastName: String!
  companyName: String
  address1: String!
  address2: String!
  city: String!
  state: String!
  postalCode: String!
  countryCode: String!
  externalReferenceId: ID
  userVerified: Boolean # Default - false, If true the address will not be validated and the address provided will be trusted
}

input CardLineInput {
  type: CardType!
  orientation: CardOrientation!
  recipients: [ContactInput!]!
  front: CardLinePanelInput!
  left: CardLinePanelInput
  right: CardLinePanelInput
  spread: CardLinePanelInput
  back: CardLinePanelInput
}

input CreateCardLineInput {
  cards: [CardLineInput!]!
  shouldFinalizeOrder: Boolean # Default - false, If true no further action is required order will be created and cards will be printed. Otherwise finalizeOrder must be called to process
}

type CardPanel {
  id: ID!
  url: String!
  location: CardPanelLocation!
}

type Address {
  line1: String!
  line2: String
  city: String!
  stateOrProvince: String!
  postalCode: String!
}

type Contact {
  id: ID!
  firstName: String!
  lastName: String!
  companyName: String
  address1: String!
  address2: String!
  city: String!
  state: String!
  postalCode: String!
  countryCode: String!
  externalReferenceId: ID
}

type CardLine {
  id: ID!
  type: CardType!
  orientation: CardOrientation!
  panels:  [CardPanel!]!
  recipients: [Contact!]!
}

type Order {
  id: ID!
  cardLines: [CardLine!]!
  ## TBD (totals, status, etc..)
}

query order(orderId: [Card]) {
  order {
    ...Order
  }
}

mutation createCardLines(input: [CreateCardLineInput!]!) {
  cardLines(input: $input) {
    errors {
      message
      code
      field
    }
    lines {
      ...CardLine
    }
  }
}

mutation finalizeOrder(cardLineIds: [ID!]!) {
  order {
    errors {
      message
      code
      field
    }
    order {
     ...Order
    }
  }
}
```

## Client Documentation

This documentation covers the two main mutations for creating and finalizing card orders:

1. `createCardLines` - Creates one or more card lines with associated recipients
2. `finalizeOrder` - Finalizes an order containing previously created card lines

## Enums

### CardType

| Value           | Description                            |
| --------------- | -------------------------------------- |
| `POSTCARD`      | A postcard with only a front panel     |
| `FLATCARD`      | A flat card with front and back panels |
| `TWO_PANEL`     | A card with two inside panels          |
| `THREE_PANEL`   | A card with three inside panels        |
| `TWO_PANEL_BIG` | A large format card with two panels    |

### CardOrientation

| Value        | Description                      |
| ------------ | -------------------------------- |
| `HORIZONTAL` | Card is in landscape orientation |
| `VERTICAL`   | Card is in portrait orientation  |

### CardPaperType

| Value      | Description                     |
| ---------- | ------------------------------- |
| `STANDARD` | Standard paper finish (default) |
| `SATIN`    | Satin paper finish              |
| `PEARL`    | Pearl paper finish              |

### CardPanelLocation

| Value           | Description                               |
| --------------- | ----------------------------------------- |
| `FRONT`         | Front panel of the card                   |
| `BACK`          | Back panel of the card                    |
| `INSIDE_LEFT`   | Inside left panel                         |
| `INSIDE_CENTER` | Inside center panel                       |
| `INSIDE_RIGHT`  | Inside right panel                        |
| `INSIDE_SPREAD` | Inside spread panel (spans across inside) |

## Types

### CardPanel

| Field      | Type                 | Description                       |
| ---------- | -------------------- | --------------------------------- |
| `id`       | `ID!`                | Unique identifier for the panel   |
| `url`      | `String!`            | URL of the panel image            |
| `location` | `CardPanelLocation!` | Location of the panel on the card |

### Address

| Field             | Type      | Description                       |
| ----------------- | --------- | --------------------------------- |
| `line1`           | `String!` | First line of address             |
| `line2`           | `String`  | Second line of address (optional) |
| `city`            | `String!` | City                              |
| `stateOrProvince` | `String!` | State or province                 |
| `postalCode`      | `String!` | Postal code                       |

### Contact

| Field                 | Type      | Description                       |
| --------------------- | --------- | --------------------------------- |
| `id`                  | `ID!`     | Unique identifier for the contact |
| `firstName`           | `String!` | First name of the contact         |
| `lastName`            | `String!` | Last name of the contact          |
| `companyName`         | `String`  | Company name (optional)           |
| `address1`            | `String!` | First line of address             |
| `address2`            | `String!` | Second line of address            |
| `city`                | `String!` | City                              |
| `state`               | `String!` | State or province                 |
| `postalCode`          | `String!` | Postal code                       |
| `countryCode`         | `String!` | Country code                      |
| `externalReferenceId` | `ID`      | External reference ID (optional)  |

### CardLine

| Field         | Type               | Description                         |
| ------------- | ------------------ | ----------------------------------- |
| `id`          | `ID!`              | Unique identifier for the card line |
| `type`        | `CardType!`        | Type of card                        |
| `orientation` | `CardOrientation!` | Orientation of the card             |
| `panels`      | `[CardPanel!]!`    | List of panels in the card          |
| `recipients`  | `[Contact!]!`      | List of recipients for this card    |

### Order

| Field       | Type           | Description                     |
| ----------- | -------------- | ------------------------------- |
| `id`        | `ID!`          | Unique identifier for the order |
| `cardLines` | `[CardLine!]!` | List of card lines in the order |

## Input Types

### CardLinePanelInput

| Field | Type      | Description            |
| ----- | --------- | ---------------------- |
| `url` | `String!` | URL of the panel image |

### ContactInput

| Field                 | Type      | Description                                                                                           |
| --------------------- | --------- | ----------------------------------------------------------------------------------------------------- |
| `firstName`           | `String!` | First name of the contact                                                                             |
| `lastName`            | `String!` | Last name of the contact                                                                              |
| `companyName`         | `String`  | Company name (optional)                                                                               |
| `address1`            | `String!` | First line of address                                                                                 |
| `address2`            | `String!` | Second line of address                                                                                |
| `city`                | `String!` | City                                                                                                  |
| `state`               | `String!` | State or province                                                                                     |
| `postalCode`          | `String!` | Postal code                                                                                           |
| `countryCode`         | `String!` | Country code                                                                                          |
| `externalReferenceId` | `ID`      | External reference ID (optional)                                                                      |
| `userVerified`        | `Boolean` | If true, the address will not be validated and the address provided will be trusted (defaults to false) |

### CardLineInput

| Field         | Type                  | Description                        |
| ------------- | --------------------- | ---------------------------------- |
| `type`        | `CardType!`           | Type of card                       |
| `orientation` | `CardOrientation!`    | Orientation of the card            |
| `recipients`  | `[ContactInput!]!`    | List of recipients for this card   |
| `front`       | `CardLinePanelInput!` | Front panel of the card (required) |
| `left`        | `CardLinePanelInput`  | Inside left panel (optional)       |
| `right`       | `CardLinePanelInput`  | Inside right panel (optional)      |
| `spread`      | `CardLinePanelInput`  | Inside spread panel (optional)     |
| `back`        | `CardLinePanelInput`  | Back panel of the card (optional)  |

### CreateCardLineInput

| Field                 | Type                | Description                                                     |
| --------------------- | ------------------- | --------------------------------------------------------------- |
| `cards`               | `[CardLineInput!]!` | Array of card line configurations                               |
| `shouldFinalizeOrder` | `Boolean`           | Whether to finalize the order immediately (defaults to `false`) |

## Mutations

### createCardLines

Creates one or more card lines with associated recipients.

```graphql
mutation createCardLines(input: [CreateCardLineInput!]!) {
  cardLines(input: $input) {
    errors {
      message
      code
      field
    }
    lines {
      ...CardLine
    }
  }
}
```

#### Input Parameters

| Parameter | Type                      | Description               |
| --------- | ------------------------- | ------------------------- |
| `input`   | `[CreateCardLineInput!]!` | Array of card line inputs |

#### Response

| Field    | Type                      | Description                                   |
| -------- | ------------------------- | --------------------------------------------- |
| `errors` | Array of error objects    | Any errors that occurred during the operation |
| `lines`  | Array of CardLine objects | The created card lines                        |

### finalizeOrder

Finalizes an order containing previously created card lines.

```graphql
mutation finalizeOrder(cardLineIds: [ID!]!) {
  order {
    errors {
      message
      code
      field
    }
    order {
      ...Order
    }
  }
}
```

#### Input Parameters

| Parameter     | Type     | Description                                    |
| ------------- | -------- | ---------------------------------------------- |
| `cardLineIds` | `[ID!]!` | Array of card line IDs to include in the order |

#### Response

| Field    | Type                   | Description                                   |
| -------- | ---------------------- | --------------------------------------------- |
| `errors` | Array of error objects | Any errors that occurred during the operation |
| `order`  | Order object           | The finalized order details                   |

## Example Requests

### Create Card Lines Request

```bash
curl -X POST https://api.example.com/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d @- << 'EOF'
{
  "query": "mutation CreateCardLines($input: [CreateCardLineInput!]!) { cardLines(input: $input) { errors { message code field } lines { id type orientation panels { id url location } recipients { id firstName lastName companyName address1 address2 city state postalCode countryCode } } } }",
  "variables": {
    "input": [
      {
        "cards": [
          {
            "type": "FLATCARD",
            "orientation": "HORIZONTAL",
            "recipients": [
              {
                "firstName": "John",
                "lastName": "Doe",
                "companyName": "Acme Inc",
                "address1": "123 Main St",
                "address2": "Apt 4B",
                "city": "San Francisco",
                "state": "CA",
                "postalCode": "94103",
                "countryCode": "US",
                "externalReferenceId": "customer_12345"
              }
            ],
            "front": {
              "url": "https://storage.example.com/images/card-front-12345.jpg"
            },
            "back": {
              "url": "https://storage.example.com/images/card-back-12345.jpg"
            }
          }
        ],
        "shouldFinalizeOrder": false
      }
    ]
  }
}
EOF
```

### Create Card Lines Response

```json
{
  "data": {
    "cardLines": {
      "errors": [],
      "lines": [
        {
          "id": "card_123456789",
          "type": "FLATCARD",
          "orientation": "HORIZONTAL",
          "panels": [
            {
              "id": "panel_front_123",
              "url": "https://storage.example.com/images/card-front-12345.jpg",
              "location": "FRONT"
            },
            {
              "id": "panel_back_123",
              "url": "https://storage.example.com/images/card-back-12345.jpg",
              "location": "BACK"
            }
          ],
          "recipients": [
            {
              "id": "contact_123",
              "firstName": "John",
              "lastName": "Doe",
              "companyName": "Acme Inc",
              "address1": "123 Main St",
              "address2": "Apt 4B",
              "city": "San Francisco",
              "state": "CA",
              "postalCode": "94103",
              "countryCode": "US"
            }
          ]
        }
      ]
    }
  }
}
```

### Finalize Order Request

```bash
curl -X POST https://api.example.com/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d @- << 'EOF'
{
  "query": "mutation FinalizeOrder($cardLineIds: [ID!]!) { order(cardLineIds: $cardLineIds) { errors { message code field } order { id cardLines { id type orientation recipients { firstName lastName } } } } }",
  "variables": {
    "cardLineIds": ["card_123456789", "card_987654321"]
  }
}
EOF
```

### Finalize Order Response

```json
{
  "data": {
    "order": {
      "errors": [],
      "order": {
        "id": "order_9876543210",
        "cardLines": [
          {
            "id": "card_123456789",
            "type": "FLATCARD",
            "orientation": "HORIZONTAL",
            "recipients": [
              {
                "firstName": "John",
                "lastName": "Doe"
              }
            ]
          },
          {
            "id": "card_987654321",
            "type": "TWO_PANEL",
            "orientation": "VERTICAL",
            "recipients": [
              {
                "firstName": "Jane",
                "lastName": "Smith"
              }
            ]
          }
        ]
      }
    }
  }
}
```

## Error Handling

Both mutations return an `errors` array when issues occur. Each error object contains:

| Field     | Type   | Description                                    |
| --------- | ------ | ---------------------------------------------- |
| `message` | String | Human-readable error message                   |
| `code`    | String | Error code for programmatic handling           |
| `field`   | String | The field that caused the error, if applicable |

## Notes

- Required panels vary by card type as follows:
  - `POSTCARD`: front panel only
  - `FLATCARD`: front and back panels
  - `TWO_PANEL`: front, back, and either (left and right) OR spread
  - `THREE_PANEL`: front, back, and either (left, center, and right) OR spread
  - `TWO_PANEL_BIG`: front, back, and either (left and right) OR spread
- The `shouldFinalizeOrder` parameter in `createCardLines` can be used to create and finalize an order in a single operation
- All requests require authentication via token in the Authorization header

## Common Errors

| Error Code                | Message                                   | Description                                              | Resolution                                                                                      |
| ------------------------- | ----------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `MISSING_REQUIRED_PANEL`  | "Required panel missing"                  | A required panel for the specified card type is missing. | Check the documentation for your selected card type to ensure all required panels are provided. |
| `INVALID_IMAGE_URL`       | "Image URL is invalid or inaccessible"    | The system could not access the provided image URL.      | Ensure the image URL is publicly accessible and correctly formatted.                            |
| `CARD_NOT_FOUND`          | "Card line not found"                     | The specified card line ID does not exist.               | Verify the card line ID is correct and the card line has been created.                          |
| `ORDER_ALREADY_FINALIZED` | "Order has already been finalized"        | Attempt to finalize an order that is already finalized.  | No action needed. The order is already being processed.                                         |
| `AUTHENTICATION_ERROR`    | "Invalid or expired authentication token" | The provided authentication token is invalid.            | Obtain a new authentication token and retry the request.                                        |
| `RATE_LIMIT_EXCEEDED`     | "Rate limit exceeded"                     | Too many requests in a short period.                     | Reduce the frequency of requests and implement exponential backoff.                             |

### Example Error Response

```json
{
  "data": {
    "cardLines": {
      "errors": [
        {
          "message": "Required panel missing",
          "code": "MISSING_REQUIRED_PANEL",
          "field": "back"
        }
      ],
      "lines": null
    }
  }
}
```

# Developer Notes

- We will download the image provided for each panel and re upload the image to our own S3 buckets.
- We need to determine the correct resolution/dimensions for images on panels
- Maybe we provide a separate endpoint to create panel previews to ensure panels doesn't get cropped/stretched?
- Need to add a max number of cards per mutation request (and then batch?)
- Implement Rate Limiting
- Dudupe images by creating a hash digest for panel URLs to get_or_create media asset
