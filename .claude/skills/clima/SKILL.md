---
name: clima
description: Previsión del tiempo local de una ciudad (por defecto Sant Cugat del Vallès). Usar cuando se pida el clima, tiempo, previsión o temperatura de un lugar. Argumento opcional - nombre de la ciudad.
argument-hint: "[ciudad]"
allowed-tools: Bash(curl *)
---

# Clima

Previsión del tiempo para una ciudad. Ciudad = `$ARGUMENTS`; si vacío, usar **Sant Cugat del Vallès**.

## Pasos

1. Determinar ciudad. Sin argumento → `Sant Cugat del Vallès`.
2. Consultar wttr.in (JSON, sin API key). Codificar espacios como `+` y acentos en URL:

   ```bash
   curl -s "https://wttr.in/Sant+Cugat+del+Valles?format=j1&lang=es"
   ```

   Sustituir la ciudad por la pedida (ej. `Madrid`, `New+York`).
3. Si la respuesta está vacía, no es JSON o la ciudad no existe, probar sin acentos o añadir país (`Valencia,Spain`). Si sigue fallando, avisar al usuario.
4. Del JSON usar:
   - `current_condition[0]`: `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `winddir16Point`, `precipMM`, `lang_es[0].value` (o `weatherDesc[0].value`).
   - `weather[0..2]`: `date`, `mintempC`, `maxtempC`, `hourly[*].chanceofrain` (máx.), `hourly[4].lang_es[0].value` (condición a mediodía).
   - `nearest_area[0]`: `areaName`, `country` para confirmar el lugar resuelto.

## Formato de salida (español, conciso)

```
📍 <ciudad>, <país> — ahora
<condición>, <temp>°C (sensación <fl>°C) · humedad <h>% · viento <v> km/h <dir>

Próximos 3 días
- <fecha>: <min>–<max>°C, <condición>, lluvia <p>%
- ...
```

Añadir una línea final con consejo práctico solo si procede (paraguas, abrigo, etc.).

## Notas

- Idioma de respuesta: español.
- Si el lugar resuelto no coincide con el pedido (otro país), mencionarlo.
