
Primary Server

        **Summary**

        The primary server is responsible for receiving all requests for the service from the system's proxy. A valid request must include a connection identifier that uniquely identifies a client’s request. It processes the request and returns an appropriate response which includes the requests identifier. It updates the backup server in parallel with the processing of all requests (POST, PUT and DELETE) with the exception of GET requests. It fails if any of the commands are malformed or the file is unreadable or malformed.

        **Core Operations of the primary server (Command Arguments)**

        1.  Local SAP for receiving shutdown commands.

        2.  Remote SAP for obtaining requests from the proxy.

        3.  Backup-Local SAP for sending update requests to the backup.

        4.  Primary-Local SAP for receiving heartbeat messages from the backup.

        5.  Write tuple space data into a file on shutdown.

        6.  A null value to denote that the process is starting as the primary.

        7.  A file from which to initialize the tuple space data on startup.




        **Phases of operations**

        -   In the initial phase of operation, the primary shall:


        -   initialize a primary-local SAP for receiving heartbeat messages from the backup.


        -   write the contents of the tuple space to a file and shutdown if initialization fails.


        -   start the backup using arguments 1-5.

        -   use the primary server ID to kill the primary.


        -   write the contents of the tuple space to a file and shutdown, if the backup can not be started.


        -   use Primary-Local SAP for receiving heartbeat messages to establish a connection from the backup server.


        -   write the contents of the tuple space to a file and shutdown, if connection is not established within a period of time.


        -   use backup-local SAP for sending update requests to the backup.


        -   write the contents of the tuple space to a file and shutdown, if connection is not established within a period of time.


        -   Use PUT statement to initialize the backup's tuple space.


        -   write the contents of the tuple space to a file and shutdown, if connection is not established within a period of time.


        -   Connect to the remote SAP for obtaining requests from the proxy.


        -   write the contents of the tuple space to a file and shutdown, if connection is not established within a period of time.


        -   In its steady-state phase of operation, the primary shall:


        -   connect to the remote SAP for obtaining requests from the proxy.

        -   repeatedly accept and respond to requests from the proxy per service requirements.

        -   periodically accept and respond to heartbeat messages from the backup.

        -   terminate the backup if proxy or primary server fails.

        -   write the current contents of the tuple space to the file named in argument 5 and shut down.

        -   make a reasonable number of attempts to restart the backup, initializing its tuple space if backup fails. If failure persists, write the current contents of the tuple space to the file named in argument 5 and shut down.