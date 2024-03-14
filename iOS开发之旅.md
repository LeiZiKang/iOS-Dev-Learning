# iOS开发之旅

## 循环
```swift
import Foundation

var forLoop = 0
// for
for i in 0..<3 {
    forLoop += i
}
// while
while forLoop > 1 {
    forLoop -= 1
}

// do-while
repeat {
    forLoop *= 2
} while (forLoop <= 2^10)
            
print(forLoop)
print(2^10) // 此处是2进制吗？```
```

swift中函数是一等公民，可以当变量去用

## 闭包

可以理解为没有函数名的函数
```swift
//: [Previous](@previous)

import Foundation

// 闭包就是匿名函数
var oldArray = [1,5,3,2,4]
var array = oldArray.sorted(by: { (a,b) -> Bool in
    return a < b
})
print(array)

let 偶数 = oldArray.filter {
    $0 % 2 == 0
}
print(偶数)

// 例子：算n的阶乘
// 正常的函数法
func facorial(of n: Int64) -> Int64 {
    return (1...n).reduce(1, *)
}
facorial(of: 20)

let yes : (Int) -> Int  = { num in
    return (1...num).reduce(1, *)
}
yes(20)

//: [Next](@next)
```

---

# UIViewControllerRepresentable`

`UIViewControllerRepresentable`是一个SwiftUI协议，它允许你在SwiftUI视图中使用UIKit的`UIViewController`。这是一个桥接SwiftUI和UIKit的重要工具，特别是在某些UIKit功能在SwiftUI中还没有等价实现的情况下。

要使用`UIViewControllerRepresentable`，你需要创建一个遵循该协议的结构体或类，然后实现两个必要的方法：

1. `makeUIViewController(context: Context) -> UIViewControllerType`：这个方法需要返回一个你想要在SwiftUI中使用的`UIViewController`实例。

2. `updateUIViewController(_ uiViewController: UIViewControllerType, context: Context)`：这个方法用于更新你的`UIViewController`实例。当包含`UIViewControllerRepresentable`的SwiftUI视图的状态发生改变时，这个方法会被调用。

此外，你还可以提供一个`Coordinator`类来处理`UIViewController`的代理回调。`Coordinator`类需要遵循`UIViewControllerRepresentableContext<YourUIViewControllerRepresentable>.Coordinator`协议。

这是一个使用`UIViewControllerRepresentable`的基本示例：

```swift
import SwiftUI
import UIKit

struct MyViewControllerRepresentable: UIViewControllerRepresentable {
    typealias UIViewControllerType = UIViewController

    func makeUIViewController(context: Context) -> UIViewController {
        // 创建你的UIViewController
        let viewController = UIViewController()
        return viewController
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        // 更新你的UIViewController
    }
}
```

在这个例子中，我们创建了一个名为`MyViewControllerRepresentable`的结构体，它遵循`UIViewControllerRepresentable`协议，并实现了`makeUIViewController(context:)`和`updateUIViewController(_:context:)`方法。

---
# swift UI实现ImagePicker(可多选)

在 SwiftUI 中，你可以使用 `PHPickerViewController` 来实现多张照片的选择。以下是一个示例：

首先，创建一个遵循 `UIViewControllerRepresentable` 协议的 `ImagePicker` 视图：

```swift
import SwiftUI
import PhotosUI

struct ImagePicker: UIViewControllerRepresentable {
    @Binding var images: [UIImage]
    @Environment(\.presentationMode) var presentationMode

    func makeUIViewController(context: Context) -> PHPickerViewController {
        var configuration = PHPickerConfiguration()
        configuration.selectionLimit = 0 // 0 表示无限制
        configuration.filter = .images
        let picker = PHPickerViewController(configuration: configuration)
        picker.delegate = context.coordinator
        return picker
    }

    func updateUIViewController(_ uiViewController: PHPickerViewController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, PHPickerViewControllerDelegate {
        let parent: ImagePicker

        init(_ parent: ImagePicker) {
            self.parent = parent
        }

        func picker(_ picker: PHPickerViewController, didFinishPicking results: [PHPickerResult]) {
            parent.presentationMode.wrappedValue.dismiss()

            for result in results {
                result.itemProvider.loadObject(ofClass: UIImage.self) { (object, error) in
                    if let image = object as? UIImage {
                        DispatchQueue.main.async {
                            self.parent.images.append(image)
                        }
                    }
                }
            }
        }
    }
}
```

