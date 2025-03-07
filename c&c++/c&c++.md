# O3优化

https://blog.csdn.net/m0_37870649/article/details/109139204

 一般来说，如果不指定优化标识的话，gcc就会产生可调试代码，每条指令之间将是独立的：可以在指令之间设置断点，使用gdb中的 p命令查看变量的值，改变变量的值等。并且把获取最快的编译速度作为它的目标。

当优化标识被启用之后，gcc编译器将会试图改变程序的结构（当然会在保证变换之后的程序与源程序语义等价的前提之下），以满足某些目标，如：代码大小最小或运行速度更快（只不过通常来说，这两个目标是矛盾的，二者不可兼得）。

- O1优化会消耗少多的编译时间，它主要对代码的分支，常量以及表达式等进行优化。对程序做部分编译优化，对于大函数,优化编译占用稍微多的时间和相当大的内存。使用本项优化，编译器会尝试减小生成代码的尺寸，以及缩短执行时间，但并不执行需要占用大量编译时间的优化。


- O2会尝试更多的寄存器级的优化以及指令级的优化，它会在编译期间占用更多的内存和编译时间。 


- O3在O2的基础上进行更多的优化，例如使用伪寄存器网络，普通函数的内联，以及针对循环的更多优化。 


- Os主要是对代码大小的优化，我们基本不用做更多的关心。 通常各种优化都会打乱程序的结构，让调试工作变得无从着手。并且会打乱执行顺序，依赖内存操作顺序的程序需要做相关处理才能确保程序的正确性。  


C++程序中的O2开关如下所示：

```
#pragma GCC optimize(2)
```

同理O1、O3优化只需修改括号中的数即可。

只需将这句话放到程序的开头即可打开O2优化开关。

开启O3优化：

```
#pragma GCC optimize(3,"Ofast","inline")
```

在不同的gcc配置和目标平台下，同一个标识所采用的优化种类也是不一样的，这可以使用-Q --help =optimizers来获取每个优化标识所启用的优化选项。

**p.s o3优化后无法debug**



# c文件操作

https://blog.csdn.net/m0_70811813/article/details/127218742



# fread

**fread 函数作用 :** 从文件中读取若干字节数据到内存缓冲区中 ;

**fread 函数原型 :**

```
size_t fread(void *buffer, size_t size, size_t count, FILE *stream);
```

**void \*buffer 参数 :** 将文件中的二进制数据读取到该缓冲区中 ;

**size_t size 参数 :** 读取的 基本单元 字节大小 , 单位是字节 , 一般是 buffer 缓冲的单位大小 ;

- 如果 buffer 缓冲区是 char 数组 , 则该参数的值是 sizeof(char) ;
- 如果 buffer 缓冲区是 int 数组 , 则该参数的值是 sizeof(int) ;

**size_t count 参数 :** 读取的 基本单元 个数 ;

**FILE \*stream 参数 :** 文件指针 ;

**size_t 返回值 :** 实际从文件中读取的 基本单元 个数 ; 读取的字节数是 基本单元数 * 基本单元字节大小 ;



# 快速读写

```c
char buf[1 << 20], *p1, *p2;  //buf缓存，p1为指向缓存开头的指针，p2为指向缓存结束的指针
							  //1 << 20即2^20
char gc() { 
    if (p1 == p2) { // 判断是否第一次执行或读取完缓存
        p1 = buf; // 重置开头指针
        p2 = buf + fread(buf, 1, 1 << 20, stdin); //读取数据， 重置结尾指针
        if (p1 == p2) return EOF;
    }
    return *p1++; // 返回第一个字符并指针后移
}

inline int read() { //普通的快读
    int f = 1, x = 0;
    char c = gc();
	while (c < '0' || c > '9') {
	    if (c == '-') f = -1;
	    c = gc();
	}
	while (c >= '0' && c <= '9') x = x * 10 + c - '0', c = gc();
	return f * x;
}

//或者宏函数版本

char buf[1<<20],*p1,*p2;
#define gc() (p1 == p2 ? (p2 = buf + fread(p1 = buf, 1, 1 << 20, stdin), p1 == p2 ? EOF : *p1++) : *p1++)
#define read() ({\
    int x = 0, f = 1;\
    char c = gc();\
    while(c < '0' || c > '9') f = (c == '-') ? -1 : 1, c = gc();\
    while(c >= '0' && c <= '9') x = x * 10 + (c & 15), c = gc();\
    f * x;\
})

```

写操作使用putchar最快。



# 整形提升

## 什么是整型提升

> 在C语言中，整型提升（integer promotion）是指当进行表达式运算时，比较小的整数类型会自动转换成较大的整数类型。这种类型转换是根据C语言的类型提升规则执行的，目的是为了保证表达式中的操作数具有相同的类型。

## 整型提升适合应用的情况

> ① 当使用算术运算符（如加法、减法、乘法、除法）时，参与运算的操作数将自动进行整型提升。比如，如果一个操作数是int类型，另一个操作数是char类型，那么char类型的操作数将被提升为int类型。
> ②当使用位运算符（如按位与、按位或、异或）时，参与运算的操作数也会自动进行整型提升。
> ③当使用关系运算符（如大于、小于、等于）时，比较的操作数也会进行整型提升。

## 举例1

