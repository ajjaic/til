# Example API

```haskell
type API = "deeply" :> "nested" :> "things" :> Get '[JSON] [Thing]
```
This means if you make a GET request to the endpoint at
`"/deeply/nested/things/"` you will get back a list of `[Thing]` in JSON format.

```haskell
type API = "events" :> Capture "id" Int :> Get [JSON] Event
```
Making a GET request to "/events/:id" will capture the id part, convert it to an
Int and send it as argument to the handler. In return you will get an `Event`
type in JSON format


```haskell
type API = Header "Cookie" String :> "auth" :> Get [HTML] String
```
The request to `/auth/` should be made with a GET request and the request must
have the `Cookie` parameter in the header of the reqest. In return a `String`
is returned in HTML format

```haskell
type API = "create" :> ReqBody '[JSON, XML] Person :> Post '[JSON] Person
```
The request to `/create/` must have in its body content a representation of a
`Person` object in either JSON or XML format. And it must be a POST request.
In return a `Person` object is returned in JSON format

```haskell
type API = "index" :> Get '[HTML] (Headers [Header "X-Whatever" Int] String)
```
The request to `/index/` is a GET request that will return a `String` as
response in HTML format. And this response will contain the `X-Whatever`
parameter in the Header.

