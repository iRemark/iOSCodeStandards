## 老板娘/铃铛iOS代码规范##

> 随着代码量的增加和项目的增大以及项目合理或不合理的迭代，代码会显得冗余和杂乱。良好的代码规范和和规则以及设计良好的封装对项目的迭代扩展以及程序员本身无疑是一件好事，因此，在这里作一些简单的代码规范规则的说明，并希望大家遵守和扩展此文档。

- **[使用属性](#使用属性_ref_id)**

- **[ARC中属性的省略写法](#ARC中属性的省略写法_ref_id)**

- **[属性的统一写法](#属性的统一写法_ref_id)**

- **[属性的一些使用技巧](#属性的一些使用技巧_ref_id)**

- **[驼峰命名法](#驼峰命名法_ref_id)**

- **[类中属性/变量的多余写法](#类中属性/变量的多余写法_ref_id)**

- **[方法的命名规则](#方法的命名规则_ref_id)**

- **[方法的书写和注释](#方法的书写和注释_ref_id)**

- **[扩展类的方法命名注意事项](#扩展类的方法命名注意事项_ref_id)**

- **[类的命名规范](#类的命名规范_ref_id)**

- **[枚举写法](#枚举写法_ref_id)**

- **[两种block的常规写法](#两种block的常规写法_ref_id)**

- **[少用tag值](#少用tag值_ref_id)**

- **[统一模型层](#统一模型层_ref_id)**

- **[Controller命名可去掉View](#Controller命名可去掉View_ref_id)**

- **[慎用通知](#慎用通知_ref_id)**

- **[表单数据的抓取](#表单数据的抓取_ref_id)**

  ----


-  <span id="使用属性_ref_id">使用属性</span>

  从C++的官方推荐和Java Bean规范来讲，都是把变量私有化，对外提供getter和setter方法，这就是我们常说的“属性”，对于Objective-C的优雅的属性语法，我们更应该在可以使用变量和属性的选择中优先选择属性，即：

  不要使用

  ```objective-c
  @interface MyObject : NSObject {
  	NSString *name;
      ...
  }
  @end
  ```

  应该使用

  ```objective-c
  @interface MyObject : NSObject
    
  @property (nonatomic, copy) NSString *name;
  @end
  ```

  使用这种命令方式的优点不仅仅在于统一了代码规范，而且更重要的是由于提供了setter/getter方法，可以更好的保护数据，以及在set/get的时候作额外的操作，并且使用@property的形式方便运行时捕捉到属性信息以便作底层次的处理。   

  但是，可能有人问使用变量会不会比使用属性效率要高？回答：很可能是，所以你写项目全用汇编或机器语言写好了，不要考虑其他语言。为了这一点点所谓的效率而失掉优雅的代码特性是错误的做法。而且可以负责任的说，真正的coding高手眼中的效率问题绝不是这种浅薄的弃掉属性用变量换取执行效率的问题。

- <span id="ARC中属性的省略写法_ref_id">ARC中属性的省略写法</span>

  ARC自从推出至今已很多年，我们的项目一律采用ARC内存管理方式处理。  

  在ARC中属性默认为strong，这个字段可以省略不写，即：  

  不要使用：

  ```objective-c
  @property (nonatomic, strong) Person *person;
  ```

  建议使用：

  ```objective-c
  @property (nonatomic) Person *person;
  ```

  对于基本类型同样可以省略assign，因为对于基本数据类型，不存在什么strong或retain:

  不要使用：

  ```objective-c
  @property (nonatomic, assign) NSInteger age;
  ```

  建议使用：

  ```objective-c
  @property (nonatomic) NSInteger age;
  ```

  在一些第三方库的代码里，可能见到省略的或不省略的写法，在这里，咱们只是为了统一规范而已，无用与不用的理由。

- <span id="属性的统一写法_ref_id">属性的统一写法</span>

  ```objective-c
  @property (nonatomic) Person *person;
  ```

  @property的括号间有间隔，nonatomic和strong间有间隔，指针*号靠近对象而不是靠近类。

  只是为了清晰代码和规范，无用与不用的理由。

- <span id="属性的一些使用技巧_ref_id">属性的一些使用技巧</span>

  在自己封装一些类库时，属性的写法有些注意事项：

  - 不必要提供出去的属性写在.m中，形如：

    ```objective-c
    // MyClass.m
    @interface MyClass()

    @property (nonatomic) SomeObject *obj; // 既然没必要提供出去，就把它私有化，外界不让访问
    @end

    @implementation MyClass
    ...
    @end
    ```

  - 提供出去的BOOL类型的属性，其getter方法写为：isXxx, 形如：

    ```objective-c
    @property (nonatomic, getter=isValid) BOOL valid; 
    // 这样调用set方法为：obj.valid = YES; 
    // 调用get方法为：BOOL isValid = obj.isValid;
    ```

    这种方式更贴近人性化，我们看Apple的官方类，对于这类BOOL类型的属性，也是如此，比如：

    ```objective-c
    // UIKit/UIControl.h
    @property(nonatomic,getter=isEnabled) BOOL enabled;
    @property(nonatomic,getter=isSelected) BOOL selected;
    ```

  - 一些属性在外部只读，但在内部需要set的做法：

    ```objective-c
    // MyClass.h
    @interface MyClass : NSObject

    @property (nonatomic, readonly) SomeObject *obj; // 对外只读
    @end

    // MyClass.m
    @interface MyClass()

    @property (nonatomic, readwrite) SomeObject *obj;
    @end

    @implementation MyClass
    - (void)myMethod {
        self.obj = [[SomeObject alloc] init]; // 在内部可以赋值
    }
    @end

    ```

    ​

- <span id="驼峰命名法_ref_id">驼峰命名法</span>

  驼峰命名法Camel-Case从早期的lower-case命名法发展而来，形如“好学生”的lower-case命名法为：good_student, 其驼峰命名法为：goodStudent, 这些都被大家所习惯和熟识，在OC中，类名第一个字母必须大写，变量第一个字母小写等规则不必多说，但值得一提的是

  不要使用：

  ```objective-c
  Student *gS;
  ```

  建议使用：

  ```objective-c
  Student *goodStudent;
  ```

  在像Objective-C类似的这种高级语言中，绝不建议过分的省略单词的写法，如果写成gS, 只有你和鬼知道是什么意思，多打一点字清晰一些不是什么坏事。有位老师在教授PHP的课堂中曾说到一个学生起的变量名为：fdckfs, 后来一问才知道这个意思是”房地产开发商“。

- <span id="类中属性/变量的多余写法_ref_id">类中属性/变量的多余写法</span>

  比如学生类：Student, 有两个属性：姓名和年龄

  不要使用：

  ```objective-c
  @property (nonatomic, copy) NSString *studentName;
  @property (nonatomic) NSInteger studentAge;
  ```

  建议使用：

  ```objective-c
  @property (nonatomic, copy) NSString *name;
  @property (nonatomic) NSInteger age;
  ```

  适当想一下就可以知道，在学生类里，name就是学生的名字，没必要再强调用studentName表示学生姓名，在Student里用studentName表示学生姓名是一种完全多余的写法。

- <span id="方法的命名规则_ref_id">方法的命名规则</span>

  不必多说，仿官方形式：

  ```objective-c
  - (void)viewDidLoad {
  	// ...
  }
  ```

  -或+距扩号有一个空格，方法名驼峰写法，首字母一律小写。

- <span id="方法的书写和注释_ref_id">方法的书写和注释</span>

  -方法和+方法，-/+符合和括号留空隔，方法体左括号靠近代码书写，for循环;内留空隔，形如：

  ```objective-c
  - (void)viewDidLoad {
  	// ...
     	for (NSInteger i = 0; i < 100; i++) {
  		// ...
     	}
  }
  ```

  其实和官方API写法一致而已，养成习惯，统一起来不致看着乱。

  注释采取Apple推荐注释，形如：

  ```objective-c
  /**
   *  @param keyValues 字典(可以是NSDictionary、NSData、NSString)
   *  @cls 即data所对应的类，如果传为nil, 则不对data进行解析
   *  @return 新建的CommonModel对象
   */
  + (instancetype)commonModelWithKeyValues:(id)keyValues dataClass:(Class)cls;
  ```

  对于//注释，//符合和注释文字间留出一个空隔, 这也是官方的写法，形如：

  ```objective-c
  [obj testMethod]; // 这是注释文字，和//符合留一空格
  ```

- <span id="扩展类的方法命名注意事项_ref_id">扩展类的方法命名注意事项</span>

  我们看一些第三方库，比如SDWebImage, MJExtension这些比较特殊，它们很多是扩展类Category，比如SDWebImage中有对UIImageView的扩展，MJExtension有对NSObject的扩展。拿SDWebImage中的UIImageView+WebCache.h中的方法声明中示例：

  ```objective-c
  - (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder;
  ```

  它的命名方式为什么加了前缀sd_, 这不符合一般的方法命名规范，其实这主要是为了防止有一天系统给UIImageView也加了setImageWithURL:placeholderImage:或是为了防止其他第三方库也实现了setImageWithURL:placeholderImage:这个方法。

  这种事情的发生并不是什么巧合，比如我在AFNetworking的UIImageView+AFNetworking中也发现了类似的代码：

  ```
  - (void)setImageWithURL:(NSURL *)url placeholderImage:(nullable UIImage *)placeholderImage;
  ```

  如果SDWebImage没有加前缀，那么这两个方法的方法签名就都是：`setImageWithURL:placeholderImage:`

  试想，在这两个库中，这个方法都是对UIImageView的Category，如果SDWebImage真的没有对方法加sd_前缀(早版本的SDWebImage确实没有加前缀)，在项目中引入SDWebImage这个库的同时又引入了AFNetworking, 会怎么样？这就会出现苹果官网上说的“Undefined problem”未知问题。

  启示：如果我们要封装自己的Category, 方法名也要加自己的统一前缀。因为Category不同于继承，对于继承出来的两个类，其对象调用相同的实例方法不会有什么问题，但Category不一样，比如上面对imageView如果调用相同的方法，编译器就无法识别了。

- <span id="类的命名规范_ref_id">类的命名规范</span>

  在传统的C++语言、高级的Java、C#语言中为了解决类名的重复问题采用了"命名空间"或"package"的方法来解决，大致意思是说在命名空间UI下的Button类和命名空间LB(联璧)下的Button类以及其他命名空间下的Button类不是同一个类，不同命名空间下可以有相同的类名，这好比上海市的“小明”和苏州市的“小明”不是一个小明一样，但可惜的是，OC并没有这样的机制，因此取而代之的是在类名前加前缀，随着一门语言体系的增大，我个人认为这种在类名加前缀的做法并不是非常的好，但好在Apple给我们封装的类都很经典和强悍，在类名前加前缀也使用这些类的用处明显。所以毫无疑问，我们的项目也要保持这种习惯。

  按照传统的MVC模式来分，统一起见类名前加前缀如下：

  ```objective-c
  Model: LB...
  View: LV...
  Controller: LC...
  ```
  不过，为了减轻这种繁锁，也可以在一个项目中统一，比如无论模型视图还是控制器，都使用LB开头。

- <span id="使用属性_ref_id">枚举写法</span>

  OC自定义了枚举的写法，可以很好的声明枚举的类型，比如枚举类型为NSInteger, 所以

  不建议使用：

  ```objective-c
  typedef enum {
      LVStyleActionSheet = 0,
      LVStyleAlert
  } LVStyle;
  ```

  建议使用：

  ```objective-c
  typedef NS_ENUM(NSInteger, LVStyle) {
      LVStyleActionSheet = 0,
      LVStyleAlert
  };
  ```

- <span id="两种block的常规写法_ref_id">两种block的常规写法</span>

  "返回值为void，没有跟参数的block类型"，Apple已经为我们定义好了:

  ```objective-c
  typedef void (^dispatch_block_t)(void);
  ```

  为了统一起见，关于这种block类型，项目中也应该用dispatch_block_t，因为常用的原因，Apple写好了这个类型，另外还有一种block类型常用："返回值为void，跟一个参数的block类型"，所以我们可以把这两种常用的block类型写进pch文件，形如：

  ```objective-c
  typedef dispatch_block_t BlockVoidType;  // 只是为了向驼峰命名靠近，重新typedef系统类型
  typedef typedef void (^BlockIdType)(id); // 返回值为void, 跟一个id参数的block类型
  ```

- <span id="少用tag值_ref_id">少用tag值</span>

  项目中如果有多个类似的控件，很多情况可以用tag值区分，比如一个页面有10个按钮，每点击一个按钮把其中一个学生的信息传给其他控制器，它们触发的方法都是buttonClick: 那么为了区分是哪个按钮, 可以用tag值区分，代码形如：

  ```objective-c
  - (void)buttonClick:(UIButton *)button {
  	if (button.tag == 100) {
        	Student *student = self.studentArray[button.tag-100];
      	// ...
  	} else if (button.tag == 101) {
        	// ..
  	}
    	...
  }
  ```

  这种是极其糟糕的做法，这种写法除了自己很少有人知道tag为100代表的是什么，一般来说我们的做法有两种，一是封装自己的Button:

  ```objective-c
  @interface MyButton
  	@property (nonatomic) Student *student;
  	// ...
  @end
  ```

  但是如果项目已写成，大量的修改这种代码并不见得好，所以我们采用第二种方式：

  二是对Button进行扩展，通过动态关联让UIButton具有Student属性，针对这种需求，为了统一起见，在项目中我们对NSObject进行扩展给其附加一个对象：

  ```objective-c
  // NSObject+LBAttatchObject.h
  #import <Foundation/Foundation.h>

  @interface NSObject (LBAttatchObject)

  @property (nonatomic) id lbAttachObject;
  @end
  ```

  ```objective-c
  #import "NSObject+LBAttatchObject.h"
  #import <objc/runtime.h>

  @implementation NSObject (LBAttatchObject)

  - (id)lbAttachObject {
      return objc_getAssociatedObject(self, _cmd);
  }

  - (void)setLbAttachObject:(id)lbAttachObject {
      objc_setAssociatedObject(self, @selector(lbAttachObject), lbAttachObject, OBJC_ASSOCIATION_RETAIN);
  }

  @end
  ```

- <span id="统一模型层_ref_id">统一模型层</span>

  用AF从服务器上down下来数据之后，用MJExtension或其他第三方库的反射机制把数据转换成模型，尽量不要使用从字典中取key-value值，把数据对象化，为了便于处理，在MJ的基础上做CommonModel, 统一处理服务器返回的数据

  ```objective-c
  // LBCommonModel作为整个App的Model，不同点在于data字段
  @interface LBCommonModel : NSObject

  @property (nonatomic, copy) NSString *msg;
  @property (nonatomic) id data;
  @property (nonatomic) NSInteger code;
  @property (nonatomic, copy) NSString *totalAmt;

  /**
   *  @param keyValues 字典(可以是NSDictionary、NSData、NSString)
   *  @cls 即data所对应的类，如果传为nil, 则不对data进行解析
   *  @return 新建的CommonModel对象
   */
  + (instancetype)commonModelWithKeyValues:(id)keyValues dataClass:(Class)cls;
  @end
  ```

- <span id="Controller命名可去掉View_ref_id">Controller命名可去掉View</span>

  前面说过，控制器前缀字符：LC, 如果所有的控制器再加上ViewController, 显得有些长，比如：LCFinalcialViewController, 为了统一起见，即然是控制器，突出Controller, 不用带View, 即命名控制器格式为：

  ```objective-c
  LCXxxController
  LCFinalcialViewController -> LCFinalcialController
  ```

- <span id="慎用通知_ref_id">慎用通知</span>

   通知NSNotificationCenter是KVO设计模式的重要实现之一，但对于一般的界面之间的回调等，少用通知，通知会导致系统的维护性很差，一旦通知用的多了，很难知道哪里注册了通知，哪里受通知的影响，这会变的杂乱。

   通知应当用在系统层次、全局层次的影响上，比如统一处理键盘遮挡输入框的问题，而不是用在简单的传值上面。为了更有效的管理通知，统一封装类LBNotificationManager来管理系统上所有的通知。形如：

   ```objective-c
   ace LBNotificationManaager : NSObject
     
   // 通知
   + (void)registNotifyOfDownloadSoftSuccess:(id)observer selector:(SEL)selector; // 注册通知
   + (void)postNotifyOfDownloadSoftSuccess:(id)object userInfo:(NSDictionary *)userInfo; // 发送通知
   + (void)removeNotifyOfDownloadSoftSuccess:(id)observer object:(id)obj; // 取消通知
   @end
   ```

- <span id="表单数据的抓取_ref_id">表单数据的抓取</span>

   在传统的web开发中，表单form提供给用户填写信息，当点击提交submit的时候，再把用户填写的表单数据提交给服务器，在iOS开发中，我们姑且认为诸如UITextField等控件为表单控件，当点击"注册、登录"等提交按钮时，我们通常做法是从这些控件中提取出值然后做处理，比如赋值给模型，过滤一些信息，针对这种需求，我们可以做些处理，点击提交时，让输入控件的值自动赋给模型，而不必每次再从控件中取值。

   这里有一个笨拙的方案：  [ZZBindDemo_2](https://github.com/ACommonChinese/ZZBindDemo_2)，但这种处理方式还是比较笨的，如果结合思路用RAC处理会好很多，推荐使用RAC解决这个问题，参照当前目录下 **BindDemo**

   ZZBindDemo_1/2使用起来比较烦琐，没有达到非常便捷的目的，需要大家共同想一个更好些的解决方案。

- Other...

   ------

   ​

> 编辑：
>
> 刘威振、许毓方
>
> 编辑软件：Typora
>
> 希望大家扩展
>
> 文件所在地址：http://172.16.103.166:9099/dashboard/projects



