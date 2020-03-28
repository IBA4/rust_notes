
# Notes on Rust programming language, from *[the book](https://doc.rust-lang.org/book/)*

I just typed what I felt like should be noted while reading *the book* simultaneously.

> Install `rustup` as a toolchain along with `cargo`.

> Don't be afraid of compiler errors. They are very helpful and give very good information.

[TOC]



## Cargo

- very good package manager for rust

```sh
cargo new <project_name> #create new project (binary)
cargo build #compile and check
cargo run #compile(if-not) and run the binary program
```
## variables

- by default, variable is immutable.
> make it mutable by  `mut`
```rust
let mut x = 5;
```

### constant vs immutable variables

- use `const` for constants because they are not meant to be changed ever.
- can't use `const` for values computed at runtime. like result of function calls. 
- naming convention : use uppercase with underscores.
```rust
const MAX_POINTS: u32 = 1000_00;
```

### Shadowing

- *shadowing* is declaring a new variable with the same
name as a previous variable.
```rust
let x = 5;
let x = x+1;// don't use mut!
```
- it is not reassigning like we do with `mut` variables.
- *benefit* : we don't have to create a variable with different type for same data. If we need a num but the input is in string. we can use the same name to get the numeric value because it is literally *redeclaring*.

## Data types

> rust is *statically typed*

- rust usually guesses the type but we can explicitly tell it , ob. 
### Scalars 

#### Integers
 - signed i8 (integer8-bit) up to 128-bit or isize(architecture size)
 - similar but **u**8

##### literals
- 98_222 = 98222 (!you can use _ as separator)
- 0xfff hex, Oo77 oct, 0b1111_000 binary, b'A' Byte(u8 only).

- while using `--release` flag, rust won't check for *integer overflow* . It performs *two's compliment wrapping*. ex: for u8 , 256 becomes 0,
    - prevents *panic* 
    - you can use standard library type `Wrapping` for explicitly wrapping

#### floating points

- default is f64, else we have f32.

### Numeric operations

- +, - , * , / , % . Same as usual

### Boolean

- `true` or `false`

### characters

- `char` : 4 bytes ,  Unicode Scalar Values range from
U+0000 to U+D7FF and U+E000 to U+10FFFF inclusive 

### Compound Types

#### Tuple type

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let (x, y, z) = tup;// pattern matching : x = 500
let first_element = tup.0;
```
- *destructuring*: use pattern matching to get value breaks single tuple into multiple parts
- we can use (.) followed by index too!! 

#### Array type

- ***Fixed sized***

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; //is same as
let a = [3, 3, 3, 3, 3];// 5 elements
```

- stack based , not as flexible as vectors
- accessing using index `a[index]`
- gives `index out of bounds` error if out of bounds

## Functions

> convention: use snake case.

### Function parameters

- must declare type of each parameters

```rust
fn another_function(x: i32, y: i32) {...}
```

### Statements and Expressions

> rust is an expression-based language

- `let x = (let y = 4);` is invalid unlike C because let y = 4 doesn't yield a vlaue ; nothing to bind for x
- calling a function or macro , block creating new scopes ,{}, is an expression

```rust
let y = {
    let x = 3;
    x+1// look at this. NO SEMICOLON!!
} // this block/scope returns value 4
```
### Function with return values

- we declare type of return values after an arrow (->)
- *implicitly* : return value is the last expression. keep in mind the facts about statement vs expressions.
- *explicitly* : we can use `return` keyword

```rust
fn main() {
    let x = plus_one(5);
    println!("The value of x is: {}", x);
}
fn plus_one(x: i32) -> i32 {
    x + 1 // Again no semicolon here.
}
```
 - no evaluation leads to `()` empty tuple.

## Comments

- no multi-line comment syntax.
- continue `//` for each line.
- `///` *documentation comment* : generates HTML ; used with `crates`

## Control Flow

### if/else if expressions
- Same as C or JS for syntax 
- condition must be `bool`.
- Blocks of code associated with the conditions in if expressions are sometimes called `arms`

#### Using if in a let statement
```rust
let number = if condition { 5 } else { "six" }; // error: both arms should evaluate to same data type
```

## Loops

### loop{}

- simple just loops code inside the block 

```rust
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};
```
 - use this with `break`.
 - use case : One of the uses of a loop is to retry an operation you know might fail, such
as checking whether a thread has completed its job. However, you might
need to pass the result of that operation to the rest of your code. 

### while loops

 - same as C/JS.

## for loops

```rust
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("the value is: {}", element);
}
```
- Range based: `for number in (1..4).rev()` . `rev()` is to reverse

# Unique features in Rust

## Ownership

> How rust lays data in memory

