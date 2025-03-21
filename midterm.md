    knitr::opts_chunk$set(fig.path='Figs/')

### Introduction

Crime rates vary significantly across countries and it is important to
understand the factors that influence crime for economics and public
policy. One key variable that is often linked to crime is income level;
specifically, whether higher income levels reduce crime by increasing
economic opportunities or conversely, whether income inequality
increases criminal activity. Additionally, education levels have been
studied as another determinant of crime since higher education is
associated with better employment prospects and lower engagement in
criminal activity.

This explanatory data analysis investigates the relationship between
average income (GDP per capita), education levels (literacy rate), and
crime rates (intentional homicides per 100,000 people) across multiple
countries. I will be using data retrieved from the World Bank API and I
will analyze whether countries with higher income and education levels
tend to have lower crime rates.

I would like to contribute to the broader discussion on how economic and
educational policies can be leveraged to reduce crime by conducting this
analysis. My findings may provide insights for people seeking to
implement strategies that promote safer societies through economic
development and educational investments.

### Methods

    library(httr)
    library(jsonlite)
    library(dplyr)
    library(ggplot2)

The data was retrieved using API queries in R with the httr and jsonlite
packages. I found the data for this analysis from the World Bank API,
which provides publicly accessible economic and social indicators for
countries worldwide. I retrieved the following datasets:

Income in GDP per capita using NY.GDP.PCAP.CD measures the economic
output per person in each region/country.

Crime Rate in Intentional Homicides per 100,000 using VC.IHR.PSRC.P5
represents the number of homicides per 100,000 individuals.

Education Levels in Adult Literacy Rate using SE.ADT.LITR.ZS measures
the percentage of literate adults in each country.

    url_income <- "http://api.worldbank.org/v2/country/all/indicator/NY.GDP.PCAP.CD?format=json&date=2015"

    response_income <- GET(url_income)
    data_income <- content(response_income, as = "text")
    json_income <- fromJSON(data_income, flatten = TRUE)

    df_income <- json_income[[2]] %>%
      select(country_code = countryiso3code, year = date, income = value) %>%
      filter(!is.na(income)) %>%  
      filter(country_code != "")

    head(df_income)

Im using an API from worldbank.org. First, I got the API URL then
fetched a dataset which contains regional aggregates/country code, year
2015, and income (GDP per capita) for each. Then, after fetching the
data, I wanted to turn it into a dataframe, so I got the JSON format to
rename the variables so that we can understand the variables more
clearly. Then, I removed the N/A values for income and regional
aggregates/country code to clean out the data.

    url_crime <- "http://api.worldbank.org/v2/country/all/indicator/VC.IHR.PSRC.P5?format=json&date=2015"

    response_crime <- GET(url_crime)
    data_crime <- content(response_crime, as = "text")
    json_crime <- fromJSON(data_crime, flatten = TRUE)

    df_crime <- json_crime[[2]] %>%
      select(country_code = countryiso3code, year = date, crime_rate = value) %>%
      filter(!is.na(crime_rate)) %>%
      filter(country_code != "")

    head(df_crime)

Similary, I got the API URL then fetched a dataset which contains
regional aggregates/country code, year 2015, and crime rate (Intentional
Homicides per 100k people) for each. Then, after fetching the data, I
wanted to turn it into a dataframe, so I got the JSON format to rename
the variables so that we can understand the variables more clearly.
Then, I removed the N/A values for crime rate and regional
aggregates/country code to clean out the data.

    url_education <- "http://api.worldbank.org/v2/country/all/indicator/SE.ADT.LITR.ZS?format=json&date=2015"

    response_education <- GET(url_education)
    data_education <- content(response_education, as = "text")
    json_education <- fromJSON(data_education, flatten = TRUE)

    df_education <- json_education[[2]] %>%
      select(country_code = countryiso3code, year = date, literacy_rate = value) %>%
      filter(country_code != "")

    head(df_education)

Similary, I got the API URL then fetched a dataset which contains
regional aggregates/country code, year 2015, and literacy rate for each.
Then, after fetching the data, I wanted to turn it into a dataframe, so
I got the JSON format to rename the variables so that we can understand
the variables more clearly. Then, I removed the N/A values for the
regional aggregates/country code to clean out the data, but not the
literacy rate since I wanted a smooth merging process.

    df_merged <- merge(df_income, df_crime, by = c("country_code", "year"))
    df_merged <- merge(df_merged, df_education, by = c("country_code", "year"))
    df_merged

