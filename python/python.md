# 文件读写

## open

```python
open('file','mode')
f = open('file','mode')
```

file：需要打开的文件路径
mode（可选）：打开文件的模式，mode参数可以省略，默认为r

mode参数还可以指定以什么样的编码方式读写文本，默认情况下open是以文本形式打开文件的，比如下面的四种mode模式。

- r：表示文件只能读取

- w：表示文件只能写入

- a：表示打开文件，在原有内容的基础上追加内容，在末尾写入

- w+:表示可以对文件进行读写双重操作

- rb：以二进制格式打开一个文件，用于只读

- wb：以二进制格式打开一个文件，用于只写

- ab：以二进制格式打开一个文件，用于追加

- wb+:以二进制格式打开一个文件，用于读写

当你在默认模式下读取文本文件时（二进制文件不可以），文件中的换行符会转换为’\n’形式。相反，在默认模式         下写入文件时，文本中的’\n’会转换为换行符。也就是说，你读取的txt文本，其中换行符会以’\n’形式出现，写入txt文本时，文本中的’\n’会变成换行指令。

## with

在打开文件时使用 with 关键字。优点是当子句体结束后文件会正确关闭，即使在某个时刻引发了异常。

```python
>>> with open('workfile') as f:
...     read_data = f.read()
>>> # 查看文件是否关闭
>>> f.closed
True
```

with 语句实现原理建立在上下文管理器之上。上下文管理器是一个实现 **__enter__** 和 **__exit__** 方法的类。使用 with 语句确保在嵌套块的末尾调用 __exit__ 方法。

https://www.runoob.com/python3/python-with.html

## close

打开文件并处理完毕后，需要关闭文件，这里用到close方法。

f.close() 用来关闭文件并立即释放它使用的所有系统资源。如果你没有显式地关闭文件，Python的垃圾回收器最终将销毁该对象并为你关闭打开的文件，但这个文件可能会保持打开状态一段时间。

```python
f = open(file) # 打开文件
f.close() # 关闭文件
```

## read

read()会读取一些数据并将其作为字符串（在文本模式下）或字节对象（在二进制模式下）返回。

```python
f.read(size) # f为文件对象
```

参数size（可选）为数字，表示从已打开文件中读取的字节计数，默认情况下为读取全部。

## readline

readline方法从文件中读取整行，包括换行符’\n’。

换行符（\n）留在字符串的末尾，如果文件不以换行符结尾，则在文件的最后一行省略，这使得返回值明确无误。

如果 f.readline() 返回一个空的字符串，则表示已经到达了文件末尾，而空行使用 ‘\n’ 表示，该字符串只包含一个换行符。

```python
f.readline(size)
```

参数size表示从文件读取的字节数。

readline方法会记住上一个readline函数读取的位置，接着读取下一行。

## readlines

readlines方法是读取所有行，返回的是所有行组成的列表。

```python
list = f.readlines()
#list为列表，为['第1行内容','第2行内容',...]
```

## write

write方法顾名思义，就是将字符串写入到文件里。

```python
f.write([str]) # f为文件对象
```

参数[str]代表要写入的字符串

## writelines

```python
# 创建一个列表
txtlist = ['Python\n', 'Java\n', 'C++\n']

# 写入文件
with open('hello.txt') as hello :
    hello.writelines(txtlist)
```

## tell

返回文件对象当前的位置，它是从文件开头开始算起的字节数

## seek

f.seek(x,0)：从起始位置即文件首行首字符开始移动x个字符，0为默认值

f.seek(x,1)：从当前位置开始移动x个字符

f.seek(-x,2)：从文件的结尾往前移动x个字符

```python
f.seek(f.tell() + 4) #跳过无用字节
```

https://blog.csdn.net/m0_59162248/article/details/127778049

https://blog.csdn.net/qq_41340258/article/details/124148415

https://blog.csdn.net/weixin_46554184/article/details/121195167



# struct模块

 struct模块中最主要的三个函数式pack()、unpack()、calcsize()。

  pack(fmt, v1, v2, ...) ------ 根据所给的fmt描述的格式将值v1，v2，...转换为一个字符串。

  unpack(fmt, bytes)  ------ 根据所给的fmt描述的格式将bytes反向解析出来，返回一个**元组**。

  calcsize(fmt)             ------ 根据所给的fmt描述的格式返回该结构的大小。

