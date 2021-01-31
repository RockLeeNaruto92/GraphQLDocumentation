# Caching

> Providing Object Identifiers allows clients to build rich caches

Trong endpoint-based API, client có thể sử dụng HTTP caching để dễ dàng tránh tìm nạp lại resource và để xác định khi nào hai resource giống nhau . Do đó, đây là một phương pháp hay nhất để API biểu thị một identifier như vậy cho client sử dụng. Tuy nhiên, không có nguyên mẫu nào giống như URL cung cấp identifier duy nhất toàn cầu này cho một object nhất định.

## Globally Unique IDs

Một pattern cho việc này là dùng một field, như `id`, để làm identifer duy nhất. Schema ví dụ được sử dụng trong tài liệu này sử dụng phương pháp này:

```
{
  starship(id:"3003") {
    id
    name
  }
  droid(id:"2001") {
    id
    name
    friends {
      id
      name
    }
  }
}
```

- Result 

```
{
  "data": {
    "starship": {
      "id": "3003",
      "name": "Imperial shuttle"
    },
    "droid": {
      "id": "2001",
      "name": "R2-D2",
      "friends": [
        {
          "id": "1000",
          "name": "Luke Skywalker"
        },
        {
          "id": "1002",
          "name": "Han Solo"
        },
        {
          "id": "1003",
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

Đây là một công cụ mạnh mẽ để cho client developer. Giống như cách mà URL của một resource-based API cung cấp một key duy nhất, field `id` này cung cấp một key duy nhất trên global.

Nếu backend sử dụng một cái gì đó như UUID làm identifier, thì việc biểu thị ID rất đơn giản! Nếu backend không có ID cho object, GraphQL layer có thể phải làm điều này (cung cấp 1 identifier cho object). Đơn gian như thêm tên của kiểu vào ID và sử dụng nó làm identifer; sau đó server có thể làm cho ID đó không rõ ràng bằng cách mã hóa base64.

ID này sau đó có thể được sử dụng để làm việc với `node` pattern của [Global Object Identification](https://graphql.org/learn/global-object-identification).

## Alternatives

Mặc dù ID đã được chứng minh là một pattern mạnh mẽ trong quá khứ, nhưng chúng không phải là pattern duy nhất được sử dụng, cũng như không phù hợp cho mọi tình huống. Chức năng thực sự quan trọng mà client cần là khả năng tạo ra một unique identifier để caching. Thông thường, sẽ đơn giản như việc kết hợp kiểu của object (được truy vấn bằng `__typename`) với một số type-unique identifier (identifier kiểu duy nhất). Mặc dù việc để server lấy ra ID đó sẽ đơn giản hóa client, nhưng client cũng có thể lấy ra từ identifier.

Ngoài ra, nếu thay thế một API hiện có bằng GraphQL API, có thể gây nhầm lẫn nếu tất cả các field trong GraphQL đều giống nhau ngoại trừ `id`. Đây sẽ là một lý do khác khiến người ta có thể chọn không sử dụng `id` như globally unique field.