在做[18. 四数之和](https://leetcode.cn/problems/4sum/)时，需要相加四个较大的int，因此会溢出，于是我这么写：

```c++
long sum = a + b + c + d;
```

立刻报错了：

```
signed integer overflow: 2000000000 + 1000000000 cannot be represented in type 'value_type' (aka 'int') (solution.cpp)
```

因为这里四个数先做运算再赋值，先溢出后报错了，因此使用整型提升：

```c++
long sum = (long)a + b + c + d;
```

将a先变成long，则做运算时bcd都会自动整数提升，因此运算结果就是一个long

## 举例2

```c++
#include <stdio.h>
int main() {
    char a=-1;
    signed char b=-1;
    unsigned char c=-1;
    printf("%d %d %d",a,b,c);
    return 0;
}

// 输出：a=-1,b=-1,c=255
```

分析如下：

> a的原码，反码，补码分别为：
> 1000000 00000000 00000000 00000001 原码
> 11111111 111111111 111111111 111111110 反码
> 11111111 111111111 111111111 111111111 补码
> 但是char类型只能储存8个位置
> 所以 11111111------a
>
> 我们还可以看见输出是以%d的形式输出a，所以这时要进行整型提升，a是char a,是默认有符号的，所以最高位1为符号位
>
> 进行整型提升时我们前面全补1，补成32位
> 即11111111 11111111 11111111 11111111
> 这是反码，其原码代表的值为-1.

> 同时，不过没有标注有无符号，默认就是有符号的
> 所以，char 和 signed char是一样的
> 所以 11111111------b
>
> 剩下的部分与a相同

> c就与a和b不同了
> 到8个比特位为止，都一样
> c也是储存的8个1
> 11111111-----c
>
> 但是，c时unsigne char，是一个无符号的数
> 其整型提升时补0，而不补1
> 变成 00000000 00000000 00000000 11111111
>
> 这时首位为0，是正数，正数的原码反码补码相同，所以此数为255.

## 举例3

```c++
#include<stdio.h>
int main() {
    char a=-128;
    printf("%u/n",a);
    return 0;
}

// 输出：4294967168
```

> 首先我们要知道 %u是以10进制的形式，打印无符号的整数。 %d是以10进制的形式，打印有符号的整数。
> a的原码，反码，补码如下
> 10000000 00000000 00000000 10000000 原码
> 11111111 11111111 11111111 01111111 反码
> 11111111 11111111 11111111 10000000 补码
>
> 但是char类型只能储存8个 所以 10000000-----a
> char是有符号的类型，所以整型提升前面补1,整型提升结果如下
> 11111111 11111111 11111111 10000000
>
> 此时输出为%u，是一个无符号数，所以首字符1不是符号位
> 即这一个正数，那么它的原码反码补码相同
> 11111111 11111111 11111111 10000000次数化为进制即是4294967168

## 小结

在整型提升中，需要注意以下几个方面：

> ① 自动类型提升：当进行运算时，参与运算的整型数值会自动提升为较大的类型（比如int提升为long）。
> ②有符号整型提升：有符号整型会在扩展时，将最高位的符号位进行复制。例如，将一个有符号的8位整数提升为16位整数时，符号位将被复制至高8位。
> ③无符号整型提升：无符号整型在提升时，会在高位补0。例如，将一个无符号的8位整数提升为16位整数时，高8位将被补0。
> ④类型转换：将一个大范围的整数类型转换为一个较小范围的整数类型时，高位的部分可能会被截断，导致数据丢失。因此，在进行类型转换时需要注意数据溢出的情况。
> ⑤存储范围：整型提升可能会导致数值超过存储范围，从而产生溢出或不确定的结果。特别是在进行算术运算时，需要注意结果是否会超出存储范围。
> ⑥带符号和无符号整型之间的转换：在带符号整型和无符号整型之间进行转换时，需要注意符号位的处理。如果将带符号整型转换为无符号整型，可能会导致符号位被解释为数值位，从而改变数值的含义。反之亦然，将无符号整型转换为带符号整型时，可能会导致数值位被解释为符号位，从而改变数值的正负。

原文链接：https://blog.csdn.net/qq_57425280/article/details/132305474



# C语言字符串

- 我们可以把字符串储存在char类型的数组中，如果char类型的数组末尾包含一个表示字符串末尾的空字符\0，则该数组中的内容就构成了一个字符串。

- 因为字符串需要用\0结尾，所以在定义字符串的时候，字符数组的长度要预留多一个字节用来存放\0，\0就是数字0
- 字符串也可以存放中文和全角的标点符号，一个中文字符占两个字节（GBK编码）。char strname[21]用于存放中文的时候，最多只能存10个汉字。
- 一个字符占用一字节的内存，字符串定义时数组的大小就是字符串占用内存的大小
- 数组名是数组元素的首地址，字符串是字符数组，所以在获取字符串的地址的时候，不需要用&取地址

```c
char charArray[21];  // 定义一个最多存放20个英文字符或十个中文的字符串

char charArray[] = {'H','e','l','l','o'};    
// 声明并初始化一个字符数组,这个数组的长度实际上为 6 ，因为会自动添加一个字符串结束符 '\0'。

char charArray[] = "Hello World!";    // 声明并初始化一个字符数组
```

## 常用函数

```c
size_t strlen(const char*  str);
```

- 功能：计算字符串的有效长度，不包含 \0
- 返回值：返回字符串的字符数 
- strlen 函数计算的是字符串的实际长度，遇到第一个\0结束
- 函数返回值一定是size_t，是无符号的整数，即typedef unsigned int size_t
- 如果您只定义字符串没有初始化，求它的长度是没意义的，它会从首地址一直找下去，遇到0停止
- 很多人对 sizeof 和 strlen 有点分不清楚，sizeof 返回的是变量所占的内存数，不是实际内容的长度

```c
char *strcpy(char* dest, const char* src);
```

- 功能: 将参数src字符串拷贝至参数dest所指的地址
- 返回值: 返回参数dest的字符串起始地址
- 复制完字符串后，在dest后追加0
- 如果参数dest所指的内存空间不够大，可能会造成缓冲溢出的错误情况

```c
char * strncpy(char* dest, const char* src, const size_t n);
```

- 功能：把src前n字符的内容复制到dest中
- 返回值：dest字符串起始地址
- 如果src字符串长度小于n，则拷贝完字符串后，在dest后追加0，直到n个
- 如果src的长度大于等于n，就截取src的前n个字符，不会在dest后追加0
- dest必须有足够的空间放置n个字符，否则可能会造成缓冲溢出的错误情况

```c
char *strcat(char* dest,const char* src);
```

- 功能：将src字符串拼接到dest所指的字符串尾部
- 返回值：返回dest字符串起始地址
- dest最后原有的结尾字符0会被覆盖掉，并在连接后的字符串的尾部再增加一个0
- dest要有足够的空间来容纳要拼接的字符串，否则可能会造成缓冲溢出的错误情况

```c
char *strncat (char* dest, const char* src, const size_t n);
```

- 功能：将src字符串的前n个字符拼接到dest所指的字符串尾部
- 返回值：返回dest字符串的起始地址
- 如果n大于等于字符串src的长度，那么将src全部追加到dest的尾部，如果n小于字符串src的长度，只追加src的前n个字符
- strncat会将dest字符串最后的0覆盖掉，字符追加完成后，再追加0
- dest要有足够的空间来容纳要拼接的字符串，否则可能会造成缓冲溢出的错误情况

```c
int strcmp(const char *str1, const char *str2 );
```

- 功能：比较str1和str2的大小

- 返回值：相等返回0，str1大于str2返回1，str1小于str2返回-1

```c
int strncmp(const char *str1, const char *str2, const size_t n);
```

- 功能：比较str1和str2前n个字符的大小


- 返回值：相等返回0，str1大于str2返回1，str1小于str2返回-1


- 两个字符串比较的方法是比较字符的ASCII码的大小，从两个字符串的第一个字符开始，如果分不出大小，就比较第二个字符，如果全部的字符都分不出大小，就返回0，表示两个字符串相等

```c
char *strchr(const char *s,const int c);
```

- 返回一个指向在字符串s中第一个出现c的位置，如果找不到，返回0

```c
char *strstr(const char* str,const char* substr);
```

- 功能：检索子串在字符串中首次出现的位置
- 返回值：返回字符串str中第一次出现子串substr的地址；如果没有检索到子串，则返回0
- 时间复杂度为`O(n ^ 2)`

# char与int转换

**法一**

```
char a = '9';
int A = a - '0';
```

**法二**

**atoi**

- 功能: 把字符串转换成整型数，可以将一个字符数组转化为整型

- 函数说明: atoi()会扫描参数nptr字符串，检测到第一个数字或正负符号时开始做类型转换，之后检测到非数字或结束符 \0 时停止转换，返回整型数。
- 原型: int atoi(const char *nptr);
- 头文件: #include <stdlib.h>

```
char str[] = "12345.67";
int a = atoi(str);
// 输出结果：12345
```

**itoa**

- 功 能:把一整数转换为字符串
- 用 法:char *itoa(int value, char *string, int radix);
- 详细解释:itoa是英文integer to array(将int整型数转化为一个字符串,并将值保存在数组string中)的缩写
- 参数：
  - value: 待转化的整数。
  - radix: 是基数的意思,即先将value转化为radix进制的数，介于2-36，比如10表示10进制
  - string: 保存转换后得到的字符串。
  - 返回值：
  - char * : 指向生成的字符串， 同*string。

https://blog.csdn.net/ITWorldView/article/details/135888041

# 多线程

**linux pthread**

https://blog.csdn.net/sjsjnsjnn/article/details/126062127

https://www.cs.cmu.edu/afs/cs/academic/class/15492-f07/www/pthreads.html

**windows**

https://blog.csdn.net/mary288267/article/details/132585472



# sleep函数

- windows下支持，需要引入头文件windows.h，切记Sleep首字母大写，单位为ms

  ```c++
  #include <iostream>
  #include <windows.h>
  using namespace std;
  
  void main() {
      //睡眠5秒再输出
      Sleep(5000);
      std::cout << "Hi,Gril!" << std::endl;
  }
  ```

- linux下支持，需要引入头文件unistd.h

  ```cpp
  #include <iostream>
  #include <unistd.h>
  using namespace std;
  int main() {
  	//5秒后输出Hi,Gril!
  	sleep(5);
  	std::cout << "Hi,Gril!" << std::endl;
  
  	//3000000微妙(相当于3秒)输出Hi,Boy!
  	usleep(3000000);
  	std::cout << "Hi,Boy!" << std::endl;
  
  	return 0;
  }
  ```



# struct&typedef struct

```c++
struct 结构体名 {
	结构体成员名
}结构体变量名;
定义时：struct 结构体名 结构体变量名;
引用时：结构体变量名.结构体成员名

typedef struct 结构体名{
	结构体成员名
}结构体类型名;
定义时：结构体类型名 结构体变量名;
引用时：结构体变量名.结构体成员名
```

typedef是在基本类型的基础上定义类型的同义字。注意typedef并不产生新的类型。例如 typedef int exam_score；这里的exam_score 就是一种基本类型，它的意义等同于 int，那么即可以用它来定义整型变量，例如：exam_score LIMING。

对于typedef struct 结构体名{}结构体类型名，结构体类型名就相当于"struct 结构体名"，只是使用typedef给其定义了一个别名。

我们都知道C中是不允许在定义结构体变量时省略struct关键字，当然C++中用不用的结果都是一样的，所以有时候为了平台的兼容性以及代码更好的可读性，我们使用typedef关键字，而且typedef的作用不止于此。

typedef只是给你的数据类型定义一个别名，不过使用typedef将能使你的代码更直观简洁，而且在代码跨平台移植时将提供极大的便利。

https://blog.csdn.net/WalterBrien/article/details/126141547

```c++
//代码1（c语言）
typedef struct test3{
	int a;
	int b;
	int c;
}test4;
//代码2（c++）
struct test3{
	int a;
	int b;
	int c;
}test4;
```

- **在c语言中**
  - **`typedef` 的作用**：在 C 语言里，`typedef` 关键字用于为已有的数据类型创建一个新的类型别名。在这段代码中，`typedef` 为 `struct test3` 这个结构体类型定义了一个新的名称 `test4`。
  - 类型使用方式：
    - 若要声明 `struct test3` 类型的变量，有两种方式。一是使用完整的结构体类型名，如 `struct test3 var1;`；二是使用 `typedef` 定义的别名，如 `test4 var2;`。
    - 这里 `test4` 就相当于 `struct test3` 的别名，可直接用于变量声明，而无需每次都写 `struct` 关键字。
- **在c++中**
  - **无 `typedef` 的情况**：在 C++ 中，结构体类型的使用更为简洁。当定义结构体时，若不使用 `typedef`，`struct` 关键字后面的标识符（如 `test3`）本身就直接成为了一个可用的类型名。
  - 类型使用方式：
    - 可以直接使用 `test3` 来声明变量，例如 `test3 var3;`。
    - 代码中的 `test4` 实际上是一个 `struct test3` 类型的变量实例，而非类型别名。这和 C 语言中使用 `typedef` 定义别名的效果是不同的。

https://github.com/guaguaupup/cpp_interview/blob/main/C%2B%2B.md

# 结构体使用malloc

如果定义一个结构体类型的普通变量，可以不malloc动态申请内存，CPU会为这个结构体变量分配内存。

如果定义的是一个结构体的指针，CPU会为这个指针开辟内存，但是此时这个大小是4（如果是32位的CPU的话），所以这个空间不足以存储结构体的数据成员，就会引发错误，此时必须要malloc申请一个，结构体类型大小的动态内存，用于数据成员存储使用。

```c++
struct node *creat1() { //生成单链表 
	struct node *head, *tail, *p;
	int e;
	head = tail = (struct node *)malloc(LENG);
	cin >> e;
	while (e != 0) {
		p = (struct node *)malloc(LENG);
		p->data = e;
		tail->next = p;
		tail = p;
		cin >> e;
	}
	tail->next = NULL;
	return head;
}
```

拓展一下，malloc是动态申请，调用后会根据虚拟映射表去找物理内存，此时内核会先产生一个请求内存异常，然后根据这个异常再去为程序分配malloc的内存。

另外注意，调用malloc后一定要free，且在free掉之后要赋值NULL，这个操作是一对一的。

https://blog.csdn.net/qq_45617555/article/details/116402188



# limits库

limits.h是 C 语言中的标准库头文件之一，它定义了各种整数类型的限制和属性。通过包含该头文件，可以使用其中定义的常量和宏来获取与整数类型相关的一些信息。

```c
#include <limits.h>  // c
#include <climits>  // c++
```

- CHAR_BIT：一个字节中比特的数量，通常为 8。

- INT_MIN、INT_MAX：int 类型的最小值和最大值。（以补码表示）
- LONG_MIN、LONG_MAX：long 类型的最小值和最大值。（以补码表示）
- LLONG_MIN、LLONG_MAX：long long 类型的最小值和最大值。（以补码表示）
- UINT_MAX：unsigned int 类型的最大值。
- ULONG_MAX：unsigned long 类型的最大值。
- ULLONG_MAX：unsigned long long 类型的最大值。
- SCHAR_MIN、SCHAR_MAX：signed char 类型的最小值和最大值。
- UCHAR_MAX：unsigned char 类型的最大值。

- SHRT_MIN、SHRT_MAX：short 类型的最小值和最大值。

- USHRT_MAX：unsigned short 类型的最大值。

- MB_LEN_MAX：多字节字符的最大字节数。

- FLT_MIN、FLT_MAX：float 类型的最小正值和最大值。

- DBL_MIN、DBL_MAX：double 类型的最小正值和最大值。

- LDBL_MIN、LDBL_MAX：long double 类型的最小正值和最大值。


上面列举的只是 <limits.h> 头文件中定义的一部分常量和宏，还有其他与整数类型相关的信息定义在其中。这些定义在不同平台和编译器下可能会有所不同，因此在使用时需要考虑到具体的编译环境。通过引入这个头文件，可以更方便地获取整数类型的限制和属性，从而进行合理的计算和处理。


数据类型的范围限制：通过使用 <limits.h> 中定义的常量，可以获取各种整数类型的最小值和最大值，以及无符号整数类型的最大值。这对于确保程序处理数据不超出类型的范围非常重要，避免溢出或其他意外情况。

```c
#include <stdio.h>
#include <limits.h>
 
int main() {
    printf("int类型的范围: %d 到 %d\n", INT_MIN, INT_MAX);
    printf("unsigned int类型的范围: 0 到 %u\n", UINT_MAX);
    printf("float类型的范围: %e 到 %e\n", FLT_MIN, FLT_MAX);
    return 0;
}
```

数值边界检查：应用程序可能需要对输入的数据进行边界检查，确保输入的数值不超过特定类型的最大值或小于最小值。通过使用 <limits.h> 中定义的常量，可以轻松地比较输入值与类型范围的边界。

```c
#include <stdio.h>
#include <limits.h>
 
void checkRange(int value) {
    if (value > INT_MAX) {
        printf("数值超出int类型的最大值\n");
    } else if (value < INT_MIN) {
        printf("数值小于int类型的最小值\n");
    } else {
        printf("数值在int类型的范围内\n");
    }
}
 
int main() {
    int num = 100000;
    checkRange(num);
    return 0;
}
```

循环计数器：在循环中，可能需要一个计数器来迭代一定次数。可以使用 <limits.h> 中定义的常量来获取特定整数类型的最大值，将其作为循环的计数器终止条件。

```c
#include <stdio.h>
#include <limits.h>
 
int main() {
    for (unsigned int i = 0; i < UINT_MAX; i++) {
        // 在这里进行循环操作
    }
    printf("循环次数：%u\n", UINT_MAX);
    return 0;
}
```

字节操作：<limits.h> 中的 CHAR_BIT 常量定义了一个字节中比特的数量，通常为 8。这个常量可以在处理二进制数据、位运算等场景中使用。

```c
#include <stdio.h>
#include <limits.h>
 
int main() {
    printf("一个字节中的比特数量：%d\n", CHAR_BIT);
    return 0;
}
```

浮点数范围检查：<limits.h> 中定义了浮点数类型（如 float、double、long double）的最小正值和最大值。这些常量可以用于验证浮点数是否在特定范围内。

```c
#include <stdio.h>
#include <float.h>
 
int main() {
    float num = 3.14;
    if (num > FLT_MAX) {
        printf("浮点数超出float类型的最大值\n");
    } else if (num < FLT_MIN) {
        printf("浮点数小于float类型的最小值\n");
    } else {
        printf("浮点数在float类型的范围内\n");
    }
    return 0;
}
```

总之，<limits.h> 头文件中定义的常量和宏适用于各种需要获取整数类型限制和属性的场景，包括范围检查、循环计数、位操作等。通过利用这些定义，可以使程序更可靠、更符合预期，并减少错误和异常情况的发生。

https://blog.csdn.net/Colorful___/article/details/131832933



# string类

## 构造

```c++
#include <iostream>
#include <string>

string s1();  // 无参构造函数
string s2("hello world");  // 常量字符串构造函数
string s3(s2);  // 拷贝构造函数
// 拷贝构造还可以写成下面这样子
string s4 = s2;

string str ("12345678");
char ch[ ] = "abcdefgh";
string a; //定义一个空字符串
string str_1 (str); //构造函数，全部复制
string str_2 (str, 2, 5); //构造函数，从字符串str的第2个元素开始，复制5个元素，赋值给str_2
string str_3 (ch, 5); //将字符串ch的前5个元素赋值给str_3
string str_4 (5,'X'); //将 5 个 'X' 组成的字符串 "XXXXX" 赋值给 str_4
string str_5 (str.begin(), str.end()); //复制字符串 str 的所有元素，并赋值给 str_5
```

## 赋值

```c++
#include <iostream>
#include <string>

// 以下三种方法 
string s1("hello");
string s2;
s2 = s1;

s2 = "world";

s2 = 'A';
```

## 遍历

### []下标遍历

- string类是提供了[]运算符重载的，这也就意味着string可以让我们像访问数组一样利用下标来访问元素

### 利用迭代器遍历

- 迭代器是容器内的内嵌类型，其实本质上是被重命名过的一个指针变量，迭代器是使用非常多的一个工具
- begin()函数返回的是string字符串的首位置，end()函数返回的是string字符串最后一个位置（即最后一个字符的下一个位置）

```C++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello world");
    string::iterator it = s.begin();
    while (it != s.end()) {
        cout << *it;
        ++it;
    }
    return 0;
}
```

### 范围for遍历

- 范围for是c++提供的一个非常方便的访问方式，它可以自动取对象的内容并且自动向后访问自动停止，范围for的底层实现其实是迭代器，我们还可以利用auto关键字来配合使用范围for

```C++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello world");
    for (auto ch : s) {
        cout << ch;
    }
    return 0;
}
```

## 反向迭代器

- 使用begin()函数和end()函数的迭代器遍历是正向迭代器，反向迭代器顾名思义，就是顺序反过来了，它的用法与正向迭代器非常类似。

- rbegin()函数返回的是string字符串的最后一个有效字符，rend()函数返回的是string字符串的第一个字符的前一个位置。

```C++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello world");
    string::reverse_iterator rit = s.rbegin();
    while (rit != s.rend()) {
        cout << *rit;
        rit++;  // 反向迭代器也是++
    }
    return 0;
}
// 输出 dlrow olleh
```

## const修饰的迭代器

- 我们前面介绍的所有迭代器都是普通类型的迭代器，这种迭代器的权限是可读可写的，但如果针对的是const修饰的string对象，我们就不能再使用可读可写的迭代器了，所以C++还提供了const修饰的迭代器，其权限只可读不可写，下面以begin()函数示例，end()、rbegin()、rend()函数都是一样的。

```c++
#include <iostream>
#include <string>
using namespace std;

void Func(const string& s) {
    string::const_iterator cit = s.begin();
    // 读操作
    while (cit != s.end()) {
        cout << *cit;
        cit++;
    }

    // 不能进行写操作，会报错
    // cit = s.begin();
    // while (cit != s.end()) {
    //     (*cit) += 1;
    //     cout << *cit;
    //     cit++;
    // }

    string::const_reverse_iterator crit = s.rbegin();
    // 读操作
    while (crit != s.rend())
    {
        cout << *crit;
        crit++;
    }

    // 不能进行写操作，会报错
    // crit = s.rbegin();
    // while (crit != s.rend()) {
    //     (*crit) += 1;
    //     cout << *crit;
    //     crit++;
    // }
}

int main () {
    string s("hello world");
    Func(s);
    return 0;
}

```

## size函数和capacity函数

- size()函数返回的是string对象的元素个数，即有效字符的个数，而capacity()函数返回的是string对象的容量
- length()函数和size()函数一模一样

## reserve函数和resize函数

- reserve函数是扩容函数，可以增大capacity的值，resize其实也是扩容函数，但resize改变的是size的值，当size的值增大时自动触发string的扩容机制从而也增大了capacity的值，并且resize在增带size值的时候还会对没有字符的位置初始化，如果没有指定初始化内容就默认初始化为’\0’，而reserve不会进行初始化。
- 另外，一般情况下reserve函数和resize函数都不能缩容（这个由编译器决定），resize函数只能够将size的值变小，但不会让capacity的值也变小

```c++
#include <iostream>
#include <string>
using namespace std;

int main() 	{
    string s("hello");
    cout << s << endl;
    cout << "size:" << s.size() << endl;
    cout << "capacity:" << s.capacity() << endl;
    cout << endl;
    // 输出为 hello 5 5

    s.reserve(25);
    cout << s << endl;
    cout << "size:" << s.size() << endl;
    cout << "capacity:" << s.capacity() << endl;
    cout << endl;
    // 输出为 hello 5 25

    s.resize(50, 'A');
    cout << s << endl;
    cout << "size:" << s.size() << endl;
    cout << "capacity:" << s.capacity() << endl;
    cout << endl;
    // 输出为 helloAAAAAAAAAAAAAAAAAAAAAAAAA 30 30
    return 0;
}

```

## clear函数和empty函数

- clear函数清空string，使其size变为0，无返回值
- empty函数判断string是否为空，返回一个bool

```cpp
string s("asdfasdf");
s.clear();
bool flag = s.empty();
```

## string的插入操作

### push_back函数

- push_back函数可以实现string对象的插入操作，但是需要注意的是push_back函数只能够尾插入一个字符，不能插入字符串

```
string s("abc");
char c = 'A';
s.pushback(c);
```

### append函数

- append函数就比push_back函数多样，它可以插入字符串，可以插入另一个string对象，而且可以指定n个字符插入。

```c++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello ");
    s.append("world");  // 插入常量字符串
    string str("world");
    s.append(str);  // 插入另一个string对象
    return 0;
}
```

### +=运算符

- 前面两种其实都不是最好用的，最好用的插入操作应该是使用+=运算符，它不仅可以插入新的string对象，还可以插入常量字符串，也可以插入单个字符，我们平时使用最多的方式也是这个方式。

```c++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello ");
    s += "world"; // 插入常量字符串
    string str("world");
    s += str;  // 插入新的string对象
    s += 'A';  // 插入单个字符
    return 0;
}
```

### insert函数

- insert函数与上面的插入方法都不同，insert函数可以在任意的指定位置进行插入。insert函数的插入也是非常多样化的，它可以在任意的指定位置插入一个新的string对象、一个常量字符串、一个常量字符串的n个字符、一个字符等等。插入即将字符串后移再放入。

```c++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello");

    // 在下标为0的位置插入一个新的string对象
    string str("hello world");
    s.insert(0, str);

    // 在下标为0的位置插入一个常量字符串
    s.insert(0, "hello world");

    // 在下标为0的位置插入一个常量字符串的前3个字符
    s.insert(0, "hello world", 3);

    // 在下标为0的位置插入一个字符x
    s.insert(0, 1, 'x');
    s.insert(s.begin(), 'x');

    // 在下标为0的位置插入三个字符x
    s.insert(0, 3, 'x');

    return 0;
}
```

## string的删除操作

### pop_back函数

- C++11提供了pop_back接口，可以实现string对象的尾删操作。

```
s.pop_back();
```

### erase函数

- erase函数可以删除任意指定位置的n个字符。

```c++
#include <iostream>
#include <string>
using namespace std;

int main () {
    string s("hello world");

    // 删除下标为3位置的一个字符
    s.erase(3, 1);
    s.erase(s.begin() + 3);

    // 删除以下标为3位置为起点的3个字符
    s.erase(3, 3);
    s.erase(s.begin() + 3, s.begin() + 6);

    // 删除以下标为3位置为起点往后的所有字符
    s.erase(3);
    
    return 0;
}

```

## string的swap函数

```c++
#include <iostream>
#include <string>
using namespace std;

int main (){
    string s1("hello world");
    string s2("string");
    s1.swap(s2);

    cout << s1 << endl;
    cout << s2 << endl;
    return 0;
}
```

- 但是STL库里的swap函数和string类提供的swap函数是有很大区别的（仅限于C++11之前，C++11引入了右值引用以后二者没有效率差别），string类提供的swap函数的效率更高，因为对于两个string对象来说，要实现交换只需要交换二者的指针指向的内容即可；而STL库提供的swap函数它是函数模板，它可以实现任意类型的交换，那就只能够将内容进行交换，也就是深拷贝，深拷贝的代价是很大的我们下面也会提到，所以其实现的效率会比较低。因此，对于string对象来说，更推荐使用string类提供的swap函数。

## c_str函数

- c_str函数可以返回string对象对应的char * 指针，这个函数可以很好地配合C语言的一些函数接口使用，因为C语言没有string类，所以有一些字符串操作的函数接口需要传递的参数是char * 类型的指针。

```c++
#include <iostream>
#include <string>
#include <cstring>
using namespace std;

int main(){
    string str("hello world");
    char* cstr = new char[str.size() + 1];
    strcpy(cstr, str.c_str());
    cout << cstr << endl;
    return 0;
}
```

## substr函数

- substr函数是用来返回string字符串的一个任意子串，我们可以通过设定起始位置pos和子串长度len来获取子串。

```c++
#include <iostream>
#include <string>
using namespace std;

int main() {
    string s1("hello world");
    // 取出子串"world"
    string s2 = s1.substr(6, 5);
    cout << s2 << endl;
    return 0;
}
```

## string的查找函数

### find函数

- find函数使用得是比较多的，它可以查找string对象、常量字符串或者是一个字符，并且可以设定pos值来规定查找的起始位置，默认从0下标开始查找。

```c++
position = s.find("abc");
// 如果没找到，返回一个特别的标志c++中用npos表示，我这里npos取值是4294967295，写成-1也行，计算机中和npos相等
if (position != s.npos) {
	printf("position is : %d\n" ,position);
}
```

### rfind函数

- rfind函数和find函数使用方法是一样的，只不过find函数是顺着查找，而rfind是倒着查找。find函数和rfind函数的区别就是查找方向不同。

### 若干不同的find类型查找

```cpp
int main() {
    string str = "hello world";
 
    // 从头开始查找字符 l
    size_t pos1 = str.find('l');
    cout << "pos1:" << pos1 << endl;
 
    // 从尾开始查找字符 l
    size_t pos2 = str.rfind('l');
    cout << "pos2:" << pos2 << endl;
 
    // 从前往后查找"world"中任意一个字符(即'w'、'o'、'r'、'l'、'd')第一次出现的位置
    size_t pos3 = str.find_first_of("world");
    cout << "pos3:" << pos3 << endl;
 
    // 从后往前查找"world"中任意一个字符(即'w'、'o'、'r'、'l'、'd')第一次出现的位置
    size_t pos4 = str.find_last_of("world");
    cout << "pos4:" << pos4 << endl;
 
    // 从前往后查找不在"helord"中任意一个字符(即'h'、'e'、'l'、'o'、'r'、'd')第一次出现的位置
    size_t pos5 = str.find_first_not_of("helord");
    cout << "pos5:" << pos5 << endl;
 
    // 从后往前查找不在"helord"中任意一个字符(即'h'、'e'、'l'、'o'、'r'、'd')第一次出现的位置
    size_t pos6 = str.find_last_not_of("helord");
    cout << "pos6:" << pos6 << endl;
 
    return 0;
}
```

### 查找函数的使用

- 在计算机网络的学习中，我们有时候要对URL进行分割，URL可以分为协议、域名和uri，在这个场景中我们可以利用到字符串的查找函数

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string url("https://legacy.cplusplus.com/reference/string/string/rfind/");
    cout << url << endl;
    
    // 提取协议
    string protocol;
    size_t pos1 = url.find("://");
    // 查找成功
    if (pos1 != string::npos) {
        protocol = url.substr(0, pos1);
        cout << "协议:" << protocol << endl;
    } else {
        cout << "非法url" << endl;
    }

    // 提取域名
    string domainName;
    size_t pos2 = url.find('/', pos1 + 3);
    // 查找成功
    if (pos2 != string::npos) {
        domainName = url.substr(pos1 + 3, pos2 - (pos1 + 3));
        cout << "域名:" << domainName << endl;
    } else {
        cout << "非法url" << endl;
    }

    // 提取uri
    string uri = url.substr(pos2);
    cout << "uri:" << uri << endl;
    return 0;
}

