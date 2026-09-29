# VLock: votación presencial con conteo en Stellar

## Problema

En una votación, la confianza en el resultado depende de que nadie altere actas ni
conteos entre la mesa y el resultado final, y de que el conteo sea rápido. Hoy eso
requiere confiar en quien custodia y transmite los datos.

## Solución

Computadoras de votación auditadas en los centros de votación. La persona se identifica
en la mesa como siempre y, en vez de marcar un papel, elige en la computadora. Cada
computadora firma los votos con su llave y los registra en un contrato de Stellar, donde
no se pueden alterar y cualquiera puede verificar el conteo en tiempo real.

## Por qué Stellar

- Contrato Soroban que hace cumplir las reglas: solo máquinas autorizadas, solo con la
  votación abierta, conteos que solo suben.
- Registro público e inalterable, auditable en stellar.expert sin confiar en el sistema.
- Confirmación en ~5 s y costo de fracciones de centavo por transacción.

Detalle técnico: `docs/stellar/blockchain-y-votaciones.md`.

## Flujo de la demo

1. Administración crea la elección, registra dos o tres computadoras (mesas) y abre.
2. Se vota en el quiosco; se muestra el comprobante con código.
3. Los lotes llegan a Stellar; el tablero público se actualiza y se ve la transacción en
   stellar.expert.
4. Se intenta votar con una máquina no registrada o con la elección cerrada: el contrato
   lo rechaza.
5. Auditoría: se ingresan los códigos de papel de una mesa y coinciden con la huella en
   Stellar.

## Alcance del MVP (antes del 5 oct 2026)

Entra: contrato `eleccion`, quiosco, tablero, panel de administración, auditoría básica,
scripts de despliegue en testnet.

Queda fuera: impresión física real (se simula en pantalla), integración con el padrón,
pruebas de conocimiento cero, mainnet.
