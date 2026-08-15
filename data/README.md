🇬🇧 English | 🇪🇸 [Español](./README.es.md)

# TripleTen Data Analytics Bootcamp — Final Project

## Context
RappiPlus is a subscription service within the Rappi ecosystem, designed to increase purchase frequency and user lifetime value.

However, the business team isn't sure whether the service is actually meeting that goal.

Key open questions:
- Are users really buying more?
- Is the model generating profit?
- Are opportunities being lost somewhere in the purchase process?

## Project objective
Answer:
- Is the data reliable?
- Are we profitable?
- Where are users dropping off?
- Do users come back?
- Do the changes we test actually move the needle?
- How do we communicate all of this?

## Executive summary of results

| Question | Answer |
|---|---|
| Is the data reliable? | Yes, after cleaning (24,920 orders, zero nulls, probabilistic imputation on `dispositivo`, standardized dates and text) |
| Are we profitable? | Yes: 60.13% gross margin ($5.78M), 30.26% net margin ($2.91M) after marketing spend. But 18.19% of orders sell at a loss, concentrated in 3 products with no cost-based price floor (Laptop-Gaming-16GB: 54% of its orders sold at a loss) |
| Where are users dropping off? | In the conversion funnel, the biggest drop (13.29 pp) happens between `begin_checkout → add_payment_info`. Total cumulative conversion: 80.04% |
| Do users come back? | Retention holds steady between 40-44% across the 3 weeks after signup, with no meaningful difference between monthly cohorts (Jan–May 2025) |
| Do the changes we test move the needle? | The checkout UI redesign (A/B test, n=10,000) showed no statistically significant effect on conversion (p=0.416) or session duration (p=0.148) |
| How do we communicate all of this? | A 2-page Power BI dashboard (Executive Overview + Drill-through Detail view) |

Full methodology and findings in [`conclusiones_proyecto_final_rappiplus.md`](./conclusiones_proyecto_final_rappiplus.md) *(Spanish)*.

## Data dictionaries
The analysis starts from three source tables:

### Orders
Each row represents an order placed on the platform.
| Column | Data type | Description |
| --- | --- | --- |
| id_pedido | Categorical | Unique order ID |
| id_usuario | Categorical | ID of the user who placed the order |
| fecha_hora_pedido | Date | Timestamp when the order was placed |
| pais | Categorical | Country the order was placed from |
| dispositivo | Categorical | Device used to place the order |
| fuente_referencia | Categorical | User's acquisition channel |
| nombre_producto | Categorical | Name of the purchased product |
| categoria_producto | Categorical | Product category |
| Cantidad | Numeric | Quantity of items purchased |
| precio_unitario | Numeric | Unit price of the product |
| monto_descuento | Numeric | Discount applied to the order |
| monto_total | Numeric | Total amount paid for the order |

### Catalog
Each row represents a product available on the platform.
| Column | Data type | Description | Example |
| --- | --- | --- | --- |
| nombre_producto | Categorical | Product name | Laptop-Gaming-16GB |
| categoria_producto | Categorical | Category the product belongs to | Electronics |
| costo_unitario | Numeric | Unit cost of the product | 280.68 |
| proveedor | Categorical | Product's supplier | Fuller, Pena and Myers |

### Marketing spend
Each row represents a marketing investment in a specific country and channel.
| Column | Data type | Description | Example |
| --- | --- | --- | --- |
| fecha | Date | Date the investment was made | 2025-01-01 |
| pais | Categorical | Country the campaign ran in | Mexico |
| id_campaña | Categorical | Unique campaign ID | organic_Mexico |
| canal | Categorical | Marketing channel used | organic |
| gasto | Numeric | Amount invested in the campaign | 2446.25 |
