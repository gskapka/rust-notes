# Rust Cheat-Sheet

* Format for printing: `println!("Curly braces are where x is interpolated: {}", x);`. The `!` indicates that this is a macro.

* Fn parameters are declared with their types: `fn something(x: i32) { ...`, here for eg as an 32-bit integer.

* Something like `let x = (let y = 6)` won't set both `x` and `y` to 6, because `let y = 6` is a _statement_, not an _expression_ & ∴ returns nothing.

* Fn returns are defined in the fn statement using an `->` arrow. Note the lack of colon after `5`:

```rust
  fn five() -> i32 {
    5
  }
```

* In the following example, if a colon is added after the _expression_ inside the block, it becomes a _statement_ and so nothing is returned, meaning the fn declaration (which defines a return) is now _wrong_, and so it won't compile:

```rust
  fn add_five(x: i32) -> i32 {
    x + 5
  }
```
* Can use an `if` in a `let` thusly:

```rust
  let number = if condition {
    1
  } else {
    2
  };
```

* Loop through array with `for / in`: 
  
```rust
  fn main() {
    let arr = [1,2,3,4,5];
    for element in arr.iter() {
      println!("The value is: {}", element);
    }
  }
```
* There are _no_ true ternaries in Rust :( Can do something like `return if value == 5 { success } else { failure }` though.

&nbsp;

### __Ownership__

&nbsp;

* The concept of _ownership_ in Rust us how it approaches memory management on the stack & the heap, and the garbage collection thereof.

* Rust has a _string literal_ type whose length is known at compile time and ∴ can be kept on the stack. It also has a _String_ type, memory for which is allocated on the heap since nothing about it is known at compile time. You can create a string literal from a string thusly:
  
```rust
  let s = String::from("hello");
```

* A string made in the above manner __CAN__ be mutated:

```rust
  let mut s = String::from("Hello");
  s.push_str(" world!);
  println!("{}",s) // => `Hello world!
```

* So the `String::from`'s implementation covers the allocating of heap mem. Normally the __GC__ would sort the rest out afterwards, but we don't have one in Rust, so we're responsible. In other languages, if we screw this up we could de-allocate the variable before we're finished with it, or forget to free up the mem. afterwards and thus have a leak. In Rust however, the variable's mem. is freed as soon as it goes out of scope. Rust basically calls a `drop` that's similar to `C++`'s __RAII__ (_Resource Acquisition Is Initialization_). 

* So here's a gotcha. The following ends with both `x` and `y` equalling `5` & compiles just fine with both ending up on the stack:
 
```rust
  let x = 5;
  let y = x;
```

But because a _String_ is allocated memory on the heap, it consists of a pointer, a variable holding the length and a variable holding the capacity, which latter is the amount of memory the alloc. has received from the __OS__. The length is the number of bytes it's currently using. So when s2 is assigned, it receives those three values and _not_ the actual contents of the string. Obvs. it doesn't copy the actual heap data else the string manip. would be ridic. expensive. Worse: 

```rust
  let s1 = String::from("Hello");
  let s2 = s1;
  println!("{}", s2); // => Compiler error!
```

Instead of copying the memory alloc. details to s2, Rust just makes s1 invalid and reuses those values already allocd. for s2, and as such, s1 is now out of scope and doesn't exist. Compiler errors moaning that you can't use the now un-allocd mem. This is sort of like a _shallow copy_, except since Rust invalidates the prev. "copy", it calls this a _move_ instead. Note that you can `println!` the version with the two integers however, because `x` has _not_ gone out of scope, ∴ is still valid, still a known size & still on the stack.  

* Additionally, Rust will never automatically perform anything resembling a _deep copy_, and so any superficial copies will always be shallow and won't harm the runtime.

* Can make a deep copy via _clone_: 
  
```rust
  let s1 = String::from("Hello");
  let s2 = s1.clone();
  println!("s1: {}, s2: {}", s1, s2);
```

* There is no clone for the integer version a bit above since the copy doesn't throw `x` out of scope and now has a copy of both on the stack whose sizes are known and so there's no difference between this shallow copy and a deep copy, ∴ no clone necessary.

* _Copy_ in Rust is used on types like _integers_. If a type has a _copy_ trait, an older variable is still usable after assignment. Types are not _copy_-able if they rely on heap memory in any form. 

* Passing a variable to a function will move or copy it, just as assignment does:
  
```rust
    fn main() {
      let s = String::from("Hello"); // s into scope
      takes_ownership(s);            // s' value moves into function
                                     // and so would no longer be valid here
      let x = 5;                     // x comes into scope
      makes_copy(x);                 // x moves into function, but i32 is a Copy...
                                     // … so it'd be okay to still use it here.
    }

    fn takes_ownership(some_string: String) {
      println!("{}", some_string);   // Console logs => "Hello"
    }                                // some_string drops out of scope, drop is called and the mem. is freed.
    
    fn makes_copy(some_integer: i32) {
      println!("{}", some_integer);  // Console logs => "5"
    }                                // some_integer goes out of scope but nothing special happens, no drop called. 
```

* Returning values can also take ownership:

```rust
  fn main() {
    let s1 = gives_ownership();        // gives_owenership moves its return value into s1
    let s2 = String::from("Hello");    // s2 comes into scope
    let s3 = takes_and_gives_back(s2); // s2 is moved into takes_and_gives_back, which also moves its return to s3
  } // So s3 goes out of scrope & is dropped, s2 was moved so nothing happens, s1 goes out of scope and is dropped.

  fn gives_ownership() -> String {             // gives_ownership moves its return value to the function that calls it
    let some_string = String::from("Goodbye"); // some_string comes into scope
    some_string                                // some_string is returned and moves out to the calling function
  }

  fn takes_and_gives_back(a_string: String) -> String {
    a_string // a_string is returned and moves out to the calling function.
  }
```

* Note that the returns don't have a semi colon after them! Note also the pattern: assigning a value to another variable _moves_ it. When a variable that includes data on the heap goes out of scope, that value is _dropped_, unless the data has been moved to some new ownership.

* You can return multiple values with a tuple:

```rust
  fn main() {
    let s1 = String::from("Hello");
    let (s2, len) = calc_length(s1);
    println!("The length of s2: {} is {}.", s2, len);
  }

  fn calc_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length)
  }
