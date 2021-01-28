# Schemas và Types

Trên trang này, chúng ta sẽ tìm hiểu tất cả những gì cần biết về GraphQL type system, cách nó mô tả dữ liệu nào có thể được query. Vì GraphQL có thể được sử dụng với bất kỳ backend framework hay programming language nào, chúng ta sẽ tránh xa việc implementation cụ thể chi tiết và chỉ nói về các khái niệm (concepts).

## Type system

Nếu bạn đã xem query GraphQL trước đây, bạn biết rằng GraphQL query languge về cơ bản là về việc chọn các field trên các object. Vì vậy, ví dụ: trong query sau:

- Query

```
{
  hero {
    name
    appearsIn
  }
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```

1. Chúng ta bắt đầu với 1 "root" object đặc biệt.
2. Chúng tôi chọn field `hero` trên đó.
3. Đối với object được trả về bởi `hero`, chúng ta chọn 2 field `name` và `appearsIn`.

Vì hình dạng của query GraphQL khớp chặt chẽ với kết quả, bạn có thể dự đoán query sẽ trả về mà không cần biết nhiều về serrver. Nhưng sẽ hữu ích nếu có mô tả chính xác về dữ liệu mà chúng ta có thể yêu cầu - chúng ta có thể chọn field nào ? Chúng có thể trả về những kiểu object nào? Những field nào có sẵn trên những object con đó? Đó là nơi gọi là `schema`.

Mọi GraphQL service đều xác định một tập hợp các type mô tả tập dữ liệu có thể có mà bạn có thể query trên service đó. Sau đó, khi các query đến, chúng được xác thực và thực thi dựa trên schema đó.

## Type language

GraphQL service có thể được viết bằng bất kỳ ngôn ngữ nào. Vì chúng ta không thể dựa vào cú pháp ngôn ngữ lập trình cụ thể, như JavaScript, để nói về GraphQL schema, chúng ta sẽ xác định ngôn ngữ đơn giản của riêng mình. Chúng ta sẽ sử dụng "GraphQL schema language" - nó tương tự như ngôn ngữ query và cho phép chúng ta nói về các GraphQL schema theo một ngôn ngữ riêng nhưng ko xác định (language-agnostic way).

## Object types and fields

Các thành phần cơ bản nhất của GraphQL schema là các object types, chỉ đại diện cho một loại object mà bạn có thể fetch từ service của mình và nó có những field nào. Trong GraphQL schema languge, chúng ta có thể biểu diễn nó như sau:

```
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

Ngôn ngữ này khá dễ đọc, nhưng chúng ta hãy xem xét nó để chúng ta có thể có vốn từ vựng được chia sẻ:

- `Character` là 1 `GraphQL Object Type`, có nghĩa là kiểu có một số fields. Hầu hết các kiểu trong schema của bạn sẽ là Object Type.
- `name` và `appearsIn` là các field thuộc kiểu `Character`.  Điều đó có nghĩa là `name` và `appearsIn` trong là các field duy nhất có thể xuất hiện trong bất kỳ phần nào của GraphQL query hoạt động trên kiểu `Character`.
- `String` là một trong những `scalar` type được tích hợp sẵn - đây là những kiểu giải quyết cho một scalar object riêng lẻ, và không thể có các sub-selections trong query. Chúng ta sẽ xem xét thêm về các scalar types sau.
- `String!` là `non-null` field, nghĩa là GraphQL service sẽ luôn cung cấp cho bạn một giá trị khi bạn query field này. Trong type language, chúng ta sẽ thể hiện cho những field đó bằng dấu chấm than (`!`).
- `[Episode!]!` biểu thị cho một mảng các `Episode` object. Vì nó cũng `non-nullable`, bạn luôn có thể mong đợi một mảng (với 0 hoặc nhiều itens) khi bạn query field `appearsIn`. Và vì `Episode!` cũng thể hiện `non-nullable` nên bạn luôn có thể mong đợi mọi items của mảng là một `Episode`.

Bây giờ bạn đã biết GraphQL Object Type trông như thế nào và cách đọc những kiến ​​thức cơ bản về GraphQL type language.

## Arguments

Mọi field  trên GraphQL Object Type có thể không có hoặc có nhiều argument, ví dụ như field `length` bên dưới:

```
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

