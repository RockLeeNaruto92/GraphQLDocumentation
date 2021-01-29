# Serving over HTTP

HTTP là lựa chọn phổ biến nhất cho giao thức client-servcer khi sử dụng GraphQL vì tính phổ biến của nó. Dưới đây là một số hướng dẫn để thiết lập GraphQL server hoạt động qua HTTP.

## Web Request Pipeline

Hầu hết các web framework sử dụng pipeline model trong đó các request được truyền thông qua một đống middleware (AKA filters/plugins). Khi request chạy qua pipeline, nó có thể được kiểm tra (inspected), chuyển đổi (transformed), sửa đổi (modified) hoặc kết thúc (terminated) bằng một response. GraphQL nên được đặt sau tất cả các authentication midleware , để bạn có quyền truy cập vào cùng một session và cùng thông tin usertrong HTTP Endpoint handlers.

## URIs, Routes

HTTP thường được kết hợp với REST, sử dụng "resource" làm khái niệm cốt lõi. Ngược lại, concept của GraphQL là một `entity graph`. Do đó, các thực thể (entity) trong GraphQL không được xác định bằng URL. Thay vào đó, GraphQL server hoạt động trên một URL / endpoint, thường là `/graphql` và tất cả các GraphQL request cho một service nhất định phải được chuyển hướng đến endpoint này.

## HTTP Methods, Headers, and Body

GraphQL HTTP server nên handle cả `GET` và `POST`.

### GET request

Khi nhận được HTTP GET request, GraphQL query phải được chỉ định trong trường "query" trong query string. Ví dụ: nếu chúng ta muốn thực hiện GraphQL query sau:

```
{
  me {
    name
  }
}
```

Thì request có thể được gửi thông qua HTTP GET như dưới đây:

```
http://myapi/graphql?query={me{name}}
```


Query variable có thể được gửi dưới dạng một string được JSON hóa trong một query parameter được gọi là `variable`. Nếu query chứa một số operation được đặt tên, query parameter `operationName` có thể được sử dụng để kiểm soát cái nào nên được thực thi.

### POST request

GraphQL POST request nên sử dụng `application/json` content type, và chứa body đã được JSON hóa của form dưới đây:

```
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
```

`operationName` và `variables` là 2 optional. `operationName` chỉ required nếu có nhiều operation trong query.

Chúng tôi suggest nên support 2 trường hợp bổ sung dưới đây:
- Nếu `query` query string parameter có mặt thì sẽ được parse và xử lý theo cách tương tự trường hợp HTTP GET.
- Nếu `Content-Type` header là `application/graphql`, hãy coi HTTP POST body content là GraphQL query string.

Nếu bạn sử dụng express-graphql, bạn có thể sử dụng các case trên free.

## Response

Bất kể phương thức nào mà query và variable được gửi đi, response phải được trả về trong phần request body ở định dạng JSON. Như đã đề cập trong phần thông số kỹ thuật, một query có thể dẫn đến một số data và một số error và những error đó phải được trả về trong một JSON object của form:

```
{
  "data": { ... },
  "errors": [ ... ]
}
```

Nếu không có error nào được trả lại thì field `error` trong response có thể không xuất hiện. Nếu không có data được trả về, [dựa theo GraphQL spec](http://facebook.github.io/graphql/#sec-Data), nên thêm field `data` vào response nếu có lỗi xảy ra khi thực thi.

## GraphiQL

GraphiQL hữu ích trong quá trình test và develop nhưng sẽ bị disable trong production theo mặc định. Nếu bạn đang sử dụng express-graphql, bạn có thể bật/tắt dựa trên biến môi trường NODE_ENV:

```
app.use('/graphql', graphqlHTTP({
  schema: MySessionAwareGraphQLSchema,
  graphiql: process.env.NODE_ENV === 'development',
}));
```

## Node

Nếu bạn đang sử dụng NodeKS, chúng tổi khuyên bạn nên sử dụng [express-graphql](https://github.com/graphql/express-graphql) hoặc [apollo-server](https://github.com/apollographql/apollo-server).