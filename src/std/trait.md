trait 是一种对某个类型参数的约束。
```rust
struct MyStruct;

trait MyTrait{}

impl MyTrait for MyStruct{}
```
上面的例子，定义了一个空 struct，一个空 trait，以及为 MyStruct 实现了 MyTrait。

***
空 struct 可以直接写成`struct MyStruct;`，那为什么空`trait`不可以写成`trait MyTrait;`,而要写成`trait MyTrait{}`呢？
***

trait 可以作用在基础类型上
类型不单可以是 struct，基础类型也可以。
```rust
trait MyTrait {
    fn play(&self) {
        println!("play happy");
    }
}

impl MyTrait for u32 {}

fn main(){
    123u32.play();
}
```

trait 内可以包含什么？
- 关联函数：分为做 4 种类型， &self，&mut self, self, Self
- 关联类型：
- 关联常量：

### 关联函数
```rust
struct Football;

trait Sport {
    fn play(&self){}
    fn play_mut(&mut self){}
    fn play_own(self);
    fn play_some() -> Self;
}

impl Sport for Football{
    fn play_own(self) {}

    fn play_some() -> Self {
        Self
    }
}

fn main(){
    let mut f = Football;
    f.play();
    f.play_mut();
    f.play_own();
    let _g = Football::play_some();
    let _g = <Football as Sport>::play_some();
}
```
其中，play_own(self)和play_some() -> Self方法使用默认实现会报错。

### 关联类型
#### 在具体实现时指定
关联类型在定义 trait 时声明，在具体实现的时候指定。
```rust
struct Football;

trait Sport {
    type Type;
    fn play(&self, t: Self::Type);
}

#[derive(Debug)]
enum SportType {
    LAND,
    WATER,
}

impl Sport for Football {
    type Type = SportType;
    
    fn play(&self, t: Self::Type) {
        println!("play happy, type: {:?}", t);
    }
}

fn main(){
    let f = Football;
    
    f.play(SportType::LAND);
}
```

#### 在 T 上在使用关联类型
```rust
fn doit<T: TraitA>(t: T::Item) {
    println!("do it")
}

trait TraitA {
    type Item;
}

struct StructA;

impl TraitA for StructA {
    type Item = String;
}

fn main() {
    doit::<StructA>("hello".to_string());
}
```
上面的示例，
- 函数 doit 需要接收一个 TraitA 类型作为参数； 
- TraitA 有关联类型 Item； 
- StructA 是一个空 struct； 
- 在 StructA 上 impl 了 TraitA； 
- doit::<StructA> 具体化成 StructA

#### 在约束上具化关联类型
```rust
trait TraitA {
    type Item;

    fn foo(&self) -> Self::Item;
}

#[derive(Debug)]
struct Foo<T: TraitA<Item=String>> {
    x: T,
}

#[derive(Debug)]
struct A;

impl TraitA for A {
    type Item = String;

    fn foo(&self) -> Self::Item {
        String::from("hello")
    }
}


fn main() {
    let f = Foo { x: A };

    println!("{}", f.x.foo());
}
```
1. TraitA 有一个关联类型 Item 和一个方法 foo，foo 可以返回一个关联类型Item。
2. struct Foo 接收泛型T，T为一个 TraitA，并且约束了Trait 的Item 是 String 类型。
3. 为 struct A 实现 TraitA，这时，Item 必须为 String 类型。

#### 对关联类型的约束
```rust
trait TraitA {
    type Item: std::fmt::Debug;

    fn foo(&self, f: Self::Item, size: usize) -> Vec<Self::Item>;
}

struct StructA;

impl TraitA for StructA {
    type Item = String;

    fn foo(&self, f: Self::Item, size: usize) -> Vec<Self::Item> {
        std::iter::repeat(f).take(size).collect()
    }
}

fn main() {
    let a = StructA;
    let v = a.foo(String::from("hello"), 3);
    println!("{:?}", v);
}
```

### 关联常量
```rust
trait MyTrait {
    const LEN: u32 = 10;
}

struct MyStruct;

impl MyTrait for MyStruct{}

fn main() {
    println!("{}", MyStruct::LEN);
    
    println!("{}", <MyStruct as MyTrait>::LEN);
}
```

### trait 中同名方法的访问
```rust
trait A {
    fn play(&self);
}

trait B {
    fn play(&self);
}

struct S;

impl A for S {
    fn play(&self) {
        println!("A");
    }
}

impl B for S {
    fn play(&self) {
        println!("B");
    }
}

impl S {
    fn play(&self) {
        println!("S");
    }
}

fn main() {
    let s = S;
    s.play();

    <S as A>::play(&s);

    <S as B>::play(&s);
}
```
`<S as A>::play(&s)`这种叫做完全限定语法。

