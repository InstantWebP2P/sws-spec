Secure WebSocket(alias SWS) - Nacl public key exchange over WebSocket.
-------------------------------------------------------------------------------------------

### SWS' handshake process consists of six steps

1. WebSocket client connect to WebSocketServer,
2. WebSocket client send ClientHello message to WebSocketServer with their public key/certification information and a Random number,
3. WebSocket server verify ClientHello message and decide to continue or reject handshake request,
```
   a. If WebSocket server grant client's handshake request, then send it's public key/certification information,  
      another encrypted Random number and Share-Key as ServerHello message back  
   b. If WebSocket server reject client's handshake request, then [send invalid certification as ServerHello message back and] close WebSocket client,
```
4. WebSocket client got Server's ServerHello message, then verify Server's certification and decide to continue or reject handshake process,
```
   a. If WebSocket client grant server's handshake request, then send encrypted Share-Key as ClientReady message to Server, then switch to secure-context,  
   b. If WebSocket client reject server's handshake request, then close WebSocket client,
```
5. WebSocket server got Client's ClientReady message, extract their Share-Key, then switch to secure-context,
6. Both WebSocket client and server run on secure-context, done.

### All exchanged data format is JSON

### Client/Server exchanged handshake JSON message define

* Handshake start ...
* -> ClientHello message, send from client to server
```js
  Version 1: {  
                  version: 1,          // protocol version  
                      opc: 0,          // message opcode as 0    
        client_public_key: byte array, // client side NACL Box public key  
                    nonce: byte array  // client generated 8 bytes of random number  
  }    
  Version 2: {  
                  version: 2,          // protocol version, will request server's public key NACL cert  
                      opc: 0,          // message opcode as 0    
        client_public_key: byte array, // client side NACL Box public key  
                    nonce: byte array  // client generated 8 bytes of random number  
  }
  ```
* <- ServerHello message, send from server to client, which responds to ClientHello message
```js
  Version 1: {  
                  version: 1,          // protocol version  
                      opc: 1,          // message opcode as 1    
        server_public_key: byte array, // server side NACL Box public key  
             s_blackbox_a: byte array  // authenticated-encrypted server-sent's ( nonce(8bytes) + sharekey ) using  
                                       // (server's Box secretkey and (client-sent's Box publickey and nonce(8bytes)))  
  }    
  Version 2: {  
                  version: 2,          // protocol version  
                      opc: 1,          // message opcode as 1    
        server_public_key: byte array, // server side NACL Box public key  
             s_blackbox_a: byte array  // authenticated-encrypted server-sent's ( nonce(8bytes) + sharekey +  
                                       // {cert,requireCert}(utf8 encoded json string) ) using  
                                       // (server's Box secretkey and (client-sent's Box publickey and nonce(8bytes)))  
  }
  ```
* -> ClientReady message, send from client to server, which responds to ServerHello message
```js
  Version 1: {  
                  version: 1,          // protocol version  
                      opc: 2,          // message opcode as 2    
             s_blackbox_a: byte array  // authenticated-encrypted client-sent's ( nonce(8bytes) + sharekey ) using  
                                       // (client's Box secretkey and (server-sent's Box publickey and nonce(8bytes)))  
  }    
  Version 2: {  
                  version: 2,          // protocol version  
                      opc: 2,          // message opcode as 2    
             s_blackbox_a: byte array  // authenticated-encrypted client-sent's ( nonce(8bytes) + sharekey +  
                                       // cert(utf8 encoded json string) ) using  
                                       // (client's Box secretkey and (server-sent's Box publickey and nonce(8bytes)))  
  }
  ```
* Handshake done.  

### for NACL Cert System define, refer to https://github.com/InstantWebP2P/nacl-cert

### Copyright tom zhou<iwebpp@gmail.com>

