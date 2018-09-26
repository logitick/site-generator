---
title: "A Close Look at Golang's defer"
date: 2018-09-22T09:44:13+08:00
draft: true
tags: ["go", "golang"]
---
I've never come across anything like Go's `defer` in any of the languages I've 
worked on before. As far as I know there isn't an equivalent 
of it in PHP and the closest we have in Java is 
[try-with-resources](https://stackoverflow.com/questions/29788307/what-is-the-defer-equivalent-for-java).

If you've been coding in Go you've probably used the 
`defer` statement plenty of times.
<!--more-->

It makes reading a file super short:
```
f, err := os.Open("myfile")
defer f.Close()
if err != nil {
    panic(err)
}
...
```

## What does it do?
>A defer statement defers the execution of a function 
>until the surrounding function returns.  
>- https://tour.golang.org/flowcontrol/12


A straight and simple definition from [A Tour of Go](https://tour.golang.org/flowcontrol/12).
Any function or method call can be deferred by prepending the
keyword. It is usually used for clean-ups like closing a file
or a connection.

Any number of functions can be deferred which will then be executed 
in a **last in first out** order when the function that contains
 the `defer` statement returns.

**In action**
```
func greet() {
    defer fmt.Println("Good")
    defer fmt.Println("day")
    defer fmt.Println("beautiful")
    defer fmt.Println("person")
    fmt.Println("How are you today?") // not deferred
}
```

**Output**
```
How are you today?
person
beautiful
day
Good
```

It is also good to keep in mind that when deferring a function, it's arguments are evaluated immediately.
```
func doStuff() {
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
}
```


Calling `doStuff()` would run something similar to:
```
// for loop starts
deferStack.Push(func(){fmt.Println(0)})  // arguments are evaluated
deferStack.Push(func(){fmt.Println(1)})  
deferStack.Push(func(){fmt.Println(2)})
deferStack.Push(func(){fmt.Println(3)})
deferStack.Push(func(){fmt.Println(4)})
// for loop exits
```


**Output:**
```
4
3
2
1
0
```


> <sup>*</sup> 