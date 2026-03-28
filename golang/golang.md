# if

## 1. 定义

条件分支用于**根据不同条件执行不同代码**，控制程序逻辑走向。

Go 仅支持：`if`、`if-else`、`if-else if-else`、`switch`，**没有三元运算符**。

------

## 2. if 语句（最常用）

### 2.1 语法

```go
if 条件 {
    // 条件为 true 执行
}
```

- 条件**不需要括号**
- 大括号`{`必须和`if`在同一行

### 2.2 例子

```go
age := 18
if age >= 18 {
    fmt.Println("成年")
}
```

------

## 3. if-else

### 3.1 语法

```go
if 条件 {
} else {
}
```

### 3.2 例子

```go
age := 16
if age >= 18 {
    fmt.Println("成年")
} else {
    fmt.Println("未成年")
}
```

------

## 4. if-else if-else（多条件）

### 4.1 语法

```go
if 条件1 {
} else if 条件2 {
} else {
}
```

### 4.2 例子

```go
score := 85
if score >= 90 {
    fmt.Println("A")
} else if score >= 80 {
    fmt.Println("B")
} else {
    fmt.Println("C")
}
```

------

## 5. if 简易赋值（极简写法，高频）

可以**先赋值，再判断**，变量只在 if 内有效。

```go
if 变量 := 表达式; 条件 {
}
```

例子（map 判断 key 是否存在）：

```go
m := map[int]int{1: 10}
if val, ok := m[1]; ok {
    fmt.Println("存在，值：", val)
}
```

例子（切片判空）：

```go
s := []int{}
if len(s) == 0 {
    fmt.Println("空切片")
}
```

------

## 6. switch 语句（替代多 if-else）

### 6.1 语法

```go
switch 变量 {
case 值1:
case 值2:
default:
}
```

- **不需要 break**（自动中断）
- 支持多值匹配

### 6.2 例子

```go
day := 2
switch day {
case 1:
    fmt.Println("周一")
case 2:
    fmt.Println("周二")
default:
    fmt.Println("其他")
}
```

### 6.3 无表达式 switch（当 if-else 用）

```go
score := 85
switch {
case score >= 90:
    fmt.Println("优秀")
case score >= 80:
    fmt.Println("良好")
}
```

------

## 7. 特别注意

1. 条件只能是 bool 类型，不能用数字代替

   ```go
   if 1 { // 错误
   }
   ```

2. **`{` 不能换行**

3. **没有三元运算符 `a > b ? a : b`**

4. switch 默认不穿透，如需穿透用 `fallthrough`（极少用）

------

## 8. 速记总结

- 单条件：`if`
- 二选一：`if-else`
- 多条件：`if-else if-else`
- 简洁赋值判断：`if a := f(); a > 0 {}`
- 多分支清晰：`switch`
- 都不用括号，`{` 不换行



# for

## 1. for 循环定义

Go 中的 `for` 循环是**唯一循环方式**，无 `while` 关键字，可灵活实现「遍历、循环执行、条件循环」等所有场景，语法简洁且适配各类需求。

## 2. 核心语法（3 种常用形式）

### 2.1 基础条件循环（替代 while）

最常用，无固定次数，满足条件就执行。

```go
// 语法：for 条件 { 执行逻辑 }
for 条件 {
    // 循环体
}
```

### 2.2 固定次数循环（类似 for-i 循环）

指定循环次数，适合已知迭代次数的场景。

```go
// 语法：for 初始值; 条件; 步长 { 执行逻辑 }
for i := 0; i < 5; i++ {
    // 循环5次，i从0到4
}
```

### 2.3 无限循环（需手动终止）

无任何条件，直接循环，需用 `break` 退出。

```go
for {
    // 执行逻辑
    if 退出条件 {
        break // 终止循环
    }
}
```

## 3. 关键使用场景 + 示例

### 场景 1：基础条件循环（替代 while）

```go
// 需求：循环打印1-5
i := 1
for i <= 5 {
    fmt.Println(i) // 输出1-5
    i++ // 步长自增
}
```

### 场景 2：固定次数循环

```go
// 需求：循环3次，打印索引
for i := 0; i < 3; i++ {
    fmt.Println("循环次数：", i+1) // 1、2、3
}
```

### 场景 3：遍历切片 / 数组（最常用）

```go
nums := []int{1,2,3,4}
// 遍历切片，获取索引和值（_忽略索引）
for _, num := range nums {
    fmt.Println(num) // 输出1、2、3、4
}
```

