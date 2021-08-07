## Rust Memory Ownership
本篇將我閱讀 [Rust Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) 與 [Lifetime](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html) 的理解寫出來

Rust 是最近非常熱門的一個程式語言，主要原因是它擁有許多現代高階語言會有的語言特性，同時又能寫出效能逼近 C 語言的程式，其中的主要原因便在於 Rust 獨特的記憶體管理方式，Rust 不使用 GC 或是 Reference Counting 的方式控管記憶體，同時又不會像 C 語言那樣手動控管記憶體容易出錯，讀完 Ownership 與 Lifetime 之後我的理解是：Rust 設計了一些規則讓開發者遵守從而達到在 compile time 就能知道所有記憶體需要被調用與釋放的時機。
### Stack 與 Heap
與其他語言一樣，Rust 中的變數分為存放在 stack 上與 heap 上，基本上在 compile time 就能知道佔用記憶體大小的 variable 會存放在 stack 上，像 ```String``` 這種 data type 由於使用的記憶體用量會髓著程式邏輯動態增減，無法在 compile 時就知道佔用記憶體大小，則會在 Stack 上存放指向實際上儲存資料的 Heap 的指標。在其他程式語言如 Java，工程師其實不需要知道資料是存在 Stack 或 Heap (大部分狀況啦...)，然而 Rust 開發者是必須要知道的，因為這會實際上影響到在程式碼該如何操控變數。
### Ownership 與 Scope
每個在 Rust 程式碼中的 value 都有一個 **Owner**，而這個 Owner 就是指向它的 variable，而**一個 value 同時只會有一個 Owner**，variable 會屬於一個 **Scope** 內，當這個 Scope 結束後 variable 的 value 會被釋放掉。以下面程式碼為例，```"hello"``` 這個 value 的 owner 是 variable ```s```，上下括號測是這個 variable 存在的 Scope。
```rust
{
  let s = "hello";
}
```
### Copy 與 Ownership 轉移
在上面有說到變數存在 Stack 與 Heap 上會影響到在程式碼上如何操控變數：
```rust
let x = 5;
let y = x;
```
上述的程式碼中，由於 5 是存在 Stack 上的，第二行的 ```y = x``` 會直接將 x 所指到的 value 也就是 5 copy 一份給 y。
```rust
let s1 = String::from("hello");
let s2 = s1;
```
上面的程式碼，由於 String type 是存在 Heap 中，```s2``` 只有 copy ```s1``` 在 Stack 上的指標與相關資訊並指到同一個 Heap，讀到這裡可能還覺得沒什麼新東西跟 Java 差不多，但現在問題來了，上面說一個 value 只會有一個 Owner，那將 ```s2``` 指向 ```s1``` 之後這個 String value 的 Owner 是誰呢？一個 value 同時擁有兩個 Owner 會造成無法在 compile time 判斷 value 何時要被釋放，Rust 的解法是：當你使用 ```s2``` 指向 ```s1``` 時，```s1``` 已經不能再被使用了，從這行以後 ```s2``` 是 Owner。
```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{}, world!", s1);
```
上面這段程式碼會無法 compile，因為在 ```s1``` 已經把 Ownership 轉移給 ```s2``` 之後又使用到了已經無效的 ```s1``` 變數。

