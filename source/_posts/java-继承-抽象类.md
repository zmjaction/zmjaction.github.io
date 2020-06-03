---
title: java-- 继承-- 抽象类
mathjax: false
date: 2018-10-14 20:31:27
categories:
    - java
    - 继承
    - 抽象类
tags:
    - java
    - java继承
    - java抽象类

---

### 继承

> Java只支持单继承，不支持多继承。
>
> 顶层父类是Object类。所有的类默认继承Object，作为父类。
>
> 继承：就是子类继承父类的属性和行为，使得子类对象具有与父类相同的属性、相同的行为。子类可以直接 
>
> 访问父类中的非私有的属性和行为
>
> 好处：
>
> 1. 提高代码的复用性。 
>
>
> 2. 类与类之间产生了关系，是多态的前提。

##### 通过 `extends` 关键字，可以声明一个子类继承另外一个父类

示例：

```java
public class Employee {
    public void work(){
        System.out.println("去工作");
    }
}

public class Teacher extends Employee {
//    @Override
//    public void work(){
//        System.out.println("老师去工作");
//    }
}
public class Demo01main {
    public static void main(String[] args) {
        Teacher th = new Teacher();
        th.work();
    }
}

```

#### 继承之后-成员变量重名、不重名

> 子父类中出现了同名的成员变量时，在子类中需要访问父类中非私有成员变量时，需要使用 `super `关键字，修饰 父类成员变量，类似于` this` ;
>
> super.父类成员变量名 

```java
public class Employee {
    int num = 6;
    int num1 = 20;
    public void work(){
        System.out.println("去工作");
    }
}

public class Teacher extends Employee {
    int num = 10;
    @Override
    public void work(){
        System.out.println("老师去工作");

    }
    public void method(){
        // 子类中访问父类得同名变量
        System.out.println(super.num);
        // 子类中访问自己得变量
        System.out.println(this.num);
    }
}


/*
直接通过子类对象访问成员变量：
    等号左边是谁，就优先用谁，没有则向上找。
间接通过成员方法访问成员变量：
    该方法属于谁，就优先用谁，没有则向上找。
 */
public class Demo01main {
    public static void main(String[] args) {
        Teacher th = new Teacher();
        th.work();
        // 优先子类得
        System.out.println(th.num);
        // 子类中没有查找父类得
        System.out.println(th.num1);
        th.method();
    }
}

```

#### 继承之后-方法重名、不重名

> 重写（Override）
> 概念：在继承关系当中，方法的名称一样，参数列表也一样。
>
> 重写（Override）：方法的名称一样，参数列表【也一样】。覆盖、覆写。
>
> 重载（Overload）：方法的名称一样，参数列表【不一样】
>
> 注意：
>
> 子类方法的权限必须【大于等于】父类方法的权限修饰符
>
> public > protected > (default) > private
>
> 不是关键字default，而是什么都不写，留空

```java
public class Animal {
    public void Eat(){
        System.out.println("吃东西");
    }
}

public class Dog extends Animal {

    @Override
    public void Eat() {
        System.out.println("吃骨头");
    }
    public void Call(){
        System.out.println("汪汪叫");
    }
}
public class Demo1main {
    public static void main(String[] args) {
        Dog dg = new Dog();
        // 不同名得方法直接调用即可
        dg.Eat();
        dg.Call();
        // 覆盖父类中得同名方法
        dg.Eat();
    }
}

```

#### 继承之后-构造方法

> 子类必须调用父类构造方法，不写默认调用super()；写了则用写的指定的super调用，super只能有一个，还必须是第一个。

```java
public class Animal {
    public Animal(){
        System.out.println("父类的无参数构造方法");
    }
    public  Animal(int num){
        System.out.println("父类中的有参构造方法");

    }
}
public class Dog extends Animal {
    public Dog(){
//        super();
        super(100);
        System.out.println("子类中构造方法");
    }
    public void method(){
        System.out.println("子类中的方法");
    }
}
public class demomain {
    public static void main(String[] args) {
        Dog dg = new Dog();
        dg.method();
    }
}

```

### 抽象类

> 抽象方法 ： 没有方法体的方法。 
>
> 抽象类：包含抽象方法的类。 
>
> 1. 抽象类不能创建对象，如果创建，编译无法通过而报错。只能创建其非抽象子类的对象。
>
> 2. 抽象类中，可以有构造方法，是供子类创建对象时，初始化父类成员使用的。
>
> 3. 抽象类中，不一定包含抽象方法，但是有抽象方法的类必定是抽象类
>
> 4. 抽象类的子类，必须重写抽象父类中所有的抽象方法，否则，编译无法通过而报错。除非该子类也是抽象 
>
>    类

```java
public abstract class Animal {
    public abstract void eat();
}
public class Cat extends Animal {
    @Override
    public void eat() {
        System.out.println("猫吃鱼");
    }

    public static void main(String[] args) {
        Cat ct = new Cat();
        ct.eat();
    }
}
```

