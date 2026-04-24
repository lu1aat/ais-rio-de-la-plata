# CLAUDE.md — posts/03-ais-vhf-propagacion

Micrositio de monitoreo en tiempo real. No es un artículo estático: la página se conecta a un broker MQTT local y actualiza el mapa y los cards sin recargar.

## Fuente de datos

- **Broker MQTT:** `192.168.68.64`, puerto WebSocket `9001`, sin autenticación
- **Tópico:** `ais/data/catcher`
- **Productor:** AIS-catcher corriendo en la misma red local

### Campo de longitud: `lon`, no `lng`

AIS-catcher emite los mensajes con el campo `"lon"` (no `"lng"`). Cualquier código que lea posición de los mensajes MQTT debe usar `msg.lon`.

### Campos relevantes del mensaje

```
mmsi         — identificador del buque
lat, lon     — posición (solo en mensajes tipo 1/2/3/18; ausente en tipo 5/24)
signalpower  — nivel de señal en dBm (ej: -12.69, -18.03)
rxtime       — timestamp UTC formato "YYYYMMDDHHmmss" (ej: "20260424021128")
shipname     — nombre del buque (solo en tipo 5/24, no siempre presente)
```

## Estaciones configuradas (stations.json)

Las estaciones fijas se cargan desde `stations.json`. Cada entrada tiene:
- `mmsi` — para matchear contra mensajes MQTT entrantes
- `score` — contribuye al índice de propagación cuando la estación está recibida
- `propagation_threshold` — umbral global (actualmente 50); si la suma de scores activos >= 50 → propagación activa

Scores actuales: ⚓ Punta Piedras = 50, ⚓ Faro Punta Piedras = 50, 🚢 DF19 Recalada = 1.

## Card "más lejano · 15 min"

Muestra el barco con mayor distancia al RX desde el último reset. Comportamiento:
- Cada mensaje con posición válida se compara contra `currentFurthest`
- Si no hay `currentFurthest` o el nuevo barco está más lejos → reemplaza y actualiza mapa + card
- **Reset cada 5 minutos** (`FURTHEST_RESET_MS`): `currentFurthest = null`, empieza selección desde cero
- Las balizas fijas configuradas en `stations.json` se excluyen de este cálculo
- La línea al mapa se anima con `stroke-dashoffset` (clase `link-animated`) y se colorea igual que la señal

## Score dinámico del barco más lejano

`PTS_PER_KM = 0.2` — cada km de distancia RX aporta 0.2 puntos al índice de propagación.
Ejemplo: barco a 80 km → 16 puntos. Se suma a los scores fijos en `updatePropPanel()`.
La barra del panel se clampea al 100% aunque el score total supere `propMaxScore`.

## Colores de señal (signalpower dBm)

| Rango      | Color     |
|------------|-----------|
| > −20      | `#2ecc71` verde  |
| −20 a −35  | `#f39c12` naranja |
| < −35      | `#e74c3c` rojo   |
