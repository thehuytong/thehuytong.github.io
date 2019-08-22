---
layout: post
published: false
title: Dependency Injection trong Swift
---
Nhiều developers vẫn còn cảm thấy lạ lẫm khi nhắc đến **Dependency Injection**. Mọi người thường nghĩ đó là một pattern khó và không dành cho những người mới. Nhưng sư thật thì Dependency Injection là một pattern rất cơ bản và rất dễ để áp dụng.

Có một câu nói về Dependency Injection của [James Shore](https://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html) mà tôi ưa thích.

{: .box-note}
Dependency Injection is a 25-dollar term for a 5-cent concept. - [James Shore](https://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html)

Khi tôi lần đầu tiên nghe đến dependency injection, tôi cũng hình dung đó là một kỹ thuật quá nâng cao so với nhu cầu của tôi vào thời điểm đó. Tôi có thể làm mà không cần dependency injection, bất kể đó là gì.

# **Dependency Injection là gì**

Sau này tôi mới biết rằng, dependency injection là một khái niệm đơn giản. James Shore đã đưa ra một định nghĩa ngắn gọn và đơn giản về dependency injection.

{: .box-note}
Dependency injection means giving an object its instance variables. Really. That's it. - [James Shore](https://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html)

Đối với các developers mới sử dụng dependency injection, điều quan trọng là phải tìm hiểu những điều cơ bản trước khi dựa vào framework hoặc library. Bắt đầu đơn giản. Rất có thể là bạn đã từng sử dụng dependency injection rồi mà không nhận ra nó.

////

# **Ví dụ**

Trong ví dụ này, chúng ta định nghĩa một lớp con của `UIViewController`, khai báo một thuộc tính `requestManager`, thuộc loại `RequestManager?`.

```swift
import UIKit

class ViewController: UIViewController {

    var requestManager: RequestManager?

}
```

Chúng ta có thể đặt giá trị cho thuộc tính `requestManager` theo một trong hai cách.

### Without Dependency Injection

Tùy chọn đầu tiên là giao nhiệm vụ cho lớp `ViewController` với việc khởi tạo thể hiện RequestManager. Chúng ta có thể làm thuộc tính lazy hoặc khởi tạo `requestManager` trong hàm khởi tạo của view controller. Đó không phải là vấn đề. Vấn đề ở đây là view controller phụ trách việc tạo thực thể của `RequestManager`.

```swift
import UIKit

class ViewController: UIViewController {

    lazy var requestManager: RequestManager? = RequestManager()

}
```

Điều này có nghĩa là lớp `ViewController` không chỉ biết về hành vi của lớp `RequestManager`. `ViewController` cũng biết về việc khởi tạo của `RequestManager`. Đó là một chi tiết nhỏ nhưng quan trọng.

### With Dependency Injection

Nhưng có một lựa chọn khác. Chúng ta có thể đưa đối tượng `RequestManager` vào đối tượng `ViewController`. Mặc dù kết quả cuối cùng có thể xuất hiện giống hệt nhau, nhưng chắc chắn là không. Bằng cách chèn request manager, view controller không biết cách khởi tạo request manager.

```swift
// Initialize View Controller
let viewController = ViewController()

// Configure View Controller
viewController.requestManager = RequestManager()
```

Nhiều developers ngay lập tức loại bỏ lựa chọn này vì nó cồng kềnh và phức tạp không cần thiết. Nhưng nếu bạn xem xét các lợi ích của nó mang lại, dependency injection sẽ trở nên hấp dẫn hơn.

# **Dependency Injection là gì**

Tôi muốn cho bạn thấy một ví dụ khác để nhấn mạnh điểm tôi đã đưa ra trước đó. Hãy xem ví dụ sau đây.

```swift
protocol Serializer {

    func serialize(data: AnyObject) -> NSData?

}

class RequestSerializer: Serializer {

    func serialize(data: AnyObject) -> NSData? {
        ...
    }

}

class DataManager {

    var serializer: Serializer? = RequestSerializer()

}
```

Lớp `DataManager` có thuộc tính `serializer` kiểu dữ liệu `Serializer?`. Trong ví dụ này, `Serializer` là một protocol. Lớp `DataManager` chịu trách nhiệm khởi tạo một đối tượng `Serializer`, trong ví dụ này là lớp `RequestSerializer`.

Lớp `DataManager` có nên biết cách khởi tạo một đối tượng kiểu `Serializer` không? Hãy xem ví dụ này. Nó sẽ cho bạn thấy sức mạnh của protocol và Dependency Injection.

```swift
// Initialize Data Manager
let dataManager = DataManager()

// Configure Data Manager
dataManager.serializer = RequestSerializer()
```

Lớp `DataManager` không còn chịu trách nhiệm khởi tạo lớp `RequestSerializer`. Nó không còn gán một giá trị cho thuộc tính `serializer`. Trên thực tế, chúng ta có thể thay thế `RequestSerializer` bằng một loại khác miễn là nó tuân theo protocol serializer. `DataManager` không còn biết hoặc quan tâm đến những chi tiết này.

# **Bạn nhận được gì**

Tôi hy vọng rằng các ví dụ tôi đã chỉ ra ít nhất đã thu hút sự chú ý của bạn. Hãy để tôi liệt kê thêm một vài lợi ích của dependency injection.