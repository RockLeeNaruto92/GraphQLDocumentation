# Execution

Sau khi được validate, GraphQL query được thực thi bởi GraphQL server trả về kết quả phản ánh hình dạng của query được yêu cầu, thường là JSON.

GraphQL không thể thực hiện query mà không có type system, hãy sử dụng type system ví dụ để minh họa việc thực thi query. Đây là một phần của type system được sử dụng trong các ví dụ trong các bài viết này:

```
type Query {
  human(id: ID!): Human
}
 
type Human {
  name: String
  appearsIn: [Episode]
  starships: [Starship]
}
 
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
 
type Starship {
  name: String
}
```

Để mô tả điều gì sẽ xảy ra khi một query được thực thi, hãy xem ví dụ dưới đây.

```
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

- Result 

```
{
  "data": {
    "human": {
      "name": "Han Solo",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "starships": [
        {
          "name": "Millenium Falcon"
        },
        {
          "name": "Imperial shuttle"
        }
      ]
    }
  }
}
```

Bạn có thể coi mỗi field trong GraphQL query là một function hoặc method của kiểu trước trả về kiểu tiếp theo. Trên thực tế, đây chính xác là cách hoạt động của GraphQL. Mỗi field trên mỗi kiểu được hỗ trợ bởi một function được gọi là `resolver` được cung cấp bởi GraphQL server developer. Khi một field được thực thi, `resolver` tương ứng được gọi để tạo ra giá trị tiếp theo.

Nếu một field tạo ra scalar value như string hoặc number, thì quá trình thực thi sẽ hoàn tất. Tuy nhiên, nếu một field tạo ra một object value thì query sẽ chứa các field khác áp dụng cho object đó. Điều này tiếp tục cho đến khi đạt được các scalar value. Các GraphQL query luôn kết thúc ở các scalar value.

## Root fields & resolvers

Ở top level của mọi GraphQL server là một kiểu thể hiẹn cho tất cả các API GraphQL entry points, nó thường được gọi là kiểu `Root` hoặc kiểu `Query`.

Trong ví dụ này, kiểu `Query` cung cấp một field có tên là `human` chấp nhận argument `id`. `resolver` cho field này có thể truy cập database, sau đó xây dựng và trả về `Human` object.

```
Query: {
  human(obj, args, context, info) {
    return context.db.loadHumanByID(args.id).then(
      userData => new Human(userData)
    )
  }
}
```

Ví dụ này được viết bằng JavaScript, tuy nhiên các GraphQL server có thể được xây dựng bằng nhiều ngôn ngữ khác nhau. Một `resolver` function nhận bốn argument:

- `obj`: Object trước đó, object dành cho một field trong kiểu root Query, thường không được sử dụng.
- `args`: Các argument được cung cấp cho field trong GraphQL query.
- `context`: Một giá trị được cung cấp cho mọi `resolver` , giữ thông tin ngữ cảnh quan trọng (important contextual) như người dùng hiện đang đăng nhập hoặc quyền truy cập vào cơ sở dữ liệu.
- `info`: Một giá trị chứa thông tin về field cụ thể có liên quan đến query hiện tại cũng như các chi tiết về schema. Tham khảo [type GraphQLResolveInfo](https://graphql.org/graphql-js/type/#graphqlobjecttype) để biết thêm chi tiết.

## Asynchronous resolvers

Chúng ta hãy xem xét kỹ hơn những gì đang xảy ra trong `resolver` function này.

```
human(obj, args, context, info) {
  return context.db.loadHumanByID(args.id).then(
    userData => new Human(userData)
  )
}
```

`context` được sử dụng để cung cấp quyền truy cập vào cơ sở dữ liệu được sử dụng để load dữ liệu cho người dùng bằng `id` được cung cấp làm argument trong GraphQL query. Vì load từ cơ sở dữ liệu là một hoạt động không đồng bộ nên nó sẽ trả về 1 `Promise`. Trong JavaScript, Promise được sử dụng để làm việc với các giá trị không đồng bộ, nhưng khái niệm tương tự tồn tại trong nhiều ngôn ngữ, thường được gọi là *Futures*, *Tasks* hoặc *Deferred*. Khi cơ sở dữ liệu trả về, chúng ta có thể xây dựng và trả về một `Human` object mới.

Lưu ý rằng trong khi resolver function cần phải biết Promises, GraphQL query thì không. Nó chỉ đơn giản mong đợi field `human` trả lại một cái gì đó mà sau đó nó có thể hỏi `name` của nó. Trong quá trình thực thi, GraphQL sẽ đợi Promises, Futures và Tasks hoàn thành trước khi tiếp tục và sẽ làm như vậy với optimal concurrency.

## Trivial resolvers

Bây giờ đã có sẵn 1 `Human` object, việc thực thi GraphQL có thể tiếp tục với các field được yêu cầu trên đó.

```
Human: {
  name(obj, args, context, info) {
    return obj.name
  }
}
```

GraphQL server được cung cấp bởi một type system, được sử dụng để xác định việc cần làm tiếp theo. Ngay cả trước khi field `human` trả về bất kỳ thứ gì, GraphQL biết rằng bước tiếp theo sẽ là giải quyết các field trên kiểu `Human` vì type system cho biết rằng field `human` sẽ trả về một `Human`.

Giải quyết tên trong trường hợp này là rất đơn giản. `name resolver function` được gọi và argument `obj` là `Human` object mới được trả về từ field trước đó. Trong trường hợp này, chúng ta mong đợi `Human` object đó có thuộc tính `name` mà chúng ta có thể đọc và trả về trực tiếp.

Trên thực tế, nhiều thư viện GraphQL sẽ cho phép bạn bỏ qua các `resolver` đơn giản này và sẽ chỉ giả sử rằng nếu một `resolver` không được cung cấp cho một field, thì một thuộc tính cùng tên sẽ được đọc và trả về.

## Scalar coercion

Trong khi field `name` đang được xử lý, field `appearsIn` và field `starships` có thể được giải quyết đồng thời. Field `appearsIn` cũng có thể có một `resolver` nhỏ, nhưng chúng ta hãy xem xét kỹ hơn:

```
Human: {
  appearsIn(obj) {
    return obj.appearsIn // returns [ 4, 5, 6 ]
  }
}
```

Lưu ý rằng type system của chúng ta yêu cầu `appearsIn` sẽ trả về giá trị Enum với các giá trị đã biết, tuy nhiên function trả về number! Thật vậy, nếu chúng ta nhìn vào kết quả, chúng ta sẽ thấy rằng các giá trị Enum thích hợp đang được trả về. Chuyện gì vậy?

Đây là một ví dụ về scalar coercion. Type system biết điều gì sẽ xảy ra và sẽ chuyển đổi các giá trị được trả về bởi một `resolver function` thành một thứ duy trì API contract. Trong trường hợp này, có thể có một Enum được xác định trên server sử dụng các số như `4`, `5` và `6` trong nội bộ, nhưng biểu thị chúng dưới dạng giá trị Enum trong GraphQL type system.

## List resolvers

Chúng ta đã thấy một chút về những gì sẽ xảy ra khi một field trả về danh sách những thứ với trường `appearsIn` ở trên. Nó trả về một `list` các giá trị enum, và vì đó là những gì mà type system mong đợi, mỗi item trong list bị ép buộc về giá trị enum thích hợp. Điều gì xảy ra khi field `starships ` được xử lý?

```
Human: {
  starships(obj, args, context, info) {
    return obj.starshipIDs.map(
      id => context.db.loadStarshipByID(id).then(
        shipData => new Starship(shipData)
      )
    )
  }
}
```

Resolver cho field này không chỉ trả lại Promise mà còn trả về một `list` các promises. `Human` object có một list các `id` của `Starship` mà họ đã lái thử, nhưng chúng ta cần load tất cả các `id` đó để có được các `StarShip` object thực sự.

GraphQL sẽ đợi đồng thời tất cả các Promise này trước khi tiếp tục và khi còn lại list các object, nó sẽ đồng thời tiếp tục một lần nữa để load field `name` trên mỗi item này.

## Producing the result

Khi mỗi field được xử lý, giá trị kết quả được đặt vào một key-value map với field name (hoặc alias) làm key và value được xử lý là value. Điều này tiếp tục từ các field lá dưới cùng của query trở lại field ban đầu trên root Query type. Nói chung, những thứ này tạo ra một cấu trúc phản chiếu query ban đầu mà sau đó có thể được gửi (thường là JSON) đến client đã request.

Hãy xem xét lần cuối query ban đầu để xem tất cả các resolver function này tạo ra kết quả như thế nào:

```
{
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

- Result 

```
{
  "data": {
    "human": {
      "name": "Han Solo",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "starships": [
        {
          "name": "Millenium Falcon"
        },
        {
          "name": "Imperial shuttle"
        }
      ]
    }
  }
}
```