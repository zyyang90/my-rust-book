# 泛型
泛型，类型参数，表示多种类型的方法。

## 在 struct 上使用泛型
```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let a = Point::<i32> {x: 1, y: 1};
    println!("x: {}, y: {}", a.x, a.y);
    
    let b = Point::<f32> {x: 2.2, y: 2.22};
    println!("x: {}, y: {}", b.x, b.y);
}
```
1. Point<T>表示声明 struct Point 接受一个泛型 T， T 是一个占位类型；
2. Point::<i32>表示，指定了泛型使用了i32类型。

#### 声明多个泛型
```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let a = Point::<i32, i32> {x: 1, y: 11};
    let b = Point::<f32, f32> {x: 2.2, y: 2.22};
    let c = Point::<i32, f32> {x: 3, y: 3.33};
}
```

## 在 enum 上使用泛型
```
Option<T> {
    Some(T),
    None,
}

Result<T, E>{
    Ok(T),
    Err(E),
}
```
`Option<T>` 和 `Result<T, E>` 是标准库中的两个泛型枚举类型。

## 在 function 上使用泛型
在函数中使用泛型，下面的例子中，print 函数要求传入的参数是一个T的引用，T 实现了 Debug 接口。
```rust
fn print<T: std::fmt::Debug>(o: &T) {
    println!("{:#?}", o);
}

fn main() {
    let p = Point { x: 1, y: 2 };
    print(&p);
}

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}
```

### 在 method 中使用泛型
和 function 一样，都是在函数的定义中使用了类型参数（泛型）
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T: std::fmt::Display> Point<T> {
    fn play(&self) {
        println!("x: {}, y: {}", self.x, self.y);
    }
}

fn main() {
    let a = Point::<i32> {x: 1, y: 11};
    a.play();
    
    let b = Point::<f32> {x: 2.2, y: 2.22};
    b.play();
    b.play();
}
```

### struct 和 method 上都使用泛型
下面这个例子，X3,Y3 和 X1, Y1在定义时用了2个不同的字符串，但本质是一种。
```rust
#[derive(Debug)]
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X3, Y3> Point<X3, Y3> {
    fn mix<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X3, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2.22 };
    println!("{:#?}", p1);

    let p2 = Point { x: 'C', y: "123" };
    println!("{:#?}", p2);

    let p3 = p1.mix(p2);
    println!("{:#?}", p3);
}
```
