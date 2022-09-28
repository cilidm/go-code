# 2. 练习

> mod.rs
```rust
pub mod user_model;
```

> user.rs
```rust
#[derive(Debug)]
pub struct UserModel {
    pub user_id: i32,
    pub user_name: String,
    pub user_age: u8,
    pub user_tags: [&'static str;5]
}

pub fn new_user_model()->UserModel{
    UserModel{
        user_id: 0,
        user_name: String::new(),
        user_age: 0,
        user_tags: ["";5]
    }
}
```

> main.rs
```rust
mod models;
use models::user_model;

fn set_user(u:&mut user_model::UserModel){
    u.user_id = 101;
    u.user_name = String::from("abc");
    u.user_age = 19;
    u.user_tags = ["java","php","js","golang","rust"];
}

fn main() {
    let mut user = models::user_model::new_user_model();
    set_user(&mut user);
    println!("{:#?}",user);
}
```