當然 String 這種 data type 還是可以使用 ```clone``` 去做 deep copy：
```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);
```
在上述程式碼中， `s1.clone()` 直接將 heap 中的資料也複製了一份並讓 `s2` 指過去，如此就兩個不同的 value 也各自有不同 Owner 故在第三行能同時使用兩個變數。
### Function 的 Ownership 轉移
在呼叫 function 時也會發生 Ownership 轉移：
```rust
fn main () {
  let x = 5;
  makes_copy(x);
  println!("{}", x);
  let s = String::from("hello");
  takes_ownership(s);
  println!("{}", s);//compile error
}

fn makes_copy(integer: i32) {
  println!("{}", integer);
}
fn takes_ownership(string: String) {
  println!("{}", string);
}
```
上述程式碼中其實慨念上跟前一部份提到的差不多，同樣在呼叫 `makes_copy` 時因為 value 存在 stack 上，會直接 copy 一份給 function 內部使用不影響原本 `x` 的 Ownership，所以在呼叫完之後還是能繼續使用，然而呼叫 `takes_ownership` 時 Ownership 就轉移給 function 內部的 `string` 了，所以第七行程式碼會導致 compile error，因為 Ownership 已經被轉移了。
然後 function 也可以透過 return value 將 Ownership 轉移給呼叫方：
```rust
fn main() {
  let s1 = String::from("hello");
  let s2 = takes_and_give_back(s1);
  println!("{}", s2);
}
fn takes_and_give_back(string: String) -> String {
  string //rust 能省略 return 直接將最後 expression 作為 return value
}
```
如此就能在呼叫 function 後繼續操控原本字串。
### References 與 Borrowing
如果呼叫 function 都要上述方式將值回傳說真的也太麻煩了，在 Rust 提供了以下寫法解決這個問題：
```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```
上述 function `calculate_length` 會在呼叫時回傳字串長度，可以看到在呼叫 function 時傳入的是 `&s1` 以及定義在 function 上的型別是 `&String`，其中 `&s1` 代表的是創造一個 **Reference** 指標指向 `s1` 指標，並沒有轉移 Ownership，`&String` 則指定傳進來的要是 Reference type，所以在 calculate_length 執行完畢時，因為 s 是 Reference 指標，並沒有任何記憶體被釋放掉。在原 Scope 也能繼續使用 `s1`。
### Mutable References
若嘗試在 function 內操作 Reference 指標更改其的值：
```rust
fn main() {
  let s = String::from("hello");
  change(&s);
}

fn change(some_string: &String) {
  some_string.push_str(", world");
}
```
上述的程式碼會 compile 失敗，因為試圖對一個 Reference 指標的值進行修改。
```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```
上面的程式碼可以成功 compile 跟運作，首先在宣告 s 變數時加了 `mut` 宣告這是一個可變變數，命且在原本只使用 `＆` 的地方改使用 `&mut` 宣告這是 mutable reference，這樣便可在 function 中對於外部 Reference 進行的值進行更改。
但 mutable reference 指標是有限制的，那就是不能同時將兩個 mutable reference 指標指向同一個指標：
```rust
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s;
println!("{}, {}", r1, r2);
```
上述程式碼會 compile 失敗，因為 `r1` 與 `r2` 都嘗試創造 mutable reference 指向 `s`，基本上這個限制是為了防止 data race，`r1` 與 `r2` 若都能對 s 的值進行修改則程式很容易出現為定義的行為。
但要注意的是這個限制是不能"同時"兩個 mutable reference 指標指向同一個指標，所以透過 Scope 的控制，以下程式碼也是合法的：
```rust
let mut s = String::from("hello");
{
  let r1 = &mut s;
} 
let r2 = &mut s;
```
因為將 `r1` 額外用括號所在另一個比較小的 Scope，Scope 結束後 `r1` 也就消失了，所以此時可以再宣告 `r2` 不會發生問題。
同一時間擁有多個 Reference 指在同一個變數是合法的：
```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);
```
然而在同一時間，若已經有普通的 Reference 指在一個變數上，就不能有 mutable reference 指在同樣的變數上：
```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &mut s;
```
上述的程式碼會 compile 失敗，因為 `r1` 已經創造了一個 Reference 指向 `s`，`r2` 沒辦法再對 `s` 創造 mutable refecence。然而還是要記住限制是"同時"，所以下程式碼是合法的：
```rust
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2);
let r3 = &mut s;
println!("{}", r3);
```
以上程式碼因為 `r1` 與 `r2` 在呼叫 `println` 已經把 Reference 的 Ownership 轉移給 `println` 了所以已經離開 Scope，故能合法宣告 `r3`。
## Dangling References
Rust 也能在 compile time 防止創造出 **dangling pointer**，即是防止繼續使用指向的記憶體已經被釋放掉的指標：
```rust
fn main() {
  let reference_to_nothing = dangle();
}

fn dangle() -> &String {
  let s = String::from("hello");
  &s
}
```
上述程式碼會 compile 失敗，因為 fuction `dangle` 嘗試回傳的 Reference 所指向的變數在這個 Scope 就會被釋放掉。
## Slice Type
內建的 String 與其他一些 Collection type 提供了 slice 的 API，以 String 為例，能創在一個 Reference 指向 String 區段的 substring：
```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];
```
其中 `0..5` 這是 Range 的 literal，所以基本上 API 的形式是這樣 `&s[Range]`，然後因為 String 這個 type 不是以 `[char]` 這種 type 來表示，所以 String 的 slice 有另外一個 type：`&str`：
```rust
fn first_word(s: &String) -> &str {
  let bytes = s.as_bytes();
  for (i, &item) in bytes.iter().enumerate() {
    if item == b' ' {
      return &s[0..i];
    }
  }
  &s[..]
}
```
有趣的是像 `"hello world"` 這個 literal 其實就是 `&str` type，所以如果設計 Rust 的 function 直接使用 `&str` 來當作 parameter type 使用上會更加方便：
```rust
fn main() {
  let my_string = String::from("hello world");
  let word = first_word(&my_string[..]);
  let my_string_literal = "hello world";
  let word = first_word(&my_string_literal[..]);
  let word = first_word(my_string_literal);
}

fn first_word(s: &str) -> &str {
//...
```
而使用 Slice type 來指向變數因為也是創造一個 Reference，所以也會防止變數被改變：
```rust
fn main() {
  let mut s = String::from("hello world");
  let word = first_word(&s);
  s.clear();
  println!("the first word is: {}", word);
}
```
上述程式碼不會 compile 成功，這也保證了使用 Slice type 的安全性。
其他內建的 Collection type 也都支援 Slice API，而因為本身就是 Collection type，所以像是 `[i32]` 的 slice type 就是很直覺的像這樣表示：`&[i32]`。
### Lifetime
**Lifetime** 指的其實就是一個 variable 能夠存活的 Scope，大部分的時候 Lifetime 都有辦法靠 Scope 自動偵測，像是到目前為止提到過的程式碼，Lifetime 都是有辦法自動偵測的，但還是會出現無法自動偵測的狀況，就需要接下來提到的 Lifetime Annotation Syntax 來解決。
### Lifetime Annotation Syntax
看看以下的程式碼邏輯：
```rust
fn main() {
  let string1 = String::from("abcd");
  let string2 = "xyz";
  let result = longest(string1.as_str(), string2);
  println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```
