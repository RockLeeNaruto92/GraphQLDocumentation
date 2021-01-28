# Queries và Mutations

Trong bài này, chúng ta sẽ học chi tiết về cách truy vấn 1 GraphQL server.

## Fields (trường)


Đơn giản nhất, GraphQL là yêu cầu các field cụ thể trên các object. Hãy bắt đầu bằng cách xem xét một truy vấn rất đơn giản và kết quả chúng ta nhận được khi chạy nó:

- Query
```
{
  hero {
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
    }
  }
}
```

Bạn có thể thấy ngay rằng truy vấn có hình dạng giống hệt như kết quả. Đây là điều cần thiết đối với GraphQL, vì bạn luôn nhận lại được những gì bạn mong đợi và server biết chính xác những trường mà client đang yêu cầu.
Field `name` trả về kiểu `String`, trong trường hợp này là tên của 1 anh hùng của Chiến tranh giữa các vì sao, `R2-D2`.

> Ngoài ra ruy vấn trên có tính tương tác. Điều đó có nghĩa là bạn có thể thay đổi nó theo ý muốn và xem kết quả mới. Thử thêm fields `appearsIn` vào object `hero` trong truy vấn và xem kết quả mới.

Trong ví dụ trước, chúng ta chỉ yêu cầu truy vấn `name` của `hero` trả về một `String`, nhưng các fields cũng có thể tham chiếu đến `Object`. Trong trường hợp đó, bạn có thể thực hiện một lựa chọn phụ (`sub-selection`) các fields cho object đó. Các truy vấn GraphQL có thể duyệt các object có liên quan và các field của chúng, cho phép khách hàng tìm nạp nhiều dữ liệu liên quan trong một yêu cầu, thay vì thực hiện một số bước đi vòng như một kiến trúc REST cổ điển.

- Query 

```
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Lưu ý rằng trong ví dụ này, trường bạn bè trả về một mảng các mục. Các truy vấn GraphQL trông giống nhau đối với cả các mục đơn lẻ hoặc danh sách các mục, tuy nhiên chúng tôi biết mong đợi cái nào dựa trên những gì được chỉ ra trong lược đồ.

## Arguments

Nếu điều duy nhất chúng ta có thể làm là duyệt các object và các fields của chúng, thì GraphQL đã là một ngôn ngữ rất hữu ích để tìm nạp dữ liệu (data fetching). Nhưng khi bạn thêm khả năng truyền đối số (argument) vào các field, mọi thứ sẽ thú vị hơn nhiều.

- Query

```
{
  human(id: "1000") {
    name
    height
  }
}
```

- Result

```
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

Trong một system như REST, bạn chỉ có thể chuyển một tập hợp các argument - query parameter và URL segments trong request. Nhưng trong GraphQL, mọi field và object lồng nhau (nested object) đều có thể nhận được tập hợp các argument của riêng nó, làm cho GraphQL thay thế hoàn toàn cho tạo nhiều API để fetch data. Bạn thậm chí có thể chuyển các argument vào các field vô hướng (scalar field), để thực thi các phép biến đổi dữ liệu một lần trên servcer, thay vì trên từng client riêng lẻ.

- Query

```
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

- Result 

```
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

Argument có thể có nhiều kiểu khác nhau. Trong ví dụ trên, chúng ta đã sử dụng kiểu Enumeration, biểu thị một trong một tập hợp hữu hạn các options (trong trường hợp này là đơn vị độ dài, `METER` hoặc `FOOT`). GraphQL đi kèm với một tập hợp mặc định nhưng GraphQL server cũng có thể khai báo các kiểu tùy chỉnh của riêng nó, miễn là chúng có thể được tuần tự hóa  (serialized) thành format truyền tải của bạn.

