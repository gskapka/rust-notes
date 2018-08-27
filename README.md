# Rust Cheat-Sheet

&nbsp;

***

&nbsp;

***

### __Notes on the language__

* _Colons_ go after statements ala Solidity. Urgh.

* Variables are _immutable_ by default unless you specificy _mut_ infront of a let. 

* Non _mut_ lets may be shadowed in order to change them, but each time the _let_ keyword needs to be used, else compiler error. The docs give an example of shadowing of a way to reuse the same name but changing the type. Seems a bit redundant though I guess it keeps the namespace smaller & stops a multitude of intermediate variable? Will keep an eye out for better uses for shadowing.

* _Constants_ actually are immutable. Use _CAPS\_CASE_ naming for them. 

* _Tuples_ can take any types. Can be accessed by destructuring or like __JS__'s object's dot notation, using an index. Indices start at zero.

* _Arrays_ are _fixed-length_ always. Must be known at compile time. Cannot grow or shrink! Useful if you want data allocated on the _stack_ rather than the _heap_. Accessing a non-existing array element _will_ compile but will _panic_ at run time instead of accessing random bits of the program's memory.

* _Vectors_ are like arrays but are _not_ fixed length!

* Functions defined with _fn_. Unlike in JS when using _const_ to define a func, Rust doesn't care about definition order. 

* Fn naming uses _snake\_casing_.

* Format for printing: `println!("Curly braces are where x is interpolated: {}", x);`

* Fn parameters are declared with their types: `fn something(x: i32) { ...`, here for eg as an 32-bit integer.

* Something like `let x = (let y = 6)` won't set both `x` and `y` to 6, because `let y = 6` is a _statement_, not an _expression_ & ∴ returns nothing.

* Fn returns are defined in the fn statement using an `->` arrow. Note the lack of colon after `5`:

  ````
    fn five() -> i32 {
      5
    }
  ```
* In the following example, if a colon is added after the _expression_ inside the block, it becomes a _statement_ and so nothing is returned, meaning the fn declaration (which defines a return) is now _wrong_, and so it won't compile:

  ```
  fn add_five(x: i32) -> i32 {
    x + 5
  }
  ```
* Can use an `if` in a `let` thusly:

  ```
  let number = if condition {
    1
  } else {
    2
  };
  ```

* Loop through array with `for / in`: 
  
  ```
  fn main() {
    let arr = [1,2,3,4,5];
    for element in arr.iter() {
      println!("The value is: {}", element);
    }
  }
  ```
* There are _no_ true ternaries in Rust :( Can do something like `return if value == 5 { success } else { failure }` though. 
