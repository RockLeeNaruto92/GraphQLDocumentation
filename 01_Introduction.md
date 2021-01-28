# Giới thiệu về GraphQL

> Tại series này, chúng ta sẽ học về GraphQL, cách GraphQL hoạt động và cách sử dụng.
> Hiện tại có rất nhiều library giúp implement 1 GraphQL service support [nhiều ngôn ngữ khác nhau](https://graphql.org/code/). 
> Bạn có thể học sâu hơn về GraphQL [tại đây](https://www.howtographql.com/).
> Chúng tôi cũng liên kết với edX để tạo 1 course online miễn phí [Exploring GraphQL: A Query Language for APIs](https://www.edx.org/course/exploring-graphql-a-query-language-for-apis).

GraphQL là 1 ngôn ngữ query cho API, cũng là 1 server-side runtime cho việc thực thi các queries bằng sử dụng 1 hệ thống type (`a type system`) mà bạn định nghĩa cho data của bạn. GraphQL không phụ thuộc vào bất kì 1 database hay 1 storage engine nào, và thay vào đó được hỗ trợ thay thế bởi code và data của bạn.

1 GraphQL service được tạo bằng cách xác định các type (kiểu) và field (trường) trên các type đó, sau đó cung cấp các function(hàm) cho từng trường trên mỗi kiểu.
Ví dụ: 1 service GraphQL cho chúng ta biết ai đã đăng nhập trong user là (`me`) trông giống như sau:

```
type Query {
  me: User
}
 
type User {
  id: ID
  name: String
}
```

Các chức năng cho từng field trên từng type:

```
function Query_me(request) {
  return request.auth.user;
}
 
function User_name(user) {
  return user.getName();
}
```

Khi service GraphQL đang chạy (thường là tại URL trên web service), nó có thể nhận các GraphQL queries để validate (xác thực) và execute (thực thi).
Trước tiên, một query nhận được sẽ được kiểm tra để đảm bảo nó chỉ tham chiếu đến các type và field được xác định, sau đó chạy các function được cung cấp để tạo ra kết quả.

Ví dụ:

```
{
  me {
    name
  }
}
```

Query trên có thể tạo ra kết quả JSON như sau:

```
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```