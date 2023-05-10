# 1. 硬件相关

## 1.1 内存分区

cpp编写的程序的进程的内存分区

```
代码区: 存放指令
常量区: 存储const变量/字面值常量
全局静态区: 存储全局变量/类的static变量
堆: 存储new/malloc创建的对象
栈: 存储函数的局部变量和返回地址
```

## 1.2 struct存储对齐

struct存储对齐有利于一次访存可以取出对象, 避免多次访存

对齐规则(以int为例)

```
1. 一个成员变量的起始地址必须是变量类型的整数倍 (int的起始地址必须是4n)
2. struct总大小必须是最大成员变量的整数倍
3. struct的最小大小=1byte
```

> 指针类型的长度在64位CPU中为 8Byte/4Byte, 在32位CPU中为4Byte

# 2. 基础

## 2.1 函数

## 2.2 数组

## 2.3 输入输出

## 2.4 string和const char*

## 2.5 指针

变量的指针

```c
    int a = 1;
    int *ptrToA = &a; // 一级指针
    int **ptrToPtrA = &ptrToA; // 二级指针
    out::print((*ptrToA) + (**ptrToPtrA)); // 1+1 = 2
```

**void指针**是通用指针类型

```c
    int a = 1;
    void *ptr = &a;
    int *aPtr = (int*) ptr; // 将void*强制转为int*
    out::print(*aPtr);
```

**函数指针**

```c
int add(int a, int b) { return a + b; }

int main() {
    int (*Add)(int, int) = &add;
    // int (*Add)(int, int) 表示指针名叫做Add, 返回类型int, 参数类型(int, int)
    int ret = (*Add)(1, 2); // (*Add)(1, 2) 等价于 add(1, 2)
    out::print(ret);
}
```



## 2.6 内存管理

# 3. 容器

## 3.1 vector
