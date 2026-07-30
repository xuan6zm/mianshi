# mianshi
面试呀

# swift面试题

<details>
<summary>1. Swift 闭包是值类型还是引用类型？为什么？</summary>

### ✅ 答案

闭包是**引用类型**，存储在**堆（Heap）**中。

### 📌 原因

- 闭包会捕获（Capture）外部变量或对象。
- 为了保存这些上下文，系统会将闭包分配到堆内存。
- 闭包在赋值、传递时，共享同一块内存，而不是复制整个闭包。
- 因此具有引用类型的特征。
- 底层实现与 Objective-C 的 Block 十分相似。</details>


<details>
<summary>2. 什么是非逃逸闭包？有什么特点？</summary>

### ✅ 定义

1.**非逃逸闭包（Non-Escaping Closure）** 是指闭包**不会逃离当前函数作用域**，会在函数返回之前执行完成，生命周期不会超过当前函数。
2.@escaping 作用：编译器强制标记，做风险提醒。
告知开发者：这个闭包生命周期不受函数管控，会长期持有捕获对象，极易产生循环引用，需要手动处理内存。

### ✅ 特点

- 不需要使用 `@escaping` 标记（默认就是非逃逸）。
- 闭包不会被保存到外部，因此**不会产生循环引用**，通常也**无需使用 `[weak self]`**。
- 编译器可以进行更多优化，性能优于逃逸闭包。
- **不能**赋值给成员变量或全局变量。
- **不能**在异步任务（如 `DispatchQueue.async`）中执行。</details>

<details>
<summary>3.什么是逃逸闭包，为什么要用 @escaping 标记？</summary>

### ✅ 定义

**逃逸闭包（Escaping Closure）** 是指**函数执行结束后，闭包仍然可能继续执行**。

通常有两种情况：

- 闭包被保存到成员变量或全局变量。
- 闭包放入异步任务（如 `DispatchQueue.async`）延迟执行。

因此，闭包的生命周期超过了当前函数作用域，所以称为**逃逸闭包**。

### ✅ 为什么需要 `@escaping`

`@escaping` 是编译器要求的标记，用于告诉编译器：

- 该闭包可能在函数返回后继续执行。
- 生命周期不再受当前函数管理。
- 闭包可能长期持有其捕获的对象。
- 开发者需要关注内存管理，避免循环引用。

### ✅ 如何手动处理内存

逃逸闭包中，如果需要访问 `self`，通常使用**捕获列表（Capture List）**：

#### 方式一：`[weak self]`（最常用）

```swift
class ViewController {

    func loadData() {
        Network.request { [weak self] in
            self?.updateUI()
        }
    }
}
```

特点：

- `self` 为弱引用。
- 对象释放后，`self` 自动变为 `nil`。
- 不会产生循环引用。
- UIKit 开发中最常用。

---

#### 方式二：`[unowned self]`

```swift
Network.request { [unowned self] in
    updateUI()
}
```

特点：

- 不增加引用计数。
- 不需要可选绑定。
- 如果 `self` 已释放，再访问会直接崩溃。
- 适用于确定 `self` 生命周期一定比闭包长的场景。

### 💡 面试回答（40 秒）

> 逃逸闭包是指函数返回后仍可能执行的闭包，例如异步回调、网络请求完成回调等。由于闭包生命周期超过函数，因此需要使用 `@escaping` 标记，提醒编译器和开发者该闭包会长期存在。如果闭包内部强引用 `self`，容易形成循环引用，所以通常使用 `[weak self]` 或 `[unowned self]` 来打破引用环，其中 UIKit 开发中最常用的是 `[weak self]`。

### ⭐ 关键字

`@escaping` `异步回调` `生命周期` `Capture List` `weak self` `unowned self` `循环引用`

### 为什么非逃逸闭包不用写 [weak self]，逃逸闭包经常要写？

因为非逃逸闭包会在函数返回前执行完成，不会被长期持有，即使闭包捕获了 self，闭包执行结束就会释放，一般不会形成循环引用。而逃逸闭包会被长期保存，如果闭包强引用 self，同时 self 又持有闭包，就会形成循环引用，因此通常使用 [weak self] 或 [unowned self] 打破引用环。

### 拓展问法，如果使用escaping标记了，还需要写weak self吗？
> 不一定。不是所有 @escaping 闭包都必须写 [weak self]。

@escaping 只是说明闭包可能在函数返回后继续执行，而 [weak self] 是为了避免闭包强引用 self 导致循环引用。只有当闭包捕获了 self，并且存在形成引用环的风险时，才需要使用 [weak self] 或 [unowned self]。


</details>

<details>
<summary>5. [weak self] 和 [unowned self] 有什么区别？分别适用于什么场景？</summary>

### ✅ 相同点

- 都属于**捕获列表（Capture List）**。
- 都不会增加 `self` 的引用计数。
- 都用于避免闭包与 `self` 之间形成循环引用。

### ✅ 区别

#### `[weak self]`

特点：

- `self` 为**弱引用**，类型变为 `Self?`。
- 对象释放后，`self` 会自动变为 `nil`。
- 使用时需要可选绑定（`self?` 或 `guard let self`）。
- 访问安全，不会崩溃。

适用场景：

- 网络请求回调
- GCD 异步任务
- Timer
- Notification
- 大多数 UIKit 开发场景

示例：

```swift
Network.request { [weak self] in
    self?.updateUI()
}
```

---

#### `[unowned self]`

特点：

- `self` 为**无主引用**，不是可选类型。
- 不需要使用 `?` 解包。
- 对象释放后不会自动置为 `nil`。
- 如果 `self` 已释放，再访问会直接崩溃（野指针）。

适用场景：

- 可以**100% 确定**闭包执行时 `self` 一定存在。
- 生命周期完全一致的对象之间。
- 一些同步执行或内部回调场景。

示例：

```swift
animation.completion = { [unowned self] in
    updateUI()
}
```

### ✅ 如何选择

- **不能确定 `self` 是否仍然存在 → 使用 `[weak self]`。**
- **能够确定 `self` 生命周期一定比闭包长 → 可以使用 `[unowned self]`。**

实际开发中，**绝大多数情况下优先使用 `[weak self]`**。

### 💡 面试回答（40 秒）

