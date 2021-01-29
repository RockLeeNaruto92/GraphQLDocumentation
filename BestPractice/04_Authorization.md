# Authorization

> Delegate authorization logic to the business logic layer
> Ủy quyền authorization logic tới business logic layer

Authorization là 1 kiểu business logic mô tả 1 user/session/context có quyền để thực thi 1 hành động hoặc xem 1 phần dữ liệu. Ví dụ:

“Only authors can see their drafts”

Việc thực thi loại hành vi này sẽ xảy ra trong business logic layer. Có thể đặt authorization logic trong GraphQL layer như sau:

```
var postType = new GraphQLObjectType({
  name: ‘Post’,
  fields: {
    body: {
      type: GraphQLString,
      resolve: (post, args, context, { rootValue }) => {
        // return the post body only if the user is the post's author
        if (context.user && (context.user.id === post.authorId)) {
          return post.body;
        }
        return null;
      }
    }
  }
});
```

Lưu ý rằng chúng ta define “author owns a post” bằng cách kiểm tra xem field `authorId` của `post` có bằng với `id` của user hiện tại hay không. Vấn đề là chúng ta cần copy code này cho mỗi endpoint vào service. Sau đó, nếu authorization logic là không được đồng bộ hóa hoàn hảo, user có thể thấy các dữ liệu khác nhau tùy thuộc vào API họ sử dụng. Rất tiếc! Chúng tôi có thể tránh điều đó bằng cách tạo ra [1 source riêng lẻ](https://graphql.org/learn/thinking-in-graphs/#business-logic-layer) để authorization.

Việc define authorization logic bên trong resolver là tốt khi học GraphQL hoặc tạo prototyping. Tuy nhiên đối với production codebase, hãy ủy quyền authorization logic cho business logic layer. Ví dụ:

```
//Authorization logic lives inside postRepository
var postRepository = require('postRepository');
 
var postType = new GraphQLObjectType({
  name: ‘Post’,
  fields: {
    body: {
      type: GraphQLString,
      resolve: (post, args, context, { rootValue }) => {
        return postRepository.getBody(context.user, post);
      }
    }
  }
});
```

Trong ví dụ trên, chúng ta thấy rằng business logic layer yêu cầu người gọi cung cấp một user object. Nếu bạn đang sử dụng GraphQL.js, `User` object phải được truyền vào argument `context` hoặc `rootValue` trong argument thứ tư của resolver.

Chúng tôi khuyên bạn nên chuyển `User` object được hydrat hóa hoàn toàn thay vì token hoặc API key. Bằng cách này, chúng ta có thể xử lý các mối quan tâm riêng biệt về [authentication](https://graphql.org/graphql-js/authentication-and-express-middleware/) và authorization trong các giai đoạn khác nhau của request processing pipeline.