### 场景 4：遍历 map（无顺序）

```go
mp := map[string]int{"a":1, "b":2}
// 遍历key和value（_忽略key，只取value）
for _, val := range mp {
    fmt.Println(val) // 输出1、2
}
```

### 场景 5：无限循环（手动终止）

```go
// 需求：循环直到输入为0
var num int
for {
    fmt.Scan(&num)
    if num == 0 {
        break // 满足条件，终止循环
    }
    fmt.Println("输入值：", num)
}
```

## 4. 注意事项（避坑重点）

1. for 循环中，`初始值; 条件; 步长` 三个部分可缺省（如只写条件，或全不写）。
2. 遍历切片 /map 时，用 `for range` 最简洁，无需手动控制索引。
3. 终止循环用 `break`，跳过当前迭代用 `continue`。

## 5. 核心总结

| 用法类型 |         语法示例          |         适用场景         |
| :------: | :-----------------------: | :----------------------: |
| 条件循环 |       for i <= 5 {}       | 替代 while，未知循环次数 |
| 固定次数 |   for i:=0; i<3; i++ {}   |       已知循环次数       |
| 无限循环 |          for {}           |    需手动 break 终止     |
| 遍历数据 | for k, v := range 数据 {} |   切片、map、字符串等    |



# 切片

## 一、切片是什么？

1. **定义**：切片是**动态数组**，长度可以随时变化，比数组更灵活。
2. **本质**：指向底层数组的「指针 + 长度 len + 容量 cap」。
3. 对比数组：
   - 数组：`[3]int{1,2,3}` 长度固定
   - 切片：`[]int{1,2,3}` 长度可变

------

## 二、切片的 4 种创建方式

### 1. 直接创建（最常用）

```go
s := []int{1, 2, 3}
```

### 2. 空切片（推荐）

```go
s := []int{} // 非 nil，可直接 append
```

### 3. nil 切片

```go
var s []int // nil切片，也能 append
```

### 4. make 创建（指定长度 / 容量）

```go
s := make([]int, 长度, 容量)
例：s := make([]int, 0, 10) // 长度0，容量10
```

------

## 三、核心操作

### 1. 追加元素 append（对应 C++ push_back）

```go
s := []int{}
s = append(s, 10)    // 追加一个
s = append(s, 20, 30) // 追加多个
```

**必须用变量接收返回值！**

------

### 2. 获取长度 & 容量

```go
len(s) // 长度：实际元素个数
cap(s) // 容量：最多能存多少
```

------

### 3. 访问 / 修改元素

```go
s := []int{10, 20, 30}
fmt.Println(s[0]) // 取：10
s[1] = 99        // 改：10,99,30
```

------

### 4. 切片截取（子切片）

```go
s := []int{1,2,3,4,5}
s[1:4] // [2,3,4] 左闭右开
```

------

### 5. 遍历切片

```go
// 常用：只取值
for _, num := range s {
    fmt.Println(num)
}

// 索引+值
for i, num := range s {
    fmt.Println(i, num)
}
```

------

### 6. 判断切片是否为空

**永远用 len (s) == 0**

```go
if len(s) == 0 {
    // 空切片
}
```

------

## 四、二维切片（算法高频）

```go
// 创建空二维切片
ans := [][]int{}

// 追加一行
ans = append(ans, []int{1,2,3})
```

------

## 五、nil 切片 vs 空切片

```go
var s []int      // nil 切片 s==nil → true
s2 := []int{}    // 空切片 s2==nil → false
```

**共同点：都能 append，都能用 len (s)==0 判断**

------

## 六、最常用示例合集

### 示例 1：创建并追加

```go
s := []int{}
s = append(s, 1)
s = append(s, 2)
// s = [1,2]
```

### 示例 2：遍历

```go
s := []int{10,20,30}
for _, v := range s {
    fmt.Println(v)
}
```

### 示例 3：截取

```go
s := []int{1,2,3,4,5}
sub := s[1:3] // [2,3]
```

------

## 七、核心总结（必背）

1. **切片是动态数组，用 append 追加**
2. **创建空切片推荐：s := [] int {}**
3. **判断空：len (s) == 0**
4. **append 必须赋值回去**
5. **for range 遍历最简洁**



# map

## 1. 定义

- `map` 是 Go 中的 **键值对（key-value）** 无序集合
- 语法：`map[KeyType]ValueType`
- key 必须是可比较类型（int、string、bool 等）
- 底层哈希表，增删改查平均 O (1)