> `[weak self]` 和 `[unowned self]` 都用于避免闭包循环引用，不会增加 `self` 的引用计数。区别在于，`weak` 会将 `self` 变为可选类型，对象释放后自动变为 `nil`，访问安全，因此适用于绝大多数异步回调；而 `unowned` 不会变成可选，也不会自动置空，如果对象提前释放，再访问会直接崩溃，所以只有在能够确定 `self` 生命周期一定长于闭包时才使用。

### ⭐ 关键字

`weak` `unowned` `Capture List` `循环引用` `弱引用` `无主引用` `可选类型`

</details>

<details>
<summary>6. 什么是闭包（Closure）？</summary>

### ✅ 定义

**闭包（Closure）** 是一段**可以被传递、保存和执行的代码块**。

它不仅包含代码，还可以**捕获（Capture）**定义时所在作用域中的变量和常量，因此称为"闭包"。

Swift 中，函数（Function）其实也是一种特殊的闭包。

---

### ✅ 闭包的特点

- 可以像变量一样赋值、传递、返回。
- 可以作为函数参数。
- 可以作为函数返回值。
- 可以捕获外部变量（Capture）。
- 属于引用类型，存储在堆（Heap）中。

---

### ✅ 示例

```swift
let greet = {
    print("Hello Swift")
}

greet()
```

带参数和返回值：

```swift
let sum: (Int, Int) -> Int = { a, b in
    return a + b
}

print(sum(3, 5)) // 8
```

---

### ✅ 为什么叫"闭包"

```swift
func makeCounter() -> () -> Int {
    var count = 0

    return {
        count += 1
        return count
    }
}
```

这里虽然 `makeCounter()` 已经执行结束，但返回的闭包仍然能够访问 `count`。

这是因为闭包**捕获并保存了外部变量**，形成了自己的上下文（Context），所以称为**闭包（Closure）**。

---

### 💡 面试回答（30 秒）

> 闭包是一段可以被传递、保存和执行的代码块。与普通函数不同的是，闭包可以捕获定义时所在作用域中的变量和常量，并在之后继续使用这些数据。Swift 中函数本质上也是一种特殊的闭包，闭包属于引用类型，广泛用于回调、高阶函数和异步编程。

### ⭐ 关键字

`代码块` `Capture` `上下文` `引用类型` `回调` `高阶函数`

</details>

<details>
  
<summary>7. 下面代码会不会内存泄漏？为什么？</summary>

### ✅ 题目

```swift
class VC {

    var block: (() -> Void)?

    let name = "测试"

    func test() {
        block = {
            print(self.name)
        }
    }
}
```

### ✅ 答案

**会发生内存泄漏（循环引用）。**

### ✅ 原因

这里形成了一个**强引用环（Retain Cycle）**：

```text
VC
 │
 │ 强引用
 ▼
block（成员变量）
 │
 │ 强引用
 ▼
Closure（闭包）
 │
 │ 强引用
 ▼
self（VC）
```

引用关系如下：

1. `VC` 强引用成员变量 `block`。
2. `block` 强引用闭包对象。
3. 闭包内部直接使用 `self`，默认会**强捕获（Strong Capture）** `self`。
4. `self` 又指向 `VC` 实例。

因此形成：

```text
VC → block → Closure → VC
```

互相持有，引用计数永远不会变为 0，所以对象无法释放，造成内存泄漏。

---

### ✅ 修复方式

使用**捕获列表（Capture List）**弱引用 `self`：

```swift
block = { [weak self] in
    print(self?.name)
}
```

这样闭包不会增加 `self` 的引用计数，当 `VC` 释放时，`self` 会自动变为 `nil`，循环引用被打破。

---

### 💡 面试回答（40 秒）

> 这段代码会发生循环引用。因为 `VC` 强引用成员变量 `block`，而 `block` 持有的闭包又默认强引用 `self`，形成 `VC → block → Closure → VC` 的强引用环，导致对象无法释放。解决方法是在闭包的捕获列表中使用 `[weak self]` 或 `[unowned self]`，通常推荐使用 `[weak self]`。

### ⭐ 面试官追问

**Q：为什么这里一定会泄漏，而 `DispatchQueue.async` 不一定会泄漏？**

**答：**

因为这里的闭包被保存到了 `VC` 的成员变量 `block` 中，`VC` 会一直持有这个闭包；而 `DispatchQueue.async` 的闭包是由 GCD 临时持有，任务执行完成后就会释放。即使闭包内部强引用了 `self`，通常也不会形成长期的双向持有关系，因此不一定会产生循环引用。

### ⭐ 关键字

`循环引用` `Retain Cycle` `Strong Capture` `Capture List` `weak self`

</details>

<details>

  
<summary>8. 多层嵌套逃逸闭包，只在最外层写 [weak self] 够用吗？</summary>

### ✅ 答案

**不一定。**

每个闭包都有自己独立的**捕获列表（Capture List）**。

如果内层逃逸闭包也直接使用 `self`，那么它会重新捕获 `self`，需要单独考虑是否使用 `[weak self]`。

因此，**每一个会捕获 `self` 的逃逸闭包，都应该单独处理。**

---

### ✅ 示例

```swift
api.request { [weak self] _ in

    guard let self else { return }

    self.subRequest { [weak self] _ in
        self?.updateUI()
    }
}
```

这里：

- 第一层闭包使用了 `[weak self]`。
- 第二层闭包仍然是新的逃逸闭包。
- 第二层如果访问 `self`，也需要使用自己的捕获列表。

---

### ✅ 为什么？

因为：

- 一个闭包对应一个捕获列表（Capture List）。
- 外层 `[weak self]` 的作用范围仅限于当前闭包。
- 内层闭包会重新分析自己使用到的变量，并重新决定如何捕获。

所以外层的 `[weak self]` **不会自动作用于内层闭包**。

---

### 💡 面试回答（40 秒）

> 不一定。每个闭包都有独立的捕获列表，外层闭包的 `[weak self]` 不会自动传递给内层闭包。如果内层逃逸闭包也需要访问 `self`，它会重新捕获 `self`，因此需要在该闭包中单独考虑是否使用 `[weak self]`，避免形成循环引用。

### ⭐ 关键字

`Capture List` `独立闭包` `weak self` `逃逸闭包` `循环引用`

</details>

<details>
<summary>9. 把非逃逸闭包赋值给类属性，为什么会编译报错？</summary>

### ✅ 答案

