# HttpTunnel

**Jodd HTTP** is so flexible that you can easily build a HTTP tunnel with it - a small proxy between you and destination. We even give you a base class: `HttpTunnel` class, that provides easy HTTP tunneling. It opens the server socket on one port and tunnels the whole HTTP traffic to some target address.

[TinyTunnel](https://github.com/igr/tiny-tunnel) is one implementation that simply prints out the whole communication to the console.

