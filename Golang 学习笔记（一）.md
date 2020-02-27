## Golang 学习笔记（一）

### Go SDK介绍

#### go/bin/

Go 的一些指令，主要有 `go` , `godoc`, `gofmt` 。

#### go/src/

Go 的源代码

### Go 环境变量

以下举例 Go SDK 安装在 `~/go/`。

#### GOROOT

指定 SDK 的安装路径。GO 安装在哪里，就配置为哪里。GO 的根目录

Example Unix:

```shell
~:cd /etc
~:sudo vim profile
/etc/profile
#添加如下语句
export GOROOT=~/go
```

#### PATH

添加 SDK 下的 `/bin` 目录

GO根目录下的 `/bin` 目录

Example:

```shell
#接上
export PATH=$GOROOT/bin:$PATH
```

 #### GOPATH

工作目录，Go 项目的工作路径,可配置多个

Example:

```shell
#接上
export GOPATH=$HOME/myGoprojects
```



### 声明变量

```Go
//1
var v1 int = 1
//2
v2 := 1
//3
var (
	v1 int
  v2 int
  v3 string
)
//驼峰命名法 var userName string = 'chaosir'
//变量可被外部包使用,全局变量首字母大写 var GlobalGender string = 'female'
```



### 常量

```go
const TIME_ZONE = 'Asia/Shanghai'
```



### 数据类型

```go
var (
	v1 bool = true	//布尔类型
  v2 int8 = 1			//带符号8位整型 值范围 -128~127 	默认值0  占用 1字节
  v3 uint8 = 1 		//无符号8位整型 值范围 0~255			默认值0	占用 1字节 还是byte类型的别名
  v4 int16 = 1		//带符号16位整型	值范围 -32768~32767 默认值0	占用 2字节
  v5 uint16 = 1 	//无符号16位整型	值范围 0~65536				默认值0	占用 2字节
  v6 int32 = 1		//带符号32位整型	值范围 -2147483648~2147483647 默认值0 占用 4字节
  v7 uint32 = 1 	//无符号32位整型	值范围 0~4294967295		默认值0 占用 4字节
  v8 int64 = 1		//带符号64位整型	值范围 -9223372036854775808~9223372036854775807	默认值0 占用 8字节
  v9 uint64 = 1 	//无符号64位整型	值范围 0~18446744073709551615	默认值0 占用 8字节
  v10 int = 1			//32位或64位			与具体平台相关	默认值0
  v11 uint = 1		//32位或64位			与具体平台相关 默认值0
  v12 uintptr			//无符号整型，足以存储指针值的未解释位	值范围 32位平台下4字节 64位平台下8字节 默认值0
  v13 float32 = 3.14 //带符号32位浮点型 等价于PHP的 float 类型(单精度浮点数) 可精确到小数点后 7 位
  v14 float64 = 1.00 //带符号64位浮点型 等价于PHP的 double 类型（双精度浮点数） 可精确到小数点后 15 位 默认赋值小数位此类型 v := 1.0 (type float64) 实际开发尽可能使用此类型
  v15 complex64 = 1.10 + 10i //由两个 float32 实数构成的复数类型
  v16 complex128 = 1.10 + 10i	//和浮点型一样，默认自动推导的实数类型是 float64，所以默认赋值是此类型 v:= 1.10 + 10i (type complex128) real(v16) 获得该复数的实部，imag(v16)获得该复数的虚部
  v17 string = "Hello world" //字符串类型
  v18 rune = 'a' //字符类型 代表单个 Unicode 字符
  v19 [8]byte 		//长度为8的数组，每个元素为1个字节
  v20 [3][3]int 	//二维数组（9宫格）,每个元素为 1个 int类型
  v21 [3][3][3]float64 //三维数组(立体的9宫格),每个元素为 1个 float64 类型
  v22 = [3]int{1,2,3} //声明时初始化 [1,2,3]
  v23 = new([3]string) //通过 new 初始化
  v24 = [...]int{1,2,3} //省略数组长度的声明
  v25 = [5]int{1,2,3} //[1 2 3 0 0] 没有填满，空位以对应类型空值填充
  v26 = [5]int{1:3,3:7} // [0 3 0 7 0]
  v27 = []string{"a","b","c"} //数组切片
  v28 = make([]int,5,10)			//切片 [0 0 0 0 0]
  v29 = map[string]int{"one":1,"two":2}	//字典类型 string是键的类型 int是值的类型
  v30 = make(map[string]int) //字典类型 这种方法初始化后可以像PHP那样往字典中添加键值对 如 v30["one"] = 1 上面那种方法不可以
  v31 = *int //指向存储 int 类型值的指针类型 
)
```



