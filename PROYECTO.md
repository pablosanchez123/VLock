# Proyecto: votación presencial con conteo verificable en Stellar

Documento de contexto completo para el equipo y sus asistentes de IA. Es autocontenido:
con este archivo se puede empezar a trabajar sin haber leído nada más.

Última actualización: 29 de septiembre de 2026.

---

## 0. Instrucciones para la IA que lea este documento

- Este es un proyecto de **hackathon con fecha límite dura** (5 oct 2026, 4:00 p. m.
  hora de Costa Rica, UTC-6). Priorizar siempre una demo que funcione de punta a punta
  sobre funciones extra.
- La idea ya está **decidida**. No proponer ideas de producto alternativas; ayudar a
  construir esta.
- Todo corre en la **testnet de Stellar**. Nunca usar mainnet ni pedir claves reales.
- Nunca subir al repositorio claves secretas de Stellar (empiezan con `S`), archivos
  `.env` ni identidades generadas por `stellar keys`.
- El repositorio será **público y de código abierto** (licencia MIT): escribir código y
  comentarios presentables.
- Idioma del equipo: español. Nombres de funciones del contrato y textos de la interfaz
  en español; identificadores técnicos de librerías en su idioma original.
- Las especificaciones de este documento (sección 6 y 7) son el contrato de trabajo
  entre las partes del equipo. Si algo necesita cambiar, actualizar este archivo.

---

## 1. El hackathon

| Dato | Valor |
|---|---|
| Nombre | Find Your Way: Hackathon |
| Organiza | Tellus Cooperative (cooperativa blockchain para Iberoamérica), anfitrión ZEEK, plataforma Stellar Passport |
| Propósito | Dar experiencia práctica con Stellar y preparar equipos para HackMeridian (Lisboa) |
| Encuentro presencial | Tecnológico de Costa Rica (TEC), Cartago, 6:00–8:00 p. m. (horario provisional); da 50 puntos de Passport |
| Luma | https://luma.com/9h5jl18k?tk=BIGM6L |
| Plataforma | https://demo.stellarpassport.xyz/hackathons/find-your-way-meridian-hackathon |
| Documentación oficial | https://docs.stellar.org |

### Fechas (hora de Costa Rica)

| Hito | Fecha |
|---|---|
| Entregas abiertas | desde el 21 sep 2026 |
| **Cierre de entregas** | **5 oct 2026, 4:00 p. m.** |
| Evaluación | hasta el 9 oct 2026 |
| Resultados | 12 oct 2026, 6:00 p. m. |

### Track y premios

Participamos en el **General Track** (4 000 USDC): 1.º 2 000 · 2.º 1 000 · 3.º 500 ·
dos menciones de 250. (El University Track es solo para universidades de Chile.)

### Criterios de evaluación

1. Ejecución técnica
2. **Uso significativo de Stellar**
3. Originalidad
4. Impacto potencial
5. Experiencia de usuario
6. Calidad de la presentación

### Qué hay que entregar (formulario)

- Nombre del proyecto
- Descripción
- Track: General Track
- Correo
- **Enlace al repositorio (obligatoriamente de código abierto)**
- **Enlace al video pitch (máximo 3 minutos)**
- Otros enlaces opcionales: demo pública, contrato en stellar.expert
- Aceptar términos de Stellar Passport

Requisitos de registro: cada integrante con cuenta en Stellar Passport, inscripción en
el hackathon, equipo creado en la plataforma (1 a 5 personas).

---

## 2. Stellar en breve

Stellar es una blockchain pública creada en 2014 para pagos. Su moneda es el XLM.
Confirma transacciones en ~5 segundos por una fracción de centavo, sin minería, usando el
Stellar Consensus Protocol (SCP) entre decenas de validadores de organizaciones
independientes.

Conceptos que usa este proyecto:

- **Cuenta / llave**: pública `G…` (identifica), secreta `S…` (firma).
- **Soroban**: contratos inteligentes en **Rust**, compilados a WebAssembly. Un contrato
  desplegado tiene dirección `C…`, guarda datos clave→valor y se ejecuta en todos los
  validadores; su resultado es definitivo tras el consenso.
- **`require_auth()`**: exige que una dirección haya firmado la transacción.
- **Eventos**: avisos públicos que emite un contrato y que las apps pueden escuchar.
- **RPC de Soroban**: punto de acceso para leer y escribir contratos.
- **Testnet**: red de pruebas; las cuentas se financian gratis con Friendbot.
- **stellar.expert**: explorador público para ver cuentas, contratos y transacciones.
- **Protocol 25 "X-Ray"** (ene 2026): agregó primitivas para pruebas de conocimiento
  cero (BN254, Poseidon). No se usa en el MVP; se menciona como trabajo futuro.

