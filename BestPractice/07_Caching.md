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