### 运算符

#### 位运算符

位运算符以二进制的方式对数值进行运算（效率更高），和 PHP 类似，Go 语言支持以下这几种位运算符：



|  运算符  |   含义   |                             结果                             |
| :------: | :------: | :----------------------------------------------------------: |
| `x & y`  |  按位与  |                 把 x 和 y 都位 1 的位设为 1                  |
| `x | y`  |  按位或  |                  把 x 或 y 为 1 的位设为 1                   |
| `x ^ y`  | 按位异或 |            把 x 和 y 一个为 1 一个为 0 的位设为 1            |
|   `^x`   | 按位取反 | 把 x 中为 0 的位设为 1，为 1 的位设为 0，PHP 中对应的位运算符是 `~`，与 C 语言一致 |
| `x << y` |   左移   |        把 x 中的位向左移动 y 次，每次移动相当于乘以 2        |
| `x >> y` |   右移   |        把 x 中的位向右移动 y 次，每次移动相当于除以 2        |

#### 逻辑运算符

与 PHP 类似，Go 语言也支持以下逻辑运算符：

|  运算符  |        含义         |                          结果                          |
| :------: | :-----------------: | :----------------------------------------------------: |
| `x && y` | 逻辑与运算符（AND） | 如果 x 和 y 都是 true，则结果为 true，否则结果为 false |
| `x || y` | 逻辑或运算符（OR）  |  如果 x 或 y 是 true，则结果为 true，否则结果为 false  |
|   `!x`   | 逻辑非运算符（NOT） |    如果 x 为 true，则结果为 false，否则结果为 true     |



#### 运算符优先级

数字越大，优先级越高

```go
6      ^（按位取反） !
5      *  /  %  <<  >>  &  &^
4      +  -  |  ^（按位异或）
3      ==  !=  <  <=  >  >=
2      &&
1      ||
```



### 字符串操作

```go
package main
import fmt

str := "Hello world"
ch := str[0]
fmt.Printf("The first character of %s\n",ch) \\ The first character of H
str[0] = "W" //编译错误 字符串为不可变值类型
str2 := str + ",超"	//字符串拼接 + 号，与 JavaScript 类似
str_1 := str[:5] //字符串切片 获取索引5(不含)之前的子串 Hello 可理解为 截取前五个字符
str_2 := str[7:] //获取索引7(包含)之后的子串	orld 可理解为 截取从第7个索引开始到最后的子串
str_3 := str[6:9] //获取从索引6(包含)到索引9(不含)之间的子串 wor


```

### Strconv 包

