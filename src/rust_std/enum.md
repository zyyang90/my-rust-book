# enum
Enum 可以包含多个 variant（变体），每个 variant 可以挂载各种类型的 payload。
```rust
enum Shape {
    Rectangle { width: u32, height: u32 },
    Triangle((u32, u32), (u32, u32), (u32, u32)),
    Circle { origin: (u32, u32), radius: u32 },
}
```

## 模式匹配
可以用一个 struct 直接匹配变量的值。
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

fn main() {
    let a = Person {
        name: String::from("Alice"),
        age: 30,
    };

    let Person {
        ref name,
        age,
    } = a;

    println!("{} is {} years old.", name, age);
    println!("{:?}", a);
}
```