In here, I merged these three datasets with country\_code and year. This
would be equivalent to a left join.

### Method Part 2 and Preliminary Result

I used kable for the summary statistics table which includes mean,
median and sd of each variables.

    library(dplyr)
    library(kableExtra)

    df_merged %>%
      summarise(
        mean_income = mean(income, na.rm = TRUE),
        median_income = median(income, na.rm = TRUE),
        sd_income = sd(income, na.rm = TRUE),
        mean_crime = mean(crime_rate, na.rm = TRUE),
        median_crime = median(crime_rate, na.rm = TRUE),
        sd_crime = sd(crime_rate, na.rm = TRUE),
        mean_literacy = mean(literacy_rate, na.rm = TRUE),
        median_literacy = median(literacy_rate, na.rm = TRUE),
        sd_literacy = sd(literacy_rate, na.rm = TRUE)
      ) %>%
      kable(caption = "Summary Statistics of Key Variables")

<table>
<caption>Summary Statistics of Key Variables</caption>
<colgroup>
<col style="width: 10%" />
<col style="width: 12%" />
<col style="width: 9%" />
<col style="width: 9%" />
<col style="width: 11%" />
<col style="width: 8%" />
<col style="width: 12%" />
<col style="width: 14%" />
<col style="width: 10%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: right;">mean_income</th>
<th style="text-align: right;">median_income</th>
<th style="text-align: right;">sd_income</th>
<th style="text-align: right;">mean_crime</th>
<th style="text-align: right;">median_crime</th>
<th style="text-align: right;">sd_crime</th>
<th style="text-align: right;">mean_literacy</th>
<th style="text-align: right;">median_literacy</th>
<th style="text-align: right;">sd_literacy</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">9242.597</td>
<td style="text-align: right;">5525.005</td>
<td style="text-align: right;">11993.84</td>
<td style="text-align: right;">7.406383</td>
<td style="text-align: right;">5.9</td>
<td style="text-align: right;">6.314062</td>
<td style="text-align: right;">78.14366</td>
<td style="text-align: right;">78.82537</td>
<td style="text-align: right;">15.46087</td>
</tr>
</tbody>
</table>

Summary Statistics of Key Variables

    # Histogram for income
    ggplot(df_merged, aes(x = income)) +
      geom_histogram(bins = 30, fill = "blue", alpha = 0.5) +
      labs(title = "Income Distribution", x = "Income (GDP per capita)", y = "Count")

![](Figs/unnamed-chunk-7-1.png)

    # Histogram for crime rate
    ggplot(df_merged, aes(x = crime_rate)) +
      geom_histogram(bins = 30, fill = "red", alpha = 0.5) +
      labs(title = "Crime Rate Distribution", x = "Crime Rate", y = "Count")

![](Figs/unnamed-chunk-7-2.png)

    # Histogram for literacy rate
    ggplot(df_merged, aes(x = literacy_rate)) +
      geom_histogram(bins = 30, fill = "red", alpha = 0.5) +
      labs(title = "Literacy Rate Distribution", x = "Literacy Rate", y = "Count")

    ## Warning: Removed 7 rows containing non-finite outside the scale range
    ## (`stat_bin()`).

![](Figs/unnamed-chunk-7-3.png)

    # Density plot for income
    ggplot(df_merged, aes(x = income)) +
      geom_density(fill = "blue", alpha = 0.5) +
      labs(title = "Density Plot of Income")

![](Figs/unnamed-chunk-7-4.png)

    # Density plot for crime rate
    ggplot(df_merged, aes(x = crime_rate)) +
      geom_density(fill = "blue", alpha = 0.5) +
      labs(title = "Density Plot of Crime Rate")

