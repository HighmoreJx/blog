# copy
这篇文章主要是看[iOS 中的 Copying](https://joeshang.github.io/),然后小结一下.

## 自问自答

🧐 什么是拷贝?  
😃 : 一模一样的来一份呗.   

🧐 拷贝的场景是啥?  
😃: 我们回到熟悉的抄作业. 每当暑假要完的时候, 我们总会在临近前的最后一天疯狂抄大佬的作业. 这个就是拷贝最好的例子了.    
🧐  一模一样会不会被老师抓?
🤫: 咳咳. 对, 抓紧改几道题的答案. 你的改动会不会影响到大佬的那份? 显然是不会的.    
😶 我头铁, 我一个字都不想改.  
😈: 你是在挑战老师的智商吗? 名字都抄一样的.你这拷贝毫无意义, 结局都是GG.  

👨‍🏫 : 拷贝的主要目的是用于拷贝一份新的数据进行修改, 而不会影响到原来的数据.如果不修改, 拷贝就没有必要. 

## Objective-C 中的拷贝
举个🌰:
```
NSInteger b = 3;
NSInteger a = b;
a = 5;
```
NSInteger是值类型, 上面就是最简单的一次拷贝.

举个🍐:
```
@interface Person : NSObject
@property (nonatomic, assign) int age;
@property (nonatomic, strong) NSMuatableString *name;
@end

Person *person1 = [[Person alloc] init];
Person *person2 = person1;
```
🤥 这是OC对象的拷贝.

```
person1.age = 10;
person2. age = 20;
```

最后发现 ~ person1 和 person2 居然同岁了…  
🧐 我们说过, 拷贝的意图在于产生一份相同的数据, 然后可以任意修改且不影响原始版本. 很明显上面的例子根本不符合条件. 所以根本不是拷贝.  

严肃一点看下官网:  [Object copying](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/ObjectCopying.html)  
我们主要关注一句话: 
>  If you receive an object from elsewhere in an application but do not copy it, you share the object with its owner (and perhaps others), who might change the encapsulated contents  

简单来说, 上面的例子只是多了一个持有关系, 根本不是拷贝! 只有两个值类型对象的赋值才完全符合我们对拷贝的认知.

😳 很懵逼, 那OC对象要怎么拷贝? 
🧐 是时候看看[苹果官方文档]([NSCopying - Foundation | Apple Developer Documentation](https://developer.apple.com/documentation/foundation/nscopying?language=objc))了.  

😀 简单来说, 就是要实现NSCoying协议.  
```
#import "Person.h"

@interface Person()<NSCopying>
@end

@implementation Person
- (id)init {
    if (self = [super init]) {
        _name = [[NSMutableString alloc] initWithString:@"mike"];
    }
    return self;
}

- (id)copyWithZone:(NSZone *)zone {
    Person *object = [[[self class] allocWithZone:zone] init];
    object.name = _name;
    object.age = _age;
    return object;
}
@end

Person *person1 = [[Person alloc] init];
Person *person2 = [person1 copy];
person1.age = 10;
person2.age = 20;
```
这个时候通过输出, 我们可以发现两个人的年纪已经不同了.

😃 这完全符合我们对拷贝的定义. 程序从堆中开辟出了完全独立的空间重新构造了person2对象. 我们对person2的操作完全独立与person1.

😈一切真的这么简单吗?
```
[person2.name insertString:@"funny " atIndex:0];
```
灵异的事情发生了, person1也被改了.这就完全不符合拷贝的意图了.  

😞 那么问题出在哪里了呢?  
`object.name = _name;`  
上面说过这个根本不是拷贝, 这是多了一个持有关系而已(多了一个引用).相当于一个盒子有两把钥匙, 盒子里面装的东西变了, 那随便你用哪把钥匙打开盒子, 最后里面的东西肯定是最新放入的.  

👨‍🏫 指针对象虽然被拷贝了, 但两者还是指向同一块内存空间.  

### 深拷贝与浅拷贝

![object copy](https://joeshang.github.io/images/blog/ios-object-copy.png)

上面是苹果官方文档的一幅图, 图中左侧就很清晰的展示了我们上面所描述的person.name的场景.   
同时我们注意到, 图中右侧还有一副对比图.两幅图分别对应了程序开发的浅复制与深复制.  

😒 那我们person如何做到深复制呢?
```
- (id)copyWithZone:(NSZone *)zone {
    Person *object = [[[self class] allocWithZone:zone] init];
    object.name = [[NSMutableString alloc] initWithString:object.name];
    object.age = _age;
    return object;
}
```
😀 这个时候我们可以再打印看看, person2对name的改变丝毫不会影响到person1了.  

👨‍🏫    
浅复制: 指针拷贝, 仅仅拷贝指向对象的指针  
深复制: 内容拷贝, 会拷贝对象本身  

⚠️ 深浅复制形容的是成员变量有对象指针的情况!
下面摘一下网上文章的例子:
```
 NSString *string1 = @"helloworld";
 NSString *string2 = [string1 copy]; // 浅拷贝
 NSString *string3 = [string1 mutableCopy]; // 深拷贝
 NSMutableString *string4 = [string1 copy]; // 浅拷贝
 NSMutableString *string5 = [string1 mutableCopy]; // 深拷贝
```
这些结论就是强行带入深浅拷贝的概念. 这根本没有像Person那样有指针成员变量, 哪有什么深不深浅不浅, 就只是分情况发生了拷贝或者没有拷贝而已.  

😒 
1. NSString本来就不可变, 拷贝一个不可变的string2有什么意义, 所以string2指针只需要持有一下stirng1的内容就可以了, 根本没有拷贝的必要
2. 剩下的几种都牵涉到改变不能相互影响, 所以需要拷贝一下. 

综上, 哪里需要什么深浅? 只是一个需不需要拷贝而已….  

👨‍🏫   
1. 不可变对象 copy：对象是不可变的，再复制出一份不可变对象没有意义，因此根本没有发生任何拷贝，对象只有一份。  
2. 不可变对象 mutableCopy：可变对象的能够修改，原来的不可变对象不支持，因此需要复制出一个新对象，拷贝。  
3. 可变对象 copy：不可变对象不能修改，原来的可变对象不支持，因此需要复制出新对象，拷贝。  
4. 可变对象 mutableCopy：可变对象的修改不应该影响到原来的可变对象，因此需要复制出新对象，拷贝。  
5. 如果拷贝的对象的成员变量有指针指向另外的内存空间, 才存在深,浅的概念.

## 集合的拷贝
![集合拷贝](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Collections/Art/CopyingCollections_2x.png)

集合对象即NSArray 与 NSMutableArray 等, 其实就拷贝而言, 他们和Person的例子并没有什么太大区别.   

举个🌰
```
NSMutableString *str1 = [[NSMutableString alloc] initWithString:@"1"];
NSArray *arr1 = [NSArray arrayWithObjects:str1, nil];
NSArray *arr2 = [arr1 copy];
NSMutableArray *arr3 = [arr1 mutableCopy];

NSLog(@"%p %p %p", arr1, arr2, arr3);
```
输出:
```
0x100514b80 0x100514b80 0x100514d10
```
侧面验证上面说的是否发生拷贝的总结, 没毛病, 完全适用.  
```
[str1 insertString:@"0" atIndex:0];
NSLog(@"%@ %@ %@", arr1[0], arr2[0], arr3[0]);
``` 
输出:
```
01 01 01
```

😒 一改怎么全改了?  
😂 这个就和person的name类似的状况, 数组的指针对象的确被拷贝了, 但是大家都指向同一个内存空间, 换句话说就是多了几个持有关系而已.这也就是浅复制.    
😀 所以我们平时用的系统的集合对象调用copy或者mutablecopy, 要么就没拷贝, 要么就浅拷贝.  

😒那怎么深复制?  
```
NSArray *arr4 = [[NSArray alloc] initWithArray:arr1 copyItems:YES];
[str1 insertString:@"0" atIndex:0];
NSLog(@"%p %p %p %p", arr1, arr2, arr3, arr4);
NSLog(@"%@ %@ %@ %@", arr1[0], arr2[0], arr3[0], arr4[0]);
```
输出:  
```
0x100514b80 0x100514b80 0x100514d10 0x100516bb0
01 01 01 1
```
😀 完美, 达到了我们最初想要的拷贝效果, 副本和源本互不影响.  

⚠️  这个深复制也不是你想的完美状态, 下面引用大神的结论, 我感觉很清晰易懂了, 也就不献丑总结了.(出自文章最开头的博客)  

> 对于集合类型的对象，将 initWithArray:copyItems: 第二个参数设置成 YES 时，会对集合内每一个元素发送 copyWithZone: 消息，元素进行复制，但是对于元素中指针类型的成员变量，依然是浅拷贝，因此这种拷贝被称为单层深拷贝（one-level-deep copy）。

> 如果想进行完全的深拷贝，可以先通过 NSKeyedArchiver 将对象归档，再通过 NSKeyedUnarchiver 将对象解归档。由于在归档时，对象中每个成员变量都会收到 encodeWithCoder: 消息，相当于将对象所有的数据均序列化保存到磁盘上（可以看成换了种数据格式的拷贝），再通过 initWithCoder: 解归档时，就将拷贝过的数据经过转换后读取出来，深拷贝。




