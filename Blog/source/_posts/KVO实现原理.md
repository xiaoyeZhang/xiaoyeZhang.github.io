---

## title: iOS -KVO 实现原理

# 一、概述

KVO（全称：Key-Value Observing），也叫键值监听，是一种观察者设计模式。它提供一种机制。也就是当 KVO 监听某一个对象时，一旦指定的对象的属性被修改后，则其观察者就会接受到通知，进而做出相应的动作。简单的说就是每次指定的被观察的对象的属性被修改后，KVO 就会自动通知相应的观察者了。

打个比方，有两个对象 A 和 B，B 中有某一个属性 D，A 观察了 B 中属性 D，当属性 D 发生变化的时候，A 收到回调而回调方法中有新的值或者旧的值。

KVO 其实也是“观察者”设计模式的一种应用。这种模式有利于两个类间的解耦合，尤其是对于 业务逻辑与视图控制 这两个功能的解耦合

# 二、KVO 基本原理

1.KVO 是基于 runtime 机制实现的 2.当某个类的属性对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的 setter 方法。派生类在被重写的 setter 方法内实现真正的通知机制 3.如果原类为 Person，那么生成的派生类名为 NSKVONotifying_Person 4.每个类对象中都有一个 isa 指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将 isa 指针指向动态生成的派生类，从而在给被监控属性赋值时执行的是派生类的 setter 方法 5.键值观察通知依赖于 NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey:；在一个被观察属性发生改变之前， willChangeValueForKey:一定会被调用，这就 会记录旧的值。而当改变发生后，didChangeValueForKey:会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。

# 三、 KVO 深入原理：

1.Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。当观察对象 A 时，KVO 机制动态创建一个新的名为： NSKVONotifying_A 的新类，该类继承自对象 A 的本类，且 KVO 为 NSKVONotifying_A 重写观察属性的 setter  方法，setter  方法会负责在调用原  setter  方法之前和之后，通知所有观察对象属性值的更改情况。

2.NSKVONotifying_A 类剖析：在这个过程，被观察对象的 isa 指针从指向原来的 A 类，被 KVO 机制修改为指向系统新创建的子类 NSKVONotifying_A 类，来实现当前类属性值改变的监听；

3.所以当我们从应用层面上看来，完全没有意识到有新的类出现，这是系统“隐瞒”了对 KVO 的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为“NSKVONotifying_A”的类()，就会发现系统运行到注册 KVO 的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为 NSKVONotifying_A 的中间类，并指向这个中间类了。

4.（isa 指针的作用：每个对象都有 isa 指针，指向该对象的类，它告诉 Runtime 系统这个对象的类是什么。所以对象注册为观察者时，isa 指针指向新子类，那么这个被观察的对象就神奇地变成新子类的对象（或实例）了。）  因而在该对象上对 setter 的调用就会调用已重写的 setter，从而激活键值通知机制。

5.子类 setter 方法剖析：KVO 的键值观察通知依赖于 NSObject 的两个方法:willChangeValueForKey:和 didChangevlueForKey:，在存取数值的前后分别调用 2 个方法： 被观察属性发生改变之前，willChangeValueForKey:被调用，通知系统该 keyPath  的属性值即将变更；当改变发生后， didChangeValueForKey: 被调用，通知系统该 keyPath  的属性值已经变更；之后,observeValueForKey:ofObject:change:context: 也会被调用。且重写观察属性的 setter  方法这种继承方式的注入是在运行时而不是编译时实现的。
![NSFileHandle.jpg](/images/1814928-335018fa848469d9.webp)

KVO 原理图

# 四、实现步骤

1、注册观察者

```

- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;
```

我们来看看每个参数的含义
**observer**

称为观察者 （这里观察 self.model 对象的属性变化）

**keyPath**

被观察的属性名称(这里观察 model 中 num 属性值的改变),这个值不能为 nil，而且这个值只能是 NSString 类型

**options**