以上的程式碼 compile 會失敗，主要的原因在 `longest` 這個 function 回傳的 `&str` type，這個 `&str` 是一個 reference type，然而 compiler 無法確認它的 Lifetime 是綁定在哪一個 variable 上，事實上連我們也沒有辦法確認，因為回傳哪個結果是在 runtime 依據 `if` 的結果來決定的。
官方提供了一個 annotation syntax 來解決這個問題：
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```
其中 `<'a>` 很像一般的 Generic type 定義語法，差別在於內部一定要使用 (`'`) 開頭，並且通常是使用非常短的全小寫單字來定義的，`<'a>` 定義了一個 Lifetime 並且同時加到了傳進來的兩個 parameter 以及 return type 上，這表示回傳的 Reference 的 Lifetime 必須小於等於傳入的兩個 parameter 的 Lifetime，如此 compiler 就有辦法判斷 Lifetime 了：
```rust
fn main() {
  let string1 = String::from("long string is long");
  {
    let string2 = String::from("xyz");
    let result = longest(string1.as_str(), string2.as_str());
    println!("The longest string is {}", result);
  }
}
```
以上程式碼能 compile 成功，因為回傳的 Reference 的 Lifetime 與較小的 `string2` 一樣在括號內，所以 `println` 能合法使用 `result`，反之：
```rust
fn main() {
  let string1 = String::from("long string is long");
  let result;
  {
    let string2 = String::from("xyz");
    result = longest(string1.as_str(), string2.as_str());
  }
  println!("The longest string is {}", result);
}
```
則會 compile 失敗，因為已經過了 `string2` 所在的 Scope 仍嘗試使用 result。
在使用 Lifetime annotation 的時候也能注意只需要標注到你需要使用的 Lifetime 就好，譬如：
```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
  x
}
```
上述程式碼就沒有為 `y` 標上 annotation，因為 function 直接回傳 `x`，跟 `y` 完全沒有關係。
Lifetime annotation 也會強制回傳的 Reference 一定要對應到指定的 Lifetime：
```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
  let result = String::from("really long string");
  result.as_str()
}
```
上述程式碼會 compile 失敗，因為回傳的完全是一個跟 `x` 與 `y` 毫無相干的 Reference。
同樣的 Annotation 也能用在 `struct` 上：
```rust
struct ImportantExcerpt<'a> {
  part: &'a str,
}

fn main() {
  let novel = String::from("Call me Ishmael. Some years ago...");
  let first_sentence = novel.split('.').next().expect("Could not find a '.'");
  let i = ImportantExcerpt {
    part: first_sentence,
  };
}
```
同樣的規則，`first_sentence` 的 Lifetime 大於等於 `i`，所以上述程式碼是合法的。
### Lifetime Elision
可能你有注意到其實在更之前的程式碼就有用到傳入 Reference 並且回傳 Reference 沒加 Lifetime annotation 並且 compile 也能成功，像是這個 function：
```rust
fn first_word(s: &str) -> &str {
  let bytes = s.as_bytes();
  for (i, &item) in bytes.iter().enumerate() {
    if item == b' ' {
      return &s[0..i];
    }
  }
  &s[..]
}
```
實際上這段程式碼在 Rust 1.0 以前是沒辦法 compile 的，但是官方有注意到很多人寫這種類型的 function，所以乾脆在這種只有一個 parameter 以及 return type 相同的時候自動在 compile 時幫忙補上 annotation，所以這段程式碼實際上會被轉成這樣：
```rust
fn first_word<'a>(s: &'a str) -> &'a str {
//...
```
基本上官方創造了一些規則叫做 **lifetime elision rules**，這些規則會在 compile 時偵測所有能夠套用的地方，隨著官方發現開發者還會很常用哪些 pattern，這些規則未來還會持續增加。
### The Static Lifetime
還有一個 `'static` 的關鍵字能指定 Reference 能存活於整個程式的生命週期：
```rust
let s: &'static str = "I have a static lifetime.";
```