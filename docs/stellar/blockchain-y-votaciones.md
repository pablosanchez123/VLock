# Cómo funciona una blockchain y cómo la usa el sistema de votación

## 1. Qué es una blockchain

Una blockchain es un **libro contable compartido**: una lista de registros que muchas
computadoras independientes guardan a la vez, que solo crece hacia adelante y que nadie
puede editar por su cuenta. Se apoya en cuatro ideas.

### 1.1 Huella digital (hash)

Un hash es una función que convierte cualquier dato en una "huella" de tamaño fijo, por
ejemplo SHA-256:

```
"Mesa 12: A=40, B=35"  →  9f2c…e71a
"Mesa 12: A=41, B=35"  →  04bd…3c90   (cambió un número y la huella es otra)
```

Es imposible, en la práctica, fabricar dos datos distintos con la misma huella o
reconstruir el dato a partir de la huella.

### 1.2 Bloques encadenados

Los registros se agrupan en bloques. Cada bloque incluye la huella del anterior:

```
[Bloque 100]        [Bloque 101]             [Bloque 102]
 datos               datos                    datos
 hash: 7a1f  ◄────── hash anterior: 7a1f ◄─── hash anterior: c93e
                     hash: c93e               hash: 55d0
```

Si alguien cambia un dato del bloque 100, su huella cambia, el bloque 101 deja de
coincidir, y así toda la cadena. Una alteración se detecta de inmediato. En Stellar a los
bloques se les llama **ledgers** y se cierra uno cada ~5 segundos.

### 1.3 Firmas digitales

Cada participante tiene un par de llaves:

- **Llave secreta** (en Stellar empieza con `S`): solo la tiene su dueño y sirve para firmar.
- **Llave pública** (empieza con `G`): la conoce todo el mundo y sirve para verificar.

Una transacción firmada prueba **quién la envió** y que **no fue modificada** en el
camino. Nadie puede enviar algo en nombre de otra llave sin tener su secreta. Stellar usa
el algoritmo Ed25519.

### 1.4 Consenso entre muchas computadoras

La cadena no vive en un servidor, sino en decenas de **validadores** operados por
organizaciones distintas (universidades, empresas, fundaciones) en varios países. Para
agregar un ledger, los validadores tienen que ponerse de acuerdo sobre su contenido.

Stellar usa el **Stellar Consensus Protocol (SCP)**: no hay minería ni gasto de energía;
cada validador declara en qué otros validadores confía (sus *quorum slices*) y un ledger
se acepta cuando esos grupos coinciden. Resultado:

- **Finalidad en ~5 s**: una vez cerrado el ledger, la transacción es definitiva (no hay
  "reorganizaciones" como en otras cadenas).
- Para alterar el pasado habría que corromper a la mayoría de validadores de
  organizaciones independientes a la vez, y aun así cualquiera que tenga una copia vería
  que las huellas no coinciden.

### 1.5 Contratos inteligentes

Un contrato inteligente es un **programa guardado en la blockchain**. Tiene su propio
almacenamiento y funciones que cualquiera puede llamar, pero sus reglas se cumplen sí o
sí porque todos los validadores lo ejecutan y verifican el mismo resultado. Nadie, ni
quien lo creó, puede saltarse sus reglas una vez desplegado.

En Stellar los contratos se llaman **Soroban**, se escriben en Rust y se compilan a
WebAssembly.

## 2. Piezas de Stellar que usa el sistema

| Pieza de Stellar | Uso en la votación |
|---|---|
| Cuentas y llaves (`G…`/`S…`) | Cada computadora de votación y el administrador electoral tienen su llave |
| `require_auth()` en Soroban | El contrato exige la firma de una computadora autorizada para aceptar votos |
| Almacenamiento del contrato | Candidaturas, computadoras autorizadas, estado de la elección, conteos |
| Eventos del contrato | Cada lote de votos emite un evento público que el tablero escucha |
| RPC de Soroban | La app lee y escribe en el contrato |
| Explorador (stellar.expert) | Cualquier persona audita las transacciones sin depender del sistema |
| Comisiones | Cada transacción cuesta fracciones de centavo; en testnet es gratis |

## 3. Diseño del sistema

### 3.1 Actores

- **Administración electoral**: crea la elección, registra las computadoras, abre y cierra.
- **Computadora de votación (quiosco)**: una por mesa, auditada, con su propia llave de
  Stellar. Es la única que puede registrar votos de su mesa.
- **Persona votante**: se identifica con la cédula ante la mesa, igual que hoy, y vota
  en la computadora. Nunca interactúa con Stellar directamente.