![](Figs/unnamed-chunk-7-5.png) In here, I explored the histogram to
check the frequency distribution of data values. Looking at the income
histogram, we can see that the data is highly right-skewed/positively
skewed. A small number of countries have very high GDP per capita,
pulling the distribution to the right. The majority of countries have
low or moderate incomes which creates a long tail towards higher values.
This means that using raw income values in this analysis might not be
appropriate due to the outliers. The crime rate distribution is also
right-skewed. Most countries have low crime rates, but a few have
exceptionally high crime rates. This means that a small number of
countries with high crime rate is dominating the dataset which could
violate normality. As stated, transformations like log(crime\_rate) can
help normalize the distribution. However, the literacy rate appears less
skewed with values clustered towards the higher end above 70-80%. Unlike
income and crime rate, literacy rates seems to be normally or slightly
left-skewed. Most countries have high literacy rates, with fewer having
low literacy levels. Both the density plots for income and crime rate
confirmed that both datasets are right skewed, so we will continue this
analysis using log transformed income and crime rate.

    ggplot(df_merged, aes(x = log(income))) +
      geom_histogram(bins = 30, fill = "blue", alpha = 0.5) +
      labs(title = "Log-Transformed Income Distribution", x = "Log(Income)", y = "Count")

![](Figs/unnamed-chunk-8-1.png)

    ggplot(df_merged, aes(x = log(crime_rate))) +
      geom_histogram(bins = 30, fill = "red", alpha = 0.5) +
      labs(title = "Log-Transformed Crime Rate Distribution", x = "Log(Crime Rate)", y = "Count")

![](Figs/unnamed-chunk-8-2.png) As expected, after log transforming
income and crime rate, it seems more normally distributed.

    ggplot(df_merged, aes(x = log(income), y = log(crime_rate))) +
      geom_point(alpha = 0.6) +
      geom_smooth(method = "lm", se = FALSE, color = "blue") +
      labs(title = "Log-Log Scatterplot: Income vs. Crime Rate",
           x = "Log(Income)", y = "Log(Crime Rate)")

![](Figs/unnamed-chunk-9-1.png)

    cor(log(df_merged$income), log(df_merged$crime_rate), use = "complete.obs")

    ## [1] -0.3522739

This scatterplot visualizes the relationship between log-transformed
income and log-transformed crime rate. A negative trend and correlation
is observed which means that as income increases, crime rates tend to
decrease. As mentioned previously, we used the log transformed variables
since both the data for income and crime rates were highly right skewed.
This reduces the effect of outliers, allows for a clearer trend in the
relationship and makes interpretation more meaningful in percentage
changes. As seen on the plot, countries with higher income generally
have lower crime rates. However, the spread of points shows that crime
rates vary significantly even among countries with similar income levels
which could mean that other factors such as education, governance and
inequality might also influence crime rate. In general, countries with
lower log(income) on the left side shows higher and more dispersed crime
rates. Some have crime rates above log(3.0) which translates to over 20
homicides per 100,000 people. Countries with higher log(income) on the
right side tend to cluster at lower crime rates supporting the
hypothesis that wealthier nations experience lower crime. However, a few
outliers exist, showing some high-income nations with unexpectedly high
crime rates.

    ggplot(df_merged, aes(x = literacy_rate, y = log(crime_rate))) +
      geom_point() +
      geom_smooth(method = "lm", se = FALSE, color = "blue") +
      labs(title = "Education Level vs. Crime Rate",
           x = "Literacy Rate (%)",
           y = "log(crime_rate)")

![](Figs/unnamed-chunk-10-1.png)

    cor(df_merged$literacy_rate, log(df_merged$crime_rate), use = "complete.obs")

    ## [1] -0.3122335

As mentioned, the crime rate was log transformed due to the heavy right
skew. This scatterplot shows the relationship between literacy rate and
the log(crime\_rate). The regression line shows a slight negative
correlation which means that countries with higher literacy rates tend
to have lower crime rates. However, the correlation seems to be bit
weaker than the previous plot with a noticeable variation in crime rates
even among countries with similar literacy rates. Countries with lower
literacy rate have higher and more dispersed crime rates with some above
log(3.0) which corresponds to more than 20 homicides per 100,000 people.
Countries with higher literacy rate seems to have generally lower and
more clustered crime rates with few outliers. However, there are some
exceptions where high literacy countries still experience relatively
high crime rates which means that other factors such as income
inequality, governance, or social instability might also play a role. In
summarny,a higher literacy rate might contribute to lower crime rates
due to more economic opportunities and reduced poverty driven crimes,
but the spread of data points shows that literacy alone does not fully
explain crime rates.

