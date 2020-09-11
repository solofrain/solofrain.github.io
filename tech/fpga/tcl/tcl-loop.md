Tcl 循环
==


# 1. for

```tcl
for {set x 0} {$x<5} {incr x} {
    puts $x;
}
```

# 2. foreach

- 对单个列表中的元素进行一个一个的遍历：

    ```tcl
    foreach var {a b c d e f} {
        puts $var
     }
    ```

    Output:

    ```bash
    a
    b
    c
    d
    e
    f
    ```

- 对列表进行多个元素一起赋值，这时varname是一个n个元素列表结构，每次遍历list列表中的n个元素赋值给以varname列表元素为名称的变量。

    ```tcl
    foreach {var1 var2 var3} {a b c d e f} {
        puts "$var1 $var2 $var3"
    }
    ```

    Output:

    ```bash
    a b c
    d e f
    ```

- 遍历多个列表中的元素，这里举例以varname为单个元素为例：
    
    ```tcl
    foreach var1 {a b c} var2 {d e f} {
        puts "$var1 $var2"
    }
    ```

    Output:

    ```bash
    a d
    b e
    c f
    ```

    如果元素不足那么会以空来补充：

    ```tcl
    foreach var1 {a b c} var2 {d e} {
        puts "$var1 $var2"
    }
    ```

    Output:

    ```bash
    a d
    b e
    c
    ```

# 3. while

```tcl
set x 0
while {$x<5} {
    puts $x;
    incr x;
}
```