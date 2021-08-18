---
marp: true
theme: gttmy-basic
---
<!-- paginate: true -->

# 5. 構造体を使用して関係のあるデータを構造化する

後藤 智哉

---

# 今回の範囲
- The Rust Programing Language 日本語版
  - `5. 構造体を使用して関係のあるデータを構造化する`
  - https://doc.rust-jp.rs/book-ja/ch05-00-structs.html

---
# 構造体

> 意味のあるグループを形成する複数の関連した値をまとめ、名前付けできる独自のデータ型

例えば、身長と体重をまとめて個人の体格とするなど。

Rustの構造体はC言語における構造体に近い。

---
# 構造体の定義

下記の書式で構造体を定義する。

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
} // Cと違ってセミコロン不要
```

本資料ではこの `User` 構造体を使って説明する。

---

# インスタンス生成

インスタンス生成は下記の書式。全てのフィールドを埋めていないとコンパイルエラーになる。

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

---

# フィールドへのアクセス

`user1.email` でフィールドにアクセスできる。変更は、構造体のインスタンス生成時に`mut` をつけていないとできない。

```rust
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

println!("username:{}", user1.username);

// mut をつけていない場合はエラーになる
user1.email = String::from("someone2@example.com")
```

「一部のフィールドのみ可変」は実現できない。全て可変 or 不変。

---

# 初期化省略記法

フィールド名と同名の変数がスコープ内にあれば、初期化を短く書ける。

```rust
fn build_user(email: String, username: String) -> User {
    User {
        // email: email と書かなくて良い
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

---

# 構造体更新記法

他の構造体インスタンスの値を部分的に使って、新たにインスタンスを生成できる。

```rust
// すでにuser1が定義されているとする
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
    // 下記を個別に書く必要がない
    // active: user1.active,
    // sign_in_count: user1.sign_in_count,
};
```

---

# タプル構造体

構造体と似た書式でtupleに名前をつけられる。フィールド名はなく、各要素へのアクセスはtupleのように行う。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
fn print_color(color: Color) {
    // tuple と同じようにアクセス可能
    println!("({},{},{})", color.0, color.1, color.2);
}

fn main() {
    // 同じ(i32, i32, i32)だが、型は別物
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
    print_color(black); // OK
    // print_color(origin); // 型が異なるのでNG
}
```

---

# ユニット様構造体

フィールドを一つも持たない構造体が定義できる。
10章で説明する **トレイト** に関連して使われる。

```rust
struct Empty {}

fn main() {
    let e = Empty {};
    println!("Hello!");
}
```

> トレイトは、Rustコンパイラに、特定の型に存在し、他の型と共有できる機能について知らせます。 トレイトを使用すると、共通の振る舞いを抽象的に定義できます。

---

# 参照をフィールドに持つ(1)

参照をフィールドに持つためには、 **ライフタイム** (10章で説明)の機能を使う必要がある。

```rust
struct User {
    username: &str,
}

fn main() {
    let user1 = User {
        username: "someusername123",
    };
}
```

---

# 参照をフィールドに持つ(2)

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:2:15
  |
2 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 | struct User<'a> {
2 |     username: &'a str,
  |
```

---

# 構造体を使ったプログラム例

「長方形の面積を求めるプログラム」を構造体を使ってリファクタリングする。
(なお、ここでは正方形も長方形に含めている。)


```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        // 長方形の面積は、{}平方ピクセルです
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

---

# 1. 長方形構造体を作る

`width` と `height` が長方形の情報である点が明確化された。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}
```

---

# 2. 面積を求める関数を修正する

長方形に対して面積を計算する点が表現されている。
rectangle はReadOnlyで十分なので、参照を受け取る。

```rust
fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

---

# 3. main を直す

```rust
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}
```

見通しが良くなりました！

---

# 構造体を簡単にprintしたい(1)

こんな感じで構造体を表示したいが、エラーになる。
```rust
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {}", rect1);
}
```

```
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
  --> src/main.rs:10:29
   |
10 |     println!("rect1 is {}", rect1);
   |                             ^^^^^ `Rectangle` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`
```

