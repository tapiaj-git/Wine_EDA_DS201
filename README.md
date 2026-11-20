Wine Quality Analysis
================

GROUP: John Tapia James Pfaff Jacob Dudas

Imported libraries

``` r
library(tidyverse)
library(ggplot2)
library(dplyr)
```

# **1) Business Understanding**

Instead of interpreting the dataset alone, it is important to research
the variables in our dataset that represent good-quality wine and which
variables we should consider most significant. The quality of wine,
according to the Wine and Spirit Education Trust (WSET), depends on
several factors, evaluated based on their balance, flavor intensity,
complexity, and how long their flavors last after tasting. From a data
standpoint, it is important to consider red and white wines’ acidity,
sweetness, alcohol content, aroma, and balance. The highest-rated wines
do not necessarily have the highest or lowest levels of any variable,
rather, they have different characteristics that work well together.

Now, let’s discuss the significant variables in the dataset. To begin,
one important factor when considering the best wines is acidity, which
contributes to the wine’s overall taste and “freshness”; too much can
make wine sour, while too little can make it taste flat. Another factor
is volatile acidity, which measures certain acids that can create an
unpleasant vinegar smell at high levels. Residual sugar and alcohol
content also affect wine taste substantially, as sweetness can balance
acidity while alcohol content can overpower other flavors. The
characteristics listed in the dataset suggest that the relationship
between a wine’s chemical makeup and its quality is more complicated
than having higher or lower levels of certain ingredients.

Sources: Green, M. (n.d.). Understanding acidity in wine. Wine & Spirit
Education Trust.
<https://www.wsetglobal.com/news-insights/insight-opinion/understanding-acidity-in-wine/>
Australian Wine Research Institute. (n.d.). Wine flavours, faults and
taints.
<https://www.awri.com.au/industry_support/winemaking_resources/sensory_assessment/recognition-of-wine-faults-and-taints/wine_faults/>

# **2) Data Understanding**

This project analyzes the UCI Wine Quality Dataset, which contains 1,599
red wines and 4,898 white wines from Portugal, for a total of 6,497
observations. Each individual wine observation includes 11 chemical
measurements and a quality score determined by the measurements. All 11
variables are quantitative (ratio and interval), while the quality score
is ordinal. The quality score is on a scale of 0 to 10; in the dataset,
the wines range from 3 to 9, with red wines ranging from 3 to 8 and
white wines from 3 to 9.

``` r
wine_features <- data.frame(
  Feature = c(
    "Fixed Acidity",
    "Volatile Acidity",
    "Citric Acid",
    "Residual Sugar",
    "Chlorides",
    "Free Sulfur Dioxide",
    "Total Sulfur Dioxide",
    "Density",
    "pH",
    "Sulphates",
    "Alcohol",
    "Quality",
    "Color"
  ),
  Description = c(
    "Amount of acids naturally present in wine that do not easily evaporate",
    "Amount of acids that can produce a vinegar-like taste or smell",
    "Amount of citric acid present, which contributes to acidity and freshness",
    "Amount of sugar remaining after fermentation",
    "Amount of salt present in the wine",
    "Amount of sulfur dioxide available to help preserve wine",
    "Total amount of sulfur dioxide, including free and bound forms",
    "Mass per unit volume of wine",
    "Measurement of how acidic or basic the wine is",
    "Amount of sulfates present, which can contribute to wine preservation",
    "Percentage of alcohol in the wine",
    "Wine quality rating based on sensory evaluations",
    "Wine color (red or white)"
  ),
  `Data Type` = c(
    rep("Ratio", 8),
    "Interval",
    "Ratio",
    "Ratio",
    "Ordinal",
    "Nominal"
  ),
  check.names = FALSE
)

knitr::kable(
  wine_features,
  caption = "Wine Quality Dataset: Feature Descriptions")
```

| Feature | Description | Data Type |
|:---|:---|:---|
| Fixed Acidity | Amount of acids naturally present in wine that do not easily evaporate | Ratio |
| Volatile Acidity | Amount of acids that can produce a vinegar-like taste or smell | Ratio |
| Citric Acid | Amount of citric acid present, which contributes to acidity and freshness | Ratio |
| Residual Sugar | Amount of sugar remaining after fermentation | Ratio |
| Chlorides | Amount of salt present in the wine | Ratio |
| Free Sulfur Dioxide | Amount of sulfur dioxide available to help preserve wine | Ratio |
| Total Sulfur Dioxide | Total amount of sulfur dioxide, including free and bound forms | Ratio |
| Density | Mass per unit volume of wine | Ratio |
| pH | Measurement of how acidic or basic the wine is | Interval |
| Sulphates | Amount of sulfates present, which can contribute to wine preservation | Ratio |
| Alcohol | Percentage of alcohol in the wine | Ratio |
| Quality | Wine quality rating based on sensory evaluations | Ordinal |
| Color | Wine color (red or white) | Nominal |

