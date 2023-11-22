
# EDA-CustomerPurchaseBehaviour-Tableau




## Overview

Project Goals for TATA's Retail Store Sales:

* Analyze revenue contributors for strategic planning.
* Assess operational and marketing metrics.
* Offer expansion guidance using demographic-based insights.

## Table of Contents

1. Defined the Questions

1. Collecting and Cleaning Data 	

3. Analysing the Data 

1. Interpreting the Results	

1. Requirements

## Defined the Questions

| As a(role) |I want(request/demand) | So that I (user value)| Acceptance criteria |
| :---:        |     :---:      |         :---: | :---:|
| CEO | To get a line chart to show the revenue with time     |Can forecast for the next year based on seasonal trends and dig deeper into this trend |A Tableau dashboard that shows monthly revenue |
| CMO    | A detailed overview of revenue and product quantity sold per country      | Can follow up with the country that contributes the most revenue    |A Tableau dashboard that allows me to filter data for each country not including the UK|
| CMO   | A detailed overview of the information on the top 10 customers by revenue      | Can follow up the highest revenue-generating customer at the start and gradually decline to the lower revenue-generating customers  |A Tableau dashboard that allows me to filter data for each customer|
| CEO   | A dashboard overview of all countries and which regions have the highest demand     | Can target these areas and generate more business from these regions   |A Tableau dashboard with a map comparing each country|




## Collecting and Cleaning Data 

* Collecting Dataset: Importing online retail datasets to Tableau.

* Size of Dataset: The dataset contains 541,909 rows (transactions) and 8 columns (attributes). *InvoiceNo,	StockCode,	Description,	Quantity,	InvoiceDate,	UnitPrice,	CustomerID,	Country*.

* Wise Deletion Missing Values: There are missing values in the 'CustomerID' field. Every transaction should ideally have a customer ID associated with it. Wisely delete 135050 rows with null values.  

* Removing Duplicate Rows: Delete duplicate and blank rows to get 488,624 transactions. 

* Create a check that the Unit price should not be below $0
```
IF [Unit Price] < 0 then 'less than 0' ELSE 'more than 0' END
``` 
* Create a check that the quantity should not be below 1 unit
```
IF [Quantity] < 1 then 'less than 0' ELSE 'more than 0' END
```

## Analysing the Data 

**RFM Customer Segments**

1. Calculating RFM Measures
```
<!-- RFM-Recency -->

DATEDIFF('day',{ FIXED [Customer ID]:MAX([Invoice Date])}, {MAX([Invoice Date])})

<!-- RFM-Frequency -->

{ FIXED [Customer ID] : COUNTD([Invoice No])}

<!-- RFM-Monetary -->

{ FIXED [Customer ID]:SUM([Revenue ])}
```
2. Calculating RFM Scores
   
After getting the RFM measures, we will create three new fields, i.e., “Score_R”, “Score_F” and “Score_M”. Here we are trying to assign a score to each customer ID, from 1 to 4 (where 1 is the best and 4 is the worst) based on the percentile in which their RFM measures fall. For example, a smaller value of RFM-Recency, i.e., recency, shows that the customer has made the purchase more recently, so all RFM-R values falling in the first 25 percentile are assigned an R-Score of 1, and so on. But, since good customers are expected to make high-value and frequent purchases, for RFM-Frequency and RFM-Monetary, all values falling in the top 75 percentile are assigned a score of 1, and so on. Finally, we will calculate a combined RFM-Score after calculating all three individual scores. These scores will be our basis for segmenting and defining the type of customers.

