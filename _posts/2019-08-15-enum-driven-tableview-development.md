---
layout: post
published: false
title: Enum-Driven TableView Development
subtitle: >-
  Trong tutorial, bạn sẽ học cách sử dụng Swift enums để handle các states khác
  nhau của app để hiển thị lên table view.
---
**UITableView** là một trong những thứ cơ bản trong iOS development. Tuy nhiên nhiều lúc nó củng chứa đựng nhiều thứ phức tạp như show loading indicators, handle errors, gọi service và hiển thị kết quả trả về.

Trong tutorial này, bạn sẽ học cách sử dụng **Enum-Driven TableView Development** để quản lý những thứ phức tạp trên.

Để làm theo kỹ thuật này, bạn sẽ refactor một app có sẵn tên là _Chirper_. Bạn sẽ học được những điều sau:
- Cách sử dụng enum để quản lý state của `ViewController`.
- Tầm quan trọng của việc phản hồi state trong view lại cho user.
- Sự nguy hiểm của việc poorly defined state.
- Cách sử dụng property observers để giữ cho view up-to-date.
- Cách làm việc với pagination(phân trang).

{: .box-note}
**Note:** This is a notification box.

## Getting Started

App _Chirper_ mà bạn sẽ refactor trong tutorial này trình bày một danh sách có thể tìm kiếm các tiếng chim được lấy từ API [xeno-canto](https://www.xeno-canto.org/article/153)

Nếu bạn tìm kiếm một loài chim trong app, nó sẽ hiển thị cho bạn một danh sách phù hợp với truy vấn tìm kiếm của bạn. Bạn có thể phát các bản ghi âm bằng cách nhấn vào nút play ở mỗi hàng.

![initial-loaded-state-1](https://koenig-media.raywenderlich.com/uploads/2018/03/initial-loaded-state-1.png){: .center-block :}

## Different States

Một well-designed table view has 4 states khác nhau:
- **Loading**: App đang loading data.
- **Error**: Một service call hoặc một operation failed.
- **Empty**: Service call trả về no data.
- **Populated**: App đã lấy được data để hiển thị.

Để bắt đ`ầu, hãy mở **MainViewControll.swift**. 
- View sẽ hiển thị loading indicator khi `isLoading` được set thành `true`.
- View sẽ thông báo lỗi khi `error` khác `nil`.
- Nếu array `recordings` là `nil` hoặc `empty`, view sẽ hiện thị 1 message cho user.
- Nếu không có điều kiện ở trên là đúng, view sẽ hiển thị danh sách `recordings`.
- `tableView.tableFooterView` sẽ được set tuỳ theo trạng thái hiện tại.

Có rất nhiều điều cần lưu ý nếu bạn muốn sửa đổi code. Mọi thứ càng trở nên phức tạp hơn khi muốn thêm nhiều chức năng mới vào app.

## Poorly Defined State

Tìm kiếm trong **MainViewController.swift** và bạn sẽ không thấy bất kì từ `state` nào được nhắc đến.

![basic-confused-2-500x500](https://koenig-media.raywenderlich.com/uploads/2018/03/basic-confused-2-500x500.png){: .center-block :}

Việc định nghĩa state không rõ ràng khiến cho khó hiểu code đang làm gì và phản hồi như thế nào khi các properties thay đổi.

## Invalid State

Nếu `isLoading` là `true`, app sẽ là loading state. Nếu `error` khác `nil`, app sẽ là error state. Nhưng điều gì xảy ra nếu cả hai điều kiện này được đáp ứng? App sẽ ở **invalid state**(trang thái không hợp lệ).

`MainViewController` hiện đang không xác định rõ các states của nó. Điều này nghĩa là có thể sẽ có lỗi do những invalid states.

## A Better Alternative

`MainViewController` cần một cách tốt hơn để quản lý các states của nó. Nó cần một kỹ thuật mà:

- Dễ hiểu
- Dễ bảo trì
- Không lỗi

Trong các bước tiếp theo, bạn sẽ refactor `MainViewController` sử dụng `enum` để quản lý các states của nó.

# **Refactoring to a State Enum**

Trong **MainViewController.swift**, thêm phần này lên trên phần khai báo của lớp:

```swift
enum State {
  case loading
  case populated([Recording])
  case empty
  case error(Error)
}
```

Đây là enum dùng để định nghĩa rõ ràng state của view controller. Tiếp theo, thêm thuộc tính state vào `MainViewController`:

```swift
var state = State.loading
```

Build and run, app vẫn chạy như bình thường.

## Refactoring the Loading State

Đầu tiên, hãy remove `isLoading` property. Trong `loadRecordings()`, remove 2 dòng sau:

```swift
isLoading = true
tableView.tableFooterView = loadingView
```

Replace bằng:

```swift
state = .loading
```

Sau đó, remove `self.isLoading = false` bên trong `fetchRecordings` completion block. `loadRecordings()` sẽ trở thành:

```swift
@objc func loadRecordings() {
  state = .loading
  recordings = []
  tableView.reloadData()
    
  let query = searchController.searchBar.text
  networkingService.fetchRecordings(matching: query, page: 1) { [weak self] response in
      
    guard let `self` = self else {
      return
    }
      
    self.searchController.searchBar.endEditing(true)
    self.update(response: response)
  }
}
```

Bây giờ bạn có thể remove `isLoading` property `MainViewController`.


Build and run the app.

![broken-loading-state](https://koenig-media.raywenderlich.com/uploads/2018/03/broken-loading-state.png){: .center-block :}

`state` property đã được set, nhưng bạn chưa làm gì với nó cả. `tableView.tableFooterView` cần phải phản ánh lại state hiện tại. Tạo một method mới trong `MainViewController` có tên `setFooterView()`.

```swift
func setFooterView() {
  switch state {
  case .loading:
    tableView.tableFooterView = loadingView
  default:
    break
  }
}
```

Bây giờ quay về `loadRecordings()`. Sau khi set state thành `.loading`, thêm đoạn sau:

```swift
setFooterView()
```

Build and run the app.

![fixed-loading-state](https://koenig-media.raywenderlich.com/uploads/2018/03/fixed-loading-state.png){: .center-block :}

Bây giờ khi bạn thay đổi state thành loading `setFooterView()` được gọi và loading indicator sẽ được hiển thị.

## Refactoring the Error State

`loadRecordings()` fetches recordings từ `NetworkingService`. Nhận response từ `networkingService.fetchRecordings()` và gọi `update(response:)`.

Trong `update(response:)`, nếu response có error, set error’s description lên `errorLabel`. `errorView` có chứa `errorLabel` được set vào `tableFooterView`. Tìm 2 dòng sau trong `update(response:)`:

```swift
errorLabel.text = error.localizedDescription
tableView.tableFooterView = errorView
```

Replace bằng:

```swift
state = .error(error)
setFooterView()
```

Trong `setFooterView()`, thêm một case mới cho `error` state:

```swift
case .error(let error):
  errorLabel.text = error.localizedDescription
  tableView.tableFooterView = errorView
```

Bây giờ có thể remove `error: Error?` trong view controller. Và remove trong `update(response:)`:

```swift
error = response.error
```

Build and run the app.

Bạn sẽ thấy loading state vẫn hoạt động bình thường. Nhưng làm sau để test error state?. Cách dễ nhất là ngắt kết nối thiết bị của bạn khỏi internet; nếu bạn đang chạy trình giả lập trên máy Mac, hãy ngắt kết nối máy Mac của bạn khỏi internet. Đây là những gì bạn sẽ thấy khi app cố tải dữ liệu:

![error-state](https://koenig-media.raywenderlich.com/uploads/2018/03/error-state.png){: .center-block :}

## Refactoring the Empty and Populated States

Có một chuỗi `if-else` dài ở đầu `update(response:)`. Để dọn dẹp phần này, replace `update(response:)` với:

```swift
func update(response: RecordingsResult) {
  if let error = response.error {
    state = .error(error)
    setFooterView()
    tableView.reloadData()
    return
  }
  
  recordings = response.recordings
  tableView.reloadData()
}
```

Bạn vừa làm hư state _populated_ and _empty_. Đừng lo lắng, bạn sẽ fix chúng sớm thôi.

## Setting the Correct State

Thêm đoạn sau vào dưới `if let error = response.error` block:

```swift
guard let newRecordings = response.recordings,
  !newRecordings.isEmpty else {
    state = .empty
    setFooterView()
    tableView.reloadData()
    return
}
```

Đừng quên gọi `setFooterView()` và `tableView.reloadData`() khi cập nhật state.

Tiếp theo, tìm dòng này trong `update(response:)`:

```swift
recordings = response.recordings
```

Replace bằng:

```swift
state = .populated(newRecordings)
setFooterView()
```
### Setting the Footer View

Tiếp theo, add 2 cases vào trong `setFooterView()`:

```swift
case .empty:
  tableView.tableFooterView = emptyView
case .populated:
  tableView.tableFooterView = nil
```

Remove `default`, bạn không cần nó nữa.

Build and run the app.

![broken-populated-state](https://koenig-media.raywenderlich.com/uploads/2018/03/broken-populated-state.png){: .center-block :}

### Getting Data from the State

App không còn hiển thị dữ liệu nữa. `recordings` property để populate cho table view nhưng nó không được set. Table view bây giờ cần phải lấy dữ liệu từ `state` property. Thêm computed property này vào trong `State` enum:

```swift
var currentRecordings: [Recording] {
  switch self {
  case .populated(let recordings):
    return recordings
  default:
    return []
  }
}
```

Trong `tableView(_:numberOfRowsInSection:)`, remove dòng này:

```swift
return recordings?.count ?? 0
```

Replace với:

```swift
return state.currentRecordings.count
```

Tiếp theo, trong `tableView(_:cellForRowAt:)` remove:

```swift
if let recordings = recordings {
  cell.load(recording: recordings[indexPath.row])
}
```

Replace với:

```swift
cell.load(recording: state.currentRecordings[indexPath.row])
```

Bạn không cần `recordings` property của `MainViewController` nữa. Hãy remove nó và remove trong `loadRecordings()`.

Build and run the app.

Tất cả các states đã hoạt động đúng. Bạn đã remove các properties `isLoading`, `error`, `recordings` và thay bằng property `state`

![AllStates2](https://koenig-media.raywenderlich.com/uploads/2018/06/AllStates2.png){: .center-block :}

# **Keeping in Sync with a Property Observer**

Bây giờ bạn có thể dễ dàng quản lý các hành đọng của view dựa trên property state. Ngoài ra, sẽ không có trường hợp loading state và error state xảy ra đồng thời. Điều đó nghĩa là sẽ không có bug do những invalid state.

Tuy nhiên vẫn còn một vấn đề nữa. Mỗi khi update giá trị cho property state, bạn phải nhớ gọi `setFooterView()` và `tableView.reloadData()`. 

Nếu bạn muốn reload table view và set footer view mỗi khi state được set, bạn cần phải add `didSet` **property observer**.

Replace `var state = State.loading` với:

```swift
var state = State.loading {
  didSet {
    setFooterView()
    tableView.reloadData()
  }
}
```

Remove tất cả những chỗ khác gọi `setFooterView()` và `tableView.reloadData()`. Bạn có thể tìm thấy chúng trong `loadRecordings()` và `update(response:)`

Build and run the app.

![after-property-observer](https://koenig-media.raywenderlich.com/uploads/2018/03/after-property-observer.png){: .center-block :}

# **Adding Pagination**
