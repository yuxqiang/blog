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
