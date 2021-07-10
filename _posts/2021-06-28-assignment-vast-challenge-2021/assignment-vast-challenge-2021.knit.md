---
title: "Assignment - VAST Challenge 2021"
description: |
  [VAST Challenge 2021](https://vast-challenge.github.io/2021/). Applying visual analytics to gain Insight.....  
author:
  - name: Choo T Tan
    url: {}
date: 06-28-2021
output:
  distill::distill_article:
    toc: true
    toc_depth: 2
    toc_float: true
---



<div class="layout-chunk" data-layout="l-body">


</div>

<div class="layout-chunk" data-layout="l-body">


</div>


# 1.0 Introduction 
The assignment is based on [VAST Challenge 2021](https://vast-challenge.github.io/2021/) where it attempted to solve the [Mini Challenges 2 (MC2)](https://vast-challenge.github.io/2021/MC2.html) via visual analytics out of the 3 mini challenges. The fictitious scenario was based on a Tethys-based natural gas production company GAStech in the country of Kronos where it has remarkable profits and strong relationship with the government. However, GAStech has not been as successful in demonstrating environmental stewardship. Several Employee went missing in January 2014 after a successful initial public offering. It was suspected that the disappearance might be associated with an organization known as the Protectors of Kronos (POK). However, things may not be so simplicity. Thus,one is tasked to assist the law enforcement from Kronos and Tethys in solving the assigned task via visual analytics.

# 2.0 Liteture Review

# 3.0 Approach and Overview
For MC2, the main task was to identify suspicious patterns of behavior from the transaction done by GASTech employees. You must cope with uncertainties that result from missing, conflicting, and imperfect data to make recommendations for further investigation.there are 4 main CSV files with Korons geospatial data and it based on last 14 days, 6 - 19 January 2014 prior to the disappearance. The CSV files are namely (1) credit card transaction, (2) loyalty card transaction, (3) geospatial tracking data for company cars which not know to employee, (4) car assignment. The investigation will the following 4 main steps:

1. **Preliminary Analysis**. The intent is to gain insights across the dataset distribution and disparity. A basic understand on the foundation layer.

2. **Data Wrangling**. To establish relationship between credit card, loyalty card and geospatial tracking data, To prepare the data for subsequent analysis.

3. **Exploratory Data Analysis (EDA)**.  To identify patterns and anomalies from the transactions and geospatial tracking. Subsequent, to correlate the discovered insights in suitable narrative for further investigation. 

# 4.0 Preliminary Analysis

Sample of credit and loyalty datasets were shown below. Preliminary, observed that there were more credit than loyalty card transaction. Whether it was a scenario on independent credit card transaction without loyalty card or vice verse was something to be discovered later.Moreover credit card has transaction data time information while loyalty card only has transaction date only. Reference to the respective column datatype, need to covert "timestamp" to datetime/date,"last4ccnum" and loyaltynum" to factor as technically, it was not a numeric data type. When one explored the "timestamp" data deeper, its observed that the variable has different date format after 13 Jan. i.e. 2014 %Y vs 14 %y. Would need to be clean. 

Glimpse of credit card dataset:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 1,490
Columns: 4
$ timestamp  <chr> "1/6/14 7:28", "1/6/14 7:34", "1/6/14 7:35", "1/6…
$ location   <chr> "Brew've Been Served", "Hallowed Grounds", "Brew'…
$ price      <dbl> 11.34, 52.22, 8.33, 16.72, 4.24, 4.17, 28.73, 9.6…
$ last4ccnum <dbl> 4795, 7108, 6816, 9617, 7384, 5368, 7253, 4948, 9…
```

</div>


Glimpse of loyalty card dataset:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 1,392
Columns: 4
$ timestamp  <chr> "1/6/14", "1/6/14", "1/6/14", "1/6/14", "1/6/14",…
$ location   <chr> "Brew've Been Served", "Brew've Been Served", "Ha…
$ price      <dbl> 4.17, 9.60, 16.53, 11.51, 12.93, 4.27, 11.20, 15.…
$ loyaltynum <chr> "L2247", "L9406", "L8328", "L6417", "L1107", "L40…
```

</div>


<div class="layout-chunk" data-layout="l-body">


</div>


Reference to the company employee records in challenge 1, GASTech has 54 staffs. However when do a unique count on the credit and loyalty card numbers, it was 55 and 54 count respectively. It alluded that one of the staff could have 2 credit cards. Moreover,a unique count on the number of transaction locations for each dataset has shown that there were 34 and 33 locations for credit and loyalty card respectively. The additional location in credit card dataset was known as "Daily Dealz" with a transaction amount "2.01".From the transaction price captured respectively in both dataset, interesting to note that the mean was relatively close but min-max and inter-quartile range were quite different. 

<div class="layout-chunk" data-layout="l-body">

```
[1] "Credit Card PriceTransaction Summary."
```

```
    Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    2.01    15.13    28.24   207.70    67.18 10000.00 
```

```
[1] "Loyalty Card Price Transaction Summary."
```

```
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   3.00   13.13   22.84  204.33   38.22 4983.52 
```

</div>


On the geospatial tracking dateset, similar observation on timestamp data type. Given there were 35 and 5 unique private company car and truck respectively, there would be some transaction credit/loyalty card that would have difficulty to associate it to an individual within the company since not everybody drives. Similar treatment on datatype would have to be done on "Timestamp" and "id".   

Glimpse of geospatial tracking dataset:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 685,169
Columns: 4
$ Timestamp <chr> "01/06/2014 06:28:01", "01/06/2014 06:28:01", "01/…
$ id        <dbl> 35, 35, 35, 35, 35, 35, 35, 35, 35, 35, 35, 35, 35…
$ lat       <dbl> 36.07623, 36.07622, 36.07621, 36.07622, 36.07621, …
$ long      <dbl> 24.87469, 24.87460, 24.87444, 24.87425, 24.87417, …
```

</div>


# 5.0 Data Wrangling
## 5.1 Preparation of Personnel Node
This will be built based on MC1 employee records. Only relevant information will be used. 

Glimpse of personnel information:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 54
Columns: 9
$ PersID                <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12,…
$ FirstName             <chr> "Nils", "Lars", "Felix", "Ingrid", "Is…
$ LastName              <chr> "Calixto", "Azada", "Balas", "Barranco…
$ `Veh ID`              <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12,…
$ Gender                <chr> "Male", "Male", "Male", "Female", "Mal…
$ Department            <chr> "Information Technology", "Engineering…
$ Appt                  <chr> "IT Technician", "Engineer", "Engineer…
$ MilitaryServiceBranch <chr> NA, "TethanDefenseForceAir", "TethanDe…
$ MilitaryDischargeType <chr> NA, "GeneralDischarge", "HonorableDisc…
```

</div>

## 5.1 Mapping Credit to Loyalty Card

Perform required data wrangling based on observation in Section 4. To resolve the datetime format for timestamp and set the credit card number as factor. In addition, create date, weekday and hour for the credit. For loyalty card, renamed timestamp as date to faciltate inner join later. Similarly change the datatype for "loyaltynum".

Glimpse of prepared credit card dataset:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 1,490
Columns: 8
$ last4ccnum <fct> 1286, 1286, 1286, 1286, 1286, 1286, 1286, 1286, 1…
$ timestamp  <dttm> 2014-01-06 08:16:00, 2014-01-06 13:27:00, 2014-0…
$ date       <chr> "1/6/14", "1/6/14", "1/7/14", "1/7/14", "1/8/14",…
$ day        <dbl> 6, 6, 7, 7, 8, 8, 8, 9, 9, 9, 10, 11, 12, 12, 12,…
$ hr         <dbl> 8, 13, 7, 13, 8, 13, 20, 8, 13, 20, 8, 20, 13, 15…
$ location   <chr> "Brew've Been Served", "Abila Zacharo", "Brew've …
$ price      <dbl> 14.97, 50.14, 11.92, 45.05, 7.26, 50.36, 62.20, 1…
$ wkday      <ord> Monday, Monday, Tuesday, Tuesday, Wednesday, Wedn…
```

</div>

<div class="layout-chunk" data-layout="l-body">


</div>


Glimpse of prepared loyalty card dataset:

<div class="layout-chunk" data-layout="l-body">

```
Rows: 1,392
Columns: 4
$ loyaltynum <fct> L2247, L2247, L2247, L2247, L2247, L2247, L2247, …
$ date       <chr> "1/6/14", "1/6/14", "1/6/14", "1/7/14", "1/7/14",…
$ location   <chr> "Brew've Been Served", "Ouzeri Elian", "Frydos Au…
$ price      <dbl> 4.17, 20.74, 175.87, 12.74, 17.71, 23.12, 14.88, …
```

</div>


__Inner join for both Datasets__. Based on date, location and price for inner join. Thereafter, compute the number of distinct count between a credit-loyalty pair. If respective credit and loyalty cards has more than one distinct pair, imply the credit card owner could be using numerous loyalty cards or vice verse. It note that inner join has 1087 observations, 403 and 305 less than credit and loyalty card transaction respectively. This could imply that individual could only transact with one card due to some reasons. When determine the unique count on both cards and location, it was determined that the unique count for credit and loyalty card remained no change. This imply all possible relationship between both card should be captured within the inner join. On location, the inner join has 2 location less than the credit card transaction, namely, Korons Mart and Daily Dealz. This implied that at any onetime, only one card has been used in those places.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>ijoin</span> <span class='op'>&lt;-</span> <span class='va'>credit</span> <span class='op'>%&gt;%</span>
  <span class='fu'>inner_join</span><span class='op'>(</span><span class='va'>loyalty</span>,by<span class='op'>=</span><span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'date'</span>, <span class='st'>'location'</span>,<span class='st'>'price'</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>last4ccnum</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>ndist <span class='op'>=</span> <span class='fu'>n_distinct</span><span class='op'>(</span><span class='va'>loyaltynum</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>loyaltynum</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>ndist <span class='op'>=</span> <span class='fu'>if_else</span><span class='op'>(</span><span class='va'>ndist</span><span class='op'>==</span><span class='fl'>1</span>,<span class='fu'>n_distinct</span><span class='op'>(</span><span class='va'>last4ccnum</span><span class='op'>)</span>, <span class='va'>ndist</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>

<span class='va'>ijoin</span>
</code></pre></div>

```
# A tibble: 1,087 x 10
   last4ccnum timestamp           date     day    hr location    price
   <fct>      <dttm>              <chr>  <dbl> <dbl> <chr>       <dbl>
 1 1286       2014-01-06 08:16:00 1/6/14     6     8 Brew've Be… 15.0 
 2 1286       2014-01-06 13:27:00 1/6/14     6    13 Abila Zach… 50.1 
 3 1286       2014-01-07 07:54:00 1/7/14     7     7 Brew've Be… 11.9 
 4 1286       2014-01-07 13:24:00 1/7/14     7    13 Kalami Kaf… 45.0 
 5 1286       2014-01-08 08:16:00 1/8/14     8     8 Brew've Be…  7.26
 6 1286       2014-01-08 13:38:00 1/8/14     8    13 Gelatogalo… 50.4 
 7 1286       2014-01-08 20:12:00 1/8/14     8    20 Ouzeri Eli… 62.2 
 8 1286       2014-01-09 08:14:00 1/9/14     9     8 Brew've Be… 17.6 
 9 1286       2014-01-09 13:47:00 1/9/14     9    13 Abila Zach… 48.4 
10 1286       2014-01-09 20:24:00 1/9/14     9    20 Guy's Gyros 18.3 
# … with 1,077 more rows, and 3 more variables: wkday <ord>,
#   loyaltynum <fct>, ndist <int>
```

</div>

<div class="layout-chunk" data-layout="l-body">


</div>

<div class="layout-chunk" data-layout="l-body">


</div>


__Bipartite Graph for Credit-Loyalty Pair__. group_by was done on the inner join dataset to determine the number of transactions for each unique pairing of credit and loyalty card. To minimise cluttering in discovering the unique pairing respective credit to loyalty card, we have filtered the dataset for distinct pairing count >=2. Will use igraph to built a bipartite graph with nodes as credit and loyalty cards, where the edges connote transaction where both cards were used and the edge weight equal to the number of transactions. The bipartite graph was shown below.

<div class="layout-chunk" data-layout="l-body">
<div class="figure" style="text-align: centre">
<img src="assignment-vast-challenge-2021_files/figure-html5/unnamed-chunk-12-1.png" alt="Bipartite Graph of Credit(Orange) and Loyalty(Pink) Cards." width="624" />
<p class="caption">(\#fig:unnamed-chunk-12)Bipartite Graph of Credit(Orange) and Loyalty(Pink) Cards.</p>
</div>

</div>



```{.r .distill-force-highlighting-css}
```
