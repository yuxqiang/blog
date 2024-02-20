# swift学习。
## 基本语法
### swift变量敞亮
``` Swift
//
//  main.swift
//  studyswift
//
//  Created by yu qiang on 2024/2/15.
//

import Foundation
print("Hello, World!")
var name="我是遇强"
print(name)

 ```
### swift常量
``` Swift
//
//  main.swift
//  studyswift
//
//  Created by yu qiang on 2024/2/15.
//

import Foundation
print("Hello, World!")
var name="我是遇强"
print(name)
let pi = 3.14
print(pi)

 ```
### swift的类型
``` Swift
import Foundation
print("Hello, World!")
var name="我是遇强"
print(name)
let pi = 3.14
print(pi)

var d = true

var a = 30

var b = "1111"

var c = 30.0

print(type(of: a))
print(type(of: b))
print(type(of: c))
print(type(of: d))
 ```
### swift的集合set
``` Swift
var s:Set<Int> = [1,2,3,4,4,5]
var s1 = Set([1,2,3,4,4,5])
print(s)
print(s1)

//集合的操作
import Foundation
var s:Set<Int> = [1,2,3,4,4,5]
var s1 = Set([1,2,3,4,4,5])
print(s1.count)
print(s1.isEmpty)
var (c,b) = s1.insert(10)//插入
s1.update(with: 11)//插入
s1.remove(11)//删除
var s2=s.union(s1) //+
print(s.subtracting(s1))//-
print(s.intersection(s1))//交
print(s2)
print(c,b)
print(s)
print(s1)
for item in s1{
    print(item)
}

for (k,v) in s1.enumerated(){
    print(k,v)
}

s1.forEach{print($0)}


//集合的运算

 ```
### swift的集合字典
``` Swift
var a:Dictionary<Int,String>=[:] //空字典
var b:Dictionary<Int,String>=[1:"222"] //空字典
print(type(of: a))
print(b[1].self)
print(b)

for (k,v) in b{
    print(k,v)
}

for k in b.keys{
    print(k)
}

for v in b.values{
    print(v)
}

b.forEach{k1,v1 in
    print(k1,v1)}

//集合的运算

 ```

### swift的区间
``` Swift
for index in 3...10{
print(index)
}

var a = 1...3

var b = 1..<3

var c = ...3

var d = ..<3

var e = 1...

var f = "a"..."z"

var g = 0.0...10.0

for index in a{
print(index)
}
 ```
### swift的元组
``` Swift
var a = (10,20)
print(type(of: a))

var b:(Int,Int) = (20,30)
print(type(of: b))
var c = (x:20,y:30)
print(c.x)

 ```