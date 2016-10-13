# Demo-SegmentControl
今年的十一过地好爽, 七天假期, 五个大晴天, 老家在河南东北角, 每年这个时候理应穿薄毛衣了, 今年依然是短袖, 我这十几年的鼻炎患者, 总是受不得一点点凉气, 尤其是季节转换的时候, 所以今年的十一格外舒服.

扯完, 回来.

十一前几天答应同事回来谈谈iOS如何设计自定view的事情. 回来也该总结一下了.

从类的构造方法说起.
自从OC开始支持NS_DESIGNATED_INITIALIZER以后呢, 就开始适应了这种写法, 每遇到一个别人定义的类, 总是第一眼先去找有没有定义NS_DESIGNATED_INITIALIZER, 找到这个才安心. NS_DESIGNATED_INITIALIZER是指定构造方法做为默认的初始化方法的一个宏, 没有其他的作用, 就是指定正确姿势的初始化方法的. 如果用NS_DESIGNATED_INITIALIZER以外的构造方法可以不可以呢? 可以的, 但是不能保证初始化正常. 这是这个宏初衷. 那么, 既然可以指定亲生的构造方法, 那可不可以指定那些构造方法不可以用呢? 可以的, NS_UNAVAILABLE.

Like this:

- (instancetype)init NS_UNAVAILABLE;

- (instancetype)initWithFrame:(CGRect)frame NS_UNAVAILABLE;

- (instancetype)initWithCoder:(NSCoder *)aDecoder NS_DESIGNATED_INITIALIZER;

- (instancetype __nullable)initWithItem:(NSArray <NSString *> * __nonnull)item NS_DESIGNATED_INITIALIZER;


当别人使用你自定义的View的时候, 只能通过initWithItem来初始化, init会编译不通过的. 这样做得好处就是, 一个宏省略了千言万语, 再也不需要在每个自定义view的开头写上你的默认的初始化方法了, 因此千言万语一点也不夸张, 别人拿来就能创建, 不用看注释, 多么优雅的一个宏.

当然这个宏也有需要注意的地方:
1. 子类必须重写父类中标记了NS_DESIGNATED_INITIALIZER的构造方法,
2. 子类的没有NS_DESIGNATED_INITIALIZER标记的构造方法内部必须调用子类自己的默认构造方法.
3. 子类指定的构造方法内部必须调用父类的指定构造方法. 虽然看着有点绕, 但是写两次就完全记住了, 没记住也没有关系, xcode会有烦人的黄色⚠️.

说完入口说说内部设计, 然后顺带提一下布局, 这里就只说主要的地方:

1. 布局在layoutsubviews, 避免重写setFrame, layoutsubviews中获取子空间的Frame并不能保证百分之百的正确, 例如, 屏幕旋转之后, 第一次调用LayoutSubviews的时候, 用AutoLayout来设置的View, 这里获取的Frame是不正确的, 这应该是iOS自己的问题.

2. 重写sizeThatFits, 返回你认为的最佳尺寸

3. 既然重写了sizeThatFits, 那么sizeToFit也一起重写了吧, 如果开发的时候我看到了重写了这两个方法的自定义view, 我都会有些小感动的, 太贴心了, 有木有

4. intrinsicContentSize 记得重写
5. 私有成员变量声明在匿名分类中
6. 私有方法用下划线开头, 这个虽然不强制要求, 但是如果这么写了, 一眼望过去, 就能分辨出是私有还是公开的的方法
7. AutoLayout关系尽量少, 尽量只设置一次, Autolayout 都可以新开一篇了, 准备下次填这个坑, 其中的技巧真是太多太多
8. LayoutSubviews的方法中避免更新约束和更新布局:[self setNeedsUpdateConstraints]; [self updateConstraintsIfNeeded]; [self setNeedsLayout]; [self layoutIfNeeded]; . 这个可以iOS7版本试一下, 妥妥的死循环, iOS8以上虽然是不会了, 但仍然是心有余悸, 万一哪天iOS20又不行了呢
9. 重写setter方法的, 用懒加载初始化的成员对象, 用null_resettable修饰. 一个null_resettable又胜过了千言万语
10.容器一定要声明其中存放的类型, 不管是参数, 返回值, 还是成员对象, 每当看到这么写的代码, 我都会默默点赞, 大家都省心, 省时
11. nullable 和 nonnull nullable nonnull 要好好利用, 马上转Swift也是必备的

以上几点, 本来以为10分钟讲完, 就能手工, 结果说了半个小时…

最后又跟大家说了说IB_DESIGNABLE 和 IBInspectable的用法. 应该是一个偶尔用用可以, 没什么大用处的Skill.

这样就结束了.
