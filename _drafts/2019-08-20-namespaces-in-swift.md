---
layout: post
published: true
title: Namespaces in Swift
---

# **Defining Namespace**

**Namespace** là một vùng được đặt tên của chương trình được sử dụng để nhóm các variables, types và methods. Namespace có những lợi ích sau:
- Cho phép cải thiện cấu trúc code bằng cách tổ chức các phần tử.
- Tránh xung đột đặt tên.
- Cung cấp tính đóng gói.

Mặc dù bị giới hạn trong Swift, chúng ta có thể có mô phỏng namespace bằng struct hoặc enum lồng nhau.

# **Using struct**

Giải pháp đầu tiên sử dụng các struc để tạo các namespace và nó trông giống như thế này.

```swift
struct API {

    static let BaseURL = "https://example.com/v1/"
    static let Token = "sdfiug8186qf68qsdf18389qsh4niuy1"

}
```

```swift
import Foundation

if let url = URL(string: API.BaseURL) {
    ...
}
```

Có một vấn đề là struct `API` có thể được khởi tạo. Mặc dù đây không phải là vấn đề nghiêm trọng, nhưng nó có thể gây nhầm lẫn cho các developer khác làm việc trong dự án. Bạn có thể khai báo initializer private, khiến nó không thể truy cập được ở các phần khác của dự án.

```swift
struct API {

    private init() {}

    static let BaseURL = "https://example.com/v1/"
    static let Token = "sdfiug8186qf68qsdf18389qsh4niuy1"

}
```


![figure-initialize-structure-1](https://cocoacasts.s3.amazonaws.com/namespaces-in-swift/figure-initialize-structure-1.jpg)



# **Using enum**

Thay vì sử dụng struct, ta có thể sử dụng enum mà không có các caces. Một enum không có cases không thể được khởi tạo, nhưng nó có thể hoạt động như một namespace.

```swift
enum API {

    static let BaseURL = "https://example.com/v1/"
    static let Token = "sdfiug8186qf68qsdf18389qsh4niuy1"

}
```

![figure-initialize-enumeration-1](https://cocoacasts.s3.amazonaws.com/namespaces-in-swift/figure-initialize-enumeration-1.jpg)

Bạn không nhất thiết phải đặt các hằng số của dự án trong một enum hoặc struct duy nhất. Hãy xem ví dụ sau đây.

```swift
enum API {

    static let BaseURL = "https://example.com/v1/"
    static let Token = "sdfiug8186qf68qsdf18389qsh4niuy1"

}

extension UserDefaults {

    enum Keys {

        static let CurrentVersion = "currentVersion"
        static let DarkModeEnabled = "darkModeEnabled"

    }

}
```

Ta tạo một extension cho class `UserDefaults` và định nghĩa một enum lồng nhau, `Keys`.

```swift
// Update User Defaults
let userDefaults = UserDefaults.standard
userDefaults.set(1.0, forKey: UserDefaults.Keys.CurrentVersion)
```

Một ví dụ khác về việc nhóm các hằng số của view controller:

```swift
class ItemListViewController {
    ...
}

extension ItemListViewController {

    enum Constants {
        static let itemsPerPage = 7
        static let headerHeight: CGFloat = 60
    }
}
```

Nếu các hằng số trên là biến toàn cục thì phải đặt tên như thế nào?

```swift
let itemListViewControllerItemsPerPage = 7
let itemListViewControllerHeaderHeight: CGFloat = 60
```

Rất là khó đọc khi so với:

```swift
ItemListViewController.Constants.itemsPerPage
ItemListViewController.Constants.headerHeight
```

# **Summary**

Việc cấu trúc code là rất quan trọng. Sử dụng namespace có thể cải thiện cấu trúc code bằng cách nhóm các thứ liên quan lại trong phạm vi cục bộ. Và nó củng rất dễ để thực hiện.

# **References**

- [The Power of Namespacing in Swift](https://www.vadimbulavin.com/the-power-of-namespacing-in-swift)
- [Namespaces in Swift](https://cocoacasts.com/namespaces-in-swift)

