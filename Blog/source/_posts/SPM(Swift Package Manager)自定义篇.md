---
title: 掌握 Swift Package Manager（构建SPM包)
date: 2024-01-12 22:29:55
---
#### **Swift Package Manager**（Swift 包管理器，一般简称 SwiftPM 或者 SPM）是苹果官方（2016年随 Swift 3.0 一起发布） 提供的一个用于管理源代码分发的工具，旨在使分享代码和复用其他人的代码变得更加容易。该工具可以帮助我们编译和链接 Swift packages（包），管理依赖关系、版本控制，以及支持灵活分发和协作（公开、私有、团队共享）等

## 一. SwiftPM （SPM）基本概念

*   什么是 Packages？

    一个 package（包）由 Swift 源码文件和一个清单文件组成。这个清单文件被称为 `Package.swift`，它使用 `PackageDescription` 模块来定义包的名称和内容。一个包通常有一个或者多个 targets（目标），每个 target 指定一个 product（产品）并声明一个或者多个 dependencies（依赖）。

*   什么是 Products？

    在 package（包）中的每一个 target（目标）最终可能构建成一个 library（库）或者一个 executable（可执行文件）作为其 product（产品）。库是包含可以被其他 Swift 代码引用的模块，可执行文件是一段可以被操作系统运行的程序。

对于 SwiftPM 的一些**基本概念**，例如：Modules、Packages、Products、Dependencies、Targets 等，在 Swift.org [官网](https://link.juejin.cn?target=https%3A%2F%2Fswift.org%2Fpackage-manager%2F "https://swift.org/package-manager/")已经有非常详细的描述和定义

## 二. 如何创建SwiftPM （SPM）项目？

### 1. Xcode 14创建

在 Xcode 的菜单栏中选择 "File"（文件）> "New"（新建）> "Package..."（包...）。

### 2. 命令行创建

使用 Swift Package Manager 创建新的 Swift 包相当简单，只需要运行 `swift package init` 命令即可：

```Shell
    $ mkdir <package name> 
    $ cd <package name> 
    $ swift package init --type library
```

上述命令会创建一个新的库类型的 Swift 包，包名就是目录名。

## 三. Swift 包的基本结构

```Swift
MyPackage
    ├── Package.swift
    ├── Sources
    │   └── MyPackage
    │       ├── File1.swift
    │       └── File2.swift
    └── Tests
        └── MyPackageTests
            ├── File1Tests.swift
            └── File2Tests.swift

```

这里主要包含三个部分：

*   `Package.swift`：这是 Swift 包的配置文件，其中定义了包的名称、产品、依赖项等信息。

*   `Sources`：这个目录包含了 Swift 包的源代码。

*   `Tests`：这个目录包含了 Swift 包的测试代码。

## 四. 配置SPM依赖

在 `Package.swift` 文件中，我们可以声明当前包依赖的其他 Swift 包。例如：

```
let package = Package(
    name: "Example", 
    platforms: [
      .iOS(.v11)
    ],
    products: [
        .library(
            name: "Example",
            targets: ["Example"]),
    ],
    dependencies: [
        .package(url: "https://github.com/SnapKit/SnapKit.git", .upToNextMajor(from: "5.0.1"))
    ],
    targets: [
        .target(
            name: "Example",
            dependencies: ["SnapKit"],
            path: "Sources"
        ),
    ]
)

```

在这个例子中，我们的 "Example" 包依赖于 "SnapKit"，它的源代码托管在 GitHub 上

### 细说 Package.swift 文件

`Package.swift` 是一个包描述文件，这里面定义了包的一些基本信息，如包的名称，产品，依赖项等。它使用 Swift 语言编写。

在 `Package` 对象中，我们可以定义以下内容：

*   `name`：这是你的包的名称。

*   `platforms`：这定义了你的包支持的平台和版本。

*   `products`：这定义了你的包提供的产品。一个产品可以是一个库或一个可执行文件。

*   `dependencies`：这定义了你的包依赖的其他包。你需要提供每个依赖包的 URL 和版本。

*   `targets`：这定义了你的包的构建目标。每个目标可以依赖于源文件和其他目标。

以下是一个 `Package.swift` 的示例：

```
import PackageDescription

let package = Package(
    name: "NCCalendarPicker",
    platforms: [
      .iOS(.v11)
    ],
    products: [
        .library(
            name: "NCCalendarPicker",
            targets: ["NCCalendarPicker"]),
    ],
    dependencies: [
        .package(url: "https://github.com/SnapKit/SnapKit.git", .upToNextMajor(from: "5.0.1"))
    ],
    targets: [
        .target(
            name: "NCCalendarPicker",
            dependencies: ["SnapKit"],
            path: "Sources"
        ),
    ]
)

```

在这个例子中，我们定义了一个名为 "NCCalendarPicker" 的包，它在 iOS 11 及以上版本上运行，依赖于 "SnapKit"。

### 解说 Package.swift 与项目目录的关系

除了 `Package.swift` 文件之外，Swift Package 通常还有以下目录结构：

*   **Sources**：这个目录包含所有的 Swift 源代码文件。每个 target（目标）在 `Sources` 目录中都有一个对应的子目录。例如，如果你在 `Package.swift` 文件中定义了一个名为 "Example" 的 target，那么你应该在 `Sources` 目录下创建一个同名的 "Example" 子目录，然后将这个 target 的所有源代码文件放在这个子目录中。

*   **Tests**：这个目录包含所有的测试代码。和 `Sources` 目录类似，每个测试 target 在 `Tests` 目录中也有一个对应的子目录。

因此，`Package.swift` 文件和目录结构是一一对应的。你在 `Package.swift` 文件中定义的每个 target，都应该在 `Sources` 或 `Tests` 目录中有一个对应的子目录。

## 构建和运行

要构建和运行 Swift 包，我们可以使用 Swift Package Manager 的 `swift build` 和 `swift run` 命令。例如，要构建你的包，可以在终端中运行：

```Shell
 $ swift build
```

## 测试

如果你的 Swift 包中包含了测试代码，可以使用 `swift test` 命令来运行这些测试：

```Shell
$ swift test
```

## 上传至 Github

*   创建 Git 仓库：

![](/images/SPM/e36884b0-aeb7-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

建议勾选：『README』、『.gitignore』和『license』，然后点击『Create repository』。

*   clone && commit（建议打上 tag）：

![](/images/SPM/92acaeb0-aeb8-11ee-a1b8-f5a5b9e80723.png?v=1\&type=image)

### 总结

Swift Package Manager 是一个强大的工具，它能帮助我们更方便地管理 Swift 代码包，同时也使代码的分享和复用变得更加简单。掌握 SPM，你会发现在 Swift 开发中，无论是管理自己的代码，还是使用他人的代码，都变得非常方便。