```

## getline函数

- 当我们用cin对string进行流提取时，由于cin遇到空格和换行会停止读取，所以我们如果想要读取带有空格的字符串就会出现读取不完整的现象。此时就需要用到getline函数，getline函数可以获取一行字符串，即遇到换行符才会停止读取，遇到空格不会停止。

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    // 键盘输入字符串"hello world"
    string s;
    // cin >> s; // 这种写法遇到空格会停止
    getline(cin, s);
    cout << s << endl;
    return 0;
}

```

## string与char相互转化

1. string 转 const char*

```c++
string str = "abc";
const char *pa = str.data();  // data()函数不会添加'\0'结束符
cout << pa << endl;
const char *p = str.c_str();  // c_str()会在数据的末尾添加'\0'结束符，多数用于使用字符串场合。
cout << p << endl;
```

​        但是，上面这种用法很不安全，因为p最后指向的内容是垃圾值，str对象被析构了。而且c_str()返回的是一个临时指针，不能对其进行操作。

2. string 转 char*

```c++
string s = "abcd";
char *ch;
ch = (char*)malloc((s.length()) * sizeof(char));
s.copy(ch, s.length(),0);  // 把当前串中以0开始的s.length()个字符拷贝到以ch为起始位置的字符数组中，返回实际拷贝的数目
cout << ch;
```

