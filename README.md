# A2 - Bias in Data Assignment

#### Data 512: Human Centered Data Science
#### Aaliyah Hänni
#### 10/7/2021


## Project Overview
The goal of this assignment is to explore the concept of bias through data on Wikipedia articles - specifically, articles on political figures from a variety of countries. Wikipedia articles and country populations datasets are combined, and ORES is used to estimate the quality of each article by country.

This notebook contains step-by-step analysis, from data aquisition to results, of how the coverage of politicians on Wikipedia and the quality of articles about politicians varies between countries.The 'Results' section of this notebook contains tables that display:

1. the countries with the greatest and least coverage of politicians on Wikipedia compared to their population.
2. the countries with the highest and lowest proportion of high quality articles about politicians.
3. a ranking of geographic regions by articles-per-person and proportion of high quality articles.

In the 'Reflection' section contains a short reflection on the project that focuses on how both findings from this analysis and the process we went through to reach the findings, helped me to understand the causes and consequences of biased data in large, complex data science projects.


## Data Source
There are two data sources used for this analysis, one for the Wikipedia articles and another for the world population. The Wikipedia politicians by country dataset can be found on Figshare. The population data is available in CSV format as WPDS_2020_data.csv. This dataset is drawn from the world population data sheet published by the Population Reference Bureau.

#### Data Source #1: Politicians by Country from the English-language Wikipedia
Keyes, Os (2017): Politicians by Country from the English-language Wikipedia. figshare. Dataset. https://doi.org/10.6084/m9.figshare.5513449.v6 

This project contains data on most English-language Wikipedia articles within the category "Category:Politicians by nationality" and subcategories, along with the code used to generate that data. Both are released under the CC-BY-SA 4.0 license.

Data
The data was extracted via the Wikimedia API using the associated code. It is formatted as a CSV and saved as page_data.csv in the "data" directory. Columns are:

1. "country", containing the sanitised country name, extracted from the category name;
2. "page", containing the unsanitised page title.
3. "last_edit", containing the edit ID of the last edit to the page.

Data Source: https://figshare.com/articles/dataset/Untitled_Item/5513449

Keyes, Os (2017): Politicians by Country from the English-language Wikipedia. figshare. Dataset. https://doi.org/10.6084/m9.figshare.5513449.v6

#### Data Source #2: World Population Data Sheet
This dataset was extracted from the Population Reference Bureau. It contains the world population counts by region for 2019.

Columns are: 
1. "FIPS", contains the Federal Information Processing Standards codes for place
2. "Name", contains the name of the place
3. "Type" , contains the type of place: World, Sub-Region, World
4. "TimeFrame", contains the year (2019)
5. "Data (M)", contains the population count in millions
6. "Population", contains the population count

About the data: https://www.prb.org/international/indicator/population/table/ 

Data Source:
https://docs.google.com/spreadsheets/d/1CFJO2zna2No5KqNm9rPK5PCACoXKzb-nycJFhV689Iw/edit#gid=283125346

## Data Processing
Both page_data.csv and WPDS_2020_data.csv contain some rows need to filter out and/or ignored when combining the datasets below. In the case of page_data.csv, the dataset contains some page names that start with the string "Template:". Note that these pages are not Wikipedia articles, and should not be included in the analysis.

Similarly, WPDS_2020_data.csv contains some rows that provide cumulative regional population counts, rather than country-level counts. These rows are distinguished by having ALL CAPS values in the 'geography' field (e.g. AFRICA, OCEANIA). These rows won't match the country values in page_data.csv, but will be retained (either in the original file, or a separate file) so that we can report coverage and quality by region in the analysis section.

## Getting Article Quality Predictions
To get the predicted quality scores for each article in the Wikipedia dataset, we're using a machine learning system called ORES. This was originally an acronym for "Objective Revision Evaluation Service" but was simply renamed “ORES”. ORES is a machine learning tool that can provide estimates of Wikipedia article quality. The article quality estimates are, from best to worst:

1. FA - Featured article
2. GA - Good article
3. B - B-class article
4. C - C-class article
5. Start - Start-class article
6. Stub - Stub-class article

