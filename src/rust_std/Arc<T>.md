和 Box<T> 相比，Arc<T> 也是智能指针。Arc<T> 是共享所有权的智能指针。T 只有一份，不会 clone，而是增加引用计数。

### Arc<T> 可以 clone，不需要 T 也实现了 Clone。
```rust
use std::sync::Arc;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Point {
    fn new(x: i32, y: i32) -> Point {
        Point { x, y }
    }
}

fn main() {
    let p = Arc::new(Point::new(1, 2));
    println!("{:#?}", p);

    let p2 = p.clone();
    println!("{:#?}", p2);
}
```
Arc<T> clone 时，不需要 T 也实现了 Clone。每次执行 clone，引用计数增加，性能更快。

### Arc<T> 作为指针，不能使用带可变引用
// TODO

### Arc<Self>
// TODO

### Arc<dyn trait>
// TODO

### Arc<T> 和 & 的区别
Arc<T> 和不可变引用 &，本质上都是指针。不同在于：
1. Arc<T> 共享了所有权模型；而&共享借用模型，
2. & 需要遵守借用的生命周期和scope那一套规则；Arc<T> 是自己管理 scope。