观察属性的新值、旧值等的一些配置（枚举值，可以根据需要设置，例如这里可以使用两项）

    typedef NS_OPTIONS(NSUInteger, NSKeyValueObservingOptions) {

      //以字典的形式提供 “更新后新的数据”;

      NSKeyValueObservingOptionNew = 0x01,

      //以字典的形式提供 “初始对象数据”;

      NSKeyValueObservingOptionOld = 0x02,

      //观察最初的值（在注册观察服务时会调用一次触发方法）

      NSKeyValueObservingOptionInitial API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)) = 0x04,

    //分别在值修改前后触发方法（即一次修改有两次触发）
      NSKeyValueObservingOptionPrior API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0)) = 0x08

    };

**context**

上下文，可以为 KVO 的回调方法传值（例如设定为一个放置数据的字典，可以作为移除观察者的标示）

2、实现 KVO 回调方法

```

//keyPath:属性名称
//object:被观察的对象
//change:变化前后的值都存储在 change 字典中
//context:注册观察者时，context 传过来的值

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{

  if([keyPath isEqualToString:@"num"] && object == model) {

      }

  }

}
```

3、触发监听

> 我们设定好接听之后，当改变被监听对象的值时，就可以触发监听回调方法了。但是要注意的是，不能直接使用\_变量名直接赋值改变，而应该使用 self.变量名对被监听对象值得修改。这一点主要是因为 \_变量名只是直接操作对象指针，sel.变量名是调用 setter 方法操作对象指针，而 KVO 是通过 runtime 实现的，必须调用 setValue:forKey 方法赋值才会触发监听的。

4、移除监听

在监听完成之后，我们必须将监听移除，否则会导致内存溢出

    - (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(nullable void *)context API_AVAILABLE(macos(10.7), ios(5.0), watchos(2.0), tvos(9.0));

一般在 dealloc 方法中调用。这里有一个问题需要了解下，注册和移除是一一对应的，有几个注册就要有几个移除。如果一个类中有多个监听对象，可以有 context 值来区分。

5、KVO 控制是否自动发送通知

我们可以在模型类中添加这样一个方法:(返回 YES 表示自动发送通知,返回 NO 表示不自动发送通知)

    + (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{

        return NO;
    }

而在外部需要调用 KVO 时,需要在改变属性值的前后加上

    [model willChangeValueForKey:@"Value"];
    -------
    [model didChangeValueForKey:@"Value"];

# [Demo](https://github.com/xiaoyeZhang/KVO_Demo.git)

接下来我们通过例子来简单的实现 KVO

**建立观察者模型（建立 KVO_Model （NSObject）类）**

```
#import <Foundation/Foundation.h>

@interface KVO_Model : NSObject

@property (nonatomic,assign)int num;

@property (nonatomic,assign)NSString *age;
@end

```

**注册监听**

        self.model = [[KVO_Model alloc]init];

        [self.model addObserver:self forKeyPath:@"num" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld|NSKeyValueObservingOptionInitial|NSKeyValueObservingOptionPrior context:nil];

**实现回调方法**

    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{

        if([keyPath isEqualToString:@"num"] && object == model) {
            // 响应变化处理：UI更新（label文本改变）
            self.label.text = [NSString stringWithFormat:@"当前的num值为：%@",
                               [change valueForKey:@"new"]];

            for (NSString *key in  change) {
                NSLog(@"\\key :%@ key_value:%@",key,
                      [change valueForKey:key]);
            }

        }

    }

**改变被监听对象的值**

```

    //按一次，使num的值+1
    self.model.num = self.model.num + 1;

}
```

**移除 KVO**

    - (void)dealloc{

        [self.model removeObserver:self forKeyPath:@"num" context:nil];

    }

**模式转换**

```

@implementation KVO_Model

+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key{

    return NO;
}

@end
```

    //按一次，使num的值+1
        [self.model willChangeValueForKey:@"num"];
        self.model.num = self.model.num + 1;
        [self.model didChangeValueForKey:@"num"];

[iOS 初探 KVO](http://liumh.com/2015/08/25/ios-know-kvo/)
[深入理解 KVO](http://zhangbuhuai.com/understanding-kvo/)