> ensures memory safety without garbage collector 

- In other languages, programmer must explicitly handle the memory (allocation & deletion)
- In rust, we have ownership concept.

### stack vs heap

- We use heap when the size of values at compile time is unknown.
- pushing in stack is faster because OS never has to search for a place to store new data unlike heap where OS has to search for space big enough to hold data
 - accessing data in heap is slower because we have to follow a pointer to get there.

> so cleaning up unused data , minimizing the amount of duplicate data so that we don't run out of space are all problems that ownership addresses.

### Ownership rules

- Each value Rust has a variable that’s called its `owner`.
- There can be only one `owner` at a time.
- When the owner goes out of scope, the value will be dropped.

### variable scopes

- Block scoped. like in JS/C++

### String types and heap

- `String` types are allocated in heaps.
- `let s = String::from("hello");` creating `String` from `string literal`
- `string literals` cannot be mutated but `Strings` can. Because we know the size of literals and can be hardcoded in the program. `Strings are growable`.

### Memory and Allocation

- memory must be created and destroyed simultaneously. because there is no garbage collector; doing this is very hard. `allocate` and `free`
- rust is automatically returned once the variable goes out of scope. Calls `drop` function.

### data interaction : Move
```rust
let s1 = String::from("hello");
let s2 = s1;
```
- Here, a stack is created with (ptr,len,capacity) ptr pointing to the heap of memory with data(hello)
- `s2 = s1` : copies the ptr of s1(stack) to s2 ; pointing to same heap.
- **But**, when we go out of scope, both of them will try to free the same memory. Oops! `double free` error.

> Rust manages this by making s1 invalid , which is what it calls `move` . Only s2 can free the memory.

### data interaction : Clone

- `let s2  = s1.clone()` , `deep`ly copies the heap data not just the stack data. 

### Stack only data: Copy

```rust
let x = 5;
let y = x;
```
- As we have learned, these basic types are stored in stack at compile time. So no need of any allocation,freeing or making it invalid.
- no need of `clone` here.

 > we have to keep in mind the `Copy` and `Drop` traits. Normally all those basic data types have `Copy` trait.

### Ownership and functions

- going into function is going out of scope. non `Copy` traits are borrowed  by the function and made invalid.

### Return Values and Scope

- multiple values can be returned using tuples
- Just like function taking the ownership , returning it gives the ownership back. *It is just changing the scopes*
- If no one takes the ownership , going out of scope just destroys it.

```rust
fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);
}
fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String
    (s, length)
}
```
### References and Borrowing

- So to avoid all those hassle, we can use *refrence* variables. `&String`
- We call it *borrowing* , like in real world
- also has dereferencing with *deference operator*

> To make the reference variable mutable, we use `&mut` for `mut` variable

- We cannot burrow mutable reference more than once at a time in a particular scope.
- this is to prevent *data race* :
    - two or more pointers accessing same data
    - At least one is being used to write
    - no mechanism being used to synchronize access to the data

- Just use curly braces { ... } to create a new scope.😉 
- *We also cannot have a mutable reference while we have an immutable one*

### Dangling references

- *`lifetimes`* help us a lot for such *dangling pointers*: pointers referencing a location in memory that may have been given to someone else.

### The Slice type

- while slicing byte by byte, we are normally working on starting index and ending index. Though these indices may be found, we can't possibly use this since the state of `String` that we are working on might change it's state.

#### String Slices

```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];
```
- new slice with different pointer is created.
- `[starting_index..ending_index]`
- `[0..2]` <=> `[..2]` ; `[3..length_of_string]` <=> `[3..]` ; `[0..length_of_string]` <=> `[..]`
- these types of slices are returned/used as `&str` (string slice) types
- `&str` are immutable.

> And.. String literals are `&str`.

- **Don't mess up with immutable/mutable borrowing rule**

- UTF-8 encoded text might be multibyte. So we need to take care of that too

#### String Slices as parameters

- It is better to pass string slice `&string[..]` (whole string) as parameter instead of whole string `&string`.

#### Other slices 

- We can use slices for arrays too `let slice = &array[1..3];`

## Using Structs to structure related data

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
} // struct definition
```
- data inside curly braces are called `fields`

```rust
let user1 = User {
    //key:value
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,   
};// creaing an instance of the User struct
```
- To change the value, instance must be mutable too!

### Fields init Shorthand

- When the parameters are same as the key , we can omit that

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email, // instead of email:email;
        username,
        active: true,
        sign_in_count: 1,
    }
}
```
### Creating instances from Other Instance

> `struct update syntax`

```rust
let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotherusername567"),
        ..user1 // rest of the values are same as that of user1
    };
```

