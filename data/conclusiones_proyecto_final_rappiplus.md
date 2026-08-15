# Proyecto Final: Análisis de Datos RappiPlus 2025
## Conclusiones y Recomendaciones

---

## 1. Limpieza de datos

Se procesó el dataset crudo de pedidos, catálogo y marketing de RappiPlus, resolviendo nulos en `dispositivo` mediante imputación probabilística (`np.random.choice` ponderado por la distribución observada), estandarización de formatos de fecha y texto, y winsorización de outliers en montos. El dataset final de órdenes quedó con **24,920 filas sin nulos**, cubriendo 4 países (Argentina, Colombia, México y un segmento "Desconocido" de ~300 registros).

---

## 2. Análisis de rentabilidad

| Métrica | Valor |
|---|---|
| Ingresos totales | $9,616,878.36 |
| Costo total (COGS) | $3,834,533.21 |
| **Ganancia bruta** | **$5,782,345.15** (margen bruto: 60.13%) |
| Gasto total de marketing | $2,871,843.53 |
| **Ganancia neta (considerando marketing)** | **$2,910,501.62** (margen neto: **30.26%**) |

### Hallazgo crítico: ventas sistemáticas en pérdida

Al comparar el precio de venta implícito por pedido (`monto_total / cantidad`) contra el costo unitario del catálogo, se detectó que **18.19% de todos los pedidos se venden por debajo del costo**. Este problema no es transversal a todo el negocio, sino que se concentra en 3 productos de costo elevado que carecen de un piso de precio:

| Producto | Costo unitario | % de sus pedidos en pérdida |
|---|---|---|
| Laptop-Gaming-16GB | $280.68 | **54.07%** |
| Jacket-Winter-M | $189.31 | 36.61% |
| Blender-XL-Red | $176.64 | 33.86% |
| Tablet-Standard-64GB | $25.21 | 1.74% |
| Sneakers-Urban-42 | $17.21 | 0.75% |
| Vacuum-Pro-Black | $16.60 | 0.24% |
| Phone-Pro-128GB | $10.12 | 0.07% |

**Recomendación:** implementar un precio mínimo de venta ligado al costo unitario (ej. costo + margen mínimo garantizado) para Laptop-Gaming-16GB, Jacket-Winter-M y Blender-XL-Red — los tres productos que concentran casi la totalidad de las pérdidas por pedido.

---

## 3. Embudo de conversión

| Paso | Evento | Usuarios únicos | Conversión del paso | Conversión acumulada |
|---|---|---|---|---|
| 1 | first_visit | 7,796 | — | 100.00% |
| 2 | add_to_cart | 7,634 | 97.92% | 97.92% |
| 3 | select_item | 7,582 | 99.32% | 97.26% |
| 4 | begin_checkout | 7,208 | 95.07% | 92.46% |
| 5 | add_payment_info | 6,250 | **86.71%** | 80.17% |
| 6 | purchase | 6,240 | 99.84% | **80.04%** |

La mayor fuga del embudo ocurre entre **begin_checkout → add_payment_info**, con una pérdida de 958 usuarios (13.29 puntos porcentuales) — el único paso con una caída de conversión de un solo dígito distinto al resto, todos por encima de 95%. La conversión acumulada final de 80.04% es alta para un funnel de e-commerce; esto obedece en parte a la naturaleza sintética del dataset, pero el cuello de botella en el paso de pago es consistente y accionable.

**Recomendación:** priorizar la revisión del formulario de método de pago (`add_payment_info`) como el punto de mayor fricción del checkout.

---

## 4. Retención por cohortes

Se analizaron 5 cohortes mensuales (enero a mayo 2025), midiendo retención en las semanas 1, 2 y 3 posteriores al registro.

| Cohorte | Usuarios totales | Retención sem. 1 | Retención sem. 2 | Retención sem. 3 |
|---|---|---|---|---|
| 2025-01 | 1,627 | 42.84% | 41.06% | 40.32% |
| 2025-02 | 1,444 | 42.31% | 42.17% | 43.98% |
| 2025-03 | 1,636 | 41.38% | 43.09% | 42.18% |
| 2025-04 | 1,606 | 42.34% | 43.40% | 41.28% |
| 2025-05 | 1,687 | 41.20% | 40.07% | 41.85% |

La retención se mantiene **estable entre 40% y 44% en las 5 cohortes y en las 3 semanas medidas**, sin una curva de decaimiento marcada (a diferencia del patrón típico donde la retención cae progresivamente semana a semana). Esto sugiere que el usuario que sobrevive a la primera semana tiende a estabilizar su comportamiento, y que no hay una cohorte particular (por mes de adquisición) con mejor o peor desempeño que las demás.

**Recomendación:** dado que la retención no mejora ni empeora por cohorte, cualquier iniciativa de retención debería enfocarse en la ventana crítica de la semana 1 (donde ya se pierde ~57-59% de los usuarios) más que en diferenciar por mes de adquisición.

---

## 5. Test A/B: nuevo diseño de checkout

Se evaluó el impacto de un cambio de UI en el checkout sobre la tasa de conversión y la duración de sesión, comparando grupos control (n=4,965) y tratamiento (n=5,035), balanceados en dispositivo y país.

| Métrica | Resultado |
|---|---|
| Conversión control | 15.69% |
| Conversión tratamiento | 16.29% |
| Prueba z de proporciones | p = 0.416 (no significativo) |
| Duración de sesión (t-test) | p = 0.148 (no significativo) |

El cambio de UI **no mostró un impacto estadísticamente significativo** en la conversión ni en la duración de sesión, tanto a nivel global como al segmentar por dispositivo y país (todos los p-values > 0.05). Las diferencias más cercanas a significancia (mobile, México) son consistentes con variación aleatoria dado el tamaño de muestra reducido al segmentar.

**Recomendación:** no atribuir cambios de negocio a esta modificación de UI. Si se considera relevante seguir explorando el diseño, se necesitaría un experimento con mayor tamaño de muestra para detectar efectos pequeños con suficiente poder estadístico.

---

## 6. Dashboard

Se construyó un reporte de 2 páginas en Power BI sobre un modelo en esquema de estrella (tablas de hechos `Ordenes` y `hecho_marketing`, dimensiones `dim_pais`, `dim_canal`, `dim_fecha`, `dim_catalogo`):

- **Reporte Ejecutivo**: 5 KPIs (ingresos, ticket promedio, artículos por pedido, gasto de marketing, profit neto), desglose de ingreso/ganancia por categoría, producto y dispositivo, tendencia mensual acumulada, y filtro interactivo por país.
- **Detalle de Ventas**: tabla de pedidos con formato condicional (rojo/verde según ganancia), tendencia de unidades vendidas por mes, tabla de % de pedidos en pérdida por categoría, y navegación drill-through desde los gráficos de producto/categoría del reporte ejecutivo.

---

## 7. Síntesis de recomendaciones

1. Establecer un precio mínimo ligado al costo para Laptop-Gaming-16GB, Jacket-Winter-M y Blender-XL-Red — es la causa más concentrada y accionable de pérdida de margen.
2. Rediseñar o simplificar el paso `add_payment_info` del checkout, el mayor punto de fuga del embudo.
3. Enfocar los esfuerzos de retención en la primera semana post-registro, ya que no hay diferencias relevantes por cohorte de adquisición.
4. Descartar el rediseño de UI de checkout probado como palanca de conversión; no mostró efecto medible.