### Trait + 泛型
Trait 带泛型 + struct 带泛型
```rust
struct Point<U> {
    x: U,
    y: U,
}

trait Add<T> {
    fn add(self, t: T) -> Self;
}

impl<T,U> Add<T> for Point<U> 
where
    U: std::fmt::Display
{
    fn add(self, t: T) -> Self{
    }
}

fn main() {
    let a = Point::<u32> {x: 1, y: 2};
    let b = Point::<u32> {x: 11, y: 22};
    let c = a.add(b);
    println!("{:?}", c);
    
    let a = Point::<i32> {x: 1, y: 1};
    let b: i32 = 10;
    let c = a.add(b);
    println!("{:?}", c);
}
```

#### trait 使用类型参数和关联类型的区别
1. 使用关联类型
```rust
struct Point {
    x: i32,
    y: i32,
}

trait Add<T> {
    type Output;

    fn add(self, rhs: T) -> Self::Output;
}

impl Add<Point> for Point {
    type Output = Point;

    fn add(self, rhs: Point) -> Point {
        Point {
            x: self.x + rhs.x,
            y: self.y + rhs.y,
        }
    }
}

impl Add<i32> for Point {
    type Output = Point;

    fn add(self, rhs: i32) -> Point {
        Point {
            x: self.x + rhs,
            y: self.y + rhs,
        }
    }
}

fn main() {
    let a = Point { x: 1, y: 1};
    let b = Point { x: 2, y: 2};
    let c = a.add(b);
    assert_eq!(3, c.x);
    assert_eq!(3, c.x);

    let a = Point { x: 1, y: 1};
    let b: i32 = 10;
    let c = a.add(b);
    assert_eq!(11, c.x);
    assert_eq!(11, c.y);
}
```

2. 使用类型参数（泛型）
```rust
struct Point {
    x: i32,
    y: i32,
}

trait Add<T, U> {
    fn add(self, other: T) -> U;
}

impl Add<Point, Point> for Point {

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

impl Add<i32, Point> for Point {

    fn add(self, other: i32) -> Point {
        Point {
            x: self.x + other,
            y: self.y + other,
        }
    }
}

fn main() {
    let a = Point { x: 1, y: 1};
    let b = Point { x: 2, y: 2};
    let c = a.add(b);
    assert_eq!(3, c.x);
    assert_eq!(3, c.x);
    
    let a = Point { x: 1, y: 1};
    let b: i32 = 10;
    let c = a.add(b);
    assert_eq!(11, c.x);
    assert_eq!(11, c.y);
}
```

### trait object
返回值为 trait object
```rust
struct S1;
struct S2;
struct S3;

trait TraitA {}

impl TraitA for S1 {}
impl TraitA for S2 {}
impl TraitA for S3 {}

fn offer(i: i32) -> Box<dyn TraitA> {
    match i {
        1 => Box::new(S1),
        2 => Box::new(S2),
        3 => Box::new(S3),
        _ => panic!(),
    }
}

fn main() {
    let a = offer(1);
    println!("{:?}", std::any::type_name_of_val(&a));

    let b = offer(2);
    println!("{:?}", std::any::type_name_of_val(&b));

    let c = offer(3);
    println!("{:?}", std::any::type_name_of_val(&c));
}
```


传参为 trait object 
```rust
struct S1;
enum S2 {
    Item,
}
struct S3;

trait TraitA {}

impl TraitA for S1 {}
impl TraitA for S2 {}
impl TraitA for S3 {}

fn offer(t: &dyn TraitA) {
    println!("{}", std::any::type_name_of_val(t));
}

fn offer_vec(t: Vec<&dyn TraitA>) {
    for i in t {
        offer(i);
    }
}

fn main() {
    offer(&S1);
    offer(&S2::Item);
    offer(&S3);

    offer_vec(vec![&S1, &S2::Item, &S3]);
}
```
上面的示例：`dyn Trait`作为入参，使得`offer`函数可以接收所有实现了`TraitA`的类型，包括：`struct`和`enum`。即：参数被当作 trait object 对待。
trait object 作为参数时，指定传递引用。即：只能是`&dyn TraitA`或者`Box<dyn Trait A>`。因为 trait object 的 size 是不确定的，只能通过引用传递。

#### 哪些 trait 可以被用作 trait object？
1. 不要再 trait 内定义构造函数，例如：new；
2. Trait 中尽量使用 &self，&mut self，不要使用self

### trait object 和泛型的对比
1. dyn trait 在编译期不会展开，dyn trait 自己就是一个类型，相当于动态代理，在运行时才会在具体对象上调用。因此，性能比在编译期直接展开一步到位要慢；
2. 静态展开会造成编译出来的体积变大，dyn trait 不会。

### 在泛型 T 上使用 trait 限制类型范围
```rust
#[derive(Debug)]
struct S1;

trait TraitA {}

impl TraitA for S1 {}

fn offer<T>(t: &T)
where
    T: TraitA + std::fmt::Debug,
{
    println!("{:?}", t);
}

fn main() {
    offer(&S1);
}
```