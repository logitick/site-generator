---
title: "A Close Look at Golang's defer"
date: 2018-09-22T09:44:13+08:00
tags: ["go", "golang"]
---
I've never come across anything like Go's `defer` in any of the languages I've 
worked on before. As far as I know there isn't an equivalent 
of it in PHP and the closest we have in Java is 
[try-with-resources](https://stackoverflow.com/questions/29788307/what-is-the-defer-equivalent-for-java).

If you've been coding in Go you've probably used the 
`defer` statement plenty of times.
<!--more-->

With `defer`, I can put the `Close` statement immediately after opening.
I love it because it helps me never to forget closing that file reader.
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
    i := 0
    defer fmt.Println(i)
    i++
}
```


**Output:**
```
0
```

How awesome is `defer`, right? There's so much more you can do with it. You can even recover from a panic. 


For me, It is just one of the things that makes Go very enjoyable to write in.