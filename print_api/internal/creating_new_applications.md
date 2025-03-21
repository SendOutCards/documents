# Creating Applications for clients of the Public API

1. Go to https://www.sendoutcards.com/.socadmin/soc_oauth2/application/add/ and fill in the data:

   - **User**: Use the search icon to fill in the user who is responsible for the application (this can be yourself)
   - **Redirect URIs**: `http://localhost/*`
   - **Client Type**: Public
   - **Authorization grant type**: Authorization code
   - **Name**: {Client Name} Public API (change {Client Name} to the name of the client)
   - **Algorithm**: RSA with SHA-2 256
   - **Scopes**: Public API Scope

2. Copy the client secret and save it now, you will need to send it to them.

> **Important**: Don't forget to copy the client secret before saving.
