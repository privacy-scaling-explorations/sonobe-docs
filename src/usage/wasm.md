# WASM - browser usage

## WASM targets
In order to build the lib for WASM-targets, use the following command:
```
cargo build -p folding-schemes --no-default-features --target wasm32-unknown-unknown --features "parallel"
```

Where the target can be any WASM one and the `parallel` feature is optional.


## WASM-compatibility & features

`getrandom/js` needs to be imported in the `Cargo.toml` of the crate that uses sonobe as dependency:
```toml
[dependencies]
folding-schemes = { git = "https://github.com/privacy-scaling-explorations/sonobe", package = "folding-schemes", default-features = false, features = ["parallel"] }
getrandom = { version = "0.2", features = ["js"] }
```
[More details](https://docs.rs/getrandom/latest/getrandom/#webassembly-support) about `getrandom`.


## Experimental frontends & WASM
Not all experimental frontends are supported in the browser.

### Circom frontend & WASM
In order to use the Circom experimental browser, the feature `circom-browser` needs to be set.
