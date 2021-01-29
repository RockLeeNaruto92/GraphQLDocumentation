# Pagination

> Different pagination models enable different client capabilities
> Các mô hình phân trang khác nhau cho phép các khả năng khác nhau của client

Một usecase phổ biến trong GraphQL là duyệt qua mối quan hệ giữa tập hợp các object. Có một số cách khác nhau để các mối quan hệ này có thể được biểu thị trong GraphQL, cung cấp một loạt các khả năng khác nhau cho client developer.

## Plurals

Cách đơn giản nhất để biểu thị kết nối giữa các object là sử dụng một field trả về kiểu số nhiều. Ví dụ: nếu chúng ta muốn có list bạn bè của R2-D2, chúng ta có thể yêu cầu:

```
{
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

## Slicing

Tuy nhiên, có những hành vi bổ sung khác mà client có thể muốn. Client có thể muốn chỉ định số lượng bạn bè mà họ muốn lấy; có lẽ họ chỉ muốn hai cái đầu tiên. Vì vậy, chúng tôi muốn viết một cái gì đó như:

```
{
  hero {
    name
    friends(first:2) {
      name
    }
  }
}
```

Nhưng nếu chúng ta chỉ fetch hai object đầu tiên, chúng ta lại muốn phân trang thông qua list; khi client fetch hai `friend` đầu tiên, họ có thể muốn request thứ hai để yêu cầu hai `friend` tiếp theo. Vậy làm thế nào để có thể xử lý trường hợp này?

## Pagination and Edges

Có 1 số cách mà chúng ta cỏ thể sử dụng để phân trang.
- `friends(first:2 offset:2)`
- `friends(first:2 after:$friendId)`
- `friends(first:2 after:$friendCursor)`

Nói chung, chúng tôi nhận thấy rằng tính năng **cursor-based pagination** là tính năng mạnh mẽ nhất trong số đó. Đặc biệt nếu con trỏ không rõ ràng, `offset-based` hoặc `ID-based` có thể được implement bằng cách sử dụng `cursor-based` pagination. Sử dụng con trỏ mang lại sự linh hoạt hơn nếu pagination model thay đổi trong tương lai. Xin lưu ý rằng các con trỏ là không rõ ràng và không nên dựa vào format của chúng, chúng tôi khuyên bạn nên encode chúng bằng base64.

Điều đó dẫn chúng ta đến một vấn đề: làm cách nào để lấy cursor từ object? Chúng tôi không muốn con trỏ hoạt động trên kiểu `User`. Nó là 1 thuộc tính của connection, không phải của object. Vì vậy, chúng tôi muốn giới thiệu một layer mới; field `friend` sẽ cung cấp cho chúng ta danh sách các `edge (cạnh)` và một edge có cả cursor và node ở dưới:

```
{
  hero {
    name
    friends(first:2) {
      edges {
        node {
          name
        }
        cursor
      }
    }
  }
}
```

Concept của edge (cạnh) tỏ ra hữu ích nếu có thông tin cụ thể về edge, thay vì một trong các object. Ví dụ: nếu chúng tôi muốn biểu thị "friendship time" trong API, thì nên ném thông tin này vào trong edge.

## End-of-list, counts, and Connections

Bây giờ chúng ta có khả năng phân trang thông qua kết nối bằng cách sử dụng con trỏ, nhưng làm cách nào để biết khi nào chúng ta đến cuối connection? Chúng ta phải tiếp tục query cho đến khi nhận lại được list rỗng, nhưng chúng ta thực sự muốn connection cho chúng ta biết khi nào chúng ta đã đến cuối, vì vậy không cần request bổ sung đó. Tương tự, điều gì xảy ra nếu chúng ta muốn biết thêm thông tin về chính connection; Ví dụ, R2-D2 có tổng số bạn bè là bao nhiêu?

Để giải quyết cả hai vấn đề này, field `friends` có thể trả về một connection object. Connection object sau đó sẽ có một field cho các edige, cũng như các thông tin khác (như tổng số và thông tin về việc trang tiếp theo có tồn tại hay không). Vì vậy, query cuối cùng của chúng ta có thể như sau:

```
{
  hero {
    name
    friends(first:2) {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```

Lưu ý rằng chúng ta cũng có thể nhét `endCursor` và `startCursor` trong `PageInfo` object. Bằng cách này, nếu chúng ta không cần bất kỳ thông tin bổ sung nào mà edge, chúng ta không cần query các edge, vì chúng ta có các cursor cần thiết để phân trang từ `pageInfo`. Điều này dẫn đến cải thiện khả năng sử dụng cho các connection; thay vì chỉ biểu thị list edges, chúng ta cũng có thể biểu thị một list riêng của các node, để tránh một layer vô hướng.

## Complete Connection Model

Thiết kế này phức tạp hơn thiết kế ban đầu của chúng ta (số nhiều)! Nhưng áp dụng thiết kế này, chúng tôi đã mở một số khả năng cho client:
- Khả năng phân trang thông qua list.
- Khả năng request thông tin về chính connection, như `totalCount` hoặc `pageInfo`.
- Khả năng request thông tin về chính edge, chẳng hạn như `cursor` hoặc `friendshipTime`.
- Khả năng thay đổi cách phân trang của backend, khi mà user chỉ sử dụng cursor không rõ ràng.

Để xem điều này trong thực tế, có một field bổ sung trong schema ví dụ, được gọi là `friendsConnection`, biểu thị tất cả các khái niệm này. Bạn có thể kiểm tra trong query ví dụ. Hãy thử xóa tham số `after` thành `friendsConnection` để xem việc phân trang sẽ bị ảnh hưởng như thế nào. Ngoài ra, hãy thử thay thế field `edge` bằng field `friends` của helper trên connection, cho phép truy cập trực tiếp vào danh sách bạn bè mà không có edge layer bổ sung.

```
{
  hero {
    name
    friendsConnection(first:2 after:"Y3Vyc29yMQ==") {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
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
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            },
            "cursor": "Y3Vyc29yMg=="
          },
          {
            "node": {
              "name": "Leia Organa"
            },
            "cursor": "Y3Vyc29yMw=="
          }
        ],
        "pageInfo": {
          "endCursor": "Y3Vyc29yMw==",
          "hasNextPage": false
        }
      }
    }
  }
}
```

## Connection Specification

Để đảm bảo việc triển khai nhất quán của pattern này, dự án Relay có một [đặc tả](https://facebook.github.io/relay/graphql/connections.htm) chính thức mà bạn có thể tuân theo để xây dựng các GraphQL API sử dụng cursor based connection pattern.