然后，你可以在你的 SwiftUI 视图中使用 `ImagePicker`：

```swift
struct ContentView: View {
    @State private var images: [UIImage] = []
    @State private var isImagePickerPresented = false

    var body: some View {
        VStack {
            ForEach(images, id: \.self) { image in
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            }
            Button("选择照片") {
                isImagePickerPresented = true
            }
        }
        .sheet(isPresented: $isImagePickerPresented) {
            ImagePicker(images: $images)
        }
    }
}
```

在这个示例中，我们创建了一个 `ImagePicker` 视图，并将其绑定到一个 `@State` 变量 `images`。当用户点击 "选择照片" 按钮时，`ImagePicker` 视图将被显示，用户可以选择多张照片。选择的照片将被添加到 `images` 数组中，并在 `ContentView` 中显示。

---
# 将照片等数据编码或解码

你可以使用 `NSKeyedArchiver` 和 `NSKeyedUnarchiver` 来将 `[UIImage]` 转化为 `Data`，以及将 `Data` 转化回 `[UIImage]`。以下是一个示例：

```swift
// 编码器：将 [UIImage] 转化为 Data
func encodeImages(images: [UIImage]) -> Data? {
    return try? NSKeyedArchiver.archivedData(withRootObject: images, requiringSecureCoding: false)
}

// 解码器：将 Data 转化回 [UIImage]
func decodeImages(data: Data) -> [UIImage]? {
    return try? NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(data) as? [UIImage]
}
```

在这个示例中，`encodeImages(images:)` 函数接受一个 `[UIImage]` 参数，然后使用 `NSKeyedArchiver.archivedData(withRootObject:requiringSecureCoding:)` 方法将其转化为 `Data`。如果转化成功，函数返回转化后的 `Data`；如果转化失败，函数返回 `nil`。

`decodeImages(data:)` 函数接受一个 `Data` 参数，然后使用 `NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(_:)` 方法将其转化回 `[UIImage]`。如果转化成功，函数返回转化后的 `[UIImage]`；如果转化失败，函数返回 `nil`。

请注意，这个示例假设你的 `[UIImage]` 可以成功地通过 `NSKeyedArchiver` 和 `NSKeyedUnarchiver` 进行编码和解码。如果你的 `[UIImage]` 包含不能被 `NSKeyedArchiver` 或 `NSKeyedUnarchiver` 编码或解码的对象，那么你可能需要自定义你的编码和解码逻辑。

**解释：**
`NSKeyedUnarchiver` 是一个类，它是 Foundation 框架中的一部分，用于从归档对象（也就是已经被编码为 `Data` 类型的对象）中解码和恢复原始的对象。

归档和解档是一种序列化和反序列化的过程，它们使得你可以将对象转换为二进制数据，然后再从二进制数据中恢复对象。这对于数据的存储和传输非常有用。

例如，你可以使用 `NSKeyedArchiver` 将对象编码（归档）为 `Data`，然后将这个 `Data` 存储到磁盘上，或者通过网络发送给其他设备。然后，你可以使用 `NSKeyedUnarchiver` 将这个 `Data` 解码（解档）回原始的对象。

`NSKeyedUnarchiver` 的使用通常与 `NSKeyedArchiver` 配合，因为 `NSKeyedArchiver` 是用于编码对象的，而 `NSKeyedUnarchiver` 是用于解码对象的。

---
# Task
在 Swift 中，`Task` 是 Swift 5.5 中引入的新特性，它是并发编程的一部分。`Task` 闭包通常用于异步操作。

以下是一个基本的 `Task` 闭包的使用示例：

```swift
Task {
    do {
        let data = try await fetchDataFromServer()
        process(data)
    } catch {
        print("Failed to fetch data: \(error)")
    }
}
```

在这个例子中，我们创建了一个新的 `Task`，并在其中执行异步操作。`fetchDataFromServer` 是一个异步函数，它可能会抛出错误，所以我们使用 `try await` 来调用它，并处理可能的错误。

注意，`Task` 闭包必须在支持并发的环境中使用，例如在 SwiftUI 的 `onAppear` 方法中，或者在你自己的并发代码中。

