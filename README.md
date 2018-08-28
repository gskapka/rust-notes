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
    height: u32
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