[Tìm hiểu thêm về GraphQL type system](https://graphql.org/learn/schema)

## Aliases

Nếu là người tinh mắt, bạn có thể nhận thấy rằng, vì các kết quả object fields khớp với tên của field trong truy vấn nhưng không bao gồm các argument, bạn không thể truy vấn trực tiếp cho cùng một field với các argument khác nhau. Đó là lý do tại sao bạn cần `aliases` - chúng cho phép bạn đổi tên kết quả của một field thành bất kỳ thứ gì bạn muốn.

- Query 

```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

- Result

```
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

Trong ví dụ trên, hai field `hero` sẽ mâu thuẫn với nhau, nhưng vì chúng ta có thể đặt `alias` chúng thành các tên khác nhau, chúng ta có thể nhận được cả hai kết quả trong một yêu cầu.

## Fragments

Giả sử chúng ta có một trang tương đối phức tạp trong ứng dụng của mình, cho phép chúng ta xem hai `hero` cạnh nhau, cùng với `friends` của họ. Bạn có thể tưởng tượng rằng một truy vấn như vậy có thể nhanh chóng trở nên phức tạp, bởi vì chúng ta sẽ cần phải lặp lại các fields ít nhất một lần - một lần cho mỗi bên của phép so sánh.

Đó là lý do tại sao GraphQL chứa các `reusable units` (đơn vị có thể sử dụng lại) được gọi là `fragments`. `fragments` cho phép bạn tạo các tập hợp fields, sau đó đưa chúng vào các truy vấn khi bạn cần. Dưới đây là ví dụ về cách bạn có thể giải quyết tình huống trên bằng cách sử dụng `fragment`:

- Query

```
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

- Result

```
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Bạn có thể thấy truy vấn trên sẽ khá lặp lại như thế nào nếu các field được lặp lại. Khái niệm `fragment` thường được sử dụng để chia các yêu cầu dữ liệu  phức tạp thành các phần nhỏ hơn, đặc biệt khi bạn cần kết hợp nhiều thành phần giao diện người dùng với các fragment khác nhau thành một `initial data fetch`.

### Sử dụng variables trong fragments

Các fragment có thể truy cập các variable được khai báo trong query hoặc mutation. Xem [variables](https://graphql.org/learn/queries/#variables).

- Query

```
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

- Result

```
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          },
          {
            "node": {
              "name": "C-3PO"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "R2-D2",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Luke Skywalker"
            }
          },
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          }
        ]
      }
    }
  }
}
```

## Operation name

Cho đến nay, chúng tôi đang sử dụng cú pháp viết tắt trong đó chúng tôi bỏ qua cả keyword `query` và query name, nhưng trong production app, việc sử dụng chúng sẽ hữu ích để làm cho code của chúng ta ít mơ hồ hơn.

Dưới đây là một ví dụ bao gồm keyword `query` như là `operation type` và `HeroNameAndFriends` như là `operation name`:

- Query

```
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

`Operation type` là `query`, `mutation` hoặc `subscription` và mô tả loại hoạt động bạn định thực hiện. Operation type là bắt buộc trừ khi bạn đang sử dụng cú pháp viết tắt query, trong trường hợp đó bạn không thể cung cấp tên hoặc `variable` cho operation của bạn.

Operation name là tên có ý nghĩa và rõ ràng cho operation của bạn. Tên này chỉ được yêu cầu trong các `multi-operation documents`, nhưng việc sử dụng nó được khuyến khích vì nó rất hữu ích cho việc debug và log phía server. Khi có sự cố (bạn sẽ thấy lỗi trong network logs hoặc trong logs của GraphQL server), việc xác định truy vấn trong codebase của bạn bằng tên sẽ dễ dàng hơn thay vì cố gắng giải mã nội dung. Hãy nghĩ về điều này giống như `function name` trong ngôn ngữ lập trình yêu thích của bạn. Ví dụ: trong JavaScript, chúng ta chỉ có thể dễ dàng làm việc với các anonumous functions(hàm ẩn danh), nhưng khi chúng ta đặt tên cho một function, sẽ dễ dàng hơn để tracking, debug, log khi function được gọi. Theo cách tương tự, GraphQL query name và mutation name, cùng với fragment name, có thể là một debug tool hữu ích ở phía server để xác định các request GraphQL khác nhau.

## Variables

Cho đến nay, chúng tôi đã viết tất cả các argument của mình bên trong query string. Nhưng trong hầu hết các application, các argument cho các field sẽ là dynamic (động): Ví dụ: có thể có một menu thả xuống cho phép bạn chọn tập Star Wars mà bạn quan tâm, hoặc một search field hoặc một set các filters.

Sẽ không phải là ý kiến ​​hay nếu chuyển trực tiếp các dynamic argument này vào query string, vì khi đó code bên client sẽ cần thao tác động query string trong lúc chạy (runtime) và tuần tự hóa (serialize) thành một format dành riêng cho GraphQL. Thay vào đó, GraphQL có một cách hạng nhất để tính các dynamic value (giá trị động) ra khỏi query và chuyển chúng dưới dạng một từ điển riêng biệt. Các giá trị này được gọi là `variable`.

