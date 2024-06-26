# Project-Sentiment-Analysis
Data Science Project – Sentiment Analysis Project in R

## R Project -- Sentiment Analysis

This project aims to build a sentiment analysis model that will allow us to categorize words based on their sentiments, that is whether they are positive, or negative, and also the magnitude of it. Before we start with our R project, let us understand sentiment analysis in detail.
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/8cd68074-fcbf-4f31-a7b9-7458e04ca3a7)


## What is Sentiment Analysis?
Sentiment Analysis is a process of extracting opinions that have different polarities. By polarities, we mean positive, negative, or neutral. It is also known as opinion mining and polarity detection. With the help of sentiment analysis, you can find out the nature of opinion that is reflected in documents, websites, social media feeds, etc. Sentiment Analysis is a type of classification where the data is classified into different classes. These classes can be binary (positive or negative) or have multiple classes (happy, sad, angry, etc.).

## Developing our Sentiment Analysis Model in R

We will carry out sentiment analysis with R in this project. The dataset we will use will be provided by the R package 'janeaustenR'.

To build our project on sentiment analysis, we will use the tidytext package comprising sentiment lexicons present in the dataset of 'sentiments'.
```
#load the libraries
library(bookdown)
suppressPackageStartupMessages(require(tidyverse))
library(tidytext)
#check sentiments
dim(sentiments)
head(sentiments)

````
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/5e303859-6529-4247-ac60-be9b23ec98ef)
We will make use of three general-purpose lexicons like –

- AFINN
- bing
- loughran
These three lexicons make use of the unigrams. Unigrams are a type of n-gram model that consists of a sequence of 1 item, that is, a word collected from a given textual data. In the AFINN lexicon model scores the words in range from -5 to 5. An increase in negativity corresponds a negative sentiment whereas an increase in positivity corresponds to a positive one. The Bing lexicon model, on the other hand, classifies the sentiment into a binary category of negative or positive. And finally, the loughran model performs analysis of the shareholder’s reports. In this project, we will make use of the Bing lexicons to extract the sentiments from our data. We can retrieve these lexicons using the get_sentiments() function. We can implement this as follows –

```
get_sentiments("bing") %>% head()
```
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/df0a8f53-eb49-419f-a4ef-d6836f9cd33e)

## Performing Sentiment Analysis with the Inner Join

In this step, we will import our libraries 'janeaustenr', 'stringr' as well as 'tidytext'. The janeaustenr package will provide us with textual data in the form of books authored by the novelist Jane Austen. Tidytext will allow us to perform efficient text analysis on our data. We will convert the text of our books into a tidy format using unnest_tokens() function.

```
library(janeaustenr)
library(stringr)
library(tidytext)
tidy_data <- austen_books() %>%  group_by(book) %>% 
     mutate(linenumber=row_number(),
            chapter=cumsum(str_detect(text,
             regex("^chapter[\\divxlc]",
                   ignore_case=TRUE)))) %>% ungroup() %>% 
    unnest_tokens(word, text)
head(tidy_data)
dim(tidy_data)
```

We have performed the tidy operation on our text such that each row contains a single word. We will now make use of the "bing" lexicon to and implement filter() over the words that correspond to joy. We will use the book Sense and Sensibility and derive its words to implement out sentiment analysis model.
```
suppressPackageStartupMessages(library(dplyr))

positive_senti <- get_sentiments("bing") %>%
 filter(sentiment == "positive")
head(positive_senti)
```

```
#filter 
tidy_data %>%
 filter(book == "Emma") %>%
 semi_join(positive_senti) %>%
 count(word, sort = TRUE) %>% head(15) %>%  arrange(desc(n))
```
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/7f7b8843-e7e9-4afe-8d80-d110eebe063b)
From the above result, we observe many positive words like "good", "happy", "love" etc. In the next step, we will use spread() function to segregate our data into separate columns of positive and negative sentiments. We will then use the mutate() function to calculate the total sentiment, that is, the difference between positive and negative sentiment.

```
suppressPackageStartupMessages(require(tidyr))

bing <- get_sentiments("bing")
Emma_sentiment <- tidy_data %>% 
     inner_join(bing) %>% 
     count(book="Emma", index=linenumber %/% 80, sentiment) %>% 
     spread(sentiment, n, fill=0) %>% 
      mutate(sentiment=positive - negative)
head(Emma_sentiment)
```

In the next step, we will visualize the words present in the book "Emma" based on their corresponding positive and negative scores.
```
library(ggplot2)

ggplot(Emma_sentiment, aes(index, sentiment, fill=book)) +
     geom_bar(stat = "identity", show.legend = TRUE) +
     facet_wrap(~book, ncol = 2, scales = 'free_x')
```
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/c0836ca4-5aa5-4815-9e2b-2c7aa0bd61fc)

Let us now proceed towards counting the most common positive and negative words that are present in the novel.

```
counting_words <- tidy_data %>%  
     inner_join(bing) %>% 
    count(word, sentiment, sort=TRUE)
 head(counting_words)
```
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/cf3b577d-79ea-4742-b3b8-3a2f08974268)

In the next step, we will perform a visualization of our sentiment score. We will plot the scores along the axis that is labeled with both positive as well as negative words. We will use ggplot() function to visualize our data based on their scores.

```
counting_words %>%  filter(n>50) %>% 
          mutate(n=ifelse(sentiment=="negative", -n, n)) %>% 
          mutate(word=reorder(word, n)) %>% 
          ggplot(aes(word, n, fill= sentiment))+
          geom_col()+
          coord_flip()+
          labs(y="Sentiment Score")
```
In the final visualization, let us create a wordcloud that will delineate the most recurring positive and negative words. In particular, we will use the comparision.cloud() function to plot both negative and positive words in a single wordcloud as follows:
```
#packages
suppressPackageStartupMessages(require(reshape2))
suppressPackageStartupMessages(require(wordcloud))

#plot
tidy_data %>%  inner_join(bing) %>% 
  count(word, sentiment, sort = TRUE) %>% 
  acast(word~sentiment, value.var = "n", fill=0) %>% 
  comparison.cloud(colors = c("red", "dark green"), 
                   max.words = 100)
```
![image](https://github.com/LangatErick/Project-Sentiment-Analysiss/assets/124883947/5948d31d-e223-4e09-9370-d0ceae60901a)

This word cloud will enable us to efficiently visualize the negative as well as positive groups of data. Therefore, we are now able to see the different groups of data based on their corresponding sentiments. You can experiment with several other datasets like tweets to gain a comprehensive insight into sentiment analysis.

# Summary

In this blog, we went through our project of sentiment analysis in R. We learned about the concept of sentiment analysis and implemented it over the dataset of Jane Austen's books. We were able to delineate it through various visualizations after we performed data wrangling on our data. We used a lexical analyzer -- 'bing' in this instance of our project. Furthermore, we also represented the sentiment score through a plot and also made a visual report of wordcloud.