For this part, we will be using the classifications from 2015 written on
the worldbank website to divide the categorical variables.
<https://blogs.worldbank.org/en/opendata/new-country-classifications#>:~:text=As%20of%201%20July%202015,GNI%20per%20capita%20of%20%2412%2C736

    df_final <- df_merged %>%
      mutate(income_group = case_when(
        income < 1045 ~ "Low Income",
        income >= 1045 & income < 4125 ~ "Lower-Middle Income",
        income >= 4125 & income < 12736 ~ "Upper-Middle Income",
        income >= 12736 ~ "High Income"
      ))

    df_income_count <- df_final %>%
      count(income_group)

    ggplot(df_income_count, aes(x = income_group, y = n, fill = income_group)) +
      geom_bar(stat = "identity") +
      labs(title = "Number of Countries in Each Income Group",
           x = "Income Group", y = "Number of Countries") +
      theme_minimal() +
      theme(axis.text.x = element_text(angle = 30, hjust = 1))  

![](Figs/unnamed-chunk-11-1.png) As we can see, the largest category
consists of Upper-Middle Income countries which make up the highest
number in this dataset. Also, a significant portion of the analyzed
countries are categorized as Lower-Middle Income countries. We can also
see that there are only few high and low income countries.

    ggplot(df_final, aes(x = income_group, y = log(crime_rate))) +
      geom_boxplot(fill = "blue") +
      labs(title = "Crime Rate by Income Group", x = "Income Level", y = "Crime Rate")

![](Figs/unnamed-chunk-12-1.png)

    df_final1 <- df_merged %>%
      mutate(literacy_group = ifelse(literacy_rate > median(literacy_rate, na.rm = TRUE), "High", "Low"))

    ggplot(df_final1, aes(x = literacy_group, y = log(crime_rate))) +
      geom_boxplot(fill = "purple") +
      labs(title = "Crime Rate by Literacy Group", x = "Literacy Level", y = "log(Crime Rate)")

![](Figs/unnamed-chunk-12-2.png) Looking at the income boxplot,
High-Income Countries have the lowest median crime rate and a smaller
spread which means less variability in crime levels. We can see that
Low-Income Countries show higher median crime rates showing a possible
link between economic hardship and crime. Lower-Middle and Upper-Middle
Income Groups have higher crime rates and larger variability meaning
that crime levels fluctuate more within these groups. The longer
whiskers and spread-out data in some groups means that some countries
have exceptionally high or low crime rates relative to their income
level. We may need further analysis to explore whether factors like
income inequality or education levels contribute to crime patterns.
Low-Literacy Countries tend to have higher crime rates with a higher
median and bigger spread than high-literacy countries. High-Literacy
Countries have a lower median crime rate and less variation though a few
outliers exist. The “NA” category represents countries with missing
literacy data which shows a wider distribution of crime rates. We did
not clean this category since we are working with a small dataset.

    ggplot(df_final, aes(x = log(crime_rate), fill = income_group)) +
      geom_density(alpha = 0.5) +
      labs(title = "Density Plot: Crime Rate by Income Group",
           x = "Log(Crime Rate)", y = "Density") +
      theme_minimal()

![](Figs/unnamed-chunk-13-1.png) In the green region meaning Low-Income
Countries, the crime rate distribution is highly concentrated around
log(Crime Rate) ≈ 2.0. This shows taht most low-income countries have
similar crime rates. In the light blue region meaning Lower-Middle
Income, the crime rate distribution is broader with peaks around
log(Crime Rate) ≈ 1.5 to 2.0. There is more variation in crime rates
among lower-middle-income countries compared to low-income countries. In
the purple region which represents the Upper-Middle Income, it shows a
wider spread in crime rates with some countries having very low crime
rates (log ≈ 0.5) and some with high crime (log &gt; 3). The
distribution is more evenly spread compared to Low-Income or
Lower-Middle income countries. In the red/pink region which are
High-Income countries, the crime rate distribution is more dispersed and
generally lower with a peak closer to log(Crime Rate) ≈ 0.5 to 1.0. This
suggests that crime rates fluctuate significantly within this group
potentially due to external factors like inequality, governance, or
urbanization.

    df_final2 <- df_merged %>%
      mutate(log_income = log(income),
             log_crime_rate = log(crime_rate))

    cor <- df_final2 %>%
      select(log_income, log_crime_rate, literacy_rate) %>%
      cor(use = "complete.obs")

    print(cor)

    ##                log_income log_crime_rate literacy_rate
    ## log_income      1.0000000     -0.2434619     0.9042342
    ## log_crime_rate -0.2434619      1.0000000    -0.3122335
    ## literacy_rate   0.9042342     -0.3122335     1.0000000

