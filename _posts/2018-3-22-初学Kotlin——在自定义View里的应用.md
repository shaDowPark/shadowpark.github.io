### 什么是Kotlin

`Kotlin`，它是`JetBrains`开发的基于JVM的面向对象的语言。2017年的时候被`Google`推荐`Android`的官方语言，同时`Android studio 3.0`正式支持这门语言，在这个编译器上创建一个`Kotlin`项目，非常方便，甚至可以`Java`转为`Kotlin`。

#### 我主要是在通过实现自定义`View`过程中,说一下`Kotlin`与`Java`的异同,其实两者非常相似
对`Kotlin`语法不是太了解的，可以先去看看它的**[官方翻译文档](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/GettingStarted/Basic-Syntax.html)**

### 以Barchart-Kotlin开始说起
**[Barchart-Kotlin](https://github.com/bmqb/Barchart-Kotlin)**是我用`Kotlin`写的一个简易灵活的柱状图库，喜欢的可以点个star！

![](https://github.com/shaDowZwy/Barchart-Kotlin/blob/master/image/demo.gif?raw=true)

#### 1.类的属性
在一个类里面我们需要定义一些属性来保存数据和状态

我们先来看看`Java`代码，在`BarChartView`定义了一些属性

```java
    private SpeedLinearLayoutManger mLayoutManager;
    private BarChartAdapter mAdapter;
    private ItemOnClickListener mClickListener;
    private int mDefaultWidth = 150;
```
然后我们再看看`Kotlin`是怎么定义这些属性的，下面的是`Kotlin`代码

```kotlin
    private lateinit var mLayoutManager: SpeedLinearLayoutManger
    private lateinit var mAdapter: BarChartAdapter
    private var mClickListener: ItemOnClickListener? = null
    private val mDefaultWidth = 150
```

你会发现不一样的声明方式，但重要的是`var`和`val`这两个关键字

`var`代表的是可变的变量，相当于现在`Java`声明变量的方式

`val`代表的是不可变的变量，初始化后不能再修改，相当于加了`final`关键字的变量

而且在`Kotlin`中属性是需要初始化的，没有值的时候你可以赋值`null`,不然编译会报错。加上`?`的意思是你不确定是否是这个类型，或者说是否为`null`。如果觉得实在是不方便你的使用逻辑，你可以使用这两种方式延迟初始化。

##### 懒初始化 by lazy
`lazy`是指推迟一个变量的初始化时机，只有在使用的时候才会去实例化它。适用于一个变量直到使用时才需要被初始化。在我这个项目里面没有使用`by lazy`,它大致的用法是这样的

```kotlin
val data: Data by lazy {
    Data(number,string)
   }

```

##### 延迟初始化 lateinit
`lateinit`是指你**保证**接下来在使用之前或者使用的时候会实例化它,不然你就会crash掉，这不就跟我们使用`Java`属性的方式一样么。。。它适用于一些`view`和必须用到数据结构的初始化，我觉得还是谨慎使用比较好。


#### 2.空安全
`Kotlin`可以说是分了两个大类型，可空类型和不可空类型，这样做的原因是它希望在编译阶段就把空指针这问题显式的检测出来，把问题留在了编译阶段，让程序更加健壮。它通过`?`来表达可为空。

```kotlin
mClickListener?.invoke1(position)
mClickListener?.invoke2(position)
mClickListener?.invoke3(position)
```
如果`mClickListener`为`null`的话，后面的语句是不会执行的。而且`Kotlin`提供了更加简洁的操作符`let`

```kotlin
mListener?.let {  
it.invoke1(position)
it.invoke2(position)
it.invoke3(position)  
}

```
只有在非空的情况下才执行`let`里面的操作，非常简洁。

#### 3.构造器
通过简单了解之后，我们开始写一个自定义View，我们需要继承`View`,`Java`的实现方式是这样的

```java
public class BarChart extends View {

    public BarChart(Context context) {
        super(context);
    }

    public BarChart(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public BarChart(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

```
用`Kotlin`你可以实现的更简洁

```kotlin
class BarChart @JvmOverloads constructor(context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0)
    : View(context, attrs, defStyleAttr) {
    
    private val mContext: Context = context

    init { }
```
你可以在`init`代码块里面获得构造函数的传参，当然你也可以直接在声明属性的时候获得，`@JvmOverloads` 如果你没有加上这个注解，它只能重载相匹配的的构造函数，而不是全部。

而且可能你也发现了，你可以在传参里面初始化，这相对于`Java`来说，灵活太多

```kotlin
fun shadow(width:Int=100,height:Int = 180){ }
//你可以这么使用
shadow()
shadow(140)
shadow(140,200)
```
#### 4.UI布局
一般用我们创造`view`的布局是`xml`,`Kotlin`也是支持的，但是它更推荐你使用`Anko`的布局方式。

那是什么是`Anko`呢,`Anko`是`JetBrains`开发的一个强大的库，它主要的目的是用来替代以前`xml`的方式来使用代码生成UI布局的，它包含了很多的非常有帮助的函数和属性来避免让你写很多的模版代码。

有兴趣的你可以去看看它的源码与更多使用方式 -- [Anko](https://github.com/Kotlin/anko)

首先你要在`Gradle`里添加`Anko`的引用

```gradle
// Anko Layouts
compile "org.jetbrains.anko:anko-recyclerview-v7:$anko_version"
compile "org.jetbrains.anko:anko-recyclerview-v7-coroutines:$anko_version"
compile "org.jetbrains.anko:anko-sdk25:$anko_version"
compile "org.jetbrains.anko:anko-appcompat-v7:$anko_version"
// Coroutine listeners for Anko Layouts
compile "org.jetbrains.anko:anko-sdk25-coroutines:$anko_version"
compile "org.jetbrains.anko:anko-appcompat-v7-coroutines:$anko_version"

```
然后你可以在代码里写UI布局的代码了，就是用`Kotlin`代码替代`xml`

```kotlin
 private fun createView(attrs: AttributeSet? = null, defStyleAttr: Int = 0) {

   var height = dip(150)
   attrs?.let {
            val typeArray = mContext.obtainStyledAttributes(it, R.styleable.BarChartView, defStyleAttr, 0)
            height = typeArray.getDimension(R.styleable.BarChartView_chart_height, dip(150).toFloat()).toInt()
            typeArray.recycle()
        }

    verticalLayout {
            lparams(width = matchParent, height = matchParent)

            frameLayout {
                lparams(width = matchParent, height = wrapContent)

                mLineView = view {
                    backgroundColor = R.color.gray_light
                }.lparams(width = matchParent, height = dip(0.5f)) {
                    gravity = Gravity.BOTTOM
                    bottomMargin = dip(9)
                }

                mBarView = recyclerView {
                    lparams(width = matchParent, height = height)
                }
            }

            mDateView = relativeLayout {
                lparams(width = matchParent, height = wrapContent) {
                    leftPadding = dip(10)
                    rightPadding = dip(10)
                }

                mLeftTv = textView {
                    textSize = 15f
                }

                mRightTv = textView {
                    textSize = 15f
                }.lparams(width = wrapContent, height = wrapContent) {
                    alignParentRight()
                }
            }
        }
    }

```

可以从代码里看见`verticalLayout`(其实就是`LinearLayout`的`vertical`模式)包裹了`frameLayout`和`relativeLayout`,里面又有各自的子`view`,而且你会发现`xml`有的属性这里也有，调用起来非常的简洁明了，熟练`xml`的来写这个，我觉得上手应该会很快，它的缺点就是没有预览效果，以及实现复杂的view结构的时候会比较繁琐，考验盲写的功力了。。。。

#### 5.扩展函数
扩展函数是Kotlin非常方便实用的一个功能，它可以让我们随意的扩展SDK的库，你如果觉得SDK的api不够用，这个时候你可以用扩展函数完全去自定义。

例如你需要这样来获取颜色，每次你都需要一个上下文`context`

```kotlin
mColor = ContextCompat.getColor(mContext, R.color.primary)
```
那你可以通过扩展`Context`这个SDK的类来实现更方便的使用

```kotlin
fun Context.color(colorRes: Int) = ContextCompat.getColor(this, colorRes)
fun View.color(colorRes: Int) = context.color(colorRes)
```
而且为了更方便在`View`里面使用，又扩展了`View`。在第一个方法里面可以发现`getColor`所需要的`this`上下文，就是`Context`。同样，`View`里面的`context`是`getContext()`所得到，也就是`View`里面本身具有的公有方法。

**这其实是很惊艳的功能**

那我们可以想一想为啥可以这样做，我们知道的是`Kotlin`和`Java`都是在`Jvm`上运行的，既然都是编译成`class`字节码，那我们是不是可以通过字节码来了解一些事情。

通过`Android studio 3.0`上的`Tools`的工具`Show Kotlin Bytecode`,可以将刚才的扩展函数的代码编译成字节码

```kotlin

  // access flags 0x19
  public final static color(Landroid/content/Context;I)I
    @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0
   L0
    ALOAD 0
    LDC "$receiver"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkParameterIsNotNull (Ljava/lang/Object;Ljava/lang/String;)V
   L1
    LINENUMBER 12 L1
    ALOAD 0
    ILOAD 1
    INVOKESTATIC android/support/v4/content/ContextCompat.getColor (Landroid/content/Context;I)I
    IRETURN
   L2
    LOCALVARIABLE $receiver Landroid/content/Context; L0 L2 0
    LOCALVARIABLE colorRes I L0 L2 1
    MAXSTACK = 2
    MAXLOCALS = 2

  // access flags 0x19
  public final static color(Landroid/view/View;I)I
    @Lorg/jetbrains/annotations/NotNull;() // invisible, parameter 0
   L0
    ALOAD 0
    LDC "$receiver"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkParameterIsNotNull (Ljava/lang/Object;Ljava/lang/String;)V
   L1
    LINENUMBER 14 L1
    ALOAD 0
    INVOKEVIRTUAL android/view/View.getContext ()Landroid/content/Context;
    DUP
    LDC "context"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkExpressionValueIsNotNull (Ljava/lang/Object;Ljava/lang/String;)V
    ILOAD 1
    INVOKESTATIC shadow/barchart/ExtensionsKt.color (Landroid/content/Context;I)I
    IRETURN
   L2
    LOCALVARIABLE $receiver Landroid/view/View; L0 L2 0
    LOCALVARIABLE colorRes I L0 L2 1
    MAXSTACK = 3
    MAXLOCALS = 2

```
这就是编译后的字节码，看不懂是吧，我也看不懂。。但是我们可以反编译啊，生成`Java`代码

```java
 public static final int color(@NotNull Context $receiver, int colorRes) {
      Intrinsics.checkParameterIsNotNull($receiver, "$receiver");
      return ContextCompat.getColor($receiver, colorRes);
   }

   public static final int color(@NotNull View $receiver, int colorRes) {
      Intrinsics.checkParameterIsNotNull($receiver, "$receiver");
      Context var10000 = $receiver.getContext();
      Intrinsics.checkExpressionValueIsNotNull(var10000, "context");
      return color(var10000, colorRes);
   }

```
通过反编译我们可以知道这是个静态函数，` Intrinsics.checkParameterIsNotNull($receiver, "$receiver");`这个函数只起到了判空的作用，真正的代码是`return ContextCompat.getColor($receiver, colorRes);`这个不就是我们刚刚用的`Java`代码嘛。

重点是`$receiver`接收的对象，接收的是`Context`实例，这样的话就可以调用这个类的所有公有方法和公有属性，而且它是静态函数，它可以通过类直接调用。所以扩展函数的实现只不过是加了一个需要当前对象的静态方法，调用的时候传入一个当前对象而已。

我们刚刚用到了反编译，因为我们知道`Kotlin`和`Java`的生成字节码是一样的，那我们可以了解一下`Kotlin`的编译过程，它跟`Java`的区别是什么。可以看一下这篇文章[Kotlin编译过程分析](http://shinelw.com/2017/03/19/kotlin-compiler-process-analysis/)

通过这篇文章你可以了解到`Kotlin`在编译过程中，与`Java`是大致相同的,只是在最后生成目标代码的时候做了很多类似于封装的事情，生成相同的语法结构，`Kotlin`将我们本来在代码层做的一些封装工作转移到了编译后端阶段。那我们可不可以在学习`Kotlin`的时候去这样理解，其实`Kotlin`是一种封装了`Java`的强大的语法糖，`Java`做不到的事情，`Kotlin`其实也做不到，例如对象只能访问公有属性。

#### 6.数据类
在`Kotlin`中你要实现数据类是非常简单的，并不需要手动加上`get/set`方法

```kotlin
data class BarItem(
        private val barData: BarData,
        var select: Boolean = false) {
    fun getData(): Double {
        return barData.getData()
    }

    fun getTag(): String {
        return barData.getTag()
    }
}
```
在这个类里面你会发现，我还声明了两个方法，我需要的是`BarData`里的数据，但又不仅仅只需要这个数据，所以我声明了一个类来封装它，其实这个相当于装饰者模式了。`Kotlin`有更好的方式实现这个模式

```kotlin
data class BarItem(
        private val barData: BarData,
        var select: Boolean = false) : BarData by barData
```

#### 7.when
在`BarChartView`里用到一个与`switch`语法类似的语句

```kotlin
 mSelectPosition = when (mStyle) {
            ScrollStyle.DEFAULT -> mDataList.size - 1
            ScrollStyle.START -> 0
            ScrollStyle.NONE -> -1
            ScrollStyle.CUSTOM -> mSelectPosition
            else -> { }
        }
```
它是起到了跟`switch`一样的作用,并且更强大，因为它是表达式，所以是有返回值的，在`Kotlin`中控制流大都是表达式，都是可以有返回值的。

#### 8.集合
`Kotlin`是区分可变集合和不可变集合的，它给你提供这两种选择。

```kotlin
 //不可变
 Set<out T>
 Map<K, out V>
 List<out T>
 //可变
 MutableSet<T>
 MutableMap<K, V>
 MutableList<T> 
```
不可变的集合提供只读属性，例如`size`,`get`等,`Kotlin`不提供专门的语法结构创建`list`或者`set`,是用标准库获取的，我们可以看一下它的源码是怎样实现。

```kotlin
/**
 * Returns an immutable list containing only the specified object [element].
 * The returned list is serializable.
 * @sample samples.collections.Collections.Lists.singletonReadOnlyList
 */
@JvmVersion
public fun <T> listOf(element: T): List<T> = java.util.Collections.singletonList(element)

/**
 * Returns an empty new [MutableList].
 * @sample samples.collections.Collections.Lists.emptyMutableList
 */
@SinceKotlin("1.1")
@kotlin.internal.InlineOnly
public inline fun <T> mutableListOf(): MutableList<T> = ArrayList()

```
从源码可以看见这是`Java`的`java.util.Collections.singletonList `和`ArrayList`，这就可以理解为啥不可变和可变的了。。。

### 总结
`Kotlin`相对于`Java`,更像是封装了`Java`的强大语法糖，使用了更简洁的语法提高了生产力。





