# Entorno de desarrollo

Estado en hermes-p (29 sep 2026):

| Herramienta | Estado |
|---|---|
| Node.js / npm | v26.8.1 / 11.19.0 instalados |
| Docker | 29.5.3 instalado |
| git | instalado |
| Rust (`rustup`, `cargo`) | **falta** |
| Stellar CLI (`stellar`) | **falta** |

## Instalación pendiente (cuando se decida el stack)

```bash
# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32v1-none

# Stellar CLI
cargo install --locked stellar-cli

# Identidad de prueba con fondos de testnet
stellar keys generate dev --network testnet --fund
```

## Flujo típico de un contrato

```bash
stellar contract init contracts/<nombre>
stellar contract build
stellar contract deploy --wasm target/wasm32v1-none/release/<nombre>.wasm \
  --source dev --network testnet
stellar contract invoke --id <CONTRACT_ID> --source dev --network testnet -- <funcion> --arg valor
stellar contract bindings typescript --id <CONTRACT_ID> --network testnet --output-dir app/src/contracts/<nombre>
```
