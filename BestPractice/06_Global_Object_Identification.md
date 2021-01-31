# Global Object Identification

> Consistent object access enables simple caching and object lookups
> Quyền truy cập đối tượng nhất quán cho phép cache và tra cứu objects đơn giản

Để cung cấp các tùy chọn cho GraphQL client để xử lý cache và refech data một cách tao nhã, GraphQL client cần biểu thị các nhận dạng object (object identifier) theo cách chuẩn hóa.

Để nó hoạt động, một client sẽ cần phải query thông qua một cơ chế tiêu chuẩn để request một object theo ID. Sau đó, trong response, schema sẽ cần cung cấp một cách tiêu chuẩn để cung cấp các ID này.

Bởi vì ít biết về object ngoài ID của nó, chúng ta gọi các object này là "node". Đây là một query mẫu cho một node:

```
{
  node(id: "4") {
    id
    ... on User {
      name
    }
  }
}
```

- GraphQL schema được định dạng để cho phép fetch (tìm nạp) bất kỳ object nào thông qua field `node` trên root query object. Nó trả về các object tuân theo "Node" [interface](https://graphql.org/learn/schema/#interfaces).
- Field `id` có thể được trích xuất ra khỏi response một cách an toàn và có thể được lưu trữ để sử dụng lại thông qua cache và tìm nạp lại (refetch).
- Client có thể sử dụng các interface fragment để trích xuất thông tin bổ sung cụ thể cho loại phù hợp với node interface. Trong trường hợp này là "User".

Node interface trông như sau:

```
# An object with a Globally Unique ID
interface Node {
  # The ID of the object.
  id: ID!
}
```

1 User thực thi node interface sẽ như sau:

```
type User implements Node {
  id: ID!
  # Full name
  name: String!
}
```

## Specification

Mọi thứ bên dưới mô tả với các requirement chính thức hơn, một specification đặc tả tuân thủ xung quanh object identifier nhằm đảm bảo tính nhất quán việc implementation các server . Các thông số kỹ thuật này tuân thủ với Relay API Client, nhưng có thể hữu ích cho bất kỳ client nào khác.

## Reserved Types

GraphQL server tương thích với thông số kỹ thuật này phải dành riêng một số kiểu (type) và tên kiểu (type name) nhất định để hỗ trợ nhất quán object identifier model. Đặc biệt, thông số kỹ thuật này tạo ra các hướng dẫn cho các kiểu sau:

- 1 interface đặt tên là `Node`
- 1 field `node` trong kiểu root query.

## Node interface

Server phải cung cấp một interface được gọi là `Node`. Interface đó phải bao gồm chính xác một field, được gọi là `id`,  trả về 1 non-null `ID`.

`id` này phải là mã định danh duy nhất trên global cho object và chỉ cần cung cấp `id`, server sẽ có thể tìm nạp lại object được.

### Introspection

Server triển khai đúng interface trên sẽ chấp nhận introspection query dưới đây và trả về response đã cung cấp:

```
{
  __type(name: "Node") {
    name
    kind
    fields {
      name
      type {
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
```

- Result 

```
{
  "__type": {
    "name": "Node",
    "kind": "INTERFACE",
    "fields": [
      {
        "name": "id",
        "type": {
          "kind": "NON_NULL",
          "ofType": {
            "name": "ID",
            "kind": "SCALAR"
          }
        }
      }
    ]
  }
}
```

## Node root field

Server phải cung cấp một root field (1 trường gốc) được gọi là `node` trả về `Node` inteface. Root field này phải nhận chính xác một argument, một non-null `ID` có tên là `id`.

Nếu một query trả về một object implement `Node`, thì root field này sẽ tìm nạp lại object khi giá trị được server trả về trong field `id` của `Node` được truyền dưới dạng argument `id` cho root field `node`.

Server phải cố gắng hết sức để tìm nạp dữ liệu này, nhưng không phải lúc nào cũng có thể thực hiện được; ví dụ: server có thể trả về `User` có `id` hợp lệ, nhưng khi request được thực hiện để tìm nạp lại user đó bằng root node field, database của user có thể không khả dụng hoặc user có thể đã xóa tài khoản của họ. Trong trường hợp này, kết quả của việc việc query field này phải là `null`.

### Introspection

Server thực hiện đúng requirement trên sẽ chấp nhận instrospection query sau và trả về resonse có chứa response đã cung cấp.

```
{
  __schema {
    queryType {
      fields {
        name
        type {
          name
          kind
        }
        args {
          name
          type {
            kind
            ofType {
              name
              kind
            }
          }
        }
      }
    }
  }
}
```

- Result

```
{
  "__schema": {
    "queryType": {
      "fields": [
        // This array may have other entries
        {
          "name": "node",
          "type": {
            "name": "Node",
            "kind": "INTERFACE"
          },
          "args": [
            {
              "name": "id",
              "type": {
                "kind": "NON_NULL",
                "ofType": {
                  "name": "ID",
                  "kind": "SCALAR"
                }
              }
            }
          ]
        }
      ]
    }
  }
}
```

## Field stability (Tính ổn định của field)

Nếu hai object xuất hiện trong một query, cả hai đều implement `Node` có `ID` giống hệt nhau, thì hai object phải bằng nhau.

Theo mục đích của định nghĩa này, sự bằng nhau của object được định nghĩa như sau:

- Nếu một field được query trên cả hai object, thì kết quả của việc query field đó trên object đầu tiên phải bằng kết quả của việc query field đó trên object thứ hai.
  - Nếu field trả về một scalar value, thì sự bằng nhau giữa 2 đối tượng được define định là sử dụng định nghĩa bằng nhau của slacar type đó.
  - Nếu field  trả về một enum, thì sự bằng nhau được định nghĩa là cả hai field trả về cùng một giá trị enum.
  - Nếu trường trả về một object, thì sự bằng nhau (đẳng thức) được định nghĩa một cách đệ quy như trên.

Ví dụ:

```
{
  fourNode: node(id: "4") {
    id
    ... on User {
      name
      userWithIdOneGreater {
        id
        name
      }
    }
  }
  fiveNode: node(id: "5") {
    id
    ... on User {
      name
      userWithIdOneLess {
        id
        name
      }
    }
  }
}
```

Có thể trả về: 

```
{
  "fourNode": {
    "id": "4",
    "name": "Mark Zuckerberg",
    "userWithIdOneGreater": {
      "id": "5",
      "name": "Chris Hughes"
    }
  },
  "fiveNode": {
    "id": "5",
    "name": "Chris Hughes",
    "userWithIdOneLess": {
      "id": "4",
      "name": "Mark Zuckerberg",
    }
  }
}
```

## Plural identifying root fields

Hãy tưởng tượng một root field có tên `username`, lấy tên của user và trả về user tương ứng:

```
{
  username(username: "zuck") {
    id
  }
}
```

Có thể trả lại:

```
{
  "username": {
    "id": "4",
  }
}
```

Rõ ràng, chúng ta có thể liên kết object trong response, user có ID 4 xác định object với username là "zuck". Bây giờ hãy tưởng tượng một root field có tên là username, field này nhận một list username và trả về một list các object:

```
{
  usernames(usernames: ["zuck", "moskov"]) {
    id
  }
}
```

có thể trả lại:

```
{
  "usernames": [
    {
      "id": "4",
    },
    {
      "id": "6"
    }
  ]
}
```

Để client có thể liên kết username với các response, cần biết rằng array trong response sẽ có cùng kích thước với array được truyền dưới dạng argument và thứ tự trong response sẽ khớp với thứ tự trong argument . Chúng tôi gọi đây là `plurals identify root field`.

## Fields

Server tuân thủ thông số kỹ thuật này có thể hiển thị các root fields nhận danh sách các input argument và trả về danh sách các response. Để các client tuân thủ đặc điểm sử dụng các field này, các field này phải là __plural identifying root fields__ và tuân theo các requirement sau.

LƯU Ý: Server tuân thủ thông số kỹ thuật có thể để lộ các root field không phải là __plural identifying root fields__; client sẽ không thể sử dụng các field đó làm root field trong các query của nó.

__Plural identifying root fields__ chỉ có một argument duy nhất. Kiểu của argument đó phải là một `non-null` list các giá trị `non-null`. Trong ví dụ về `usernames`, field sẽ nhận một argument duy nhất có tên là `usernames`, có kiểu là sẽ là `[String!] !`.

Kiểu trả về của __plural identifying root fields__ phải là một list hoặc một `non-null` wrapper xung quanh một list. List phải bao bọc `Node` interface, một object implement `Node` interface hoặc một `non-null` wrapper xung quanh các kiểu đó.

Bất cứ khi nào __plural identifying root field__ được sử dụng, độ dài của list trong response phải bằng độ dài của list trong các arguments. Mỗi item trong response phải tương ứng với item của nó trong đầu vào; nếu truyền vào root field một list đầu vào `Lin` dẫn đến giá trị đầu ra `Lout`, thì đối với một hoán vị `P` tùy ý, việc truyền `P(Lin)` phải dẫn đến giá trị đầu ra `P(Lout)`.

Do đó, các server được khuyến nghị không có kiểu response `non-null` wrapper, bởi vì nếu không thể tìm nạp object cho một mục đã cho trong đầu vào, nó vẫn phải cung cấp một giá trị trong đầu ra; thường là `null`.