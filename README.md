# Protocol Specification

## Backend

### Overview
The backend consists of a mesh network of REST API servers that communicate with each other to transmit client packets. Clients use separate servers for transmission (TX) and reception (RX) to protect their anonymity. Clients send messages by POSTing to the TX server and retrieve messages by GETting from the RX server. Servers must implement mechanisms to detect and handle misbehaving nodes that refuse to relay messages or drop packets.

### Key Features
1. **Mesh Network of REST API Servers**: Servers interconnect to relay client packets.
2. **Separate TX and RX Servers**: Clients use different servers for sending and receiving messages to protect anonymity.
3. **Message Broadcasting**: Client messages are broadcasted to find the recipient's server.
4. **Message Retention for Undelivered Messages**: Undelivered messages are stored in a database with a seven-day expiration.
5. **Message Retrieval**: Clients can request undelivered messages periodically.
6. **RSA 2048 Encryption**: Messages are encrypted using RSA 2048 to ensure security and privacy.
7. **Signature Verification**: Messages include a signature that is encrypted separately to prevent unauthorized verification.
8. **Server Behavior Monitoring**: Mechanisms to detect and handle misbehaving servers.

### Packet Structure (JSON)
- **Client Message Packet (POST)**
  ```json
  {
    "recipient": "recipient_public_key",
    "message": "base64_encoded_encrypted_json",
    "signature": "base64_encoded_encrypted_signature"
  }
  ```
  - **Encrypted JSON Object**
    ```json
    {
      "sender": "sender_public_key",
      "content": "message_content",
      "uuid": "message_uuid",
      "timestamp": "ISO_8601_timestamp"
    }
    ```
  - **Encrypted Signature**
    ```json
    {
      "signature": "base64_encoded_signature_of_encrypted_message"
    }
    ```

- **Message Retrieval Packet (GET)**
  ```json
  {
    "public_key": "client_public_key",
    "signature": "base64_encoded_signature_of_phrase"
  }
  ```
  - **Phrase for Signature**
    ```plaintext
    "I AM WHO I SAY I AM"
    ```

### Example Encrypted Message Content
- **Example Encrypted JSON Object (Base64 Encoded)**
  ```json
  {
    "sender": "sender_public_key",
    "content": "Hello, this is a test message!",
    "uuid": "123e4567-e89b-12d3-a456-426614174000",
    "timestamp": "2024-06-15T12:34:56Z"
  }
  ```

### Example Encrypted Signature
- **Example Signature of Encrypted Message (Base64 Encoded)**
  ```json
  {
    "signature": "base64_encoded_signature"
  }
  ```

## Frontend

### Overview
Clients connect to the tx and rx servers with the lowest load from their known list of servers. Security improves over time as more servers are learned, reducing dependence on the default servers. A local contact list is used to manage metadata such as profile pictures and names. Clients send fake GET and POST packets to enhance anonymity.

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

### Decentralization
The plan is to maintain a decentralized network, subject to change based on public meetings.

### Security Measures
- **Message Signing**: Messages are signed with keys to prevent MITM attacks.
- **Dynamic Server List Sharing**: Ensures no single server is trusted. Initial connections to compromised servers can be detected through key signatures.
- **Login with Private Key**: Users log in using a private key and send messages using the recipient's public key. Usernames are used in the UI.
- **Dual Proxy**: The clients will use separate proxy servers for their TX and RX traffic in case the TX and RX servers are owned by the same person. That way their TX and RX traffic cannot be mapped back to the same IP address by a bad actor.
### Packet Structure (JSON)
- **Server List Request Packet**
  ```json
  {
    "type": "server_list_request",
    "requester_id": "client_or_server_id",
    "timestamp": "ISO_8601_timestamp"
  }
  ```
- **Server List Response Packet**
  ```json
  {
    "type": "server_list_response",
    "known_servers": ["server_id_1", "server_id_2", ...],
    "timestamp": "ISO_8601_timestamp"
  }
  ```

### Example of Packet JSON Structures
- **Message Signing Example**
  ```json
  {
    "type": "signed_message",
    "message": {
      "content": "original_message_content",
      "timestamp": "ISO_8601_timestamp"
    },
    "signature": "message_signature",
    "public_key": "sender_public_key"
  }
  ```

## Conclusion
This document outlines the protocol's backend and frontend architecture, security measures, and packet structures in JSON. The protocol aims for a decentralized, secure communication network leveraging a mesh of REST API servers, dynamic server lists, and strong encryption practices, while implementing mechanisms to detect and handle misbehaving servers.
