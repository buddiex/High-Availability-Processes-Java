# High-Availability-Processes-Java

## Tuple Service

This service provides a key value pair tuple service to clients.

### Details
- it a collection of alpha numeric keys and values tuples with at most one value per key.
- provides operation GET, PUT, POST, DELETE
- GET operation takes a tuple with a way to query the key or the values with a query expression eg. (keyexp, valexp). it returns all (key, value) tuples that matches the given expression.
- PUT operation takes a list of (key, value) tuple and saves them if the keys do not exist. it returns all (key, value) that where not saved with proper 
- POST operation takes a list of (key, value) tuple and with proper error message.
- DELETE operation takes a tuple with a way to query keys or the values with a query expression eg. (keyexp, valexp). it deletes all (key, value) pairs in the space that matches the keys or values query express
- replies any unimplemented operation with appropriate error message


## Proxy Server
The proxy server receives requests from clients, maintains connections with clients, forwards request to primary server and tracks requests so they can be returned to originating client.

### Details
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


##Primary Server

The primary server is responsible for receiving all requests for the service from the system's proxy. A valid request must include a connection identifier that uniquely identifies a client’s request. It processes the request and returns an appropriate response which includes the requests identifier. It updates the backup server in parallel with the processing of all requests (POST, PUT and DELETE) with the exception of GET requests. It fails if any of the commands are malformed or the file is unreadable or malformed.

###Details
####Arguments:
1.  Shutdown SAP for receiving shutdown commands.
2.  Request SAP for obtaining requests from the proxy.
3.  Backup-request SAP for sending update requests to backup.
4.  Heartbeat SAP for receiving heartbeat messages from backup.
5.  Tuple space storage file on shutdown.
6.  Process id if primary. is null to indicate primary initialization
7.  A file from which to initialize the tuple space data on startup.

####Phases of operations
#####In the initialization phase:
- Initialize a heartbeat SAP for receiving heartbeat messages from the backup and shutdown SAP connection for receiving shutdown messages. Shutdown if initialization fails.
- Start the backup using arguments 1-5 as appropriate for the secondary, process id of the primary for argument 6 and null value argument 7. Shutdown, if the backup can not be started.
- Establish heartbeat and backup-request SAP connection with backup server. Shutdown if connection is not established within a period of time.
- Use PUT statement to initialize the backup's tuple space. Shutdown if connection is not established within a period of time.
- Establish request SAP connection with proxy. Shutdown if connection is not established within a period of time.
#####Steady-state operation:
- Repeatedly accept and respond to requests from the proxy per service requirements via its request SAP.
- Periodically accept and respond to heartbeat messages from the backup.
- On proxy failure, self-failure, or a shutdown message from the proxy, terminate the backup and shutdown.
- On backup failure, make a set number of attempts to restart the backup and initialize its tuple space else shutdown.
#####NB: Write the tuple space to a file before any shutdown 

##Backup Server

The backup server receives request relayed from primary server to keep its tuple space in sync. It also maintains heartbeat messages with primary switching to primary if it the heartbeat stop.

##Details
###Arguments:
Same as primary excerpt argument 6 which is the process id of the primary server and argument 7 is a null value.

###Phases of operations
####In the initialization phase:
Initializes like primary without starting another backup.
####Steady-state operation:
- Repeatedly accept and respond to requests from primary as per service requirements via its request SAP.
- Periodically accept and respond to heartbeat messages from the primary.
- On self-failure, or a shutdown message from primary it shut's down.
- On primary failure, terminate primary, close primary request sap (argument 3) initialize as primary as describe in primary's requirements.
#####NB: Write the tuple space to a file before any shutdown 