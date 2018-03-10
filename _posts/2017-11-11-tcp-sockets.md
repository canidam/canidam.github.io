---
title: "How to create a TCP session between client and server?"
tags: [python]
---

I was learning tcp sockets in Python, and in order to understand it I had written a
simple client-server model that echos the client input back to the screen.
This was made to understand how sockets works, and check their different options.
These are my notes from that session.

Server-client model has two players; the server side that listens for new connections
and serve them, and the client side. In order for the server to serve more than a single
client, it has to be multi-threaded. Let's examine how the *server.py* looks like.


```python
import socket
import threading

class ThreadedServer(object):
    def __init__(self):
        self.server_addr = ('localhost', 2222)
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # create socket object
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        self.sock.bind(self.server_addr)

    def listen(self):
        self.sock.listen(4)
        while True:
            client, address = self.sock.accept()  # wait for connections; blocking
            client.settimeout(60)
            print 'got connection from {}'.format(address)
            threading.Thread(target=self.client_handler, args=(client, address)).start()

    def client_handler(self, client, address):
        size = 1024
        while True:
            try:
                data = client.recv(size)  # reading data from connection; blocking
                if data:
                    print 'got data: {}'.format(data)
                    # Set the response to echo back the recieved data
                    response = data
                    client.send(response)
                else:
                    raise Exception('Client disconnected')
            except Exception as e:
                client.close()
                return False


if __name__ == "__main__":
    ThreadedServer().listen()
```

The server object is initialized with *localhost* and port *2222*. It then creates
a TCP socket, and binds it.

## listen
**listen** task is to wait for new connections; the socket `.accept()` method is blocking,
so the thread will be idling until a someone comes. When the client creates a connection,
this thread accepts the connection and print a log message on the screen. It then opens
a new thread which runs the `client_handler` code, then go back to *LISTENING* state,
waiting for more clients. (multi-threaded server)

**client_handler** is the function that.. handles the client. That means after the server
listening thread accepts the connection, it executes this function in a different thread.
the size defines how many bytes to read on `recv()`. recv is blocking, that means it will sit waiting
until the client sends data to the server. You can configure timeout to this function, in which case
if there is nothing to read, recv returns an empty string. In some implementations, I saw this actually
raised an exception. This is why I throw an exception in the else clause, and captures all kind of exceptions
from recv. If we reached this point, it means session is closed.

When data arrives, we can read it from the buffer using `client.recv()` and send data back to the client
using.. you guessed it right: `client.send(data)`.

That's it for the server side. Not too complicated.
