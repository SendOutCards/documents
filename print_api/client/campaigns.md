## Sending a campaign

Send the following data on a POST request to the public API endpoint. Update the data on `variables` to match the campaign UUID from the last step and the shipping information for the contact you want to send to. `externalReference` should be the contact ID on your system, so that we can re-use it on future orders.

### Request details:

- **Headers**:
  `{"Authorization": "{token_type} {access_token}"}`

(add your actual token_type and access_token here)

- **Body**:

```json
{
  "query": "mutation CreateOrderFromCampaignWithContacts($campaignUuid:ID!,$contacts: [ContactInput!]!){createOrderFromCampaignWithContacts(input:{campaignUuid: $campaignUuid,contacts:$contacts}){order{uuid}}}",
  "variables": {
    "campaignUuid": "e3d10ee0-d1bc-4edb-a78c-d8f2ee52c802",
    "contacts": [
      {
        "firstName": "John",
        "lastName": "Doe",
        "companyName": "",
        "address1": "1 Main St",
        "address2": "",
        "city": "New York",
        "state": "NY",
        "postalCode": "10001",
        "countryCode": "US",
        "externalReference": "654321"
      }
    ]
  },
  "operationName": "CreateOrderFromCampaignWithContacts"
}
```
