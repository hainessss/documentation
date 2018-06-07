# WebSockets

Need some real-time goodness added to your application? You have come to the right place. The container exposes an interface for setting up a web socket server  implemented by [ws.js](https://github.com/websockets/ws). Utilization of socket connections is made available through the container's native pub/sub architecture. 

### Getting Started

Provisioning a web socket server starts with a defining a config file at `conf/socket.yaml`.

```
gateway:
  socket:
    path: "/connect"
    onconnect:
      publish: "socket:connection"
    onmessage:
      publish: "socket:generalMessage"
      types: 
          - type: "message_type_one"
            publish: "socket:type_one"
          - type: "message_type_two"
            publish: "socket:type_two"
    onclose:
      publish: "socket:close"
    send:
      subscribe: "socket:send"
    broadcast:
      subscribe: "socket:broadcast"
```

Let's breakdown the socket config object:

**path** - (String) - The HTTP endpoint where the web socket server will accept connection upgrade requests and provision a socket connection. This route accepts a namespace url parameter. A full connection url from the client side would look like `wss://service.bodhi-dev.io/connect?namespace=mynamespace` based on the above config.

**onconnect** - The config object for an event hook that publishes a message to the container event bus when a socket instance emits an `onconnect` event. 

* **publish** - (String) - The topic name of the message to publish to the event bus. 

**onmessage** - The config object for an event hook that publishes a message to the container event bus when a socket instance emits an `onmessage` event. 

* **publish** - (String) - The topic name of the message to publish to the event bus. This will be published every time a message is received by the socket. 
* **types** - (Array) - The `onmessage` config object allows for a more granular message handling strategy. The  types array is used to publish messages to the event bus when the socket receives messages of a certain type.  

	* **type** - (String) - The desired type property of incoming message that needs a specific event hook. 
	* **publish** - (String) - The topic name of the message to publish to the event bus. This will be published every time a message of the specified type is received by the socket. 

**onclose** - The config object for an event hook that publishes a message to the container event bus when a socket instance emits an `onclose` event. 

* **publish** - (String) - The topic name of the message to publish to the event bus. 
	
**send** - The config object for the interface that allows the container to send messages to specific connected clients.
	* **subscribe** - (String) - The topic name of the message to publish to the event bus when a message needs to be sent to a client. 

**broadcast** - The config object for the interface that allows the container to send messages to all connected socket clients.
	* **subscribe** - (String) - The topic name of the message to publish to the event bus when a message needs to be sent to all clients. 
	
### Connecting to the socket server

To connect to your shiny new web socket server: send an HTTP handshake request to the endpoint you specified in your config (`socket.connect`). Using a browser exposed api it might look something like `new WebSocket('wss://service.bodhi-dev.io/connect?namespace=alex')`.  Using curl it looks something like (local example):

```
curl 	--include
		--no-buffer      
		--header "Connection: Upgrade"      
		--header "Upgrade: websocket"      
		--header "Host: localhost:4000"      
		--header "Origin: http://localhost:4000"      
		--header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" 
		--header "Cookie: ID_TOKEN=validAuthToken"   
		--header "Sec-WebSocket-Version: 13" 
		http://localhost:4000/connect?namespace=alex
```

Socket clients are mapped to usernames by default. If the namespace parameter is present they are also mapped to namespace(highly recommended). For example, let's say user Alex opened four connections to the service across two namespaces. If namespace was not specified when opening the socket connections, the container would send messages to all of Alex's connections when emitting messages. 

### Handling Connections

To handle incoming connections from the client in your service container: start by adding a handler to your `app.js` file that listens for the topic specified in your `onconnection` config object.

`/lib/app.js`

```
[...]
	.addHandler('socket:connection', socketConnectionHandler, 'socket:connection', {
         ...miscDeps
    })
[...]
```

Then add a handler function to handle the connection event. 

`/lib/handlers/socketConnection.handler.js`


```
module.exports =  function (msg) {
	//get context from msg if needed
    const   username = get(msg, 'context.headers.jwt.preferred_username'),
            namespace = get(msg, 'context.params.namespace'),
            responseToClient = {
            	data: [...someRelevantData]
            }
            
	//send data back to client
    msg.reply(responseToClient);
}
	
```

The handler function takes and `EventBus` Message object as a parameter that will contain data about the username and namespace of the client trying to open a connection. Calling `msg.reply` in the context of the 'onconnection' handler will automatically send a response message back to the client with the data that was passed to `msg.reply`. In this case:

```
	{
		type: 'onconnect', 
		payload: {
            	data: [...someRelevantData]
        }
	}
	
```

Calling `message.fail` will send and error and close the socket.

### Handling Messages 

Handling messages works very similar to handling connections; the `msg.reply` and `msg.fail` methods offer a direct interface to respond to the client that sent the message.


`/lib/app.js`

```
[...]
	.addHandler('socket:generalMessage', socketMessageHandler, 'socket:generalMessage', {
         ...miscDeps
    })
[...]
```

However there are some differences: In message handlers calling `msg.fail` won't close the connection, but instead send a payload back to the client with the error:

```
	{
		type: 'error', 
		payload: someErrorObject
	}
	
```

In addition, the `type` property of messages sent back to the client by `msg.repy` are defined in the config objects.  They will reflect the `publish` property of the object. For example:

```
	{
		type: 'socket:generalMessage', 
		payload: {
		}
	}
```

`onmessage.publish` will be published to the event bus every time a socket receives a message. `onmessage.types[0].publish` will only be published when messages of that specific type(`onmessage.types[0].type`) are received. 

### Sending Messages

The handler for sending messages to clients is provided by default, so you do not need to add a custom handler to the container in `lib/app.js`.

Instead, you can simply publish a message to the event bus with the topic that was specified in the `send` config object and a payload that includes the username and namespace of the client that the container would like to send a message to.  

`anywhere/container/is/exposed.handler.js`

```
[...]

	//there are many ways to access the message bus api. Here is one way to access it in handlers. 
	
	const { producer } = this;
	
	producer.send('socket:send', { 
			username, 
			namespace, 
			type: 'message_type', 
			payload: { someData: relevantData } 
		}
	);
	
[...]
```

The `send` method takes two parameters: a topic and a data object. The topic will need to be the string specified under `send.broadcast` in the socket config file. The data object will has the following scheme.

**username** - (String) - Socket connections are mapped to platform usernames, so you will need to provide the username in order to identify the proper sockets to send messages to.  
**namespace** - (String) - Socket connections are also mapped to platform namespaces, so you will need to provide the namespace in order to identify the proper sockets to send messages to.  This ensures that there will not be message leaks across tenancies. 
**type** - (String) - The type property of the message that will be sent over the socket connection.
**payload** - (Any) - The payload of the message that will be sent over the socket connection.

### Broadcasting Messages

Broadcasted messages are sent to all connected clients! Similar to sending messages, the handler for broadcasting messages to socket connections is also provided by default.

To broadcast, publish a message to the event bus with the topic that was specified in the `broadcast` config object.  Username and namespace are no longer required as all connected clients will be receiving messages.

```
[...]

	//there are many ways to access the message bus api. Here is one way to access it in handlers. 
	
	const { producer } = this;
	
	producer.send('socket:send', { 
			type: 'message_type', 
			payload: { someData: relevantData } 
		}
	);
	
[...]
```

### Handling closed connections

The container monitors client connections via WebSocket's heartbeat protocol. This will keep idle connections alive.  In addition, it will close sockets in the container when a client goes dead or closes the connection, so socket connections do **NOT** need to be explicitly closed.

However, if there is additional logic or state management that needs to be performed when a socket is closed, a handler can be added. 

To handle closed socket connections in your service container: add a handler to your `app.js` file that listens for the topic specified in your `onclose` config object.

`/lib/app.js`

```
[...]
	.addHandler('socket:close', socketCloseHandler, 'socket:close', {
         ...miscDeps
    })
[...]
```


### Wrapping up

Annnnd, boom, you have web sockets! Need more? You can check out this web socket implementation that is currently out in the wild: [https://github.com/hotschedules/clarifi-notifications-service](https://github.com/hotschedules/clarifi-notifications-service). Go get 'em tiger.