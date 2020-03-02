# High-Availability-Processes-Java

## Tuple Service
## Summary
this service provides a key value pair tuple service to clients.

### Details
- it a collection of alpha numeric keys and values tuples with at most one value per key.
- provides operation GET, PUT, POST, DELETE
- GET operation takes a tuple with a way to query the key or the values with a query expression eg. (keyexp, valexp). it returns all (key, value) tuples that matches the given expression.
- PUT operation takes a list of (key, value) tuple and saves them if the keys do not exist. it returns all (key, value) that where not saved with proper 
- PUT operation takes a list of (key, value) tuple and with proper error message.
- DELETE operation takes a tuple with a way to query keys or the values with a query expression eg. (keyexp, valexp). it deletes all (key, value) pairs in the space that matches the keys or values query express
- replies any unimplemented operation with appropriate error message


## Proxy Server
### Summary
The proxy server receives requests from clients, maintains connections with clients, forwards request to primary server and tracks requests so they can be returned to originating client.

### Details:
The proxy: 
- Maintains three Service Access Points (SAP)
   - a local SAP for receiving connections from clients
   - a local SAP for receiving connections from the primary
   - a local SAP for receiving shutdown commands
   
   When the connections to these SAPs are broken, it blocks its service to reinitialize them prioritizing the primary server connection, clients connection and then shutdown connections. if primary connection cannot be reinitialized after a set timeout the proxy shuts down.

- Maintains a connection-oriented tcp connection with a set number of clients and refuses additional clients with an appropriate error message
- Queues up and sends all connection requests to the primary server tagging with an identifier to track the requesting client. If the request fails the connection is considered as a broken connection to be reinitialize. 
- Relay response back to requesting clients in the order the response was received stripping of the client identifier tag.
- Shuts down when it receives a shutdown request. it relays the request to the primary server before shutting down.