------

## 2. 创建 map

### 2.1 字面量创建（推荐）

```go
// 空 map
m := map[string]int{}

// 带初始值
m := map[string]int{
    "apple":  1,
    "banana": 2,
}
```

### 2.2 make 创建（可指定容量）

```go
// 空 map
m := make(map[string]int)

// 指定容量，提升性能
m := make(map[string]int, 10)
```

### 2.3 注意

```go
// nil map，不能赋值，会 panic
var m map[string]int
```

------

## 3. 增 / 改

```go
m := map[string]int{}

// 新增 key
m["alice"] = 18

// 已存在 key 会覆盖
m["alice"] = 20
```

------

## 4. 查（判断 key 是否存在）

### 4.1 标准写法（必掌握）

```go
val, ok := m["alice"]
if ok {
    // 存在
} else {
    // 不存在
}
```

### 4.2 简写

```go
if val, ok := m["alice"]; ok {
    fmt.Println(val)
}
```

> 直接 `m["alice"]` 不存在会返回零值，无法区分是真 0 还是不存在。

------

## 5. 删

```go
// 删除 key
delete(m, "alice")

// key 不存在时，delete 不会 panic
```

------

## 6. 遍历 map

### 6.1 遍历 key + value

```go
for k, v := range m {
    fmt.Println(k, v)
}
```

### 6.2 只遍历 key

```go
for k := range m {
    fmt.Println(k)
}
```

### 6.3 只遍历 value

```go
for _, v := range m {
    fmt.Println(v)
}
```

> map 遍历**无序**，每次顺序可能不一样。

------

## 7. 长度

```go
// 获取键值对数量
fmt.Println(len(m))

// 判断空 map
if len(m) == 0 {
}
```

------

## 8. 常用示例

### 8.1 统计字符出现次数

```go
s := "abac"
cnt := map[rune]int{}
for _, c := range s {
    cnt[c]++
}
// cnt: a:2, b:1, c:1
```

### 8.2 map 存切片

```go
m := map[string][]string{}
m["group1"] = append(m["group1"], "a", "b")
```

------

## 9. 总结

- 创建：`map[K]V{}` 或 `make(map[K]V)`
- 增改：`m[key] = val`
- 查询：`val, ok := m[key]` 判断存在
- 删除：`delete(m, key)`
- 遍历：`for k, v := range m`
- 判空：`len(m) == 0`



# sort

## 1. 定义

`sort` 是 Go 标准库，用于对**切片、自定义类型**进行排序，支持升序、降序与自定义排序规则。

## 2. 常用排序方式

### 2.1 基本类型快捷排序

适用于 `[]int`/`[]string`/`[]float64`，无需自定义函数。

```go
package main

import "sort"

func main() {
    // int 升序
    ints := []int{3, 1, 4, 2}
    sort.Ints(ints)

    // string 字典序
    strs := []string{"banana", "apple", "cat"}
    sort.Strings(strs)

    // float64 升序
    floats := []float64{3.1, 1.4, 2.0}
    sort.Float64s(floats)
}
```

### 2.2 万能排序 sort.Slice

对任意切片排序，必须传入**比较函数**定义规则。

- `i, j`：切片下标
- 返回 `true`：`i` 排在 `j` 前

```go
package main

import "sort"

func main() {
    nums := []int{3,1,4,2}
    // 升序
    sort.Slice(nums, func(i, j int) bool {
        return nums[i] < nums[j]
    })
    // 降序
    sort.Slice(nums, func(i, j int) bool {
        return nums[i] > nums[j]
    })
}
```

### 2.3 结构体排序

使用 `sort.Slice` 按字段排序。

```go
type User struct {
    Name string
    Age  int
}

users := []User{
    {"Bob", 20},
    {"Alice", 18},
}

// 按年龄升序
sort.Slice(users, func(i, j int) bool {
    return users[i].Age < users[j].Age
})
```

### 2.4 字节切片排序（字符串字母排序）

```go
s := []byte("bac")
sort.Slice(s, func(i, j int) bool {
    return s[i] < s[j]
})
// s → abc
```

## 3. 常用判断

- `sort.IntsAreSorted([]int)`：是否已升序
- `sort.StringsAreSorted([]string)`：是否已字典序

## 4. 简明总结

1. 基本类型 → `sort.Ints/Strings/Float64s`
2. 任意切片 / 结构体 → `sort.Slice` + 比较函数
3. `<` 升序，`>` 降序