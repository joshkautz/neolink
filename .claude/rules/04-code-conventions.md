# Code Conventions

## Rust Edition and Toolchain

- **Rust edition 2018** (specified in Cargo.toml)
- Uses stable Rust toolchain

## Naming

- `snake_case` for functions, methods, variables, and modules
- `PascalCase` for types (structs, enums, traits)
- `SCREAMING_SNAKE_CASE` for constants and statics

## Async

- Tokio is the async runtime (`#[tokio::main]` with multi-thread)
- Use `async`/`await` throughout; do not block the async runtime
- `tokio-stream` and `tokio-util` for stream utilities

## Error Handling

- **anyhow** for application-level errors (main binary, subcommands)
- **thiserror** for library-level error types (neolink_core)
- Propagate errors with `?` operator; use `.context()` for additional info

## Serialization

- XML bodies use `quick-xml` with serde derives
- Protocol structures use `#[derive(Debug, Clone, Serialize, Deserialize)]`
- TOML configuration parsed with the `toml` crate and serde
- JSON used for MQTT payloads via `serde_json`

## Memory Allocator

- **jemalloc** (`tikv-jemallocator`) on non-MSVC platforms for better long-running performance
- Standard allocator on MSVC/Windows

## Formatting and Linting

- `cargo fmt` for formatting (configuration in `rustfmt.toml`)
- `cargo clippy` for linting
- `deny.toml` for supply chain security (cargo-deny)

## Logging

- `log` crate macros (`info!`, `debug!`, `warn!`, `error!`, `trace!`)
- `env_logger` for runtime log level control via `RUST_LOG`
- Release builds cap at debug level (`release_max_level_debug` feature on log crate)

## Dependencies

- Prefer well-maintained crates from the Rust ecosystem
- Feature-gate optional functionality (see `[features]` in Cargo.toml)
- Keep workspace members in `crates/` directory
