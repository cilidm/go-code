# 3. 泛型
> mod.rs
```rust
pub mod user_score;
```

> user_score.rs
```rust
#[derive(Debug)]
pub struct UserScore<A,B>{
    pub user_id: A,
    pub score: B,
    pub comment: &'static str
}

pub fn new_user_score_a()->UserScore<i32,i32>{
    UserScore { user_id: 0, score: 0, comment: "基础用户组" }
}

pub fn new_user_score_b()->UserScore<&'static str,f32>{
    UserScore { user_id: "", score: 0.0, comment: "超级用户组" }
}
```

> main.rs
```rust
use models::user_score;

mod models;

fn main() {
    let mut user_score = models::user_score::new_user_score_a();
    user_score.user_id = 1;
    user_score.score = 65;
    println!("{:#?}",user_score);

    let mut user_score2 = models::user_score::new_user_score_b();
    user_score2.user_id = "admin";
    user_score2.score = 65.0;
    println!("{:#?}",user_score2)
}
```

# 增加泛型方法使用
> user_score.rs
```rust
#[derive(Debug)]
pub struct UserScore<A, B> {
    pub user_id: A,
    pub score: B,
    pub comment: &'static str,
}

impl<A,B> UserScore<A,B> {
    pub fn get_user_type(&self) -> &'static str {
        &self.comment
    }

    pub fn get_user_id(&self) -> &A {
        &self.user_id
    }
}


pub fn new_user_score_a() -> UserScore<i32, i32> {
    UserScore {
        user_id: 0,
        score: 0,
        comment: "基础用户组",
    }
}

pub fn new_user_score_b() -> UserScore<&'static str, f32> {
    UserScore {
        user_id: "",
        score: 0.0,
        comment: "超级用户组",
    }
}
```

> main.rs
```rust
pub mod user;

fn main() {
    let mut user_score = user::user_score::new_user_score_a();
    user_score.user_id = 1;
    user_score.score = 65;
    println!("{:#?}",user_score.get_user_type());
    println!("{:#?}",user_score.get_user_id());

    let mut user_score2 = user::user_score::new_user_score_b();
    user_score2.user_id = "admin";
    user_score2.score = 65.0;
    println!("{:#?}",user_score2.get_user_type());
    println!("{:#?}",user_score2.get_user_id());
}
```