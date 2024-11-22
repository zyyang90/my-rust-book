# 类型的构建

## Struct 和 enum 嵌套
用 enum 和 struct 可以构建复杂的数据结构，例如下面的例子：
```rust
struct Point(u32, u32);

struct Rectangle {
    p1: Point,
    p2: Point,
}

struct Triangle {
    p1: Point,
    p2: Point,
    p3: Point,
}

struct Circle(Point, u32);

enum Shape {
    Rectangle(Rectangle),
    Triangle(Triangle),
    Circle(Circle),
}

struct Drawing {
    shapes: Vec<Shape>,
}
```

## newtype 模式
用单元组的 struct 将已有的 struct 进行包装，这种模式叫做 newtype。
```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }

    fn distance(&self, other: &Point) -> f64 {
        let x_squared = (self.x - other.x).pow(2);
        let y_squared = (self.y - other.y).pow(2);

        ((x_squared + y_squared) as f64).sqrt()
    }
}

#[derive(Debug)]
struct MyPoint(Point);

fn main() {
    let p1 = MyPoint(Point::new(0, 0));
    let p2 = MyPoint(Point::new(3, 4));
    println!("p1: {:?}", p1);
    println!("p2: {:?}", p2);
    println!("Distance: {}", p1.0.distance(&p2.0));
}
```

## 洋葱结构
使用泛型 + struct 嵌套
```rust
use std::collections::HashMap;

type AAA = HashMap<String, Vec<u8>>;
type BBB = Vec<AAA>;
type CCC = HashMap<String, BBB>;
type DDD = Vec<CCC>;

type EEE = HashMap<String, DDD>;
```

## type 关键字
type 关键字可以将遗传洋葱结构简化成一个简单的类型名称。
```rust
type EEE = HashMap<String, Vec<HashMap<String, Vec<HashMap<String, Vec<u8>>>>>>;
```