- **Público y fiscales**: consultan resultados y auditan.

### 3.2 Contrato `eleccion` (Soroban)

Estado:

- `admin`: llave de la administración electoral.
- `estado`: `Preparacion` → `Abierta` → `Cerrada`.
- `candidaturas`: lista de opciones (incluye blanco y nulo).
- `maquinas`: llave de cada computadora → número de mesa.
- `conteo_mesa`: (mesa, candidatura) → votos.
- `conteo_total`: candidatura → votos.
- `lotes`: por mesa, lista de huellas de lotes recibidos.

Funciones:

| Función | Quién puede | Qué hace |
|---|---|---|
| `inicializar(admin, candidaturas)` | una sola vez | Crea la elección |
| `registrar_maquina(maquina, mesa)` | admin, solo en Preparacion | Autoriza una computadora |
| `abrir()` / `cerrar()` | admin | Cambia el estado |
| `registrar_lote(maquina, votos, huella_comprobantes)` | la computadora, solo si está Abierta | Suma votos al conteo de su mesa y al total; guarda la huella; emite un evento |
| `resultados()` / `resultados_mesa(mesa)` | cualquiera | Lectura pública |

Reglas que el contrato hace cumplir sin excepción:

- Solo computadoras registradas antes de abrir pueden votar, y solo por su mesa.
- No se aceptan votos antes de abrir ni después de cerrar.
- Los conteos solo suben; no existe función para restar o editar.
- Nadie, tampoco el administrador, puede cambiar un voto ya registrado.

### 3.3 Por qué lotes y no voto por voto

Si cada voto llegara a Stellar al instante con su hora exacta, alguien que anote a qué
hora votó cada persona podría deducir su voto. La computadora acumula votos y envía un
lote cada N votos (por ejemplo 5) o al cierre, con el orden mezclado. Esto además
tolera cortes de internet: los votos quedan en la máquina hasta que se puedan enviar.

### 3.4 Comprobante en papel y auditoría

1. Al votar, la computadora imprime un comprobante con la opción elegida y un código
   aleatorio (sin datos de la persona). La persona lo revisa y lo deposita en la urna.
2. Al enviar un lote, la computadora calcula la huella SHA-256 de los códigos de ese
   lote y la registra en el contrato junto con los votos.
3. En la auditoría se cuentan los papeles de una mesa, se recalculan las huellas y se
   comparan con lo que está en Stellar. Si alguien alteró el software de la máquina, el
   papel y la cadena no coinciden y se detecta.

### 3.5 Recorrido de un voto

```
Persona ──cédula──► Mesa (padrón) ──habilita──► Computadora de la mesa 12
                                                   │ elige "B"
                                                   │ imprime comprobante B-7KQ2 → urna
                                                   ▼
                                        buffer local: [B, A, B, C, A]
                                                   │ al 5.º voto: mezcla, calcula huella,
                                                   │ firma con la llave de la mesa 12
                                                   ▼
                                  Stellar: contrato.registrar_lote(mesa 12, {A:2,B:2,C:1}, huella)
                                                   │ valida firma y estado, suma, emite evento
                                                   ▼
                          Tablero público y stellar.expert muestran el nuevo conteo (~5 s)
```

## 4. Componentes de la demo

| Componente | Carpeta | Tecnología |
|---|---|---|
| Contrato `eleccion` | `contracts/eleccion` | Rust + soroban-sdk |
| Quiosco de votación | `app/` (ruta `/quiosco`) | Web (TypeScript), guarda la llave de la mesa en la máquina |
| Tablero público | `app/` (ruta `/resultados`) | Lee el contrato vía RPC |
| Panel de administración | `app/` (ruta `/admin`) | Registrar máquinas, abrir, cerrar |
| Auditoría | `app/` (ruta `/auditoria`) | Ingresa códigos de papel y compara huellas |
| Scripts | `scripts/` | Crear cuentas de testnet, desplegar, simular una jornada |

## 5. Límites que hay que decir en el pitch

- La blockchain protege los votos **después** de registrados. La confianza en lo que
  pasa antes se apoya en máquinas auditadas y en el comprobante en papel.
- El control de "una persona, un voto" sigue en la mesa con el padrón.
- En una versión posterior, las pruebas de conocimiento cero (BN254 y Poseidon, activas
  en Stellar desde Protocol 25 "X-Ray", enero 2026) permitirían ocultar incluso los
  conteos parciales por mesa hasta el cierre.
