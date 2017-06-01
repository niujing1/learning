### DataCreate

数组：

```
var array [5]int //声明一个数组，设置为0
array := [5]int{1, 2, 3, 4, 5} //声明一个包含5个int元素的数组
array[2] //取出数组下标为2 的元素

var array[2][3] //声明多维数组
array := [4][2]int{{1, 2}, {3, 4}, {5, 6}, {7, 8}}
array := [4][2]int{1:{0: 20}, 3:{1: 4}} //声明并初始化外层数组和内层数组的单个元素
foo(array) //在函数间传递数组
foo(&array) //将数组的指针传递给数组
```

切片：

```
slice := make([]int, 5) //创建一个包含5个int元素的切片
slice := make([]int, 3, 5) //创建一个长度为3，容量为5的切片
slice := []int{1, 2, 3} //创建一个长度是2容量是3的切片
//声明空切片
slice := make([]int, 3)
slice := []int{}
newSlice := slice[1:3] //基于slice创建一个长度为3-1=2的切片
newSlice := slice[1:3:5] //基于slice创建一个长度为2，容量为5-1=4的切片
newSlice := append(newSlice, 60) //使用append向slice追加元素
append会智能的处理底层数组容量的增长
迭代： for \_, value := range slice{...} //range是操作的副本，且总是从头开始迭代
slice = foo(slice) slice在函数间传递

```

映射：

```
dict := make(map[string]int) //创建一个映射，键为string，值为int
dict := map[string]string{"red":"#da1337", "orage":"#e95a22"} //创建一个键和值都是string的映射

map的键可以是任意值，可以是内置类型，也可以是结构类型，只要可以使用==做比较即可
切片、函数和包含切片的结构体由于具有引用意义，不可作为键

空映射
dict := map[[]string]int{} //使用切片作为映射的键--报错
dict := map[int][]sting{} //使用字符串切片作为值--可以

colors := map[string]string{} //创建一个空的映射，用来存储值
colors["red"] = "#da1337"

//通过声明未初始化的映射来创建一个值为nil的映射不能用来存储键值对
var colors map[string]string
colors["red"] = "#da1337" //--报错

value, exists := colors["Blue"] //判断键是否存在
if exists {
	fmt.Println(value)
}

//另一种方式，只返回值，并通过值是否为0来判断键是否存在
value := colors["Red"]
if value != "" {
	fmt.Println(value)
}

//使用range迭代映射
colors := map[string]string{
	"A": "a",
	"B": "b",
	"C": "c",
	"D": "d"
}
//迭代所有值
for key, value := range colors{
	fmt.Printf("Key is %s, Value is %s\n", key, value)
}

//从映射中删除一项
delete(colors, "A")
在函数间传递映射时，传递的是映射本身，而不是其副本

```


用户定义的类型：

```
//user在程序里定义一个用户类型
type user struct {
	name  string
	email string
	age   int
	privileged bool
}

//声明user类型的变量
var bill user

//声明user类型的变量，并初始化所有值
lisa := user{
	name:  "nj"
	email: "nj@126.com"
	age: 25
	privileged: true
}

//使用结构体字面量创建并初始化一个用户类型
user {
	name:  "nj"
	email: "nj@126.com"
	age: 25
	privileged: true
}

//或者使用短变量声明
nj := user{"nj", "nj@126.com", 25, true} //不指明字段时，顺序要和定义一致

// 声明结构体时，字段类型不局限在内置类型，也可以使用其它的用户定义类型
type admin struct {
	person user
	level string
}

//声明admin类型的变量
fred := admin{
	person : user {
		name: "nj",
		email: "nj@126.com"
		age:   25
		privileged: true
	}
	"level" : "super"
}

//另一个定义用户类型的方式是基于一个现有的类型，重定义为新的类型
type myInt int64 //把int64定义为myInt类型

```


**接口**
多态是值代码可以根据类型的具体实现采取不同行为的能力
接口定义了行为的类型，由用户定义的类型来实现

对接口值方法的调用会执行接口值存储的用户定义的类型的值对应的方法，因为任何用户定义的类型都可以实现任意接口，所以对接口的调用自然是一种多态

如果用户定义的类型实现了某个接口声明的一组方法，这个用户定义类型的值就可以赋值给这个接口，这个用户定义的类型通常叫做实体类型


//方法集定义了一组关联到给定类型的值或者指针的方法，定义方法时使用的接收者的类型决定了这个方法时关联到值还是关联到指针

接收 |  传入
-----|------
T    |   T & \*T
\*T   |  \*T


**嵌入类型**
Go允许用户扩展或者修改已有类型的行为，嵌入类型是将已有的类型直接声明在新的结构类型里，被嵌入的类型称为新的外部类型的内部类型
内部类型对于外部类型总是可访问的

```
admin := admin{
	user: user{
		name: "bill",
		email: "bill@qq.com"
	},
	level: "super",
}
//假如user有一个notify方法可以调用
admin.user.notify()
或者
admin.notify()都可以访问的到

//假如admin实现了内部方法的同名方法，它会调用admin的实现，覆盖内部方法的实现
//当一个标识符以小写字母开头的时候，就是对外不可见的，只能在本包内调用
//以大写字母开头的时候就是公开的，可以外部调用
//若一个未公开的结构体包含两个公开的字段，可以通过它嵌入的结构体直接调用公开的字段

```

###通道 channel

```
unbuffered := make(chan int) //无缓冲的整型通道
buffered := make(chan string, 10) //有缓冲的字符串通道，第二个参数指定通道缓冲区的大小
buffered <- "Gopher" //向通道发送一个字符串
value := <-buffered //从通道接收一个值

无缓冲的通道是指在接收前没有能力保存任何值的通道，这种通道要求发送和接收的goroutine同时准备好，才能进行发送和接收操作，若一方未准备好，就会导致阻塞等待，双方要共存，无法单独存在

有缓冲的通道是一种在被接收前能存储一个或者多个值的通道，它不要求goroutine之间同时完成发送和接收，只有在通道中没有可用缓存区容纳被发送的值时，发送动作才会被阻塞，只有在通道无要接收的值时，接收才会被阻塞

### 常用包
runner包用于展示如何使用通道来监视程序的执行时间，运行时间太长的程序也可以使用runner包来终止，当开发需要调度后台处理任务的程序的时候，这种模式会很有用

