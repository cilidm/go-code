# 1. 结构体的使用和输出+数组、元组

```rust
#[derive(Debug)]
struct User {
    name: String,
    age: u8,
}

impl User {
    fn version(&self) {
        println!("1.0")
    }
    fn to_string(&self) -> String {
        return String::from(format!("{},{}", &self.name, &self.age));
    }
}

fn main() {
    println!("----------------结构体的使用和输出----------------");
    let me = User {
        name: String::from("shenyi"),
        age: 19,
    };
    me.version();
    println!("{}", me.to_string());
    println!("{:#?}", me);

    println!("----------------数组----------------");
    let tags = ["java", "php", "rust", "go"];
    println!("{}", tags.len());
    for i in 0..tags.len() {
        // 循环
        println!("{}", tags[i]);
    }
    for item in tags.iter() {
        // 迭代器
        println!("{}", item)
    }

    // 另一种初始化方式
    let tag: [&str; 10] = [""; 10];
    println!("{}", tag.len());

    let mut tag_mut: [u8; 10] = [0; 10];
    for i in 0..tag_mut.len() {
        tag_mut[i] = (i + 1) as u8;
    }
    println!("{:#?}", tag_mut);

    println!("----------------元组----------------");
    let my:(&str,u8) = ("abc",18);
    println!("{:#?}",my);
    println!("{},{}",my.0,my.1);

    let tags:[u8;10] = [0;10];
    for (i,item) in tags.iter().enumerate(){
        println!("index:{},value:{}",i,item)
    }
}

```
