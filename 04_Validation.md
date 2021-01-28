# Validation

Bằng cách sử dụng type system, có thể được xác định trước liệu một GraphQL query có hợp lệ hay không. Điều này cho phép server và client thông báo hiệu quả cho developer khi một query không hợp lệ đã được tạo mà không cần phải dựa vào runtime checks.

Đối với ví dụ về StarWar, file `starWarsValidation-test.js` chứa một số query thể hiện các case không hợp lệ khác nhau và là file test có thể chạy để luyện tập implemetation validator.

Để bắt đầu, hãy thực hiện một query hợp lệ phức tạp. Đây là một nested query, tương tự như một ví dụ từ phần trước, nhưng với các field trùng lặp được tính thành một fragment:

```
{
  hero {
    ...NameAndAppearances
    friends {
      ...NameAndAppearances
      friends {
        ...NameAndAppearances
      }
    }
  }
}

fragment NameAndAppearances on Character {
  name
  appearsIn
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
      ],
      "friends": [
        {
          "name": "Luke Skywalker",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Han Solo",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Leia Organa",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "C-3PO",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        },
        {
          "name": "Han Solo",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Luke Skywalker",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Leia Organa",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        },
        {
          "name": "Leia Organa",
          "appearsIn": [
            "NEWHOPE",
            "EMPIRE",
            "JEDI"
          ],
          "friends": [
            {
              "name": "Luke Skywalker",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "Han Solo",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "C-3PO",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            },
            {
              "name": "R2-D2",
              "appearsIn": [
                "NEWHOPE",
                "EMPIRE",
                "JEDI"
              ]
            }
          ]
        }
      ]
    }
  }
}
```

Và query này là hợp lệ. Hãy xem xét một số query không hợp lệ ...

Một fragment không thể tham chiếu đến chính nó hoặc tạo ra một cycle (chu trình), vì điều này có thể dẫn đến một kết quả không bị ràng buộc (unbounded result)! Đây là cùng một query ở trên nhưng không có ba cấp lồng ghép rõ ràng:

```
{
  hero {
    ...NameAndAppearancesAndFriends
  }
}

fragment NameAndAppearancesAndFriends on Character {
  name
  appearsIn
  friends {
    ...NameAndAppearancesAndFriends
  }
}
```

- Result

```
{
  "errors": [
    {
      "message": "Cannot spread fragment \"NameAndAppearancesAndFriends\" within itself.",
      "locations": [
        {
          "line": 11,
          "column": 5
        }
      ]
    }
  ]
}
```

Khi chúng ta query các field, chúng ta phải query một field tồn tại trên kiểu đã cho. Vì vậy, khi `hero` trả về một `Character`, chúng ta phải query một field trên `Character`. Kiểu đó không có field `favouriteSpaceship`, vì vậy query này không hợp lệ:

```
# INVALID: favoriteSpaceship does not exist on Character
{
  hero {
    favoriteSpaceship
  }
}
```

- Result

```
{
  "errors": [
    {
      "message": "Cannot query field \"favoriteSpaceship\" on type \"Character\".",
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

Bất cứ khi nào chúng ta query một field và nó trả về thứ gì đó không phải là scalar hoặc enum, chúng ta cần chỉ định dữ liệu nào chúng ta muốn lấy lại từ field tương ứng. `hero` trả về một `Character`, và chúng ta đã request các field như `name` và `appearsIn` trong đó; nếu chúng tôi bỏ qua điều đó, query sẽ không hợp lệ:

```
# INVALID: hero is not a scalar, so fields are needed
{
  hero
}
```

- Result 

```
{
  "errors": [
    {
      "message": "Field \"hero\" of type \"Character\" must have a selection of subfields. Did you mean \"hero { ... }\"?",
      "locations": [
        {
          "line": 3,
          "column": 3
        }
      ]
    }
  ]
}
```

Tương tự, nếu một field là một scalar, việc query các field  bổ sung trên đó sẽ không hợp lý và làm như vậy sẽ làm cho query không hợp lệ:


```
# INVALID: name is a scalar, so fields are not permitted
{
  hero {
    name {
      firstCharacterOfName
    }
  }
}
```

- Result

```
{
  "errors": [
    {
      "message": "Field \"name\" must not have a selection since type \"String!\" has no subfields.",
      "locations": [
        {
          "line": 4,
          "column": 10
        }
      ]
    }
  ]
}
```

Trước đó, lưu ý rằng một query chỉ có thể query các field thuộc kiểu được đề cập; khi chúng ta truy vấn `hero` trả về một `Character`, chúng ta chỉ có thể query các field tồn tại trên `Character`. Tuy nhiên, điều gì xảy ra nếu chúng ta muốn query `primaryFunction` của R2-D2s?

```
# INVALID: primaryFunction does not exist on Character
{
  hero {
    name
    primaryFunction
  }
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
          "line": 5,
          "column": 5
        }
      ]
    }
  ]
}
```

Query đó không hợp lệ, bởi vì `primaryFunction` không phải là một field của `Character`. Chúng ta muốn biết rằng chúng ta hi vọng fetch `primaryFunction` nếu `Chẩcxter` là `Droid` và bỏ qua field đó nếu ngược lại. Chúng ta có thể sử dụng các fragment mà chúng ta đã giới thiệu trước đó để thực hiện việc này. Bằng cách thiết lập một fragment được xác định trên `Droid` và bao gồm (including) nó, chúng ta đảm bảo rằng chỉ query cho `primaryFunction` tại chỗ mà nó được define.

```
{
  hero {
    name
    ...DroidFields
  }
}

fragment DroidFields on Droid {
  primaryFunction
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

Query này hợp lệ, nhưng hơi dài dòng; các fragment được đặt tên ở trên có giá trị khi chúng ta sử dụng chúng nhiều lần, nhưng chúng ta chỉ sử dụng fragment này một lần. Thay vì sử dụng một fragment được đặt tên, chúng ta có thể sử dụng một `inline fragment`; điều này vẫn cho phép chúng ta chỉ ra kiểu mà chúng ta đang query, nhưng không đặt tên cho một fragment riêng biệt:

```
{
  hero {
    name
    ... on Droid {
      primaryFunction
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
      "primaryFunction": "Astromech"
    }
  }
}
```

Điều này chỉ tác động nhỏ đến bề mặt của validation system; có một số quy tắc validation để đảm bảo rằng GraphQL query có ý nghĩa về mặt ngữ nghĩa. Specification đi chi tiết hơn về chủ đề này trong phần "Validation", và [validation](https://github.com/graphql/graphql-js/blob/master/src/validation) folder trong GraphQL.js chứa code implement GraphQL validator có tuân thủ hết đặc điểm kỹ thuật.