### Create different Types using `tuple structs`

> `Tuple structs` : structs without named fields

```rust
struct Color(i32, i32, i32);
let black = Color(0, 0, 0);
```

- You cannot pass many `tuple structs` with same fields as a parameter for a function : Behaves like tuples

- again, keep in mind the `lifetime` parameter. (Explained later)

### Printing structure for debugging

```rust
// here add this. This is called deriving Debug Trait
#[derive(Debug)] 
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    println!("rect1 is {:?}", rect1); //Notice the :? inside
    // prints rect1 is Rectangle { width: 30, height: 50 }
}
```
- `println!("rect1 is {:?#}", rect1);` even prints with line breaks (pretty print)

### Methods

#### Defining Methods

```rust
//after defining struct in above code
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
}
```
- `&self` knows that it is `&Rectangle` because it is implemented inside `imp Rectangle` context.
- `&self` because we want to *borrow* the ownership when we have to read.
- we'd use `&mut self` to change the instance that we’ve called the method on as part of what the method does.

#### Where's the -> operator?

- Rust has *automatic referencing and dereferencing* while calling methods. so it automatically adds in `&`, `&mut`, or `*` so object matches the signature of the
method.
- This automatic referencing behavior works
because methods have a clear receiver—the type of self

#### Methods with more parameters

- `rect1.can_hold(&rect2));`. here one instance has a method taking another instance as a parameter.
- To use this we add this in the `imp Rectangle` block:

```rust
fn can_hold(&self, other: &Rectangle) -> bool {
    ...
}
```
> so `&self` makes rust clear which instance it is taking about. 

#### Associated Functions

> useful for creating *Constructors*

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

- to call this we use `let sq = Rectangle::square(3)`.
- The function is namespaced by the struct

#### Multiple impl Blocks

> We can use many such `impl { }` blocks for the same struct. (useful for generic types and traits)

## Enums and pattern matching

- *enums* : enumerations : types that enumerates its possible values.

### Defining an enum

 ```rust
enum IpAddressKind {
    V4,
    V6,
}
 ```
- `let four = IpAddrKind::V4;` : variants of the enum are namespaced under its identifier and we use `::` to resolve the namespace.
- enums are useful when used along with structs since an enum is a type like this:

```rust
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}
fn main() {
    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };
}

```

- But we can use it directly too like this:
```rust
enum IpAddressKind {
    V4(u8,u8,u8,u8),
    V6(String),
}
let home = IpAddr::V4(192,168,1,1);
let loopback = IpAddr::V6(String::from("::1"));
 ```
 > checkout standard library for IpAddr

- The real benefit of enum rather than using multiple structs is to define a function. Remember `impl` from structure using `&self`.

### *Option* Enum and its advantages over Null values

- There is no `Null` value in rust. Why? 
>The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.
- But *null* concept is still useful so rust has an *Option* 😉. It has `Option<T>`enum for that.

```rust
enum Option<T> {
    Some(T),
    None,
}
```
- Being so useful, it can be used without bringing it to scope. which means we can use `Some(T)` and `None` without `Option::` prefix.

- `let some_number = Some(5);` we know what value is present
- `let absent_number: Option<i32> = None;` we don't have a value; essentially a null value.

> **Remember! `Option<T>` and `T` are not the same type**

- The reason why `Option<T>` is that we are explicitly deciding that we are going to handle cases that might have null or some value.
- So to use a value of type `T` from `Option<T>` when the code is handled , what control flow operator can be used ? *match* operator

## `match` control flow operator

- compares values against a series of pattern.
- Patterns can be made up of literal values,
variable names, wildcards, and many other thing
- code associated with the pattern matched first will be executed.

```rust
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

value_in_cents(Coin::Quarter(UsState::Alaska)); //will match with

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => { // will match with this 
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```
- as we can see, we can match the enums within the enums.

#### Matching with Option<T>

- to get the value of T from Option<T> we use it like this:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1), // plus_one(five) below matched this
    }
}
let five = Some(5);
let six = plus_one(five);
```
- We pass *Some* value to match an `Option<T>` type and match it.

> Matches are exhaustive : So there must be an *match `arm` for every possible cases. Rust will throw error otherwise. It's good. Handles everything. But there is a solution for that too. (`_` palceholder)

#### The `_` placeholder

- whenever we don't want/have to list all possible value , we can use `_` pattern and link it `=>` with `()`:a unit value  which is basically nothing.

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

### if let control flow

- the above code is equivalent to:
```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```
- We want to do something with the `Some(3)` match but do nothing with any other `Some<u8>` value or the None value
- works the same way as `match` . However, we loose the exhaustive checking.
- also works with the same logic that we used for matching enums within enums