3. string 转 char[]

   1）strcpy_s

   ```c++
   char ch[5];
   string s = "abcd";
   strcpy_s(ch, s.c_str());
   cout << ch << endl;
   ```

   2）遍历添加

   ```c++
   string pp = "helloworld";
   char p[20];
   int i;
   for (i = 0; i < pp.length(); i++) {
       p[i] = pp[i];
   }
   p[i] = '\0';  //添加结束符
   
   cout << p << endl；
   ```

4. const char * 或者 const char [] 或者 char [] 转 string

```c++
string a;
const char* b = "study";
const char c[] = "study well";
char p[20] = "helloworld";

a = b;
cout << a << endl;
cout << a.length() << endl;

a = c;
cout << a << endl;
cout << a.length() << endl;

a = p;
cout << a << endl;
cout << a.length() << endl;
```

5. char[] 转 char*  或者 const char[] 转 const char*

```c++
char a[20] = "hello world";
char* p = a;
cout << p << endl;

const char a1[20] = "hello world";
const char* p1 = a1;

cout << p1 << endl;
```

6. const char* 转char[]

   1）strcpy_s

   ```c++
   char arr[20];
   const char* tmp = "helloworld";
   strcpy_s(arr, tmp);
   
   cout << arr << endl;
   ```

   2）循环遍历

   ```c++
   char arr[20];
   const char* tmp = "hello world";
   int i = 0;
   while (*tmp != '\0')
   {
       arr[i++] = *tmp++;
   }
   arr[i] = '\0';  //添加结束符
   
   cout << arr << endl;
   ```

7. char* 转 string

```c++
char a[10] = "hello";
char *b = a;
string c = b;

cout << a << endl;
cout << b << endl;
cout << c << endl;
```

https://blog.csdn.net/pxxzhk/article/details/131637961

## string转化为整数函数

注意：stoi、stol、stoll 函数是C++11标准加入的，用g++编译器编译需要加参数：-std=c++11

### stoi

```
int stoi(const std::string& str, std::size_t* pos = 0, int base = 10);
int stoi(const std::wstring& str, std::size_t* pos = 0, int base = 10);
```

功能：将字符串str转成 有符号 int 整数

参数：

- str：字符串
- pos：存储将字符串str转成有符号整数，处理了str中字符的个数的地址，默认为NULL
- base：进制，10：十进制，8：八进制，16：十六进制，0：则自动检测数值进制，str是 0 开头为八进制，str是 0x 或 0X 开头是十六进制，默认为十进制

```c++
#include <iostream>
#include <string>
using namespace std;
 
int main(int argc, char *argv[])
{
    int a;
    size_t pos = 0;
    string str;
 
    str = "-1235";
    a = stoi(str);
    cout << "a = " << a << endl; //a = -1235
 
    str = "1235";
    a = stoi(str);
    cout << "a = " << a << endl; //a = 1235
 
    str = "  -12  35"; 
    a = stoi(str, &pos); //会舍弃空白符
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "  -12ab35";
    a = stoi(str, &pos);
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "0123";
    a = stoi(str);
    cout << "a = " << a << endl; //a = 123
 
    str = "0x123";
    a = stoi(str);
    cout << "a = " << a << endl; //a = 0
 
    return 0;
}
```

```c++
// stoi()函数指定转换字符串为八进制用法
#include <iostream>
#include <string>
using namespace std;
 
int main(int argc, char *argv[])
{
    int a;
    size_t pos = 0;
    string str;
 
    str = "0x123";
    a = stoi(str, NULL, 8); //base = 8，指定八进制
    cout << "a = " << a << endl; //a = 0
 
   str = "0123"; //(3 + 2*8 + 1*8*8)
    a = stoi(str, NULL, 0); //base = 0，自动检测数值进制
    cout << "a = " << a << endl; //a = 83
 
    str = "-12";
    a = stoi(str, &pos, 8); //-(2 + 1*8)
    cout << "a = " << a << endl; //a = -10
    cout << "pos = " << pos << endl; //pos = 3
 
    str = "12";
    a = stoi(str, &pos, 8); //2 + 1*8
    cout << "a = " << a << endl; //a = 10
    cout << "pos = " << pos << endl; //pos = 2
 
    str = "  -12  35"; 
    a = stoi(str, &pos, 8); //会舍弃空白符
    cout << "a = " << a << endl; //a = -10
    cout << "pos = " << pos << endl; //pos = 5
 
    // str = "  -a78"; 
    // a = stoi(str, &pos, 8); //数字前有字母，调用会崩掉
    // cout << "a = " << a << endl; 
    // cout << "pos = " << pos << endl; 
 
    return 0;
}
```

### stol()

```
long stol(const std::string& str, std::size_t* pos = 0, int base = 10);
long stol(const std::wstring& str, std::size_t* pos = 0, int base = 10);
```

功能：将字符串str转成 有符号 long 整数

参数：

- str：字符串
- pos：存储将字符串str转成有符号整数，处理了str中字符的个数的地址，默认为NULL
- base：进制，10：十进制，8：八进制，16：十六进制，0：则自动检测数值进制，str是 0 开头为八进制，str是 0x 或 0X 开头是十六进制，默认为十进制

```
#include <iostream>
#include <string>
using namespace std;
 
int main(int argc, char *argv[])
{
    long a;
    size_t pos = 0;
    string str;
 
    str = "-1235";
    a = stol(str);
    cout << "a = " << a << endl; //a = -1235
 
    str = "1235";
    a = stol(str);
    cout << "a = " << a << endl; //a = 1235
 
    str = "  -12  35"; 
    a = stol(str, &pos); //会舍弃空白符
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "  -12ab35";
    a = stol(str, &pos);
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "0123";
    a = stol(str);
    cout << "a = " << a << endl; //a = 123
 
    str = "0x123";
    a = stol(str);
    cout << "a = " << a << endl; //a = 0
 
    return 0;
}
```

```c++
// stol()函数指定转换字符串为十六进制用法
#include <iostream>
#include <string>
using namespace std;
 
int main(int argc, char *argv[])
{
    long a;
    size_t pos = 0;
    string str;
 
    str = "0x123";
    a = stol(str, NULL, 16); //base = 16，指定十六进制
    cout << "a = " << a << endl; //a = 291
 
   str = "0x123";
    a = stol(str, NULL, 0); //base = 0，自动检测数值进制
    cout << "a = " << a << endl; //a = 291
 
    str = "-12";
    a = stol(str, &pos, 16); //-(2 + 1*16)
    cout << "a = " << a << endl; //a = -18
    cout << "pos = " << pos << endl; //pos = 3
 
    str = "12";
    a = stol(str, &pos, 16); //2 + 1*16
    cout << "a = " << a << endl; //a = 18
    cout << "pos = " << pos << endl; //pos = 2
 
    str = "  -12  35"; 
    a = stol(str, &pos, 16); //会舍弃空白符
    cout << "a = " << a << endl; //a = -18
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "  -ab";
    a = stol(str, &pos, 16); //-(11 + 10*16)
    cout << "a = " << a << endl; //a = -171
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "0123";
    a = stol(str, NULL, 16); //(3 + 2*16 + 1*16*16)
    cout << "a = " << a << endl; //a = 291
 
    return 0;
}
```

### stoll

```
long long stoll(const std::string& str, std::size_t* pos = 0, int base = 10);
long long stoll(const std::wstring& str, std::size_t* pos = 0, int base = 10);
```

功能：将字符串str转成 有符号 long long 整数

参数：

- str：字符串
- pos：存储将字符串str转成有符号整数，处理了str中字符的个数的地址，默认为NULL
- base：进制，10：十进制，8：八进制，16：十六进制，0：则自动检测数值进制，str是 0 开头为八进制，str是 0x 或 0X 开头是十六进制，默认为十进制

```c++
#include <iostream>
#include <string>
using namespace std;
 
int main(int argc, char *argv[])
{
    long long a;
    size_t pos = 0;
    string str;
 
    str = "-1235";
    a = stoll(str);
    cout << "a = " << a << endl; //a = -1235
 
    str = "1235";
    a = stoll(str);
    cout << "a = " << a << endl; //a = 1235
 
    str = "  -12  35"; 
    a = stoll(str, &pos); //会舍弃空白符
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "  -12ab35";
    a = stoll(str, &pos);
    cout << "a = " << a << endl; //a = -12
    cout << "pos = " << pos << endl; //pos = 5
 
    str = "0123";
    a = stoll(str);
    cout << "a = " << a << endl; //a = 123
 
    str = "0x123";
    a = stoll(str);
    cout << "a = " << a << endl; //a = 0
 
    return 0;
}
```

## to_string整数转string

```c++、
string to_string(numberic_value);
```

- string是返回类型，即函数返回一个字符串对象，其中包含字符串格式的数字值。
- numbric_value是可以为整数，浮点数，长整数，双精度数的数字。

```c++
#include <iostream>
#include <string>
using namespace std;

int main ()
{
	//定义不同类型的数据类型
	int intVal =12345;
	float floatVal = 123.45f;
	long longVal = 123456789;

	//将值转换为字符串以打印
	cout<<"intVal (string format) : "<<to_string (intVal) <<endl;
	cout<<"floatVal (string format) : "<<to_string (floatVal) <<endl;
	cout<<"floatVal (string format) : "<<to_string (longVal) <<endl;

	return 0;
}
// 输出结果
    intVal (string format) : 12345
    floatVal (string format) : 123.449997
    floatVal (string format) : 123456789
```



# vector

vector底层本质就是一个顺序表，它是一个**可变长**的数组，采用**连续存储**的空间来存储数据，它的元素类型也可以是任意的内置类型或者自定义类型。

因此不能有

```c++
int a[n];
```

但可以有

```c++
vector<int> a(n);
```

二维数组：https://blog.csdn.net/m0_57298796/article/details/123952640

```cpp
#include <vector>
```

## 初始化

```cpp
vector<int> a(10);  // 定义了10个整型元素的向量（尖括号中为元素类型名，它可以是任何合法的数据类型），但没有给出初值，其值是不确定的。

vector<int> vec{1, 2, 3, 4, 5, 6};  // vec中的内容为1,2,3,4,5,6

vector<int> a(10, 1);  // 定义了10个整型元素的向量,且给出每个元素的初值为1

vector<int> a(b);  // 用b向量来创建a向量，整体复制性赋值

vector<int> a(b.begin(), b.begin + 3);  // 定义了a值为b中第0个到第2个（共3个）元素

int b[7] = {1, 2, 3, 4, 5, 9, 8};
vector<int> a(b, b + 7);  // 从数组中获得初值

vector<int> list = {1,2,3,4,5,6,7}; 
vector<int> list {1,2,3,4,5,6,7};  // 直接赋值

// 二维数组
vector<vector<int>> a(row, vector<int>(column))；  
int row = a.size();          //获取行数
int column = a[0].size();    //获取列数

vector< vector<int> > a(row, vector<int>(column, 0));  // 初始值为0

```