Tất cả các argument đều được đặt tên. Không giống như các ngôn ngữ như JavaScript hay Python, nơi các function lấy một list các argument có thứ tự, tất cả các argument trong GraphQL đều được truyền theo tên cụ thể. Trong trường hợp này, field `length` có một argument được xác định, `unit`.

Argument có thể là optional hoặc required. Khi argument là optional, chúng ta có thể set 1 giá trị mặc định - nếu argument `unit` không được truyền vào, nó sẽ được set thành `METER` theo mặc định.

## Query and Mutation types

Hầu hết các type trong schema của bạn sẽ chỉ là các object type bình thường, nhưng có hai type đặc biệt trong một schema:

```
schema {
  query: Query
  mutation: Mutation
}
```

Mỗi GraphQL service đều có một `query` type và có thể có hoặc không có `mutation` type. Những kiểu này giống như object type thông thường, nhưng chúng đặc biệt vì chúng define `entry point` của mọi GraphQL query. Vì vậy, nếu bạn thấy một query giống như:

- Query 

```
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

- Result 

```
{
  "data": {
    "hero": {
      "name": "R2-D2"
    },
    "droid": {
      "name": "C-3PO"
    }
  }
}
```

Có nghĩa là GraphQL service cần có `query` type với các field `hero` và `droid`:

```
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

Mutations hoạt động theo cách tương tự - bạn xác định các fields trên `Mutation` tupe và những fields đó có sẵn dưới dạng `root mutation fields` mà bạn có thể gọi trong query.

Điều quan trọng cần nhớ là ngoài trạng thái đặc biệt là "entry point" trong schema, các `Query` và `Mutation` type giống với bất kỳ GraphQL Object type nào khác và các fields của chúng hoạt động theo cùng một cách.

## Scalar types

GraphQL object type có `name` và các `field`, nhưng tại một số thời điểm, các field đó phải giải quyết một số dữ liệu cụ thể. Đó là nơi các scalar type xuất hiện: chúng đại diện cho các phần của query.

Trong query sau, các field `name` và `appearsIn` sẽ được xử lý thành các `scalar types`:

- Query 

```
{
  hero {
    name
    appearsIn
  }
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```

Chúng ta biết điều này vì những field đó không có bất kỳ sub-field nào - chúng là lá (leaves) của query.

GraphQL đi kèm với một tập hợp các scalar type mặc định như dưới đây:

- `Int`: Số nguyên có dấu 32‐bit.
- `Float`: A signed double-precision floating-point value.
- `String`: A UTF‐8 character sequence.
- `Boolean`: `true` or `false`.
- `ID`: ID scalar type biểu thị cho một số unique identifier, thường được sử dụng để refetch một object hoặc làm key cho cache. ID type được serialize (tuần tự hóa) theo cách giống như `String`. Tuy nhiên, việc xác định nó là một ID có nghĩa là nó không nhằm mục đích cho con người có thể đọc.

Trong việc implement GraphQL service, cũng có một cách để chỉ định scalar type tùy chỉnh. Ví dụ: chúng ta có thể xác định kiểu `Date`:

```
scalar Date
```

Sau đó, việc implemetation của chúng ta tùy thuộc vào việc xác định cách type đó sẽ được serialized, deserialized và validated. Ví dụ: bạn có thể chỉ định rằng kiểu `Date` phải luôn được serialized thành integer timestamp và khách hàng của bạn nên biết format đó cho tất cả các date field nào.

## Enumeration types

Còn được gọi là Enums, enumaeration type là một scalar đặc biệt được giới hạn trong một tập hợp các giá trị cụ thể. Điều này cho phép chúng ta:

1. Validate rằng bất kỳ argument nào thuộc kiểu này là một trong những giá trị được phép.
2. Giao tiếp thông qua type system rằng một field sẽ luôn là một trong một tập giá trị hữu hạn.

