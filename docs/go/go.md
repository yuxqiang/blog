# go学习。
## 基本语法
### go打包
``` Go
 go build xxx.go
 ./main 
 nohup ./main & 
 ```

### 类型转换
``` Go
func main() {
	var name bool
	var s string = "true"
	name, err := strconv.ParseBool(s)
	name2 := strconv.FormatBool(true)
	fm.Println(name2)
	fm.Println(reflect.TypeOf(name))
	fm.Println(err)
}
```
### 变量声明
``` Go
func main() {
var name bool
var s string = "true"
var name2 ="32"
 i:=25
}
```

### point
``` Go
package main
import "fmt"
func main() {
	i := 32
	fmt.Println(&i)
	show(&i)
}
func show(x *int) {
	fmt.Println(*x) //*打印的值 不是地址了
}
```
### 函数
``` Go
package main
import (
"fmt"
)
func main() {
i := 32
j := 32
fmt.Println(showMe(i, j))

}
func showMe(x int, y int) int {
return x + y
}
``` 
``` Go
//函数当参数
package main

import "fmt"

func main() {
f := func() string {
fmt.Println("11111")
return "111"
}
//f()
fmt.Println(func2(f))
}

func func2(f func() string) string {

	return f()
}
``` 
### 控制流程
``` Go
package main

import "fmt"

func main() {

	numbers := []int{1, 2, 3, 4}

	for i, n := range numbers {
		fmt.Println(i)
		fmt.Println(n)
	}
}
``` 
### def
``` Go
package main

import "fmt"

func main() {

	numbers := []int{1, 2, 3, 4}

	defer fmt.Println("111111")
	for i, n := range numbers {
		fmt.Println(i)
		fmt.Println(n)
	}
}
``` 
### 数组切片映射
``` Go
package main

import "fmt"

func main() {
var arrays [2]string
arrays[0] = "1111"
arrays[1] = "2222"
fmt.Println(arrays[1])
}
``` 
# go基础语法

### go的切片 不同于数组 你可以在切片中添加删除元素
```Go
package main

import fm "fmt"

func main() {

	fm.Println("hello world")
	var cheeses = make([]string, 2)
	cheeses[0] = "1111"
	cheeses[1] = "222222"
	fm.Println(cheeses[0])
	fm.Println(cheeses[1])
	cheeses = append(cheeses, "111111") //添加

	fm.Println(len(cheeses))
	cheeses = append(cheeses[:2]) //删除
	fm.Println(len(cheeses))
	var smecheese = make([]string, 2)
	copy(smecheese, cheeses) //复制
	fm.Println(cheeses[1])
	fm.Println(smecheese[0])
	fm.Println(smecheese[1])
}


```
### go的map
```Go
package main

import fm "fmt"

func main() {

	var players = make(map[string]int)
	players["cook"] = 32
	players["cook1"] = 32
	players["cook2"] = 32
	players["cook3"] = 32
	fm.Println(players["cook"])
	fm.Println(players)
	delete(players, "cook") //删除元素
	fm.Println(players)

}
```
### 结构体
```Go
package main

import fm "fmt"

type Movie struct {
	Name   string
	Rating float32
}

func (m *Movie) summer() string {
	return m.Name

}
func main() {

	var m = Movie{
		Name:   "11111",
		Rating: 10,
	}
	var b = m
	b.Name = "100"
	fm.Println(m.Name)
	fm.Println(m.Rating)
	fm.Println(b.Name)
	fm.Println(b.Rating)
	fm.Printf("%+v\n", m)
	fm.Printf("%+v\n", b)

	fm.Printf("%p\n", &m)
	fm.Printf("%p\n", &b)
	fm.Println(m.summer())
}

```
### 使用缓冲区拼接字符串
```Go
package main

import (
"bytes"
"fmt"
)

func main() {

	var buffer bytes.Buffer
	for i := 0; i < 500; i++ {
		buffer.WriteString("11")
	}
	fmt.Println(buffer.String())
}
```
### 错误
```Go
package main

import (
	"fmt"
	"os"
)

func main() {

	content, err := os.ReadFile("1.txt")
	if err != nil {
		fmt.Println(err)
		//panic(err)
	}
	fmt.Printf(string(content))
}

```
### 创建错误
```Go
package main

import (
"errors"
"fmt"
)

func main() {

	err := errors.New("11111")
	fmt.Println(err)
	//content, err := os.ReadFile("1.txt")
	//if err != nil {
	//	fmt.Println(err)
	//	//panic(err)
	//}
	//fmt.Printf(string(content))
}
```


### goroutine
```Go
package main

import (
"errors"
"fmt"
)

func main() {

	err := errors.New("11111")
	fmt.Println(err)
	//content, err := os.ReadFile("1.txt")
	//if err != nil {
	//	fmt.Println(err)
	//	//panic(err)
	//}
	//fmt.Printf(string(content))
}
```
```Go
package main

import (
fm "fmt"
"time"
)

func slowSec(t int) {
fm.Println("sleep finsher")
time.Sleep(time.Second * 3)

}
func main() {
go slowSec(10)
fm.Println("11111")
}
```
```Go
package main

import (
fm "fmt"
"time"
)

func slowSec(t int) {
fm.Println("sleep finsher")
time.Sleep(time.Second * 3)

}
func main() {
go slowSec(10)
fm.Println("11111")
time.Sleep(time.Second * 4)
}
```
### 通道
```Go
package main

import (
"fmt"
"time"
)

var c chan string = make(chan string)

func slowSec1() {
time.Sleep(time.Second * 2)
c <- "msssss"
}
func main() {
go slowSec1()

	msg := <-c
	fmt.Println(msg)
}
```
### go的通道发送大小之缓冲通道
``` Go
package main

import (
"fmt"
"time"
)

var c chan string = make(chan string, 2)

func slowSec2() {
for msg := range c {
fmt.Println(msg)

	}
}
func main() {
c <- "msssss"
c <- "msssss"
//c <- "msssss"
close(c)
fmt.Println("1111111")
time.Sleep(time.Second * 2)
slowSec2()
//msg := <-c
//fmt.Println(msg)
}
```

### 将通道作为函数参数

```Go
package main

import "fmt"

//读
func channelReader(msg <-chan string) {
msg1 := msg
fmt.Println(msg1)
}
//写
func channelWrite(msg chan<- string) {
msg <- "111111"

}
//可读可写
func channelWriteReader(msg chan string) {
msg <- "111111"
fmt.Println(msg)
}
func main() {

}
```
### 使用select语句
```Go
package main

import (
"fmt"
"time"
)

func ping1(m chan string) {
time.Sleep(time.Second * 2)
m <- "1111"
}
func ping2(m chan string) {
time.Sleep(time.Second * 2)
m <- "2222"
}
func main() {
channel := make(chan string)
channel1 := make(chan string)
go ping1(channel)
go ping2(channel1)
select {
case msg := <-channel:
fmt.Println(msg)
case msg2 := <-channel1:
fmt.Println(msg2)
}
}
```
### 使用退出通道
```Go
package main

import (
"fmt"
"time"
)

func send(msg chan string) {
t := time.NewTicker(time.Second * 1)
for {
msg <- "i m send a message"
<-t.C
}

}
func main() {
message := make(chan string)
stop := make(chan bool)
go send(message)
go func() {
time.Sleep(time.Second * 2)
fmt.Println("time is up")
stop <- true
}()
for {
select {
case <-stop:
return
case msg := <-message:
fmt.Println(msg)
}

	}
}
```