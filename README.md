# VLock

**Votación presencial con un conteo que nadie puede alterar.**

VLock reemplaza la papeleta por una computadora en cada mesa de votación. Cada voto
queda guardado en Stellar, una blockchain pública, donde nadie puede cambiarlo ni
borrarlo, y cualquier persona puede revisar el resultado.

Proyecto para **Find Your Way: Hackathon** (Stellar · Tellus Cooperative).

## El problema

Hoy, para confiar en el resultado de una elección hay que confiar en las personas que
cuentan los votos, llenan las actas y las transmiten. Contar todo a mano toma tiempo y
se presta a errores.

## Cómo funciona

1. **La persona llega a su mesa** y se identifica con la cédula, como siempre.
2. **Vota en una computadora** revisada y autorizada antes de la elección.
3. **Recibe un comprobante en papel**, revisa que diga lo que eligió y lo deposita en
   la urna. No se lo lleva, para que nadie pueda comprar ni exigir su voto.
4. **La computadora envía los votos a Stellar en grupos pequeños**, así nadie puede
   saber quién votó qué por la hora en que votó.
5. **Stellar suma los votos** y los guarda para siempre. Solo acepta votos de
   computadoras autorizadas y solo mientras la votación está abierta.
6. **Cualquiera puede ver el resultado** en tiempo real, sin tener que creerle a nadie.
7. **Al cierre se revisan unas pocas urnas al azar**: los papeles deben coincidir con lo
   que está en Stellar. Si una computadora hizo trampa, se descubre.

## Por qué Stellar

- **Nadie puede cambiar los votos**, ni siquiera quienes administran la elección. Las
  reglas están en un contrato inteligente que no se puede modificar.
- **No depende de un solo servidor**: el conteo está copiado en decenas de computadoras
  de distintas organizaciones en el mundo.
- **Es rápido y barato**: cada envío se confirma en unos 5 segundos y cuesta una
  fracción de centavo.
- **Es público**: cualquiera puede comprobar el resultado en
  [stellar.expert](https://stellar.expert).

## Qué incluye la demo

- **Quiosco**: la pantalla donde se vota.
- **Resultados**: el conteo en vivo, por mesa y total.
- **Administración**: autorizar computadoras, abrir y cerrar la votación.
- **Auditoría**: compara los comprobantes en papel con lo guardado en Stellar.

Todo funciona en la red de pruebas de Stellar (testnet).

## Tecnología

- Contrato inteligente en **Rust** con **Soroban** (Stellar).
- Aplicación web en **TypeScript** con **React**.

## Estructura

```
contracts/   Contrato inteligente
app/         Aplicación web
scripts/     Instalación y despliegue
docs/        Documentación
```


## Contrato en testnet

Pendiente de despliegue.

## Licencia

MIT