| 格式 | C 类型             | Python 类型       | 标准大小 |
| :--- | :----------------- | :---------------- | :------- |
| `x`  | 填充字节           | 无                |          |
| `c`  | char               | 长度为 1 的字节串 | 1        |
| `b`  | signed char        | 整数              | 1        |
| `B`  | unsigned char      | 整数              | 1        |
| `?`  | _Bool              | bool              | 1        |
| `h`  | short              | 整数              | 2        |
| `H`  | unsigned short     | 整数              | 2        |
| `i`  | int                | 整数              | 4        |
| `I`  | unsigned int       | 整数              | 4        |
| `l`  | long               | 整数              | 4        |
| `L`  | unsigned long      | 整数              | 4        |
| `q`  | long long          | 整数              | 8        |
| `Q`  | unsigned long long | 整数              | 8        |
| `n`  | ssize_t            | 整数              |          |
| `N`  | size_t             | 整数              |          |
| `e`  | (6)                | float             | 2        |
| `f`  | float              | float             | 4        |
| `d`  | double             | float             | 8        |
| `s`  | char[]             | 字节串            |          |
| `p`  | char[]             | 字节串            |          |
| `P`  | void*              | 整数              |          |

注意： 

- c,s和p按照bytes对象执行转码操作，但是在使用UTF-8编码时，也支持str对象。
- ‘？’按照C99中定义的_Bool类型转码。如果该类型不可用，可使用一个char冒充。
- ‘q'和’Q‘仅在64位系统上有用。

| 字符 | 字节顺序      | 大小     | 对齐方式 |
| :--- | :------------ | :------- | :------- |
| `@`  | 按原字节      | 按原字节 | 按原字节 |
| `=`  | 按原字节      | 标准     | 无       |
| `<`  | 小端          | 标准     | 无       |
| `>`  | 大端          | 标准     | 无       |
| `!`  | 网络（=大端） | 标准     | 无       |

例子1：

```c
struct Header {
	unsigned short id;
	char[4] tag;
	unsigned int version;
	unsigned int count;
}
```

通过socket.recv接收到了一个上面的结构体数据，存在字符串s中，现在需要把它解析出来，可以使用unpack()函数。

```python
import struct
id, tag, version, count = struct.unpack('!H4s2I', s)
```

上面的格式字符串中，!表示我们要使用网络字节顺序解析，因为我们的数据是从网络中接收到的，在网络上传送的时候它是网络字节顺序的。后面的H表示 一个unsigned short的id,4s表示4字节长的字符串，2I表示有两个unsigned int类型的数据/

例子2：

```python
import struct

#将a变为二进制
a = 12.34
bytes = struct.pack('i', a)
```

此时bytes就是一个string字符串，字符串按字节同a的二进制存储内容相同。

再进行反操作，现有二进制数据bytes（其实就是字符串），将它反过来转换成python的数据类型：

```python
a,=struct.unpack('i', bytes)
```

注意，unpack返回的是tuple，所以解码的时候需要这样

```python
a, = struct.unpack('i', bytes)
#或者
(a,) = struct.unpack('i', bytes)
#或者
a = struct.unpack('i', bytes)[0]
```

如果直接用a = struct.unpack('i', bytes)，那么 a = (12.34,) ，是一个tuple而不是原来的浮点数了。



# if __name__ == '__main__':的作用和原理

作用：

一个python文件通常有两种使用方法，第一是作为脚本直接执行，第二是 import 到其他的 python 脚本中被调用（模块重用）执行。因此 if __name__ == '__main__': 的作用就是控制这两种情况执行代码的过程，在 if __name__ == '__main__': 下的代码只有在第一种情况下（即文件作为脚本直接执行）才会被执行，而 import 到其他脚本中是不会被执行的。

原理：

每个python模块（python文件，也就是此处的 test.py 和 import_test.py）都包含内置的变量 __name__，当该模块被直接执行的时候，__name__ 等于文件名（包含后缀 .py ）；如果该模块 import 到其他模块中，则该模块的 __name__ 等于模块名称（不包含后缀.py）。

举例：

```python
#test.py
print('1')
print('__name__:', __name__)
if __name__ = '__main__':
	print('2')
```

```python
#import_test.py
import test
```

分别执行两个程序，输出为：

```
1
__name__:__main__
2
```

```
1
__name__:test
```

原文链接：https://blog.csdn.net/heqiang525/article/details/89879056



# 内置函数

## ord chr unichr

