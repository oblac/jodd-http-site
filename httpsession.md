# HttpSession

Sending requests and receiving responses is not enough when we have to emulate browsing through a website. For example, we might need to login, capture and carry-on the cookies to preserve the session, follow the redirects, make requests to get dynamic content... in the same way as a real web browser.

`HttpSession` is a tool just for that. It sends requests for you; handles 301 and 302 redirections automatically, reads and preserve cookies across the requests, and so on.

Usage is simple:

```java
HttpSession session = new HttpSession();

HttpRequest request = HttpRequest.get("www.facebook.com");
session.sendRequest(request);

// request is sent and response is received

// process the HTML page
String page = session.getPage();

// create new request
HttpRequest newRequest = HttpRequest.post(formAction);

session.sendRequest(newRequest);
```

`HttpSession` instance handles all the cookies, allowing the session to be tracked while browsing using HTTP and supports keep-alive persistent connections.

