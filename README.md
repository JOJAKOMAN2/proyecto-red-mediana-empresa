# Proyecto de red para mediana empresa (PNETLab)

Red empresarial redundante con segmentación por VLANs, HSRP, EtherChannel, Rapid-PVST+, OSPFv2 y ACLs.
Implementada en PNETLab con imágenes IOL (L2 para los switches, L3 para los routers).

## Integrantes

| Nombre | Matrícula |
| --- | --- |
| José Jadiel Gomez | _por completar_ |
| _por completar_ | _por completar_ |

## Topología

- `topologia/topologia-pnetlab.png`: captura de la topología en PNETLab.
- `topologia/topologia-logica.png`: diagrama lógico con los puertos.

## Conexiones

| Enlace | Extremo A | Extremo B | Función |
| --- | --- | --- | --- |
| CORE1 – CORE2 | CORE1 e0/1 y e0/2 | CORE2 e0/1 y e0/2 | EtherChannel Po1 (LACP) |
| CORE1 – SW-ACC1 | CORE1 e0/3 | SW-ACC1 e0/0 | Troncal 802.1Q |
| CORE1 – SW-ACC2 | CORE1 e1/0 | SW-ACC2 e0/3 | Troncal 802.1Q |
| CORE2 – SW-ACC1 | CORE2 e1/0 | SW-ACC1 e0/3 | Troncal 802.1Q |
| CORE2 – SW-ACC2 | CORE2 e0/3 | SW-ACC2 e0/0 | Troncal 802.1Q |
| CORE1 – R-EDGE | CORE1 e0/0 | R-EDGE e0/2 | Enrutado 10.0.0.0/30 |
| CORE2 – R-EDGE | CORE2 e0/0 | R-EDGE e0/1 | Enrutado 10.0.0.4/30 |
| R-EDGE – R-ISP | R-EDGE e0/0 | R-ISP e0/0 | Enrutado 200.0.0.0/30 |
| PC VENTAS | SW-ACC1 e0/1 | eth0 | VLAN 10 |
| PC TI | SW-ACC1 e0/2 | eth0 | VLAN 20 |
| PC FINANZAS | SW-ACC2 e0/2 | eth0 | VLAN 30 |
| PC GERENCIA | SW-ACC2 e0/1 | eth0 | VLAN 40 |

## Plan de direccionamiento

| VLAN | Nombre | Red | Gateway virtual (HSRP) | CORE1 | CORE2 | HSRP activo / root STP |
| --- | --- | --- | --- | --- | --- | --- |
| 10 | VENTAS | 192.168.10.0/24 | 192.168.10.1 | .2 | .3 | CORE1 |
| 20 | TI | 192.168.20.0/24 | 192.168.20.1 | .2 | .3 | CORE2 |
| 30 | FINANZAS | 192.168.30.0/24 | 192.168.30.1 | .2 | .3 | CORE1 |
| 40 | GERENCIA | 192.168.40.0/24 | 192.168.40.1 | .2 | .3 | CORE2 |
| 99 | GESTION | 192.168.99.0/24 | 192.168.99.1 | .2 | .3 | CORE1 |

| Enlace | Red | Direcciones |
| --- | --- | --- |
| CORE1 – R-EDGE | 10.0.0.0/30 | CORE1 .1, R-EDGE .2 |
| CORE2 – R-EDGE | 10.0.0.4/30 | CORE2 .5, R-EDGE .6 |
| R-EDGE – R-ISP | 200.0.0.0/30 | R-EDGE .1, R-ISP .2 |
| Internet simulado | 8.8.8.8/32 | R-ISP Loopback0 |

PCs: `192.168.X.10` (X = número de VLAN), gateway `192.168.X.1`.
SW-ACC1: `192.168.99.11`. SW-ACC2: `192.168.99.12`.
Router IDs OSPF: CORE1 `1.1.1.1`, CORE2 `2.2.2.2`, R-EDGE `3.3.3.3`.

## Política de ACLs

- VENTAS no puede acceder a FINANZAS, GERENCIA ni GESTION (`ACL_VENTAS`, SVI VLAN 10, entrada).
- FINANZAS no puede acceder a VENTAS ni GESTION (`ACL_FINANZAS`, SVI VLAN 30, entrada).
- TI y GERENCIA tienen acceso total; `ACL_VENTAS` deja pasar el retorno de las sesiones iniciadas por GERENCIA.
- Todas las VLANs pueden salir a Internet.

## Configuraciones

Un archivo por equipo en `configs/`. Cada uno se pega completo en la consola del equipo.

| Archivo | Equipo |
| --- | --- |
| `configs/SW-ACC1.txt` | Switch de acceso 1 |
| `configs/SW-ACC2.txt` | Switch de acceso 2 |
| `configs/CORE1.txt` | Switch core 1 (L3) |
| `configs/CORE2.txt` | Switch core 2 (L3) |
| `configs/R-EDGE.txt` | Router de borde |
| `configs/R-ISP.txt` | Router del ISP |
| `configs/PCs-VPCS.txt` | Las cuatro PCs (VPCS) |

Orden recomendado: switches de acceso, CORE1 y CORE2, R-EDGE y R-ISP, y por último las PCs.

## Pruebas

La lista de comandos y los resultados esperados está en `pruebas/pruebas-conectividad.txt`.