chr()函数⽤⼀个范围在range（256）内的（就是0～255）整数作参数，返回⼀个对应的字符。

unichr()跟它⼀样，只不过返回的是 Unicode字符，这个从Python 2.0才加⼊的unichr()的参数范围依赖于你的Python是如何被编译的。如果是配置为USC2的Unicode，那么它的允许范围就是range（65536）或0x0000-0xFFFF；如果配置为UCS4，那么这个值应该是 range（1114112）或0x000000-0x110000。如果提供的参数不在允许的范围内，则会报⼀个ValueError的异常。

ord()函数是chr()函数（对于8位的ASCII字符串）或unichr()函数（对于Unicode对象）的配对函数，它以⼀个字符（长度为1的字符串） 作为参数，返回对应的ASCII数值，或者Unicode数值，如果所给的Unicode字符超出了你的Python定义范围，则会引发⼀个TypeError的 异常。

```python
>>>chr(65)
'A'
>>>ord('a')
97
>>>unichr(12345)
u'\u3039'
>>>chr(12345)
Traceback (most recent call last):
File "<stdin>", line 1,in ?
chr(12345)
ValueError:chr() arg not in range(256)
>>>ord(u'\ufffff')
Traceback (most recent call last):
File "<stdin>", line 1,in ?
ord(u'\ufffff')
TypeError:ord() expected a character, but string of length 2 found
>>>ord(u'\u2345')
9029
```

## set

set() 函数创建一个无序不重复元素集，可进行关系测试，删除重复数据，还可以计算交集、差集、并集等。

set，接收一个list作为参数

```python
list1 = [1, 2, 3, 4, 3, 5, 2]
s = set(list1)
print(s)
#逐个遍历
for i in s:
	print(i)

#输出:
{1, 2, 3, 4, 5}
1
2
3
4
5
```

使用add(key)往集合中添加元素，重复的元素自动过滤

```python
list1 = [1, 2, 3, 4]
s = set(list1)
print(s)
s.add(4)
s.add(5)
print(s)

#输出：
{1, 2, 3, 4}
{1, 2, 3, 4, 5}
```

通过remove(key)方法可以删除元素：

```python
list1 = ['a', 'b', 'zhang', 'kang']
s = set(list1)
print(s)
s.remove('zhang')
print(s)

#输出：
{'b', 'zhang', 'a', 'kang'}
{'b', 'a', 'kang'}
```

set还可以像数学上那样求交集和并集

```python
list1 = ['a', 'b', 'zhang', 'kang']
list2 = ['a', 'b', 'c', 'd']
s1 = set(list1)
s2 = set(list2)
#交集，使用&操作符
s3 = s1 & s2
#并集，使用|操作符
s4 = s1|s2
print(s3)
print(s4)

#输出：
{'a', 'b'}
{'kang', 'zhang', 'd', 'b', 'c', 'a'}
```



# 字符串

## join

```python
'sep'.join(sep_object)
```

sep：分割符，可为“，、；”等。

sep_object：分割对象，可为字符串、以及储存字符串的元组、列表、字典。

用法：连接任意数量的字符串（包括要连接的元素字符串、元组、列表、字典），用新的目标分隔符连接，返回新的字符串。

- 对象为字符串

  ```python
  ';'.join('abc') 
  #输出结果为：'a;b;c'  
   
  string1 = 'good idea'
  ' '.join(string1)  
  #输出结果：'g o o d   i d e a' 
  #说明：由于字符串里没指明按字符串之间是怎么连接的，默认每个字符之间插入目标字符
  ```
  
- 对象为元组

  ```python
  tuple1 = ('a','b','c')
  '、'.join(tuple1)
  #输出：'a、b、c'
  
  tuple2 = ('hello','peace','world')
  ' '.join(tuple2)
  #输出：'hello peace world'
  ```

- 对象为列表

  ```python
  b = ['a','b','c']
  '、'.join(b)
  #输出：'a、b、c'
   
  list1 = ['hello','peace','world']
  ' '.join(list1)
  #输出：'hello peace world'
  ```

- 对象为字典

  ```python
  c={'hello':1,'world':2}
  ';'.join(c)
  #输出：'hello;world'
  
  d = {'hello':'hi','world':'2'}
  ' '.join(d)
  #输出：'hello world'
  ```

问题：储存非字符串的元组、列表、字典等报错，比如元组储存数字进行连接

```python
a = (1,2,3)
';'.join(a)
#报错:TypeError: sequence item 0: expected str instance, int found
```