| Red | RPC | Horizon | Passphrase |
|---|---|---|---|
| Testnet | https://soroban-testnet.stellar.org | https://horizon-testnet.stellar.org | `Test SDF Network ; September 2015` |

---

## 3. El problema

En una elección, la confianza en el resultado depende de que nadie altere actas ni
conteos entre la mesa y el resultado final. Hoy eso exige:

- contar a mano todas las papeletas de todas las mesas;
- transcribir actas a mano (errores humanos);
- transmitir y luego revisar todas las actas en un escrutinio que toma días;
- y, sobre todo, **confiar en quienes custodian, transmiten y suman los datos**.

## 4. La solución

Votación **presencial** con computadoras de votación auditadas en cada mesa, cuyo
conteo queda en un contrato de Stellar.

1. La persona se identifica con la cédula ante la mesa, como hoy.
2. Vota en una computadora de la mesa, en un recinto privado.
3. La computadora muestra/imprime un **comprobante** que la persona revisa y deposita en
   una urna (no se lo lleva, para impedir compra de votos y coacción).
4. La computadora acumula votos y los envía a Stellar **por lotes**, firmados con su
   propia llave.
5. El contrato solo acepta lotes de máquinas autorizadas, con la votación abierta, y
   solo suma. Nadie, ni la administración, puede editar o borrar votos.
6. El resultado es público y verificable en tiempo real por cualquiera.
7. Al cierre, se audita una **muestra al azar** de urnas: los comprobantes deben
   coincidir con lo registrado en Stellar.

**Frase del pitch:** "El conteo no solo es más rápido: ya no hay que confiar en quien
cuenta."

### Por qué no es voto por teléfono

Se descartó a propósito. Votar desde el teléfono no permite verificar identidad de forma
robusta, abre la puerta a coacción y compra de votos, y depende de dispositivos no
controlados. La votación presencial con máquinas auditadas elimina esos problemas.

### Por qué lotes y no voto por voto

Si cada voto entrara a Stellar al instante con hora exacta, quien anote a qué hora votó
cada persona podría deducir su voto. La máquina acumula N votos (en la demo, 5), los
mezcla y los envía juntos. También tolera cortes de internet: los votos esperan en la
máquina.

### Por qué el comprobante en papel

La blockchain protege el voto **después** de registrado, pero no puede saber si la
máquina registró lo que la persona eligió. Si el software de una máquina se altera,
Stellar guardaría fielmente un voto equivocado. El comprobante es un segundo registro,
verificado con los ojos de la persona e independiente del software. En la auditoría,
papel y Stellar deben coincidir; si no, se detecta la alteración. (Principio de
"independencia del software", práctica recomendada en voto electrónico.)

### Qué se agiliza si igual hay papel

No se cuentan todas las urnas: solo una muestra al azar (auditoría por muestreo). El
resultado completo existe en Stellar al momento del cierre. Se eliminan el conteo manual
masivo, las actas escritas a mano, la transmisión y el escrutinio de todas las actas.

### Límites que se dicen con honestidad en el pitch

- La blockchain protege el voto después de registrado; antes, la confianza se apoya en
  máquinas auditadas y en el comprobante en papel.
- "Una persona, un voto" lo sigue controlando la mesa con el padrón.
- En Costa Rica el TSE ya da resultados provisionales la misma noche; el argumento
  fuerte no es la velocidad sino la **verificabilidad sin confiar en nadie**.

---

## 5. Arquitectura

```
 ┌──────── MESAS DE VOTACIÓN ────────┐
 │ Quiosco 1   Quiosco 2   Quiosco 3 │
 │ (llave 1)   (llave 2)   (llave 3) │
 └───────────────┬───────────────────┘
                 │ registrar_lote (firmado)
                 ▼
 ╔═══════════════════════════════════╗
 ║     RED STELLAR (testnet)         ║
 ║  Contrato "eleccion"              ║
 ║  reglas + conteo oficial          ║
 ╚═══════════════╤═══════════════════╝
                 │ lectura (gratis)
     ┌───────────┼────────────┐
     ▼           ▼            ▼
 ┌────────┐ ┌──────────┐ ┌──────────┐
 │Tablero │ │Auditoría │ │ stellar  │
 │público │ │          │ │ .expert  │
 └────────┘ └──────────┘ └──────────┘
        ▲
 ┌──────┴─────┐
 │ Panel admin│ (inicializar, registrar máquinas, abrir, cerrar)
 └────────────┘
```

