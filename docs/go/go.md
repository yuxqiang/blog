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