# 宏——iOS 开发者可能需要知道的宏相关知识

这篇文章其实是一个缝合怪，主要缝合了：

* MikeAsh 的： [https://mikeash.com/pyblog/friday-qa-2010-12-31-c-macro-tips-and-tricks.html](https://mikeash.com/pyblog/friday-qa-2010-12-31-c-macro-tips-and-tricks.html)
* 喵神的： [https://onevcat.com/2014/01/black-magic-in-macro/](https://onevcat.com/2014/01/black-magic-in-macro/)
* 宋劲杉的：[https://akaedu.github.io/book/index.html](https://akaedu.github.io/book/index.html)
* 苹果官方文档：[https://developer.apple.com/documentation/swift/imported\_c\_and\_objective-c\_apis/using\_imported\_c\_macros\_in\_swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/using_imported_c_macros_in_swift)

此致，敬礼！

## 什么是宏

我们先来看一下“宏”在维基百科中的定义：[计算机科学](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)里的宏是一种[抽象](https://zh.wikipedia.org/wiki/%E6%8A%BD%E8%B1%A1)（Abstraction），它根据一系列预定义的规则替换一定的文本模式。[解释器](https://zh.wikipedia.org/wiki/%E8%A7%A3%E9%87%8A%E5%99%A8)或[编译器](https://zh.wikipedia.org/wiki/%E7%BC%96%E8%AF%91%E5%99%A8)在遇到宏时会自动进行这一模式替换。对于[编译语言](https://zh.wikipedia.org/wiki/%E7%B7%A8%E8%AD%AF%E8%AA%9E%E8%A8%80)，宏展开在编译时发生，进行宏展开的工具常被称为宏展开器。宏这一术语也常常被用于许多类似的环境中，它们是源自宏展开的概念，这包括[键盘宏](https://zh.wikipedia.org/w/index.php?title=%E9%94%AE%E7%9B%98%E5%AE%8F&action=edit&redlink=1)和宏语言。绝大多数情况下，“宏”这个词的使用暗示着将小命令或动作转化为一系列指令。

这个定义看起来还是比较抽象的，在理解宏之前，我们需要先去了解 C 程序是怎么编译的。

在 Alex Smith 的 [Building C Project](http://nethack4.org/blog/building-c.html) 这篇文章中，详细介绍了 C 程序的编译过程。其中有一步为Preprocessing，在这一步中预处理器执行一些简单的文本操作，比如：

* 去掉注释
* 解析 `#include` 命令，将这些 include 指令替换成引用的文件
* 计算 `#if` 与 `#ifdef` 指令
* 计算 `#define`
* 将代码中其他使用 `#define` 定义的宏进行展开

我们重点关注在后边这两点：

* 计算 `#define`
* 将代码中其他使用 `#define` 定义的宏进行展开

其实预处理器对它处理的内容没有什么概念，比如说你不能使用 `#if` 来检查一个类型是否被定义过：

```text
#ifndef MyInteger
typedef int MyInteger
#endif
```

即使 `MyInteger`已经被定义了， `#ifndef MyInteger` 也总会为真，因为类型定义是在编译阶段被处理的，在预编译阶段类型定义还没有被处理。

另外，`#define` 定义的内容也不需要是语法正确的，像下面的这种写法是完全正确的。

```text
#define STARTLOG NSLog(@
#define ENDLOG , @"testing");
STARTLOG "just %@" ENDLOG
```

预处理器仅仅会把 `STARTLOG` 与 `ENDLOG` 替换成它定义的内容，在编译器运行的时候会检查语法是不是正确的，在编译阶段编译器会发现这个语法是正确的，所以可以正确的进行编译。

## 宏的分类

### **对象宏（object-like macro）**

对象宏是一种会在预处理时被代码片段替代的简单标识，最常见的使用方式是用其为一个常量定义一个别名。

```text
#define BUFFER_SIZE 1024
```

不过在 Objective-C 中我们不推荐使用这种方式来声明常量，详见《Effective Objective-C 2.0 编写高质量iOS与OS X代码的52个有效方法》中“第 4 条：多用类型常量，少用\#define 预处理指令”一节的内容。

### **函数宏（function-like macro）**

函数宏顾名思义，就是行为类似函数，可以接受参数的宏。具体来说，在定义的时候，如果我们在宏名字后面跟上一对括号的话，这个宏就变成了函数宏。

```text
#define MAX(a, b) ((a)>(b)?(a):(b))
```

相比较于对象宏，这种函数宏我们会比较难以理解，但同时它的能力也比较强大。

## 警告

C 语言的宏有时候功能很强大，有时候又不够强大。宏的特殊性质使宏很危险，所以要谨慎使用。

C 语言的预处理器是近乎图灵完备的。仅仅使用一个简单的的驱动程序，你就可以使用预处理器计算出任何可计算的方法。然而，想要做到这些有时候代码写起来会很困难并且很奇怪，所以相比较起来图灵完备的 C++ 模板看起来要简单一些。

尽管 C 语言的宏很强大，但是这些宏其实是很简单的。因为宏展开仅仅是简单的文本处理，所以可能就会有一些陷阱。比如涉及到运算符优先级的时候：

```text
#define ADD(x, y) x+y

// 结果会是 14，不是 20
ADD(2, 3) * 4

#define MULT(x, y) x*y

// 结果会是 13，不是 20
MULT(2 + 3, 4)
```

对于宏定义中任何可能需要用到括号的地方都加上括号，并且要考虑到被传入宏的各种形式的参数，以及会使用到宏的各种上下文。

在宏的展开中，多次计算一个宏的参数可能会导致不可预期的结果：

```text
#define MAX(x, y) ((x) > (y) ? (x) : (y))

int a = 0;
int b = 1;
int c = MAX(a++, b++)
// 现在 a = 1， c = 1， b = 3
// (a++ > b++ ? a++ : b++)
// b++ 会被计算两次
```

这种使用括号的宏的使用看起来像是进行函数调用，但是不要被迷惑，他们仅仅是文本替换。

## 如何编写宏

这一节主要介绍一下我们在编写宏的时候可能会用到的一些工具或者技巧，相信在掌握了谢谢技巧之后你不会畏惧编写宏了，因为你已经掌握了足够的工具。

### 多语句的宏

写一个包含很多语句的宏是很常见的。比如说，一个 timing 宏：

```text
#define TIME(name, lastTimeVariable) NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime]; if(lastTimeVariable) NSLog(@"%s : %f seconds", name , now - lastTimeVariable);lastTimeVariable = now
```

你会在一些常用方法的调用里使用这个宏，来确定这个方法的调用有多频繁。

```text
- (void)callALot 
{
	// do some work

	// time it
	Time ("callAlot", _calledALotLastTimeIvar);
}
```

这个代码运行起来没问题，但是读起来太丑了。我们可以把这个宏拆成多行。一般来说，一个`#define` 会在这行的结尾结束，但是如果你把一个 `\\` 符号放在这行的结尾，你就可以让预处理器知道这个定义会下一行结束。

```text
#define TIME(name, lastTimeVariable) \\
	NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime]; \\
	if(lastTimeVariable) \\
		NSLog(@"%s : %f seconds", name , now - lastTimeVariable);\\
	lastTimeVariable = now
```

这很容易办到，但是这个宏有缺陷。假设我们考虑这种用法：

```text
- (void)callALot 
{
	if(...) 
		TIME("callALot", _calledALotLastTimeIvar);
}
```

这个宏会被展开成这样：

```text
- (void)callALot 
{
	if(...) // only time some calls
		NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime];
	if(_calledALotLastTimeIvar)
		NSLog(@"%s: %f seconds", name, now - _calledALotLastTimeIvar);
	_calledALotLastTimeIvar = now;
}
```

这个代码不会被编译，因为在 if 语句里定义 now 是非法的。我们可以这样解决：

```text
#define TIME(name, lastTimeVariable) \\ 
{ \\
	NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime]; \\
	if(lastTimeVariable) \\
		NSLog(@"%s : %f seconds", name , now - lastTimeVariable);\\
	lastTimeVariable = now; \\
}
```

现在这个代码的展开会是这样：

```text
- (void)calledALot
{
	if(...) // only time some calls
	{
		NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime];
		if(lastTimeVariable)
			NSLog(@"%s: %f seconds", name, now - lastTimeVariable);
		lastTimeVariable = now;
	};
}
```

基本上没啥问题，除了最后多了一个分号。不过不是什么大问题，对吧？

实际上，还是有问题，假设有这样的代码：

```text
- (void)calledALot
{
	if(...) // only time some calls
		TIME("calledALot", _calledALotLastTimeIvar);
	else // otherwise do something else
		// stuff
}
```

代码展开会是这样：

```text
- (void)calledALot
{
	if(...) // only time some calls
	{
		NSTimeInterval now = [[NSProcessInfo processInfo] systemUptime];
		if(lastTimeVariable)
			NSLog(@"%s: %f seconds", name, now - lastTimeVariable);
		lastTimeVariable = now;
	};
	else // otherwise do something else
		// stuff
}
```

现在这个分号会造成语法错误...

你告诉使用这个宏的用户他们不应该在这个宏的后边加分号。但是，这是非常不自然的，并且容易使诸如自动代码缩进之类的事情变得混乱。

一个更好的解决方式是将这个函数封装到`do... while(0)` 结构里边。这个结构在结尾会需要一个分号，正好是我们想要的，使用 `while(0)` 确保循环不会重复执行，同时循环里的内容只执行一次。

这个宏定义了一个变量，叫 `now` ，这个变量名对于宏定义里的变量来说是一个不太好的选择，因为它可能会跟外部定义的变量名冲突。如果有这样的代码：

```text
NSTimeInterval now; // ivar
    
TIME("whatever", now);
```

这个宏就不能正常运行。

遗憾的是，在 C 语言并没有很好的方式去生成一个唯一的变量名，所以只能像是 Objective-C 里的类名定义一样，给这个变量加一个前缀。

```text
#define TIME(name, lastTimeVariable) \\
do { \\
	NSTimeInterval MA_now = [[NSProcessInfo processInfo] systemUptime]; \\
	if(lastTimeVariable) \\
		NSLog(@"%s : %f seconds", name , MA_now - lastTimeVariable);\\
	lastTimeVariable = MA_now; \\
} while(0)
```

这个宏现在对于意外的冲突来说比较安全了。

### 字符串拼接

这个特性严格来说不是宏的一部分，但是对于构建宏是很有用的，所以值得被提起。C 语言一个比较不常用的特性是，如果你在源代码中将两个字符串字面量一个接一个的放在一起，这两个字符串会被拼接在一起：

```text
char *helloworld = "hello, " "world!";
// 也就等于 "hello, world!"
```

对于 Objective-C 中的代码也是一样的：

```text
NSString *helloworld = @"hello, " @"world!";
```

更神奇的是，如果你有一个 Objective-C 的字符串，并将一个 C 字符串放在它后边，会得到一个拼接后的 Objective-C 字符串：

```text
NSString *helloworld = @"hello, " "world!";
```

你可以使用这个特性来拼接宏的参数：

```text
#define COM_URL(domain) [NSURL URLWithString: @"<http://www>." domain ".com"];

COM_URL("google"); // <http://www.google.com>
COM_URL("apple"); // <http://www.apple.com>
```

### 字符串化

通过将一个`#`放在宏的参数前，预处理器会把这个参数的文本内容转为一个 C 字符串，比如：

```text
#define TEST(condition) \\
	do { \\
		if(!(condition)) { \\
			NSLog(@"Failed test: %s", #condition); \\
		} \\
	} while(0)

TEST(1 == 2);
// log: Failed test: 1 == 2
```

但是，在使用这个特性的时候需要注意。如果参数里包含一个宏的话，这个宏不会被展开，比如：

```text
#define WITHIN(x, y, delta) (fabs((x) - (y)) < delta)

TEST(WITHIN(1.1, 1.2, 0.05));
// log: Failed test: WITHIN(1.1, 1.2, 0.05)
```

有些时候这个是符合预期的，但是有的时候是不符合预期的，为了避免这种情况，你可以增加一个间接级别：

```text
// 通过这种方式 x 如果是一个宏的话也可以正常展开，并且字符串化
#define STRINGIFY(x) #x 

#define TEST(condition) \\
	do { \\
		if(!(condition)) { \\
			NSLog(@"Failed test: %s", STRINGIFY(condition)); \\
		} \\
	} while(0)

TEST(WITHIN(1.1, 1.2, 0.05));

do {
	if (!WITHIN(1.1, 1.2, 0.05)) {
		NSLog(@"Failed test: %s", STRINGIFY(WITHIN(1.1, 1.2, 0.05)));
	}
}

do {
	if (!(fabs((1.1) - (1.2)) < 0.05)   ) {
		NSLog(@"Failed test: %s", STRINGIFY(WITHIN(1.1, 1.2, 0.05)));
	}
}

do {
	if (!(fabs((1.1) - (1.2)) < 0.05)   ) {
		NSLog(@"Failed test: %s", STRINGIFY((fabs((1.1) - (1.2)) < 0.05)));
	}
}

do {
	if (!(fabs((1.1) - (1.2)) < 0.05)   ) {
		NSLog(@"Failed test: %s", "(fabs((1.1) - (1.2)) < 0.05))");
	}
}
```

### token 粘贴

预处理器提供了一个 `##` 操作符来讲 token 拼接到一起。这个操作符允许你在宏中创建多个相关的项来减少冗余。通过 `a ## b` 可以创建一个 token `ab`。如果 a 或 b 是宏参数的话，`ab` 的值会被替换成宏参数。一个没有什么用处的例子是：

```text
#define NSify(x) NS ## x

NSify(String) *s; // 即 NSString *s;
```

你还可以使用这个操作符来生成方法组合。下面这个宏将自动为NSMutableArray支持的属性生成索引访问器，以实现键值合规性：

```text
#define ARRAY_ACCESSORS(capsname, lowername) \\
	- (NSUInterger)countOf ## capsname { \\
		return [lownername count]; \\
	} \\
	- (id)objectIn ## capsname ## AtIndex:(NSUInteger)index { \\
		return [lowername objectAtIndex: index]; \\
	} \\
	- (void)insertObject:(id)obj in ## capsname ## AtIndex:(NSUInterger)index { \\
		[lowername insertObject: obj atIndext: index]; \\
	} \\
	- (void)removeObjectFrom ## capsname ## AtIndex:(NSUInterger)index { \\
		[lowername removeObjectAtIndex: index]; \\
	}
```

可以通过这种方式使用这个宏：

```text
// instance variable
NSMutableArray *thingies;

// in @implementation
ARRAY_ACCESSORS(Thingies, thingies)
```

请注意，宏如何采用两个几乎相同的参数。 这是因为使用所有小写字母命名实例变量是惯例，但是方法名称要求键名以大写字母开头。 为了能够引用这两种情况，宏必须将它们都作为参数。 （不幸的是，没有C预处理操作符可以大写或小写一个token！）

就像 `stringify` 操作符一样，这个链接操作符的参数如果是宏的话，这个宏将不会被展开，所以我们还需要一个额外的间接级别。

```text
#define ARRAY_NAME thingies
#define ARRAY_NAME_CAPS Thingies

// incorrectly creates accessors for "ARRAY_NAME_CAPS"
ARRAY_ACCESSOR(ARRAY_NAME, ARRAY_NAME_CAPS)

#define CONCAT(x, y) x ## y
```

与stringify一样，哪种行为最理想，取决于个人情况。

### 可变参数列表

加入我们想要写这样的一个宏，这个宏只会在一个变量有值的时候打印它：

```text
#define LOG(string) \\
	do { \\
		if(gLoggingEnable) \\ 
			NSLog(@"Conditional log: %s", string); \\
	} while(0)
```

可以像下面这种方式一样使用这个宏：

```text
LOG("hello");
// 如果 gLoggingEnable 为 YES 的话，会打印出
// Conditional log: hello
```

这个宏很方便，但是还可以更方便一点。`NSLog` 会取一个格式化的字符串跟一个可变列表，如果 `LOG` 宏能像 `NSLog` 一样就更好了：

```text
LOG("count: %d  name: %s", count, name);
```

对于一开始创建的这个宏的初始的版本，像这么写的话会导致错误，因为它只能接受一个参数。

如果在宏的参数列表最后放上一个 `...` 参数的话，这个宏就可以接受可变数量的参数。如果你在宏的实现里使用 `__VA_ARGS__` 标记的话，在宏被展开时，这个标记会被替换成以逗号分隔的可变参数列表中的项。因此我们的 LOG 宏可以这样改造：

```text
#define LOG(...) \\
	do { \\
		if(gLoggingEnable) \\ 
			NSLog(@"Conditional log: %s", __VA_ARGS__); \\
	} while(0)
```

尽管目前这个宏是可以使用可变列表了，但是如果我们可以为这个宏提供一个固定的参数会更有用。例如，您可能想在自定义日志记录之后以及之前放置一些样板文本：

```text
#define LOG(fmt, ...) \\
	do { \\
		if(gLoggingEnabled) \\
			NSLog(@"Conditional log: ---" fmt "---", __VA_ARGS__); \\
    } while(0)
```

这个宏除了一个问题之外，目前工作的很好。这个问题就是，你不能仅仅为这个宏提供一个字符串来打日志。如果你这样使用这个宏的话 `LOG("Hello")`，这个宏会被展开成这样：

```text
NSLog(@"Conditional log: --- " "hello" " ---", );
```

中间的逗号将造成一个语法错误。幸运的是，gcc/clang 提供了一个扩展，这个扩展允许更自然的定义。通过将 `##` 操作符放在尾部的逗号与 `__VA_ARGS__` 中间，预处理器会在可变参数没有提供的时候，去除掉尾部的这个逗号。

```text
#define LOG(fmt, ...) \\
	do { \\
		if(gLoggingEnabled) \\
			NSLog(@"Conditional log: ---" fmt "---", ## __VA_ARGS__); \\
    } while(0)
```

### 一些魔法标志符

C 语言提供了一些内置的标志符，这些标志符在构建宏的时候会比较有用：

* **LINE**：会被展开成当前的行号。
* **FILE**：另一个内置宏，可扩展为包含当前源文件名称的字符串文字。
* **func**：当前函数名的c 字符串表示。

如果我们这样定义一个 log 宏：

```text
#define LOG(fmt, ...) NSLog(fmt, ## __VA_ARGS__)
```

看起来没什么意义。但是如果我们扩展一下，这样定义这个 log 宏：

```text
#define LOG(fmt, ...) NSLog(@"%s:%d (%s): ", fmt, __FILE__, __LINE__, __func__, ## __VA_ARGS__)
```

就会有用很多，你可以写这样的 log：

```text
LOG("something happened");
```

它会被展开成：

```text
MyFile.m:42 (MyFunction): something happened
```

这是非常有价值的调试辅助工具。 您可以在整个代码中散布LOG语句，日志输出将自动包含放置每个日志语句的文件名，行号和函数名。

### 复合字面量（Compound Literals）

这个是另一个不属于宏的一部分，但是对于构建宏很有用的东西。复合文字是 C99 的一项新特性。这个特性使你可以创建任何类型的字面量（即给定值的常量表达式，例如42或“ hello”），而不仅仅是内置类型。

复合字面量的语法有些奇怪，但是并不难，大概长这样：

```text
(type){ intializer }
```

举个栗子，下面的这些定义是完全一样的：

```text
NSPoint p = {1, 100};
DoSomething(p);

// 复合字面量
DoSomething((NSPoint) {1, 100});
```

作为一个更有用的例子，你可以在行内创建一个array：

```text
NSArray *array = [NSArray arrayWithObjects: (id []){ @"one", @"two", @"three" } count: 3];
```

可以使用一个宏来表示：

```text
#define ARRAY(num, ...) [NSArray arrayWithObjects: (id []){ __VA_ARGS__ } count: num]
```

就可以这么写了：

```text
NSArray *array = ARRAY(3, @"one", @"two", @"three");
```

当然，手动输入数组项的个数是很烦人的，更好的情况下应该避免写项个数。好消息是这是很容易实现的。

你可能已经知道，`sizeof` 操作符会返回给你某个特定对象或者类型的 bytes 大小。当给定了一个 array 的时候，通过`sizeof` 可以获取到整个 array 的大小，然后再使用这个大小除以单个项的大小就能得到这个数组的个数：

```text
#define ARRAY(...) [NSArray arrayWithObjects:(id[]){ __VA_ARGS__ } count:sizeof((id []){ __VA_ARGS__ }) / sizeof(id)];
```

这样我们就可以不用写数字，这样来创建一个数组：

```text
NSArray *array = ARRAY(@"one", @"two", @"three");
```

不错，但是还可以进行一些优化调整，现在的宏定义字符数太多了，而且有冗余，最好是将这个宏重构成一些可以共用的部分：

```text
#define IDARRAY(...) (id []){__VA_ARGS__}
#define IDCOUNT(...) (sizeof(IDARRAY(__VA__ARGS)) / sizeof(id))

#define ARRAY(...) [NSArray arrayWithObjects: IDARRAY(__VA_ARGS__) count: IDCOUNT(__VA_ARGS__)]
```

这个宏现在是非常好用，而且也容易被理解。

我们可以对 `NSDictionary` 也提供一个类似的宏。`NSDictionary` 并没有一个相应的方法，所以我们需要让这个宏来调用一个小的辅助方法来做一些事情：

```text
#define DICT(...) DictionaryWithIDArray(IDARRAY(__VA_ARGS__), IDCOUNT(__VA_ARGS__) / 2)
```

辅助函数将对象数组中的项解包出来然后通过 `NSDictionary` 来创建 dictionary：

```text
NSDictionary *DictionaryWithIDArray(id *array, NSUInteger count)
{
	id keys[count];
	id objs[count];

	for(NSUInterger i = 0; i < count; i++) {
		key[i] = array[i * 2];
		objs[i] = array[i * 2 + 1];
	}
	return [NSDictionary dictionaryWithObjects:objs forKeys:keys count:count];
}
```

这样你就可以这样写一个字典：

```text
NSDictionary *d = DICT(@"key", @"value", @"key2", @"value2");
```

### typeof

`typeof` 是一个 gcc 的扩展，并不是 C 语言标准的一部分，但是这个扩展是非常有用的。`typeof` 就像 `sizeof` 一样，会返回给定参数的类型。如果你把一个表达式给 `typeof`，`typeof` 会计算出这个表达式的值。如果你把一个类型给 `typeof`，`typeof` 仅仅返回这个 type 本身。

为了最大的可移植性，最好是使用`__typeof__`的写法。为了避免冲突， typeof 关键词在一些 gcc 的模式下是不允许使用的。

让我们看一下我们一开始写的那个有缺陷的 MAX 宏：

```text
#define MAX(x, y) ((x) > (y) ? (x) : (y))
```

你可能想起来，这个写法是有缺陷的，因为它可能会将一个参数计算两次。我们可以使用 block 以及一些局部变量来尝试修复这个问题。

```text
#define MAX(x, y) (^{ \\
	int my_localx = (x); \\
	int my_localy = (y); \\
	return my_localx > my_localy ? (my_localx) : (my_localy); \\
}())
```

这个方法可以运行，但是问题是这个宏只能对 `int` 使用，对于 `float`，`long long`或者其他类型就不适用了。

使用`__typeof__`，这个宏可以是完全泛型的。

```text
#define MAX(x, y) (^{ \\
	__typeof__(x) my_localx = (x); \\
	__typeof__(y) my_localy = (y); \\
	return my_localx > my_localy ? (my_localx) : (my_localy); \\
}())
```

这个版本像我们预期的一样工作。值得注意的是，**typeof** 是一个纯编译期的结构，所以并不会导致宏的参数被计算两次。你可以使用这个小技巧来创建一个指针指向任何你想要的值：

```text
#define POINTERIZE(x) ((__typeof__(x) []) {x})
```

尽管这个宏单独使用的时候可能没啥用，但是却可以为创建其他宏提供帮助。比如，这是一个可以把任何类型的值包装成一个 `NSValue` 对象的宏：

```text
#define BOX(x) [NSValue valueWithBytes:POINTERIZE(x) objcType:@encode(__typeof(x))]
```

### 内置函数

gcc 提供了两个对构建宏很有用的内置方法。

第一个是 `__builtin_types_compatible_p` ，你可以将两个type（使用`__typeof__`做这个事情很方便）参数传递给这个函数，如果这两个类型是兼容的（campatible，大概就是说这两个类型是相等的） 的，函数返回值会是 1，否则的话会返回 0。

第二个是 `__builtin_choose_expr`，这个函数就像 C 语言的 `?:`一样，除了谓词必须是编译时常量，整个函数表达式的类型会是最终被选中的分支的类型，两个分支可能是不同类型歪。

这就可以让你写出可以根据参数类型的不同执行不同事情的宏。比如说，下面就是一个宏，整个宏将一个表达式转为 `NSString` ，并且使输出的结果尽量有用：

```text
//  make the compiler treate x as the given type on matter what
#define FORCETYPE(x, type) *(type *)(__typeof__(x) []){ x }

#define STRINGIFY(x) \\
	__builtin_choose_expr( \\
		__builtin_types_compatible_p(__typeof__(x), NSRect), \\
			NSStringFromRect(FORCETYPE(x, NSRect)), \\
		\\
	__builtin_choose_expr( \\
		__builtin_type_campatible_p(__typeof__(x), NSSize), \\
			NSStringFromeSize(FORCETYPE(x, NSSize)), \\
		\\
	__builtin_choose_expr( \\
    __builtin_types_compatible_p(__typeof__(x), NSPoint), \\
      NSStringFromPoint(FORCETYPE(x, NSPoint)), \\
    \\
  __builtin_choose_expr( \\
    __builtin_types_compatible_p(__typeof__(x), SEL), \\
      NSStringFromSelector(FORCETYPE(x, SEL)), \\
    \\
  __builtin_choose_expr( \\
    __builtin_types_compatible_p(__typeof__(x), NSRange), \\
      NSStringFromRange(FORCETYPE(x, NSRange)), \\
    \\
    [NSValue valueWithBytes: (__typeof__(x) []){ x } objCType: @encode(__typeof__(x))] \\
  )))))
```

注意FORCETYPE宏。 即使在编译时选择了要遵循的代码分支，未使用的分支仍然必须是有效的代码。 即使永远不会选择该分支，编译器也不接受NSStringFromRect（42）。 通过指针化值，然后在取消引用值之前对其进行转换，可以确保代码可以编译。 强制转换对于除已采取的一个分支之外的所有事物均无效，但对于其他任何分支均无效。

### X-Macros

这是一些我不会用到的宏，但是它们非常有意思。这是我从未使用过的东西，X-Macros是一种根据另一个宏来定义宏的方法，然后将其重新定义多次以赋予该宏新的含义。 这很令人困惑，所以这是一个例子：

```text
#define MY_ENUM \\ 
	MY_ENUM_MEMBER(kStop) \\
	MY_ENUM_MEMBER(kGo) \\
	MY_ENUM_MEMBER(kYield)

enum MYEnum {
		#define MY_ENUM_MEMBER(x) x,
		MY_ENUM
		#undef MY_ENUM_MEMBER
}

// stringification
const char *MyEnumToString(enum MyEnum value) 
{
	#define MY_ENUM_MEMBER(x) if(value == (x)) return #x;
	MY_ENUM
	#undef MY_ENUM_MEMBER
}

enum MyEnum MyEnumFromString(const char *str) 
{
		#define MY_ENUM_MEMBER(x) if(strcmp(str, #x) == 0) return x;
		MY_EUNM
		#undef MY_ENUM_MEMBER
}
```

这是一种先进且令人恐惧的技术，但在某些特定情况下，它可以帮助消除许多无聊的重复。 有关X宏的更多信息，请查阅Wikipedia文章。

## 宏与 Swift

Swift会自动将使用`#define`指令声明对象宏作为全局常量导入。

比如我们有以下的宏定义：

```text
#define FADE_ANIMATION_DURATION 0.35
#define VERSION_STRING "2.2.10.0a"
#define MAX_RESOLUTION 1268
#define HALF_RESOLUTION (MAX_RESOLUTION / 2)
#define IS_HIGH_RES (MAX_RESOLUTION > 1024)
```

当导入到Swift中时，以上示例中的宏等效于以下常量声明：

```text
let FADE_ANIMATION_DURATION = 0.35
let VERSION_STRING = "2.2.10.0a"
let MAX_RESOLUTION = 1268
let HALF_RESOLUTION = 634
let IS_HIGH_RES = true
```

对于比较复杂的宏定义，官方推荐我们使用函数、泛型或其他 Swift 语言的特性而不是宏来完成相应的功能。