`std::fmt::Display` トレイト境界(C#/Javaにおけるinterface)がない。(i32などはデフォルトで定義済み)

---

# 構造体を簡単にprintしたい(2)
`{:?}` と `#[derive(Debug)]` を使うことで、自作の構造体を簡単にprintできる

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1);
    // -> rect1 is Rectangle { width: 30, height: 50 }
}
```

細かい出力方法の調整はできないが、デバッグに有用。

---

# 構造体を簡単にprintしたい(3)

`{:?}` の代わりに `{:#?}` を使うともう少し見やすい形で出力。

```rust
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:#?}", rect1);
    // -> rect1 is Rectangle {
    //       width: 30,
    //       height: 50
    //    }
}
```

---

# 出力される仕組み

`{:?}` はDebugという出力整形で出力するように`println!`に指示している。この文字列を生成する機能がDebugトレイト。

`[derive(Debug)]` はderive注釈という。derive注釈を付与すると、自作の構造体にトレイトを追加(導出)してくれる。デバッグ以外のトレイトは[付録C: 導出可能なトレイト](https://doc.rust-jp.rs/book-ja/appendix-03-derivable-traits.html) を参照。

---

# メソッドの定義

メソッド ... 構造体に対して定義される。Javaでいうインスタンスメソッド。

- `impl` ブロックで定義する
- `self` を第1引数に取る
  - `self`, `&self`, `&mut self`のどれでも良い
  - メソッドで所有権を奪うのはまれ

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

---

# メソッドの利用

```rust
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

---

# メソッドの呼び出し方法

`self`, `&self`, `&mut self`のどれ第1引数としてメソッドを定義しても、呼び出しは `.func()` の形式で良い。

```rust
impl Rectangle {
    // &self ではなく self を第１引数に取る
    fn zoom(self, scale: u32) -> Rectangle {
        Rectangle { width: self.width * scale, height: self.height * scale }
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("zoomed rectangle {:?}", rect1.zoom(2));
}
// -> zoomed rectangle Rectangle { width: 60, height: 100 }
```

---

# 関連関数

implブロック内の構造体を引数に **取らない** 関数を **関連関数** という。コンストラクタの定義に使われることが多い。

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
// 正方形を生成する
fn main() {
    let sq = Rectangle::square(2);
    println!("square {:?}", sq);
}
```

---

# implブロックの分割

複数のブロックに分けて書いても良い。10章で有用性がわかるとのこと。

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

---

# Appendix
- 気になること
  - メモリ上でどのように配置される?
  - `std::fmt::Debug` を実装するとどうなる?

---

# メモリ配置(1)

Cと違い、順番は定義順と一致しない。( https://docs.rust-embedded.org/book/c-tips/index.html#packed-and-aligned-types より引用 )

```rust
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}
fn main() {
    let v = Foo { x: 0, y: 0, z: 0 };
    println!("{:p} {:p} {:p}", &v.x, &v.y, &v.z);
}
// 0x7ffecb3511d0 0x7ffecb3511d4 0x7ffecb3511d2
// {:p}でメモリのアドレスを出力
// x, z, y の順番になっている!
```
---

# メモリ配置(2)

Cと同じように配置するなら、`#[repr(C)]` をつける

```rust
#[repr(C)]
struct Foo {
    x: u16,
    y: u8,
    z: u16,
}
```

また、アラインメントを指定するには `#[repr(align(n))]` をつける。(nは2の累乗)。`#[repr(packed)]` は n=1と同じ意味で、パディングしない。

Cと同様の配置かつパディングしない場合は、`#[repr(C)]`と`#[repr(packed)]`をつける

---

# `std::fmt::Debug` の実装 (1)

` #[derive(Debug)]` は `std::fmt::Debug` を自動で実装してくれるらしいが、逆に手作業で実装できないか?

下記公式ドキュメントを参考に実装してみたところ、動いた

- [Trait std::fmt::Debug](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [Type Definition std::fmt::Result](https://doc.rust-lang.org/std/fmt/type.Result.html)

---

# `std::fmt::Debug` の実装 (2)

```rust
use std::fmt;

struct Rectangle {
    width: u32,
    height: u32,
}

impl fmt::Debug for Rectangle {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({}, {})", self.width, self.height)
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1);
    // -> rect1 is (30, 50)
}
```