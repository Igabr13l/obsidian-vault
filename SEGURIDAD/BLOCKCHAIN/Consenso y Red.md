---
title: "Ataques a Nivel Consenso y Red"
type: note
status: active
tags:
  - seguridad
  - blockchain
  - consenso
  - 51-percent-attack
  - double-spend
  - red-p2p
aliases:
  - Consensus Attacks
  - 51% Attack
  - Double Spending
  - Eclipse Attack
created: 2026-09-07
updated: 2026-09-07
source: "Papers fundamentales de consenso (Nakamoto, Buterin), incidentes de ETC y Bitcoin Gold"
---

# Ataques a Nivel Consenso y Red

> [!INFO] Fuente
> Análisis de vulnerabilidades y vectores de explotación dirigidos a los protocolos de consenso distribuido (Proof of Work, Proof of Stake) y a la topología de la red P2P subyacente.

---

## 1. El Ataque del 51% (51% Attack)

Ocurre cuando una entidad o grupo coordinado logra controlar más del 50% de la tasa de cómputo (*hashrate*) en Proof of Work o más de la mitad del capital en participación (*stake*) en Proof of Stake.

```mermaid
graph LR
    subgraph Cadena Pública Legítima
        A1["Bloque 100"] --> A2["Bloque 101"] --> A3["Bloque 102 (Pago a Exchange)"]
    end
    subgraph Cadena Privada del Atacante (Mayor Hashrate)
        A1 --> B1["Bloque 101"] --> B2["Bloque 102 (Fondos devueltos al atacante)"] --> B3["Bloque 103"] --> B4["Bloque 104"]
    end
    B4 -.->|Atacante publica cadena más larga| Reorg["Reorganización Forzada: Cadena Pública Descartada"]
```

### Lo que un Ataque del 51% PUEDE hacer
* **Doble Gasto (Double Spend)**: Revertir transacciones propias confirmadas recientemente.
* **Censura de Transacciones**: Negarse a incluir transacciones de direcciones específicas.
* **Monopolio de Bloques**: Bloquear a otros mineros/validadores de generar recompensas.

### Lo que un Ataque del 51% NO PUEDE hacer
* **NO puede robar fondos de billeteras ajenas** sin las claves privadas correspondientes (las firmas criptográficas siguen siendo inviolables).
* **NO puede crear monedas de la nada** fuera de las reglas del consenso (los nodos completos honestos rechazarían bloques inválidos).
* **NO puede modificar transacciones históricas antiguas** que requieran reescribir meses de trabajo computacional acumulado.

---

## 2. Doble Gasto (Double Spending)

El objetivo primario de los ataques de consenso. El atacante transfiere criptomonedas a un comercio o exchange a cambio de dinero fiduciario u otra criptomoneda líquida:

```
1. Depositar 1.000 monedas en Exchange A (en cadena pública)
2. Esperar confirmaciones mínimas del Exchange
3. Retirar activos estables / fiat del Exchange
4. Publicar rama privada más larga que no incluye el depósito inicial
5. Las 1.000 monedas vuelven a la billetera del atacante en la cadena reorganizada
```

---

## 3. Eclipse Attacks (Ataques de Eclipse)

Ataque dirigido contra un nodo específico dentro de la red P2P (por ejemplo, el nodo de un exchange o un oráculo):

```
       [Nodo Malicioso 1]
               │
[Nodo Malicioso 4] ── [NODO VÍCTIMA (Exchange)] ── [Nodo Malicioso 2]
               │
       [Nodo Malicioso 3]
```

* El atacante monopoliza todas las conexiones entrantes y salientes del nodo víctima con direcciones IP bajo su control.
* El nodo víctima queda aislado del resto de la red global y solo recibe los bloques y transacciones que el atacante decide mostrarle.
* Permite ejecutar ataques de doble gasto con una fracción muy reducida de hashrate, engañando a la víctima haciéndole creer que su transacción tiene múltiples confirmaciones.

---

## 4. Ataques de Gobernanza On-Chain (Sybil Governance Takeover)

En protocolos donde las actualizaciones de código se deciden mediante votaciones de tokens (DAOs), los atacantes pueden acumular poder de voto temporal:
* Comprar masivamente tokens en pools con poca liquidez.
* Solicitar flash loans para votar en una propuesta maliciosa en el mismo bloque si la votación no utiliza snapshots históricos.

---

## Casos Reales Donde Ocurrió

### 1. Ethereum Classic (ETC — 2019 y 2020)
* **Vector**: Ataques reiterados del 51% y reorganizaciones profundas.
* **Mecanismo**: Como ETC utilizaba el mismo algoritmo de minería que Ethereum (Ethash) pero representaba menos del 3% del hashrate global, los atacantes alquilaron hashrate en servicios como NiceHash. En agosto de 2020 ejecutaron reorganizaciones de hasta 7.000 bloques, generando doble gasto de millones de dólares en exchanges.

### 2. Bitcoin Gold (BTG — Mayo 2018 y Enero 2020)
* **Vector**: 51% hashrate attack contra exchanges.
* **Mecanismo**: Reversión de depósitos en exchanges extranjeros tras retirar los activos canjeados, sustrayendo más de $18M en 2018 y $70.000 adicionales en 2020.

### 3. Tornado Cash Governance Hack (Mayo 2023)
* **Vector**: Propuesta de gobernanza maliciosa encubierta.
* **Mecanismo**: El atacante presentó una propuesta formal que aparentaba ser idéntica a una anterior, pero incluyó una función oculta con `selfdestruct` y actualización de lógica. Una vez aprobada, ejecutó la función oculta para otorgarse 1.2 millones de votos falsos (más del quorum total), tomando el control absoluto de la tesorería del DAO y de los contratos del protocolo.

---

## ¿Dónde Podría Ocurrir Hoy? (Superficie de Ataque)

1. **Blockchains PoW con Algoritmos Comunes y Bajo Hashrate**:
   * Cualquier blockchain que use SHA-256 o Scrypt pero cuente con menos del 5% del hashrate de Bitcoin o Litecoin puede ser atacada alquilando potencia por pocas horas en NiceHash.
2. **Exchanges con Bajos Requisitos de Confirmación**:
   * Exchanges que acreditan fondos en cadenas menores con menos de 50 o 100 confirmaciones de bloque.
3. **Sistemas de Gobernanza DAO sin Snapshots de Saldo**:
   * DAOs que permiten votar con balances en tiempo real sin congelar los derechos de voto a una altura de bloque anterior (*snapshot block*).

---

## Contramedidas

* **Aumento de Requisitos de Confirmación**: Exchanges y custodios deben exigir cientos o miles de confirmaciones para monedas con bajo hashrate antes de liberar retiros.
* **Finalidad Determinística (Checkpoints)**: Implementar capas de finalización estricta (como Gasper en Ethereum PoS o checkpoints de Avalanche) donde bloques finalizados no puedan ser reorganizados matemáticamente.
* **Gobernanza con Retardo y Timelocks**: Toda propuesta aprobada por un DAO debe tener un periodo de bloqueo forzoso de 48 a 72 horas (*timelock*) antes de ejecutarse, permitiendo a la comunidad auditar los contratos finales.

---

## Ver también

- [[SEGURIDAD/BLOCKCHAIN/00 - INDICE]]
- [[SEGURIDAD/BLOCKCHAIN/Bridges y Cross-Chain]]
- [[SEGURIDAD/BLOCKCHAIN/DeFi y Oráculos]]
- [[SEGURIDAD/Fundamentos Redes]]
