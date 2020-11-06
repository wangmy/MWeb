# 1. @objc、@objcMembers

https://www.shangmayuan.com/a/4ab7ff0a848c48dd9a16e5a0.html

>运行时->动态派发
编译时

>Objective-C 中全部类都继承自NSObject，Swift 中的类若是要供 Objective-C 调用，必须也继承自NSObject

## @objc
>@objc修饰符的根本目的是用来暴露接口给 Objective-C 的运行时（类、协议、属性和方法等）

>添加@objc修饰符并不意味着这个方法或者属性会采用 Objective-C 的方式变成动态派发，Swift 依然可能会将其优化为静态调用

>@objc 的版本变化
    1. swift3 继承了NSObject 的非private的类和成员，默认加@objc，private接口想要暴露给 Objective-C 须要@objc的修饰
    2. Swift 4 中继承自NSObject的类只有在特定的情况下在默认加@objc

>使用@objc能够修改 Swift 接口暴露到 Objective-C 后的名字

## @objcMembers

>Swift4 后继承自NSObject的类再也不隐式添加@objc关键字，但在某些状况下很是依赖 Objective-C 的运行时（如 XCTest），因此在 Swift4 中提供了@objcMembers关键字，对类和子类、扩展和子类扩展从新启用@objc推断。

>使用@objc和@nonobjc能够指定开启或关闭某一extension中的全部方法的@objc推断。

## dynamic
>当前 Swift 的动态性依赖于 Objective-C，Swift3 中dynamic就隐式包含了@objc的意思，但考虑到之后版本的 Swift 语言和运行时将会自支持dynamic而再也不依赖于 Objective-C，因此在 Swift4 中将dynamic和@objc含义进行了抽离。
>
>dynamic methods don’t appear in the vtable
>SIL分析

[swift编译流程图](https://miro.medium.com/max/1400/1*wzgnQ32uL0GGi3XrX4rlTA.png)


码流 、码率 、比特率 、帧速率 、分辨率 。
https://zhuanlan.zhihu.com/p/99637622


# 2. Swift 循环引用

## 思考
class YourViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {  // [weak self] in
            print(self)
        }
        dismiss(animated: true)
    }
}

// 需不需要  [weak self]

Block -> self   self…> block,  不存在循环引用，所以dismiss后，不加 [weak self] 控制器不会释放，2s后block释放，才会释放，如果这时候，有些其他逻辑（比如网络）2s内返回，可能触发控制器的刷新逻辑，所以可以加上 [weak self]，按照逻辑来说控制器dismiss，就应该销毁掉


##[闭包引起的循环引用](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID56)

```
creating a strong reference cycle:
closure {
	->self // self.someMethod()
}
self.someProperty ->  closure
```

**reference types**

## ? ?? ! 解包 Any
```
if let touchView = touch.view as? UIPickerView
dict.updateValue(fakeFace! as Any, forKey: "fakeFace")

if let maskMode: Bool = userInfo?.maskMode as? Bool ?? false 

if let maskMode: Bool = userInfo?.maskMode ?? false

Non-optional expression of type 'Bool' used in a check for optionals

guard let fakeFace = fakeFace else {
                portraitImageView.fakeFace = nil
                return
            }

```

## swift 集合
https://juejin.im/post/6844904198228672520
https://www.lagou.com/lgeduarticle/119245.html
```
dict.updateValue(fakeFace! as Any, forKey: "fakeFace")

```

## provides an elegant solution to this problem
 - closure capture list



# 高清大图加载优化
https://juejin.im/post/6844903935933677581

[图片的工作流](https://github.com/path/FastImageCache#the-problem)。

## 优化的思路
>SDWebImage加载高清图的性能问题，主要是其直接加载高清图的原来尺寸导致的，而加载一张图片的内存占用 = 图片Width * 图片Height * 4。所以直接加载测试图的原始分辨率的内存占用是 7033 * 10100 * 4 / (1024 * 1024) = 约270MB。

高清图的加载，需要采用缩减分辨率的方法来减轻加载的内存压力

***参考资料***
[Image and Graphics Best Practices 谈谈 iOS 中图片的解压缩](http://blog.leichunfeng.com/blog/2017/02/20/talking-about-the-decompression-of-the-image-in-ios/)
[WWDC2018-最佳图像实践](https://developer.apple.com/videos/play/wwdc2018/219/)
[SDWebImage源码分析](https://juejin.im/post/6844904147544702989)

### DownSampling
>DownSampling的具体原理是，在图像解码之前加入创建缩略图的过程，对加载的Image进行预处理，减少解码后的Image Buffer的大小，从而减少加载的内存占用和内存峰值

