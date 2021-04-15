


# Project 3:  Web APIs & NLP. Trekkies vs. Warsies

Author: Artem Lukinov

### [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#contents)Contents:

-   [Problem Statement](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Problem-Statement)
-   [Executive Summary](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Executive-Summary)
-   [Data Dictionary](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Data-Dictionary)
-   [Collecting the Data](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Collecting-the-Data)
-   [Data Cleaning](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Data-Cleaning)
-   [Pre-processing](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Pre-processing)
-  [Initial EDA](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Initial-EDA)
-  [The Modeling Process](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#The-Modeling-Process)
-   [Conclusions and Recommendations](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main#Conclusions-and-Recommendations)

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Problem Statement

This project is exploring two subreddits: r/StarWars and r/startrek to identify common terms, trends and differences between the two groups. The model built as a result helps classify posts into correct subreddits.

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Executive Summary

Almost everyone has heard at least once about Star Wars and Star Trek universes. The Star Trek TV series was released in 1966 and was followed by Star Wars launch in 1977. Since then, countless episodes, movies, cartoons, games and other media have been released under both titles. Each universe has an almost cult following that has their own conventions, competitions and, of course, online presence. Two of those are on Reddit. Star Wars subreddit has 1.8m active members and Star Trek has over 300k trekkies on it. This project uses data from those subreddits to develop a Natural Language Processing model to identify which subreddit a certain post belongs to. The goal is to identify how similar or different those subreddits are.  
## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Data Dictionary
After collecting and cleaning the data, this was the final Data Dictionary:

| Column Name | Description |	Data type	|
|--|--|--|
| subreddit | Name of the subreddit | string |
| title | Title of the post | string |
| title_tok | Tokenized title | object |
| title_char_c | Character count of the title | int64 |
| title_word_c | Word count of the title | int64 |
| is_SW | Binary representation of subreddit name. Star Wars = 1, Star Trek = 0 | int64 |

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Collecting the Data
In order to scrape the posts data from Reddit, this project used Pushshift's API. At first, a 1,000 posts from each subreddit were downloaded, but after cleaning that number was increased to 2,000 per subreddit to total 4,000 entries. This was accomplished by creating a helper function that went through the subreddit's pages and delivered 100 posts at a time (a maximium set by the API) 10 times (length of the list), using our parameters, using the last item to determine the time it was created and setting that time as the start of the new scrape. Then combining all 20 dataframes into one. The files were then exported to the data folder. While adjusting workflow of the project, this process had to be split out into a separate Jupyter notebook, which is titled "1. Data Scrape.ipynb". This helped alleviate the problem of the data being re-pulled every time the code would run. 

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Data Cleaning
After the two dataframes were concatenated together, a thorough review of all 80+ columns was performed to identify potential features for the model. Three columns were kept initially:

 - Subreddit title
 - Title content
 - The text of the post

After much consideration to keep both text columns, the text of the post had over 37% missing values (1494 out of 4000) and the column was dropped. The concentration of the analysis fell on the title text since this represent the behavior of a typical Reddit user - most of the information is presented or asked in the title, often accompanied by a media display (picture, link to video, etc.) This caused an idea to check for any atypical entries in the column.

Some of the entries were blank and represented a media post with no title. Those were identified via simple sort and dropped by corresponding Index value.

After initial data cleaning, 3,978 rows were left and 124 removed. Further look at the removed values identified that they were equally split between subreddits - a welcome coincidence that stratified the predicted variable.

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Pre-processing

 1. Further exploration identified multiple titles with emojis and those characters were removed by applying a helper formula via a lambda function.
 2. Since it would be easy to identify a subreddit by the combination of 'Star Wars' and 'Star Trek', those were removed, leaving single words intact.
 3. URLs were also removed
 4. Lemmatization was applied to the title column and an extra column with lemmatized values was added.
 5. Standard English stopwords from nltk were augmented with extra words that were discovered as part of the analysis and a custom list was created.

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Initial EDA

Word clouds are not a very clean way to visualize text data as the length of certain terms may overpower the significance, but it was a helpful way to see the data without  any cleaning and allowed to identify some trends. Let's see what the word cloud would look like before any cleaning is applied:

![Star Wars word cloud before data cleaning](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/SW_word_cloud.png)

![Star Trek word cloud before data cleaning](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/ST_word%20_cloud.png)

The default stopwords() parameter did a good job removing most stop words. An interesting observation is that 'episode', 'series', 'good', 'best', 'movie', 'favorite' are significant in both sets. A closer look later in EDA identified that these words were not in the top common words between subreddits.

Let's look at word counts and character counts to determine potential differences between subreddits. Looking at the character count first:


![character count histogram](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/char_table.png)

![character count](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/char_count_table.png)

Star Trek titles are a bit longer at 57 characters, compared to 54 in Star Wars. Also, if doing train-test splits we will not need to stratify as it seems that the empty 22 rows that were previously removed were split in half between the subreddits.

Now the word count:

![word count histogram](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/word_table.png)

![word count](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/word_count_table.png)

Same trends in title word count. Star Trek and Star Wars fans are not that different so far.

Next step is to vectorize the title column in order to perform additional EDA. Count Vectorizer was used to achieve that since we want to look at a simple word count in vectorized forms. 976 feature names were produced as a result of vectorization. This process helped identify the top 20 words in each subreddit. 

![Top 20 words in Star Wars subreddit](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/Top_words_SW.png) 

No surprises here with most generally Star Wars themed words occupying top spots. The term "mandalorian" is of interest here. It occupies 7th spot and it represents how new the data is as it was the latest release in Star Wars Universe.

![Top 20 words in Star Trek subreddit](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/Top_words_ST.png)

Star Trek top terms actually mirror Star Wars terms. The term "picard" is number 8 on the list and corresponds to the newest release in Star Trek franchise. One more proof that the two subreddits are very similar. Naturally, the next step was to identify if there are any top words that the two share. At first, this was done with just 20 top words, but the results didn't return many similarities since the subject-specific names are very unique. Once the list of top words was expanded to 50, there were some common words:

**

## ['like',  'one',  'anyone',  'new',  'would',  'think',  'first',  'time',  'best']

**
The words overlap proves that the subreddits have different vocabularies as the common words in both of them are general terms that cannot identify a subreddit.

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)The Modeling Process

1.  The dataset was split into train and test sets in a 2/3 ratio. Although stratification was not required as our y-variable was evenly distributed between 2 subreddits, the parameter was set to 'yes'.
2. Two models were identified for this project: Multinomial Naive Bayes and Random Forrest. The Random Forrest models are typically overfit but produce better classification results, and Multinomial NB do no suffer as much form being overfit. This polar scenario was very interesting to observe and compare.
3.  Pipeline was created to search the best parameters for the models and grid search was performed on both.
4. After fitting the models the following results were observed:

![model comparison](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/model_comparison.png)

The Multinomial Naive Bayes performed better than Random Forrest model. The MNB model achieved test accuracy score of 85.3 compared to 81.4 via Random Forest. Both models are still slightly overfit, but the difference between train and test scores is smaller in MNB model.

![model comparison](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/corr_matrix.png)

The Multinimial Naive Bayes specificity is 82.6%. When the model predicted that the post is from Star Trek, it got it right 82.6% of the time. Sensitivity came in at 88%. When the model predicted Star Wars as a subreddit, it got it correct 88% of the time.

## [](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main)Conclusions and Recommendations

The best way to interpret the model's performance is to look at the coefficients. The coef_ attribute of MultinomialNB is a re-parameterization of the naive Bayes model as a linear classifier model. This is basically the log of the estimated probability of a feature given the positive class - Star Wars subreddit. 

Here are the top most influential terms of the model's performance:

![Top important features for Star Wars](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/Top_features_SW.png)

![Top important features for Star Trek](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/top_features_ST.png)

It's also worth taking a look at the bottom terms that influence our model the least:

![Bottom important features for Star Wars](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/Bottom_features_SW.png)

![Bottom important features for Star Trek](https://git.generalassemb.ly/krosaf4eg/dsir-125/blob/master/projects/project-03-main/images/Bottom_features_ST.png)

It is anecdotal that most of the bottom terms for each subreddit are words from the opposite universe.

The results of the model with optimal parameters shows that although the subreddits are very similar in behavior, the fact that they use unique terms in the posts helps identify which subreddit the post belongs to. 

There are definitely ways to improve this process by trying other models like Logistic Regression, Bernoulli Naive Bayes, K-nearest neighbors. They can be combined with TFIDF or Count vectorizer and evaluated via pipelines. 

Also, further analysis of bi-grams and tri-grams could be considered.

As of now, the project helped to learn difference between the two subreddits and help classify posts into correct categories.