Khi bắt đầu làm việc với các `variable`, chúng ta cần thực hiện ba điều:

1. Thay thế giá trị tĩnh trong query bằng `$variableName`
2. Khai báo `$variableName` là một trong các `variable` được query chấp nhận
3. Thêm `variableName: value` trong từ điển `variable` riêng biệt, dành riêng cho `transport` (thường là JSON)

Khi tất cả cùng nhau sẽ có dạng như dưới đây:

- Query

```
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

- Variable

```
{
  "episode": "JEDI"
}
```

- Result: 

```
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Bây giờ, trong code bên client, chúng ta có thể chỉ cần đưa một variable khác chứ không cần phải tạo một query hoàn toàn mới. Nói chung, đây cũng là một phương pháp hay để biểu thị argument nào trong query  (được expect là dynamic) - chúng ta không bao giờ thực hiện string interpolation (nội suy chuỗi) để tạo query từ các giá trị do người dùng cung cấp.

### Variable definitions

Định nghĩa biến là phần trông giống như (`$episode: Episode`) trong query ở trên. Nó hoạt động giống như định nghĩa `argument` cho một function trong ngôn ngữ lập trình khác. Nó liệt kê tất cả các `variable`, có tiền tố là `$`, sau là kiểu của chúng , trong trường hợp này là `Episode`.

Tất cả variable phải là kiểu scalars, enums hoặc kiểu object đầu vào. Vì vậy, nếu bạn muốn chuyển một object phức tạp vào một field, bạn cần biết kiểu đầu vào nào phù hợp trên server. Tìm hiểu thêm về kiểu object đầu vào tại trang Schema.

Variable có thể là optional hoặc required. Trong trường hợp trên, vì không có `!` bên cạnh kiểu Episode nên nó là optional. Nhưng nếu field bạn đang truyền variable vào 1 yêu cầu 1 `non-null argument`, thì variable ấy sẽ được required.

Để tìm hiểu thêm về cú pháp cho các định nghĩa variable này, tôi khuyên nên tìm hiểu GraphQL schema language. Schema language được giải thích chi tiết trong trang Schema.

### Default variables

Giá trị mặc định cũng có thể được gán cho các variable trong query bằng cách thêm giá trị mặc định đó sau khai báo kiểu.

```
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

Khi các giá trị mặc định được cung cấp cho tất cả các biến, bạn có thể gọi query mà không cần truyền giá trị cho bất kỳ variable nào. Nếu bất kỳ variable nào được truyền vào như một phần của từ điển variable, chúng sẽ ghi đè các giá trị mặc định.

### Directives

Chúng ta đã thảo luận ở trên về cách các variable cho phép chúng ta tránh thực hiện nội suy chuỗi (string interpolation) thủ công để xây dựng các dynamic query. Việc truyền variable vào argument giải quyết một lớp khá lớn các vấn đề này, nhưng chúng ta cũng có thể cần một cách để thay đổi động cấu trúc và hình dạng (shape) các query của mình bằng cách sử dụng variable. Ví dụ, chúng ta có thể tưởng tượng một UI Component có chế độ xem tóm tắt và chi tiết, trong đó một thành phần bao gồm nhiều field hơn thành phần kia.

Hãy tạo một truy vấn cho một component như vậy:

- Query

```
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

- Variable

```
{
  "episode": "JEDI",
  "withFriends": false
}
```

- Result

