# Using the Client

### Simple GET request

```java
HttpRequest httpRequest = HttpRequest.get("http://jodd.org");
HttpResponse response = httpRequest.send();

System.out.println(response);
```

All **HTTP** classes offer a _fluent interface_, too; so you can write:

```java
HttpResponse response = HttpRequest.get("http://jodd.org").send();

System.out.println(response);
```

You can build the request step by step:

```java
HttpRequest request = new HttpRequest();
request
    .method("GET")
    .protocol("http")
    .host("srv")
    .port(8080)
    .path("/api/jsonws/user/get-user-by-id");
```

### Reading Response

When an HTTP request is sent, the whole response is stored in the `HttpResponse` instance. You can use the response for various stuff: read the `statusCode()` or `statusPhrase()`; or any header attribute.

A common thing is how to read the received response body. You may use one of the following methods:

* `bodyRaw()` - raw body content, always in `ISO-8859-1` encoding.
* `bodyText()` - body text, i.e. string encoded as specified by `Content-Type` header.
* `bodyBytes()` - returns the raw body as a byte array, so e.g. downloaded file

  can be saved.

The character encoding used in `bodyText()` is one set in the response headers. If the response does not specify the encoding in its headers \(but e.g. only on the HTML page\), you _must_ specify the encoding with `charset()` the method before calling `bodyText()`. {: .attn}

### Query parameters

Query parameters may be specified in the URL line \(but then they have to be encoded correctly\):

```java
HttpResponse response = HttpRequest
    .get("http://srv:8080/api/user/get-user-by-id?userId=10194")
    .send();
```

A better and recommended way is with the `query()` method:

```java
HttpResponse response = HttpRequest
    .get("http://srv:8080/api/user/get-user-by-id")
    .query("userId", "10194")
    .send();
```

You can use `query()` for each parameter, or pass many arguments in one call \(varargs\). You can also provide `Map<String, String>` as a parameter too.

{% hint style="info" %}
Query parameters \(as well as headers and form parameters\) can be duplicated. Therefore, they are stored in an array internally. Use method `removeQuery` to remove some parameters or overloaded methods to replace a parameter.
{% endhint %}

Finally, you can reach the internal query map, that actually holds all parameters:

```java
Map<String, Object[]> httpParams = request.query();
httpParams.put("userId", new String[] {"10194"});
```

### Authentication

Basic authentication is made easy:

```java
request.basicAuthentication("user", "password");
```

Token-based authentication:

```java
request.tokenAuthentication("M4ORM....");
```

### POST and form parameters

```java
HttpResponse response = HttpRequest
    .post("http://srv:8080/api/jsonws/user/get-user-by-id")
    .form("userId", "10194")
    .send();
```

Use `form()` in the same way as `query()` to specify form parameters. Everything that is said for `query()` applies to the `form()`.

### Upload files

Again, it's easy: just add the file form parameter. Here is one real-world example:

```java
HttpRequest httpRequest = HttpRequest
    .post("http://srv:8080/api/dlapp/add-file-entry")
    .form(
        "repositoryId", "10178",
        "folderId", "11219",
        "sourceFileName", "a.zip",
        "mimeType", "application/zip",
        "title", "test",
        "description", "Upload test",
        "changeLog", "testing...",
        "file", new File("d:\\a.jpg.zip")
    );

HttpResponse httpResponse = httpRequest.send();
```

#### Monitor upload progress

When uploading a large file, it is helpful to monitor the progress. For that purpose, you can use `HttpProgressListener` like this:

```java
HttpResponse response = HttpRequest
    .post("http://localhost:8081/hello")
    .form("file", file)
    .monitor(new HttpProgressListener() {
        @Override
        public void transferred(long len) {
            System.out.println(len/size);
        }
    })
    .send();
```

Before the upload starts, `HttpProgressListener` calculates the `callbackSize` - the size of the chunk in bytes that will be transferred. By default, this size equals `1%` of the total size. Moreover, it is never less than `512` bytes.

`HttpProgressListener` contains the inner field `size` with the total size of the request. Note that this is the size of the whole request, not only the files! This is the actual number of bytes that are going to be sent, and it is always a bit larger than file size \(due to protocol overhead\).

### Headers

Add or reach header parameters with the method `header()`. Some common header parameters are already defined as methods, so you will find `contentType()` etc.

There are some shortcut methods that are commonly used:

* `contentTypeJson()` - specifies JSON content type
* `acceptJson()` - accepts JSON content.

### GZipped content

Just `unzip()` the response.

```java
HttpResponse response = HttpRequest
    .get("http://jodd.org")
    .acceptEncoding("gzip")
    .send();

System.out.println(response.unzip());
```

The `unzip()` method is safe; it will not fail if the response is not zipped.

### Set the body

You can set the request body manually:

```java
HttpResponse response = HttpRequest
    .get("http://srv:8080/api/jsonws/invoke")
    .body("{'a':1 23, 'b': 'hi'}")
    .basicAuthentication("test", "test")
    .send();
```

{% hint style="warning" %}
Setting the body discards all previously set `form()` parameters. 
{% endhint %}

### Charsets and Encodings

By default, query and form parameters are encoded in UTF-8.

```java
    HttpResponse response = HttpRequest
        .get("http://server/index.html")
        .queryEncoding("CP1251")
        .query("param", "value")
        .send();
```

You can set form encoding similarly. Moreover, form posting detects the value of **charset** in the "Content-Type" header, and if present, it will be used.

With received content, `body()` method always returns the **raw** string \(encoded as ISO-8859-1\). To get the string in usable form, use the method `bodyText()`. This method uses a provided **charset** from the "Content-Type" header and encodes the body string.

### Following redirection

By default `HttpRequest` does not follow redirection response. This can be changed by setting the `followRedirects(true)`. Now redirect responses are followed. When redirection is enabled, the original URL will NOT be preserved in the request object!

### Asynchronous sending

When `send()` is called, the program blocks until the response is received. By using `sendAsync()` the execution of the sending is passed to Javas fork-join pool, and will be executed asynchronously. Method returns `CompletableFuture<Response>`.

