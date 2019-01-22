Refactoring Our Code
===

Our `create_kitty()` function is pretty bulky and has code we will probably want to use later when we introduce kitty breeding and other ways to "mint" new kitties.

We will take this opportunity to teach you about writing private internal functions which are not directly exposed through your runtime's API, but are still accessible by your module.

## Public Interfaces and Private Functions

Within your runtime, you are able to include an implementation of your runtime module like so:

```
impl<T: Trait> Module<T> {
    // Your functions here
}
```

Functions in this block are usually public interfaces or private functions. Public interfaces should be labeled `pub` and generally fall into inspector functions that do not write to storage and operation functions that do. Private functions are your usual private utilities unavailable to other modules.

You can call functions defined here using the `Self::function_name()` pattern you have seen before. Here is an intentionally overcomplicated example:

```
decl_module! {
    pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        fn adder_to_storage(origin, num1: u32, num2: u32) -> Result {
            let _sender = ensure_signed(origin)?;
            let result = _adder(num1, num2);

            _store_value(result)?;

            Ok(())
        }
    }
}

impl<T: Trait> Module<T> {
    fn _adder(num1: u32, num2: u32) -> u32 {
        let final_answer = match num1.checked_add(num2) {
            Some(c) => c,
            None => Err("Overflow when adding"),
        };
    }

    fn _store_value(value: u32) -> Result {
        <myStorage<T>>::put(value);
        
        Ok(())
    }
}
```

Remember that we still need to follow a "check, then store"  pattern, so it is important to not daisy chain private functions which do writes to storage where there is a chance one will throw an error.

## Your Turn!

For us, the process of creating a new kitty from a `Kitty` object and updating all the storage variables is somthing that we should make reusable so that we can create other which takes advantage of the same code.

[TODO: Finish this]