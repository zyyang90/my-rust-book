Box<T> 是一种智能指针，可以当成是一种数据类型。

Box<T> 上实现了几种 trait：
1. Deref：解引用
2. Drop：释放
3. AsRef<T>：引用，建议使用 &Box<T>，而不是 Box<T>.as_ref()

Box<T> clone 时，需要 T 也实现了 Clone
```rust
#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }

    fn play(&self) {
        println!("{:#?}", self);
    }
}


fn main() {
    let p = Box::new(Point::new(1, 2));
    p.play();

    // Box<T> 内的 T 需要实现了 Clone 才可以 clone
    let mut p2 = p.clone();
    // *可以解引用，并更新值，需要 p2 是 mut
    *p2 = Point {
        x: 100,
        y: 200,
    };
    p2.play();
}
```

Box<T> 作为函数参数
```rust
#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }

    fn play(&self) {
        println!("{:#?}", self);
    }
}

fn foo(p: Box<Point>) {
    p.play();
}

fn main() {
    foo(Box::new(Point::new(1, 2)));
}
```

Box<T> 的引用：&Box<T>, &mut Box<T>
```rust
#[derive(Debug, Clone)]
struct Point {
x: i32,
y: i32,
}

impl Point {
fn new(x: i32, y: i32) -> Point {
Point { x, y }
}

    fn play(&self) {
        println!("{:#?}", self);
    }
}


fn main() {
let boxed = Box::new(Point::new(1, 2));

    // &Box<T>
    let p = &boxed;
    p.play();
    println!("{:#?}", p);

    let mut boxed = Box::new(Point::new(1, 2));
    // &mut Box<T>
    let p2 = &mut boxed;
    // 修改&mut Box<T>，需要 2 次解引用
    **p2 = Point {
        x: 3,
        y: 4,
    };
    p2.play();
    println!("{:#?}", p2);
}
```
上面的实例中：`boxed: Box<T>` 是一个所有权型变量，`p: &Box<T>`是一个引用型变量，二者都能调用 Point 上的方法。同时，可以对`Box<T>`做可变引用，创建`&mut Box<T>`。

Box<Self> 作为参数是 self 的变体
```rust
#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }

    fn get(&self) -> (i32, i32) {
        (self.x, self.y)
    }

    fn set(&mut self, x: i32, y: i32) {
        self.x = x;
        self.y = y;
    }

    fn play(self) {
        println!("{:#?}", self);
    }

    fn play_with_box(self: Box<Point>) {
        println!("{:#?}", self);
    }
}

fn main() {
    // 创建 Box<Point> 类型的指针
    let mut p = Box::new(Point::new(1, 2));
    
    // 调用 &self
    println!("{:?}", p.get());
    
    // 调用 &mut self
    p.set(3, 4);
    
    // 调用 self
    // p.play();
    
    // 调用 self: Box<Point>
    p.play_with_box();
}
```

Box<T> 作为 struct 的 field
```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }
}

#[derive(Debug)]
struct Triangle {
    a: Box<Point>,
    b: Box<Point>,
    c: Box<Point>,
}

fn main() {
    let s = Triangle {
        a: Box::new(Point::new(0, 0)),
        b: Box::new(Point::new(3, 0)),
        c: Box::new(Point::new(0, 4)),
    };

    println!("{:#?}", s);
}
```
`Box<T>`是所有权型的变量，当然可以作为 struct 的 field。

Box<dyn trait>
```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Self {
        Self { x, y }
    }
}

#[derive(Debug)]
struct Triangle {
    a: Box<Point>,
    b: Box<Point>,
    c: Box<Point>,
}
#[derive(Debug)]
struct Circle {
    center: Box<Point>,
    radius: i32,
}

#[derive(Debug)]
struct Rectangle {
    a: Box<Point>,
    b: Box<Point>,
}

trait Shape {
    fn area(&self) -> f64;

    fn perimeter(&self) -> f64;
}

impl Shape for Triangle {
    fn area(&self) -> f64 {
        let a = self.a.as_ref();
        let b = self.b.as_ref();
        let c = self.c.as_ref();
        let ab = ((a.x - b.x).pow(2) + (a.y - b.y).pow(2)) as f64;
        let bc = ((b.x - c.x).pow(2) + (b.y - c.y).pow(2)) as f64;
        let ca = ((c.x - a.x).pow(2) + (c.y - a.y).pow(2)) as f64;
        let s = (ab + bc + ca) / 2.0;
        (s * (s - ab) * (s - bc) * (s - ca)).sqrt()
    }

    fn perimeter(&self) -> f64 {
        let a = self.a.as_ref();
        let b = self.b.as_ref();
        let c = self.c.as_ref();
        let ab = ((a.x - b.x).pow(2) + (a.y - b.y).pow(2)) as f64;
        let bc = ((b.x - c.x).pow(2) + (b.y - c.y).pow(2)) as f64;
        let ca = ((c.x - a.x).pow(2) + (c.y - a.y).pow(2)) as f64;

        ab.sqrt() + bc.sqrt() + ca.sqrt()
    }
}

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius as f64).powi(2)
    }

    fn perimeter(&self) -> f64 {
        2.0 * std::f64::consts::PI * (self.radius as f64)
    }
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        let a = self.a.as_ref();
        let b = self.b.as_ref();
        ((a.x - b.x).abs() * (a.y - b.y).abs()) as f64
    }

    fn perimeter(&self) -> f64 {
        let a = self.a.as_ref();
        let b = self.b.as_ref();
        2.0 * ((a.x - b.x).abs() + (a.y - b.y).abs()) as f64
    }
}

fn main() {
    let shape_vec = vec![
        Box::new(Triangle {
            a: Box::new(Point::new(0, 0)),
            b: Box::new(Point::new(3, 0)),
            c: Box::new(Point::new(0, 4)),
        }) as Box<dyn Shape>,
        Box::new(Circle {
            center: Box::new(Point::new(0, 0)),
            radius: 3,
        }) as Box<dyn Shape>,
        Box::new(Rectangle {
            a: Box::new(Point::new(0, 0)),
            b: Box::new(Point::new(3, 4)),
        }) as Box<dyn Shape>,
    ];

    for shape in shape_vec.iter() {
        println!("area: {}", shape.area());
    }

    for shape in shape_vec.iter() {
        println!("perimeter: {}", shape.perimeter());
    }
}
```
