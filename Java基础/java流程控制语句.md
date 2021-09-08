## 流程控制基本介绍

流程控制:
顺序结构: 代码从上到下依次执行
分支结构: if, if-else, if-else if-else, switch
循环结构: for, while, do...while


## 顺序结构

```java
System.out.println("今天你学习了么?");
System.out.println("Java程序入门到女装");
System.out.println("Java入门到入土");
```




## if的使用

```java
System.out.print("请输入你刀的长度: ");
int size = new Scanner(System.in).nextInt();

if (size >= 40) {
    System.out.println("您被确定为恐怖分子");
} else if (size <= 10) {
    System.out.println("欢迎上车");
} else {
    System.out.println("对不起, 您的长度不符合");
}
```




## switch的使用

支持强制类型转换与自动类型转换
Case分支需要使用break进行跳出

```java
// switch -> char搞事
System.out.print("请输入一个数字: ");
int nextInt = new Scanner(System.in).nextInt();

// 判断
switch (nextInt) {
    case 1:
        System.out.println("星期一");
        break;
    case 2:
        System.out.println("星期二");
        break;
    default:
        System.out.println("默认处理->结束");
}
```




## for

```java
// for (定义的变量(可以是多个), 条件判断(只能是一个), 操作(可以是多个, 根据你定义的变量, 不能比定义的多, 但是能比定义的少))

public class Demo12 {
    public static void main(String[] args) {
        // for (定义的变量(可以是多个), 条件判断(只能是一个), 操作(可以是多个, 根据你定义的变量, 不能比定义的多, 但是能比定义的少))

        for (int i = 0, j = 0, k = 0; i < 10; i++, j++, k++) {
            System.out.println("i: " + i);
            System.out.println("j: " + j);
            System.out.println("k: " + k);
        }
    }

}
```




## break与continue

break: 结束本层循环(不执行下面的代码了)
continue: 跳出本层循环(跳过本次的循环, 下面的代码还是需要执行)

```java
public class Demo13 {
    public static void main(String[] args) {
        int i = 0;
        while (true) {
            i++;
            if (i < 10) {
                if (i == 6) {
                    // 跳过本次执行
                    continue;
                }

                System.out.println("i: " + i);
            } else {
                // 结束循环, 跳出一层
                break;
            }
        }
    }

}
```




## while与do while

while: 优先判断条件, 条件符合执行循环中的代码

```java
while(true/false) {

}
```

do while: 先执行do循环里面的代码, 在进行条件判断.

```java
do {

} while(true/false)
```

