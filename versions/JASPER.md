# Protocol Specification
Version: 1.0.0

Codename: Jasper
## Backend

### Overview
The backend consists of a mesh network of REST API servers that communicate with each other to transmit client packets. Clients use separate servers for transmission (TX) and reception (RX) to protect their anonymity. Clients send messages to the TX server and retrieve messages from the RX server. Servers must implement mechanisms to detect and handle misbehaving nodes that refuse to relay messages or drop packets. Any encrypted data will be base64 encoded and stored as a string. Files can be embedded using GH markdown syntax in messages. The recommended file distribution platform is IPFS. Servers will store messages in a cache for 7 days before they expire. It is recommended to use Redis to implement this functionality due to its speed. Servers will perform a message sync with each other every minute and a server sync every hour.

### Key Features
1. **Mesh Network of REST API Servers**: Servers interconnect to relay client packets.
2. **Separate TX and RX Servers**: Clients use different servers for sending and receiving messages to protect anonymity. Clients should use a separate server for EACH key they interact with. Ex. sending a message in a GC or having multiple receive keys configured.
3. **Message Broadcasting**: Client messages are broadcasted to find the recipient's server.
4. **Message Retention for Undelivered Messages**: Undelivered messages are stored in a database with a seven-day expiration.
5. **Message Retrieval**: Clients can request undelivered messages periodically.
6. **RSA 4096 Encryption**: Messages are encrypted using RSA 4096 keys to ensure security and privacy.
7. **Signature Verification**: Messages include a signature that is encrypted separately to prevent unauthorized verification.
8. **Anonymous & Transparent Modes**: Users are given the option to sacrifice some anonymity in order to make calls or message faster. This will still be kept as secure as possible by the restraints of the WebRTC protocol.

### Anonymous Mode (REST API)
This mode uses the TX and RX servers in order to send and receive messages. By utilizing a different server for these two functions, the user's identity is kept secret. This is because when sending and receiving messages only the recipients key is visible, whether it be your key that you are trying to fetch the messages for, or your friends key that you are sending messages to. With a separate TX and RX server, these keys will never be associated together unless the servers are owned by the same person. This is an issue the protocol developers plan on addressing in the future.

### Transparent Mode (WebRTC)
Transparent mode makes use of WebRTC in order to send packets between clients faster. The content of these packets will still be encrypted. However, the owner of the STUN server in use will be able to see which 2 clients are communicating. These clients will need to expose their public keys at the beginning of contact in order to discover each other and start the transaction, thus breaching anonymity.

### REST API Endpoints
- `/fetch`
  - Method Type: `POST`
  - Request Body:
    ```json
    {
      "recipient": "recipient_public_key",
      "hashes": [
        "message_sha256"
      ]
    }
    ```
  - Response Body:
    ```json
    {
      "messages": [
        {
          "recipient": "recipient_public_key",
          "message": "encrypted_json",
          "hash": "message_sha256",
          "signature": "encrypted_json"
        }
      ]
    }
    ```
  - General Notes:  
    You can provide either recipient OR hashes. Recipient is intended for clients fetching messages and hashes are intended for servers fetching messages. The presence of the recipient field will be checked first by a server when the endpoint is called.
- `/send`
  - Method Type: `POST`
  - Request Body:
    ```json
    {
      "recipient": "recipient_public_key",
      "message": "encrypted_json",
      "hash": "message_sha256",
      "signature": "encrypted_signature"
    }
    ```
- `/sync/servers`
  - Method Type: `POST`
  - Request Body:
    ```json
    {
      "servers": [
        {
          "ip": "IPv4|IPv6|DOMAIN",
          "port": "8080"
        }
      ]
    }
    ```
  - Response Body:
    ```json
    {
      "servers": [
        {
          "ip": "IPv4|IPv6|DOMAIN",
          "port": "8080"
        }
      ]
    }
    ```
  - General Notes:  
    Server A will send its known servers list to Server B. Server B will respond with any servers Server A did not have in its known servers list. Then, Server B will add any servers it does not already have to its known server list. 
- `/sync/hashes`
  - Method Type: `POST`
  - Request Body:
    ```json
    {
      "hashes": [
        "message_sha256"
      ]
    }
    ```
  - Response Body:
    ```json
    {
      "hashes": [
        "message_sha256"
      ]
    }
    ```
  - General Notes:  
    Server A will send the hashes of all the messages it has stored to Server B. Server B will respond with the hashes of any messages Server A does not have stored. Then, Server B will fetch any messages it does not already have from Server A based on their hashes.

### Objects
- `message`
  - Content:
    ```json
    {
      "sender": "sender_public_key",
      "channel_id": "UUID",
      "type": "MESSAGE|EDIT|DELETE|REACT",
      "content": "message_content",
      "id": "message_id",
      "timestamp": "ISO_8601_timestamp"
    }
    ```
  - General Notes:  
    The channel ID will be generated when the first message is ever sent. In DMs, this will most likely be disregarded although may be used for message sync in the future. In GCs, the same message packet will be sent out multiple times (once for each recipient) via a different server each time to increase anonymity. Clients can then determine which GC it was sent in based on the channel ID that is included in the encrypted message body.
  - Types:
    - `MESSAGE`: A new message will be created in the chat with the specified id.
    - `EDIT`: The message with the given id will be updated to the new content.
    - `DELETE`: The message with the given id will be removed from the chat. Anything in the content field will be ignored.
    - `REACT`: A reaction will be added to the given message id. The reaction name will be stored in the content field of the packet in the form of a **:reaction:**. Users must have the reaction in the reactions database on their client for it to appear.

## Frontend

### Overview
Clients connect to the TX and RX servers with the lowest load from their known list of servers. Security improves over time as more servers are learned, reducing dependence on the default servers. A local contact list is used to manage metadata such as profile pictures and names.

### Key Features
1. **Optimal Server Connection**: Connect to the server with the lowest load.
2. **Incremental Security**: As clients learn about more servers, security increases.
3. **Contact List**: Use a local contact list to manage metadata such as profile pictures and names.
4. **Multiple Location Login**: Mechanism to handle message sync across multiple login locations.
5. **Voice Call Support**: Clients will use an encrypted STUN server on the backend to use WebRTC to make calls. This sacrifices some anonymity but is required for a usable chat app experience.

### Communication Options
- **REST API**: Default method for sending and receiving messages.
- **Encrypted STUN Server**: Clients can use an encrypted STUN server for faster communication and WebRTC calls, while maintaining some anonymity.

## General Notes

### Security Measures
- **Message Signing**: Messages are signed with keys to prevent MITM attacks.
- **Dynamic Server List Sharing**: Ensures no single server is trusted. Initial connections to compromised servers can be detected through key signatures.
- **Login with Private Key**: Users log in using a private key and send messages using the recipient's public key. Usernames are used in the UI.