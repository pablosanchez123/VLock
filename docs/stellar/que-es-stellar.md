# Stellar en una página

Stellar es una red blockchain pública, de código abierto, creada en 2014 para mover
dinero: pagos rápidos, baratos y entre monedas distintas. La mantiene la Stellar
Development Foundation (SDF), una organización sin fines de lucro. Su criptomoneda
nativa es el **XLM (lumen)**.

## Piezas clave

- **Cuentas**: se identifican con una clave pública `G...` y se firman con una secreta
  `S...`. Para existir necesitan un saldo mínimo de XLM (reserva).
- **Activos (assets)**: cualquiera puede emitir un token en Stellar (un dólar digital,
  puntos, un bono). Una cuenta debe aceptarlo antes con una *trustline*. Así existen
  USDC (de Circle) y muchas monedas estables locales.
- **Anchors**: empresas que conectan la red con el dinero tradicional (depósitos y
  retiros en bancos, efectivo). Se integran con estándares llamados SEP (SEP-10
  autenticación, SEP-24 y SEP-6 depósitos y retiros, SEP-31 pagos transfronterizos).
- **DEX integrado**: la red trae un mercado de intercambio propio y *path payments*:
  se paga en una moneda y el receptor recibe otra, con la conversión automática.
- **Consenso (SCP)**: no hay minería. Las transacciones se confirman en unos 5 segundos
  y cuestan una fracción de centavo.
- **Soroban (contratos inteligentes)**: desde 2024 Stellar ejecuta contratos escritos en
  **Rust** y compilados a WebAssembly. Permiten lógica propia: escrow, préstamos,
  votaciones, identidad, lealtad, etc.
- **Passkeys / smart wallets**: cuentas controladas con la huella o Face ID del teléfono,
  sin frases semilla, gracias a Soroban.

## Redes

| Red | Uso | RPC | Horizon |
|---|---|---|---|
| Testnet | Desarrollo, gratis (XLM de prueba vía Friendbot) | https://soroban-testnet.stellar.org | https://horizon-testnet.stellar.org |
| Mainnet | Dinero real | proveedores de RPC | https://horizon.stellar.org |

Para el hackathon se trabaja en **testnet**.

## Herramientas del desarrollador

- **Stellar CLI** (`stellar`): crea proyectos de contratos, compila, despliega, invoca y
  genera clientes TypeScript (`stellar contract bindings typescript`).
- **Rust + target `wasm32v1-none`** para compilar contratos.
- **SDK de JavaScript** `@stellar/stellar-sdk` (también hay de Python, Go, Java, etc.).
- **Stellar Wallets Kit** para conectar billeteras (Freighter, xBull, Lobstr…) desde la web.
- **Stellar Lab** (https://lab.stellar.org) para probar transacciones en el navegador.
- **Exploradores**: https://stellar.expert (ver cuentas, contratos y transacciones).

## Enlaces

- Documentación: https://docs.stellar.org
- Contratos (Soroban): https://developers.stellar.org/docs/build/smart-contracts/overview
- Primeros pasos con contratos: https://developers.stellar.org/docs/build/smart-contracts/getting-started
- Ejemplos oficiales: https://github.com/stellar/soroban-examples
- Estándares SEP: https://github.com/stellar/stellar-protocol/tree/master/ecosystem