```

* To get rid of the faff of explicitly passing the `String` around above, we can use _references_ (`&`):

```rust
  fn main() {
    let s1 = String::from("Hello");
    let len = calc_length(&s1);
    println("The length of s1: {}, is {}.", s1, len);
  }

  fn calc_length(s: &String) -> (usize) { // No more tuples!
    s.len()
  }
```
* Having a _reference_ here as a func. param is the concept of _borrowing_. Borrowed things are _immutable_ unless you explicitly declare them otherwise via: 

```rust
  fn main() {
    let mut s = String::from("Hello");
    change(&mut s);
  }

  fn change(str: &String) {
    str.push_str(" world");
  }
```

* Massive caveat: You can only have one mutable reference to a particular piece of data in a particular piece of code. This prevents data races:

```rust
  fn main() {
    let mut s = String::from("Hello");
    let r1 = &mut s;
    let r2 = &mut s; // => Compiler error! Can't borrow 's' as mutable more than once!
  }
```

* Can be fixed by creating a new scope with curlies, so now we've _ordered_ the mutations and so races cannot occur:

```rust
  fn main() {
    let mut s = String::from("Hello")'
    {
      let r1 = &mut s;
    } // r1 drops out of scope here so the following works fine.
    let r2 = &mut s;
  }
```

* Can't borrow something as _mutable_ if you've also borrowed it as _immutable_:

```rust
  let mut s = String::from("hello");
  let r1 = &s; // no problem
  let r2 = &s; // no problem
  let r3 = &mut s; // BIG PROBLEM
```

* Dangling references will result in compiler error too:

```rust
  fn main() {
    let reference_to_nothing = dangle();
  } 

  fn dangle() -> &String {
    let s = String::from("hello");
    &s // s falls out of scope after the next curly and so this pointer points to nothing!
  }
