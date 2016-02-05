#OC的description方法
在APP的开发过程中，经常要打印并查看对象的信息，比较low的方法是编写代码，把对象的全部的属性都输出到日志中。最常用的做法是NSLog一下。
在构建需要打印到日志的字符串时，object对象会收到description消息，该方法所返回的描述信息将取代“格式字符串”里的“%@”等，比如：我们要打印数组时。

```
NSLog(@"arr = %@", arr);

```
控制台会输入：

```
arr = (
	"我是数组中的元素"
	"我也是数组中得元素"
)
```
对应的字典之类的系统自带的类中也会输出相应地信息，然而自定义的类就不是这样的，如果我们直接打印对象（不实现类的description方法）通常会在控制台打出如下信息：

```
//person 为 Person类的一个对象
person = <Person: 0x7fsjks898939>
```
与系统自带类相比，这样的输出并没什么鸟用，除非在自己的类中重写的description方法，否则打印信息时就会调用NSObject类所实现的默认方法。此方法定义在NSObject协议中，不过NSObject类实现了它。因为NSObject并不是唯一的“根类”，所以许多方法都要定义在NSObject协议里。比方说，NSProxy也是一个遵循了NSObject协议的“根类”。由于description等方法定义在NSObject协议里，因此像NSProxy这种“根类”及其子类也必须实现它们。如前所见。这些实现好的方法并没有打印出较为有用的信息，只不过是输出了类名和对象的内存地址。只有在你想判断两指针是否真的指向同一对象时，这种信息才有用处。除此之外，再也看不出其他有用的内容。

通常情况下，如果我们想让自定义的类输出更有用的信息只需覆盖description方法并将描述此对象的字符串返回即可，比如：

```
#import <Foundation/Foundation.h>

@interface Person : NSObject

@property (nonatomic, copy, readonly) NSString * name;
@property (nonatomic, copy, readonly) NSString * address;

- (id)initWithName:(NSString *)name address:(NSString *)address;

@end

@implementation Person

- (id)initWithName:(NSString *)name address:(NSString *)address{
	if ((self = [super init])){
		_name = [name copy];
		_address = [address copy];
	}
	return self;
}

- (NSString *)description{
	return [NSString stringWithFormat:@"<%@ : %p, \"%@ %@\">", [self class], self, _name, _address];
}


@end

```
Person类的.h和.m文件如上所示。
这样的话当我们再次打印Person类的对象时，比如：

```
Person * peron = [Person alloc] initWithName:@"zhangsan" address:@"霾都"];
NSLog(@"person -> %@", person);
//Output
//peron -> <Person: 0x7fsjks898939, "zhangsan 霾都">
```
这样就比重写description之前所输出的信息更加清楚了，也更加方便我们在开发过程中得调试。

当然了，还有种更加简单地方法，也比较实用，那就是借助NSDictionary类的description方法，在自定义的description方法中，把待打印的信息放到字典里面，然后将字典对象的description方法所输出的内容包含在字符串里并返回，这样就可以实现精简的信息输出方式，比如还是上面的Person类，这次我们把description方法改写成如下所示：

```
- (NSString *)description{
	return [NSString stringWithFormat:@“%@: %p, %@”, [self class], self, @{@"name":_name,
					@"address":_address}];
}
//NSLog
//NSLog(@"%@", person);

//output:
//person = <Person: 0x7fsjks898939, {
				name = zhangsan;
				address = 霾都;
			}>
```
怎么样？是不是比上一个还要好点，这样做的也很方便在自定义类中增加，删除和修改属性。

NSObject协议中还有一个方法在程序的调试中，当我们需要用到po命令时需要注意，那就是debugDescription，具体原理和description差不多，也就是没实现debugDescription时，当我们po一个对象时，控制台打印出的信息也就顶多一个内存地址，不方便我们调试程序，这是实现debugDescription，其中返回的数据格式和我们最终改写的description方法一样就ok了，这样我们就可以在程序的运行过程中方便的打断点，po对象，调试程序，而不用一遍又一遍的终止程序，添加NSLog语句了。

