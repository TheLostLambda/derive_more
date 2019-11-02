% What #[derive(AsMut)] generates

Deriving `AsMut` generates one or more implementations of `AsMut`, each corresponding to one of the
fields of the decorated type. This allows types which contain some `T` to be passed anywhere that an
`AsMut<T>` is accepted.

# Newtypes and Structs with One Field

When `AsMut` is derived for a newtype or struct with one field, a single implementation is generated
to expose the underlying field.

```rust
# #[macro_use] extern crate derive_more;
# fn main(){}
#[derive(AsMut)]
struct MyWrapper(String);
```


Generates:

```
# struct MyWrapper(String);
impl AsMut<String> for MyWrapper {
    fn as_mut(&mut self) -> &mut String {
        &mut self.0
    }
}
```

# Structs with Multiple Fields

When `AsMut` is derived for a struct with more than one field (including tuple structs), you must
also mark one or more fields with the `#[as_mut]` attribute. An implementation will be generated for
each indicated field.

```rust
# #[macro_use] extern crate derive_more;
# fn main(){}
#[derive(AsMut)]
struct MyWrapper {
    #[as_mut]
    name: String,
    #[as_mut]
    num: i32,
    valid: bool,
}


```


Generates:

```
# struct MyWrapper {
#     name: String,
#     num: i32,
#     valid: bool,
# }
impl AsMut<String> for MyWrapper {
    fn as_mut(&mut self) -> &mut String {
        &mut self.name
    }
}

impl AsMut<i32> for MyWrapper {
    fn as_mut(&mut self) -> &mut i32 {
        &mut self.num
    }
}
```

Note that `AsMut<T>` may only be implemented once for any given type `T`. This means any attempt to
mark more than one field of the same type with `#[as_mut]` will result in a compilation error.

```compile_fail
# #[macro_use] extern crate derive_more;
# fn main(){}
// Error! Conflicting implementations of AsMut<String>
#[derive(AsMut)]
struct MyWrapper {
    #[as_mut]
    str1: String,
    #[as_mut]
    str2: String,
}
```

# Enums

Deriving `AsMut` for enums is not supported.