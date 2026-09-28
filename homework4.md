Homework 4
================

- [Q2](#q2)
- [Q3](#q3)
- [Q4.1](#q41)
- [Q4.2](#q42)
- [Q4.3](#q43)
- [Q4.4](#q44)
- [Q4.5](#q45)

``` r
install.packages(c("DBI", "RMariaDB", "mdsr", "knitr", "rmarkdown"))
```

``` r
library(DBI)
library(RMariaDB)
library(mdsr)
```

## Q2

``` r
con_wai <- dbConnect(
  MariaDB(),
  host = "scidb.smith.edu",
  user = "waiuser",
  password = "smith_waiDB",
  dbname = "wai"
)

q2 <- dbGetQuery(con_wai, "
  SELECT COUNT(*) AS n_rows
  FROM Measurements;
")
knitr::kable(q2)
```

|  n_rows |
|--------:|
| 5052304 |

``` r
dbDisconnect(con_wai)
```

The `Measurements` table contains **5,052,304 rows**.

## Q3

``` r
con_air <- dbConnect_scidb("airlines")

q3 <- dbGetQuery(con_air, "
  SELECT DISTINCT year
  FROM flights
  ORDER BY year;
")
knitr::kable(q3)
```

| year |
|-----:|
| 2013 |
| 2014 |
| 2015 |

The available years are **2013, 2014, 2015**. `DISTINCT` returns each
year only once.

## Q4.1

``` r
q4_1 <- dbGetQuery(con_air, "
  SELECT COUNT(*) AS n_flights,
         SUM(cancelled = 1) AS n_cancelled,
         SUM(cancelled = 0 AND diverted = 0) AS n_completed
  FROM flights
  WHERE year = 2015 AND month = 5 AND day = 14
    AND dest = 'DFW';
")
knitr::kable(q4_1)
```

| n_flights | n_cancelled | n_completed |
|----------:|------------:|------------:|
|       737 |           2 |         735 |

There are **737 flight records** with DFW as the destination on May 14,
2015. This total includes 2 cancelled flights. If “flew into” means
non-cancelled, non-diverted arrivals, the answer is **735 flights**.

## Q4.2

``` r
q4_2 <- dbGetQuery(con_air, "
  SELECT dest, COUNT(*) AS n_flights
  FROM flights
  WHERE year = 2015 AND origin = 'ORD'
  GROUP BY dest
  ORDER BY n_flights DESC, dest
  LIMIT 6;
")
knitr::kable(q4_2, format.args = list(big.mark = ","))
```

| dest | n_flights |
|:-----|----------:|
| LGA  |     10492 |
| LAX  |      8720 |
| DFW  |      8384 |
| SFO  |      8156 |
| BOS  |      7240 |
| ATL  |      7104 |

The six most common destinations were **LGA, LAX, DFW, SFO, BOS, ATL**,
in descending order. **LGA** was the most common, with **10,492
flights**.

## Q4.3

``` r
q4_3 <- dbGetQuery(con_air, "
  SELECT dest,
         AVG(arr_delay) AS avg_arr_delay_minutes,
         COUNT(arr_delay) AS n_observed_delays
  FROM flights
  WHERE year = 2015
  GROUP BY dest
  ORDER BY avg_arr_delay_minutes DESC, dest
  LIMIT 1;
")
knitr::kable(q4_3, digits = 2)
```

| dest | avg_arr_delay_minutes | n_observed_delays |
|:-----|----------------------:|------------------:|
| STC  |                 21.62 |                82 |

**STC** had the highest average arrival delay, approximately **21.62
minutes**, based on 82 non-missing delay values. `AVG()` excludes SQL
`NULL` values. Negative delays, representing early arrivals, are
retained, and no minimum flight-count threshold is imposed.

## Q4.4

``` r
q4_4 <- dbGetQuery(con_air, "
  SELECT COUNT(*) AS n_flights,
         SUM(dest = 'BDL') AS n_inbound,
         SUM(origin = 'BDL') AS n_outbound,
         SUM(cancelled = 1) AS n_cancelled,
         SUM(
           cancelled = 0 AND
           (origin = 'BDL' OR (dest = 'BDL' AND diverted = 0))
         ) AS n_operated_to_or_from_bdl
  FROM flights
  WHERE year = 2015
    AND (origin = 'BDL' OR dest = 'BDL');
")
knitr::kable(q4_4, format.args = list(big.mark = ","))
```

| n_flights | n_inbound | n_outbound | n_cancelled | n_operated_to_or_from_bdl |
|----------:|----------:|-----------:|------------:|--------------------------:|
|     41025 |    20,512 |     20,513 |         729 |                    40,258 |

There are **41,025 flight records** to or from BDL in 2015: 20,512
inbound and 20,513 outbound. The parentheses ensure that the year
condition applies to both directions.

The total includes 729 cancelled flights. Excluding cancellations and
inbound flights diverted away from BDL gives **40,258 flights** that
operated to or from BDL according to these fields. A non-cancelled
flight departing BDL is included even if it was later diverted.

## Q4.5

``` r
q4_5 <- dbGetQuery(con_air, "
  SELECT c.name AS airline, f.carrier, f.flight,
         f.origin, f.dest
  FROM flights AS f
  LEFT JOIN carriers AS c ON f.carrier = c.carrier
  WHERE f.year = 2015 AND f.month = 9 AND f.day = 26
    AND (
      (f.origin = 'LAX' AND f.dest = 'JFK')
      OR (f.origin = 'JFK' AND f.dest = 'LAX')
    )
  ORDER BY f.origin, c.name, f.flight;
")
knitr::kable(q4_5)
```

| airline                | carrier | flight | origin | dest |
|:-----------------------|:--------|-------:|:-------|:-----|
| American Airlines Inc. | AA      |      3 | JFK    | LAX  |
| American Airlines Inc. | AA      |     19 | JFK    | LAX  |
| American Airlines Inc. | AA      |     21 | JFK    | LAX  |
| American Airlines Inc. | AA      |     33 | JFK    | LAX  |
| American Airlines Inc. | AA      |    117 | JFK    | LAX  |
| American Airlines Inc. | AA      |    255 | JFK    | LAX  |
| American Airlines Inc. | AA      |    293 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    420 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    423 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    447 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    464 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    472 | JFK    | LAX  |
| Delta Air Lines Inc.   | DL      |    477 | JFK    | LAX  |
| JetBlue Airways        | B6      |     23 | JFK    | LAX  |
| JetBlue Airways        | B6      |    123 | JFK    | LAX  |
| JetBlue Airways        | B6      |    223 | JFK    | LAX  |
| JetBlue Airways        | B6      |    323 | JFK    | LAX  |
| JetBlue Airways        | B6      |    423 | JFK    | LAX  |
| JetBlue Airways        | B6      |    523 | JFK    | LAX  |
| JetBlue Airways        | B6      |    623 | JFK    | LAX  |
| United Air Lines Inc.  | UA      |    441 | JFK    | LAX  |
| United Air Lines Inc.  | UA      |    535 | JFK    | LAX  |
| United Air Lines Inc.  | UA      |    703 | JFK    | LAX  |
| United Air Lines Inc.  | UA      |    841 | JFK    | LAX  |
| United Air Lines Inc.  | UA      |   1985 | JFK    | LAX  |
| Virgin America         | VX      |    399 | JFK    | LAX  |
| Virgin America         | VX      |    407 | JFK    | LAX  |
| Virgin America         | VX      |    411 | JFK    | LAX  |
| Virgin America         | VX      |    413 | JFK    | LAX  |
| Virgin America         | VX      |    415 | JFK    | LAX  |
| American Airlines Inc. | AA      |      2 | LAX    | JFK  |
| American Airlines Inc. | AA      |      4 | LAX    | JFK  |
| American Airlines Inc. | AA      |     22 | LAX    | JFK  |
| American Airlines Inc. | AA      |     30 | LAX    | JFK  |
| American Airlines Inc. | AA      |     32 | LAX    | JFK  |
| American Airlines Inc. | AA      |     34 | LAX    | JFK  |
| American Airlines Inc. | AA      |    118 | LAX    | JFK  |
| American Airlines Inc. | AA      |    180 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |    412 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |    476 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |    920 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |   1162 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |   1262 | LAX    | JFK  |
| Delta Air Lines Inc.   | DL      |   1908 | LAX    | JFK  |
| JetBlue Airways        | B6      |     24 | LAX    | JFK  |
| JetBlue Airways        | B6      |    124 | LAX    | JFK  |
| JetBlue Airways        | B6      |    224 | LAX    | JFK  |
| JetBlue Airways        | B6      |    324 | LAX    | JFK  |
| JetBlue Airways        | B6      |    424 | LAX    | JFK  |
| JetBlue Airways        | B6      |    524 | LAX    | JFK  |
| JetBlue Airways        | B6      |    624 | LAX    | JFK  |
| United Air Lines Inc.  | UA      |    275 | LAX    | JFK  |
| United Air Lines Inc.  | UA      |    779 | LAX    | JFK  |
| United Air Lines Inc.  | UA      |    912 | LAX    | JFK  |
| United Air Lines Inc.  | UA      |   1752 | LAX    | JFK  |
| Virgin America         | VX      |    404 | LAX    | JFK  |
| Virgin America         | VX      |    406 | LAX    | JFK  |
| Virgin America         | VX      |    412 | LAX    | JFK  |
| Virgin America         | VX      |    416 | LAX    | JFK  |
| Virgin America         | VX      |    420 | LAX    | JFK  |

The table lists all flights in **both directions**.
