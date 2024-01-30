-------------------------
title: 组件化之CocoaPods库的创建和管理
date: 2024-01-29 10:09:32
-------------------------

#### 在iOS项目开发中，我们可以使用CocoaPods制作自己的远程私有库或者开源库，然后使pods方式来管理。CocoaPods库项目一般选择在github或者gittee、gitLab上创建开源库或者私有库。

### 一、制作自己的开源库

CocoaPods库在Mac电脑中的地址：`/Users/xxx(用户目录)/.cocoapods/repos`,查看隐藏文件夹通`command + shift + .`快捷键，在目录里可以看到自己电脑里面包含的Cocoapods官方索引库和自己的索引库。

![](/images/CocoaPods/6dcf28c0-be53-11ee-b320-1374ce0190c8.jpeg)

#### 1.在github创建自己的远程开源仓库

我们在github中创建一个开源仓库，这里使用github为远程库

![](/images/CocoaPods/f789ff80-be63-11ee-b320-1374ce0190c8.jpeg)

#### 2.创建本地代码仓库，cd进入你要创建模块的目录下

##### 2.1 利用 pod lib create 来创建 库项目

```Shell
- 创建命令（此处NCCalendarPicker就是你的库名称，后面要用到，取名最好严谨一些）
> pod lib create NCCalendarPicker

- 选择开源库平台
What platform do you want to use?? [ iOS / macOS ]
> iOS

- 选择编程语言
What language do you want to use?? [ Swift / ObjC ]
> Swift

- 在你的项目中是否创建一个demo工程，为了方便测试，我选择了Yes
Would you like to include a demo application with your library? [ Yes / No ]
> Yes

- 测试框架选择哪一个
Which testing frameworks will you use? [ Specta / Kiwi / None ]
> None

- 要不要做视图测试
Would you like to do view based testing? [ Yes / No ]
> Yes

- 类前缀名
What is your class prefix?
> NC

```

##### 2.2 不使用 `pod lib create`, 利用Xcode创建

###### 2.2.1 克隆远程项目到本地

    git clone https://github.com/xiaoyeZhang/NCCalendarPicker.git

###### 2.2.2 点击Xcode菜单：File -> New -> Package

![](/images/CocoaPods/lC6Pbfle0nLuhLCp.png)

![](/images/CocoaPods/OziiKrMXxebbX4sr3On.png)

###### 2.2.3 将写好的开源库源代码放入本地的Git仓库下的Sources目录下

    // 创建NCCalendarPicker.podspec
    pod spec create NCCalendarPicker

#### 3.将自己的代码文件放到NCCalendarPicker/Classes/中

在Example中打开自己的工程，命令行执行pod install后, Example运行，如果没有问题，接下来编辑好`NCCalendarPicker.podspec`文件.

```Swift
Pod::Spec.new do |s|

  s.name          = "NCCalendarPicker" //库名称
  s.version       = "0.0.1" //库版本
  s.summary       = "Public" //库简短描述
  s.description   = <<-DESC
                    日历选择库
                    DESC  // 库具体描述
  s.homepage      = "https://github.com/xiaoyeZhang/NCCalendarPicker.git" //库首页地址
  s.license       = "MIT" //协议，默认MIT
  s.author        = { "user_name" => "email" } //作者
  s.platform      = :ios, "11.0" //库最低支持版本
  s.swift_version = '5.0' //库最低支持swift版本
  s.source        = { :git => "https://github.com/xiaoyeZhang/NCCalendarPicker.git", :tag => "#{s.version}" }
  s.source_files  = "Classes","Classes/**/*.swift" //库文件路径
  s.exclude_files = "Classes/Exclude"
  s.resources = "Resources/*" 资源文件地址
  s.user_target_xcconfig = {
    'GENERATE_INFOPLIST_FILE' => 'YES'
  }
  s.pod_target_xcconfig = {
    'GENERATE_INFOPLIST_FILE' => 'YES'
  }
  s.frameworks    = "UIKit", "Foundation" //如果使用了原生库，在这里添加
  s.dependency   "SnapKit" //如果依赖了其他三方库，在这里添加
end
```

上述填好之后进行下一步验证podspec文件有没有填对, 命令行cd到最外层的NCCalendarPicker文件夹.

    1. pod lib lint / pod lib lint --allow-warnings // 执行命令检测本地项目是否通过编译
    如果检查正确了会提示
    NCCalendarPicker passed validation.

    2. 验证NCCalendarPicker.podspec文件(NCCalendarPicker为库名字)
    pod spec lint NCCalendarPicker.podspec
    如果检查正确了会提示
    NCCalendarPicker.podspec passed validation.

#### 4.将项目代码提交到远程Github仓库中

```Shell
# git add .
# git commit -m "第一次提交 版本为0.0.1"
# git push origin master

# git tag -a 0.0.1 -m "v0.0.1封版" // 这里要跟podspec文件中的s.version保持一致
# git push origin 0.0.1(将tag提交到远程)

```

自此代码就成功上传到了远程代码仓库中，接下来就是要把本地索引podspec添加到索引公有库

#### 5.push到Cocoapods

```Shell
使用push命令，发布自己的版本库到Cocoapods
pod trunk push NCCalendarPicker.podspec

如果依赖了第三方库，需要加上 --use-libraries 选项

```

### 二、push错误处理

如果 `pod trunk push`出现 `[!] You need to run pod trunk register to register a session first.`说明没有注册Cocoapods账号,输入以下命令注册.

```Shell
pod trunk register email 'user_name' --verbose // 把email和user_name换成自己的邮箱和用户名
```

会给邮箱发送确认邮件，在邮箱里点击链接同意就可以了，重新`pod trunk push NCCalendarPicker.podspec`
