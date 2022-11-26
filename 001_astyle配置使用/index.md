# Astyle 配置使用


## Astyle 工具下载

----

## Astyle 参数详解

### 括号类型

#### default brace style

默认采用大括号样式，开口大括号不变，闭合大括号将与上一行断开。

#### --style=allman / --style=bsd / --style=break / -A1

```c
int Foo(bool isBar)
{
    if (isBar)
    {
        bar();
        return 1;
    }
    else
        return 0;
}
```

#### --style=java / --style=attach / -A2

```c
int Foo(bool isBar) {
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

#### --style=kr / --style=k&r / --style=k/r / -A3

```c
int Foo(bool isBar)
{
    if (isBar) {
        bar();
        return 1;
    } else
        return 0;
}
```

#### --style=stroustrup / -A4

```c
int Foo(bool isBar)
{
    if (isBar) {
        bar();
        return 1;
    }
    else
        return 0;
}
```

## 在 Keil 中的使用



----