因为**赋值给成员变量意味着闭包会在函数返回后继续存在**，生命周期已经超出了当前函数作用域，因此属于**逃逸闭包（Escaping Closure）**。

而函数参数默认都是**非逃逸闭包**，编译器禁止将非逃逸闭包保存到外部，所以会直接报错。

如果确实需要保存闭包，就必须使用 `@escaping` 标记。

---

### ✅ 示例（错误）

```swift
class Test {

    var block: (() -> Void)?

    func setBlock(_ completion: () -> Void) {
        block = completion   // ❌ 编译报错
    }
}
```

报错原因：

> Escaping closure captures non-escaping parameter

---

### ✅ 正确写法

```swift
class Test {

    var block: (() -> Void)?

    func setBlock(_ completion: @escaping () -> Void) {
        block = completion
    }
}
```

这里使用 `@escaping` 告诉编译器：

> **这个闭包会在函数返回后继续存在，请按逃逸闭包处理。**

---

### ✅ 为什么编译器知道它逃逸了？

因为下面这些行为都会让闭包生命周期超过函数：

- 保存到成员变量（`self.block = completion`）
- 保存到全局变量
- 保存到数组、字典等容器
- 作为返回值返回
- 放到异步任务（`DispatchQueue.async`）
- 网络请求完成回调

只要出现这些情况，编译器都会认为闭包**逃逸**，因此要求使用 `@escaping`。

---

### 💡 面试回答（30 秒）

> Swift 中函数参数默认都是非逃逸闭包，只能在当前函数内同步执行。如果将闭包赋值给成员变量，意味着闭包会在函数返回后继续存在，生命周期已经超出函数作用域，因此变成了逃逸闭包。为了提醒开发者关注闭包生命周期和内存管理，编译器要求必须使用 `@escaping` 标记，否则会直接报错。

### ⭐ 关键字

`非逃逸闭包` `逃逸闭包` `@escaping` `成员变量` `生命周期`

</details>

<details>
<summary>9. 逃逸闭包在子线程回调，直接操作 UI 会发生什么？怎么解决？</summary>

### ✅ 问题

如果逃逸闭包在子线程执行，并且直接操作 UIKit UI，会导致：

- UI 更新不稳定。
- 出现界面刷新异常。
- 可能出现偶现崩溃（尤其是多线程竞争时）。
- 数据状态和界面状态不一致。

原因：

**UIKit 不是线程安全的，所有 UI 操作必须在主线程执行。**

---

### ✅ 错误示例

```swift
request { result in

    self.label.text = "完成"   // ❌ 可能在子线程更新 UI

}
```

如果 `request` 的回调是在后台线程执行：

```text
子线程
  ↓
修改 UILabel
  ↓
UIKit 非线程安全
  ↓
异常行为
```

---

### ✅ 解决方式

在回调内部切换到主线程：

```swift
request { [weak self] result in

    DispatchQueue.main.async {
        self?.refreshUI()
    }

}
```

执行流程：

```text
网络线程
    ↓
请求完成回调
    ↓
DispatchQueue.main.async
    ↓
主线程
    ↓
更新 UI
```

---

### ✅ Swift Concurrency 写法

如果使用 Swift 并发：

```swift
Task { @MainActor in
    self.refreshUI()
}
```

或者：

```swift
await MainActor.run {
    self.refreshUI()
}
```

`MainActor` 可以保证代码运行在主线程环境。

---

### 💡 面试回答（40 秒）

> 逃逸闭包通常用于异步回调，例如网络请求完成回调。如果回调是在子线程执行，直接操作 UIKit 会违反 UIKit 的线程安全要求，可能导致界面异常甚至崩溃。解决方式是在回调内部切换到主线程，例如使用 `DispatchQueue.main.async` 更新 UI。在 Swift Concurrency 中，也可以使用 `@MainActor` 保证 UI 操作始终在主线程执行。

### ⭐ 关键字

`UIKit` `主线程` `DispatchQueue.main` `MainActor` `线程安全` `异步回调`

</details>


<details>
<summary>10. OC Block 和 Swift 逃逸闭包有什么对应关系？</summary>

### ✅ 整体关系

Objective-C 的 Block 和 Swift Closure 都可以：

- 保存一段代码。
- 捕获外部变量。
- 作为参数传递。
- 作为回调使用。

两者在**生命周期管理和循环引用问题上非常相似**。

---

### ✅ OC 栈 Block（Stack Block）

特点：

- 默认创建在栈（Stack）上。
- 生命周期较短。
- 只能在当前作用域内执行。
- 函数结束后可能销毁。

类似 Swift：

```swift
非逃逸闭包（Non-Escaping Closure）
```

示例：

```objc
[self execute:^{
    NSLog(@"同步执行");
}];
```

---

### ✅ OC 堆 Block（Heap Block）

特点：

- 当 Block 需要长期保存时，会从栈复制到堆（Heap）。
- 生命周期超过当前函数。
- 可以被对象属性、变量长期持有。
- 常见于异步回调。

类似 Swift：

```swift
@escaping 逃逸闭包（Escaping Closure）
```

例如：

```objc
self.completion = ^{
    NSLog(@"异步执行");
};
```

对应 Swift：

```swift
var completion: (() -> Void)?

func request(completion: @escaping () -> Void) {
    self.completion = completion
}
```

---

### ✅ 捕获 self 的循环引用

OC Block：

```objc
self.block = ^{
    [self updateUI];
};
```

引用关系：

```text
self
 ↓
block
 ↓
self
```

形成循环引用。

解决：

```objc
__weak typeof(self) weakSelf = self;

self.block = ^{
    [weakSelf updateUI];
};
```

---

Swift：

```swift
self.block = { [weak self] in
    self?.updateUI()
}
```

通过弱引用打破循环引用。

---

### 💡 面试回答（40 秒）

> OC Block 和 Swift Closure 在内存管理上有很多相似之处。OC 中栈 Block 生命周期较短，类似 Swift 的非逃逸闭包；当 Block 被复制到堆上长期保存，例如异步回调或赋值给属性时，类似 Swift 的 `@escaping` 逃逸闭包。两者都会捕获外部对象，如果强引用 `self` 都可能产生循环引用，OC 使用 `__weak`，Swift 使用 `[weak self]` 解决。

### ⭐ 关键字

`Block` `Closure` `Stack` `Heap` `@escaping` `__weak` `weak self` `循环引用`

</details>