Dưới đây là cách định nghĩa enum trong GraphQL schema language:

```
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

Code trên nghĩa là bất cứ chỗ nào chúng ta sử dụng kiểu `Episode` trong schema của mình, chúng ta mong đợi nó chính xác là một trong `NEWHOPE`, `EMPIRE` hoặc `JEDI`.

Lưu ý rằng việc implementation GraphQL service ở các ngôn ngữ khác nhau sẽ có cách dành riêng cho từng ngôn ngữ cụ thể để xử lý với enums. Trong các ngôn ngữ hỗ trợ enums, chúng ta có thể tận dụng điều đó. Còn đối với ngôn ngữ như JavaScript không có hỗ trợ enum, các giá trị này có thể được ánh xạ  thành một tập hợp các số nguyên. Tuy nhiên, đừng rò rĩ những cho client, chỗ mà có thể hoạt động hoàn toàn theo string name của enum values.

## Lists và Non-Null

Object types, scalars và enums là những kiểu duy nhất bạn có thể define trong GraphQL. Nhưng khi bạn sử dụng các kiểu trong các phần khác của schema hoặc trong khai báo query variable, bạn có thể áp dụng `type modifiers` để ảnh hưởng đến việc validate giá trị. Hãy xem một ví dụ:

```
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

Ở đây, chúng ta đang sử dụng kiểu `String` và đánh dấu nó là `Non-Null` bằng cách thêm dấu chấm than (`!`) sau tên kiểu. Điều này có nghĩa là server luôn mong đợi trả về giá trị khác `null` cho field này và nếu nó kết thúc nhận một giá trị null sẽ thực sự gây ra GraphQL execution error, cho client biết rằng đã xảy ra lỗi.

`Non-Null` type modifier cũng có thể được sử dụng khi define arrgument cho một field, điều này sẽ khiến GraphQL server trả về lỗi validation nếu giá trị `null` được truyền cho argument đó, dù trong GraphQL string hay trong các variable.

- Query

```
query DroidById($id: ID!) {
  droid(id: $id) {
    name
  }
}
```

- Variable 

```
{
  "id": null
}
```

- Result

```
{
  "errors": [
    {
      "message": "Variable \"$id\" of non-null type \"ID!\" must not be null.",
      "locations": [
        {
          "line": 1,
          "column": 17
        }
      ]
    }
  ]
}
```

Danh sách hoạt động theo cách tương tự: Chúng ta có thể sử dụng type modifier để đánh dấu một kiểu là `List`, cho biết rằng field này sẽ trả về một mảng thuộc kiểu đó. Trong schema language, điều này được biểu thị bằng cách đặt kiểu trong dấu ngoặc vuông, `[` và `]`. Nó hoạt động tương tự đối với arguments, trong đó bước validation sẽ mong đợi một mảng cho giá trị đó.

Có thể kết hợp sử dụng `Non-Null` và `List` type modifier. Ví dụ, bạn có thể có một List các `Non-Null` strings như dưới đây:

```
myField: [String!]
```

Điều này có nghĩa là bản thân list có thể là `null`, nhưng nó không thể chứa bất kỳ thành viên nào có giá trị `null`. Ví dụ, trong JSON:

```
myField: null // valid
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // error
```

Bây giờ, chúng ta thử define một `non-null` list các string:

```
myField: [String]!
```

Điều này có nghĩa là bản thân list không thể null, nhưng nó có thể chứa các giá trị null:

```
myField: null // error
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // valid
```

Bạn có thể tùy ý lồng Non-Null và List type modifier với số lượng bất kì tùy theo nhu cầu của bạn.

## Interfaces

Giống như nhiều type system khác, GraphQL hỗ trợ `interface`. `Interface` là một kiểu trừu tượng bao gồm một tập hợp các field nhất định mà một kiểu phải bao gồm khi implement interface đó.