Wine Quality Dataset: Feature Descriptions

Based on additional research and discussions with generative AI, pH is
logarithmic rather than a conventional linear scale. It is classified
here as interval because differences in pH are meaningful, but ratios of
pH values are not.

# **3) Data Preparation**

We needed to change the delimiter to “;” when importing data and we
removed extra spaces and tabs around unquoted values. We also hid the
import summary we see to take out unnecessary information.

We then added a new column to each dataset called ‘color’ where its
either red or white. Then we joined the dataset, one on top of the other
since they have the same columns.

``` r
library(readr)

data_white <- readr::read_delim(
  "winequality-white.csv",
  delim = ";",
  trim_ws = TRUE,
  show_col_types = FALSE
)

data_red <- readr::read_delim(
  "winequality-red.csv",
  delim = ";",
  trim_ws = TRUE,
  show_col_types = FALSE
)

data_white$color <- "white"
data_red$color <- "red"

data <- dplyr::bind_rows(data_red, data_white)

view(data)
```

The two datasets of red wine and white wine were combined into one
larger dataset, but the color variable was re-added to distinguish color
types. This variable is already established on the wine quality’s
website via a variable type table. Combining these allows us to see
whether our hypotheses hold true, regardless of wine color. Outliers
were not removed from the dataset. Outliers were considered before
combining the sets of red wine and white wine, because both colors may
have varying definitions of what an “outlier” means.

# **4) Modeling+Evaluating**

## **Basic EDA**

Summary statistics using red wine. Tells us about the ranges,
means/medians, and NAs of red wine. As the table below shows, there are
no NAs for red wine. Residual sugar, chlorides, free sulfur dioxide, and
total sulfur dioxide seem to have a more erratic jump to the max from
the 3rd quartile. It does not mean we need to get rid of such data, but
it could help us understand the boxplots or distribution of each
feature, respective of its color.

``` r
summary(data[data$color == "red", ])
```

    ##  fixed acidity   volatile acidity  citric acid    residual sugar     chlorides       free sulfur dioxide
    ##  Min.   : 4.60   Min.   :0.1200   Min.   :0.000   Min.   : 0.900   Min.   :0.01200   Min.   : 1.00      
    ##  1st Qu.: 7.10   1st Qu.:0.3900   1st Qu.:0.090   1st Qu.: 1.900   1st Qu.:0.07000   1st Qu.: 7.00      
    ##  Median : 7.90   Median :0.5200   Median :0.260   Median : 2.200   Median :0.07900   Median :14.00      
    ##  Mean   : 8.32   Mean   :0.5278   Mean   :0.271   Mean   : 2.539   Mean   :0.08747   Mean   :15.87      
    ##  3rd Qu.: 9.20   3rd Qu.:0.6400   3rd Qu.:0.420   3rd Qu.: 2.600   3rd Qu.:0.09000   3rd Qu.:21.00      
    ##  Max.   :15.90   Max.   :1.5800   Max.   :1.000   Max.   :15.500   Max.   :0.61100   Max.   :72.00      
    ##  total sulfur dioxide    density             pH          sulphates         alcohol         quality     
    ##  Min.   :  6.00       Min.   :0.9901   Min.   :2.740   Min.   :0.3300   Min.   : 8.40   Min.   :3.000  
    ##  1st Qu.: 22.00       1st Qu.:0.9956   1st Qu.:3.210   1st Qu.:0.5500   1st Qu.: 9.50   1st Qu.:5.000  
    ##  Median : 38.00       Median :0.9968   Median :3.310   Median :0.6200   Median :10.20   Median :6.000  
    ##  Mean   : 46.47       Mean   :0.9967   Mean   :3.311   Mean   :0.6581   Mean   :10.42   Mean   :5.636  
    ##  3rd Qu.: 62.00       3rd Qu.:0.9978   3rd Qu.:3.400   3rd Qu.:0.7300   3rd Qu.:11.10   3rd Qu.:6.000  
    ##  Max.   :289.00       Max.   :1.0037   Max.   :4.010   Max.   :2.0000   Max.   :14.90   Max.   :8.000  
    ##        color     
    ##  Length   :1599  
    ##  N.unique :   1  
    ##  N.blank  :   0  
    ##  Min.nchar:   3  
    ##  Max.nchar:   3  
    ## 

``` r
colSums(is.na(data[data$color == "red", ]))
```

    ##        fixed acidity     volatile acidity          citric acid       residual sugar            chlorides 
    ##                    0                    0                    0                    0                    0 
    ##  free sulfur dioxide total sulfur dioxide              density                   pH            sulphates 
    ##                    0                    0                    0                    0                    0 
    ##              alcohol              quality                color 
    ##                    0                    0                    0