要实现类似 PHP 中字符串与其他基本数据类型之间的转化，可以通过 [strconv](https://golang.org/pkg/strconv/) 这个包提供的函数来实现：

```go
v1 := "100"
v2, err := strconv.Atoi(v1)  // 将字符串转化为整型，v2 = 100

v3 := 100
v4 := strconv.Itoa(v3)   // 将整型转化为字符串, v4 = "100"

v5 := "true"
v6, err := strconv.ParseBool(v5)  // 将字符串转化为布尔型
v5 = strconv.FormatBool(v6)  // 将布尔值转化为字符串

v7 := "100"
v8, err := strconv.ParseInt(v7, 10, 64)   // 将字符串转化为整型，第二个参数表示几进制，第三个参数表示最大位数
v7 = strconv.FormatInt(v8, 10)   // 将整型转化为字符串，第二个参数表示几进制

v9, err := strconv.ParseUint(v7, 10, 64)   // 将字符串转化为无符号整型，参数含义同 ParseInt
v7 = strconv.FormatUint(v9, 10)  // 将字符串转化为无符号整型，参数含义同 FormatInt

v10 := "99.99"
v11, err := strconv.ParseFloat(v10, 64)   // 将字符串转化为浮点型，第二个参数表示精度
v10 = strconv.FormatFloat(v11, 'E', -1, 64)

q := strconv.Quote("Hello, 世界")    // 为字符串加引号
q = strconv.QuoteToASCII("Hello, 世界")  // 将字符串转化为 ASCII 编码
```

### 数组遍历

Go 中的数组只能是索引数组，与 PHP 中的数组不同

数组的遍历

* `len` 遍历 类似于 JavaScript

  ```go
  arr := [5]int{1,2,3,4,5}
  for i := 0;i < len(arr);i++{
    fmt.Println(arr[i])
  }
  ```

  

* `range` 遍历 类似于 PHP 中的 foreach

  ```go
  arr := [5]int{1,2,3,4,5}
  for k , v := range arr {
    fmt.Println(k + ":" + v)
  }
  //不获取索引，只想获取值
  for _ , v := range arr {
    fmt.Println( v)
  }
  //只获取索引，不获取值
  for k := range arr {
    fmt.Println( k)
  }
  ```




### 切片操作

元素个数

```go
var slc = make([]int,5,10)
// [0 0 0 0 0]
fmt.Println(len(slc))		//5
```

切片容量

```go
// 接上
fmt.Println(cap(slc))		//10
```

增加元素

```go
oldSlice := []int{1,2,3,4,5}
newSlice := append(oldSlice,1,2,3)
//[1 2 3 4 5 1 2 3]
newSlice2 := append(oldSlice,newSlice...) //如果第二个参数是切片的话,末尾的 ... 不能省略
//[1 2 3 4 5 1 2 3 4 5 1 2 3]
```

内容复制

```go
oneSlice := []int{1,2,3,4,5}
twoSlice := []int{7,8,9}
copy(oneSlice,twoSlice) //把twoSlice 拷贝到 oneSlice
fmt.Println(oneSlice)
// [7 8 9 4 5]

```

删除元素

```go
slice3 := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
slice3 = slice3[:len(slice3) - 5]  // 删除 slice3 尾部5个元素
slice3 = slice3[5:]  // 删除 slice3 头部 5 个元素
//append 删除
slice3 := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

slice4 := append(slice3[:0], slice3[3:]...)  // 删除开头三个元素
slice5 := append(slice3[:1], slice3[4:]...)  // 删除中间三个元素
slice6 := append(slice3[:0], slice3[:7]...)  // 删除最后三个元素

```



### 字典操作

查找元素

```go
testMap := make(map[string]int)
testMap["one"] = 1
value , ok := testMap["one"]
if ok {	//找到了
  // 处理找到的 value
  fmt.Println(value)
}
```

删除元素

```go
delete(testMap,"one")
```

遍历字典

```go
testMap := map[string]int{
  "one":1,
  "two":2,
  "three":3,
}

for k , v := range testMap {
  fmp.Println(k,v)
}
// 以上与PHP 中 foreach类似
```

键值对调

在 PHP 关联数组中，有内置数组函数 [array_flip](https://www.php.net/manual/zh/function.array-flip.php) 来实现类似的功能，在 Go 语言中，我们需要手动编写代码来实现，比如我们要对调 `testMap` 字典的键值，可以这么做：

```go
//声明一个新字典
newMap := make(map[int]string)
//遍历 testMap 键值互换存入新字典
for k ,v := range testMap {
  newMap[v] = k
}

```

字典排序

 Go 语言的字典不同于 PHP 的关联数组，是一个无序集合，如果你想要对字典进行排序，可以通过分别为字典的键和值创建切片，然后通过对切片进行排序来实现，换句话说，如果要对字典按照键进行排序，可以这么做：

```go
//创建一个键的切片
keys := make([]string, 0)
for k, _ := range testMap {
    keys = append(keys, k)
}

sort.Strings(keys)  // 对键进行排序

//遍历键切片，打印相应的testMap[k]
for _, k := range keys {
    fmt.Println(k, testMap[k])
}
// one 1
// three 3
// two 2
// 按照键名在字母表中的排序
```

如果要对字典按照值进行排序，可以这么做：

```go
// 创建值切片
values := make([]int, 0)
for _, v := range testMap {
    values = append(values, v)
}

sort.Ints(values)   // 对值进行排序

//借助键值对调字典。
for _, v := range values  {
    fmt.Println(newMap[v], v)
}
```

Go 语言内置 `sort` 包，该包提供了对集合进行排序的函数

### 指针操作

变量的本质对一块内存空间的命名，可以通过引用变量名来使用这块内存空间存储的值，而指针的含义则指向存储这些变量值的内存地址。和 PHP、Java 不同，Go 语言支持指针，如果一个变量是指针类型的，那么就可以用这个变量来存储指针类型的值：

```go
a := 100
var ptr *int //声明指针类型 表示指向存储 int 类型值的指针
ptr = &a			//类似引用传值 可获取变量 a 所在的内存地址
fmt.Println(ptr) //0xc0000a2000
fmt.Println(*ptr)//100 *ptr 获取指针指向内存地址存储的变量值 （间接引用）

```

Go 语言之所以引入指针类型，主要基于两点考虑，一个是为程序员提供操作变量对应内存数据结构的能力；另一个是为了提高程序的性能（指针可以直接指向某个变量值的内存地址，可以极大节省内存空间，操作效率也更高），这在系统编程、操作系统或者网络应用中是不容忽视的因素。

指针在 Go 语言中有两个使用场景：

* 类型指针

- 数组切片

