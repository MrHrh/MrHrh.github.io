TS基础知识
=======
# 目录
- [TS基础知识](#ts基础知识)
- [目录](#目录)
- [介绍](#介绍)
- [常用语法](#常用语法)
  - [变量类型](#变量类型)
    - [number类型](#number类型)
    - [any类型](#any类型)
    - [void类型](#void类型)
    - [array类型](#array类型)
      - [多类型元素数组定义](#多类型元素数组定义)
      - [类类型的数组定义](#类类型的数组定义)
      - [数组赋值](#数组赋值)
    - [tuple类型](#tuple类型)
  - [类型注解和类型推断](#类型注解和类型推断)
  - [函数](#函数)
    - [普通函数](#普通函数)
    - [箭头函数](#箭头函数)
    - [函数泛型](#函数泛型)
    - [函数参数](#函数参数)
  - [面向对象编程](#面向对象编程)
    - [类的基本实现语法](#类的基本实现语法)
    - [类的继承和重写](#类的继承和重写)
    - [类的访问限定符](#类的访问限定符)
    - [类的构造函数](#类的构造函数)
    - [类的只读属性](#类的只读属性)
  - [模块](#模块)
- [与js的区别](#与js的区别)
- [拓展](#拓展)
  - [箭头函数深入研究](#箭头函数深入研究)
  - [js迁移ts](#js迁移ts)
- [参考文档](#参考文档)
# 介绍
TypeScript是一种由微软开发的自由和开源的编程语言，它是JavaScript的超集。TypeScript在JavaScript的基础上添加了可选的静态类型和基于类的面向对象编程，并且增强了JavaScript编写应用的开发和调试环节。  
本文主要会介绍TypeScript的常用语法以及其和JavaScript相比而言的优缺点等几个方面来介绍TypeScript。  
# 常用语法
## 变量类型
TypeScript的一个特点就是变量是强类型的，当我们在声明一个变量时必须给他指定一个类型。TypeScript中的数据类型有下面几种：  

|类型|含义|
|---|---|
|undefined| 未定义|
|number   |数值类型|
|string   | 字符串类型|
|boolean  | 布尔类型|
|enum:    | 枚举类型|
|any:     | 任意类型|
|void:    | 空类型|
|Array:   | 数组类型|
|tuple:   | 元组类型|
|null:    | 空类型|

### number类型
在TypeScript中，所有的数字都是浮点数，类型是Number类型。当我们定义了一个Number类型变量（其他类型同样适用）但没有给它赋任何值的时候，编译器并不会给它赋一个默认值，而是将它的类型设置为Undefined，这一点和JavaScript中的做法是相同的。  
- 除了支持十进制和十六进制的字面量之外，TypeScript中还支持二进制和八进制的字面量。

```TypeScript
//10进制整数
let temp1:number = 18;
//10进制小数
let temp2:number = 18.123;
//16进制
let temp3:number = 0xf00d;
//2进制
let temp4:number = 0b1010;
//8进制
let temp5:number = 0o744;
```  

- 特殊的Number类型
  - NaN：是Not a Number的缩写，意思是不是一个数值。当一个计算结果或者函数的返回值本应该是数值，但是由于某些原因返回的值不是数字，出现这种情况时编译器不会报错，而是把它的结果看成了NaN。

```TypeScript
var month = 0;
if (month <= 0 || month > 12) {
    month = Number.NaN;
    console.log("月份是：" + month);
}else {
    console.log("输入月份数值正确。");
}
```  

  - Infinity：正无穷
  - -Infinity：负无穷

### any类型
当程序中的某些变量在编程阶段还不清楚类型时，比如用户的输入。这种情况下，如果我们不希望类型检查器对这些值进行检查而是直接通过编译阶段的检查，那么我们可以将这些变量的类型设置为any。  

```TypeScript
var temp:any = 10
temp = "hahaha"
temp = true
```  

### void类型
void表示没有任何类型，当我们声明了一个void类型的变量时，我们只能该变量赋予Undefined和null。某种意义上来说void和any是相反的。  
### array类型
TypeScript中声明数组有两种方式，第一种是可以在元素类型后面加上[]，表示由此类型元素组成的一个数组。第二种方式是使用数组泛型，Array<元素类型>的方式声明。  

```TypeScript
//元素类型后加[]
let list: number[] = [1, 2, 3];
//使用数组泛型
let list: Array<number> = [1, 2, 3];
```  

#### 多类型元素数组定义
TypeScript中的数组支持多种类型的元素，只需要我们在定义时将类型用()括起来，类型之间用|分割即可：  

```TypeScript
const arr: (number | string)[] = [1, "string", 2];
```  

#### 类类型的数组定义
```TypeScript
class Student {
  name: string;
  age: number;
}

const ClassOne: Student[] = [
  { name: "哈哈", age: 18 },
  { name: "呵呵", age: 18 },
];
```  
#### 数组赋值
TypeScript中给数组赋值有两种方法，分别是：  
- 字面量赋值法
- 构造函数赋值法

```TypeScript
//定义一个空数组，数组容量为0
let arr1:number[] = [];
//定义一个数组时，直接给数组赋值
let arr2:number[] = [1, 2, 3, 4, 5];
//定义数组 的同事给数组赋值
let arr3:Array<string> = ['hehe', '呵呵', '哈哈'];
let arr4:Array<boolean> = [ true, false, false];

let arr1:number[] = new Array();
let ara2:number[] = new Array(1, 2, 3, 4, 5);
let arr3:Array<string> = new Array('hehe', '呵呵', '哈哈');
let arr4:Array<boolean> = new Array(true, false, false);
```  

### tuple类型
元组类型允许表示一个已知元素数量和类型的数组，元组中各元素的类型不必相同。  

```TypeScript
let x: [string, number];
//正确写法
x = ['hello', 10];
//错误写法
x = [10, 'hello'];
```  

## 类型注解和类型推断
- 类型注解（type annotation） 
 
其实类型注解在前面的讲解中已经多次使用，当我们声明一个变量时，为该变量指定类型的操作即为类型注解，类型注解告诉编译器我们的变量是什么类型，代码如下：  

```TypeScript
let count: number;
count = 123;
```  

- 类型推断（type inferrence） 
 
当我们明白了类型注解的含义后，再学习类型推断就很容易理解了。 

```TypeScript
let count = 123;
```  

在上面这段代码里我们并没有显示的告诉编译器变量count是什么类型的，但是当我们把鼠标放到变量上时，我们会发现TypeScript自动把变量注释为了number类型，这就是类型推断，通过我们的代码TypeScript会自动的去尝试分析变量的类型。  
- 什么时候需要类型注解？

```TypeScript
function getTotal(one , two) {
  return one + two;
}

const total = getTotal(1, 2);
```  

在上面这段代码中，如果我们不使用类型注解明确one，two两个参数的类型，那么如果我们传入字符串的话，业务逻辑就会出错，所以这时我们就需要给其加上明确的类型注解。  
## 函数
### 普通函数
正常来说函数的定义需要使用function关键字，函数名，参数列表和返回值来定义一个函数，写法如下：  

```TypeScript
function add(n1: number, n2: number): number {
    return n1+n2
}

function add(n1: number, n2: number) {
    return n1+n2
}

let add = function(n1: number, n2: number): number {
    return n1+n2
}
console.log(add(1,4))
```  

这里值得注意的点是，我们可以不为函数的返回值添加类型注解，因为TypeScript具有类型推导的能力，比如我们上面例子中的两个函数定义都是正确的。  
匿名函数的实现是将一个函数表达式赋值给一个变量，通过变量来调用函数。如上面例子中第三段代码所示。  
### 箭头函数
箭头函数（arrow function）是ES6中新增的函数定义的方式，它省去了function关键字，采用=>来定义函数，函数体跟在=>后面的花括号中。我们来看一个具体的例子：  

```TypeScript
// 箭头函数
let fun = (name: string) => {
    // 函数体
    return `Hello ${name} !`;
};

// 等同于
let fun = function (name: string) {
    // 函数体
    return `Hello ${name} !`;
};
```  

- 箭头函数的参数
  - 如果没有参数，定义形参处可以直接写一个空括号即可。
  - 如果只有一个参数，可省去括号。
  - 如果有多个参数，依次用逗号分割，包裹在括号内。
- 箭头函数的函数体
  - 如果函数体中只有一句代码，就是返回某个变量或者一个简单的表达式，这时可以省去函数体的花括号。

```TypeScript
let f = (val: number) => val;
// 等同于
let f = function (val: number) { return val };

let sum = (num1: number, num2: number) => num1 + num2;
// 等同于
let sum = function(num1: number, num2: number) {
  return num1 + num2;
};
```  

### 函数泛型
假设现在有这样一个名为join的方法，方法中有两个参数first和second，两个参数的类型都有可能是数字或者字符串。我们实现如下：  

```TypeScript
function join(first: string | number, second: string | number) {
  return `${first}${second}`;
}
join("jspang", ".com");
```  

这个方法的实现是正确的，但是如果有需求为：first和second两个参数的类型同时只能传一种类型，否则内部计算有可能会出错。这时目前的实现就无法完成了，所以就需要学习函数泛型有关的知识。  
泛型的定义使用<>来进行，我们只需要在函数名的后面加上<name>并且将参数类型设置为name类型即可，这里name可以是任何一个符合实际语义的词。在正式调用的时候再去指定该泛型对应的具体类型：  

```TypeScript
function join<name>(first: name, second: name) {
  return `${first}${second}`;
}
join < string > ("baidu", ".com");
join < number > (1, 2);
```  

### 函数参数
- 可选参数

可选参数就是在我们定义形参的时候可以指定该参数非必须，在TypeScript中使用?来标注一个参数为可选参数。  

```TypeScript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
```  

- 默认参数

我们可以给形参指定一个默认的值，当函数调用时如果我们没有传入实参，则函数体内参数的值为默认参数的值。  

```TypeScript
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}
```  

- 剩余参数

上面说的两种参数的共同点是都只能表示一个参数，当我们要同时操作多个参数或者我们并不清楚会有多少参数传递进来时，我们可以使用剩余参数来表示。  

```TypeScript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}
```  

剩余参数相当于个数不限的可选参数。可以是0个也可以是任意个。编译器会创建一个参数数组，名字是...之后的变量名，我们可以在函数体内使用这个数组。  
## 面向对象编程
面向对象编程的基本思想大家可能已经比较熟悉了，这里我们主要说明一下面向对象编程在TypeScript中的实现和具体语法。  
### 类的基本实现语法
假设我们现在有一个Dog的类，它有name，age，kind属性和bark方法，在TypeScript中的实现如下：  

```TypeScript
class Dog {
    public name: string;
    public age: number;
    public kind: string;
    bark() {
        console.log('Woof! Woof!');
    }
}

const wangcai = new Dog();
wangcai.bark();
```  

### 类的继承和重写
当我们有一个Cat类并且Cat类和Dog拥有共同的属性时，我们就可以将他们共同的属性抽离出来作为他们两个的共同的父类Animal，然后Cat类和Dog去继承Animal类。这样就不用实现两边他们的公共属性了，直接使用从父类中继承而来的即可。  
在TypeScript中声明类的继承关系的关键字是extends。  

```TypeScript
class Animal {
    public name: string;
    public age: number;
    public kind: string; 
    bark() {
        console.log('bark');
    }
}

class Dog extends Animal {
    bark() {
        console.log('wang! wang!');
    }
}

class Cat extends Animal {
    bark() {
        console.log('miao! miao!');
    }
}

const wangcai = new Dog();
wangcai.bark();
```  

### 类的访问限定符
几乎所有的面向对象编程语言的类中都会有访问限定符的概念，共有三种：public，private，protected。这是大家都已经很熟悉的概念，就不再赘述。  
- public：公共属性，允许在类的内部和外部被访问。
- private：私有属性，只允许在类的内部访问，派生类中不能访问。
- protected：受保护属性，允许在类的内部和派生类中访问。

### 类的构造函数
构造函数在类被初始化的时候会自动执行，我们可以通过构造函数来给类中的成员变量进行赋值。TypeScript中类的构造函数的名称为constructor。  

```TypeScript
class Person {
    public name: string;
    constructor(name: string) {
        this.name = name;
    }
}

const person= new Person('哈哈');
console.log(person.name);
```  

- 继承类中的构造函数  
在子类中使用构造函数需要使用super()方法调用父类的构造函数，即便父类没有实现构造函数，子类也需要super()调用，否则会报错。  

```TypeScript
class Person {
    public name: string;
    constructor(name: string) {
        this.name = name;
    }
}

class Teacher extends Person {
    constructor(name: string) {
        super(name);
    }
}
```  

- 定义属性的简化语法  
在给一个类定义一个属性的时候，除了上面那种普通的写法之外，还可以借助构造函数来实现一种简化的写法，如下代码所示，这种写法等价于定义了一个name，并且在构造函数中进行了赋值。

```TypeScript
class Person {
    constructor(public name:string) {
    }
}

const person= new Person('哈哈');
console.log(person.name);
```  

### 类的只读属性
在TypeScript中我们可以使用readonly关键字将属性设置为只读的，只读属性必须在声明时或构造函数里被初始化，readonly关键字在使用时需要放在访问限定符之后。  

```TypeScript
class Person {
    //构造函数中赋值
    public readonly name: string;
    //声明时赋值
    public readonly age: number = 10;
    constructor(name: string) {
        this.name = name;
    }
}
```  

## 模块
模块在其自身的作用域里执行，而不是在全局作用域里，这意味着定义在一个模块里的变量，函数，类等在模块外部是不可见的。  
当我们想在外部使用模块内的变量，函数或者类时，我们需要明确地使用export关键字导出。同理，如果想要使用其他模块定义的变量，函数等，我们需要使用import关键字导入。  
- 命名导出  

```TypeScript
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
//对导出模块进行重命名
export { ZipCodeValidator as mainValidator };
```  

- 默认导出  
每个模块都可以有并且最多只能有一个默认导出，使用default关键字标记。在导入时，可以使用任意名称导入默认导出。  

```TypeScript
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export default ZipCodeValidator;
```  

- 导入模块  

```TypeScript
//对导入模块进行重命名
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```  

# 与js的区别
- TypeScript是JavaScript的超集，所有JavaScript的代码都可以在TypeScript中运行。
- TypeScript具有面向对象编程的能力，它引入了类的概念，而JavaScript是一种脚本语言。
- TypeScript是强类型语言，它会在开发时会为我们检测错误，因此在运行时出现错误的可能性小于JavaScript，也为开发人员创建了一个更高效的编码和调试过程。
- TypeScript增加了void/any/tuple/enum等JavaScript中没有的类型。  

# 拓展
## 箭头函数深入研究
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions  
https://www.cnblogs.com/freelyflying/p/6978126.html  
## js迁移ts
https://github.com/Microsoft/TypeScript-React-Conversion-Guide#typescript-react-conversion-guide  
# 参考文档
https://jspang.com/detailed?id=38#toc21  
https://jspang.com/detailed?id=63#toc235  
https://zhuanlan.zhihu.com/p/75337266  
https://www.tslang.cn/docs/handbook/functions.html  