<details>
<summary>11. 函数参数同时存在逃逸 + 非逃逸闭包合法吗？</summary>

### ✅ 答案

**合法。**

一个函数可以同时接收多个闭包参数：

- 同步执行的闭包：默认非逃逸，不需要 `@escaping`。
- 异步执行或需要保存的闭包：需要添加 `@escaping`。

---

### ✅ 示例

```swift
class Manager {

    var completion: (() -> Void)?

    func execute(
        syncBlock: () -> Void,
        asyncBlock: @escaping () -> Void
    ) {

        // 非逃逸闭包
        syncBlock()

        // 逃逸闭包
        completion = asyncBlock
    }
}
```

这里：

`syncBlock`

- 当前函数内立即执行。
- 函数结束后销毁。
- 不需要 `@escaping`。


`asyncBlock`

- 被保存到属性。
- 生命周期超过当前函数。
- 必须使用 `@escaping`。

---

## ✅ 代码判断题

---

## 题 1：分析是否泄漏

### 代码

```swift
func syncPrint(cb: () -> Void) {
    cb()
}

func run() {
    syncPrint {
        print(self.title)
    }
}
```

### 答案

**不会发生内存泄漏。**

原因：

- `cb` 是非逃逸闭包。
- 闭包只在 `syncPrint` 函数内部执行。
- 函数返回后闭包生命周期结束。
- 不会被长期保存。

执行过程：

```text
run()
 ↓
syncPrint()
 ↓
执行闭包
 ↓
函数结束
 ↓
闭包释放
```

因此不会形成：

```text
self → closure → self
```

这种长期循环引用。

---

## 题 2：分析是否泄漏

### 代码

```swift
var globalCb: (() -> Void)?

func saveCb(cb: @escaping () -> Void) {
    globalCb = cb
}

func run() {
    saveCb {
        print(self.title)
    }
}
```

### 答案

**会发生内存泄漏。**

原因：

1. `cb` 使用 `@escaping`，说明闭包可以逃逸。
2. 闭包被保存到全局变量 `globalCb`。
3. 闭包内部强捕获 `self`。

引用关系：

```text
globalCb
   ↓
Closure
   ↓
self
```

如果 `self` 同时持有该闭包：

```text
self
 ↓
globalCb / property
 ↓
Closure
 ↓
self
```

会形成循环引用。

---

### ✅ 修复方式

使用弱引用：

```swift
saveCb { [weak self] in
    print(self?.title)
}
```

---

### 💡 面试回答（50 秒）

> 一个函数可以同时存在逃逸和非逃逸闭包参数。非逃逸闭包默认同步执行，生命周期不会超过函数，所以不需要 `@escaping`；而逃逸闭包会在函数返回后继续存在，例如保存到属性、全局变量或异步执行，因此需要 `@escaping`。判断闭包是否造成内存泄漏，关键看闭包是否长期被持有，以及是否强引用了 `self`。如果逃逸闭包被长期保存，同时捕获 `self`，就需要使用 `[weak self]` 避免循环引用。

### ⭐ 关键字

`@escaping` `Non-Escaping` `生命周期` `Capture` `循环引用` `weak self`

</details>

<details>
<summary>12. 实际开发中遇到过哪些内存泄漏问题？都是怎么解决的？</summary>

### ✅ 面试回答

实际开发中遇到的内存泄漏主要集中在以下几类：

---

## 1. 闭包强引用 self 导致循环引用

### 问题场景

网络请求、异步回调中：

```swift
class ViewController {

    func loadData() {

        request {

            self.updateUI()

        }
    }
}
```

### 原因

闭包默认强引用捕获外部变量。

引用关系：

```text
ViewController
        ↓
      Closure
        ↓
   ViewController
```

导致循环引用。

---

### 解决方式

使用捕获列表：

```swift
request { [weak self] in

    self?.updateUI()

}
```

如果能够确定生命周期：

```swift
request { [unowned self] in

    updateUI()

}
```

实际开发中大多数使用 `[weak self]`。

---

# 2. Timer 导致循环引用

### 问题场景

例如：

```swift
class ViewController {

    var timer: Timer?

    func startTimer() {

        timer = Timer.scheduledTimer(
            withTimeInterval: 1,
            repeats: true
        ) { _ in

            self.update()

        }
    }
}
```

---

### 引用关系

```text
ViewController
        ↓
      Timer
        ↓
    Closure
        ↓
 ViewController
```

形成循环引用。

---

### 解决方式

方式一：

```swift
timer = Timer.scheduledTimer(
    withTimeInterval: 1,
    repeats: true
) { [weak self] _ in

    self?.update()

}
```

方式二：

页面销毁时释放：

```swift
deinit {

    timer?.invalidate()

}
```

实际项目中通常两个一起做。

---

# 3. NotificationCenter 闭包监听导致泄漏

### 问题场景

```swift
NotificationCenter.default.addObserver(
    forName: .update,
    object: nil,
    queue: .main
) { notification in

    self.refresh()

}
```

---

### 原因

NotificationCenter 长期持有 observer 和闭包。

关系：

```text
NotificationCenter
        ↓
      Closure
        ↓
        VC
```

导致 VC 无法释放。

---

### 解决方式

保存 observer：

```swift
private var observer: NSObjectProtocol?

observer =
NotificationCenter.default.addObserver(
    forName: .update,
    object: nil,
    queue: .main
) { [weak self] _ in

    self?.refresh()

}
```

销毁：

```swift
deinit {

    if let observer {
        NotificationCenter.default.removeObserver(observer)
    }

}
```

---

# 4. Delegate 没有使用 weak

### 问题场景

```swift
protocol DemoDelegate: AnyObject {

}

class Manager {

    var delegate: DemoDelegate?

}
```

---

### 原因

双方强引用：

```text
ViewController
        ↓
     Manager
        ↓
   ViewController
```

形成循环。

---

### 解决方式

代理必须 weak：

```swift
weak var delegate: DemoDelegate?
```

并且协议：

```swift
protocol DemoDelegate: AnyObject {

}
```

---

# 5. Combine / RxSwift 订阅导致泄漏

### 问题场景

Combine：

```swift
publisher
    .sink {

        self.update()

    }
    .store(in: &cancellables)
```

---

### 原因

关系：

```text
ViewController
        ↓
 cancellables
        ↓
 AnyCancellable
        ↓
 Closure
        ↓
 ViewController
```