These were learned based on articles in Wikipedia that were peer-reviewed using the Wikipedia content assessment procedures.These quality classes are a sub-set of quality assessment categories developed by Wikipedia editors. 

In order to get article predictions for each article in the Wikipedia dataset, we will first need to read page_data.csv into Python, and then read through the dataset line by line, using the value of the rev_id column to make an API query.

ORES REST API - 
Documentation: https://ores.wikimedia.org/v3/#!/scoring/get_v3_scores_context_revid_model

Note: The ORES API returns a prediction value that contains the name of one category, as well as probability values for each of the 6 quality categories. For this assignment, we 
only need to capture and use the value for prediction. 

### Missing Predictions

It is important to mention that some Wikipedia artcles are not able to receive a score from the ORES api. Due to limits in the ORES model, there are inconclusive predictions. The list of Wikipedia articles that were not able to receive a valid score are stored in a csv file labled, "wikipedia_politcian_pages_no_ORES_pred.csv", and contains the following columns: 

1. page, the name of the Wikipedia article
2. country, the assocaited country that the article politician represents
3. rev_id, the unique id used to identify the article

## Combining Datasets
Given the predictions and the two distinct data sets, of Wikipedia articles and country population, we will now need to merge them all together into a final dataset. This final dataset is labeled, 'wp_wpds_politicians_by_country.csv' and contains the following columns:
1. country, the associated country name
2. article_name, the Wikipedia article name
3. revision_id, the unique revision id of the Wikipedia article
4. article_quality_est, the predicted article quality (FA) or (GA)
5. population, the country population

As a result, there is several data that cannot be matched because of missing predictions, null populations, or null articles. The data that was not able to be merges is contained in the file labeled, 'wp_wpds_countries-no_match.csv', and contains the following columns:
1. country, the associated country name
2. page, the Wikipedia article name
3. rev_id, the unique revision id of the Wikipedia article
4. pred, the predicted article quality (FA) or (GA)
5. population, the country population
6. FIPS, contains the Federal Information Processing Standards codes for place
7. Name, contains the name of the place
8. TimeFrame, contains the year (2019)
9. Data (M), contains the population count in millions

## Analysis
The analysis below consists of several calculations and merges in order to get the concluding datasets used in the results section. Speficially, below calculates the proportion of articles-per-population and high-quality articles for each country AND for each geographic region. These calculations allow us to conclude the countires with the best and worst coverage of articles to their population, and proportion of good articles to total articles. We then do the same anaylsis by region.

By "high quality" articles, in this case we mean the number of articles about politicians in a given country that ORES predicted would be in either the "FA" (featured article) or "GA" (good article) classes.

## Results
1. Top 10 countries by coverage: 10 highest-ranked countries in terms of number of politician articles as a proportion of country population
    1. Tuvalu
    2. Nauru
    3. San Marino
    4. Monaco
    5. Liechtenstein
    6. Marshall Islands
    7. Tonga
    8. Iceland
    9. Andorra
    10. Federated States of Micronesia

2. Bottom 10 countries by coverage: 10 lowest-ranked countries in terms of number of politician articles as a proportion of country population
    1. India
    2. Indonesia
    3. China
    4. Uzbekistan
    5. Ethiopia
    6. Zambia
    7. Korea, North
    8. Thailand
    9. Mozambique
    10. Bangladesh

3. Top 10 countries by relative quality: 10 highest-ranked countries in terms of the relative proportion of politician articles that are of GA and FA-quality
    1. 'Korea, North',
    2. 'Saudi Arabia', 
    3. 'Romania', 
    4. 'Central African Republic',
    5. 'Uzbekistan', 
    6. 'Mauritania', 
    7. 'Guatemala', 
    8. 'Dominica', 
    9. 'Syria'
    10. 'Benin'
       
4. Bottom 10 countries by relative quality: 10 lowest-ranked countries in terms of the relative proportion of politician articles that are of GA and FA-quality
    1. 'Solomon Islands', 
    2. 'Tonga', 
    3. 'Nauru', 
    4. 'Namibia', 
    5. 'Djibouti',
    6. 'Mozambique', 
    7. 'Monaco', 
    8. 'Eritrea', 
    9. 'Estonia', 
    10. 'Moldova'

