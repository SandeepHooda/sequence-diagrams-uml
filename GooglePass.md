::: mermaid
sequenceDiagram
    autonumber
    actor User
    participant Client as Email Client / Browser
    participant Walletron as Walletron Server
    participant Google as Google Wallet API

    %% Step 1 & 2: Invitation and Verification
    Walletron->>Client: Send email invitation to enroll
    Client->>User: Display email
    User->>Client: Click Enrollment link, enter Account Number & Zip Code
    Client->>Walletron: Account number & zip code
    %% Step 3: Payload Creation
    Note over Walletron: Create JSON Payload<br/>ObjectID: IssuerID.serialNumber
    %% Step 4: Pre-insert the pass
    Walletron->>Google:  /walletobjects/v1/genericobject/insert (JSON Payload)
    Google-->>Walletron: 200 OK (Pass stored, unassociated)
    %% Step 5: Thin JWT & Redirect
    Note over Walletron: Generate "Thin JWT"<br/>(References objectId only)
    Walletron-->>Client: HTTP 302 Redirect<br/>Location: https://pay.google.com/gp/v/save/{JWT}
    %% Step 6: Browser navigates to Google
    Client->>Google: Follow Redirect (GET /gp/v/save/{JWT})
    Google-->>Client: Serve "Add to Google Wallet" UI
    %% Step 7: User saves pass
    Client->>User: Display Add to Wallet screen
    User->>Client: Click "Add to Google Wallet"
    Client->>Google: Confirm Save
    Note over Google: Pass associated with<br/>User's Google Account
    %% Step 8: Callback Notification
    Google->>Walletron: POST /your-callback-endpoint ("add to wallet" event)
    Walletron-->>Google: 200 OK