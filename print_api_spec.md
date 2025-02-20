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

Simplified integration for clients
Consistent order processing
Maintains compatibility with existing workflows
Reduces API complexity by hiding implementation details

## Proposed GraphQL Spec

```gql
enum CardTye {
  FLAT_CARD
  TWO_PANEL_CARD
  THREE_PANEL_CARD
  BIG_CAR
  POSTCARD
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

input AddressInput {
  line1: String!
  line2: String
  city: String!
  stateOrProvince: String!
  postalCode: String!
}

input RecipientsInput {
  firstName: String!
  lastName: String!
  address: AddressInput!
}

input FlatCardInput {
  front: CardLinePanelInput!
  back: CardLinePanelInput!
}
# The Card Variation inputs have duplication but it keeps it straight forward for client as opposed to them guessing what panel will go where if we allow a list of panels. Also explictly lets them state an inside spread or left & right
input TwoPanelInput {
  front: CardLinePanelInput!
  left: CardLinePanelInput
  right: CardLinePanelInput
  spread: CardLinePanelInput
  back: CardLinePanelInput!
}

input ThreePanelInput {
  front: CardLinePanelInput!
  back: CardLinePanelInput!
  left: CardLinePanelInput
  center: CardLinePanelInput
  right: CardLinePanelInput
  spread: CardLinePanelInput
}

input BigCardInput {
  front: CardLinePanelInput!
  left: CardLinePanelInput
  right: CardLinePanelInput
  spread: CardLinePanelInput
  back: CardLinePanelInput!
}

input PostCardInput {
  front: CardLinePanelInput!
}

input CardLineInput {
  orientation: CardOrientation!
  paperType: CardPaperType # Default to standard if not given
  recipients: [RecipientsInput!]! #You can send a single card to multiple recipients if needed
  flatCard: FlatCardInput
  twoPanelCard: TwoPanelInput
  threePanelCard: ThreePanelInput
  bigCard: BigCardInput
  postCard: PostCardInput
}

input CardLineInput {
  cards: CardLineInput!
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

type Recipient {
  id: ID!
  firstName: String!
  lastName: String
  address: Address!
}

type CardLine {
  id: ID!
  type: CardType!
  orientation: CardOrientation!
  panels:  [CardPanel!]!
  recipients: [Recipients!]!
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

| Value              | Description                            |
| ------------------ | -------------------------------------- |
| `FLAT_CARD`        | A flat card with front and back panels |
| `TWO_PANEL_CARD`   | A card with two inside panels          |
| `THREE_PANEL_CARD` | A card with three inside panels        |
| `BIG_CAR`          | A large format card                    |
| `POSTCARD`         | A postcard with only a front panel     |

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

### Recipient

| Field       | Type       | Description                           |
| ----------- | ---------- | ------------------------------------- |
| `id`        | `ID!`      | Unique identifier for the recipient   |
| `firstName` | `String!`  | First name of the recipient           |
| `lastName`  | `String`   | Last name of the recipient (optional) |
| `address`   | `Address!` | Shipping address of the recipient     |

### CardLine

| Field         | Type               | Description                         |
| ------------- | ------------------ | ----------------------------------- |
| `id`          | `ID!`              | Unique identifier for the card line |
| `type`        | `CardType!`        | Type of card                        |
| `orientation` | `CardOrientation!` | Orientation of the card             |
| `panels`      | `[CardPanel!]!`    | List of panels in the card          |
| `recipients`  | `[Recipients!]!`   | List of recipients for this card    |

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

### AddressInput

| Field             | Type      | Description                       |
| ----------------- | --------- | --------------------------------- |
| `line1`           | `String!` | First line of address             |
| `line2`           | `String`  | Second line of address (optional) |
| `city`            | `String!` | City                              |
| `stateOrProvince` | `String!` | State or province                 |
| `postalCode`      | `String!` | Postal code                       |

### RecipientsInput

| Field       | Type            | Description                       |
| ----------- | --------------- | --------------------------------- |
| `firstName` | `String!`       | First name of the recipient       |
| `lastName`  | `String!`       | Last name of the recipient        |
| `address`   | `AddressInput!` | Shipping address of the recipient |

### FlatCardInput

| Field   | Type                  | Description             |
| ------- | --------------------- | ----------------------- |
| `front` | `CardLinePanelInput!` | Front panel of the card |
| `back`  | `CardLinePanelInput!` | Back panel of the card  |

### TwoPanelInput

| Field    | Type                  | Description                                        |
| -------- | --------------------- | -------------------------------------------------- |
| `front`  | `CardLinePanelInput!` | Front panel of the card                            |
| `left`   | `CardLinePanelInput`  | Inside left panel (required with right)            |
| `right`  | `CardLinePanelInput`  | Inside right panel (required with left)            |
| `spread` | `CardLinePanelInput`  | Inside spread panel (required if no left or right) |
| `back`   | `CardLinePanelInput!` | Back panel of the card                             |

### ThreePanelInput

| Field    | Type                  | Description                                                  |
| -------- | --------------------- | ------------------------------------------------------------ |
| `front`  | `CardLinePanelInput!` | Front panel of the card                                      |
| `back`   | `CardLinePanelInput!` | Back panel of the card                                       |
| `left`   | `CardLinePanelInput`  | Inside left panel (required with center and right)           |
| `center` | `CardLinePanelInput`  | Inside center panel (required with left and right)           |
| `right`  | `CardLinePanelInput`  | Inside right panel (required with left and right)            |
| `spread` | `CardLinePanelInput`  | Inside spread panel (required if no left, right, and center) |

### BigCardInput

| Field    | Type                  | Description                    |
| -------- | --------------------- | ------------------------------ |
| `front`  | `CardLinePanelInput!` | Front panel of the card        |
| `left`   | `CardLinePanelInput`  | Inside left panel (optional)   |
| `right`  | `CardLinePanelInput`  | Inside right panel (optional)  |
| `spread` | `CardLinePanelInput`  | Inside spread panel (optional) |
| `back`   | `CardLinePanelInput!` | Back panel of the card         |

### PostCardInput

| Field   | Type                  | Description                 |
| ------- | --------------------- | --------------------------- |
| `front` | `CardLinePanelInput!` | Front panel of the postcard |

### CardLineInput

| Field            | Type                  | Description                                               |
| ---------------- | --------------------- | --------------------------------------------------------- |
| `orientation`    | `CardOrientation!`    | Orientation of the card                                   |
| `paperType`      | `CardPaperType`       | Paper type (defaults to `STANDARD` if not specified)      |
| `recipients`     | `[RecipientsInput!]!` | List of recipients for this card                          |
| `flatCard`       | `FlatCardInput`       | Configuration for a flat card (mutually exclusive)        |
| `twoPanelCard`   | `TwoPanelInput`       | Configuration for a two-panel card (mutually exclusive)   |
| `threePanelCard` | `ThreePanelInput`     | Configuration for a three-panel card (mutually exclusive) |
| `bigCard`        | `BigCardInput`        | Configuration for a big card (mutually exclusive)         |
| `postCard`       | `PostCardInput`       | Configuration for a postcard (mutually exclusive)         |

### CreateCardLineInput

| Field                 | Type             | Description                                                     |
| --------------------- | ---------------- | --------------------------------------------------------------- |
| `cards`               | `CardLineInput!` | Card line configuration                                         |
| `shouldFinalizeOrder` | `Boolean`        | Whether to finalize the order immediately (defaults to `false`) |

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
    cards {
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
| `cards`  | Order object           | The finalized order details                   |

## Example Requests

### Create Card Lines Request

```bash
curl -X POST https://api.example.com/graphql \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d @- << 'EOF'
{
  "query": "mutation CreateCardLines($input: [CreateCardLineInput!]!) { cardLines(input: $input) { errors { message code field } lines { id type orientation panels { id url location } recipients { id firstName lastName address { line1 line2 city stateOrProvince postalCode } } } } }",
  "variables": {
    "input": [
      {
        "cards": {
          "orientation": "HORIZONTAL",
          "paperType": "PEARL",
          "recipients": [
            {
              "firstName": "John",
              "lastName": "Doe",
              "address": {
                "line1": "123 Main St",
                "line2": "Apt 4B",
                "city": "San Francisco",
                "stateOrProvince": "CA",
                "postalCode": "94103"
              }
            }
          ],
          "flatCard": {
            "front": {
              "url": "https://storage.example.com/images/card-front-12345.jpg"
            },
            "back": {
              "url": "https://storage.example.com/images/card-back-12345.jpg"
            }
          }
        },
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
          "type": "FLAT_CARD",
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
              "id": "recipient_123",
              "firstName": "John",
              "lastName": "Doe",
              "address": {
                "line1": "123 Main St",
                "line2": "Apt 4B",
                "city": "San Francisco",
                "stateOrProvince": "CA",
                "postalCode": "94103"
              }
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
  "query": "mutation FinalizeOrder($cardLineIds: [ID!]!) { order(cardLineIds: $cardLineIds) { errors { message code field } cards { id cardLines { id type orientation recipients { firstName lastName } } } } }",
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
      "cards": {
        "id": "order_9876543210",
        "cardLines": [
          {
            "id": "card_123456789",
            "type": "FLAT_CARD",
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
            "type": "TWO_PANEL_CARD",
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

- When creating card lines, you must specify exactly one card type (flatCard, twoPanelCard, etc.).
- The `shouldFinalizeOrder` parameter in `createCardLines` can be used to create and finalize an order in a single operation
- Required panels vary by card type - refer to the input type definitions
- For inside panels, you can either specify individual panels (left, center, right) or a spread panel, depending on your design.
- For two panel cards you must supply either left AND right OR spread
- For three panel cards you must supply either left, center, AND right OR spread
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
          "field": "flatCard.back"
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