Ví dụ: bạn có thể có một interface `Character` biểu thị cho bất kỳ nhân vật nào trong bộ ba Chiến tranh giữa các vì sao:

```
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

Điều này có nghĩa là bất kỳ kiểu nào implement  inteface `Character` đều cần có các field chính xác này, với các argument và kiểu trả về này.

Ví dụ: đây là một số kiểu implement interface `Character`:

```
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}
 
type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

Bạn có thể thấy rằng cả hai kiểu này đều có tất cả các field từ interface `Character`, nhưng cũng thêm các field bổ sung, `totalCredits`, `starhips` và `primaryFunction`, dành riêng cho loại character cụ thể đó.

Các interface hữu ích khi bạn muốn trả về một object hoặc một tập hợp các object, nhưng chúng có thể thuộc một số kiểu khác nhau.

Ví dụ: query sau đây tạo ra lỗi:

- Query

```
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}
```

- Variable

```
{
  "ep": "JEDI"
}
```

- Result

```
{
  "errors": [
    {
      "message": "Cannot query field \"primaryFunction\" on type \"Character\". Did you mean to use an inline fragment on \"Droid\"?",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

Field `hero` trả về kiểu `Character`, có nghĩa là nó có thể là `Human` hoặc `Droid` tùy thuộc vào `episode` argument. Trong query ở trên, bạn chỉ có thể yêu cầu các field tồn tại trên interface `Character`, không bao gồm `primaryFunction`.

Để yêu cầu một field  trên một object type cụ thể, bạn cần sử dụng `inline fragment`:

- Query 

```
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

- Variable 

```
{
  "ep": "JEDI"
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

Tìm hiểu thêm trong phần [inline fragment](https://graphql.org/learn/queries/#inline-fragments) trong query guide.

## Union types

Union type rất giống với interface, nhưng chúng không xác định được bất kỳ field chung nào giữa các kiểu.

```
union SearchResult = Human | Droid | Starship
```

Bất cứ chỗ nào trả về kiểu `SearchResult` trong schema, chúng ta có thể nhận được `Human`, `Droid` hoặc `Starship`. Lưu ý rằng các member của union type cần phải là object type cụ thể; bạn không thể tạo một union type từ các interface hoặc các union type khác.

Trong trường hợp này, nếu bạn query một field trả về union type `SearchResult`, bạn cần sử dụng một inline fragment để có thể query bất kỳ field nào:

- Query 

```
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

- Result 

```
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
```

Field `__typename` phân giải thành `String` cho phép bạn phân biệt các kiểu dữ liệu khác nhau với nhau trên client.

Ngoài ra, trong trường hợp này, `Human` và `Droid` chia sẻ một interface chung (`Character`), bạn có thể query các field chung của chúng ở một chỗ thay vì phải lặp lại các field giống nhau trên nhiều kiểu:

```
{
  search(text: "an") {
    __typename
    ... on Character {
      name
    }
    ... on Human {
      height
    }
    ... on Droid {
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

Lưu ý rằng `name` vẫn được chỉ định trên `Starship` vì nếu không nó sẽ không hiển thị trong kết quả khi `Starship` không phải là một `Character`!

## Input types

Cho đến nay, chúng ta chỉ nói về việc truyền các scalar values, như enums hoặc strings, dưới dạng argument vào một field. Nhưng bạn cũng có thể dễ dàng vượt qua các object phức tạp. Điều này đặc biệt có giá trị trong trường hợp mutation, khi bạn có thể muốn truyền toàn bộ object sẽ được tạo ra. Trong GraphQL schema language, input types trông giống hệt như object type thông thường, nhưng với từ khóa `input` thay vì `type`:

```
input ReviewInput {
  stars: Int!
  commentary: String
}
```

Đây là cách bạn có thể sử dụng input object type trong một mutation:


- Mutation

```
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

- Variable 

```
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

- Result 

```
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

Bản thân các fields trên một input object type có thể tham chiếu đến input object types khacs, nhưng bạn không thể kết hợp input types và output types trong schema. Các input object types cũng không được có argument trên các field của chúng.