## 操作

```cpp
a.assign(b.begin(), b.begin() + 3);  // b为向量，将b的0~2个元素构成的向量赋给a

a.assign(4, 2);  // 是a只含4个元素，且每个元素为2

a.back();  // 返回a的最后一个元素

a.front();  // 返回a的第一个元素

a[i];  // 返回a的第i个元素，当且仅当a[i]存在

a.clear();  // 清空a中的元素

a.empty();  // 判断a是否为空，空则返回ture,不空则返回false

a.pop_back();  // 删除a向量的最后一个元素

a.erase(a.begin() + 1, a.begin() + 3);  // 删除a中第1个（从第0个算起）到第2个元素，也就是说删除的元素从a.begin() + 1算起（包括它）一直到a.begin() + 3（不包括它）

a.push_back(5);  // 在a的最后一个向量后插入一个元素，其值为5

a.insert(a.begin() + 1, 5);  // 在a的第1个元素（从第0个算起）的位置插入数值5，如a为1,2,3,4，插入元素后为1,5,2,3,4

a.insert(a.begin() + 1, 3, 5);  // 在a的第1个元素（从第0个算起）的位置插入3个数，其值都为5

a.insert(a.begin() + 1, b + 3, b + 6);  // b为数组，在a的第1个元素（从第0个算起）的位置插入b的第3个元素到第5个元素（不包括b + 6），如b为1,2,3,4,5,9,8，插入元素后为1,4,5,9,2,3,4,5,9,8

a.size();  // 返回a中元素的个数；
 
a.capacity();  // 返回a在内存中总共可以容纳的元素个数

a.resize(10);  // 将a的现有元素个数调至10个，多则删，少则补，其值随机

a.resize(10,2);  // 将a的现有元素个数调至10个，多则删，少则补，其值为2

a.reserve(100);  // 将a的容量（capacity）扩充至100，也就是说现在测试a.capacity();的时候返回值是100.这种操作只有在需要给a添加大量数据的时候才显得有意义，因为这将避免内存多次容量扩充操作（当a的容量不足时电脑会自动扩容，当然这必然降低性能） 

a.swap(b);  // b为向量，将a中的元素和b中的元素进行整体性交换

a == b;  // b为向量，向量的比较操作还有!=,>=,<=,>,<
```

## 添加元素

```cpp
// 向向量a中添加元素
vector<int> a;
for (int i = 0; i < 10; i++)
	a.push_back(i);
```

```cpp
// 也可以从现有向量中选择元素向向量中添加
int a[6] = {1, 2, 3, 4, 5, 6};
vector<int> b;
vector<int> c(a, a + 4);
for (vector<int>::iterator it = c.begin(); it < c.end(); it++)
	b.push_back(*it);
```

```cpp
// 也可以从文件中读取元素向向量中添加
ifstream in("data.txt");
vector<int> a;
for (int i; in >> i)
    a.push_back(i);
```

```cpp
// 错误示范，下标只能用于获取已存在的元素，而现在的a[i]还是空的对象
vector<int> a;
for (int i = 0; i < 10; i++)
    a[i] = i;
```

## 读取元素

```cpp
// 通过下标方式读取
int a[6] = {1, 2, 3, 4, 5, 6};
vector<int> b(a, a + 4);
for (int i = 0; i <= b.size() - 1; i++)
    cout << b[i] << " ";
```

```cpp
// 通过遍历器方式读取
int a[6] = {1, 2, 3, 4, 5, 6};
vector<int> b(a, a + 4);
for (vector<int>::iterator it = b.begin(); it != b.end(); it++)
    cout << *it << " ";
```

## 配合其他算法

```cpp
#include <algorithm>
sort(a.begin(), a.end());  // 对a中的从a.begin()（包括它）到a.end()（不包括它）的元素进行从小到大排列

reverse(a.begin(), a.end());  // 对a中的从a.begin()（包括它）到a.end()（不包括它）的元素倒置，但不排列，如a中元素为1,3,2,4,倒置后为4,2,3,1

copy(a.begin(), a.end(), b.begin() + 1);  // 把a中的从a.begin()（包括它）到a.end()（不包括它）的元素复制到b中，从b.begin()+1的位置（包括它）开始复制，覆盖掉原有元素

find(a.begin(), a.end(), 10);  // 在a中的从a.begin()（包括它）到a.end()（不包括它）的元素中查找10，若存在返回其在向量中的位置
// 这里最后一个位置是const _Tp& _val，貌似不能是比如vector<int>这样的复杂元素
```

## size与capacity

**首先vector的底层实现也是普通数组**。

size就是我们平时用来遍历vector时候用的。

而capicity是vector底层数组（就是普通数组）的大小，capicity可不一定就是size。

当insert数据的时候，如果已经大于capicity，capicity会成倍扩容，但对外暴漏的size其实仅仅是+1。

那么既然vector底层实现是普通数组，怎么**扩容**的？

**就是重新申请一个二倍于原数组大小的数组，然后把数据都拷贝过去，并释放原数组内存。因此数组内存的起始地址已经改变了。**

https://blog.csdn.net/wkq0825/article/details/82255984

https://blog.csdn.net/230176366823/article/details/128876930 **这里面包含了迭代器失效的问题**

https://blog.csdn.net/2301_76366823/article/details/128876930

https://blog.csdn.net/m0_57298796/article/details/123952640

https://github.com/Lwyan127/leetcode-master/blob/master/problems/%E6%A0%B9%E6%8D%AE%E8%BA%AB%E9%AB%98%E9%87%8D%E5%BB%BA%E9%98%9F%E5%88%97%EF%BC%88vector%E5%8E%9F%E7%90%86%E8%AE%B2%E8%A7%A3%EF%BC%89.md

# 迭代器

**迭代器(iterator)**：一种访问string对象以及容器中的元素的**类型**，不必拘泥于迭代器概念本身，或它到底是容器定义的迭代器类型还是一种对象，只需要理解它的操作是用来访问string对象以及容器里的元素的。其**类似于指针**。

## 类型

```cpp
vector<int> vint;
vector<int>::iterator iit;    //读写类型
vector<int>::const_iterator icit;   //只读类型
vector<int>::reverse_iterator reit;    //反向迭代器
vector<int>::const_reverse_iterator reit;    //反向只读迭代器
```

## begin()和end()函数

- 有迭代器的类型同时拥有返回迭代器的成员函数：begin()和end()函数，它们的返回值就是迭代器类型iterator，所以我们可以使用begin()和end()函数来对容器和string对象的元素进行遍历。**begin()指向的是容器的第一个元素**，比如：vector就是下标为0的元素，**end()指向的是最后一个元素的下一个元素**，比如：vector里有10个元素，end()就是指向下标为10的元素，注意：这个元素的值是未知的，未定义的，只是用来表示遍历到了头而已。

  为什么begin()要指向开头第一个元素，而end()要指向末尾最后一个元素的下一个呢？

  因为：方便判断，左闭合范围

  ```
  begin()==end()：意味着容器里没有元素
  
  begin()+1==end()：意味着容器里只有一个元素
  
  begin()+1<end()：意味着容器里不止一个元素
  ```

- cbegin()函数和cend()函数是C++11的新标准，与begin()和end()的区别是，**cbegin()和cend()返回的是const_iterator类型**。就是用来，1.避免在循环体中做出了对元素修改的操作，2.当容器中所储对象是常量，则也只能使用这种类型。
- **rbegin()和rend()返回的是reverse_iterator类型**。同理还有crbegin()和crend()函数。**注意：当迭代器是反向迭代器时，对迭代器进行的运算符操作也是反向的**，如++操作，会返回指向容器的上一个元素。

## 运算符

| 运算符               | 解释                                                         |
| :------------------- | ------------------------------------------------------------ |
| iterator->item       | 解引用，获取该元素名为item的成员，等价于(*iterator).item     |
| *iterator            | 获取迭代器所指容器的元素                                     |
| ++iterator           | 下一个元素                                                   |
| --iterator           | 上一个元素，forward_list不支持此操作                         |
| iterator1==iterator2 | 判断两个迭代器是否在同一位置                                 |
| iterator1!=iterator2 | 判断两个迭代器是否在同一位置                                 |
| iterator+=n          | 迭代器向后移n个位置，n必须是整数                             |
| iterator-=n          | 迭代器向前移n个位置，n必须是整数                             |
| iterator1-iterator2  | 看iterator1比iterator2领先几个位置，注意，iterator1必须大于iterator2，且返回类型是difference_type |
| \>、>=、<、<=        | 关系运算，比较两个迭代器的前后位置大小                       |

## 插入迭代器

```
#include <iterator>
```

插入迭代器：这些迭代器被绑定在一个容器上，可用来向容器插入元素

三种插入迭代器：

    back_insert_iterator<_container>  (_container：如vector<string>)：使用back_inserter函数创建一个对容器使用push_back进行插入的迭代器
    
    front_insert_iterator<_container> ：使用front_inserter函数创建一个对容器使用push_front进行插入的迭代器
    
    insert_iterator<_container> ：使用inserter函数创建一个使用insert进行插入的迭代器，接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素会插入到给定迭代器所指向的元素之前，注意：它第二个参数的位置是固定的，不是说在内存里的存储位置固定，是说它会一直指向它所指向的元素。
    此迭代器类型为： insert_iterator<_container>  (_container：如vector<string>)

注意：只有支持相应操作的容器才能使用对应的迭代器！

```cpp
list<int> lis={1,2,3,4};
list<int> lis2;
 
auto it=back_inserter(lis2);
 
copy(lis.begin(),lis.end(),back_inserter(lis2));          //lis2：1,2,3,4
copy(lis.begin(),lis.end(),front_inserter(lis2));         //lis2：4,3,2,1
copy(lis.begin(),lis.end(),inserter(lis2,lis2.begin());   //lis2：1,2,3,4，因为lis2.begin()==lis2.end()，这里也可以看成指向end()，向end()前插入元素
```

## 流迭代器

## 反向迭代器

**rbegin**
语法：const reverse_iterator rbegin();
解释：rbegin()返回一个逆向迭代器，指向字符串的最后一个字符。

**rend**
语法：const reverse_iterator rend();
解释：rend()函数返回一个逆向迭代器，指向字符串的开头（第一个字符的前一个位置）。

```c++
#include<iostream>
#include<string>
using namespace std;
int main() {
    string str1,str2;
    cin >> str1;
    //定义一个正向迭代器
    string::iterator ptr1 = str1.begin();
    //正向输出字符串
    while (ptr1 != str1.end())
        cout << *(ptr1++) << " ";
    cout << endl;

    cin >> str2;
    //定义一个逆向迭代器
    string::reverse_iterator ptr2 = str2.rbegin();
    //逆向输出字符串
    while (ptr2 != str2.rend())
    //注意逆向迭代器移动方向相反，所以从尾部仍然通过++来移动
        cout << *(ptr2++) << " ";
    cout << endl;

}

// sorts vec in "normal" order
sort(vec.begin(), vec.end());
// sorts in reverse: puts smallest element at the end of vec
sort(vec.rbegin(), vec.rend());
```

原文链接：https://blog.csdn.net/qq_36134437/article/details/103357850

**注意：调用泛型算法的时候，如果算法里的迭代器是反向迭代器，那么其返回的迭代器也将是反向迭代器，这意味着后面如果有对这个迭代器的操作，那么将都会按着反向迭代器的操作来进行，这可能会导致预料之外的结果，比如：**

```cpp
序列vec：a b c d , e f g
auto iterator=find(vec.rbegin(),vec.rend(),',');        //反向查找第一个出现‘,’的位置
cout<<string(vec.rbegin(),iterator)<<endl;        //输出：gfe，error,因为我的预期结果是efg
cout<<string(iterator,vec.end())<<endl;        //error！因为此时iterator迭代器是反向迭代器，
//对这个迭代器进行++操作时，是向vec.begin()靠拢的！
cout<<string(iterator.base(),vec.end())<<endl;    //true！base()函数可以将反向迭代器转换为普通迭代器，指向e字符
```

