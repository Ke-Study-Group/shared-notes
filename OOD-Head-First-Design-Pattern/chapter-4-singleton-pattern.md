## Chapter 4: Singleton Pattern 单件模式

</br>

## 1. Definition

</br>

单件模式(Singleton Pattern)确保每一个类(Class)只有一个实例(Instance), 并提供一个全局访问点(global point of access)。

</br>

### 1.1 A Simple Example

</br>

```Java
public class Singleton {
    private static Singleton uniqueInstance;
    
    private Singleton(){}
    
    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    } 
}
```

## 2.Why Using it

</br>

### 2.1 Without Singleton Pattern

</br>

多个对象可能会造成Reference混乱。
![Design UML](./image/chapter-4/2-1.png)

</br>

### 2.2 With Singleton Pattern

</br>

单件模式保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象(Global Object)，这样有利于我们协调系统整体的行为。
![Design UML](./image/chapter-4/2-2.png)

</br>