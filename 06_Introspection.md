# Introspection

Sẽ hữu ích khi hỏi GraphQL schema để biết thông tin về những query mà nó hỗ trợ. GraphQL cho phép chúng ta làm như vậy bằng cách sử dụng `introspection` system!

Đối với ví dụ về StarWars, file [starWarsIntrospection-test.js](https://github.com/graphql/graphql-js/blob/master/src/__tests__/starWarsIntrospection-test.js) chứa một số query thể hiện introspection system.

Chúng ta đã thiết kế type system, vì vậy chúng ta biết những kiểu nào có sẵn, nhưng nếu không, chúng tôi có thể hỏi GraphQL, bằng cách query field `__schema`, luôn có sẵn trên root type của `Query`.

```
{
  __schema {
    types {
      name
    }
  }
}
```

- Result

```
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Query"
        },
        {
          "name": "String"
        },
        {
          "name": "ID"
        },
        {
          "name": "Mutation"
        },
        {
          "name": "Episode"
        },
        {
          "name": "Character"
        },
        {
          "name": "Int"
        },
        {
          "name": "LengthUnit"
        },
        {
          "name": "Human"
        },
        {
          "name": "Float"
        },
        {
          "name": "Droid"
        },
        {
          "name": "FriendsConnection"
        },
        {
          "name": "FriendsEdge"
        },
        {
          "name": "PageInfo"
        },
        {
          "name": "Boolean"
        },
        {
          "name": "Review"
        },
        {
          "name": "ReviewInput"
        },
        {
          "name": "Starship"
        },
        {
          "name": "SearchResult"
        },
        {
          "name": "__Schema"
        },
        {
          "name": "__Type"
        },
        {
          "name": "__TypeKind"
        },
        {
          "name": "__Field"
        },
        {
          "name": "__InputValue"
        },
        {
          "name": "__EnumValue"
        },
        {
          "name": "__Directive"
        },
        {
          "name": "__DirectiveLocation"
        }
      ]
    }
  }
}
```

Hãy nhóm chúng lại:
- **Query, Character, Human, Episode, Droid**: Các kiểu mà chúng ta đã define trong type system.
- **String, Boolean**: Đây là các scalar type tích hợp sẵn mà type system cung cấp.
- **__Schema, __Type, __TypeKind, __Field, __InputValue, __EnumValue, __Directive**: Tất cả chúng đều được đặt trước bằng một dấu gạch dưới kép (`__`), cho thấy rằng chúng là một phần của introspection system.

Chúng ta hãy thử và tìm ra một nơi tốt để bắt đầu khám phá những query có sẵn. Khi chúng ta thiết kế type system của mình, chúng ta đã chỉ định kiểu của tất cả các query sẽ bắt đầu.

```
{
  __schema {
    queryType {
      name
    }
  }
}
```

- Result 

```
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Query"
      }
    }
  }
}
```

Và điều đó khớp với những gì chúng ta đã nói trong phần type system, rằng kiểu `Query` là chỗ chúng ta sẽ bắt đầu! Lưu ý rằng việc đặt tên ở đây chỉ là theo quy ước; chúng ta có thể đặt tên cho kiểu `Query` của mình là bất kỳ thứ gì khác, và nó vẫn sẽ được trả lại ở đây nếu chúng tôi chỉ định nó là kiểu bắt đầu cho các query. Đặt tên nó `Query`, tuy nhiên, là một quy ước hữu ích.

```
{
  __type(name: "Droid") {
    name
  }
}
```

- Result 

```
{
  "data": {
    "__type": {
      "name": "Droid"
    }
  }
}
```

Tuy nhiên, nếu chúng ta muốn biết thêm về `Droid` thì sao? Ví dụ, nó là một `interface` hay một `object`?

```
{
  __type(name: "Droid") {
    name
    kind
  }
}
```

- Result

```
{
  "data": {
    "__type": {
      "name": "Droid",
      "kind": "OBJECT"
    }
  }
}
```

`kind` trả về một enum `__TypeKind`, một trong các giá trị của nó là `OBJECT`. Nếu chúng ta hỏi về `Character`, chúng ta sẽ thấy rằng đó là một interface:

```
{
  __type(name: "Character") {
    name
    kind
  }
}
```

- Result 

```
{
  "data": {
    "__type": {
      "name": "Character",
      "kind": "INTERFACE"
    }
  }
}
```

Rất hữu ích cho một object để biết những field nào có sẵn, vì vậy hãy hỏi introspection system về `Droid`:

```
{
  __type(name: "Droid") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

- Result

```
{
  "data": {
    "__type": {
      "name": "Droid",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          }
        },
        {
          "name": "name",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          }
        },
        {
          "name": "friends",
          "type": {
            "name": null,
            "kind": "LIST"
          }
        },
        {
          "name": "friendsConnection",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          }
        },
        {
          "name": "appearsIn",
          "type": {
            "name": null,
            "kind": "NON_NULL"
          }
        },
        {
          "name": "primaryFunction",
          "type": {
            "name": "String",
            "kind": "SCALAR"
          }
        }
      ]
    }
  }
}
```

Đó là những fields mà chúng ta đã define trên `Droid`!

`id` trông hơi kỳ lạ, nó không có tên cho kiểu. Đó là bởi vì nó là loại "wrapper" kiểu `NON_NULL`. Nếu chúng ta query `ofType` trên kiểu của field đó, chúng ta sẽ tìm thấy `ID` type ở đó, cho chúng ta biết rằng đây là một `non-null` ID.

Tương tự, `friends` và `appearsIn` đều không có tên, vì chúng là  kiểu `LIST` wrapper. Chúng ta có thể query `ofType` trên các kiểu đó, sẽ biết đây là list gì.

```
{
  __type(name: "Droid") {
    name
    fields {
      name
      type {
        name
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
  "data": {
    "__type": {
      "name": "Droid",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "ID",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "name",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "String",
              "kind": "SCALAR"
            }
          }
        },
        {
          "name": "friends",
          "type": {
            "name": null,
            "kind": "LIST",
            "ofType": {
              "name": "Character",
              "kind": "INTERFACE"
            }
          }
        },
        {
          "name": "friendsConnection",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "FriendsConnection",
              "kind": "OBJECT"
            }
          }
        },
        {
          "name": "appearsIn",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": null,
              "kind": "LIST"
            }
          }
        },
        {
          "name": "primaryFunction",
          "type": {
            "name": "String",
            "kind": "SCALAR",
            "ofType": null
          }
        }
      ]
    }
  }
}
```

Một tính năng của introspection system đặc biệt hữu ích cho việc tạo tool; hãy request `description`.

```
{
  __type(name: "Droid") {
    name
    description
  }
}
```

- Result

```
{
  "data": {
    "__type": {
      "name": "Droid",
      "description": "An autonomous mechanical character in the Star Wars universe"
    }
  }
}
```

Vì vậy, chúng ta  có thể truy cập documentation về type system bằng cách sử dụng introspection và tạo documentation browsers hoặc trải nghiệm IDE.

Trên đây chỉ là bề mặt của introspection system; chúng ta có thể query các giá trị enum, các interface, v.v. Chúng ta thậm chí có thể xem chính introspection system. Specification sẽ đi vào chi tiết hơn về chủ đề này trong phần "Introspection" và field [introspection](https://github.com/graphql/graphql-js/blob/master/src/type/introspection.js) trong GraphQL.js chứa code implement introspection system.