- **No hay base de datos propia.** El conteo oficial vive solo en el contrato.
- Las pantallas solo escriben (quiosco, admin) o leen (tablero, auditoría).
- Cualquiera puede verificar el resultado directamente en Stellar sin pasar por nuestra app.

### Stack

| Parte | Tecnología |
|---|---|
| Contrato | Rust + `soroban-sdk`, target `wasm32v1-none` |
| Herramientas | Stellar CLI (`stellar`) |
| App web | TypeScript + React + Vite |
| Cliente del contrato | Bindings TypeScript generados con `stellar contract bindings typescript` + `@stellar/stellar-sdk` |
| Hash de comprobantes | SHA-256 (Web Crypto en el navegador, `env.crypto()` no se necesita en el contrato) |

### Estructura del repositorio

```
contracts/eleccion/     Contrato Soroban (Rust) y sus tests
app/                    App web (React + Vite + TS)
  src/pages/Quiosco.tsx
  src/pages/Resultados.tsx
  src/pages/Admin.tsx
  src/pages/Auditoria.tsx
  src/contracts/eleccion/   bindings generados (no editar a mano)
  src/lib/comprobantes.ts   formato y hash de comprobantes
scripts/                Cuentas de testnet, despliegue, simulación de jornada
docs/                   Bases del hackathon, notas de Stellar, decisiones
idea/                   Resumen de la idea
pitch/                  Guion y material del video
entrega/checklist.md    Checklist de entrega
PROYECTO.md             Este documento
```

---

## 6. Especificación del contrato `eleccion`

### 6.1 Tipos

```rust
#[contracttype]
pub enum Estado { Preparacion, Abierta, Cerrada }

#[contracttype]
pub struct Lote {
    pub numero: u32,        // 1, 2, 3… por mesa
    pub huella: BytesN<32>, // SHA-256 de los comprobantes del lote (ver 7.2)
    pub total: u32,         // cantidad de votos del lote
    pub ledger: u32,        // ledger en que se registró
}

#[contracttype]
pub enum Clave {
    Admin,
    Nombre,
    Estado,
    Candidaturas,              // Vec<Symbol>
    Mesas,                     // Vec<u32>
    Maquina(Address),          // -> u32 (mesa)
    MaquinaDeMesa(u32),        // -> Address
    Conteo(u32, Symbol),       // (mesa, candidatura) -> u32
    Total(Symbol),             // candidatura -> u32
    Lotes(u32),                // mesa -> Vec<Lote>
}

#[contracterror]
pub enum Error {
    YaInicializada = 1,
    NoInicializada = 2,
    EstadoInvalido = 3,
    MaquinaNoAutorizada = 4,
    MaquinaYaRegistrada = 5,
    MesaYaTieneMaquina = 6,
    CandidaturaInvalida = 7,
    LoteVacio = 8,
}
```

Candidaturas: `Symbol` cortos en mayúsculas, por ejemplo `A`, `B`, `C`, `BLANCO`.

### 6.2 Funciones

| Función | Firma requerida | Estado permitido | Efecto |
|---|---|---|---|
| `inicializar(admin: Address, nombre: String, candidaturas: Vec<Symbol>)` | `admin` | sin inicializar | Guarda datos, estado = Preparacion |
| `registrar_maquina(maquina: Address, mesa: u32)` | admin | Preparacion | Asocia máquina↔mesa (1 máquina por mesa) |
| `abrir()` | admin | Preparacion | Estado = Abierta; evento `abierta` |
| `cerrar()` | admin | Abierta | Estado = Cerrada; evento `cerrada` |
| `registrar_lote(maquina: Address, votos: Map<Symbol, u32>, huella: BytesN<32>)` | `maquina` | Abierta | Valida máquina y candidaturas, suma a `Conteo(mesa, c)` y `Total(c)`, agrega `Lote`, evento `lote` |
| `estado() -> Estado` | — | cualquiera | Lectura |
| `info() -> (String, Vec<Symbol>, Vec<u32>)` | — | cualquiera | Nombre, candidaturas, mesas |
| `resultados() -> Map<Symbol, u32>` | — | cualquiera | Totales |
| `resultados_mesa(mesa: u32) -> Map<Symbol, u32>` | — | cualquiera | Conteo de una mesa |
| `lotes_mesa(mesa: u32) -> Vec<Lote>` | — | cualquiera | Historial de lotes de la mesa |