解决办法：要将数字连接起来成为一个字符串，则结合for循环语句并将数字转为字符串再连接起来

```python
b = (186234,1385475,1235462)
';'.join(str(i) for i in b) 
#输出：'186234;1385475;1235462'
 
e = (1,2,3,2)
'、'.join(str(i) for i in e)
#输出：'1、2、3、2'
```

## replace

在python3中，replace用于对字符串中的元素进行替换，返回一个新的字符串，__不会对原有字符串进行更改__。因为字符串是不可变类型。

语法：

```python
string.replece(old, new, count)
```

参数:

- old : 需要被替换的字符

- new：新的字符
- count ：最多替换几次

返回值:

- 返回一个新的字符串

实例：

```python
s = 'abcabcabc'
new_s = s.replace('a', 'b', 2)
print(s.replace('a', 'b', 2))
print(s)
 
# 结果
bbcbbcabc
abcabcabc
```



# skimage

| 子模块       | 功能                                                        |
| ------------ | ----------------------------------------------------------- |
| io           | 读取、保存和显示图片或视频                                  |
| data         | 提供一些测试图片和样本数据                                  |
| color        | 颜色空间变换                                                |
| filters      | 图像增强、边缘检测、排序滤波器、自动阈值等                  |
| draw         | 操作于numpy数组上的基本图形绘制，包括线条、矩形、圆和文本等 |
| transform    | 几何变换或其它变换，如旋转、拉伸和拉东变换等                |
| morphology   | 形态学操作，如开闭运算、骨架提取等                          |
| exposure     | 图片强度调整，如亮度调整、直方图均衡等                      |
| feature      | 特征检测与提取等                                            |
| measure      | 图像属性的测量，如相似性或等高线等                          |
| segmentation | 图像分割                                                    |
| restoration  | 图像恢复                                                    |
| util         | 通用函数                                                    |

## 读取图像

```python
from skimage import io
img = io.imread('test.png', as_gray=False)  # 第一个参数是文件名可以是网络地址，第二个参数默认为False，True时为灰度图
```

彩色图片格式为：img[i, j, c]

i表示图片的行数，j表示图片的列数，c表示图片的通道数（RGB三通道分别对应0，1，2）。坐标是从左上角开始。

灰度图片格式为：gray[i, j]

查看image文件的信息

```python
print(type(img))  #显示类型
print(img.shape)  #显示尺寸
print(img.dtype)   #数据类型
print(img.shape[0])  #图片高度
print(img.shape[1])  #图片宽度
print(img.shape[2])  #图片通道数
print(img.max())  #最大像素值
print(img.min())  #最小像素值
print(img.mean()) #像素平均值
```

skimage读出来的图片可以直接img[0] [0]获得，但是一定记住它的格式

```python
print(img[200][100])
```

skimage.io.imread打开的图片类型为np数组, 值为0-255，尺寸为 H,W,C，resize后值为0-1

```python
image = transform.resize(img, (100, 200), order=1)
print(skimage.img_as_float(image))   # img_as_float可以把image转为double，即float64 
```

# PIL image

https://www.jb51.net/article/184195.htm

## image.new

- **Image.new( mode, size, color )**
- **mode**：图片的模式。"1", "CMYK", "F", "HSV", "I", "L", "LAB", "P", "RGB", "RGBA", "RGBX", "YCbCr"
- 1：1位像素，表示黑和白，但是存储的时候每个像素存储为8bit。
  L：8位像素，表示黑和白。
  P：8位像素，使用调色板映射到其他模式。
  RGB：3x8位像素，为真彩色。
  RGBA：4x8位像素，有透明通道的真彩色。
  CMYK：4x8位像素，颜色分离。
  YCbCr：3x8位像素，彩色视频格式。
  I：32位整型像素。
  F：32位浮点型像素。
  PIL也支持一些特殊的模式，包括RGBX（有padding的真彩色）和RGBa（有自左乘alpha的真彩色）。
- **size**：一个含有图片 宽，高 的元组；图片的尺寸(width, height)
- **color**：图片的颜色；其默认值为0，即 黑色；

> **特别情况：当设置图片的 mode 为 ‘RGBA’ 时，如果不填 color 参数的话，图片是 透明底！**
> 即以下方法，建立的图片对象为 透明底！如果添加文字后，保存为 png 格式，你会得到一张透明底的文字图片！需要绘制透明底图的时，你会需要的。
>
> ```text
> Image.new('RGBA', (800, 400))
> ```