```

* Above can be fixed by returning just `s` & altering the func. sig. to `fn dangle() -> String {`. Easy!

* Slice types do not have ownership. Lets you reference a _slice_ of a contiguous sequence of elements in a collection instead of the whole thing. Use slices to get first word of a string:

```rust
  fn main() {
    fn first_word(s: &String) -> usize {
      let bytes = s.as_bytes();                   // convert string to bytes
      for(i, &item) in bytes.iter().enumerate() { // iterate over the bytes. Enumerate returns tuple of
                                                  // index & element, the if destructures it.
        if item == b' ' {                         // 'byte literal' syntax  to search for the bytes that represent ' '
          return i;
        }
      }
      s.len()
    } 
  }
```

* The issue with the above is we return an index that points to the first word in the string. What if we drop string? We're stuck. How to return the _actual first word?_ Slices!: 

```rust
  let s = String::from("Hello world");
  let hello = &s[0..5];
  let world = &s[6..11];
```
* The `[]` hold the starting index and the ending index, separated by `..`. Can drop first num. if index is 0: `[..5]`. If you want to go to end of string, you can drop the last index: `[6..]`. So `[0..len]` === `[..]`. So now lets return the actual slice of the word:

```rust
  fn main() {
    fn first_word() -> &str {
      let bytes = s.as_bytes();
      for (i, &item) in bytes.iter().enumerate() {
        it item == b' ' {
          return &s[0..i]
        }
      }
      &s[..]
    }
  }
```
* Better still, change the func. sig. to `fn first_word(s: &str) -> &str {` so now it can accept both `String` & slices of strings.

* _Structs_ work just like in Solidity. Use dot-notation to get at the stuff inside. Can define structs as mutable in order to change things in them.

* Logging a struct requires bringing the `Debug` into scope at the top via `#[derive(Debug)]` then using the `:?` operator:

```rust
  struct Rectangle {
    width: u32,
    height: u32,
  }

  impl Rectangle {
    fn area(&self) -> u32 {
      self.width * self.height
    }
  }

  fn main() {
    let rec1 = Rectangle { width: 30, height: 50};
    println!("rec1 is {:?}", rect1);
  }
```

* Notice in the above the `impl`. This defines a method on our struct type. No we can define a Rectangle per ln 288, then calc. its area via `rect.area()`. Cool beans. Note that in C++, methods are called by either the dot operator: `thing.method()` or via an arrow `thing->method()`. The former is for calling a method on the object, and the latter when calling a method on the _pointer_ to an object, and you want to dereference it first. Rust has no such thing, since it use _automatic_ referencing & de-. Rust basically adds the required`&`, `&mut` & `*` to match the sig. method.

* Methods can accept multiple parameters after the `&self` param. Can also define methods that _don't_ use self, they're just associated with the object. The ` impl` block can hold as many funcs. as you want.

* _ENUMS_ in Rust are apparently similar to the ADT's in Haskell. We'll see!

```rust
  fn main() {
    enum IpAddrKind {
      V4(String),           // So we've named a type (V4) & sepcified it's data type (String)
      // V4(u8, u8, u8, u8) // Could also do something like this to save the IP4 type!
      // V4(Ipv4Addr)       // Or here `Ipv4Addr` could itself be a struct!
      V6(String),
    }
    let home = IpAddrKind::V4(String::from("127.0.0.1"));
    let loopback  = IpAddrKind::V6(String::from("::1"));
  }

  fn route(it_type: IpKindAddr) {} // This can now accept both kinds of IP type.
```

* Enums can hold more data types more succinctly than structs: 

```
  enum Message {
    Quit,
    Move {x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
  }

  // holds the same data as: 

  struct QuitMessage; // unit struct
  struct MoveMessage {
    x: i32,
    y: i32, // Note: these trailing commas are in all the docs - are they necessary?
  }
  struct WriteMessage(String) // tuple struct
  struct ChangeColorMessage(i32,i32,i32); // tuple struct
```

* Like structs, enums can have methods defined on them: 

```
  impl Message {
    fn call(&self) { // self gets the value that we call the method on
      // ...do something...
    }
  }

  let m = Message::Write(String::from("Hello"));
  m.call();
```

* Rust does not have `null`s in it! Instead it uses type safety here called the `Option<T>` enum, which if you look closely is actually a `Maybe` monad!

```
  enum Option<T> {
    Some(T);
    None,
  }
```

* Here, `<T>` is a generic type parameter, so `Some` can hold any type. We can place it into context:

```
  let y: Option<i8> = Some(5);
  let absent_number: Option<i8> = None;
```

* Can't btw (obvs) add a `u8` to an `Option<u8>` because they are diff. types. Same as we can't add `5` to `Maybe.of(5)`. Has all the `Maybe` methods I'm used to, slightly diff. names of course (not all follow, see the [spec here](http://devdocs.io/rust/std/option/enum.option)):

```
  is_some(&self) -> bool
  is_none(&self) -> bool
  as_ref(&self) -> Option<&T>
  as_mut(&mut self) -> Option<&mut T>
  map<U, F>(self, f: F) -> Option<U>  // maps a function over the value durr :p
  unwrap_or(self, def: T) -> T        // Caution: Eagerly evaluated
  unwrap_or_else(self, f: F) -> T     // where F is a fn :: a -> T
  // ...and a shit load more!
``` 

* _Match_ -> allows pattern matching. Patterns can be made of literals, var names, wildcards etc.

```
  fn main() {
    enum Coin {
      Penny,
      Nickel,
      Dime,
      Quarter,
    }
    fn value_in_cents(coin: Coin) -> u32 {
      match coin {
        Coin::Penny => { // Can do cool lambdas!
          println!("Lucky penny!");
          1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
      }
    }
  }
```

* So to use with our `Option<T>`:

```
  fn main() {
    fn plus_one(x: OPtion<i32>) -> Option<i32> {
      match x {
        None => None,
        Some(i) => Some(i + 1),
        // Note: Patterns MUST be exhaustive, hence can use a placeholder per below:
        // _ => (),
      }
    }
    let five = Some(5);
    let six = plus_one(five);
    let none - plus_one(None);
  }
```

* `If...let` control flow. Less verbose than above to handle values that match one pattern but ignore the rest (like say if in above you'd ignored the `None` by using `_ => (), `, instead you could use the `if...let` sugar: 

```
  fn main() {
    let some_u8 = some(0u8);
    if let Some(3) = some_u8_value {
      println!("Three");
    }
  }
```

* _Modules_: can be public or priv. `mod` declares a new module. Private by default. `pub` make them public and visible outside of their namespace. `use` brings modules or the definitions inside modules into scope so it's easier to use them. Using `mod` forces a file structure. Rust will first look to `./lib.rs` for modules. Any defined there in can be brought out in their own files, which can have further mods which needs another folder which can have further files etc. Read into!

* _Vectors_: like arrays. Can define them as a generic type: `Vec<T>` or a specific one: `Vec<u32>`. Create an empty one via: `Vec::new()` or instantiate with things in via the vec macro: `let v = vec![1,2,3,4,5]`. Can push into vecs: `v.push(6)`. Two ways to get at stuff inside one:

```rust
  let vec![1,2,3,4,5]
  let a: &i32 = &v[2];            // Will panic if accessing index out of the arr.
  let b: Option<&i32> = v.get(2); // Now we're null safe thanks to Maybe!
```

* Can do a `for i in &v {}` loop to access all vars, and can make mutable if we need to change them via: `for i in &mut v {}`.

* Note that vectors can only store things of the same type. To get around this we can use an enum holding _different_ types, and store parts of _that_ in a vector, to essentially achieve storing of different types in a vector:

```rust
  enum SpreadsheetCell {
    Int (i32),
    Float(f64),
    Text(String),
  }

  let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Float(10.12),
    SpreadsheetCell::String(String::from("Yo")),
  ];
```

* _Strings_ again. Can create via `let s = String::from("String here");` or `let s = "String here".to_string();`. Can concat via pushing, using `+` or using `format`:

```rust
  let &mut s = String::from("Hello");
  s.push_str(" buddy ");

  let s1 = String::from("Yes");
  let s2 = String::from(" please!");
  let s3 = s1 + &s2 // Note: s1 now out of scope! Also note the reference to s2.

  let s4 = String::from("Thank");
  let s5 = String::from("you!");
  
  let s = format!("{} {} {}", s3,s4,s5); // => Yes please! Thank you! Note: the string interpolation is adding spaces here!
```

* String is a wrapper of `Vec<u8>` and so we can't access strings like arrays in Javascript. Can slice from strings but caution required as not all chars are two bytes long and the prog will panic at runtime if you slice mid char. Instead, it has an inbuilt `chars` method: `for c in "I think ∴ I am".chars() {}` or if you need the bytes: `for b in "I think ∴ I am".bytes() {}`

* _Hash maps_ are used to store JS style objects. Data is stored on the heap.

```rust
  fn main() {
    use std::collections::HashMap;
    let mut hMap - HashMap::new()
    hMap.insert(String::from("Key1"), 1);
    hMap.insert(String::from("Key2"), 2); 
    }
```

* If strings are created as vars to be used in the hash map, adding to the map will transfer their ownership so they'll no longer be accessible from the orig. vars. If you need to, you can insert references in the hmap instead, but you need to take care that the lifetime of the vars matches that of the map.

* Get a value out of a hmap using `hashmap.get(&key);` or `hashmap.get(String::from("key"));`

* Loop over a hash map via: `for (k,v) in &hashmap {println!("Key: {}, value: {}", k,v)}`

* The `insert` hashmap method overwrites keys. Use `hashmap.entry(&keyStr).or_insert(something_else);` if you only want to write to a key if it doesn't exist already.

```rust
  fn main() {
    use std::collections::HashMap;
    let text = "hello world wonderful world";
    let mut hmap = HashMap::new();
    for word in text.split_whitespace() {
      let count = hmap.entry(word).or_insert(0); // Uses existing value, else sets it to 0
      *count += 1;
    }
    println!("{:?}", hmap)
    // {"world": 2, "hello": 1, "wonderful": 1}
  }
```
