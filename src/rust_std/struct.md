# struct

## method（实例方法）
1. 使用.操作符，调用的是 Struct 实例的method。
2. self 有 3 种形式：&self，&mut self，self: Self
```rust
#[derive(Debug, Default)]
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

impl User {
    fn print(&self) {
        println!("{:#?}", self);
    }

    fn set_name(&mut self, name: &str) {
        self.username = name.to_string();
    }

    fn clean(self) {
        println!("{:#?}", self);
    }
}

fn main() {
    let mut u1 = User {
        username: String::from("someone"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
        active: true,
    };

    u1.print();

    u1.set_name("abc");

    u1.print();

    u1.clean();
}
```

## function（关联函数）
不使用 self 的函数，可以看成是和 struct 关联的静态方法，例如：
```rust
#[derive(Debug, Default)]
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

impl User {
    fn print(user: &User) {
        println!("{:#?}", user);
    }
}

fn main() {
    let mut u1 = User {
        username: String::from("someone"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
        active: true,
    };

    User::print(&u1);
}
```
### 构造函数
不需要写一个 new 函数，有的写 from_xxx。

### Default
```rust
// 声明 Default
#[derive(Debug, Default)]
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    // 需要指定 u1 的类型
    let u1: User = Default::default();
    println!("u1: {:#?}", u1);
    
    // 显示调用 User 的 default 方法
    let u2 = User::default();
    println!("u2: {:#?}", u2);
}
```