```
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

Hãy thử chỉnh sửa các variable ở trên để thay vào đó truyền `true` cho `withFriends` và xem kết quả thay đổi như thế nào.

Chúng tôi cần sử dụng một tính năng mới trong GraphQL được gọi là `derective`. Một `directive` có thể được đính kèm (attach) với field hoặc bao gồm fragment inclusion và có thể ảnh hưởng đến việc thực thi query theo cách mà server mong muốn. GraphQL core specification bao gồm chính xác hai `directives`, phải được hỗ trợ bởi mọi GraphQL server tuân thủ thông số kỹ thuật:

□ `@include(if: Boolean)`: Chỉ bao gồm field này trong kết quả nếu argument là `true`
□ `@skip(if: Boolean)`: Bỏ qua field này nếu argument là `true`

Directive hữu ích để thoát khỏi các tình huống mà nếu không, bạn sẽ cần thực hiện các thao tác với string để thêm và xóa các field trong query của mình. Việc implementation cũng có thể thêm các tính năng thử nghiệm bằng cách xác định các directive hoàn toàn mới.

## Mutations

Hầu hết các cuộc thảo luận về GraphQL đều tập trung vào việc `data fetching`, nhưng bất kỳ data platform hoàn chỉnh nào cũng cần có cách sửa đổi dữ liệu phía server.

Trong REST, bất kỳ request nào cũng có thể gây ra một số tác dụng phụ trên server, nhưng theo quy ước, người ta không sử dụng các yêu cầu `GET` để sửa đổi data. GraphQL cũng tương tự - về mặt kỹ thuật, bất kỳ truy vấn nào cũng có thể được thực hiện để ghi data. Tuy nhiên, sẽ hữu ích khi thiết lập một quy ước rằng bất kỳ thao tác nào gây ra việc ghi phải được gửi một cách rõ ràng thông qua một `mutation`.

Cũng giống như trong các query, nếu mutation field trả về một loại object, bạn có thể yêu cầu các field lồng nhau (nested field). Điều này có thể hữu ích để fetch (tìm nạp) state mới của object sau khi update. Hãy xem một ví dụ mutation đơn giản:

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

Lưu ý cách field của `createReview` trả về 2 field `stars` và `commentary` của review mới được tạo. Điều này đặc biệt hữu ích khi thay đổi dữ liệu hiện có, chẳng hạn như khi tăng một field, chúng ta có thể thay đổi và query giá trị mới của field chỉ với một request.

Bạn cũng có thể nhận thấy rằng, trong ví dụ này, variable `review` mà chúng ta đã truyền vào không phải là `scalar`. Đó là một kiểu object đầu vào (input object type), một loại object đặc biệt có thể được truyền vào dưới dạng argument. Tìm hiểu thêm về các input types trên trang Schema.

### Multiple fields trong mutations

 Một mutation có thể chứa nhiều fields giống như một query. Có một điểm khác biệt quan trọng giữa query và mutation, ngoài name:

**Trong khi các query fields được thực thi song song, các mutation fields chạy theo chuỗi, nối tiếp nhau.**

**(While query fields are executed in parallel, mutation fields run in series, one after the other.)**

Điều này có nghĩa là nếu chúng ta gửi hai mutation `incrementCredits` trong một request, thì request đầu tiên kết thúc trước khi request thứ hai bắt đầu, đảm bảo rằng chúng ta không kết thúc với điều kiện chạy đua (race condition) với chính mình.

## Inline Fragments

Giống như nhiều type systems khác, GraphQL schema bao gồm khả năng xác định interface và union types (kiểu liên kết). Bạn có thể tìm hiểu thêm tại schema guide.

Nếu bạn đang query một field trả về interface hoặc union type, bạn sẽ cần sử dụng các `inline fragment` để access data về kiểu cụ thể bên dưới. Xem ví dụ sau:

- Query

```
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
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

Trong query này, field `hero` trả về kiểu `Character`, có thể là `Human` hoặc `Droid` tùy thuộc vào argument `episode`. Trong lựa chọn trực tiếp, bạn chỉ có thể yêu cầu các field tồn tại trên `Character` interface, chẳng hạn như `name`.

Để yêu cầu một field trên kiểu cụ thể, bạn cần sử dụng một `inline fragment` với điều kiện về type. Vì fragment đầu tiên được gắn nhãn là `... on Droid`, field `primaryFunction` sẽ chỉ được thực thi nếu `Character` được trả về nếu `hero` thuộc kiểu `Droid`. Tương tự đối với field `height` cho kiểu `Human`.

Các fragment cũng có thể được sử dụng theo cách tương tự, vì một đoạn được đặt tên luôn có một kiểu tương ứng đính kèm.

### Meta fields

Do có một số tình huống mà bạn không biết kiểu nào bạn sẽ nhận lại từ GraphQL service, bạn cần một số cách để xác định cách xử lý dữ liệu đó trên client. GraphQL cho phép bạn request `__typename`, một meta field, tại bất kỳ thời điểm nào trong một query để lấy tên của `object type` tại điểm đó.

```
{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
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
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}
```

Trong query trên, `search` trả về `union type`  có thể là một trong ba options. Sẽ không thể phân biệt được các kiểu khác nhau với client nếu không có field `__typename`.

GraphQL service cung cấp một vài meta fields, phần còn lại được sử dụng để trình bày hệ thống [Introspection](https://graphql.org/learn/introspection/).