5. Geographic regions by coverage: Ranking of geographic regions (in descending order) in terms of the total count of politician articles from countries in each region as a proportion of total regional population

    1. 'OCEANIA', 
    2. 'NORTHERN EUROPE', 
    3. 'SOUTHERN EUROPE', 
    4. 'WESTERN EUROPE',
    5. 'CARIBBEAN', 
    6. 'EASTERN EUROPE', 
    7. 'SOUTHERN AFRICA', 
    8. 'WESTERN ASIA',
    9. 'CENTRAL AMERICA', 
    10. 'SOUTH AMERICA', 
    11. 'EASTERN AFRICA', 
    12. 'WESTERN AFRICA',
    13. 'NORTHERN AMERICA', 
    14. 'MIDDLE AFRICA', 
    15. 'NORTHERN AFRICA', 
    16. 'CENTRAL ASIA',
    17. 'SOUTHEAST ASIA', 
    18. 'SOUTH ASIA', 
    19. 'EAST ASIA'

6. Geographic regions by coverage: Ranking of geographic regions (in descending order) in terms of the relative proportion of politician articles from countries in each region that are of GA and FA-quality

    1. 'WESTERN EUROPE', 
    2. 'SOUTH AMERICA', 
    3. 'EASTERN AFRICA', 
    4. 'SOUTHERN AFRICA',
    5. 'CENTRAL AMERICA', 
    6. 'SOUTH ASIA', 
    7. 'WESTERN AFRICA', 
    8. 'CARIBBEAN',
    9. 'SOUTHERN EUROPE',         
    10. 'OCEANIA', 
    11. 'NORTHERN AFRICA', 
    12. 'MIDDLE AFRICA',
    13. 'NORTHERN EUROPE', 
    14. 'CENTRAL ASIA', 
    15. 'EAST ASIA', 
    16. 'EASTERN EUROPE',
    17. 'WESTERN ASIA', 
    18. 'SOUTHEAST ASIA', 
    19. 'NORTHERN AMERICA'

## Writeup: Reflections and Implications
I expected the top 10 countries by the number of articles to the population to contain several larger more popular countries, such as the United States, China, and Russia. The reasoning for is expectation, is that in my experience, the larger populated countries tend to disproportionately populate international news sources and be more well known, and consequently, politicians in these larger countries, tend to be more famous. The larger the following of a politician, the more thorough their Wikipedia article I would expect to be, and the more populated a country is, the more politician articles they will likely have. The results in fact included only small countries in terms of population ( < 400,000 ). This is possibly indicative of strong bias that I had towards the more I hear about a country, the large I expected everything to be, including politician's Wikipedia articles. In retrospect, the results do make sense as this is likely due to the low population allowing the total number of articles to have more weight. 

Similarly to the above arguement, I expected to find a heavy bias in the proportion of good to total politician Wikipedia articles, that more popular countries will have a better proportion. I found that countries that had the best proportion of good to total, were countries that are fairly popular, but have small populations, such as North and South Korea, Romania, and the Central African Republic. Several countries also were found to have no good articles, and this consisted of mostly small third world countries that are not very popular.

One possible issue with this analysis, is that a larger population does not imply that there is a larger number of politicians, and the ratio is dipropionate when looking at coverage. This raises some questions about the accurate comparisons with the analysis performed above, because we are not standardizing for the population size. For example, the United States and Aruba might have close to the same number of politician articles, but have a several magnitude difference in population, therefore Aruba would rank far higher. Thus, I would expect that the coverage will be fairly constant for most countries, but looking at the proportion of articles to population, heavily skews this.

A potential issue for consideration, is that we are only analyzing articles written in English, and thus there may be some heavy bias about the number of articles for English speaking countries. It would be interesting to redo this analysis but using each country’s native language to count the number of Wikipedia articles.

For further analysis, I am curious to explore the opposite end of the Wikipedia article spectrum, and analyze in more detail how countries compare based on the number of poor articles, and how the article quality varies by country or region. 


## Cited Sources

https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop.html

https://pandas.pydata.org/docs/reference/api/pandas.Series.str.isupper.html

https://datascienceparichay.com/article/pandas-groupby-count-of-rows-in-each-group

https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.sort_values.html
