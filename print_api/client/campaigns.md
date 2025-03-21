## Creating a campaign

After you have an account, you need to create or purchase a campaign so you can send it via the API. To create a campaign:

1. On the sidebar, click on "Campaigns"
2. Make sure you're on the "My Campaigns" tab
3. Select the "+" button at the right side
4. Choose a campaign name and click on "Create Campaign"
5. Click on "Add a Card"
6. Choose any catalog card or "Build Your Own" and card options, click "next" to start creating your card
7. Customize your card in any way you want and click on "Add to campaign"
8. Click on "Save"

After this you should be able to see the campaign if you go back to the Campaigns list and refresh.

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
