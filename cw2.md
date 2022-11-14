# cw2: contract versioning

Looking at the [`cw-plus` contracts](https://github.com/CosmWasm/cw-plus), I was seeing a lot of this:

```rust
use cw2::set_contract_version;

// version info for migration info
const CONTRACT_NAME: &str = "crates.io:cw20-base";
const CONTRACT_VERSION: &str = env!("CARGO_PKG_VERSION");

pub fn instantiate(…) {
    set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;
```

I'm curious to learn more about how CosmWasm contracts are versioned.

Start with the README: [cw-plus/packages/cw2/README.md](https://github.com/CosmWasm/cw-plus/tree/main/packages/cw2).

Ok, fair enough. So this is for migrations. Surprisingly, the contracts that `use cw2::set_contract_version` don't use any `cw2` imports in the `migrate` logic:

```rust
pub fn migrate(…) {
    let original_version =
        ensure_from_older_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;
```

This `ensure_from_older_version` comes from `cw_utils`:

```rust
use cw_utils::ensure_from_older_version;
```

Which, as Cargo.toml tells me, is also in the `packages` directory within `cw-plus`:


```toml
[dependencies]
cosmwasm-schema = { version = "1.0.0" }
cw-utils = { path = "../../packages/utils", version = "0.14.0" }
```

Or we can just option-click (or, in vim mode, `gd`) on `ensure_from_older_version` to go right to the definition.

And, aha, there at the top of the file we see `cw2` again:

```rust
use cosmwasm_std::{StdError, StdResult, Storage};
use cw2::{get_contract_version, set_contract_version};
use semver::Version;
```

So now we know our contract is using `set_contract_version` twice, once directly and once via `cw-utils`. One wonders if `ensure_from_older_version` ought to be moved to the `cw2` package. 🤔

Let's see how these two `cw2` exports work.

### `get_contract_version`

Amusingly simple:

```rust
pub fn get_contract_version(store: &dyn Storage) -> StdResult<ContractVersion> {
    CONTRACT.load(store)
}
```

Literally just loading a value out of storage, where `CONTRACT` is defined as:

```rust
pub const CONTRACT: Item<ContractVersion> = Item::new("contract_info");
```

Just like the README said. Beautiful.

### `set_contract_version`

Some cool stuff here!

```rust
pub fn set_contract_version<T: Into<String>, U: Into<String>>(
    store: &mut dyn Storage,
    name: T,
    version: U,
) -> StdResult<()> {
    let val = ContractVersion {
        contract: name.into(),
        version: version.into(),
    };
    CONTRACT.save(store, &val)
}
```

My favorite part here, and the part that confused me at first, is the `T` and `U` generics, rather than just seeing `String` types. Like, this would be "simpler", no?

```rust
pub fn set_contract_version(store: &mut dyn Storage, name: String, version: String) -> StdResult<()> {
    let val = ContractVersion {
        contract: name,
        version: version,
    };
    CONTRACT.save(store, &val)
}
```

So what's with the `T: Into<String>` ceremony?

Well, scroll back up and you'll see that `CONTRACT_NAME` and `CONTRACT_VERSION` were defined as `&str`, string slices! No `.to_string()` or `.clone()` needed! Without going and searching for any docs, I can understand that this style of generic allows me to pass any arguments _that can be turned into_ strings.

This works with the `.into()` in the function body. Rust knows from the definition of `ContractVersion` that whatever arguments are passed in need to be turned into `String`s:

```rust
pub struct ContractVersion {
    pub contract: String,
    pub version: String,
}
```

I will definitely be using this!
