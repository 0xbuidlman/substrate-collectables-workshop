Storing a Structure
===

If you thought everyone getting their own number was cool, lets try to give them all digital kitties!

First we need to define what properties our kitties have in the form of a `struct`, and then we need to learn how to store these custom `structs` in our runtime storage.

Let's start with defining the properties of our kitties:

```
#[derive(Encode, Decode, Default, Clone, PartialEq, Debug)]
pub struct Kitty<Hash, Balance> {
  name: Vec<u8>,
  dna: Hash,
  price: Balance,
  gen: u32,
}
```

The `name` and `price` properties of a kitty should be self-explanatory. We will use the `dna` represented as a `Hash` to store the specific physical characteristics the cat has. Finally, we will introduce a `gen` properity, which will keep track of what generation each kitty is once we introduce breeding.

Note that we define our struct using [*generics*](https://doc.rust-lang.org/rust-by-example/generics.html), which is a powerful concept in Rust that we will not dive deep into. For the purposes of this tutorial, you can assume these are simply aliases to Substrate specific types like `system::Trait::Hash` and `balances::Trait::Balance`.

There is one last thing we are going to glaze over, which is the `derive` macro used at the very top. In order for custom structs to be used throughout the Substrate codebase, they must satisfy a set of `traits` about what can and cannot be done to that type. This macro, automatically generates the required code to give this struct the ability to be encoded, decoded, cloned, etc... for the purposes of this tutorial you can treat it like magic. (because it is...)

Disclaimers out of the way, we can finally plug this kitty into our runtime:

```
use system::ensure_signed;
use srml_support::{StorageMap, dispatch::Result};
use runtime_primitives::traits::{Hash, As};
use rstd::prelude::*;

pub trait Trait: balances::Trait {}

#[derive(Encode, Decode, Default, Clone, PartialEq, Debug)]
pub struct Kitty<Hash, Balance> {
  name: Vec<u8>,
  dna: Hash,
  price: Balance,
  gen: u32,
}

decl_storage! {
    trait Store for Module<T: Trait> as CryptokittiesStorage {
        Value: map T::AccountId => Kitty<T::Hash, T::Balance>;
    }
}

decl_module! {
    pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        fn create_kitty(origin, name: Vec<u8>) -> Result {
            let sender = ensure_signed(origin)?;

            let new_kitty = Kitty {
                                name: name,
                                dna: <T as system::Trait>::Hashing::hash_of(&0),
                                price: <T::Balance as As<u64>>::sa(0),
                                gen: 0,
                            };

            <Value<T>>::insert(sender, new_kitty);
            Ok(())
        }
    }
}
```

You can see that when the user called our new `create_kitty()` function, we generate a `new_kitty` using the `Kitty` struct, and then insert that directly into our storage like before.

Additionally, our storage is now configured to store a `Kitty` for each `AccountId`.

Substrate does not directly support `Strings`, so we are using a `Vec<u8>` to act as one for our kitties. We will need to remember to convert this value to and from a human readble string with our front end UI.