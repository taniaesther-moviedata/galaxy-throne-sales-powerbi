# Medidas DAX

El modelo tiene **88 medidas**, todas en la tabla `#Tabla de medidas` y organizadas en carpetas. Esta página las recoge tal como están en `galaxy_throne_analisis_ventas.pbix`.

Convención usada en las correcciones: las ventas se calculan siempre sobre transacciones **cobradas** (`declined = 0`) y un **cliente activo** es el que tiene al menos una compra cobrada en el periodo filtrado.

## Índice

- [Correcciones de la segunda versión](#correcciones-de-la-segunda-versión) (23)
- [Metas](#metas) (4)
- [Valores de los medidores](#valores-de-los-medidores) (3)
- [Variación interanual de ventas](#variación-interanual-de-ventas) (2)
- [Ventas cobradas y rechazadas](#ventas-cobradas-y-rechazadas) (4)
- [Clientes B2C](#clientes-b2c) (23)
- [Abandono de clientes B2C](#abandono-de-clientes-b2c) (4)
- [Abandono de clientes B2B](#abandono-de-clientes-b2b) (3)
- [Producto](#producto) (3)
- [Color de producto](#color-de-producto) (4)
- [Foco Alemania](#foco-alemania) (3)
- [Otras medidas](#otras-medidas) (12)

## Correcciones de la segunda versión

Medidas añadidas al aplicar el feedback del Sprint 8: transacciones cobradas, clientes activos, normalización por cliente, facturación real por producto (unidades × precio), bloques temporales comparables, índices base 2020 y escenarios.

### % clientes

```dax
% clientes =
DIVIDE (
    [Clientes B2C activos],
    CALCULATE ( [Clientes B2C activos], REMOVEFILTERS ( 'user' ) )
)
```

### % del gasto del país cliente

```dax
% del gasto del país cliente =
DIVIDE (
    [Ventas_efectivas],
    CALCULATE ( [Ventas_efectivas], REMOVEFILTERS ( 'company' ) )
)
```

### % rechazo

```dax
% rechazo =
DIVIDE ( [Rechazos], COUNTROWS ( 'transaction' ) )
```

### % sobre total de rechazos

```dax
% sobre total de rechazos =
DIVIDE (
    [Rechazos],
    CALCULATE ( [Rechazos], REMOVEFILTERS ( 'user' ), REMOVEFILTERS ( 'company' ) )
)
```

### % trimestre sobre año

```dax
% trimestre sobre año =
// Para el mapa de calor año x trimestre (Nivel 2)
DIVIDE (
    [Ventas_efectivas],
    CALCULATE (
        [Ventas_efectivas],
        REMOVEFILTERS (
            'calendar'[Quarter], 'calendar'[quarter_num],
            'calendar'[month], 'calendar'[month_num]
        )
    )
)
```

### % trimestre sobre producto

```dax
% trimestre sobre producto =
// Para el mapa de calor: 25 % = venta repartida por igual en el año
DIVIDE (
    [Ventas producto],
    CALCULATE (
        [Ventas producto],
        REMOVEFILTERS ( 'calendar'[Quarter], 'calendar'[quarter_num] )
    )
)
```

### % variación bloque B vs A

```dax
% variación bloque B vs A =
DIVIDE (
    [Ventas bloque B (jul 2022 - dic 2024)] - [Ventas bloque A (ene 2020 - jun 2022)],
    [Ventas bloque A (ene 2020 - jun 2022)]
)
```

### Clientes B2C activos

```dax
Clientes B2C activos =
// Cliente activo = al menos una compra cobrada en el periodo filtrado
CALCULATE (
    DISTINCTCOUNT ( 'transaction'[user_id] ),
    'transaction'[declined] = 0
)
```

### Compras por cliente

```dax
Compras por cliente =
DIVIDE ( [Transacciones cobradas], [Clientes B2C activos] )
```

### Escenario +10% productos < 50 USD

```dax
Escenario +10% productos < 50 USD =
CALCULATE ( [Ventas producto], 'product'[price(USD)] < 50 ) * 0.10
```

### Rechazos

```dax
Rechazos =
CALCULATE ( COUNTROWS ( 'transaction' ), 'transaction'[declined] = 1 )
```

### Ticket medio

```dax
Ticket medio =
DIVIDE ( [Ventas_efectivas], [Transacciones cobradas] )
```

### Transacciones cobradas

```dax
Transacciones cobradas =
CALCULATE ( COUNTROWS ( 'transaction' ), 'transaction'[declined] = 0 )
```

### Unidades vendidas

```dax
Unidades vendidas =
CALCULATE (
    COUNTROWS ( 'transaction_product' ),
    'transaction'[declined] = 0
)
```

### Ventas bloque A (ene 2020 - jun 2022)

```dax
Ventas bloque A (ene 2020 - jun 2022) =
CALCULATE (
    [Ventas_efectivas],
    REMOVEFILTERS ( 'calendar' ),
    'calendar'[Date] >= DATE ( 2020, 1, 1 ),
    'calendar'[Date] <= DATE ( 2022, 6, 30 )
)
```

### Ventas bloque B (jul 2022 - dic 2024)

```dax
Ventas bloque B (jul 2022 - dic 2024) =
CALCULATE (
    [Ventas_efectivas],
    REMOVEFILTERS ( 'calendar' ),
    'calendar'[Date] >= DATE ( 2022, 7, 1 ),
    'calendar'[Date] <= DATE ( 2024, 12, 31 )
)
```

### Ventas por cliente

```dax
Ventas por cliente =
DIVIDE ( [Ventas_efectivas], [Clientes B2C activos] )
```

### Ventas producto

```dax
Ventas producto =
CALCULATE (
    SUMX ( 'transaction_product', RELATED ( 'product'[price(USD)] ) ),
    'transaction'[declined] = 0
)
```

### sum_total_transaction

```dax
sum_total_transaction =
SUM ( 'transaction'[amount(USD)] )
```

### Índice clientes activos

```dax
Índice clientes activos =
DIVIDE ( [Clientes B2C activos], CALCULATE ( [Clientes B2C activos], 'calendar'[year] = 2020 ) ) * 100
```

### Índice compras por cliente

```dax
Índice compras por cliente =
DIVIDE ( [Compras por cliente], CALCULATE ( [Compras por cliente], 'calendar'[year] = 2020 ) ) * 100
```

### Índice ticket medio

```dax
Índice ticket medio =
DIVIDE ( [Ticket medio], CALCULATE ( [Ticket medio], 'calendar'[year] = 2020 ) ) * 100
```

### Índice ventas

```dax
Índice ventas =
DIVIDE ( [Ventas_efectivas], CALCULATE ( [Ventas_efectivas], 'calendar'[year] = 2020 ) ) * 100
```

## Metas

Objetivos de negocio usados como líneas de referencia en el Sprint 7.

### meta anual

```dax
meta anual =
2600000
```

### meta_amount_mes

```dax
meta_amount_mes =
220000
```

### meta_media_transaccion

```dax
meta_media_transaccion =
260
```

### meta_num_companies

```dax
meta_num_companies =
3
```

## Valores de los medidores

Mínimos y máximos de los gráficos de medidor.

### V_max_media

```dax
V_max_media =
300
```

### V_min_media

```dax
V_min_media =
200
```

### alemania_V_max_medidor

```dax
alemania_V_max_medidor =
400
```

## Variación interanual de ventas

### %_Crecimiento_YoY

```dax
%_Crecimiento_YoY =
DIVIDE([suma_transactions] - [Ventas Año Anterior], [Ventas Año Anterior])
```

### Ventas Año Anterior

```dax
Ventas Año Anterior =
CALCULATE([suma_transactions], SAMEPERIODLASTYEAR(calendar[Date]))
```

## Ventas cobradas y rechazadas

### %_company_declined

```dax
%_company_declined =
Var Ventas_Declined =
CALCULATE(
    SUM('transaction'[amount(USD)]),
    'transaction'[declined] = 1
)

RETURN
DIVIDE(
    Ventas_Declined,
    SUM('transaction'[amount(USD)])
)
```

### %_total_transactions_company

```dax
%_total_transactions_company =
DIVIDE([sum_total_transaction_declined], CALCULATE([sum_total_transaction_declined], ALL(company)))
```

### Ventas_efectivas

```dax
Ventas_efectivas =
CALCULATE(
SUM('transaction'[amount(USD)]), 'transaction'[declined] = 0
)
```

### sum_total_transaction_declined

```dax
sum_total_transaction_declined =
CALCULATE(SUM('transaction'[amount(USD)]), 'transaction'[declined] = 1)
```

## Clientes B2C

Ticket medio, distribución del gasto (cuartiles para el diagrama de caja), frecuencia de compra y productos de mayor y menor precio por cliente.

### %_Crecimiento_usuarios_YoY

```dax
%_Crecimiento_usuarios_YoY =
DIVIDE([usuarios_anual] - [Usuarios Año Anterior], [Usuarios Año Anterior])
```

### Edad_ultima_transaccion

```dax
Edad_ultima_transaccion =
VAR UltimaFechaTransaccion =
    CALCULATE( 
        MAX('transaction'[timestamp]),
        ALL('transaction')
    )
RETURN
    DATEDIFF(
        SELECTEDVALUE('user'[birth_date]),
        UltimaFechaTransaccion,
        YEAR
    )
```

### Frecuencia de Compra

```dax
Frecuencia de Compra =
DIVIDE(
    COUNTROWS('transaction'),
    DISTINCTCOUNT('transaction'[user_id])
)
```

### IQR

```dax
IQR =
[p75_amount] - [p25_amount]
```

### IQR_25_50

```dax
IQR_25_50 =
[Mediana_ventas] - [p25_amount]
```

### IQR_50_75

```dax
IQR_50_75 =
[p75_amount] - [Mediana_ventas]
```

### Mediana_ventas

```dax
Mediana_ventas =
MEDIANX('transaction','transaction'[amount(USD)])
```

### NombreConEstrella

```dax
NombreConEstrella =
VAR Estrella = [top_user]
RETURN
    SELECTEDVALUE('user'[full_name]) & Estrella
```

### Usuarios Año Anterior

```dax
Usuarios Año Anterior =
CALCULATE([usuarios_anual], SAMEPERIODLASTYEAR(calendar[Date]))
```

### lower_whisker

```dax
lower_whisker =
[p25_amount] - (1.5 * [IQR])
```

### max_amount_transaction

```dax
max_amount_transaction =
MAX('transaction'[amount(USD)])
```

### max_precio_producto

```dax
max_precio_producto =
CALCULATE(
    MAX('product'[price(USD)]),
    TOPN(
        1,
        'transaction_product',
        MAX('product'[price(USD)]),
        DESC
    )
)
```

### media_amount

```dax
media_amount =
CALCULATE(AVERAGE('transaction'[amount(USD)]))
```

### media_amount_user

```dax
media_amount_user =
CALCULATE(AVERAGE('transaction'[amount(USD)]),'transaction'[user_id])
```

### min_amount_transaction

```dax
min_amount_transaction =
MIN('transaction'[amount(USD)])
```

### min_precio_producto

```dax
min_precio_producto =
CALCULATE(
    MIN('product'[price(USD)]),
    TOPN(
        1,
        'transaction_product',
        MIN('product'[price(USD)]),
        DESC
    )
)
```

### nombre_max_precio_producto

```dax
nombre_max_precio_producto =
VAR MaxPriceVal =
    CALCULATE(
        MAX('product'[price(USD)]),
        TOPN(
            1,
            'transaction_product',
            MAX('product'[price(USD)]),
            DESC
        )
    )
RETURN
    CALCULATE(
        MAX('product'[product_name]),
        'product'[price(USD)] = MaxPriceVal
    )
```

### nombre_min_precio_producto

```dax
nombre_min_precio_producto =
VAR MinPriceVal =
    CALCULATE(
        MIN('product'[price(USD)]),
        TOPN(
            1,
            'transaction_product',
            MIN('product'[price(USD)]),
            DESC
        )
    )
RETURN
    CALCULATE(
        MIN('product'[product_name]),
        'product'[price(USD)] = MinPriceVal
    )
```

### p25_amount

```dax
p25_amount =
PERCENTILE.INC('transaction'[amount(USD)],0.25)
```

### p75_amount

```dax
p75_amount =
PERCENTILE.INC('transaction'[amount(USD)], 0.75)
```

### top_user

```dax
top_user =
IF(
    [media_amount_euros] >= 300 && [media_amount_dolars] >= 320,
    "⭐",
    "☆" 
)
```

### upper_whisker

```dax
upper_whisker =
[p75_amount] + (1.5 * [IQR])
```

### usuarios_anual

```dax
usuarios_anual =
DISTINCTCOUNT('transaction'[user_id])
```

## Abandono de clientes B2C

Un cliente se considera perdido si no compra en los 6 meses anteriores a la última fecha del dataset.

### Churn Rate B2C

```dax
Churn Rate B2C =
DIVIDE(
    [Usuarios Perdidos],
    [Usuarios Activos] + [Usuarios Perdidos]
)
```

### Ultima Compra Usuario

```dax
Ultima Compra Usuario =
CALCULATE(
    MAX('transaction'[date_from_timestamp]),
    ALLEXCEPT('transaction', 'transaction'[user_id])
)
```

### Usuarios Activos

```dax
Usuarios Activos =
VAR FechaMaxima =
    CALCULATE(
        MAX('transaction'[date_from_timestamp]),
        REMOVEFILTERS('calendar')
    )
VAR Umbral = EDATE(FechaMaxima, -6)
VAR Activos =
    CALCULATE(
        COUNTROWS(
            FILTER(
                VALUES('transaction'[user_id]),
                CALCULATE(
                    MAX('transaction'[date_from_timestamp]),
                    REMOVEFILTERS('calendar')
                ) >= Umbral
            )
        ),
        REMOVEFILTERS('calendar')
    )
RETURN
IF(ISBLANK(Activos), 0, Activos)
```

### Usuarios Perdidos

```dax
Usuarios Perdidos =
VAR FechaMaxima =
    CALCULATE(
        MAX('transaction'[date_from_timestamp]),
        REMOVEFILTERS('calendar')
    )
VAR Umbral = EDATE(FechaMaxima, -6)
VAR Perdidos = 
    CALCULATE(
        COUNTROWS(
            FILTER(
                VALUES('transaction'[user_id]),
                CALCULATE(
                    MAX('transaction'[date_from_timestamp]),
                    REMOVEFILTERS('calendar')
                ) < Umbral
            )
        ),
        REMOVEFILTERS('calendar')
    )
RETURN
IF(ISBLANK(Perdidos), 0, Perdidos)
```

## Abandono de clientes B2B

Una compañía se considera perdida si no vende en los 12 meses anteriores a la última fecha del dataset.

### Churn Rate B2B

```dax
Churn Rate B2B =
DIVIDE(
    [Compañias Perdidas],
    [Compañias Activas] + [Compañias Perdidas]
)
```

### Compañias Activas

```dax
Compañias Activas =
VAR FechaMaxima =
    CALCULATE(
        MAX('transaction'[date_from_timestamp]),
        REMOVEFILTERS('calendar')
    )
VAR Umbral = EDATE(FechaMaxima, -12)
VAR Activas =
    CALCULATE(
        COUNTROWS(
            FILTER(
                VALUES('transaction'[company_id]),
                CALCULATE(
                    MAX('transaction'[date_from_timestamp]),
                    REMOVEFILTERS('calendar')
                ) >= Umbral
            )
        ),
        REMOVEFILTERS('calendar')
    )
RETURN
IF(ISBLANK(Activas), 0, Activas)
```

### Compañias Perdidas

```dax
Compañias Perdidas =
VAR FechaMaxima =
    CALCULATE(
        MAX('transaction'[date_from_timestamp]),
        REMOVEFILTERS('calendar')
    )
VAR Umbral = EDATE(FechaMaxima, -12)
VAR Perdidas =
    CALCULATE(
        COUNTROWS(
            FILTER(
                VALUES('transaction'[company_id]),
                CALCULATE(
                    MAX('transaction'[date_from_timestamp]),
                    REMOVEFILTERS('calendar')
                ) < Umbral
            )
        ),
        REMOVEFILTERS('calendar')
    )
RETURN
IF(ISBLANK(Perdidas), 0, Perdidas)
```

## Producto

### %_Crecimiento_productos_YoY

```dax
%_Crecimiento_productos_YoY =
DIVIDE([productos_anual] - [Producto Año Anterior], [Producto Año Anterior])
```

### Producto Año Anterior

```dax
Producto Año Anterior =
CALCULATE([productos_anual], SAMEPERIODLASTYEAR(calendar[Date]))
```

### productos_anual

```dax
productos_anual =
DISTINCTCOUNT(transaction_product[product_id])
```

## Color de producto

### % Diferencia color top

```dax
% Diferencia color top =
VAR ColorMax =
    MAXX(
        VALUES(product[colour]),
        CALCULATE(SUM('transaction'[amount(USD)]))
    )
VAR ColorMin =
    MINX(
        VALUES(product[colour]),
        CALCULATE(SUM('transaction'[amount(USD)]))
    )
RETURN DIVIDE(ColorMax - ColorMin, ColorMax)
```

### Color producto

```dax
Color producto =
SELECTEDVALUE(product[colour], "#cccccc")
```

### Diferencia color top

```dax
Diferencia color top =
VAR ColorMax =
    MAXX(
        VALUES(product[colour]),
        CALCULATE(SUM('transaction'[amount(USD)]))
    )
VAR ColorMin =
    MINX(
        VALUES(product[colour]),
        CALCULATE(SUM('transaction'[amount(USD)]))
    )
RETURN ColorMax - ColorMin
```

### Max colour product

```dax
Max colour product =
VAR ColorMax =
    MAXX(
        VALUES(product[colour]),
        CALCULATE(SUM('transaction'[amount(USD)]))
    )
RETURN
	CALCULATE(
	MAXX(VALUES('product'[colour]),
	SUM('transaction'[amount(USD)])) = ColorMax
	)
```

## Foco Alemania

Ticket medio de las compañías alemanas en 2018 y 2019 (Sprint 7).

### 2018_media_Alemania_efectivas

```dax
2018_media_Alemania_efectivas =
CALCULATE(
AVERAGE('transaction'[amount(USD)]),
'transaction'[declined] = 0,
'calendar'[year] = 2018, 
'company'[country] = "Germany"
)
```

### 2019_media_Alemania_efectivas

```dax
2019_media_Alemania_efectivas =
CALCULATE(
AVERAGE('transaction'[amount(USD)]),
'transaction'[declined] = 0,
'calendar'[year] = 2019, 
'company'[country] = "Germany"
)
```

### media_Alemania_efectivas

```dax
media_Alemania_efectivas =
CALCULATE(
    AVERAGE('transaction'[amount(USD)]),
    'transaction'[declined] = 0,
    company[country] = "Germany"
)
```

## Otras medidas

### % Transacciones Declinadas

```dax
% Transacciones Declinadas =
DIVIDE(
    CALCULATE(
        COUNTROWS('transaction'),
        'transaction'[declined] = 1
    ),
    CALCULATE(
        COUNTROWS('transaction'),
        ALL(user[country]),
        ALL(company[country])
    )
)
```

### %sum_transactions

```dax
%sum_transactions =
DIVIDE(
    [suma_transactions],                          -- el valor que quieres en %
    CALCULATE([suma_transactions], ALL('transaction'))    -- el total ignorando filtros
)
```

### %total

```dax
%total =
DIVIDE([suma_transactions],
    CALCULATE([suma_transactions], ALL('transaction'))
)
```

### Num_total_transactions

```dax
Num_total_transactions =
COUNTROWS('transaction')
```

### Ultima Fecha Transaccion

```dax
Ultima Fecha Transaccion =
CALCULATE(
    MAX('transaction'[date_from_timestamp]),
    REMOVEFILTERS('calendar')
)
```

### Ventas Declinadas %

```dax
Ventas Declinadas % =
DIVIDE(
    CALCULATE(
        [suma_transactions],
        'transaction'[declined] = 1,
        ALL(user[country]),
        ALL(company[country]),
        ALL('calendar')
    ),
    CALCULATE(
        [suma_transactions],
        'transaction'[declined] = 1,
        ALL('transaction')
    )
)
```

### auxiliar%sum_transactions

```dax
auxiliar%sum_transactions =
DIVIDE(
    [suma_transactions],
    CALCULATE(
        [suma_transactions],
        ALL(user[country]),
        ALL(company[country])
    )
)
```

### media_amount_dolars

```dax
media_amount_dolars =
CALCULATE(AVERAGE('transaction'[amount(USD)]))
```

### media_amount_euros

```dax
media_amount_euros =
DIVIDE([media_amount_dolars], 1.08)
```

### media_transaction_2021

```dax
media_transaction_2021 =
CALCULATE(
AVERAGE('transaction'[amount(USD)]),'calendar'[year] = 2021
)
```

### media_transaction_2022

```dax
media_transaction_2022 =
CALCULATE(
AVERAGE('transaction'[amount(USD)]),'calendar'[year] = 2022
)
```

### suma_transactions

```dax
suma_transactions =
SUM('transaction'[amount(USD)])
```