---
# do 与try
在 Swift 中，`try` 和 `do` 语句通常一起使用，用于处理可能会抛出错误的代码。他们的主要区别在于他们的使用方式和目的。

`do` 语句用于创建一个新的包含作用域，这个作用域内的代码可能会抛出错误。在 `do` 语句内部，你可以使用 `try` 语句来标记可能会抛出错误的代码。

`try` 语句用于标记可能会抛出错误的代码。在 Swift 中，任何可能抛出错误的函数或方法都必须在调用时使用 `try` 关键字。

以下是一个使用 `do` 和 `try` 的示例：

```swift
do {
    try someFunctionThatCanThrowAnError()
} catch {
    // Handle the error
}
```

在这个例子中，`do` 语句用于标记可能会抛出错误的代码块，`try` 语句用于标记可能会抛出错误的函数调用。如果 `someFunctionThatCanThrowAnError()` 确实抛出了错误，那么控制流会立即转移到 `catch` 块，你可以在这里处理错误。

注意，`try` 还有两个变体：`try?` 和 `try!`。`try?` 用于将结果转换为可选值，如果函数抛出错误，它会返回 `nil`。`try!` 用于在你确定函数不会抛出错误时禁用错误传播，如果函数确实抛出错误，你的程序会崩溃。

## try?
在 Swift 中，`try?` 关键字用于将可能抛出错误的表达式的结果转换为可选值。如果表达式抛出错误，`try?` 表达式的值就是 `nil`。如果表达式成功，`try?` 表达式的值就是一个包含原始返回值的可选值。

这是一个使用 `try?` 的例子：

```swift
let result = try? someFunctionThatCanThrowAnError()
```

在这个例子中，如果 `someFunctionThatCanThrowAnError()` 抛出错误，`result` 的值就是 `nil`。如果 `someFunctionThatCanThrowAnError()` 成功，`result` 的值就是一个包含函数返回值的可选值。

使用 `try?` 可以让你更简洁地处理错误，特别是在你不关心具体的错误类型，只关心操作是否成功的情况下。

---
# await
在 Swift 中，`await` 关键字用于等待一个异步操作完成。`await` 只能在 `async` 函数或方法中使用。

在你的代码中，`saveImagesToPhotoAlbum` 是一个异步函数，这意味着它可能会花费一些时间来完成，例如，它可能需要从网络下载图片，或者将图片保存到照片库。在这种情况下，你需要使用 `await` 关键字来等待操作完成。

以下是如何在另一个异步函数中使用 `await` 来调用 `saveImagesToPhotoAlbum` 的示例：

```swift
func processImages() async {
    let images = [UIImage(named: "image1")!, UIImage(named: "image2")!]
    await saveImagesToPhotoAlbum(images: images)
}
```

在这个例子中，我们在 `processImages` 函数中调用 `saveImagesToPhotoAlbum` 函数，并使用 `await` 关键字等待它完成。注意，因为 `await` 只能在 `async` 函数中使用，所以 `processImages` 函数也必须被声明为 `async`。


---
# do, try , await 与Task

在 Swift 中，`Task`，`try` 和 `do` 是用于处理异步操作和错误处理的关键字。

1. `Task`: 在 Swift 5.5 及更高版本中，`Task` 是并发编程的一部分。它表示一个可以在后台执行的工作单元，这个工作单元可以是异步的，并且可以被取消。`Task` 通常用于执行异步操作，如网络请求或者数据库操作。

```swift
Task {
    do {
        let data = try await fetchDataFromServer()
        process(data)
    } catch {
        print("Failed to fetch data: \(error)")
    }
}
```

2. `try`: 在 Swift 中，`try` 关键字用于标记可能会抛出错误的代码。如果一个函数、方法或者闭包有可能抛出错误，你就必须在调用它的地方前面加上 `try` 关键字。

```swift
do {
    try throwingFunction()
} catch {
    print(error)
}
```

3. `do`: `do` 关键字用于开始一个新的错误处理块。在 `do` 块中的代码，如果有可能抛出错误，就必须使用 `try` 或者 `try?` 或者 `try!` 来调用。如果 `do` 块中的代码抛出了错误，这个错误会被立即传递到相应的 `catch` 块中。

```swift
do {
    try throwingFunction()
} catch {
    print(error)
}
```

