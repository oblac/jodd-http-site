# Connection

### HttpConnection

Socket HTTP communication is encapsulated by `HttpConnection` interface. On `send()`, **Jodd HTTP** will `open()` connection if not already opened. HTTP connections are created by the connection provider instance: `HttpConnectionProvider`. The default connection provider is socket-based and it always returns a new `SocketHttpConnection` instance - that simply wraps a `Socket` and opens it.

It is common to use custom `HttpConnectionProvider`, based on default implementation. For example, you may extend the `SocketHttpConnectionProvider` and override `createSocket()` method to return sockets from some pool, or sockets with a different timeout.

Alternatively, you may even provide an instance of `HttpConnection` directly, without any provider.

### Working with Sockets

As said, the default communication goes through the plain `Socket`. Since it is a common need to tweak socket behavior before sending data, here are two ways of how you can do it with **Jodd HTTP**.

#### SocketHttpConnection

Since we know the default type of `HttpConnection`, we can simply get the instance after explicitly calling the `open()` and cast it:

```java
HttpRequest request = HttpRequest.get()...;
request.open();

SocketHttpConnection httpConnection =
    (SocketHttpConnection) request.httpConnection();
Socket socket = httpConnection.getSocket();
socket.setSoTimeout(1000);

...

HttpResponse response = request.send();
```

#### SocketHttpConnectionProvider

The other way is to use custom `HttpConnectionProvider` based on `SocketHttpConnectionProvider`. So you may create your own provider like this:

```java
public class MyConnectionProvider extends SocketHttpConnectionProvider {
    protected Socket createSocket(
            SocketFactory socketFactory, String host, int port)
            throws IOException {
        Socket socket = super.createSocket(socketFactory, host, port);
        socket.setSoTimeout(1000);
        return socket;
    }
}
```

The custom provider is set by `withConnectionProvider()`:

```java
HttpResponse response = HttpRequest
    .get()
    .withConnectionProvider(new MyConnectionProvider())
    ...
    send();
```

Alternatively, you can explicitly open a connection with the `open()` method:

```java
HttpConnectionProvider connectionProvider = new MyConnectionProvider();
...
HttpRequest request = HttpRequest.get()...;
HttpResponse response = request.open(connectionProvider).send();
```

{% hint style="danger" %}
Once when a connection is open by `open()` method, you can not alter it via the **Jodd HTTP** interface. For example, setting the timeouts _after_ the open will have no effect.
{% endhint %}

### Keep-Alive

By default, all connections are marked as _closed_, to keep servers happy. **Jodd HTTP** allows usage of permanent connections through the keep-alive mode. The `HttpConnection` is opened on the first request and then re-used in communication session; the socked is not opened again if not needed and therefore it is reused for several requests.

There are several ways how to do this. The easiest way is the following:

```java
HttpRequest request = HttpRequest.get("http://jodd.org");
HttpResponse response = request.connectionKeepAlive(true).send();

// next request
request = HttpRequest.get("http://jodd.org/jodd.css");
response = request.keepAlive(response, true).send();

...

// last request
request = HttpRequest.get("http://jodd.org/jodd.png");
response = request.keepAlive(response, false).send();

// optionally
//response.close();
```

This example fires several requests over the same `HttpConnection` \(i.e. the same socket\). When in 'keep-alive' mode, _HTTP_ continues using the existing connection, while paying attention to server responses. If the server explicitly requires a connection to be closed, _HTTP_ will close it and then it will open a new connection to continue your session. You don't have to worry about this, just keep calling `keepAlive()` and it will magically do everything for you in the background. Just don't forget to pass `false` argument to the last call to indicate the server that is the last connection and that we want to close after receiving the last response. \(if for some reasons the server does not respond correctly, you may close communication on the client-side with an explicit call to `response.close()`\). One more thing - if a new connection has to be opened during this persistent session \(when e.g. keep-alive max counter is finished or timeout expired\) the same connection provider will be used as for the initial, first connection.

### Proxy

`HttpConnectionProvider` also allows you to specify the proxy. Just provide the `ProxyInfo` instance with the information about the used proxy \(type, address, port, username, password\):

```java
    SocketHttpConnectionProvider s = new SocketHttpConnectionProvider();
    s.useProxy(ProxyInfo.httpProxy("proxy_url", 1090, null, null));

    HttpResponse response = HttpRequest
        .get("http://jodd.org/")
        .withConnectionProvider(s)
        .send();
```

**Jodd HTTP** supports HTTP, SOCKS4, and SOCKE5 proxy types.

### Parse from InputStreams

Both `HttpRequest` and `HttpResponse` have a method `readFrom(InputStream)`. Basically, you can parse the input stream with these methods. This is, for example, how you can read the request on server-side.