**使用base()函数可以将反向迭代器转换为普通迭代器，但是其位置会有变化，base后的迭代器指向反向迭代器的所指元素的后一个元素（正序后一个）**。

https://blog.csdn.net/A_ns_wer_/article/details/126140643

https://blog.csdn.net/Qiuhan_909/article/details/129939184



# algorithm库

在 C++ 中，Algorithm 是一个标准库，它提供了许多通用的算法函数，可以用于对容器（如数组、vector、list 等）中的元素进行操作和处理。Algorithm 库包含了许多常用的算法，如排序、查找、合并和删除等。

## sort

sort() 函数可以对容器中的元素进行排序，使用快速排序算法或堆排序算法。

```c++
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
sort(v.begin(), v.end());  // 对 v 中的元素进行排序

// 升序排序
int a[10] = {9, 6, 3, 8, 5, 2, 7, 4, 1, 0};
sort(a, a + 10);  // 10为元素个数

// 降序排序
static bool cmp(int num1, int num2) {  // 需要静态函数给sort用
    return num1 > num2;  // 可以简单理解为 >： 降序排列;  < ： 升序排列
}

int a[10] = {9, 6, 3, 8, 5, 2, 7, 4, 1, 0};
sort(a, a + 10, cmp);  // 使用自定义排序函数
```

参数解释： 第一个参数是数组的首地址，一般写上数组名就可以，因为数组名是一个指针常量。第二个参数相对较好理解，即首地址加上数组的长度n（代表尾地址的下一地址）。最后一个参数是比较函数的名称（自定义函数cmp），这个比较函数可以不写，即第三个参数可以缺省，这样sort会默认按数组**升序排序**。

时间复杂度：**n\*log(n)**

实现原理：sort并不是简单的**快速排序**，它对普通的快速排序进行了优化，此外，它还结合了**插入排序**和**推排序**。系统会根据你的数据形式和数据量自动选择合适的排序方法，这并不是说它每次排序只选择一种方法，它是在一次完整排序中不同的情况选用不同方法，比如给一个数据量较大的数组排序，开始采用快速排序，分段递归，分段之后每一段的数据量达到一个较小值后它就不继续往下递归，而是选择插入排序，如果递归的太深，他会选择推排序。

简单例子：对数组A的0~n-1元素进行升序排序，只要写sort(A,A+n)即可；对于向量V也一样，sort(v.begin(),v.end())即可。

https://blog.csdn.net/VariatioZbw/article/details/125155432

https://www.cnblogs.com/alvinzh/p/6784862.html

## find

find() 函数可以在容器中查找指定元素的位置，返回该元素在容器中的迭代器。

```c++
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
auto it = find(v.begin(), v.end(), 5);  // 查找 5 在 v 中的位置
```

## reverse

reverse() 函数可以将容器中的元素反转，使第一个元素变为最后一个，最后一个元素变为第一个。

```c++
vector<int> v = {1, 2, 3, 4, 5};
reverse(v.begin(), v.end());  // 反转 v 中的元素
```

## copy

copy() 函数可以将容器中的元素复制到另一个容器中。

```c++
vector<int> v1 = {1, 2, 3, 4, 5};
vector<int> v2(5);  // 创建一个大小为 5 的 vector
copy(v1.begin(), v1.end(), v2.begin());  // 将 v1 中的元素复制到 v2 中
```

## accumulate

accumulate() 函数可以对容器中的元素进行累加操作，返回累加结果。

```c++
vector<int> v = {1, 2, 3, 4, 5};
int sum = accumulate(v.begin(), v.end(), 0);  // 对 v 中的元素求和
```

## count

count() 函数可以统计容器中指定元素的个数。

```cpp
vector<int> v = {1, 2, 3, 2, 5};
int num = count(v.begin(), v.end(), 2);  // 统计 v 中 2 的个数
```

## min和max

min() 和 max() 函数可以返回两个值中的最小值和最大值。

```cpp
int a = 1, b = 2;
int c = min(a, b);  // c = 1
int d = max(a, b);  // d = 2
```

还可以返回多个值中的最大和最小值。

```c++
cout << min({3, 2, 3, 4, 5});
cout << max({3, 2, 3, 4, 5});
```

## replace

replace() 函数可以将容器中指定值的元素替换为新值。

```cpp
vector<int> v = {1, 2, 3, 2, 5};
replace(v.begin(), v.end(), 2, 4);  // 将 v 中所有的 2 替换为 4
```

## unique

unique() 函数可以将容器中相邻的重复元素去重，返回去重后的容器尾部迭代器。

```cpp
vector<int> v = {1, 2, 2, 3, 3, 3, 4, 4, 5};
auto it = unique(v.begin(), v.end());  // 去重
v.erase(it, v.end());  // 删除重复的元素
```

## lower_bound和upper_bound

lower_bound() 和 upper_bound() 函数可以在有序容器中查找指定元素的位置，返回该元素在容器中的迭代器。lower_bound() 返回第一个大于等于指定元素的位置，而 upper_bound() 返回第一个大于指定元素的位置。

```cpp
vector<int> v = {1, 2, 3, 4, 5};
auto it1 = lower_bound(v.begin(), v.end(), 3);  // 查找 3 在 v 中的位置
auto it2 = upper_bound(v.begin(), v.end(), 3);  // 查找第一个大于 3 的元素在 v 中的位置
```

## next_permutation和prev_permutation

next_permutation() 和 prev_permutation() 函数可以获取容器中下一个或上一个排列。它们会将容器中的元素重新排列，返回 true 表示获取成功，false 表示已经是最后一个或第一个排列。

```cpp
vector<int> v = {1, 2, 3};
do {
    // 处理排列
} while (next_permutation(v.begin(), v.end()));
```

## for_each

for_each() 函数可以遍历容器中的所有元素，并对其进行指定操作。例如：

```cpp
vector<int> v = {1, 2, 3, 4, 5};
for_each(v.begin(), v.end(), [](int& n){ n *= 2; });  // 将 v 中的所有元素乘以 2
```

## merge

merge() 函数可以将两个已排序的容器合并为一个已排序的容器，返回合并后的容器尾部迭代器。

```cpp
vector<int> v1 = {1, 3, 5};
vector<int> v2 = {2, 4, 6};
vector<int> v3(v1.size() + v2.size());  // 创建一个足够大的 vector
merge(v1.begin(), v1.end(), v2.begin(), v2.end(), v3.begin());  // 合并 v1 和 v2 到 v3
```

## nth_element

nth_element() 函数可以对容器中的元素进行部分排序，将第 n 小的元素放在第 n 个位置，前 n-1 个元素都小于等于第 n 个元素，后面的元素都大于等于第 n 个元素。

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
nth_element(v.begin(), v.begin() + 5, v.end());  // 将 v 中第 5 小的元素放在第 5 个位置
```

## partial_sort

partial_sort() 函数可以对容器中的元素进行部分排序，将前 n 个最小的元素放在容器的前 n 个位置，其余元素的顺序不保证。

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
partial_sort(v.begin(), v.begin() + 3, v.end());  // 将 v 中前 3 个最小的元素放在前 3 个位置
```

## partition

partition() 函数可以将容器中的元素分为两部分，使满足指定条件的元素在前面，不满足条件的元素在后面。

```cpp
vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3};
auto it = partition(v.begin(), v.end(), [](int n){ return n % 2 == 0; });  // 将 v 中的偶数放在前面，奇数放在后面
```

## binary_search

binary_search() 函数可以在有序容器中二分查找指定元素，返回 true 表示找到，false 表示未找到。

```cpp
vector<int> v = {1, 2, 3, 4, 5};
bool found = binary_search(v.begin(), v.end(), 3);  // 在 v 中查找 3 是否存在
```

https://blog.csdn.net/Uperrr/article/details/130300450



# max_element/min_element

今天做题时遇到了官方题解中用到的min_element函数，发现这个函数很方便的用于求vector容器中的最小元素。

**max_element（）与min_element（）**分别用来求最大元素和最小元素的**位置**。

接收参数：容器的首尾地址（迭代器）（可以是一个区间）

返回：最值元素的**地址**（迭代器），需要减去序列头以转换为下标

```cpp
vector<int> n;
int maxPosition = max_element(n.begin(),n.end()) - n.begin(); //最大值下标

int minPosition = min_element(n.begin(),n.end()) - n.begin();//最小值下标

// 普通数组
int a[]={1,2,3,4};
int maxPosition = max_element(a,a+2) - a; //最大值下标

int minPosition = min_element(a,a+2) - a;//最小值下标
```

***max_element（）与\*min_element（）**分别用来求最大元素和最小元素的值。

接收参数：容器的首尾地址（迭代器）（可以是一个区间）

返回：最值元素的值

```cpp
int maxValue = *max_element(n.begin(),n.end()); //最大值

int minValue = *min_element(n.begin(),n.end());//最小值


int maxValue = *max_element(a,a+2); //最大值

int minValue = *min_element(a,a+2);//最小值
```

https://zhuanlan.zhihu.com/p/435905136



# unordered_set

## **使用前提：**

```c++
#include <unordered_set>
```

## **特性**

1. 不再以键值对的形式存储数据，而是直接存储数据的值 ；
2. 容器内部存储的各个元素的值都互不相等，且不能被修改；
3. 不会对内部存储的数据进行排序

## **初始化**

```c++
// 创建空的set
unordered_set<int> set1;

// 拷贝构造
unordered_set<int> set2(set1);

// 使用迭代器构造
unordered_set<int> set3(set1.begin(), set1.end());  // 这样可以借用unordered_set不可重复的性质对set1去重

// 使用数组作为其初值进行构造
unordered_set<int> set4(arr,arr+5);

// 移动构造
unordered_set<int> set5(move(set2));

// 使用处置列表进行构造
unordered_set<int> set6 {1,2,10,10};
```

## **内置函数**

```c++
// empty()函数——判断是否为空
// 若容器为空，则返回 true；否则 false
set1.empty();

// find()函数——查找
// 查找2，找到返回迭代器，失败返回end()
set1.find(2);

// count()函数——出现次数
// 返回指2出现的次数，0或1
set1.count(2);

// insert()函数——插入元素
// 插入元素，返回pair<unordered_set<int>::iterator, bool>
set1.insert(3);
// 使用initializer_list插入元素
set1.insert({1,2,3});
// 指定插入位置，如果位置正确会减少插入时间，返回指向插入元素的迭代器
set1.insert(set1.end(), 4);
// 使用范围迭代器插入
set1.insert(set2.begin(), set2.end());

/*
关于insert函数的返回值：
insert()只传入单个参数（待插入元素）
1 会返回一个 pair 对象
2 这个 pair 对象包含一个迭代器，以及一个附加的布尔值用来说明插入是否成功
3 如果元素被插入，返回的迭代器会指向新元素
4 如果没有被插入，迭代器指向阻止插入的元素
*/
auto pr = words.insert("ninety"); // Returns a pair - an iterator & a bool value

/*
insert()传入两个参数（迭代器+待插入元素）
1 可以用一个迭代器作为insert()的第一个参数，它指定了元素被插入的位置
2 在这种情况下，只会返回一个迭代器
*/
auto iter = words.insert (pr.first, "nine"); // 1st arg is a hint. Returns an iterator

/*
insert()传入初始化列表
1 插入初始化表中的元素
2 在这种情况下，什么都没有返回
*/
words.insert({"ten", "seven", "six"});  // Inserting an initializer list

// emplace()函数——插入元素(转移构造)
// 使用转移构造函数添加新元素3，比insert效率高
set1.emplace(3);

// erase()函数——删除元素
// 删除操作，成功返回1，失败返回0
set1.erase(1);
// 删除操作，成功返回下一个pair的迭代器
set1.erase(set1.find(1));
// 删除set1的所有元素，返回指向end的迭代器
set1.erase(set1.begin(), set1.end());

// bucket_count()函数——篮子数目
// 返回容器中的篮子总数
set1.bucket_count();

// bucket_size()函数——篮子中元素数目
// 返回1号篮子中的元素数
set1.bucket_size(1);

// bucket()函数——在哪个篮子
// 元素1在哪一个篮子
set1.bucket(1);

// clear()函数——清空
set1.clear();

// load_factor()函数——负载因子
// 负载因子，返回每个篮子元素的平均数，即size/float(bucket_count);
set1.load_factor();

// rehash()函数——设置篮子数目并重新分布
// 设置篮子的数量为20，并且重新rehash
set1.rehash(20);
```