```
<!-- Score_R -->

IF [RFM_Recency] <= { FIXED :PERCENTILE([RFM_Recency],0.25)} THEN 1
ELSEIF [RFM_Recency]> { FIXED :PERCENTILE([RFM_Recency],0.25)} AND [RFM_Recency]<= { FIXED :PERCENTILE([RFM_Recency],0.5)} THEN 4
ELSEIF [RFM_Recency]>{ FIXED :PERCENTILE([RFM_Recency],0.5)} AND [RFM_Recency]<= { FIXED :PERCENTILE([RFM_Recency],0.75)} THEN  3
ELSE 4
END
```
```
<!-- Score_F -->

IF [RFM_Frequency] <= {FIXED:PERCENTILE([RFM_Frequency],0.25)} 
THEN 4
ELSEIF [RFM_Frequency] > {FIXED:PERCENTILE([RFM_Frequency],0.25)} AND [RFM_Frequency] <= {FIXED:PERCENTILE([RFM_Frequency],0.5)}
THEN 3
ELSEIF [RFM_Frequency] > {FIXED:PERCENTILE([RFM_Frequency],0.5)} AND [RFM_Frequency] <= {FIXED:PERCENTILE([RFM_Frequency],0.75)}
THEN 2
ELSE 1
END
```
```
<!-- Score_M -->

IF [RFM_Monetary] <= {FIXED:PERCENTILE([RFM_Monetary],0.25)} 
THEN 4
ELSEIF [RFM_Monetary] > {FIXED:PERCENTILE([RFM_Monetary],0.25)} AND [RFM_Monetary] <= {FIXED:PERCENTILE([RFM_Monetary],0.5)}
THEN 3
ELSEIF [RFM_Monetary] > {FIXED:PERCENTILE([RFM_Monetary],0.5)} AND [RFM_Monetary] <= {FIXED:PERCENTILE([RFM_Monetary],0.75)}
THEN 2
ELSE 1
END
```
```
<!-- Score combine -->

INT(STR([Score_R])+STR([Score_F])+STR([Score_M]))
```

3. Creating RFM Segments

With a series of if-else statements in the calculation field, we will categorize the customers based on the combination of different RFM scores we calculated in the last step. For example, if all three R, F, and M-Scores are equal to 1 for a customer, then that customer is considered the “Best Customer,” and if all three R, F, and M-Scores are equal to 4, then that customer is regarded as the “Lost Cheap Customer.” It is easy to understand the logic behind the terminology of customer groups. Here, we have defined nine customer groups, i.e., Best, Potential to become Best, Loyal, Big Spenders, Almost Lost, Lost, Lost Cheap, Look Out Buyers, and Occasional Buyers.
```
<!-- RFM_customer segments -->

IF [Score combine] == 111 

THEN 'Best Customers'

ELSEIF [Score_R] == 1 AND [Score_F] == 2 AND [Score_M]<=2

THEN 'Potential To Become Best Customer'

ELSEIF [Score_F] == 1

THEN 'Loyal Customer'

ELSEIF [Score_M] == 1

THEN 'Big Spenders'

ELSEIF [Score combine] == 311

THEN 'Almost Lost'

ELSEIF [Score combine] == 411

THEN 'Lost Customers'

ELSEIF [Score combine] == 444

THEN 'Lost Cheap Customers'

ELSEIF [Score_R] <= 2 AND [Score_F]>= 2

THEN 'Look Out Buyers'

ELSEIF [Score_R] >= 2 AND [Score_F] >= 2

THEN 'Occasional Buyers'

END
```

## Interpreting the Results	click [Portfolio](https://public.tableau.com/app/profile/sherry.wang6643/viz/OnlineSalesGrowthAnalysis/Dashboard1)

<img src= "https://github.com/SheriWon/Tableau-CustomerPurchaseBehaviour/blob/main/image/Country%20Sales%20Pareto.png "  width="400" height="250">

<img src= "https://github.com/SheriWon/Tableau-CustomerPurchaseBehaviour/blob/main/image/customer%20purchase%20latency.png"  width="400" height="250">

<img src= "https://github.com/SheriWon/Tableau-CustomerPurchaseBehaviour/blob/main/image/RFM%20details.png"  width="400" height="250">


## Requirements

* Visualisation tool

    - Tableau 
* Analysis concept 

    - EDA exploratory data analysis:
        - Variable Identification 
        - Univariate Analysis 
        - Missing values treatment 
        - Outlier treatment
        - Variable transformation
        - Variable creation
* [Data Source Files](https://www.theforage.com/virtual-experience/MyXvBcppsW2FkNYCX/tata/data-visualisation-empowering-business-with-effective-insights/creating-effective-visuals)