log(Income) and Literacy Rate ≈ 0.904 which means a strong positive
correlation. It means that higher-income countries tend to have higher
literacy rates. This makes sense because wealthier nations invest more
in education, infrastructure, and public services, leading to improved
literacy levels. This relationship is the strongest in the matrix
implying that economic growth is strongly linked to education.
log(Income) and Crime Rate ≈ 0.243 which means a weak negative
correlation. This means that higher-income countries tend to have lower
crime rates. However, the correlation is weak meaning that while income
might influence crime, other factors such as inequality, governance, or
social policies plays a significant role. This aligns with previous
boxplots and density plots where Low-Income countries had consistently
higher crime rates, but variation existed in Middle-Income and
High-Income countries. Literacy Rate and Crime Rate ≈ -0.312 which means
a Weak to moderate negative correlation. This means that countries with
higher literacy rates tend to have lower crime rates. While this
relationship is stronger than income-crime, it is still not very strong.
As explained, Other factors may be involved in this variaton.

    library(ggcorrplot)
    cor_matrix <- df_final2 %>%
      select(log_income, log_crime_rate, literacy_rate) %>%
      cor(method = "spearman", use = "complete.obs")

    ggcorrplot(cor_matrix, lab = TRUE, title = "Correlation Heatmap")

![](Figs/unnamed-chunk-15-1.png) We use the Spearman method and a
heatmap for this. Income and Literacy Rate is 0.86 which shows a strong
positive correlation. This suggests that higher-income countries tend to
have higher literacy rates. This relationship is expected, as wealthier
countries typically invest more in education, leading to higher literacy
levels. Income and Crime Rate is -0.24 which shows a weak negative
correlation. This suggests that higher-income countries tend to have
lower crime rates, but the relationship is not strong. This implies that
income alone does not fully explain crime rates, and other factors plays
a role. Literacy Rate and Crime Rate is -0.40 which is a moderate
negative correlation. This shows that higher literacy rates tend to have
lower crime rates. While literacy appears to have a stronger negative
correlation with crime than income, the relationship is still not
definitive, meaning other factors might also influence crime rates as
explained.

### Summary

The goal of this exploratory data analysis was to investigate the
relationship between income, education levels, and crime rates across
different regions/countries. The analysis used World Bank API data to
retrieve GDP per capita, literacy rate, and intentional homicide rates.

Some key findings were: Income and Literacy Rate: A strong positive
correlation of 0.86 was found between income and literacy rate,
confirming that wealthier countries tend to have higher literacy levels.
Income and Crime Rate: A weak negative correlation of -0.24 suggested
that higher-income countries generally have lower crime rates, but the
relationship is not strong. Literacy and Crime Rate: A moderate negative
correlation -0.40 showed that higher literacy rates are associated with
lower crime rates, but other factors may also influence crime levels.
Distribution of Crime Rates: Low-income countries showed higher crime
rates with less variation. Middle-income countries had the highest
variation in crime which meant that income alone does not fully explain
crime trends. High-income countries exhibited generally lower crime
rates but still showed some variation. Boxplots & Density Plots: These
visualizations confirmed that crime rates tend to be higher in
lower-income and lower-literacy countries, but exceptions exist.

For the final project, I plan to build on these findings by———————————–

Statistical testing: I would like to dive deeper on Spearman’s
correlation to verify relationships while accounting for non-linear
trends. I would also like to do Shapiro-Wilk test (learned from STA355)
to test for normality, look at residual plots and do weighted regression
if needed.

Regression Analysis: I would like to conduct multiple regression to
determine whether income or literacy rate is a stronger predictor of
crime rates. I could also introduce the Gini Index (income inequality)
(learned from STA355) to examine its impact on crime.

Additional Visualizations: With a bigger dataset, I want to create a
geospatial crime maps to visualize patterns across regions and compare
crime rates across regions.

I could also dive deeper on interpretations and discuss potential
socioeconomic policies that could reduce crime rates based on findings.