https://blog.csdn.net/qq_40286920/article/details/124731777



# map

- map是STL的一个关联容器，以**键值对**存储的数据，其类型可以自己定义，每个**关键字不可重复**，**关键字不能修改**，值可以修改
- map同set、multiset、multimap（与map的差别仅在于multimap允许一个键对应多个值，multimap中key可以重复）内部数据结构都是**红黑树**，而java中的hashmap是以hash table实现的
- 所以map**内部有序**（**自动排序**，自动按key升序排序（从小到大）的，单词时按照字母序排序）
- 查找时间复杂度为**O(logn)**。
- 增删时间复杂度为**O(logn)**。
- value值默认：
  - char的默认值为0x00
  - string的默认值为""
  - 整数默认为0


```c++
#include<map>
```

## unordered_map

- **哈希表**实现，因此查询效率和删改效率为**O(n)**
- **内部无序**
- **关键字不可重复，不可修改**
- 使用函数类似于map

```
#include <unordered_map>
```

## 定义

```c++
map<string,int> my_map;
也可以使用
typedef map<string,int> My_Map;
My_Map my_map;
```

## 基本方法

| **函数名**                        | **功能**                            |
| --------------------------------- | ----------------------------------- |
| my_map.insert()或按照数组直接赋值 | 插入                                |
| my_map.find()                     | 查找一个元素                        |
| my_map.clear()                    | 清空                                |
| my_map.erase()                    | 删除一个元素                        |
| my_map.size()                     | map的长度大小                       |
| my_map.begin()                    | 返回指向map头部的迭代器             |
| my_map.end()                      | 返回指向map末尾的迭代器             |
| my_map.rbegin()                   | 返回一个指向map尾部的逆向迭代器     |
| my_map.rend()                     | 返回一个指向map头部的逆向迭代器     |
| my_map.empty()                    | map为空时返回true                   |
| swap()                            | 交换两个map,两个map中所有元素都交换 |

## 插入数据

```c++
// 第一种：用insert函数插入pair数据：
map<int,string> my_map;
my_map.insert(pair<int,string>(1,"first"));
my_map.insert(pair<int,string>(2,"second"));
```

```c++
// 第二种：用insert函数插入value_type数据：
map<int,string> my_map;
my_map.insert(map<int,string>::value_type(1,"first"));
my_map.insert(map<int,string>::value_type(2,"second"));
 
map<int,string>::iterator it;           //迭代器遍历
for(it=my_map.begin();it!=my_map.end();it++)
    cout<<it->first<<it->second<<endl;
```

```c++
// 第三种：用数组的方式直接赋值：
map<int,string> my_map;
my_map[1]="first";
my_map[2]="second";
 
map<int,string>::iterator it;
for(it=my_map.begin();it!=my_map.end();it++)
    cout<<it->first<<it->second<<endl;
```

以上三种用法，虽然都可以实现数据的插入，但是它们是有区别的，当然了第一种和第二种在效果上是完成一样的，用insert函数插入数据，在数据 插入上涉及到集合的唯一性这个概念，即当map中有这个关键字时，insert操作是插入数据不了的，但是用数组方式就不同了，它可以覆盖以前该关键字对应的值

## 查找元素

第一种：用count函数来判断关键字是否出现，其缺点是无法定位元素出现的位置。由于map一对一的映射关系，count函数的返回值要么是0，要么是1。

```c++
map<string,int> my_map;
my_map["first"]=1;
cout<<my_map.count("first")<<endl;    //输出1；
```

第二种：用**find函数**来定位**关键字**出现的位置，它**返回一个迭代器**，当数据出现时，返回的是数据所在位置的迭代器；若map中没有要查找的数据，返回的迭代器等于end函数返回的迭代器。

```c++
#include <map>  
#include <string>  
#include <iostream>  
 
using namespace std;  
  
int main()  
{  
    map<int, string> my_map;  
    my_map.insert(pair<int, string>(1, "student_one"));  
    my_map.insert(pair<int, string>(2, "student_two"));  
    my_map.insert(pair<int, string>(3, "student_three"));  
    map<int, string>::iterator it;  
    it = my_map.find(1);  
    if(it != my_map.end())  
       cout<<"Find, the value is "<<it->second<<endl;      
    else  
       cout<<"Do not Find"<<endl;  
    return 0;  
}
//通过map对象的方法获取的iterator数据类型是一个std::pair对象，包括两个数据iterator->first和iterator->second，分别代表关键字和value值。
```

## 删除元素

```c++
#include <map>  
#include <string>  
#include <iostream>  
  
using namespace std;  
  
int main()  
{  
    map<int, string> my_map;  
    my_map.insert(pair<int, string>(1, "one"));  
    my_map.insert(pair<int, string>(2, "two"));  
    my_map.insert(pair<int, string>(3, "three"));  
    //如果你要演示输出效果，请选择以下的一种，你看到的效果会比较好
    //如果要删除1,用迭代器删除
    map<int, string>::iterator it;  
    it = my_map.find(1);  
    my_map.erase(it);                   //如果要删除1，用关键字删除
    int n = my_map.erase(1);            //如果删除了会返回1，否则返回0
    //用迭代器，成片的删除
    //一下代码把整个map清空
    my_map.erase( my_map.begin(), my_map.end() );  
    //成片删除要注意的是，也是STL的特性，删除区间是一个前闭后开的集合
    //自个加上遍历代码，打印输出吧
    return 0;
}  
```

## 排序，按value排序

map中元素是自动按key升序排序（从小到大）的；按照value排序时，想直接使用sort函数是做不到的，sort函数只支持数组、vector、list、queue等的排序，无法对map排序，那么就需要**把map放在vector中，再对vector进行排序**。

```c++
#include <iostream>
#include <string>
#include <map>
#include <algorithm>
#include <vector>
using namespace std;
 
bool cmp(pair<string,int> a, pair<string,int> b) {
	return a.second < b.second;
}
 
int main()
{
    map<string, int> ma;
    ma["Alice"] = 86;
    ma["Bob"] = 78;
    ma["Zip"] = 92;
    ma["Stdevn"] = 88;
    vector< pair<string,int> > vec(ma.begin(),ma.end());
    //或者：
    //vector< pair<string,int> > vec;
    //for(map<string,int>::iterator it = ma.begin(); it != ma.end(); it++)
    //    vec.push_back( pair<string,int>(it->first,it->second) );
 
    sort(vec.begin(),vec.end(),cmp);
    for (vector< pair<string,int> >::iterator it = vec.begin(); it != vec.end(); ++it)
    {
        cout << it->first << " " << it->second << endl;
    }
    return 0;
}
```

https://blog.csdn.net/weixin_41501074/article/details/114532738

https://www.cnblogs.com/langyao/p/8823092.html



# pair

pair 是一个很实用的"小玩意"，当想要将两个元素绑在一起作为一个合成元素、又不想要因此定义结构体时，使用 pair 可以很方便地作为一个代替品。

也就是说，pair 实际上可以看作一个内部有两个元素的结构体，且这两个元素的类型是可以指定的，如下面的短代码所示：

```text
struct pair {
    typeName1 first;
    typeName2 second;
};
```

**1、pair 的定义**

要使用 pair，应先添加头文件 `#include <utility>`，并在头文件下面加上 `using namespace std;` ，然后就可以使用了。

注意：由于 map 的内部实现中涉及 pair，因此添加 map 头文件时会自动添加 utility 头文件，此时如果需要使用 pair，就不需要额外再去添加 utility 头文件了。因此，记不住 utility 拼写的，可以偷懒地用 map 头文件来代替 utility 头文件。

pair 有两个参数，分别对应 first 和 second 的数据类型，它们可以是任意基本数据类型或容器。

```
pair<typeName1,typeName2> name;
```

因此，想要定义参数为 string 和 int 类型的 pair，就可以使用如下写法：

```
pair<string,int> p;
```

如果想在定义 pair 时进行初始化，只需要跟上一个小括号，里面填写两个想要初始化的元素即可：

```
pair<string,int> p("haha",5);
```

而如果想要在代码中临时构建一个 pair，有如下两种方法：

① 将类型定义写在前面，后面用小括号内两个元素的方式。

```
pair<string,int>("haha",5)
```

② 使用自带的 make_pair 函数。

```
make_pair("haha",5)
```

**2、 pair 中元素的访问**

pair 中只有两个元素，分别是 first 和 second，只需要按正常结构体的方式去访问即可。

示例如下：

```text
#include <iostream>
#include <utility>
#include <string>
using namespace std;
int main() {
    pair<string,int> p;
    p.first="haha";
    p.second=5;
    cout<<p.first<<" "<<p.second<<endl;
    p=make_pair("xixi",55);
    cout<<p.first<<" "<<p.second<<endl;
    p=pair<string,int>("heihei",555);
    cout<<p.first<<" "<< p.second<<endl;
    return 0;
}
```

输出结果：

```text
haha 5 
xixi 55 
heihei 555
```

**3、pair 常用函数实例解析**

比较操作数

两个pair 类型数据可以直接使用 =、!=、<、<=、>、>= 比较大小，比较规则是先以 first 的大小作为标准，只有当 first 相等时才去判别 second 的大小。

示例如下：

```text
#include <cstdio>
#include <utility>
using namespace std;
int main() {
    pair<int,int> p1(5,10);
    pair<int,int> p2(5,15);
    pair<int,int> p3(10,5);
    if(p1<p3)	printf("p1<p3\n");
    if(p1<=p3) 	printf("p1<=p3\n");
    if(p1<p2)	printf("p1<p2\n");
    return 0;
}
```

输出结果：

```text
p1<p3
p1<=p3 
p1<p2 
```

**4、pair的常见用途**

关于 pair 有两个比较常见的例子：

① 用来代替二元结构体及其构造函数，可以节省编码时间。

② 作为 map 的键值对来进行插入，例如下面的例子。

```text
#include<iostream>
#include<string>
#include <map>
using namespace std;
int main() {
    map<string,int> mp;
    mp.insert(make pair("heihei",5));
    mp.insert(pair<string,int>("haha",10));
    for(map<string,int>::iterator it=mp.begin();it!= mp.end();it++) {
        cout << it->first <<""<< it->second << endl;
    }
    return 0;
}
```

输出结果：

```text
haha 10 
heihei 5
```

https://zhuanlan.zhihu.com/p/482540092



# STL

## stack

**1.包含头文件及初始化**

```c++
#include <stack>
//声明
stack<int> stk1;
stack<string> stk2;
stack<ListNode*> stk3;
```

**2.基本操作**

| 代码        | 含义                     |
| ----------- | ------------------------ |
| stk.push()  | 压栈，增加元素           |
| stk.pop()   | 移除栈顶元素             |
| stk.top()   | 取得栈顶元素（但不删除） |
| stk.empty() | 检测栈内是否为空，空为真 |
| stk.size()  | 返回stack内元素的个数    |

**3.举例**

```c++
myStack.push(10); // 入栈
myStack.pop(); // 出栈
if (!myStack.empty()) {
    int topElement = myStack.top(); // 访问栈顶元素
}
if (myStack.empty()) {
    // 栈为空
}
int stackSize = myStack.size(); // 获取栈的大小
```

## queue

**1.包含头文件及初始化**

```c++
#include<queue>

//queue的定义 
std::queue<int> q1; 		//定义一个储存数据类型为int的queue容器q1 
std::queue<double> q2; 	//定义一个储存数据类型为double的queue容器q2
std::queue<string> q3; 	//定义一个储存数据类型为string的queue容器q3
std::queue<结构体类型> q4; //定义一个储存数据类型为结构体类型的queue容器q4
```

**2.基本操作**