在这个例子中，如果 `throwingFunction()` 抛出了错误，这个错误会被 `catch` 块捕获，并打印出来。
 

---
# 多线程和异步
多线程和异步是两个在并发编程中经常被提到的概念，它们之间有一定的关系，但也有区别。

1. 多线程：多线程是指程序中包含两个或多个并行运行的线程，每个线程都在处理不同的任务。在多线程环境中，CPU可以在多个线程之间进行切换，以实现并行处理。多线程可以提高程序的执行效率，但也会带来一些复杂性，比如线程同步和数据竞争等问题。

2. 异步：异步是指程序不必等待某个长时间运行的任务完成，而是在这个任务运行的同时，可以继续执行其他任务。异步操作可以在单线程或多线程环境中进行。在单线程环境中，异步操作通常通过事件循环和回调函数实现。在多线程环境中，异步操作可以通过创建新的线程来实现。

多线程和异步的关系是：多线程是实现异步的一种方式，但并不是唯一的方式。异步操作可以在单线程环境中通过非阻塞 I/O 和事件驱动编程实现，也可以在多线程环境中通过线程创建和管理实现。

---
# 多线程

## 四种多线程的区别

-   
    pthread：运用C语言，是一套通用的API，可跨平台Unix/Linux/Windows。线程的生命周期由程序员管理。
- NSThread：面向对象，可直接操作线程对象。线程的生命周期由程序员管理。
- GCD：代替NSThread，可以充分利用设备的多核，自动管理线程生命周期。
- NSOperation：底层是GCD，比GCD多了一些方法，更加面向对象，自动管理线程生命周期。

## GCD
在Swift中，你可以使用`DispatchQueue`来在另一个线程上执行函数。这是一个例子：

```swift
DispatchQueue.global().async {
    // 在这里执行你的函数
}
```

在这个例子中，我们使用`DispatchQueue.global().async`来在全局队列的一个后台线程上执行函数。全局队列是并发的，所以你的函数可能会在任何一个可用的线程上执行。

如果你想在主线程上执行函数，你可以使用`DispatchQueue.main.async`：

```swift
DispatchQueue.main.async {
    // 在这里执行你的函数
}
```

在这个例子中，我们使用`DispatchQueue.main.async`来在主线程上执行函数。你应该在主线程上执行所有的UI更新，因为UI更新不是线程安全的。

