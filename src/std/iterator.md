# 迭代器

## 无限区间
```rust
fn main() {
    (1..).take(100).into_iter().for_each(|x| {
        if x % 3 == 0 && x % 5 == 0 {
            println!("{}", x)
        }
    })
}
```