| 代码      | 定义                                       |
| --------- | ------------------------------------------ |
| q.front() | 返回队首元素                               |
| q.push()  | 尾部添加一个元素副本 进队                  |
| q.pop()   | 删除第一个元素 出队                        |
| q.size()  | 返回队列中元素个数，返回值类型unsigned int |
| q.empty() | 判断是否为空，队列为空，返回true           |
| q.back()  | 返回队尾元素                               |

**3.举例**

```c++
#include<iostream>
#include<queue>
using namespace std;
int main() {
	queue<int> q; //定义一个数据类型为int的queue 
	q.push(1); //向队列中加入元素1 
	q.push(2); //向队列中加入元素2
	q.push(3); //向队列中加入元素3 
	q.push(4); //向队列中加入元素4 
	cout<<"将元素1、2、3、4一一加入队列中后，队列中现在的元素为：1、2、3、4"<<endl;
	cout<<"队列中的元素个数为："<<q.size()<<endl;
	//判断队列是否为空 
	if (q.empty()) {
		cout<<"队列为空"<<endl;
	}
	else {
		cout<<"队列不为空"<<endl;
	}
	cout<<"队列的队首元素为："<<q.front()<<endl;
	//队列中的队首元素出队 
	q.pop();
	cout<<"将队列队首元素出队后，现在队列中的元素为2、3、4"<<endl;		
}

/* 运行结果：
将元素1、2、3、4一一加入队列中后，队列中现在的元素为：1、2、3、4
队列中的元素个数为：4
队列不为空
队列的队首元素为：1
将队列队首元素出队后，现在队列中的元素为2、3、4
*/
```

**队列中的数据和堆栈一样是不允许随机访问的，即不能通过下标访问，且队列内的元素也是无法遍历的。**

> 我们可以通过while循环的方法将queue中的元素读取一遍，但是这种方法非常局限，因为我们每读取一个元素就需要将这个元素出队，因此该方法只能读取一遍queue中的元素。

## dequeue双端队列

- deque具有以下特点：
  - 双向开口：deque可以在两端进行高效的插入和删除操作，即在队首和队尾都可以进行操作。
  - 动态扩展：deque的内部实现使用了分段连续线性空间，可以动态扩展以适应元素的增加。
  - 随机访问：deque支持随机访问，可以通过下标访问元素。

**1.包含头文件及初始化**

1）头文件

```c++
#include <deque>
```

2）可以使用以下方式声明和初始化一个deque对象：

```c++
std::deque<int> myDeque; // 声明一个空的整数双端队列
```

3）也可以使用已有的容器初始化双端队列对象：

```c++
std::vector<int> myVector = {1, 2, 3, 4, 5};
std::deque<int> myDeque(myVector.begin(), myVector.end()); // 使用vector初始化双端队列
```

**2.基本操作**

| 代码                                | 含义                                |
| ----------------------------------- | ----------------------------------- |
| push_back(x)/push_front(x)          | 把x压入后/前端                      |
| back()/front()                      | 访问(不删除)后/前端元素             |
| pop_back() pop_front()              | 删除后/前端元素                     |
| erase(iterator it)                  | 删除双端队列中的某一个元素          |
| erase(iterator first,iterator last) | 删除双端队列中[first,last）中的元素 |
| empty()                             | 判断deque是否空                     |
| size()                              | 返回deque的元素数量                 |
| clear()                             | 清空deque                           |

**3.举例**

```c++
myDeque.push_back(10); // 在队尾插入元素
myDeque.push_front(20); // 在队头插入元素
myDeque.pop_back(); // 删除队尾元素
myDeque.pop_front(); // 删除队头元素
if (!myDeque.empty()) {
    int frontElement = myDeque.front(); // 访问队头元素
    int backElement = myDeque.back(); // 访问队尾元素
}
if (myDeque.empty()) {
    // 双端队列为空
}
int dequeSize = myDeque.size(); // 获取双端队列的大小
int element = myDeque[2]; // 访问下标为2的元素
```

关于erase()：

```c++
deque<int> myDeque = {1, 2, 3, 4, 5};
deque<int>::iterator it = myDeque.begin() + 2; // 指向第三个元素的迭代器
myDeque.erase(it); // 删除第三个元素

deque<int> myDeque = {1, 2, 3, 4, 5};
deque<int>::iterator first = myDeque.begin() + 1; // 指向第二个元素的迭代器
deque<int>::iterator last = myDeque.begin() + 4; // 指向第五个元素的迭代器
myDeque.erase(first, last); // 删除第二个到第四个元素
```

## priority_queue优先队列

- 优先队列（priority_queue）是一个非常有用的容器适配器。它提供了一种特殊的队列，其中的元素按照一定的优先级进行排序和访问。

**1.包含头文件及初始化**

1）头文件

```c++
#include <queue>
```

2）可以使用以下方式声明和初始化一个优先队列对象：

```c++
priority_queue<int> myQueue; // 声明一个空的整数优先队列
```

3）也可以使用已有的容器初始化优先队列对象：

```c++
vector<int> myVector = {1, 2, 3, 4, 5};
priority_queue<int> myQueue(myVector.begin(), myVector.end()); // 使用vector初始化优先队列
```

**2.基本操作**

| 代码                                                  | 含义                 |
| ----------------------------------------------------- | -------------------- |
| push()                                                | 入队                 |
| pop()                                                 | 堆顶（队首）元素出队 |
| size()                                                | 队列元素个数         |
| empty()                                               | 是否为空             |
| 注意没有clear()！                                     |                      |
| 优先队列只能通过top()访问队首元素（优先级最高的元素） |                      |

**3.设置优先级**

```c++
priority_queue<int, vector<int>, less<int> >pq;
//最后两个>之间要有空格
```

> 解释：
> int：表示队列中元素的类型，这里是整数类型。
> vector：表示底层容器的类型，这里使用vector作为底层容器。
> less：表示元素的比较函数对象，用于定义元素的优先级。
> less:是一个函数对象，用于按照升序对元素进行比较。

```c++
priority_queue<int,vector<int>, greater<int>  > pq;
//此为降序
```

**自定义排序：**

```c++
struct MyComparator {
    bool operator()(int a, int b) {
        // 自定义排序规则，按照元素的绝对值进行降序排序
        return abs(a) < abs(b);
    }
};
int main() {
    priority_queue<int, vector<int>, MyComparator> pq;

    pq.push(10);
    pq.push(-5);
    pq.push(8);
    pq.push(-3);
    while (!pq.empty()) {
        cout << pq.top() << " ";
        pq.pop();
    }
    return 0;
}
```

**结构体(或自定义)优先级设置**

```c++
//要排序的结构体（存储在优先队列里面的）
struct node {
	int x,y;
};

//定义的比较结构体
//注意：cmp是个结构体 
struct cmp {//自定义堆的排序规则 
	bool operator()(const node& a,const node& b) {
		return a.x > b.x;  //从堆底到堆顶 降序排序 即小顶堆 
	}  //如果要升序改变不等号方向就好
};
//初始化定义
priority_queue<node, vector<node>, cmp> pq; 
```

```c++
struct node {
	int x,y;
	friend bool operator<(node a,node b) {  //为两个结构体参数，结构体调用一定要写上friend
		return a.x > b.x;  //按x从小到大排 
	}
};
```

**存储特殊类型的优先级**

- 存储pair类型
  规则：默认先对pair的first进行降序排序，然后再对second降序排序
  对first先排序，大的排在前面，如果first元素相同，再对second元素排序，保持大的在前面。

```c++
int main() {
    priority_queue<pair<int,int> >q;
	q.push({1,2});
	q.push({1,3});
	q.push(make_pair(2,3));
    while(!q.empty())
    {
        cout<<q.top().first<<" "<<q.top().second<<endl;
        q.pop();
    }
    return 0;
}
```

> 结果
> 2 3
> 1 3
> 1 2

## list双向链表

| 构造函数 (constructor)                                   | 接口说明                            |
| -------------------------------------------------------- | ----------------------------------- |
| list (size_type n, const value_type& val = value_type()) | 构造的list中包含n个值为val的元素    |
| list()                                                   | 构造空的list                        |
| list (const list& x)                                     | 拷贝构造函数                        |
| list (InputIterator first, InputIterator last)           | 用[first, last)区间中的元素构造list |

| **函数声明**   | 接口说明                                    |
| -------------- | ------------------------------------------- |
| **empty**      | 检测list是否为空，是返回true，否则返回false |
| **size**       | 返回list中有效节点的个数                    |
| **front**      | 返回list的第一个节点中值的引用              |
| **back**       | 返回list的最后一个节点中值的引用            |
| **push_front** | 在list首元素前插入值为val的元素             |
| **pop_front**  | 删除list中第一个元素                        |
| **push_back**  | 在list尾部插入值为val的元素                 |
| **pop_back**   | 删除list中最后一个元素                      |
| **insert**     | 在list position位置中插入值为val的元素      |
| **erase**      | 删除list position位置的元素                 |
| **swap**       | 交换两个list中的元素                        |
| **clear**      | 清空list中的有效元素                        |



# 关于auto使用

```c++
for (auto str: strs) 
for (auto it = m.begin(); it != m.end(); it++) 
```

https://www.cnblogs.com/KunLunSu/p/7861330.html

# 范围基 for 循环

`for (声明 : 容器)` 这种结构是 C++11 引入的 **范围基 for 循环**（Range-based for loop），它是用来遍历容器（如数组、`vector`、`list` 等）中每个元素的简洁方式，相比传统的基于索引的 `for` 循环，它的代码更加简洁易读，也避免了手动管理迭代器或索引的复杂性。

```c++
for (声明 : 容器) {
    // 循环体
}
```

这里，`声明` 表示你想要使用的变量类型，它会用来保存容器中每一个元素的值，`容器` 则是你想要遍历的容器。

### 1. **声明部分（声明）**

`声明` 部分指定了遍历过程中用于接收容器元素的变量类型。通常这可以是：

- **值**：例如 `int`、`string`，表示容器中的元素是复制到这个变量中进行处理（修改这个变量不会影响容器中的原始数据）。
- **引用**：例如 `int&`、`string&`，表示容器元素的引用，可以直接修改容器中的元素。如果不打算修改容器中的元素，通常使用 `const` 引用。
- **常量引用**：例如 `const string&`，这通常用于避免复制开销，并且确保我们不会意外地修改容器中的元素。

### 2. **容器部分（容器）**

`容器` 是你想遍历的对象，可以是数组、`vector`、`list`、`set`、`map` 等任何支持迭代的容器。

### 3. **循环体（循环体）**

`循环体` 是你希望对容器中的每个元素执行的操作。每次循环时，`声明` 中的变量会接收容器中的一个元素，然后你可以在循环体中对它进行操作。

# . 和 ->

```c++
struct Point {
    int x, y;
};

int main() {
    Point p{3, 4};
    p.x = 10;  // 直接访问成员
    return 0;
}
```

```c++
struct Point {
    int x, y;
};

int main() {
    Point p{3, 4};
    Point* ptr = &p;   // 指针指向对象
    ptr->x = 10;       // 相当于 (*ptr).x = 10;
    return 0;
}

```

| 代码示例   | 适用情况             | 解释                                         |
| ---------- | -------------------- | -------------------------------------------- |
| `q.first`  | `q` 是对象或对象引用 | 直接访问 `q` 的成员                          |
| `q->first` | `q` 是指针           | 访问 `q` 指向对象的成员，相当于 `(*q).first` |

| 成员访问                            | 使用 `.`                          | 使用 `->`                   |
| ----------------------------------- | --------------------------------- | --------------------------- |
| 访问对象的成员                      | `object.member`                   | `pointer->member`           |
| 访问对象的引用的成员                | `reference.member`                | ❌ 不适用                    |
| 访问指针指向的对象的成员            | `(*pointer).member`（需要解引用） | `pointer->member`（更简洁） |
| 访问迭代器指向的 `std::pair` 的成员 | `(*it).first`                     | `it->first`                 |

**如果你有对象，使用 `.`**。

**如果你有指针，使用 `->`**。

**如果是迭代器（如 `std::map<int, string>::iterator`），通常使用 `->`**（因为迭代器返回的是 `std::pair<int, T>*`）。
