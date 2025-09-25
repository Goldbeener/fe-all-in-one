## 所有权
Rust 要求
1. 一个值在**任一时刻**只能有一个**所有者**
2. 引用必须遵循 借用规则
	1. 同时可以有多个不可变引用 `&T`
	2. 或一个可变引用 `&mut T`
	3. 不能混用，不能同时存在可变借用和不可变借用
3. 所有权移动，变量就不可用

数据竞争
悬垂引用

```rust
let guess = String::new("hello")

fn borrow(s: &String) {
 println!("{}", s);
}
```

关键点：
声明一个变量时，
在内存开辟了一段存储空间，并限定了这个变量对此的所有权

> 通俗理解就是只能在当前作用域内使用

当需要跨作用域使用时，可以有两种方式
	1. 借用 borrow
	2. 转移 move

**借用场景**下，所有权并没有发生变化，还是变量持有，只是在新的作用域下创建了一个指针引用，指向变量所持有的内存， 可以读取或者修改； 当新作用域使用完成时，**引用生命周期结束**，所有权不变

> 引用/借用 允许程序中的多段代码访问同一段内存，而无需进行复制

**转移场景**下，原变量失去所有权，新作用域下的变量拥有所有权；
如果新作用域在结束时，没有return，则变量直接销毁；
如果return，则原变量重新拥有所有权

**任一时刻，一段内存只有唯一的、明确的所有者。**

当值要跨作用域时，需要考虑借用或是转移
1. 函数调用参数传递
2. 函数返回值
3. 赋值/变量绑定
4. 容器类型的插入/取出  Vec:new().push(s)
5. 闭包获取变量
6. 多线程
7. 结构体/枚举字段的使用


### 作用域
所有权规则本质是跟作用域强绑定的
**作用域就是变量有效的生命周期范围**，变量离开作用域时，它的drop方法被自动调用，资源被释放

会产生新的作用域的典型操作
1. 花括号 {} 块
2. 函数定义
3. 条件分支 if/match
4. 循环 while/for/loop
5. 闭包
6. 线程
7. 结构体/枚举的字段

这些场景下就需要考虑变量所有权问题，权衡是使用借用还是转移。

## 生命周期
1. 当函数返回引用时，需要指定生命周期参数，以保证引用指向的数据在使用期间有效
2. Rust需要保证引用**不会悬空**



## 模式匹配
match

### Option 
**表示可能为空的值**

**Some 和 None 是Option类型的两个枚举值**

如果函数返回一个Option类型返回值，
那么函数要么返回一个Some(xxx)
要么返回一个None

**match 匹配Option类型的变量时**
**需要处理Some 和 None两个分支**

### Result<T, E> 
**表示可能失败的操作**
**推荐在函数可能失败时返回Result**

Result 类型的两个枚举， `Err` 和 `Ok`

函数返回Result时，要返回Err和Ok

match 匹配Result 时
也需要处理两个分支，Err 和 Ok

### Result 和 Option的转换
```rust
Option<T>.ok_or(err)  把None转换成Err(err)
Result<T, E>.ok() 忽略错误，只保留Some(T) 或 None

let maybe_user = Some('Alice);
let result: Result<&str, &str> = maybe_user.ok_or("No user");

```

### ？运算符

`?` 运算符 自动展开Result或Option
```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("divide by zero".to_string())
    } else {
        Ok(a / b)
    }
}

let result = divide(10, 2)?; // 如果是 Err，直接返回
```


#### 组合子
combinator
避免match的重复样板代码
```rust
fn find_username(id: u32) -> Option<&'static str> {
    match id {
        1 => Some("Alice"),
        2 => Some("Bob"),
        _ => None,
    }
}

let username = find_username(1).unwrap_or("Guest");

```
对Option数据，可以执行`.unwrap_or(default)`
如果是None，使用default默认值

也可以执行`.map(f)` 如果是Some(x) 就执行函数`f(x)`

`.and_then(f)`


### 链式组合子
+ `map/map_err` 
	+ 处理Result类型数据，
	+ map接收的函数，处理Result成功的结果 
	+ map_err接收的函数， 处理result失败的结果
+ `and_then/or_else` 继续链式调用

```rust
let res = divide(10, 2).map(|x| x * 2).map_err(|e| format!("failed {}", e))
```




## 错误处理
Result<T, E> 函数式的Either

## crate & trait
crate 板条箱
rust中的编译单元，
每一个crate都会被编译成一个库或可执行文件

> **类似于js生态中的npm包**

每个crate有自己的模块树， mod/pub

使用`use`把crate下的某个模块路径导入当前作用域，方便直接使用短名，而不是从crate开始调用



trait 特征，行为、接口
crate包里面提供的 能力，类似于interface或者常量等

`：：`
**关联函数**
绑定在类型本身，类似静态函数，不依赖具体实例实现

`.`调用，
**实例函数**
基于某个对象的实例调用，第一个参数隐式传递`self`

一个外部`crate`，只要在`Cargo.toml`中声明依赖之后, 就可以在rust项目任意位置通过`<crateName>::xxx` 使用

一个crate下，有多个trait， 使用trait时，可以有两种方式
1. `crate：：trait.fn()`
2. 或者先use，后面直接使用 `traitName.fn()`



## 并发与多线程


## 宏与泛型