循环引用。

---

### 解决方式

```swift
publisher
    .sink { [weak self] value in

        self?.update()

    }
    .store(in: &cancellables)
```

RxSwift：

```swift
observable
    .subscribe { [weak self] event in

    }
    .disposed(by: disposeBag)
```

---

# 6. WKWebView 导致循环引用

### 问题场景

```swift
webView.navigationDelegate = self
```

---

### 原因

如果 delegate 没有弱引用：

```text
WebView
   ↓
Delegate
   ↓
ViewController
```

导致页面无法释放。

---

### 解决方式

销毁：

```swift
webView.navigationDelegate = nil
webView.uiDelegate = nil
```

同时：

```swift
webView.removeFromSuperview()
```

---

# 7. GCD 异步任务延长对象生命周期

### 问题场景

```swift
DispatchQueue.global().async {

    self.loadData()

}
```

---

### 问题

闭包强引用 self。

关系：

```text
GCD Queue
      ↓
   Closure
      ↓
      VC
```

通常不会永久泄漏，但是会导致 VC 延迟释放。

---

### 解决

```swift
DispatchQueue.global().async { [weak self] in

    self?.loadData()

}
```

---

# 8. 图片缓存、播放器、数据缓存导致内存暴涨

### 常见问题

例如：

- 大量 UIImage 强引用。
- AVPlayer 没释放。
- 视频缓存无限增长。
- 大数组长期保存对象。

---

### 解决方式

图片：

- 使用缓存策略。
- 控制缓存大小。
- 页面退出释放。

播放器：

```swift
player.pause()
player.replaceCurrentItem(with: nil)
```

列表：

- 使用复用机制。
- 及时释放不可见资源。

---

# 9. 排查内存泄漏方法

实际开发中一般使用：

### Xcode Memory Graph

查看：

```
Debug Memory Graph
```

观察：

- 谁持有对象。
- 循环引用路径。


---

### Instruments

使用：

```
Leaks
Allocations
```

查看：

- 内存增长。
- 未释放对象。


---

### deinit 检查

例如：

```swift
deinit {

    print("释放")

}
```

页面返回后没有打印：

说明存在强引用。

---

### 💡 面试总结回答

> 实际开发中遇到过比较多的内存泄漏主要有闭包循环引用、Timer、NotificationCenter、Delegate、Combine/RxSwift 订阅以及播放器资源未释放等问题。排查时主要通过 Xcode Memory Graph 和 Instruments Leaks 分析引用链。解决方式主要是合理使用 weak/unowned、解除 observer、invalidate Timer、delegate 使用 weak、取消订阅以及及时释放资源。核心原则是找到是谁持有对象，打破不必要的强引用链。

### ⭐ 关键字

`ARC`
`循环引用`
`weak`
`unowned`
`Timer`
`NotificationCenter`
`Delegate`
`Combine`
`RxSwift`
`Memory Graph`
`Instruments`

</details>


<details>
<summary>13. 实际开发中遇到过哪些内存泄漏问题？都是怎么解决的？口述版本</summary>
  
> 实际项目中遇到比较多的内存问题，主要集中在闭包、Timer、通知监听、代理以及一些资源释放方面。

最常见的是闭包循环引用。比如网络请求、异步回调这些场景，如果闭包里面直接使用 self，而这个闭包又被某个对象长期保存，就可能导致 ViewController 不能释放。

比如之前处理网络请求回调的时候，发现页面退出之后 deinit 没有执行，后来通过 Xcode 的 Memory Graph 查看引用链，发现是闭包强引用了当前页面。解决方式就是在闭包里面使用 [weak self]，避免闭包强持有页面对象。

第二类比较常见的是 Timer。比如一些倒计时、轮询功能，如果 Timer 的回调闭包里面直接引用 self，同时 ViewController 又持有 Timer，就会形成一个循环引用：

ViewController 持有 Timer，Timer 持有闭包，闭包又持有 ViewController。

这种情况一般会在停止计时的时候调用 invalidate，同时闭包里面使用 weak self，双重保证释放。

第三类是 NotificationCenter。以前 block 形式添加通知监听的时候，如果没有保存 observer，也没有及时移除监听，同时闭包里面又引用了 self，可能会导致页面一直被持有。

解决方式一般是保存 observer，在页面销毁的时候 removeObserver，同时闭包里面使用弱引用。

还有代理相关的问题，比如 delegate 如果没有声明 weak，也会造成双方互相持有。所以项目里定义 delegate 的时候，一般协议会继承 AnyObject，然后 delegate 使用 weak 修饰。

除了这些，还有一些资源类的问题，比如播放器、WebView、图片缓存等。比如播放器退出页面后，需要及时 pause，并且释放 currentItem；WebView 需要处理 delegate 引用；列表页面也需要注意大图片和视频资源释放，避免内存持续增长。

排查这类问题，我一般会先通过 deinit 打日志确认对象有没有释放，然后使用 Xcode 的 Memory Graph 查看引用链，必要的时候使用 Instruments 的 Leaks 和 Allocations 分析。

总体来说，处理内存问题的核心就是找到对象之间的引用关系，判断是谁强持有了对象，然后通过 weak、取消监听、invalidate、释放资源等方式打破不必要的强引用。</details>


<details>
<summary>13. 项目中有使用多线程吗？讲一下应用场景、线程通信、线程安全以及内存问题。</summary>

# ✅ 面试回答（口语版）

有，项目里面多线程使用还是比较多的，因为 iOS 的主线程主要负责 UI 渲染、事件响应和 RunLoop，如果把一些耗时操作放在主线程执行，就会导致页面卡顿、掉帧，影响用户体验。

所以我们项目一直遵循一个原则：

> **耗时操作放后台线程，UI 更新回主线程。**

实际项目中，我主要会从几个方面考虑多线程的使用：**应用场景、线程通信、线程安全、内存管理以及问题排查。**

---

# 一、项目中哪些地方使用了多线程？

### 1、网络请求

这是使用最多的场景。

比如首页列表、详情页、聊天消息、用户信息等接口请求。

网络请求本身就是异步执行的，返回数据之后，会在后台解析 JSON、模型转换、业务计算，最后再回到主线程刷新 UI。

例如：

- 刷新 TableView
- 更新 CollectionView
- 修改按钮状态
- 更新 Label

因为 UIKit 不是线程安全的，所以所有 UI 更新都必须放到主线程。