```python
#创建一个 RGB 模式，宽800、高400，白色的图片
from PIL import Image
img = Image.new('RGB', (800, 400), "white")
img.show()
```

https://zhuanlan.zhihu.com/p/585399599

## image.save

```python
img.save("example.jpg", quality=90)
#使用参数“quality”来指定图片的质量，取值范围为0~100
img.save("example.jpeg", format="JPEG")
```

https://www.python100.com/html/89063.html

## image.size

- 这是一个二元组，包含水平和垂直方向上的像素数

  ```python
  from PIL import Image
  im = Image.open("xiao.png")
  print(im.size)
  输出:
  (670, 502)
  ```

## image.load

```python
from PIL import Image
img_array = image.load()
#然后就可以通过img_array[x,y]来读取像素值了
```

# wave

https://blog.csdn.net/daydayup858/article/details/128253776

- 不支持压缩/解压，但支持单声道/立体声语音的读取

## wave.open()：

打开一个文件以读取/写入音频数据

两个参数：文件名，模式（写入“ wb” /读取“ rb”）

- 模式“ rb”返回Wave_read对象
- 模式“ wb”返回Wave_write对象

## wave_write对象：

```
close()
如果文件是通过wave打开的，则将其关闭。

setnchannels()
设置频道数。1单声道2个立体声通道

setsampwidth()
将样本宽度设置为n个字节。

setframerate()
将帧频设置为n。

setnframes()
将帧数设置为n。

setcomptype()
设置压缩类型和描述。目前，仅支持压缩类型NONE（无压缩）。

setparams()
接受参数元组(nchannel,sampwidth,framerate,nframe,comptype,compname)

tell()
检索文件中的当前位置

writeframesraw()
编写音频帧，而不进行校正。

writeframes()
编写音频帧，并确保它们正确。
```

注意：请注意，在调用`writeframes()`或`writeframesraw()`之后设置任何参数都是无效的，任何try都会引发[wave.Error](https://www.docs4dev.com/docs/zh/python/2.7.15/all/library-wave.html#wave.Error)。

## wave_read对象：

```
close()
如果流是通过wave模块打开的，则将其关闭。

getnchannels()
返回音频通道的数量（单声道为1，立体声为2）。

getsampwidth()
返回以字节为单位的样本宽度。

getframerate()
返回采样频率。

getnframes()
返回音频帧数。

getcomptype()
返回压缩类型（“ NONE”是唯一受支持的类型）。

getparams()
返回一个namedtuple()(nchannels,sampwidth,framerate,nframe,comptype,compname)，它等于get *()方法的输出。

readframes(n)
作为字节对象读取和返回最多n帧音频。

rewind()
将文件指针倒退到音频流的开头。
```

# 异常处理

```
try:
  可能产生异常的代码块
except [ (Error1, Error2, ... ) [as e] ]:
  处理异常的代码块1
except [ (Error3, Error4, ... ) [as e] ]:
  处理异常的代码块2
except [Exception]:
  处理其它异常
```

```
try:
    a = int(input("输入被除数："))
    b = int(input("输入除数："))
    c = a / b
    print("您输入的两个数相除的结果是：", c )
except (ValueError, ArithmeticError):
    print("程序发生了数字格式异常、算术异常之一")
except :
    print("未知异常")
print("程序继续运行")
```

- 在原本的try except结构的基础上，Python 异常处理机制还提供了一个 else 块，也就是原有 try except 语句的基础上再添加一个 else 块，即try except else结构。 使用 else 包裹的代码，只有当 try 块没有捕获到任何异常时，才会得到执行；反之，如果 try 块捕获到异常，即调用对应的 except 来处理异常，else 块中的代码也不会得到执行。
- Python 异常处理机制还提供了一个 finally 语句，通常用来为 try 块中的程序做扫尾清理工作。在整个异常处理机制中，finally 语句的功能是：无论 try 块是否发生异常，最终都要进入 finally 语句，并执行其 中的代码块。

# 格式化输出

- 使用**f"...{}..."**

```python
# x_train is the input variable (size in 1000 square feet)
x_train = np.array([1.0, 2.0])
print(f"x_train = {x_train}")
# m is the number of training examples
print(f"x_train.shape: {x_train.shape}")
m = x_train.shape[0]
print(f"Number of training examples is: {m}")
```