Reglas invariables:

- **No existe ninguna función para restar, editar o borrar votos.**
- **El contrato no incluye función de actualización de código** (no se puede cambiar
  después de desplegado).
- `registrar_lote` falla con `LoteVacio` si la suma de votos es 0, y con
  `CandidaturaInvalida` si trae una candidatura no registrada.
- Los resultados de lectura incluyen todas las candidaturas, en 0 si no tienen votos.

### 6.3 Eventos

| Tópicos | Datos |
|---|---|
| `("abierta",)` | — |
| `("cerrada",)` | — |
| `("lote", mesa: u32)` | `(numero: u32, huella: BytesN<32>, total: u32)` |

### 6.4 Almacenamiento

- `instance`: Admin, Nombre, Estado, Candidaturas, Mesas.
- `persistent`: el resto. Extender TTL al escribir para que no expire durante la demo.

### 6.5 Tests mínimos (`cargo test`)

1. Flujo completo: inicializar → registrar 2 máquinas → abrir → lotes → cerrar →
   resultados correctos por mesa y totales.
2. Lote de máquina no registrada → `MaquinaNoAutorizada`.
3. Lote antes de abrir o después de cerrar → `EstadoInvalido`.
4. Registrar máquina con la votación abierta → `EstadoInvalido`.
5. Candidatura desconocida → `CandidaturaInvalida`.
6. Lote vacío → `LoteVacio`.
7. Sin firma del admin en `abrir` → falla la autorización.
8. `inicializar` dos veces → `YaInicializada`.

---

## 7. Especificación de la app web

Una sola app con cuatro rutas. Lee configuración de variables de entorno (ver
`.env.example`): red, RPC, passphrase y `CONTRACT_ID`.

### 7.1 `/quiosco` (computadora de votación)

- Configuración de la máquina: número de mesa y **llave secreta de testnet** de esa
  mesa, cargadas desde variables de entorno locales o un formulario de configuración
  guardado en `localStorage` (aceptable solo por ser demo en testnet).
- Pantalla grande y simple: nombre de la elección, botones por candidatura, pantalla de
  confirmación ("¿Confirma su voto por B?").
- Al confirmar: genera y muestra el **comprobante** (ver 7.2) con instrucción "Revise y
  deposite en la urna".
- Acumula votos en memoria (persistidos en `localStorage` para no perderlos al recargar).
- Cada **5 votos** (configurable): mezcla, calcula conteos y huella, llama
  `registrar_lote` firmado con la llave de la mesa, muestra estado discreto
  ("Lote 3 registrado") sin revelar votos.
- Botón de personal de mesa "Enviar votos pendientes" para vaciar el búfer antes del
  cierre.
- Si falla la red: reintenta; los votos no se pierden.
- **Modo alterado para la demo** (`/quiosco?alterada=1`): el comprobante muestra lo que
  eligió la persona, pero al contrato se envía siempre `A`. Sirve para demostrar que la
  auditoría lo detecta.

### 7.2 Formato del comprobante y huella del lote

Comprobante (texto):

```
M{mesa}-L{lote}-{candidatura}-{codigo}
ej.: M12-L03-B-7KQ2
```

- `codigo`: 4 caracteres aleatorios `[A-Z2-9]` (sin 0/O/1/I).
- El número de lote se conoce al emitir el comprobante (lote en curso de la máquina).
- **Huella del lote** = SHA-256 de los comprobantes del lote **ordenados
  alfabéticamente y unidos con `\n`**, sin salto final. En bytes UTF-8. 32 bytes.

Esto permite que la auditoría recalcule la huella solo con los papeles.

### 7.3 `/resultados` (tablero público)

- Totales por candidatura (barras) y tabla por mesa.
- Estado de la elección (Preparación / Abierta / Cerrada).
- Se actualiza cada ~5 s (polling de lecturas o eventos).
- Enlace al contrato en stellar.expert:
  `https://stellar.expert/explorer/testnet/contract/{CONTRACT_ID}`.
- Lista de lotes recientes con enlace a su transacción si está disponible.

### 7.4 `/admin`

- Muestra estado y mesas registradas.
- Formulario para registrar máquina (dirección `G…` + mesa).
- Botones Abrir y Cerrar con confirmación.
- Firma con la llave del admin (en la demo, desde configuración local o Freighter).

