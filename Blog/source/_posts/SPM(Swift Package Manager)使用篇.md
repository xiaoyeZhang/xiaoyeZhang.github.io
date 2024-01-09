---
title: 掌握 Swift Package Manager（使用SPM依赖包)
date: 2024-01-05 21:09:32
---
#### 至从苹果 WWDC 18 推出了 Swift Package Manager 后，我们不再只局限于使用 CocoaPods 和Carthage第三方包管理工具，现有项目还是会延续使用 CocoaPods，但新的项目，特别是打算完全使用 Swift 开发的项目，可以考虑使用 SPM 来作为包管理工具，也是一个不错的选择；当然，CocoaPods 与 SPM 共用也并不冲突，不过维护两份也是较麻烦的。

## 一、项目中集成

在 Xcode 的项目中选择 PROJECT，如图：

![](/images/SPM/ff3573e0-aeb9-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

这里，我选择 AFNetworking 的 Swift 版本：Alamofire，输入Alamofire的github地址：<https://github.com/Alamofire/Alamofire>

![](/images/SPM/13146d80-aeba-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

点击Add Package，等待获取Alamofire的源（可能会比较慢），获取成功后就会出现如下图：

![](/images/SPM/15717050-aeba-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

再点击 Add Package，添加源到项目中，会以下图方式展现：

![](/images/SPM/13146d80-aeba-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

## 二、更新依赖

更新依赖也比较简单：菜单 ->『File』-> 『Swift Packages』->『Update to Latest Package Versions』即可，如下图：

![](/images/SPM/e8dd9590-aeba-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

## 三、测试依赖源

```Swift
import Alamofire

AF.request("https://httpbin.org/get").response { response in
    debugPrint(response)
}
```

执行代码，结果如下：

![](/images/SPM/37e3cf80-aebe-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

## 四、其它

### 4.1、源码存放位置

我们可以通过『Show in Finder』直接打开源码所在的目录，如下图：

![](/images/SPM/14397340-aebf-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)

实际上，这就是该项目在 Xcode 中编译缓存的地方。

![](/images/SPM/111a31e0-aebf-11ee-a1b8-f5a5b9e80723.jpeg?v=1\&type=image)
