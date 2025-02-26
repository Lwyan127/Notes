# devc++设置

## 语言

工具 -> 环境选项 -> 语言

## debug

1. 工具 -> 编译选项 -> 设定编译器配置：选择Debug编译器
2. 工具 -> 编译选项 -> 代码优化 -> 连接器 -> 产生调试信息：yes

**一些debug功能：**

Next Step（F7）：运行下一行代码；如果下一行是对函数的调用，不进入函数体

Step Into（shift F7）：运行下一行代码，如果下一行是对函数的调用，则进入函数体；

Continue（Ctrl F7）：运行到下一个断点处；

Run to Cursor(Shift F4)：运行到光标所在出；

原文链接：https://blog.csdn.net/m0_61723200/article/details/121749949

## debug遇到的问题

无法查看直接查看stl变量

解决方法：

就是*&v[i]@n:取v[i]的地址，查看往后n个变量的值

但是仍然有限制，前提必须vector容器已经开好了空间，提前给vector开空间大小的n可以是变量。

还可以通过输入q.front(), q.size() 等来查看容器内数据。

查看不了：vector< string >、二维vector等等，总之dev的调试就是一坨屎

![image-20250225161152633](devc++设置.assets/image-20250225161152633.png)

https://tencentcloud.csdn.net/67690156f3b8a55e4e994ab7.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MzA1MjQwMSwiZXhwIjoxNzQxMDc0MjM5LCJpYXQiOjE3NDA0Njk0MzksInVzZXJuYW1lIjoiTHd5YW4xMjcifQ.5DnL5lUmKDcuaJSB6Ds5L0pLk096GaHjFS3_hKS0Wfo&spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-1-137097633-blog-102540542.235%5Ev43%5Econtrol&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Eactivity-1-137097633-blog-102540542.235%5Ev43%5Econtrol&utm_relevant_index=2

https://blog.csdn.net/xiaoh1999/article/details/102540542

## 使用c11标准

工具 -> 编译选项 -> 编译器：添加

```
-std=c++11
```

并勾选编译时加入以下命令