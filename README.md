# High-Availability-Processes-Java
## Proxy Server
### Summary:
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