### 7.5 `/auditoria`

- Elegir mesa; pegar los comprobantes de la urna (uno por línea).
- La app agrupa por lote, recalcula huellas y conteos, y los compara con
  `lotes_mesa(mesa)` y `resultados_mesa(mesa)`.
- Resultado visual claro: ✔ "Coincide" o ✘ "No coincide" con el detalle (qué lote,
  qué candidatura, diferencia).

---

## 8. Scripts

| Script | Qué hace |
|---|---|
| `scripts/setup-testnet.sh` | Crea identidades `admin`, `mesa1`, `mesa2`, `mesa3` con `stellar keys generate --network testnet --fund` |
| `scripts/deploy.sh` | Compila, despliega, inicializa (candidaturas A, B, C, BLANCO), registra las 3 máquinas, escribe `CONTRACT_ID` en `app/.env.local` y regenera bindings |
| `scripts/simular-jornada.ts` | Opcional: genera votos aleatorios por mesa para poblar el tablero |

---

## 9. Entorno de desarrollo

```bash
# Rust y target de contratos
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32v1-none

# Stellar CLI
cargo install --locked stellar-cli

# Flujo del contrato
stellar contract build
stellar contract deploy --wasm target/wasm32v1-none/release/eleccion.wasm \
  --source admin --network testnet
stellar contract invoke --id <CONTRACT_ID> --source admin --network testnet -- abrir
stellar contract bindings typescript --id <CONTRACT_ID> --network testnet \
  --output-dir app/src/contracts/eleccion

# App
cd app && npm install && npm run dev
```

Referencias:

- Contratos: https://developers.stellar.org/docs/build/smart-contracts/overview
- Primeros pasos: https://developers.stellar.org/docs/build/smart-contracts/getting-started
- Ejemplos oficiales: https://github.com/stellar/soroban-examples
- SDK JS: https://github.com/stellar/js-stellar-sdk
- ZK en Stellar (trabajo futuro): https://developers.stellar.org/docs/build/apps/zk

---

## 10. Guion de la demo (video de 3 minutos máximo)

| Tramo | Duración | Qué se muestra |
|---|---|---|
| Problema | 0:00–0:25 | Confiar en quien cuenta; conteo manual, actas a mano, escrutinio de días |
| Solución | 0:25–0:45 | Máquinas auditadas en la mesa + conteo en Stellar + comprobante en urna |
| Demo: votar | 0:45–1:20 | Voto en el quiosco, comprobante, lote enviado |
| Demo: verificar | 1:20–1:45 | Tablero se actualiza; la transacción en stellar.expert |
| Demo: reglas | 1:45–2:05 | Máquina no autorizada o elección cerrada → Stellar rechaza |
| Demo: auditoría | 2:05–2:35 | Mesa normal ✔; mesa con máquina alterada ✘ detectada |
| Por qué Stellar y futuro | 2:35–3:00 | Contrato inmutable, ~5 s, costo ínfimo, verificable por cualquiera; ZK para ocultar parciales; cooperativas, sindicatos, gobiernos estudiantiles como primer mercado |

---

## 11. Plan de trabajo

| Fecha | Objetivo |
|---|---|
| 30 sep | Entorno listo; contrato con tests pasando; desplegado en testnet |
| 1 oct | Bindings; quiosco votando contra testnet; panel admin |
| 2 oct | Tablero de resultados; auditoría; modo alterado |
| 3 oct | Integración, pulido de UI, demo pública, README final, repo público |
| 4 oct | Grabación y edición del video |
| 5 oct | Revisión final y **entrega antes de las 4:00 p. m.** (idealmente en la mañana) |

División sugerida para dos personas:

- **Persona 1**: contrato, tests, scripts de despliegue, bindings.
- **Persona 2**: app web (quiosco, tablero, admin, auditoría) contra los bindings.

El punto de encuentro entre ambas partes es la especificación de la sección 6. La
persona de la app puede empezar con datos simulados que respeten esas firmas antes de
que el contrato esté desplegado.

---

## 12. Pendientes y decisiones abiertas

- [ ] Nombre del proyecto
- [ ] Repositorio público en GitHub (quién lo crea, nombre)
- [ ] Registro de todo el equipo en Stellar Passport y creación del equipo
- [ ] Dónde se publica la demo (sitio estático)
- [ ] Quién graba y edita el video
- [ ] Asistencia al encuentro presencial en el TEC