---

### 2、图片处理

图片下载完成以后，还需要做：

- 图片解码
- 图片压缩
- 图片裁剪
- 图片缓存

这些操作都比较耗 CPU。

如果直接放到主线程，列表滑动的时候就很容易掉帧。

所以通常都是后台线程处理，处理完成以后再切回主线程显示。

---

### 3、视频业务

我之前做的视频项目里面，多线程使用会更多。

例如：

- 视频下载
- 视频缓存
- 视频预加载
- 首帧图片生成
- 文件写入
- 视频解码

这些全部都是后台执行。

因为如果这些任务阻塞主线程，视频播放和列表滑动都会明显卡顿。

---

### 4、数据库操作

例如：

- SQLite
- WCDB
- CoreData

聊天记录、本地缓存、收藏列表等数据库查询，都不会放主线程。

因为数据库属于 IO 操作，如果放主线程，很容易导致页面卡顿。

---

### 5、文件操作

例如：

- 文件下载
- 文件上传
- 日志写入
- Zip 解压

这些都会放后台线程执行。

---

# 二、多线程之间如何通信？

线程之间最常见的通信，就是：

**后台线程处理数据，主线程刷新 UI。**

例如：

```text
后台线程

↓

请求数据

↓

解析数据

↓

主线程

↓

刷新UI
```

以前项目里面使用 GCD 比较多。

后台完成以后：

```swift
DispatchQueue.main.async {

    self.tableView.reloadData()

}
```

如果使用 Swift Concurrency：

可以使用：

```swift
await MainActor.run {

}
```

或者：

```swift
Task { @MainActor in

}
```

来保证 UI 更新一定发生在主线程。

---

除了主线程通信之外，后台线程之间也会通信。

例如视频项目：

```text
线程A

下载视频

↓

线程B

生成首帧

↓

线程C

写入缓存

↓

主线程

刷新页面
```

这种任务之间存在依赖关系。

项目里面一般会使用：

- DispatchGroup
- OperationQueue
- async / await

去组织多个任务之间的执行顺序，而不是自己创建大量线程。

---

# 三、线程安全是怎么保证的？

线程安全主要发生在：

**多个线程同时访问同一份共享数据。**

例如：

项目里面有一个缓存数组：

```swift
var cacheList = [Model]()
```

线程 A：

新增数据。

线程 B：

删除数据。

线程 C：

遍历数据。

如果没有任何同步措施，就可能出现：

- 数据错乱
- 数组越界
- Crash
- 数据丢失

因为 Swift 的 Array、Dictionary 默认都不是线程安全的。

---

### 项目里面一般怎么处理？

### 第一种：串行队列（最常用）

如果共享数据比较简单，我一般会使用串行队列。

例如：

所有写操作都放到同一个串行队列里面。

这样同一时间只有一个线程能够修改数据。

实现简单，而且性能也不错。

---

### 第二种：锁

如果多个地方都需要访问共享资源。

例如：

缓存管理器。

下载管理器。

数据库。

我会使用：

- NSLock
- os_unfair_lock

保证同一时间只有一个线程能够访问共享资源。

避免数据竞争。

---

### 第三种：Barrier（读写分离）

项目里面缓存模块比较适合这种方式。

因为：

读取次数很多。

写入次数比较少。

这时候：

读操作可以并发执行。

写操作独占执行。

这样既保证了线程安全，又提高了整体性能。

---

### 第四种：Actor（Swift Concurrency）

如果项目使用 Swift Concurrency。

我更倾向使用 Actor 管理共享状态。

Actor 本身就是线程隔离的。

系统保证：

同一时间只有一个任务访问内部数据。

不用自己再去加锁。

代码也更加安全。

---

# 四、多线程里面遇到过哪些问题？

## 1、线程安全问题

例如：

多个下载任务同时修改下载列表。

多个播放器同时修改播放器状态。

多个请求同时更新缓存。

如果没有同步，很容易导致：

数据异常。

后来统一通过串行队列或者锁管理共享资源。

---

## 2、死锁问题

比较典型的就是：

在同一个串行队列里面又调用 sync。

或者：

主线程里面调用：

```swift
DispatchQueue.main.sync
```

都会造成死锁。

后来项目里面基本遵循一个原则：

- 主线程不用 sync。
- 同一个串行队列不嵌套 sync。
- 耗时任务使用 async。

---

## 3、数据竞争（Data Race）

例如：

两个线程同时修改同一个对象。

有时候不会立即崩溃。

但是数据已经出现异常。

这种问题比较隐蔽。

一般会打开：

**Thread Sanitizer**

检查数据竞争。

---

# 五、多线程里面有哪些内存问题？

### 第一类：异步闭包强引用 self

例如：

网络请求。

GCD。

下载任务。

如果闭包里面直接使用 self，而任务执行时间又比较长。

页面退出以后：

对象可能一直不能释放。

所以项目里面：

只要是异步回调，我都会先分析闭包生命周期。

如果属于长期任务。

都会使用：

```swift
[weak self]
```

避免对象被长期持有。

---

### 第二类：后台任务没有取消

例如：

页面已经退出。

但是：

下载还在继续。

上传还在继续。

视频解析还在继续。

虽然页面已经没有了。

但是任务依然持有很多资源。

不仅浪费 CPU。

还会导致内存持续上涨。

所以页面销毁的时候。

如果任务已经没有意义。

都会主动：

- cancel Task
- cancel Operation
- cancel URLSessionTask

及时结束后台任务。

---

### 第三类：并发数量过高

例如：

一次下载几十张图片。

一次缓存几十个视频。

如果全部一起执行。

瞬间就会创建很多对象。

容易导致：

- 内存峰值过高
- CPU 飙升
- OOM（内存不足）

所以项目里面都会限制最大并发数量。

例如：

下载同时最多 3～5 个。

其余排队等待。

---

### 第四类：资源没有及时释放

例如：

AVPlayer。

WKWebView。

大图片。

数据库连接。

页面退出以后。

如果资源没有及时释放。

同样会导致内存持续增长。

所以播放器退出页面：

会暂停播放。

释放播放资源。

WebView 退出页面：

会移除代理。

释放资源。

---

# 六、平时怎么排查这些问题？

一般我会分几个步骤。

第一步：

在 ViewController 的 deinit 打印日志。

确认页面退出以后对象是否正常释放。

第二步：

