# Chapter 8: Iterator and Composite 迭代器模式和组合模式

<div align=center>
	<img src="./image/chapter-2/design-pattern-overview.png" width="">
</div>

## 迭代器模式

#### 迭代器是什么 and 它解决什么问题？

书上的例子是：需要合并两家餐厅的菜单，但是其中一家使用数组，另一家使用 ArrayList，如何设计一个统一的“查看菜单”方法？

- 通过多个 `if` 来判断（耦合度高、冗余多、接口适配者需要了解所有下层数据结构的访问方法然而我们并不关心菜单具体怎么实现）
- 修改底层代码（工作量大）

抽象地看，需要解决的问题是通过一个统一的方式**遍历**不同的集合/数据结构，且尽量不对外体现数据结构的实现。因此，本章的核心概念是：迭代器模式的主要思想是将集合的遍历行为单独做一个抽象层次。

![](https://refactoringguru.cn/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0)

1. 迭代器 （Iterator 或者叫 CreateIterator）定义创建迭代器的接口，声明了遍历集合所需的操作： 获取下一个元素、 获取当前位置和重新开始迭代等。←在一些地方也可能被叫做 Aggregate
2. 具体迭代器 （Concrete Iterators） 实现遍历集合的一种具体算法。 迭代器对象必须跟踪自身遍历的进度。←例如，Java的迭代器具有`remove()`方法，因此访问自带锁等
3. 集合 （Collection） 接口声明一个或多个方法来获取与集合兼容的迭代器。 请注意， 返回方法的类型必须被声明为迭代器接口。←例如，
4. 具体集合 （Concrete Collections） 请求迭代器时返回一个特定的具体迭代器类实体。 

#### 什么是可迭代对象

在 Python 中：

- 集合或序列类型（如 `list`、`tuple`、`set`、`dict`、`str`），但是注意可迭代不一定有序
- 文件对象
- 任何自定义了 `__iter__()` 方法的对象，
    - 在 Python 中，只要该 `__iter__()` 方法正确地实现，就可以使用 `iter()` 函数将它转化成迭代器
    - 但是反之，可以使用 `iter()` 转化为迭代器的，不一定是可迭代对象（如果`__getitem__()` 方法正确实现也可以转化）

迭代器最基本的操作就是用于遍历的 `next()` 方法：

```
currPath = os.path.dirname(os.path.abspath(__file__))
with open(currPath+'/model.py') as file:
    print(isinstance(file, Iterable)) # true


fuck = iter([1,2,3,4])
next(fuck) # 1
next(fuck) # 2
```

#### 迭代器的使用

下面的例子是遍历一个封装访问所有微信好友关系的特殊 collection，其中，“好友 （friends）” 迭代器可用于遍历指定档案的好友。 “同事 （coworkers）” 迭代器也提供同样的功能。

```
class WeChat implements SocialNetwork is
    blabla

    // 迭代器创建代码。
    method createFriendsIterator(profileId) is
        return new WeChatIterator(this, profileId, "friends")
    method createCoworkersIterator(profileId) is
        return new WeChatIterator(this, profileId, "coworkers")

// 所有迭代器的通用接口。
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool

// 具体迭代器类。
class WeChatIterator implements ProfileIterator is
    // 迭代器需要一个指向其遍历集合的引用。
    private field weChat: WeChat
    private field profileId, type: string

    // 如上所述，迭代器需要自行管理访问状态
    private field currentPosition
    private field cache: array of Profile

    constructor WeChatIterator(weChat, profileId, type) is
        this.weChat = weChat
        this.profileId = profileId
        this.type = type

    // 每个具体迭代器类都会自行实现通用迭代器接口。
    method getNext() is
        if (hasMore())
            currentPosition++
            return cache[currentPosition]

    method hasMore() is
        lazyInit()
        return currentPosition < cache.length

```

#### 总结

- 迭代器允许访问聚合的元素，而不需要暴露它的内部结构。
- 迭代器将遍历操作封装进一个对象中。
- 当使用迭代器的时候，我们通过接口（或者聚合 Aggregate）进行遍历操作。
- 迭代器提供了一个通用的接口，让我们遍历聚合的项，当我们编码使用聚合的项时，就可以使用多态机制。

作用：

- 隐藏复杂性（提高安全等目的）
- 减少重复的遍历代码
- 希望代码能够遍历不同的甚至是无法预知的数据结构

#### 题外话：Python 的生成器

在 Python 中生成器的一个经典应用是用于实现协程（在较早版本中），

这里 `yield` 的作用就相当于 `return` ，通过 `next()` 或使用 for 循环来遍历。执行到 `yield` 关键字时，生成器函数就返回了，直到再次执行了 `next()` 函数，它就会从上次函数返回的执行点继续执行，即yield退出时保存了函数执行的位置、变量等信息，再次执行时，就从这个`yield`退出的地方继续往下执行。

且 Python 的生成器是惰性求值的，利用生成器的这些特点可以实现协程。相对于直接 return 整个对象，可以节约内存拷贝的开销和空间。

```
def producer(c):
    n = 0
    while n < 5:
        n += 1
        print('producer {}'.format(n))
        r = c.send(n)
        print('consumer return {}'.format(r))


def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('consumer {} '.format(n))
        r = 'ok'


if __name__ == '__main__':
    c = consumer()
    next(c)
    producer(c)
```
以上代码执行时，会交替在 consumer 和 producer 之间执行，类似以下结果：

```
producer 1
consumer 1 
producer return ok
producer 2
consumer 2 
producer return ok
```


## 组合模式

#### 组合模式是什么 and 它解决什么问题？

书上的例子感觉不够好，借用一下其他地方的：假设你是老板，你需要统计整个部门的报价，那一般你会选择让秘书把统计任务分摊到每个小组，等小组汇报后汇总。

![](https://refactoringguru.cn/images/patterns/content/composite/composite-comic-1-zh.png?id=845971cd0cc64fb0f3e303f393a9102c)

组合模式是一种结构型设计模式， 你可以使用它将对象组合成树状结构（或者说**可递归**的结构）， 并且能像使用独立对象一样使用它们。

该方式的最大优点在于你无需了解构成树状结构的对象的具体类。 你也无需了解对象是简单的产品还是复杂的盒子。 你只需调用通用接口以相同的方式对其进行处理即可。 当你调用该方法后， 对象会将请求沿着树结构传递下去。

![](https://refactoringguru.cn/images/patterns/diagrams/composite/structure-zh.png?id=205c2c970f77efe15b681525e0c676d3)

1. 组件 （Component） 接口描述了树中简单项目和复杂项目所共有的操作。
2. 叶节点 （Leaf） 是树的基本结构， 它不包含子项目。一般情况下， 叶节点最终会完成大部分的实际工作， 因为它们无法将工作指派给其他部分。
3. 容器 （Container）或 “组合 （Composite）”——是包含叶节点或其他容器等子项目的单位。 容器不知道其子项目所属的具体类， 它只通过通用的组件接口与其子项目交互。容器接收到请求后会将工作分配给自己的子项目， 处理中间结果， 然后将最终结果返回给客户端。
4. 客户端 （Client） 通过组件接口与所有项目交互。 因此， 客户端能以**相同方式**与树状结构中的简单或复杂项目交互。

#### 为什么本书要把组合模式和迭代器放在一起？

因为迭代器模式可以用于访问组合模式，在上一节中的“菜单”可以被重新设计，但是由于我们有迭代器接口，遍历菜单这个操作可以与底层是数组还是树形无关。

#### 组合模式的使用

我们有一个类 Employee，该类被当作组合模型类。CompositePatternDemo 类使用 Employee 类来添加部门层次结构，并打印所有员工。

```
# 创建 Employee 类，该类带有 Employee 对象的列表。
public class Employee {
   private String name;
   private String dept;
   private int salary;
   private List<Employee> subordinates;
 
   //构造函数
   public Employee(String name,String dept, int sal) {
      this.name = name;
      this.dept = dept;
      this.salary = sal;
      subordinates = new ArrayList<Employee>();
   }
 
   public void add(Employee e) {
      subordinates.add(e);
   }
 
   public void remove(Employee e) {
      subordinates.remove(e);
   }
 
   public List<Employee> getSubordinates(){
     return subordinates;
   }
 
   public String toString(){
      return ("Employee :[ Name : "+ name 
      +", dept : "+ dept + ", salary :"
      + salary+" ]");
   }   
}

# 使用 Employee 类来创建和打印员工的层次结构。
public class CompositePatternDemo {
   public static void main(String[] args) {
      Employee CEO = new Employee("John","CEO", 30000);
 
      Employee headSales = new Employee("Robert","Head Sales", 20000);
 
      Employee headMarketing = new Employee("Michel","Head Marketing", 20000);
 
      Employee clerk1 = new Employee("Laura","Marketing", 10000);
      Employee clerk2 = new Employee("Bob","Marketing", 10000);
 
      Employee salesExecutive1 = new Employee("Richard","Sales", 10000);
      Employee salesExecutive2 = new Employee("Rob","Sales", 10000);
 
      CEO.add(headSales);
      CEO.add(headMarketing);
 
      headSales.add(salesExecutive1);
      headSales.add(salesExecutive2);
 
      headMarketing.add(clerk1);
      headMarketing.add(clerk2);
 
      //打印该组织的所有员工
      System.out.println(CEO); 
      for (Employee headEmployee : CEO.getSubordinates()) {
         System.out.println(headEmployee);
         for (Employee employee : headEmployee.getSubordinates()) {
            System.out.println(employee);
         }
      }        
   }
}
```
运行结果：

```
Employee :[ Name : John, dept : CEO, salary :30000 ]
Employee :[ Name : Robert, dept : Head Sales, salary :20000 ]
Employee :[ Name : Richard, dept : Sales, salary :10000 ]
Employee :[ Name : Rob, dept : Sales, salary :10000 ]
Employee :[ Name : Michel, dept : Head Marketing, salary :20000 ]
Employee :[ Name : Laura, dept : Marketing, salary :10000 ]
Employee :[ Name : Bob, dept : Marketing, salary :10000 ]
```

#### 总结

- 前提是对象需要有一定程度的可归纳性（有相似性），确保核心模型能够以树状结构表示， 尝试将其分解为简单元素和容器。
- 对于功能差异较大的类， 提供公共接口或许会有困难。 在特定情况下， 你需要过度一般化组件接口。
- 组合和装饰模式的结构图很相似， 因为两者都依赖递归组合来组织无限数量的对象。装饰类似于组合， 但其只有一个子组件。 此外还有一个明显不同： 装饰为被封装对象添加了额外的职责， 组合仅对其子节点的结果进行了 “求和”。

作用：

- 实现树状对象结构
- 希望客户端代码以相同方式处理简单和复杂元素


ref:
- https://refactoringguru.cn/design-patterns/iterator
- https://refactoringguru.cn/design-patterns/composite
- https://kelepython.readthedocs.io/zh/latest/c01/c01_11.html
- https://segmentfault.com/a/1190000039079890

Appendix: slide https://docs.google.com/presentation/d/1mNv15WY8VasgCz5AFVd_RcZ-4BjqlWPgvIrUKUKaO78/edit?usp=sharing 