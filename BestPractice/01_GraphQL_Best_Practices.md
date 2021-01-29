# GraphQL Best Practices

Đặc điểm kỹ thuật GraphQL cố ý không nhắc đến một số vấn đề quan trọng mà API phải đối mặt như network, authorization và pagination. Điều này không có nghĩa là không có giải pháp cho những vấn đề này khi sử dụng GraphQL, chỉ là chúng không có ở trong mô tả về GraphQL và thay vào đó chỉ là thông lệ (common practice).

Các bài viết trong phần này không nên được coi là chân lý và trong một số trường hợp có thể bỏ qua một cách chính đáng để ủng hộ một số cách tiếp cận khác. . để giải quyết các vấn đề phổ biến như serving qua HTTP và authorization.

Sau đây là những mô tả ngắn gọn về một số phương pháp hay, phổ biến hơn và có lập trường do các GraphQL service nắm giữ, tuy nhiên mỗi bài viết trong phần này sẽ đi sâu hơn về từng chủ đề

## HTTP

GraphQL thường được phân phối qua HTTP thông qua một end point biểu thị toàn bộ khả năng của service. Điều này trái ngược với các API REST hiển thị một bộ URL, mỗi URL hiển thị một tài nguyên. Mặc dù GraphQL có thể được sử dụng cùng với một tập URL, nhưng có thể khiến việc sử dụng với các công cụ như [GraphiQL](https://github.com/graphql/graphiql) trở nên khó khăn hơn.

Xem thêm tại page [Serving over HTTP](https://graphql.org/learn/serving-over-http/).

## JSON (với GZIP)

GraphQL service thường respond kết quả bằng JSON, tuy nhiên GraphQL spec [không yêu cầu điều đó](http://spec.graphql.org/draft/#sec-Serialization-Format). JSON có vẻ như là một lựa chọn kỳ lạ cho API layer, hứa hẹn đem lại hiệu suất mạng tốt hơn, tuy nhiên vì văn bản nên nó nén đặc biệt tốt với GZIP.

Chúng tôi khuyến khích các GraphQL service cung cấp GZIP và khuyến khích client gửi header:

```
Accept-Encoding: gzip
```

JSON cũng rất quen thuộc với các API developer và client application, đồng thời rất dễ đọc và dễ debug. Trên thực tế, cú pháp GraphQL một phần được lấy cảm hứng từ cú pháp JSON.

## Versioning

Mặc dù không có gì ngăn cản service GraphQL được tạo phiên bản giống như bất kỳ API REST nào khác, nhưng GraphQL có quan điểm mạnh mẽ về việc tránh lập phiên bản (versioning) bằng cách cung cấp các tool cho sự phát triển liên tục của GraphQL schema.

Tại sao hầu hết các API đều có version? Khi có sự kiểm soát hạn chế đối với dữ liệu được trả về từ API endpoint, `bất kỳ thay đổi nào` cũng có thể được coi là `breaking change` và các `breaking change` luôn phải yêu cầu 1 version mới. Nếu việc thêm các tính năng mới vào API phải cần một version mới, thì mới giữ được sự cân bằng giữa việc release thường xuyên và có nhiều version gia tăng so với tính dễ hiểu và khả năng bảo trì của API.

Ngược lại, GraphQL chỉ trả về dữ liệu được request rõ ràng, do đó, các capability (khả năng) mới có thể được thêm vào thông qua các kiểu mới và field mới trên các kiểu đó mà không tạo ra thay đổi đột ngột. Điều này dẫn đến một thực tiễn phổ biến là luôn tránh các `breaking change` và cung cấp API không có phiên bản (versionless API).

## Nullability

Hầu hết các type system có "null" đều cung cấp cả kiểu chung (common type) và phiên bản `nullable` riêng của kiểu đó, cụ thể là các kiểu mặc định không bao gồm "null" trừ khi được khai báo rõ ràng. Tuy nhiên, trong GraphQL type system, mọi field đều có thể `nullable` theo mặc định. Bởi vì database có thể bị hỏng, một `asynchronous action` có thể fail, một exxcpetion có thể được ném ra. Ngoài lỗi hệ thống đơn giản, authorization thường có thể là hạt nhân sinh lỗi, vì có nhiều fields đơn lẻ trong 1 request có thể có nhiều authorization rule khác nhau.

Bằng cách đặt mặc định mọi field thành `nullable`, bất kỳ lý do nào trong số này có thể dẫn đến việc field  đó được trả về "null" chứ không phải trả về lỗi cho request. Thay vào đó, GraphQL cung cấp các biến thể `non-null` của các kiểu, đảm bảo cho client rằng nếu request, field sẽ không bao giờ trả về "null". Thay vào đó, nếu lỗi xảy ra, parent field trước đó sẽ là "null".

Khi thiết kế một GraphQL schema, điều quan trọng là phải ghi nhớ tất cả các vấn đề có thể xảy ra sai và nếu "null" là một giá trị thích hợp cho một field bị lỗi. Thường là như vậy, nhưng đôi khi, không phải vậy. Trong những trường hợp đó, hãy sử dụng các kiểu `non-null` để đảm bảo điều đó.

## Pagination

GraphQL type system cho phép một số field trả về [list values](https://graphql.org/learn/schema/#lists-and-non-null), nhưng để phân trang list values dài hơn thì tùy thuộc vào thiết kế API.

Thông thường, các field có thể trả về list dài chấp nhận đối số "first" và "after" để cho phép chỉ định một vùng cụ thể của list, trong đó "after" là số nhận dạng duy nhất của từng giá trị trong list.

Cuối cùng, việc thiết kế API với phân trang đã dẫn đến `best practice pattern` được gọi là "Connection". Một số client tool cho GraphQL, chẳng hạn như [Relay](https://facebook.github.io/relay/), biết về Connections pattern và có thể tự động hỗ trợ phân trang cho phía client khi API GraphQL sử dụng pattern này.

Xem thêm [Pagination](https://graphql.org/learn/pagination/).