使用 Xcode 的 Memory Graph。

查看对象引用关系。

看看是谁一直持有当前对象。

第三步：

如果怀疑线程安全问题。

会打开：

Thread Sanitizer。

检查是否存在 Data Race。

第四步：

如果怀疑内存持续增长。

会使用 Instruments。

主要查看：

- Leaks
- Allocations

分析对象是否存在泄漏或者内存持续增长。

---

# 七、总结（面试回答）

项目里面多线程主要应用在网络请求、图片处理、视频下载、数据库操作以及文件处理这些耗时任务。

我在开发过程中主要关注四个方面。

第一是**线程通信**。

后台处理完成以后，一定回到主线程更新 UI。

第二是**线程安全**。

多个线程访问共享资源时，根据业务场景选择串行队列、锁、Barrier 或 Actor 保证数据一致性。

第三是**内存管理**。

重点关注异步闭包、后台任务、下载任务等场景，避免对象生命周期被无意义延长，同时及时取消任务、释放资源。

第四是**问题排查**。

平时主要结合 deinit、Memory Graph、Thread Sanitizer 和 Instruments 来定位线程和内存问题。

整体来说，我认为多线程开发最重要的不是如何创建线程，而是如何合理地管理线程、保证线程安全，并且控制好对象生命周期和资源释放。

---

## ⭐ 面试关键词

- GCD
- DispatchQueue
- async / sync
- RunLoop
- 主线程
- 后台线程
- Thread Communication（线程通信）
- DispatchGroup
- OperationQueue
- async/await
- MainActor
- Thread Safe（线程安全）
- NSLock
- os_unfair_lock
- Barrier
- Actor
- Data Race（数据竞争）
- DeadLock（死锁）
- weak self
- Task.cancel()
- URLSessionTask.cancel()
- Thread Sanitizer
- Memory Graph
- Instruments
- Leaks
- Allocations

</details>

<details>
<summary>13. 项目中有使用多线程吗？讲一下应用场景、线程通信、线程安全以及内存问题。</summary>

## ✅ 面试回答（口语版）

有，项目里面多线程使用还是挺多的，因为 iOS 主线程主要负责 UI 渲染和用户交互，如果把一些耗时操作放到主线程，比如网络请求、图片处理、数据解析这些，就很容易造成页面卡顿。

所以我们基本都是遵循一个原则：

> **耗时操作放后台线程，UI 更新回主线程。**

---

### 一、项目中哪些地方使用了多线程？

项目里面比较常见的场景主要有下面几个。

#### 1、网络请求

像首页列表、详情页这些接口，请求回来之后，会先在后台线程做：

- JSON 解析
- 模型转换
- 数据处理

处理完成以后，再切回主线程刷新 TableView 或者 CollectionView。

因为 UIKit 本身不是线程安全的，所以所有 UI 更新都必须放在主线程。

---

#### 2、图片和视频处理

比如图片下载完成以后，还要进行：

- 图片解码
- 图片压缩
- 图片缓存

这些都是比较耗 CPU 的操作。

如果放到主线程，列表滑动的时候很容易掉帧。

视频项目里面也是一样，比如：

- 视频下载
- 视频缓存
- 视频预加载
- 首帧生成

这些都会放到后台线程执行，最后再通知主线程更新界面。

---

#### 3、数据库和文件操作

另外像：

- 数据库查询
- 文件上传
- 文件下载

这些 IO 操作，我们也都会放到后台线程去处理，避免影响用户操作。

---

### 二、线程通信

线程通信这一块，项目里面最常见的就是：

> **后台线程处理完成以后，回到主线程刷新 UI。**

以前使用 GCD 比较多，就是：

```swift
DispatchQueue.main.async {

}
```

如果项目使用 Swift Concurrency，也可以通过：

```swift
MainActor
```

来保证 UI 更新是在主线程执行。

如果有多个异步任务需要一起完成，比如：

- 下载图片
- 解析数据
- 写缓存

全部完成以后再刷新页面。

我会使用：

- DispatchGroup
- OperationQueue

来管理多个任务之间的执行顺序。

---

### 三、线程安全

线程安全也是项目里面比较关注的一点。

比如多个线程同时去修改：

- Array
- Dictionary
- 缓存数据

因为 Swift 的 Array、Dictionary 默认都不是线程安全的，如果不做同步，就可能出现：

- 数据竞争（Data Race）
- 数组越界
- Crash

所以我们会根据不同场景选择不同方案。

#### 第一种：串行队列

如果只是简单的共享数据。

一般会使用串行队列。

保证同一时间只有一个线程修改数据。

---

#### 第二种：Barrier

如果是读多写少的场景，比如缓存模块。

我会使用：

> 并发队列 + Barrier

这样：

- 读操作可以并发，提高性能。
- 写操作独占，保证数据一致性。

---

#### 第三种：锁

如果多个模块都要访问同一份共享资源。

也会使用锁，比如：

- NSLock
- os_unfair_lock

保证共享资源不会被多个线程同时修改。

---

#### 第四种：Actor

如果项目使用 Swift Concurrency。

我也会优先考虑使用 Actor 管理共享状态。

因为 Actor 天然就是线程安全的，不需要自己再去加锁。

---

### 四、多线程中的内存问题

多线程里面比较容易出现的问题就是内存管理。

例如网络请求、下载任务这些异步操作。

如果闭包里面直接使用 `self`，而任务执行时间比较长，就可能导致页面退出以后对象不能及时释放。

所以项目里面，只要是生命周期比较长的异步任务，我都会先分析闭包是不是会长期存在。

如果会，就使用：

```swift
[weak self]
```

避免闭包强引用当前页面。

---

另外还有一种情况。

页面已经退出了。

但是：

- 下载任务
- 上传任务
- 视频解析

还在继续执行。

如果这些任务已经没有意义。

我会主动调用：

```swift
cancel()
```

及时结束后台任务，避免继续占用 CPU 和内存。

---

还有就是并发数量不会无限制创建。

比如：

- 下载图片
- 缓存视频

我都会控制最大并发数量。

避免短时间创建太多对象，导致内存峰值过高。

---

### 五、问题排查

平时如果怀疑有内存或者线程问题。

我一般会先在：

```swift
deinit
```

打印日志，确认页面退出以后有没有正常释放。

如果没有释放。

我会使用：

**Memory Graph**

