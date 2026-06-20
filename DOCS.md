# vivapizza-panel — Documentación técnica

## Descripción
Panel web estático de una sola página (SPA) para visualizar y analizar ventas de Viva Pizza. Consume el proxy `vivapizza-proxy` para obtener datos de ambas sucursales (Armenia e Izalco) y los presenta en dos paneles interactivos.

## Tecnologías
- **HTML5 + CSS3 + JavaScript vanilla** (sin frameworks)
- **Chart.js 4.4.1** (vía CDN de cdnjs) para gráficas
- Desplegado en **Render** como sitio estático

## Estructura del proyecto
```
vivapizza-panel/
├── index.html     ← Aplicación completa (HTML + CSS + JS en un solo archivo)
└── README.md
```

## Paneles disponibles

### ⚖️ Comparador de períodos
Compara totales de ventas entre dos períodos libremente configurables.

- **Período A:** rango de fechas libre con accesos rápidos (Hoy, Ayer, Esta semana, Este mes)
- **Período B:** se sincroniza automáticamente al mismo rango del año anterior; editable manualmente
- Muestra resultados para 🔴 Armenia y 🟠 Izalco en paralelo
- Cada sucursal muestra: total A (azul), total B (amarillo), diferencia absoluta y variación porcentual con indicador ▲▼
- Total combinado de ambas sucursales al fondo

**Flujo de datos:**
```
Selección de fechas
      │
      ▼
4 llamadas en paralelo (Promise.allSettled):
  /armenia/orden-trabajo/total?from=...&to=...  (Período A)
  /armenia/orden-trabajo/total?from=...&to=...  (Período B)
  /izalco/orden-trabajo/total?from=...&to=...   (Período A)
  /izalco/orden-trabajo/total?from=...&to=...   (Período B)
      │
      ▼
Renderizado de tarjetas y diferencias
```

### 📊 Gráficas de ventas
Visualiza la evolución de ventas en el tiempo con gráficas combinadas (barras + línea).

**Agrupaciones disponibles:**
| Agrupación | Controles requeridos | Notas |
|-----------|---------------------|-------|
| Diario    | Desde / Hasta       | Un punto por día |
| Semanal   | Desde / Hasta       | Un punto por semana ISO |
| Mensual   | Año                 | 12 puntos, uno por mes |
| Anual     | Año inicio / Año fin| Un punto por año |

**Sucursales:** Armenia (barras rojas), Izalco (barras naranjas), Combinado (línea negra). Cada una se puede activar/desactivar individualmente.

**Endpoint consumido:** `GET /[sucursal]/ventas?agrupacion=...&from=...&to=...`

## Relación con vivapizza-proxy
El panel **no se comunica directamente** con las APIs de sucursal. Toda comunicación pasa por el proxy:

```
vivapizza-panel (navegador)
        │
        │ fetch('https://vivapizza-proxy.onrender.com/...')
        ▼
vivapizza-proxy
        │
        ├──► api-vivapizza-armenia.ajkcloud.work
        └──► api-vivapizza-izalco.ajkcloud.work
```

La URL del proxy está definida como constante al inicio del script:
```javascript
const PROXY = 'https://vivapizza-proxy.onrender.com';
```

**Si el proxy cambia de URL**, solo hay que actualizar esa constante.

## Lógica de fechas
El panel aplica `+1 día` al parámetro `to` antes de enviarlo al proxy, para que el rango sea inclusivo:
```javascript
function addDays(iso, n) { ... }
// Ejemplo: to = '2026-06-19' → se envía '2026-06-20' al proxy
```

## Despliegue
- Repositorio: `edwinfloresv/vivapizza-panel`
- URL producción: `https://vivapizza-panel.onrender.com`
- Render redespliega automáticamente en cada push a `main`
- No requiere servidor ni build — es HTML estático puro

## Uso local
Basta con abrir `index.html` en el navegador. No requiere servidor local ya que consume el proxy desplegado en producción.
