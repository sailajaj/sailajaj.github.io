### Keyword Trends

Keyword analysis of search terms and their effective use in site analysis have been popular with many interesting publications supporting them. Typically, businesses process these keywords to analyze user interests, popular categories, including keyword trend analysis. Most trend analysis are fairly complicated with varied language sets and are often analyzed as trends within a sub-set of categories. What we questioned was, to find a computationally simple process that could show us trends by country. A trend by location could reveal events/incidents that could explain variance of usage.

Keywords and its intent analysis is very specific to the site and its use cases. In this article, we are not exploring those details, rather, suggesting a generic way of spotting trends within streams of keyword. As an example, Google offers a daily view of the trending terms: [Google Daily Trends](https://trends.google.com/trends/trendingsearches/daily?geo=US) . Here is a trend of search term ‘_Idaho Earthquake_’ that google shows. If your business, in Idaho, is showing a behavior that is different from the norm, it is likely related to this event; a suggestion that can be revealed by analyzing keyword trends.

<figure class="graf graf--figure" name="1229">![](/images/trends-google.png)</figure>

### Type of Trends

Based on the type of data one collects, a keyword trend has different utilities. Here we identified simple three simple use cases to work with : 1\. keywords that are always-trending, 2\. keywords from spammers, 3\. keywords from a real trend.

There are popular keywords that are always trending:

<figure class="graf graf--figure" name="7e0c">![Popular keywords that have a daily trend](/images/trends-popular.png)

<figcaption class="imageCaption">Keywords that have a daily trend</figcaption>

</figure>

There are spammers that cause a trend for a short burst

<figure class="graf graf--figure" name="ba5f">![](/images/trends-spam.png)</figure>

And, then there are real trends that can give some meaningful insight:

<figure class="graf graf--figure" name="493b">![](/images/trends-true.png)

<figcaption class="imageCaption">Keywords that have a clear trend</figcaption>

</figure>

### Algorithm

To identify patterns in the keywords we are evaluating the data by the hour. A short spike (as shown in blue below) is likely a spammer. A long term pattern is a popular term. A query term that is trending in the most recent hours (as shown in pink below) is a trend worth identifying.

<figure class="graf graf--figure" name="8435">![](https://cdn-images-1.medium.com/max/1600/1*YLuOSlOTcBLhHgsn6QVO1A.png)</figure>

For calculation of hourly search trends, we calculate query count by keyword, and derive an RMSE score (Root mean square error) for the window, using a formula that weighs the trend for the current window more heavily. 

When we weight the current window more heavily than the other, the spam trends with time will have a lower RMSE score. The long term popular terms will have a consistent RMSE score with time, making it easy to evaluate and remove.

<figure class="graf graf--figure" name="8ec2">![](https://cdn-images-1.medium.com/max/1600/1*2XS1dBG5ndOCdruMeur1ZQ.png)</figure>

<figure class="graf graf--figure" name="af0e">![](https://cdn-images-1.medium.com/max/1600/1*j7djGRhznGKL1Fph3phd-w.png)</figure>

### Data Preparation

#### Data Cleanup

First step in this process is to clean the search terms by removing stop words and adult keywords.

#### Grouping of Terms

Second is to group similar keywords. More often, trends follow a collective theme that can be best identified by grouping the terms. The grouping is based on calculating cosine distance between any two search-terms and then the pairs with cosine similarity higher than a threshold (currently 0.5) are grouped together. The basics steps involved in calculating cosine similarity are:

1.  N-grams- It is a way of breaking text into smaller chunks where N is the chunk size. This allows for more overlap between strings where there are spelling mistakes or just grammar differences.
2.  TF-IDF- It is a way of converting words into numerical vectors required for calculating distances.
3.  Cosine similarity- Calculate the cosine similarity between pairs of vectors.
4.  Group Map- Utilize the cosine similarity to create a group-map between search-term and groups.
5.  Group Naming- Create more reasonable groups using most frequent words from the search terms within a group.

#### Separate always-trending terms

We filter always-trending queries from the data. The logic is to identify always-trending queries _by finding any search-term which has a frequency higher than 70% over the last 30 days is considered as always-trending_.

### Trend Analysis

#### Hourly Trend

We are using a rolling window to identify trends. Using RMSE score rate the _groups _(_always-trending groups or newly identified groups_) in the current hour. The lower the score the higher the ranking. The optimal window size turned out to be 6 hours, meaning that every hour we look back 6 hours to calculate the hourly trends.

#### Daily Trends

Daily trends are calculated from the hourly-trends data. For daily trends, we look back at previous 24 window-slices and pick those with hourly ranks higher than 30 and then rank them based on a weighted score of (Frequency x Query Counts). The search-term/group with higher weighted score being considered as trending over the day.

### Results

Grouping of our 

<figure class="graf graf--figure" name="8128">![](https://cdn-images-1.medium.com/max/1600/1*zDUFvfEfUkmKouC0Xw_bbA.png)</figure>

Trends for 12/25 : 

<figure class="graf graf--figure" name="9213">![](https://cdn-images-1.medium.com/max/1600/1*s-6VVkyv8oQ0k4TzZ9UcQw.png)</figure>