查看对象引用关系，看是谁一直持有当前对象。

如果怀疑是线程安全问题。

我会打开：

**Thread Sanitizer**

检查是否存在数据竞争。

如果怀疑内存持续增长。

我会使用 Instruments 的：

- Leaks
- Allocations

分析对象的创建和释放情况。

---

## ✅ 总结

我觉得多线程开发最重要的有四点。

**第一，合理使用多线程。**

耗时任务放后台执行，UI 更新回主线程。

**第二，保证线程安全。**

多个线程访问共享资源时，根据业务场景选择串行队列、锁、Barrier 或 Actor。

**第三，关注对象生命周期。**

避免异步闭包、后台任务导致对象无法释放，同时及时取消无意义的任务。

**第四，学会排查问题。**

结合：

- deinit
- Memory Graph
- Thread Sanitizer
- Instruments

定位线程和内存相关问题。

</details>


<details>
<summary>13. 项目中有使用多线程吗？讲一下应用场景、线程通信、线程安全以及内存问题，口语版本。</summary>

**回答：**

有，项目里面多线程使用还是比较多的，因为 iOS 主线程主要负责 UI 渲染和用户交互，如果把网络请求、图片处理、数据解析这些耗时操作放到主线程，就容易造成页面卡顿。所以我们一直遵循一个原则：**耗时操作放后台线程，UI 更新回主线程。**

实际项目中，多线程主要应用在网络请求、图片和视频处理、数据库查询、文件上传下载这些场景。比如接口请求回来以后，我会先在后台线程完成 JSON 解析、模型转换和数据处理，然后再通过 `DispatchQueue.main.async` 或者 `MainActor` 回到主线程刷新 UI，因为 UIKit 本身不是线程安全的。

线程通信方面，最常见的就是后台线程处理完成以后通知主线程更新界面。如果有多个异步任务需要协同完成，例如下载图片、解析数据、写缓存之后再刷新页面，我一般会使用 `DispatchGroup` 或者 `OperationQueue` 来管理多个任务之间的执行顺序。

线程安全方面，主要是避免多个线程同时修改同一份共享数据。因为 Swift 的 `Array`、`Dictionary` 默认都不是线程安全的，如果多个线程同时读写，就可能导致数据竞争、数组越界甚至 Crash。所以我会根据不同业务场景选择不同方案，比如使用串行队列保证同一时间只有一个线程修改数据；读多写少的场景会使用并发队列配合 `Barrier`；多个模块共享资源时会使用 `NSLock` 或 `os_unfair_lock`；如果项目采用 Swift Concurrency，也会优先使用 `Actor` 来管理共享状态。

多线程开发里面，我也比较关注内存管理。比如网络请求、下载任务这些异步操作，如果闭包生命周期比较长，又直接使用了 `self`，就可能导致页面退出以后对象不能及时释放。因此对于长期存在的异步任务，我都会评估是否需要使用 `[weak self]`，避免闭包强引用页面。另外，页面退出以后，如果下载、上传或者视频解析这些后台任务已经没有意义，我也会主动调用 `cancel()` 结束任务，避免继续占用 CPU 和内存。同时也会控制最大并发数量，避免短时间创建大量任务导致内存峰值过高。

平时排查问题时，我会先通过 `deinit` 确认页面是否正常释放；如果没有释放，再使用 **Memory Graph** 查看对象引用关系；如果怀疑线程安全问题，会打开 **Thread Sanitizer** 检查是否存在数据竞争；如果怀疑内存持续增长，则会使用 **Instruments** 的 **Leaks** 和 **Allocations** 分析对象创建和释放情况。

总的来说，我认为多线程开发最重要的是四点：第一，合理使用多线程，耗时任务放后台，UI 更新回主线程；第二，保证线程安全，根据业务场景选择串行队列、锁、Barrier 或 Actor；第三，关注对象生命周期，避免异步闭包和后台任务导致对象无法释放；第四，善于利用 `deinit`、Memory Graph、Thread Sanitizer 和 Instruments 排查线程和内存问题。

</details>

<details>
<summary>14. struct 和 class 有什么区别？</summary>

**回答：**

`struct` 和 `class` 都可以用来定义数据类型，但它们最大的区别在于**内存管理、赋值方式以及是否支持继承**。

首先，`struct` 是**值类型**，数据一般存储在栈上（如果包含引用类型成员或发生逃逸等情况，也可能涉及堆内存）。赋值或者作为参数传递时，会拷贝一份新的数据，修改副本不会影响原来的对象。

而 `class` 是**引用类型**，实例存储在堆上，变量保存的是对象地址。赋值或者传递时只是复制引用，多个变量指向同一块内存，所以修改其中一个对象，其他引用也会受到影响。

第二个区别是**内存管理**。

`struct` 不参与 ARC 引用计数，没有循环引用的问题。

而 `class` 由 ARC 管理，当多个对象相互强引用时，就可能出现循环引用，因此在闭包或者对象之间相互引用时，需要合理使用 `weak` 或 `unowned`。

第三个区别是**继承**。

`struct` 不支持继承，也不能被继承。

`class` 支持继承、多态，可以通过重写方法扩展子类功能，所以像 `UIViewController`、`UIView` 等 UIKit 类都是基于 `class` 实现的。

第四个区别是**身份比较**。

`struct` 比较的是值是否相等，通过实现 `Equatable` 使用 `==`。

`class` 除了可以比较值，还可以通过 `===` 判断两个变量是否指向同一个对象。

第五个区别是**可变性**。

`struct` 的实例如果使用 `let` 声明，整个对象都不能修改，即使某个属性是 `var` 也不可以。

而 `class` 即使实例使用 `let` 声明，只要属性是 `var`，依然可以修改属性值，因为 `let` 限制的是引用地址不能改变，而不是对象内容。

实际开发中，我一般遵循 Apple 推荐的原则：

如果对象只是保存数据，没有继承需求，也不需要多个地方共享同一份数据，我会优先使用 `struct`，因为它更安全、性能更好，也没有循环引用问题。

如果对象需要继承、需要共享状态、需要使用 ARC 管理生命周期，比如 ViewController、Manager、网络管理器等，我会使用 `class`。

**总结来说：**

- `struct`：值类型、拷贝传递、不支持继承、不参与 ARC、更安全。
- `class`：引用类型、共享对象、支持继承、由 ARC 管理、适合需要共享状态的场景。

</details>
