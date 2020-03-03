# ML with Bigquery


I already discarded the columns through Python, but we could do something like the following

## Creating the model

        CREATE OR REPLACE MODEL
        `projectid.dataset.tabletoplaywith_vargas_regression` OPTIONS (model_type="linear_reg",
            labels=["ConversionRate"]) AS
        WITH
        params AS (
        SELECT
            1 AS train,
            2 AS eval)
        SELECT
        EXTRACT(DAY
        FROM
            Date) AS Day,
        EXTRACT(dayofweek
        FROM
            Date) AS DayOfWeek,
        EXTRACT(month
        FROM
            Date) AS Month,
        ConversionTrackerId,
        AdNetworkType1,
        AdNetworkType2,
        ConversionAttributionEventType,
        ConversionCategoryName,
        ConversionRate,
        ConversionTypeName,
        ConversionValue,
        CostPerConversion,
        HourOfDay,
        ValuePerConversion
        FROM
        `projectid.dataset.tabletoplaywith`

- In theory, we could use dnn_regressor and boosted_tree_regressor, but Google changed things and they're not available anymore. We'll need to use tensorflow statements

## Evaluate the model

With this we have all metrics. For some reason what google people wants is to minimize the mean_absolute_error. they don't give shit abour r2_score


        SELECT
        *
        FROM
        ML.EVALUATE(MODEL `projectid.dataset.tabletoplaywith_vargas_regression`)

## Prediction

        SELECT
        *
        FROM
        ML.PREDICT(MODEL `projectid.dataset.tabletoplaywith_vargas_regression`,
        (SELECT
        ConversionRate,
        EXTRACT(DAY
                FROM
                    Date) AS Day,
                EXTRACT(dayofweek
                FROM
                    Date) AS DayOfWeek,
                EXTRACT(month
                FROM
                    Date) AS Month,
                ConversionTrackerId,
                AdNetworkType1,
                AdNetworkType2,
                ConversionAttributionEventType,
                ConversionCategoryName,
                ConversionTypeName,
                ConversionValue,
                CostPerConversion,
                HourOfDay,
                ValuePerConversion
        from
        `projectid.dataset.tabletoplaywith`
        ))


## Results

![alt](./pics/bq-predictions-regression.png "")

- Hemos recogido datos de 2 meses, enero y febrero

- 1. Agrupando por día del mes:
    A partir de 0.25 de conversion rate se empiezan a diferenciar los resultados, Realmente no tiene mucho sentido agrupar por día del mes
- 2. Agrupando por horas del día

- 3. Agrupando por día de la semana se ve que el miércoles la predicción difiere más que el resto de días, miércoles-jueves

- 4. Agrupando por mes obtenemos una predicción muy muy buena


https://datastudio.google.com/u/0/reporting/1awkU_GPtZBy13oRX21c2fIo1NOigrJ9j/page/pgZGB/edit


