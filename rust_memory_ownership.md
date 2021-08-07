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