注意，`async`方法是异步的，所以它不会阻塞当前线程。如果你想要阻塞当前线程，直到函数执行完成，你可以使用`sync`方法。但是，你应该避免在主线程上使用`sync`方法，因为这可能会导致你的应用程序冻结。
在Swift中，你可以使用`DispatchQueue`的`async`方法来设置任务的优先级。你可以通过`qos`参数来设置优先级。这是一个例子：

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // 在这里执行你的函数
}
```

在这个例子中，我们使用`DispatchQueue.global(qos: .userInitiated).async`来在全局队列的一个后台线程上执行函数，并设置了任务的优先级为`.userInitiated`。这意味着这个任务是由用户直接发起的，所以它应该尽快执行。

这是所有可用的优先级：

- `.userInteractive`: 这个优先级用于需要立即发生的任务，如UI更新。
- `.userInitiated`: 这个优先级用于由用户发起并需要立即知道结果的任务。
- `.default`: 这个优先级用于普通的任务。这是默认的优先级。
- `.utility`: 这个优先级用于不需要立即知道结果的任务，如下载文件。
- `.background`: 这个优先级用于用户不直接知道的任务，如索引数据库。

你可以根据你的需求来选择合适的优先级。

在Swift中，你可以使用`DispatchQueue`来创建一个新的线程，并在这个线程上执行异步任务。以下是如何修改你的`saveImagesToPhotoAlbum`函数来使用GCD执行异步任务：

```swift
func saveImagesToPhotoAlbum(images: [UIImage]) {
    DispatchQueue.global(qos: .background).async {
        for image in images  {
            UIImageWriteToSavedPhotosAlbum(image, self, #selector(self.image(_:didFinishSavingWithError:contextInfo:)), nil)
        }
    }
}
```

在这个例子中，`DispatchQueue.global(qos: .background).async`创建了一个新的后台线程，并在这个线程上执行闭包中的代码。`qos: .background`参数表示这个线程的优先级是后台级别，也就是说，它不会阻塞用户界面或者其他更重要的操作。

---


# 静态属性
静态属性是属于一个类的属性，而不是属于这个类的实例的属性。你可以直接通过类名来访问静态属性，而不需要创建类的实例。在Swift中，你可以使用`static`关键字来定义静态属性。

这是一个例子：

```swift
class MyClass {
    static var myStaticProperty = "Hello, world!"
}

print(MyClass.myStaticProperty) // 输出 "Hello, world!"
```

在这个例子中，`myStaticProperty`是一个静态属性。你可以直接通过`MyClass.myStaticProperty`来访问它，而不需要创建`MyClass`的实例。

静态属性在所有的类实例之间是共享的。这意味着，如果你在一个地方更改了静态属性的值，那么在其他地方访问这个静态属性时，你会看到这个更改。

在你的`MainViewModel`中，`shared`是一个静态属性。这意味着，无论你在哪里访问`MainViewModel.shared`，你都会得到同一个`MainViewModel`实例。这就是为什么你可以在一个地方更改`.photos`属性，然后在其他地方看到这个更改的原因。

---
# ios保存照片到相册

在iOS中，你可以使用`UIImageWriteToSavedPhotosAlbum`函数将`UIImage`对象保存到用户的相册。这个函数需要一个`UIImage`对象，一个目标对象，一个完成选择器和一个上下文信息对象。

以下是一个简单的示例，展示了如何将`UIImage`数组中的每个图像保存到相册：

```swift
import Photos

func saveImagesToPhotoAlbum(images: [UIImage]) {
    for image in images {
        UIImageWriteToSavedPhotosAlbum(image, self, #selector(image(_:didFinishSavingWithError:contextInfo:)), nil)
    }
}

@objc func image(_ image: UIImage, didFinishSavingWithError error: Error?, contextInfo: UnsafeRawPointer) {
    if let error = error {
        // 我们在这里得到了一个错误!
        print("Error: \(error.localizedDescription)")
    } else {
        // 图片保存成功
        print("Image saved successfully!")
    }
}
```

请注意，你需要在你的应用的Info.plist文件中添加`NSPhotoLibraryAddUsageDescription`键，并提供一个描述，解释为什么你的应用需要这个权限。

---

# Mutipeer Connectivity 多点连接

`sendResource(at:withName:toPeer:withCompletionHandler:)` 和 `send(_:toPeers:with:)` 是 MultipeerConnectivity 框架中的两种发送数据的方式，它们的主要区别在于发送的数据类型和发送方式。

1. `sendResource(at:withName:toPeer:withCompletionHandler:)`：这个方法用于发送文件资源，例如图片、音频、视频等。这个方法会返回一个 `Progress` 对象，你可以使用这个对象来跟踪发送进度。这个方法是异步的，当资源发送完成时，会调用提供的完成处理器。

2. `send(_:toPeers:with:)`：这个方法用于发送小量的数据，例如文本、数字、小的数据对象等。这个方法不会返回 `Progress` 对象，你不能直接跟踪发送进度。这个方法是同步的，当方法返回时，数据已经被发送。

总的来说，如果你需要发送的是文件资源，并且需要跟踪发送进度，你应该使用 `sendResource(at:withName:toPeer:withCompletionHandler:)` 方法。如果你需要发送的是小量的数据，并且不需要跟踪发送进度，你可以使用 `send(_:toPeers:with:)` 方法。

---
# 文件的URL

选择的文件的 URL 包含以下信息：

1. **Scheme**：URL 的开头部分，表示 URL 的类型。对于文件，这通常是 "file"。

2. **Host**：对于文件 URL，这通常是空的。

3. **Path**：文件在文件系统中的路径。这包括目录和文件名。

4. **Query**：URL 的可选部分，用于提供额外的参数。对于文件 URL，这通常是空的。

5. **Fragment**：URL 的可选部分，用于指定资源的特定部分。对于文件 URL，这通常是空的。

例如，一个文件 URL 可能看起来像这样：`file:///Users/username/Documents/myfile.txt`。在这个例子中，"file" 是 scheme，"/Users/username/Documents/myfile.txt" 是 path。

你可以使用 `URL` 类的属性来访问这些信息。例如，`url.scheme`、`url.host`、`url.path`、`url.query` 和 `url.fragment`。

注意，由于沙盒限制，你的应用只能访问用户通过 `UIDocumentPickerViewController` 明确选择的文件，或者你的应用的沙盒目录中的文件。你不能访问其他目录中的文件。

## 更具URL获取文件大小
如果你的 URL 是从文件系统获取的，而不是通过 `FileManager`，你可以使用 `URL` 的 `resourceValues(forKeys:)` 方法来获取文件的大小。这个方法返回一个 `URLResourceValues` 对象，其中包含了文件的各种属性，包括其大小。

以下是一个函数，它接受一个 `URL` 参数，返回该 URL 指向的文件的大小：

```swift
import Foundation

func getFileSize(url: URL) -> Int64? {
    do {
        let resourceValues = try url.resourceValues(forKeys: [.fileSizeKey])
        return resourceValues.fileSize
    } catch {
        print("Failed to get file size: \(error)")
        return nil
    }
}
```

在这个函数中，我们首先使用 `resourceValues(forKeys:)` 方法获取 URL 的资源值。然后，我们从资源值中获取文件大小，并返回它。如果获取资源值或文件大小失败，我们打印一个错误消息，并返回 `nil`。

你可以在你的 `HStack` 中使用这个函数来显示文件的大小，如下所示：

```swift
HStack{
    Text("\(url.pathExtension)") // 文件类型
    Text("\(url.lastPathComponent)")// 文件名称
    if let fileSize = getFileSize(url: url) {
        Text("Size: \(fileSize) bytes")
    }
}
```

在 Swift 中，你可以使用 `FileManager` 的 `attributesOfItem(atPath:)` 方法来获取文件的大小。这个方法返回一个字典，其中包含了文件的各种属性，包括其大小。

以下是一个函数，它接受一个 `URL` 参数，返回该 URL 指向的文件的大小：

```swift
import Foundation

func getFileSize(url: URL) -> UInt64? {
    let filePath = url.path
    let fileManager = FileManager.default

    do {
        let attributes = try fileManager.attributesOfItem(atPath: filePath)
        if let fileSize = attributes[.size] as? UInt64 {
            return fileSize
        } else {
            print("Failed to get file size")
            return nil
        }
    } catch {
        print("Failed to get file attributes: \(error.localizedDescription)")
        return nil
    }
}
```

在这个函数中，我们首先获取 URL 的路径，然后使用 `FileManager` 获取该路径的文件属性。然后，我们从属性字典中获取文件大小，并返回它。如果获取文件属性或文件大小失败，我们打印一个错误消息，并返回 `nil`。

你可以在你的 `HStack` 中使用这个函数来显示文件的大小，如下所示：

```swift
HStack{
    Text("\(url.pathExtension)") // 文件类型
    Text("\(url.lastPathComponent)")// 文件名称
    if let fileSize = getFileSize(url: url) {
        Text("Size: \(fileSize) bytes")
    }
}
```


---
# 预览文件的API
在SwiftUI中，你可以使用`QLPreviewController`来预览文档，包括PDF。但是，`QLPreviewController`是UIKit的一部分，所以你需要创建一个`UIViewControllerRepresentable`的实现来在SwiftUI中使用它。

以下是一个示例，展示了如何创建一个可以预览PDF的`QLPreviewController`：

```swift
import SwiftUI
import QuickLook

struct DocumentPreview: UIViewControllerRepresentable {
    var url: URL

    func makeUIViewController(context: Context) -> QLPreviewController {
        let controller = QLPreviewController()
        controller.dataSource = context.coordinator
        return controller
    }

    func updateUIViewController(_ uiViewController: QLPreviewController, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, QLPreviewControllerDataSource {
        var parent: DocumentPreview

        init(_ parent: DocumentPreview) {
            self.parent = parent
        }

        func numberOfPreviewItems(in controller: QLPreviewController) -> Int {
            return 1
        }

        func previewController(_ controller: QLPreviewController, previewItemAt index: Int) -> QLPreviewItem {
            return parent.url as QLPreviewItem
        }
    }
}
```

你可以像这样使用`DocumentPreview`：

```swift
DocumentPreview(url: URL(fileURLWithPath: "/path/to/your/document.pdf"))
```

请注意，你需要将`/path/to/your/document.pdf`替换为你的文档的实际路径。

---
# App显示名字

bundle Display Name

---
# UIkit与swiftUI的区别

1. 最大的区别就是UIKit使用class，swiftUI使用struct

在 UIKit 中，所有的视图都继承自一个叫 `UIView` 的类，它有非常多的属性和方法 —— 背景颜色，布局约束，用于渲染的层，等等。还有更多诸如此类的属性，而每一个 `UIView` 和 `UIView` 的子类都有，因为这正是继承的工作方式。

涉及一个性能原理：结构体比类更简单，更轻量。之所以第一个说这个原因，是因为大多数都认为这是 SwiftUI 采用结构体的主要原因。其实，纵观全局，这只是原因之一。

2. 用 struct 表示 view 还有其他重要原因：它强迫我们以一种更干净的方式隔离状态。类可以自由地修改它的值 —— 这可能导致更凌乱的代码，这样的话 SwiftUI 就无法通过某个值的变化来自动更新 UI 了。

用 struct 表示 view 还有其他重要原因：它强迫我们以一种更干净的方式隔离状态。类可以自由地修改它的值 —— 这可能导致更凌乱的代码，这样的话 SwiftUI 就无法通过某个值的变化来自动更新 UI 了。

---
# CocoaPods和SPM的区别

1. CocoaPods是中心化的，SPM是去中心化的
2. CocoaPods的项目入侵比较严重

- [CocoaPods](https://blog.csdn.net/u014600626/article/details/102922568)、[Carthage](https://www.jianshu.com/p/42118918177b)、[SPM](https://zhuanlan.zhihu.com/p/103197303)对比

![[f7dd8aca06b241698df35f4d43b2b997.png]]

主要区别：
1. 原理
2. 项目入侵性，源码是否可见


---
# iOS使用FastLane自动打包
https://blog.csdn.net/long4512524/article/details/129733908


---
# 渐变色

在SwiftUI中，`LinearGradient`，`RadialGradient`和`AngularGradient`都是用来创建渐变色的。

1. `LinearGradient`：线性渐变，颜色沿直线方向逐渐改变。它有两个关键参数：`gradient`和`startPoint`，`endPoint`。`gradient`定义了渐变的颜色和位置，`startPoint`和`endPoint`定义了渐变的方向。

```swift
LinearGradient(gradient: Gradient(colors: [.red, .blue]), startPoint: .top, endPoint: .bottom)
```

2. `RadialGradient`：径向渐变，颜色从一个中心点向外逐渐改变。它有三个关键参数：`gradient`，`center`和`startRadius`，`endRadius`。`gradient`定义了渐变的颜色和位置，`center`定义了渐变的中心点，`startRadius`和`endRadius`定义了渐变的半径范围。

```swift
RadialGradient(gradient: Gradient(colors: [.red, .blue]), center: .center, startRadius: 0, endRadius: 200)
```

3. `AngularGradient`：角度渐变，颜色沿圆周方向逐渐改变。它有两个关键参数：`gradient`和`center`。`gradient`定义了渐变的颜色和位置，`center`定义了渐变的中心点。

```swift
AngularGradient(gradient: Gradient(colors: [.red, .blue]), center: .center)
```

	以上都是创建渐变色的基本方法，你可以根据需要调整参数来创建不同的渐变效果。


---

# ScrollerViewReader

你的代码中的问题在于，你试图在 `ScrollViewReader` 的 `content` 闭包中使用 `ScrollView`。然而，`ScrollViewReader` 的目的是允许你控制其内部的 `ScrollView` 的滚动位置，所以你应该将 `ScrollView` 放在 `ScrollViewReader` 的外部。

此外，你需要为你想要滚动到的视图设置一个 ID，然后你可以使用这个 ID 来滚动到这个视图。以下是修复后的代码：

```swift
ScrollView(.vertical, showsIndicators: false) {
    ScrollViewReader { proxy in
        VStack {
            // 系统信息模块
            // ...

            // 已接收文件模块
            // ...

            // 连接模块
            VStack {
                // ...
            }
            .id(bottomID) // 设置 ID
            .padding(.bottom, 20)
            .padding(.top, 20)
        }
        .onAppear(perform: {
            proxy.scrollTo(bottomID, anchor: .bottom) // 滚动到指定的 ID
        })
    }
}
```

在这个修复后的版本中，我们将 `ScrollView` 移动到了 `ScrollViewReader` 的外部，并为连接模块设置了一个 ID。然后，在 `onAppear` 方法中，我们使用 `proxy.scrollTo(_:anchor:)` 方法来滚动到连接模块。

---
# UIImage保存到文件app（HEIC形式）
在 iOS 11 及更高版本中，你可以使用 `AVAssetWriter` 和 `AVAssetWriterInputPixelBufferAdaptor` 将 `UIImage` 数组保存为 HEIC 格式的文件。以下是一个示例：

```swift
import AVFoundation
import UIKit

func saveImages(_ images: [UIImage], toFile file: String) {
    let fileURL = URL(fileURLWithPath: file)
    let assetWriter = try! AVAssetWriter(outputURL: fileURL, fileType: AVFileType.heic)
    let outputSettings: [String: Any] = [
        AVVideoCodecKey: AVVideoCodecType.heic,
        AVVideoWidthKey: NSNumber(value: Float(images[0].size.width)),
        AVVideoHeightKey: NSNumber(value: Float(images[0].size.height))
    ]
    let assetWriterInput = AVAssetWriterInput(mediaType: .video, outputSettings: outputSettings)
    let adaptor = AVAssetWriterInputPixelBufferAdaptor(assetWriterInput: assetWriterInput, sourcePixelBufferAttributes: nil)
    assetWriter.add(assetWriterInput)

    assetWriter.startWriting()
    assetWriter.startSession(atSourceTime: CMTime.zero)

    let context = CIContext()
    for (i, image) in images.enumerated() {
        let buffer = context.createPixelBuffer(for: image.cgImage!)!
        adaptor.append(buffer, withPresentationTime: CMTime(value: Int64(i), timescale: 1))
    }

    assetWriterInput.markAsFinished()
    assetWriter.finishWriting {}
}
```

在这个示例中，我们首先创建了一个 `AVAssetWriter`，并设置了输出文件的类型为 HEIC。然后，我们创建了一个 `AVAssetWriterInput`，并设置了输出的宽度和高度。接着，我们创建了一个 `AVAssetWriterInputPixelBufferAdaptor`，并将 `AVAssetWriterInput` 添加到 `AVAssetWriter` 中。

然后，我们开始写入会话，并遍历图像数组。对于每个图像，我们创建一个像素缓冲区，并将其添加到 `AVAssetWriterInputPixelBufferAdaptor` 中。

最后，我们标记 `AVAssetWriterInput` 为已完成，并结束写入。

请注意，这个函数将图像保存为一个 HEIC 文件，而不是多个文件。每个图像都被保存为文件中的一个帧。

---
# mvvm和mvc区别

MVVM和MVC的区别主要在于==**数据绑定和通信方式的不同**==，MVVM是双向数据绑定，而MVC是单向数据绑定。1234567

MVVM模式中，视图(View)和模型(Model)之间通过视图模型(ViewModel)进行交互，视图模型负责将数据绑定到视图上，并处理视图相关的逻辑，同时，视图模型也能监听视图的变化，并更新数据，这种双向数据绑定的特性使得数据和视图之间的同步更加简单和高效。

MVC模式中，视图和模型之间通过控制器(Controller)进行交互，控制器负责协调模型和视图之间的逻辑和流程，在MVC中，视图的变化会直接影响模型，而在MVVM中，视图的变化不会直接影响到模型，这种设计思想更加符合单向数据流的思想。

总的来说，MVVM模式在数据绑定、视图与模型之间的解耦、开发效率和用户体验方面优于MVC模式。

---

# 实例化对象

实例化一个对象在内存中干了三件事
1:开辟存储空间
2:初始化成员变量
3:返回指针地址


---
## CGRectMake
`CGRectMake`是一个在Objective-C中用于创建CGRect结构的函数。CGRect是一个结构，用于描述一个矩形的位置和尺寸。

`CGRectMake`函数接受四个参数：x, y, width, height。其中，x和y定义了矩形左上角的位置，width和height定义了矩形的宽度和高度。

这是一个例子：

```objective-c
CGRect rect = CGRectMake(0, 0, 100, 100);
```

在这个例子中，我们创建了一个新的CGRect，其左上角位于(0,0)，宽度和高度都是100。

---

# swiftUI的视图


