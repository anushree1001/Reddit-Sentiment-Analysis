Reddit Project for Title and Comments Analysis
================
Anushree1001
2/26/2020

## Business Context

The “Reddit” API was taken to analyze the views against word “Trump” in
the title as well as in the comments. This project will try to find the
sentiment behind the Title and Comments and compare the results.
Additionally a Deep learning model on the same was built

## Problem Description

#### 1.HOW DO PEOPLE FROM DIFFERENT PLACES JUDGE A SAME THING?

  - Perform sentiment analysis to find the tone of comments (positive,
    negative or neutral) related to a particular search term.

#### 2.HOW TO BECOME FAMOUS?

  - Use Machine Learning models to find out which words in the existing
    title and text data help to increase number of comments.

## URL link to download the zip file of data you collected from the API

## First Part:

Data collection was done using Reddit API only, The data is stored in
below github link. There are two files in below link, one is for all the
comments and another is for all the titles having word “Trump” in it.

``` r
#install.packages("tidytext")
#install.packages("wordcloud")
#install.packages("sentimentr")
#install.packages(c("igraph", "ggforce", "ggraph"))
#install.packages("topicmodels")
#install.packages("broom")
#install.packages("tictoc")
#install.packages("spacyr")
#install.packages("foreach")
#install.packages("doParallel")
#install.packages(c("cowplot", "googleway", "ggplot2", "ggrepel","ggspatial", "libwgeom", "sf", #"rnaturalearth","rnaturalearthdata"))
#("ggridges")

library(tidytext)
library(stringr)
library(tidyverse)
library(foreach)
library(doParallel)
library(ggplot2)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
library(wordcloud)
library(reshape2)
library(kableExtra)
library(RedditExtractoR)
library(tidyr)
library(stringr)
library(magrittr)
library(dplyr)
library(lubridate)
library(tidytext)
library(ggplot2)
library(ggridges)
library(tidytext)
library(textdata)
library(plyr)
library(forcats)
library(kernlab)
library(rpart)
library(psych) 
library(caret)
library(h2o)
```

Read the data from the reddit API for all the comments having word
“Trump” in it. P.S. - Using the term “trump” for this instance, the
program will run in same way for other keyword as well.

``` r
# We have read the data from the reddit file and load it in Comments.RDa and Title.RDA file for future references.
# The code is mentioned below and commented out since it takes lot of time to load the data verytime you knit the file.

#reddit <- get_reddit(search_terms = "trump", regex_filter = "", subreddit = NA,
#           page_threshold = 2, sort_by = "comments",
#           wait_time = 2)
#save(reddit,file="Comments.Rda")

load("Comments.Rda")
summary(reddit)
```

    ##        id         structure          post_date          comm_date        
    ##  Min.   :  1.0   Length:10888       Length:10888       Length:10888      
    ##  1st Qu.:109.0   Class :character   Class :character   Class :character  
    ##  Median :221.0   Mode  :character   Mode  :character   Mode  :character  
    ##  Mean   :225.6                                                           
    ##  3rd Qu.:338.0                                                           
    ##  Max.   :500.0                                                           
    ##   num_comments    subreddit          upvote_prop       post_score    
    ##  Min.   :29662   Length:10888       Min.   :0.6100   Min.   :   656  
    ##  1st Qu.:30843   Class :character   1st Qu.:0.7500   1st Qu.: 29019  
    ##  Median :31919   Mode  :character   Median :0.8800   Median : 45692  
    ##  Mean   :35978                      Mean   :0.8382   Mean   : 46467  
    ##  3rd Qu.:39473                      3rd Qu.:0.9100   3rd Qu.: 55998  
    ##  Max.   :63230                      Max.   :0.9700   Max.   :144793  
    ##     author              user           comment_score     controversiality 
    ##  Length:10888       Length:10888       Min.   : -108.0   Min.   :0.00000  
    ##  Class :character   Class :character   1st Qu.:    2.0   1st Qu.:0.00000  
    ##  Mode  :character   Mode  :character   Median :    7.0   Median :0.00000  
    ##                                        Mean   :  244.3   Mean   :0.03766  
    ##                                        3rd Qu.:   43.0   3rd Qu.:0.00000  
    ##                                        Max.   :54597.0   Max.   :1.00000  
    ##    comment             title            post_text             link          
    ##  Length:10888       Length:10888       Length:10888       Length:10888      
    ##  Class :character   Class :character   Class :character   Class :character  
    ##  Mode  :character   Mode  :character   Mode  :character   Mode  :character  
    ##                                                                             
    ##                                                                             
    ##                                                                             
    ##     domain              URL           
    ##  Length:10888       Length:10888      
    ##  Class :character   Class :character  
    ##  Mode  :character   Mode  :character  
    ##                                       
    ##                                       
    ## 

``` r
df = reddit
```

## Data summary

There are total “17” fields in the Reddit API which we are going to use
in our analysis. 1. id - Reddit id 2. Structure - Strucure of the post
3. post\_date - Date of the post 4. comm\_date - Date of the Comment 5.
num\_comments - Total number of comments 6. subreddit - Subreddit of the
post 7. upvote\_prop - likes and dislike probability of the post 8.
Post\_score - Score of the post 9. Author - Author of the post 10. user
- User of the comment 11. Comment Score - Score of the comments given by
other users 12. Contraversiality - ContraversialityScore of the comments
given by other users 13. Comment - Comment text 14. title - Title of the
post 15. link - Link to the post 16. Domain - Domain of the post
(subreddit) 17. URL - URL to the comment

There are total 22k instances of reddit comments and 21k of reddit
titles for our analysis. We would be performing Sentiment Analysis on
Reddit Comments and Titles and try to find the tone of the Comments and
Titles. Additionallyin H2O deep learning, I am trying to find any
relationship between words used in title and the number of comments.

## Data exploration using NLP technique.

1.  Turn text to tokens.

<!-- end list -->

``` r
which_train <- sample(x = c(TRUE, FALSE), size = nrow(df),
                      replace = TRUE, prob = c(0.1, 0.9))

df1 <- df[which_train, ]
df2 <- df[!which_train, ]

tokens <- df1 %>%
  unnest_tokens(output = word, input = post_text)
tokens %>%  dplyr::count(word,sort = TRUE)
```

    ## # A tibble: 6,235 x 2
    ##    word         n
    ##    <chr>    <int>
    ##  1 trump    77205
    ##  2 https    55618
    ##  3 to       38169
    ##  4 the      30506
    ##  5 news     27853
    ##  6 u        24017
    ##  7 of       23789
    ##  8 politics 20651
    ##  9 on       18590
    ## 10 comey    18504
    ## # ... with 6,225 more rows

2.  Remove all the Stopwords

<!-- end list -->

``` r
# Remove Stop words 
sw = get_stopwords()
sw
```

    ## # A tibble: 175 x 2
    ##    word      lexicon 
    ##    <chr>     <chr>   
    ##  1 i         snowball
    ##  2 me        snowball
    ##  3 my        snowball
    ##  4 myself    snowball
    ##  5 we        snowball
    ##  6 our       snowball
    ##  7 ours      snowball
    ##  8 ourselves snowball
    ##  9 you       snowball
    ## 10 your      snowball
    ## # ... with 165 more rows

``` r
cleaned_tokens <- tokens %>%
  filter(!word %in% sw$word)
```

3.  Remove all the Numbers

<!-- end list -->

``` r
# Remove Numbers
nums <- cleaned_tokens %>% 
  filter(str_detect(word, "^[0-9]")) %>% 
  select(word) %>% unique()

cleaned_tokens <- cleaned_tokens %>% 
  filter(!word %in% nums$word)
```

4.  Plot the word frequency from the cleaned tokens

<!-- end list -->

``` r
cleaned_tokens %>%   
  dplyr::count(word, sort = T) %>%  
  dplyr::rename(word_freq = n) %>%  
  ggplot(aes(x=word_freq)) +  
  geom_histogram(aes(y=..count..), color="black", fill="blue", alpha=0.3) +
  scale_x_continuous(breaks=c(0:5,10,100,500,10e3), trans="log1p", expand=c(0,0)) + 
  scale_y_continuous(breaks=c(0,100,1000,5e3,10e3,5e4,10e4,4e4), expand=c(0,0)) +  
  theme_bw() 
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

5.  Remove all the rare words to improve the performance of text
    analytic. Lets remove words that have less than 10 appearances in
    our collection.

<!-- end list -->

``` r
# Remove Rare words
rare <- cleaned_tokens %>% 
  dplyr::count(word) %>%
  filter(n<10) %>%
  select(word) %>% unique()

cleaned_tokens <- cleaned_tokens %>% 
  filter(!word %in% rare$word)
length(unique(cleaned_tokens$word))
```

    ## [1] 5112

6.  Remove words which are meaning less and not removed in previous
    steps. For example https is used mostly but its not meaningful word
    so we removed it.

<!-- end list -->

``` r
dropwords <- c("https",".com")
cleaned_tokens <- cleaned_tokens  %>%   
                  filter(!word %in% dropwords)  
```

7.  Visualize the most common words used using wordcloud to get idea of
    mostly used words.Plot 100 most common words used.

<!-- end list -->

``` r
pal <- brewer.pal(8,"Dark2")
cleaned_tokens %>%   
  dplyr::count(word) %>%  
  with(wordcloud(word, n, random.order = FALSE, max.words = 100, colors=pal))
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

It is seen from above wordcloud that trump, politics,clinton,news are
some of the words which are mostly used words for tagging. Lets find out
further what users are thinking by doing sentiment analysis.

## Sentiment Analysis of comments for word Trump.

1.  Use 3 lexicons nrc, bing, afinn for sentiment keywords

<!-- end list -->

``` r
sent_reviews = cleaned_tokens %>% 
  left_join(get_sentiments("nrc")) %>%
  dplyr::rename(nrc = sentiment) %>%
  left_join(get_sentiments("bing")) %>%
  dplyr::rename(bing = sentiment) %>%
  left_join(get_sentiments("afinn")) %>%
  dplyr::rename(afinn = value)
```

2.  Find most common most positive and negetive words.

<!-- end list -->

``` r
bing_word_counts <- sent_reviews %>%
  filter(!is.na(bing)) %>%
  dplyr::count(word, bing, sort = TRUE)
bing_word_counts
```

    ## # A tibble: 507 x 3
    ##    word         bing         n
    ##    <chr>        <chr>    <int>
    ##  1 trump        positive 77205
    ##  2 emergency    negative 30676
    ##  3 bomb         negative 20900
    ##  4 top          positive  9837
    ##  5 suspicious   negative  9240
    ##  6 collusion    negative  8490
    ##  7 lie          negative  8216
    ##  8 attack       negative  8091
    ##  9 explosive    negative  7950
    ## 10 intelligence positive  7228
    ## # ... with 497 more rows

3.  Plot the contribution to sentiments

<!-- end list -->

``` r
bing_word_counts %>%  
  filter(n > 6000) %>%  
  mutate(n = ifelse(bing == "negative", -n, n)) %>%  
  mutate(word = reorder(word, n)) %>%  
  ggplot(aes(word, n, fill = bing)) +  
  geom_col() +  
  coord_flip() +  
  labs(y = "Contribution to sentiment")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

We can see from the graph that the positive sentiments are more than
negetive.

4.  Plot the word cloud which gives top 1000 words used from which we
    can distinguish which are negetive words and positive owrds.

<!-- end list -->

``` r
bing_word_counts_temp <- bing_word_counts  %>% filter(n > 1000 )

bing_word_counts %>% 
  inner_join(get_sentiments("bing")) %>%
  dplyr::count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("gray20", "gray80"),
                   max.words = 1000)
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

Mostly used negetive and positive words are plotted in above graph to
get basic idea of what words were used for sentiment analysis.

5.  Plot the emotions from the sentiments.

<!-- end list -->

``` r
glimpse(sent_reviews )
```

    ## Observations: 1,922,495
    ## Variables: 21
    ## $ id               <int> 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14, 14...
    ## $ structure        <chr> "4_1", "4_1", "4_1", "4_1", "4_1", "4_1", "4_1", "...
    ## $ post_date        <chr> "18-01-19", "18-01-19", "18-01-19", "18-01-19", "1...
    ## $ comm_date        <chr> "09-02-19", "09-02-19", "09-02-19", "09-02-19", "0...
    ## $ num_comments     <dbl> 29695, 29695, 29695, 29695, 29695, 29695, 29695, 2...
    ## $ subreddit        <chr> "politics", "politics", "politics", "politics", "p...
    ## $ upvote_prop      <dbl> 0.81, 0.81, 0.81, 0.81, 0.81, 0.81, 0.81, 0.81, 0....
    ## $ post_score       <dbl> 84481, 84481, 84481, 84481, 84481, 84481, 84481, 8...
    ## $ author           <chr> "PoliticsModeratorBot", "PoliticsModeratorBot", "P...
    ## $ user             <chr> "Sydthebarrett", "Sydthebarrett", "Sydthebarrett",...
    ## $ comment_score    <dbl> 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15...
    ## $ controversiality <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,...
    ## $ comment          <chr> "Quit spreading Russian propaganda I saw this in a...
    ## $ title            <chr> "Megathread: President Trump Directed His Attorney...
    ## $ link             <chr> "https://www.reddit.com/r/politics/comments/ah6gxc...
    ## $ domain           <chr> "self.politics", "self.politics", "self.politics",...
    ## $ URL              <chr> "http://www.reddit.com/r/politics/comments/ah6gxc/...
    ## $ word             <chr> "president", "president", "donald", "trump", "dire...
    ## $ nrc              <chr> "positive", "trust", NA, "surprise", NA, NA, "ange...
    ## $ bing             <chr> NA, NA, NA, "positive", NA, NA, NA, NA, NA, NA, NA...
    ## $ afinn            <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA...

``` r
sent_reviews %>%
  filter(!is.na(nrc)) %>%
  ggplot(.,aes(x=nrc)) +
  geom_bar(fill = "tomato3")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

sentimenal analysis summary for comments having word “Trump”-

## Second Part

## Find relationship for words in title and number of hits

6.  Collect data from 50 different subreddits

This step will take more than 30 mins, I saved the codes in the R file,
here ill read the data I saved.

``` r
load("title.Rda")
```

7.  Similar Data cleaning for titles as Comments

<!-- end list -->

``` r
df <- total[,c("num_comments","title")]

df <- df %>% mutate(title = as.character(title))


#count words in each title for future use
df <- df %>% mutate(n= sapply(strsplit(df$title, " "), length)  )

tokens <- df %>%  
  unnest_tokens(output = word, input = title) 

tokens %>%  dplyr::count(word, sort = TRUE)
```

    ## # A tibble: 25,682 x 2
    ##    word      n
    ##    <chr> <int>
    ##  1 the   14068
    ##  2 to     8618
    ##  3 a      8038
    ##  4 in     6550
    ##  5 of     6351
    ##  6 and    5702
    ##  7 is     4592
    ##  8 i      3883
    ##  9 for    3704
    ## 10 movie  3577
    ## # ... with 25,672 more rows

``` r
length(unique(tokens$word))
```

    ## [1] 25682

``` r
sw = get_stopwords()

cleaned_tokens <- tokens %>%  
  filter(!word %in% sw$word)

nums <- cleaned_tokens %>%   
  filter(str_detect(word, "^[0-9]")) %>%   
  select(word) %>% unique()


cleaned_tokens <- cleaned_tokens %>%   
  filter(!word %in% nums$word)

cleaned_tokens <- cleaned_tokens %>%   
  filter(!grepl("[^\u0001-\u007F]+",cleaned_tokens$word))

cleaned_tokens %>% dplyr::count(word, sort = TRUE)
```

    ## # A tibble: 24,210 x 2
    ##    word            n
    ##    <chr>       <int>
    ##  1 movie        3577
    ##  2 ai           2608
    ##  3 election     1935
    ##  4 flu          1431
    ##  5 movies       1327
    ##  6 coronavirus  1226
    ##  7 can          1140
    ##  8 s            1099
    ##  9 like         1041
    ## 10 new           971
    ## # ... with 24,200 more rows

``` r
length(unique(cleaned_tokens$word))
```

    ## [1] 24210

``` r
df <- df %>% mutate(title = as.character(title))


#count words in each title for future use
df <- df %>% mutate(n= sapply(strsplit(df$title, " "), length)  )

tokens <- df %>%  
  unnest_tokens(output = word, input = title) 

tokens %>%  dplyr::count(word, sort = TRUE)
```

    ## # A tibble: 25,682 x 2
    ##    word      n
    ##    <chr> <int>
    ##  1 the   14068
    ##  2 to     8618
    ##  3 a      8038
    ##  4 in     6550
    ##  5 of     6351
    ##  6 and    5702
    ##  7 is     4592
    ##  8 i      3883
    ##  9 for    3704
    ## 10 movie  3577
    ## # ... with 25,672 more rows

``` r
length(unique(tokens$word))
```

    ## [1] 25682

``` r
sw = get_stopwords()

cleaned_tokens <- tokens %>%  
  filter(!word %in% sw$word)

nums <- cleaned_tokens %>%   
  filter(str_detect(word, "^[0-9]")) %>%   
  select(word) %>% unique()


cleaned_tokens <- cleaned_tokens %>%   
  filter(!word %in% nums$word)

cleaned_tokens <- cleaned_tokens %>%   
  filter(!grepl("[^\u0001-\u007F]+",cleaned_tokens$word))

cleaned_tokens %>% dplyr::count(word, sort = TRUE)
```

    ## # A tibble: 24,210 x 2
    ##    word            n
    ##    <chr>       <int>
    ##  1 movie        3577
    ##  2 ai           2608
    ##  3 election     1935
    ##  4 flu          1431
    ##  5 movies       1327
    ##  6 coronavirus  1226
    ##  7 can          1140
    ##  8 s            1099
    ##  9 like         1041
    ## 10 new           971
    ## # ... with 24,200 more rows

``` r
length(unique(cleaned_tokens$word))
```

    ## [1] 24210

``` r
rare <- cleaned_tokens %>%   
  dplyr::count(word) %>%  
  filter(n<10) %>%  
  select(word) %>% unique() 
rare
```

    ## # A tibble: 20,759 x 1
    ##    word    
    ##    <chr>   
    ##  1 ___     
    ##  2 ____    
    ##  3 _2020   
    ##  4 _should_
    ##  5 a'rollin
    ##  6 a's     
    ##  7 a.k.a   
    ##  8 a.l     
    ##  9 a.m     
    ## 10 a:e     
    ## # ... with 20,749 more rows

``` r
cleaned_tokens <- cleaned_tokens %>%   
  filter(!word %in% rare$word) 

length(unique(cleaned_tokens$word))
```

    ## [1] 3451

8.  Run sentiment analysis on titles, to see the difference.

<!-- end list -->

``` r
sent_reviews = cleaned_tokens %>%   
  left_join(get_sentiments("nrc")) %>%  
  rename(replace=c("sentiment"="nrc")) %>%
  left_join(get_sentiments("bing")) %>%  
  rename(replace=c("sentiment"="bing")) %>%
  left_join(get_sentiments("afinn")) %>%  
  rename(replace=c("value"="affin"))


bing_word_counts <- sent_reviews %>%  
  filter(!is.na(bing)) %>%  
  dplyr::count(word,bing, sort = TRUE) 

bing_word_counts %>%  
  filter(n > 5) %>%  
  mutate(n = ifelse(bing == "negative", -n, n)) %>%  
  mutate(word = reorder(word, n)) %>%  
  ggplot(aes(word, n, fill = bing)) +  
  geom_col() +  coord_flip() +  
  labs(y = "Contribution to sentiment")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

``` r
sent_reviews %>%  
  filter(!is.na(bing)) %>%  
  dplyr::count(bing, sort = TRUE) 
```

    ## # A tibble: 2 x 2
    ##   bing         n
    ##   <chr>    <int>
    ## 1 negative 18354
    ## 2 positive 15709

9.  Sentiment Analysis on words related to the number of views

<!-- end list -->

``` r
high_hit_word_list<- cleaned_tokens %>%
  ddply("word",summarize,mean=round(mean(num_comments))) %>%
  arrange(desc(mean))

sent_reviews = high_hit_word_list %>%   
  left_join(get_sentiments("nrc")) %>%  
  rename(replace=c("sentiment"="nrc")) %>%
  left_join(get_sentiments("bing")) %>%  
  rename(replace=c("sentiment"="bing")) %>%
  left_join(get_sentiments("afinn")) %>%  
  rename(replace=c("value"="affin"))

sent_reviews
```

    ##                 word mean          nrc     bing affin
    ## 1            supreme 3557     positive positive    NA
    ## 2         the_donald 3464         <NA>     <NA>    NA
    ## 3          instantly 3393         <NA> positive    NA
    ## 4               semi 3315         <NA>     <NA>    NA
    ## 5          genuinely 3145         <NA>     <NA>    NA
    ## 6            croatia 2932         <NA>     <NA>    NA
    ## 7             fucked 2908         <NA>     <NA>    -4
    ## 8            actress 2627         <NA>     <NA>    NA
    ## 9            updates 2587         <NA>     <NA>    NA
    ## 10          consider 2360         <NA>     <NA>    NA
    ## 11           england 2353         <NA>     <NA>    NA
    ## 12        ridiculous 2166        anger negative    -3
    ## 13        ridiculous 2166      disgust negative    -3
    ## 14        ridiculous 2166     negative negative    -3
    ## 15         generally 2121         <NA>     <NA>    NA
    ## 16             hated 2056         <NA> negative    -3
    ## 17                 v 2013         <NA>     <NA>    NA
    ## 18             quote 1989 anticipation     <NA>    NA
    ## 19             quote 1989     negative     <NA>    NA
    ## 20             quote 1989     positive     <NA>    NA
    ## 21             quote 1989     surprise     <NA>    NA
    ## 22            finish 1882         <NA>     <NA>    NA
    ## 23            remade 1837         <NA>     <NA>    NA
    ## 24              sold 1681         <NA>     <NA>    NA
    ## 25          enjoying 1634 anticipation positive     2
    ## 26          enjoying 1634          joy positive     2
    ## 27          enjoying 1634     positive positive     2
    ## 28          enjoying 1634        trust positive     2
    ## 29            oscars 1628         <NA>     <NA>    NA
    ## 30        megathread 1451         <NA>     <NA>    NA
    ## 31          speaking 1447         <NA>     <NA>    NA
    ## 32               wtf 1353         <NA>     <NA>    -4
    ## 33            remake 1322     positive     <NA>    NA
    ## 34          audience 1299 anticipation     <NA>    NA
    ## 35            thread 1292         <NA>     <NA>    NA
    ## 36           critics 1271         <NA> negative    -2
    ## 37           opening 1261         <NA>     <NA>    NA
    ## 38           excited 1219 anticipation positive     3
    ## 39           excited 1219          joy positive     3
    ## 40           excited 1219     positive positive     3
    ## 41           excited 1219     surprise positive     3
    ## 42           excited 1219        trust positive     3
    ## 43             facts 1218     positive     <NA>    NA
    ## 44             facts 1218        trust     <NA>    NA
    ## 45            caused 1214         <NA>     <NA>    NA
    ## 46              loud 1212         <NA> negative    NA
    ## 47           clinton 1204         <NA>     <NA>    NA
    ## 48           discuss 1199         <NA>     <NA>    NA
    ## 49            kansas 1188         <NA>     <NA>    NA
    ## 50              fifa 1187         <NA>     <NA>    NA
    ## 51         difficult 1176         fear negative    -1
    ## 52          relevant 1168     positive     <NA>    NA
    ## 53          relevant 1168        trust     <NA>    NA
    ## 54          animated 1154          joy     <NA>    NA
    ## 55          animated 1154     positive     <NA>    NA
    ## 56         bolsonaro 1141         <NA>     <NA>    NA
    ## 57             ended 1141         <NA>     <NA>    NA
    ## 58               cup 1133         <NA>     <NA>    NA
    ## 59             actor 1114         <NA>     <NA>    NA
    ## 60       quarantined 1109         <NA>     <NA>    NA
    ## 61             opens 1076         <NA>     <NA>    NA
    ## 62           totally 1068         <NA>     <NA>    NA
    ## 63             court 1062        anger     <NA>    NA
    ## 64             court 1062 anticipation     <NA>    NA
    ## 65             court 1062         fear     <NA>    NA
    ## 66            stupid 1020     negative negative    -2
    ## 67             loved 1014         <NA> positive     3
    ## 68             smoke 1011         <NA> negative    NA
    ## 69           perfect  999 anticipation positive     3
    ## 70           perfect  999          joy positive     3
    ## 71           perfect  999     positive positive     3
    ## 72           perfect  999        trust positive     3
    ## 73               cia  998         <NA>     <NA>    NA
    ## 74            matrix  994         <NA>     <NA>    NA
    ## 75         happening  988         <NA>     <NA>    NA
    ## 76             final  965         <NA>     <NA>    NA
    ## 77        zuckerberg  961         <NA>     <NA>    NA
    ## 78          finished  953         <NA>     <NA>    NA
    ## 79           studios  944         <NA>     <NA>    NA
    ## 80             proud  928 anticipation positive     2
    ## 81             proud  928          joy positive     2
    ## 82             proud  928     positive positive     2
    ## 83             proud  928        trust positive     2
    ## 84           swedish  867         <NA>     <NA>    NA
    ## 85        discussion  840     positive     <NA>    NA
    ## 86              none  816         <NA>     <NA>    NA
    ## 87          discover  815         <NA>     <NA>    NA
    ## 88            senate  811        trust     <NA>    NA
    ## 89           georgia  807         <NA>     <NA>    NA
    ## 90            toward  803         <NA>     <NA>    NA
    ## 91             voted  799         <NA>     <NA>    NA
    ## 92          grossing  794         <NA>     <NA>    NA
    ## 93             staff  786         <NA>     <NA>    NA
    ## 94             usage  783         <NA>     <NA>    NA
    ## 95               fbi  779         <NA>     <NA>    NA
    ## 96          official  775        trust     <NA>    NA
    ## 97      thanksgiving  775          joy     <NA>    NA
    ## 98      thanksgiving  775     positive     <NA>    NA
    ## 99            moment  770         <NA>     <NA>    NA
    ## 100          arrived  765         <NA>     <NA>    NA
    ## 101            often  758         <NA>     <NA>    NA
    ## 102            crowd  756         <NA>     <NA>    NA
    ## 103           muslim  744         <NA>     <NA>    NA
    ## 104           golden  732         <NA> positive    NA
    ## 105            birds  728         <NA>     <NA>    NA
    ## 106            match  720         <NA>     <NA>    NA
    ## 107           missed  712         <NA> negative    -2
    ## 108            putin  702         <NA>     <NA>    NA
    ## 109            rated  702         <NA>     <NA>    NA
    ## 110         colorado  693         <NA>     <NA>    NA
    ## 111         deadpool  674         <NA>     <NA>    NA
    ## 112           piracy  667     negative     <NA>    NA
    ## 113            stuck  666         <NA> negative    -2
    ## 114       neutrality  664        trust     <NA>    NA
    ## 115          destroy  661         <NA> negative    -3
    ## 116         russians  657         <NA>     <NA>    NA
    ## 117             mind  649         <NA>     <NA>    NA
    ## 118             hole  640         <NA>     <NA>    NA
    ## 119              eve  636         <NA>     <NA>    NA
    ## 120       propaganda  635     negative negative    -2
    ## 121             hack  632         <NA> negative    NA
    ## 122          mexican  619         <NA>     <NA>    NA
    ## 123             went  619         <NA>     <NA>    NA
    ## 124            worst  616         <NA> negative    -3
    ## 125         activist  613         <NA>     <NA>    NA
    ## 126             mess  606      disgust negative    -2
    ## 127             mess  606     negative negative    -2
    ## 128        announces  605         <NA>     <NA>    NA
    ## 129          founder  604         <NA>     <NA>    NA
    ## 130             hurt  603        anger negative    -2
    ## 131             hurt  603         fear negative    -2
    ## 132             hurt  603     negative negative    -2
    ## 133             hurt  603      sadness negative    -2
    ## 134             seen  600         <NA>     <NA>    NA
    ## 135             sony  592         <NA>     <NA>    NA
    ## 136       referendum  588         <NA>     <NA>    NA
    ## 137         congress  587      disgust     <NA>    NA
    ## 138         congress  587        trust     <NA>    NA
    ## 139            price  587         <NA>     <NA>    NA
    ## 140            notes  586         <NA>     <NA>    NA
    ## 141          welcome  581         <NA> positive     2
    ## 142       department  579         <NA>     <NA>    NA
    ## 143      controversy  578     negative negative    NA
    ## 144             upon  578         <NA>     <NA>    NA
    ## 145   administration  577         <NA>     <NA>    NA
    ## 146           bodies  576         <NA>     <NA>    NA
    ## 147             shit  576        anger negative    -4
    ## 148             shit  576      disgust negative    -4
    ## 149             shit  576     negative negative    -4
    ## 150         domestic  571         <NA>     <NA>    NA
    ## 151          highest  571 anticipation     <NA>    NA
    ## 152          highest  571         fear     <NA>    NA
    ## 153          highest  571          joy     <NA>    NA
    ## 154          highest  571     negative     <NA>    NA
    ## 155          highest  571     positive     <NA>    NA
    ## 156          highest  571     surprise     <NA>    NA
    ## 157             pick  569     positive     <NA>    NA
    ## 158              mil  568         <NA>     <NA>    NA
    ## 159         services  566         <NA>     <NA>    NA
    ## 160          percent  563         <NA>     <NA>    NA
    ## 161          samsung  561         <NA>     <NA>    NA
    ## 162             bush  560         <NA>     <NA>    NA
    ## 163           famous  560     positive positive    NA
    ## 164           agents  559         <NA>     <NA>    NA
    ## 165           corner  556         <NA>     <NA>    NA
    ## 166            avoid  555         fear     <NA>    -1
    ## 167            avoid  555     negative     <NA>    -1
    ## 168           pirate  555        anger     <NA>    NA
    ## 169           pirate  555     negative     <NA>    NA
    ## 170        equipment  552         <NA>     <NA>    NA
    ## 171       homecoming  548         <NA>     <NA>    NA
    ## 172           values  547         <NA>     <NA>    NA
    ## 173       legitimate  543         <NA>     <NA>    NA
    ## 174               oh  543         <NA>     <NA>    NA
    ## 175        otherwise  543         <NA>     <NA>    NA
    ## 176       personally  543         <NA>     <NA>    NA
    ## 177           access  541         <NA>     <NA>    NA
    ## 178        overrated  538         <NA> negative    NA
    ## 179         breaking  536         <NA> negative    NA
    ## 180           scenes  536         <NA>     <NA>    NA
    ## 181         parasite  535      disgust negative    NA
    ## 182         parasite  535         fear negative    NA
    ## 183         parasite  535     negative negative    NA
    ## 184            genre  533         <NA>     <NA>    NA
    ## 185          charged  530         <NA>     <NA>    -3
    ## 186            comes  529         <NA>     <NA>    NA
    ## 187            funny  528         <NA> negative     4
    ## 188        renewable  527         <NA>     <NA>    NA
    ## 189          watched  524         <NA>     <NA>    NA
    ## 190          biggest  522         <NA>     <NA>    NA
    ## 191          sheriff  521        trust     <NA>    NA
    ## 192        confirmed  519     positive     <NA>    NA
    ## 193        confirmed  519        trust     <NA>    NA
    ## 194            mitch  517         <NA>     <NA>    NA
    ## 195             zero  516         <NA>     <NA>    NA
    ## 196           mexico  514         <NA>     <NA>    NA
    ## 197           friday  513         <NA>     <NA>    NA
    ## 198        directors  511         <NA>     <NA>    NA
    ## 199           moscow  511         <NA>     <NA>    NA
    ## 200           london  508         <NA>     <NA>    NA
    ## 201          growing  505         <NA>     <NA>     1
    ## 202          indiana  503         <NA>     <NA>    NA
    ## 203           teaser  503         <NA>     <NA>    NA
    ## 204          writers  503         <NA>     <NA>    NA
    ## 205              oil  502         <NA>     <NA>    NA
    ## 206              hit  500        anger     <NA>    NA
    ## 207              hit  500     negative     <NA>    NA
    ## 208           native  500         <NA>     <NA>    NA
    ## 209             suit  496         <NA>     <NA>    NA
    ## 210           roasts  495         <NA>     <NA>    NA
    ## 211          answers  494         <NA>     <NA>    NA
    ## 212           facing  494         <NA>     <NA>    NA
    ## 213          ranking  492         <NA>     <NA>    NA
    ## 214           entire  491         <NA>     <NA>    NA
    ## 215         attorney  490        anger     <NA>    NA
    ## 216         attorney  490         fear     <NA>    NA
    ## 217         attorney  490     positive     <NA>    NA
    ## 218         attorney  490        trust     <NA>    NA
    ## 219          ballots  490         <NA>     <NA>    NA
    ## 220        countries  490         <NA>     <NA>    NA
    ## 221    entertainment  490 anticipation     <NA>    NA
    ## 222    entertainment  490          joy     <NA>    NA
    ## 223    entertainment  490     positive     <NA>    NA
    ## 224    entertainment  490     surprise     <NA>    NA
    ## 225    entertainment  490        trust     <NA>    NA
    ## 226          quietly  490         <NA>     <NA>    NA
    ## 227       restaurant  490         <NA>     <NA>    NA
    ## 228        mcconnell  489         <NA>     <NA>    NA
    ## 229             live  487         <NA>     <NA>    NA
    ## 230            filed  486         <NA>     <NA>    NA
    ## 231          updated  486         <NA>     <NA>    NA
    ## 232         suggests  485         <NA>     <NA>    NA
    ## 233           victim  483        anger     <NA>    -3
    ## 234           victim  483         fear     <NA>    -3
    ## 235           victim  483     negative     <NA>    -3
    ## 236           victim  483      sadness     <NA>    -3
    ## 237        worldwide  480         <NA>     <NA>    NA
    ## 238          arsenal  479         <NA>     <NA>    NA
    ## 239              mum  479         fear     <NA>    NA
    ## 240              mum  479     negative     <NA>    NA
    ## 241             ring  478         <NA>     <NA>    NA
    ## 242           banned  472         <NA>     <NA>    -2
    ## 243             fuck  472         <NA> negative    -4
    ## 244           judges  471         <NA>     <NA>    NA
    ## 245       opposition  471        anger negative    NA
    ## 246       opposition  471     negative negative    NA
    ## 247          russian  471         <NA>     <NA>    NA
    ## 248              net  470         <NA>     <NA>    NA
    ## 249           always  468         <NA>     <NA>    NA
    ## 250           denied  468     negative negative    -2
    ## 251           denied  468      sadness negative    -2
    ## 252          pirates  467         <NA>     <NA>    NA
    ## 253               ve  467         <NA>     <NA>    NA
    ## 254         solution  465     positive     <NA>     1
    ## 255        liverpool  462         <NA>     <NA>    NA
    ## 256            loses  461         <NA> negative    NA
    ## 257        streaming  461         <NA>     <NA>    NA
    ## 258        everytime  460         <NA>     <NA>    NA
    ## 259            chain  459         <NA>     <NA>    NA
    ## 260         terrible  458        anger negative    -3
    ## 261         terrible  458      disgust negative    -3
    ## 262         terrible  458         fear negative    -3
    ## 263         terrible  458     negative negative    -3
    ## 264         terrible  458      sadness negative    -3
    ## 265         involved  455         <NA>     <NA>    NA
    ## 266       everything  453         <NA>     <NA>    NA
    ## 267           minute  453         <NA>     <NA>    NA
    ## 268        exclusive  452         <NA>     <NA>     2
    ## 269           khabib  451         <NA>     <NA>    NA
    ## 270              lee  451         <NA>     <NA>    NA
    ## 271            finds  450         <NA>     <NA>    NA
    ## 272       commercial  448         <NA>     <NA>    NA
    ## 273          privacy  446         <NA>     <NA>    NA
    ## 274           defeat  444     negative positive    NA
    ## 275          records  444         <NA>     <NA>    NA
    ## 276            clean  443          joy positive     2
    ## 277            clean  443     positive positive     2
    ## 278            clean  443        trust positive     2
    ## 279              fox  443         <NA>     <NA>    NA
    ## 280          chicago  442         <NA>     <NA>    NA
    ## 281        languages  442         <NA>     <NA>    NA
    ## 282          midterm  442         <NA>     <NA>    NA
    ## 283              cry  439     negative negative    -1
    ## 284              cry  439      sadness negative    -1
    ## 285            rally  439         <NA>     <NA>    NA
    ## 286        cancelled  436         <NA>     <NA>    -1
    ## 287          roasted  435         <NA>     <NA>    NA
    ## 288               pm  433         <NA>     <NA>    NA
    ## 289          airport  432 anticipation     <NA>    NA
    ## 290           record  432         <NA>     <NA>    NA
    ## 291          rigging  431         <NA>     <NA>    NA
    ## 292          streets  431         <NA>     <NA>    NA
    ## 293        attempted  430         <NA>     <NA>    NA
    ## 294       protesters  430         <NA>     <NA>    -2
    ## 295           survey  429         <NA>     <NA>    NA
    ## 296          academy  427     positive     <NA>    NA
    ## 297          decades  427         <NA>     <NA>    NA
    ## 298          improve  426 anticipation positive     2
    ## 299          improve  426          joy positive     2
    ## 300          improve  426     positive positive     2
    ## 301          improve  426        trust positive     2
    ## 302         virginia  425         <NA>     <NA>    NA
    ## 303               el  424         <NA>     <NA>    NA
    ## 304            faces  424         <NA>     <NA>    NA
    ## 305   infrastructure  424         <NA>     <NA>    NA
    ## 306       struggling  423         <NA> negative    -2
    ## 307            begin  422         <NA>     <NA>    NA
    ## 308          walking  419         <NA>     <NA>    NA
    ## 309         happened  417         <NA>     <NA>    NA
    ## 310             knew  417         <NA>     <NA>    NA
    ## 311          flights  416         <NA>     <NA>    NA
    ## 312           elects  415         <NA>     <NA>    NA
    ## 313       protesting  411         <NA> negative    -2
    ## 314             teen  410         <NA>     <NA>    NA
    ## 315     vaccinations  410         <NA>     <NA>    NA
    ## 316           factor  409         <NA>     <NA>    NA
    ## 317        boyfriend  408         <NA>     <NA>    NA
    ## 318            queen  408         <NA>     <NA>    NA
    ## 319         complete  407         <NA>     <NA>    NA
    ## 320           option  407     positive     <NA>    NA
    ## 321            plant  407         <NA>     <NA>    NA
    ## 322           awards  406         <NA> positive     3
    ## 323           gender  406         <NA>     <NA>    NA
    ## 324        celebrity  403        anger     <NA>    NA
    ## 325        celebrity  403 anticipation     <NA>    NA
    ## 326        celebrity  403      disgust     <NA>    NA
    ## 327        celebrity  403          joy     <NA>    NA
    ## 328        celebrity  403     negative     <NA>    NA
    ## 329        celebrity  403     positive     <NA>    NA
    ## 330        celebrity  403     surprise     <NA>    NA
    ## 331        celebrity  403        trust     <NA>    NA
    ## 332            wants  402         <NA>     <NA>    NA
    ## 333             trip  400     surprise     <NA>    NA
    ## 334           donald  399         <NA>     <NA>    NA
    ## 335              led  399         <NA> positive    NA
    ## 336         protests  398         <NA> negative    -2
    ## 337              bay  396         <NA>     <NA>    NA
    ## 338         lifetime  396         <NA>     <NA>    NA
    ## 339            child  395 anticipation     <NA>    NA
    ## 340            child  395          joy     <NA>    NA
    ## 341            child  395     positive     <NA>    NA
    ## 342             film  395         <NA>     <NA>    NA
    ## 343           hacked  393         <NA>     <NA>    -1
    ## 344           anyway  392         <NA>     <NA>    NA
    ## 345          example  392         <NA>     <NA>    NA
    ## 346             plot  392         <NA> negative    NA
    ## 347          charges  391         <NA>     <NA>    -2
    ## 348               hd  389         <NA>     <NA>    NA
    ## 349     pennsylvania  389         <NA>     <NA>    NA
    ## 350         arrested  388         <NA>     <NA>    -3
    ## 351          cinemas  386         <NA>     <NA>    NA
    ## 352            films  386         <NA>     <NA>    NA
    ## 353        minnesota  386         <NA>     <NA>    NA
    ## 354         people's  385         <NA>     <NA>    NA
    ## 355           asians  383         <NA>     <NA>    NA
    ## 356       impressive  382         <NA> positive     3
    ## 357              mod  382         <NA>     <NA>    NA
    ## 358             flag  380         <NA>     <NA>    NA
    ## 359          polling  380         <NA>     <NA>    NA
    ## 360              ban  379     negative     <NA>    -2
    ## 361          bitcoin  379         <NA>     <NA>    NA
    ## 362            moore  379         <NA>     <NA>    NA
    ## 363          billion  378         <NA>     <NA>    NA
    ## 364          advance  375 anticipation     <NA>    NA
    ## 365          advance  375         fear     <NA>    NA
    ## 366          advance  375          joy     <NA>    NA
    ## 367          advance  375     positive     <NA>    NA
    ## 368          advance  375     surprise     <NA>    NA
    ## 369           follow  375         <NA>     <NA>    NA
    ## 370          netflix  374         <NA>     <NA>    NA
    ## 371            judge  373         <NA>     <NA>    NA
    ## 372             mods  373         <NA>     <NA>    NA
    ## 373         watching  373         <NA>     <NA>    NA
    ## 374             ever  371         <NA>     <NA>    NA
    ## 375        moviepass  370         <NA>     <NA>    NA
    ## 376         offering  369        trust     <NA>    NA
    ## 377            apart  368         <NA>     <NA>    NA
    ## 378            start  366 anticipation     <NA>    NA
    ## 379            april  365         <NA>     <NA>    NA
    ## 380           askmen  365         <NA>     <NA>    NA
    ## 381             okay  364         <NA>     <NA>    NA
    ## 382        subreddit  364         <NA>     <NA>    NA
    ## 383       adaptation  363         <NA>     <NA>    NA
    ## 384          creator  363         <NA>     <NA>    NA
    ## 385          picture  362         <NA>     <NA>    NA
    ## 386           rotten  361         <NA> negative    NA
    ## 387           batman  359         <NA>     <NA>    NA
    ## 388            great  359         <NA> positive     3
    ## 389            obama  359         <NA>     <NA>    NA
    ## 390            quite  359         <NA>     <NA>    NA
    ## 391         theaters  359         <NA>     <NA>    NA
    ## 392             eyes  358         <NA>     <NA>    NA
    ## 393          century  357         <NA>     <NA>    NA
    ## 394            heard  357         <NA>     <NA>    NA
    ## 395           disney  356         <NA>     <NA>    NA
    ## 396             musk  355         <NA>     <NA>    NA
    ## 397          reviews  355         <NA>     <NA>    NA
    ## 398         engineer  354         <NA>     <NA>    NA
    ## 399          content  353          joy     <NA>    NA
    ## 400          content  353     positive     <NA>    NA
    ## 401          content  353        trust     <NA>    NA
    ## 402         mcgregor  353         <NA>     <NA>    NA
    ## 403  representatives  353         <NA>     <NA>    NA
    ## 404               ip  352         <NA>     <NA>    NA
    ## 405             died  351         <NA> negative    -3
    ## 406          targets  351         <NA>     <NA>    NA
    ## 407            total  351         <NA>     <NA>    NA
    ## 408          history  350         <NA>     <NA>    NA
    ## 409          outrage  350        anger negative    -3
    ## 410          outrage  350      disgust negative    -3
    ## 411          outrage  350     negative negative    -3
    ## 412            crack  349     negative negative    NA
    ## 413             moon  349         <NA>     <NA>    NA
    ## 414        character  348         <NA>     <NA>    NA
    ## 415           robert  348         <NA>     <NA>    NA
    ## 416        fictional  347         <NA> negative    NA
    ## 417         location  346         <NA>     <NA>    NA
    ## 418           albums  345         <NA>     <NA>    NA
    ## 419           series  345        trust     <NA>    NA
    ## 420             view  345         <NA>     <NA>    NA
    ## 421       throughout  343         <NA>     <NA>    NA
    ## 422             porn  342      disgust     <NA>    NA
    ## 423             porn  342     negative     <NA>    NA
    ## 424             beer  341          joy     <NA>    NA
    ## 425             beer  341     positive     <NA>    NA
    ## 426             many  341         <NA>     <NA>    NA
    ## 427             trek  340         <NA>     <NA>    NA
    ## 428          counted  338         <NA>     <NA>    NA
    ## 429             kill  338         fear negative    -3
    ## 430             kill  338     negative negative    -3
    ## 431             kill  338      sadness negative    -3
    ## 432         michigan  337         <NA>     <NA>    NA
    ## 433             note  336         <NA>     <NA>    NA
    ## 434      potentially  336         <NA>     <NA>    NA
    ## 435           report  336         <NA>     <NA>    NA
    ## 436           frozen  335         <NA> negative    NA
    ## 437        interfere  335         <NA> negative    NA
    ## 438            reach  335         <NA>     <NA>     1
    ## 439       registered  335         <NA>     <NA>    NA
    ## 440             head  334         <NA>     <NA>    NA
    ## 441             ohio  334         <NA>     <NA>    NA
    ## 442            sites  334         <NA>     <NA>    NA
    ## 443           stands  334         <NA>     <NA>    NA
    ## 444             main  333     positive     <NA>    NA
    ## 445         shooting  333        anger     <NA>    NA
    ## 446         shooting  333         fear     <NA>    NA
    ## 447         shooting  333     negative     <NA>    NA
    ## 448            times  333         <NA>     <NA>    NA
    ## 449          charity  330          joy     <NA>    NA
    ## 450          charity  330     positive     <NA>    NA
    ## 451           normal  330         <NA>     <NA>    NA
    ## 452        officials  330         <NA>     <NA>    NA
    ## 453           ballot  327 anticipation     <NA>    NA
    ## 454           ballot  327     positive     <NA>    NA
    ## 455           ballot  327        trust     <NA>    NA
    ## 456        elizabeth  327         <NA>     <NA>    NA
    ## 457           racism  327         <NA> negative    -3
    ## 458          returns  326         <NA>     <NA>    NA
    ## 459                r  324         <NA>     <NA>    NA
    ## 460            trump  324     surprise positive    NA
    ## 461           accept  323         <NA>     <NA>     1
    ## 462         laughing  323          joy     <NA>     1
    ## 463         laughing  323     positive     <NA>     1
    ## 464         develops  322         <NA>     <NA>    NA
    ## 465         meddling  322         <NA>     <NA>    NA
    ## 466       industrial  321         <NA>     <NA>    NA
    ## 467          leaving  321         <NA>     <NA>    NA
    ## 468         bathroom  320         <NA>     <NA>    NA
    ## 469   schwarzenegger  320         <NA>     <NA>    NA
    ## 470            green  319          joy     <NA>    NA
    ## 471            green  319     positive     <NA>    NA
    ## 472            green  319        trust     <NA>    NA
    ## 473            rules  318         <NA>     <NA>    NA
    ## 474             back  317         <NA>     <NA>    NA
    ## 475           brazil  317         <NA>     <NA>    NA
    ## 476           crying  317     negative     <NA>    -2
    ## 477           crying  317      sadness     <NA>    -2
    ## 478         launches  317         <NA>     <NA>    NA
    ## 479          staying  317         <NA>     <NA>    NA
    ## 480           target  315         <NA>     <NA>    NA
    ## 481             wing  315         <NA>     <NA>    NA
    ## 482             band  314         <NA>     <NA>    NA
    ## 483              usa  314         <NA>     <NA>    NA
    ## 484              box  312         <NA>     <NA>    NA
    ## 485           bottom  311     negative     <NA>    NA
    ## 486           bottom  311      sadness     <NA>    NA
    ## 487            sends  311         <NA>     <NA>    NA
    ## 488          special  311          joy     <NA>    NA
    ## 489          special  311     positive     <NA>    NA
    ## 490           thinks  311         <NA>     <NA>    NA
    ## 491            users  311         <NA>     <NA>    NA
    ## 492          credits  310         <NA>     <NA>    NA
    ## 493          hacking  310         <NA>     <NA>    NA
    ## 494          sources  310         <NA>     <NA>    NA
    ## 495          surgery  310         fear     <NA>    NA
    ## 496          surgery  310      sadness     <NA>    NA
    ## 497            costs  308         <NA>     <NA>    NA
    ## 498              ill  307        anger     <NA>    -2
    ## 499              ill  307      disgust     <NA>    -2
    ## 500              ill  307         fear     <NA>    -2
    ## 501              ill  307     negative     <NA>    -2
    ## 502              ill  307      sadness     <NA>    -2
    ## 503             anti  306         <NA>     <NA>    -1
    ## 504          theater  306         <NA>     <NA>    NA
    ## 505              ufc  306         <NA>     <NA>    NA
    ## 506         analysis  305         <NA>     <NA>    NA
    ## 507     technologies  305         <NA>     <NA>    NA
    ## 508        yesterday  303         <NA>     <NA>    NA
    ## 509            apply  302         <NA>     <NA>    NA
    ## 510          explain  302     positive     <NA>    NA
    ## 511          explain  302        trust     <NA>    NA
    ## 512             talk  302     positive     <NA>    NA
    ## 513              guy  301         <NA>     <NA>    NA
    ## 514              bob  299         <NA>     <NA>    NA
    ## 515           caucus  298         <NA>     <NA>    NA
    ## 516       signatures  298         <NA>     <NA>    NA
    ## 517             walk  298         <NA>     <NA>    NA
    ## 518             elon  297         <NA>     <NA>    NA
    ## 519           george  297         <NA>     <NA>    NA
    ## 520           prison  297        anger negative    -2
    ## 521           prison  297         fear negative    -2
    ## 522           prison  297     negative negative    -2
    ## 523           prison  297      sadness negative    -2
    ## 524           stolen  297        anger negative    -2
    ## 525           stolen  297     negative negative    -2
    ## 526        newspaper  296         <NA>     <NA>    NA
    ## 527            anime  295         <NA>     <NA>    NA
    ## 528           bullet  295         <NA>     <NA>    NA
    ## 529            thing  295         <NA>     <NA>    NA
    ## 530          believe  294         <NA>     <NA>    NA
    ## 531              ceo  294         <NA>     <NA>    NA
    ## 532             roll  294         <NA>     <NA>    NA
    ## 533        announced  293         <NA>     <NA>    NA
    ## 534              bed  293         <NA>     <NA>    NA
    ## 535         revealed  293         <NA>     <NA>    NA
    ## 536       censorship  292         <NA>     <NA>    NA
    ## 537            movie  292         <NA>     <NA>    NA
    ## 538            troll  292        anger     <NA>    NA
    ## 539            troll  292         fear     <NA>    NA
    ## 540            troll  292     negative     <NA>    NA
    ## 541            watch  292 anticipation     <NA>    NA
    ## 542            watch  292         fear     <NA>    NA
    ## 543             half  291         <NA>     <NA>    NA
    ## 544           horror  291        anger     <NA>    NA
    ## 545           horror  291      disgust     <NA>    NA
    ## 546           horror  291         fear     <NA>    NA
    ## 547           horror  291     negative     <NA>    NA
    ## 548           horror  291      sadness     <NA>    NA
    ## 549           horror  291     surprise     <NA>    NA
    ## 550            patch  291     negative     <NA>    NA
    ## 551          arguing  290         <NA>     <NA>    NA
    ## 552     restrictions  290         <NA>     <NA>    NA
    ## 553        including  288     positive     <NA>    NA
    ## 554             pass  288         <NA>     <NA>    NA
    ## 555           reeves  288         <NA>     <NA>    NA
    ## 556     interference  287     negative negative    NA
    ## 557           studio  287         <NA>     <NA>    NA
    ## 558        transform  287         <NA>     <NA>    NA
    ## 559             euro  286         <NA>     <NA>    NA
    ## 560             felt  286         <NA>     <NA>    NA
    ## 561          nations  286         <NA>     <NA>    NA
    ## 562              sue  286        anger negative    NA
    ## 563              sue  286     negative negative    NA
    ## 564              sue  286      sadness negative    NA
    ## 565          declare  285         <NA>     <NA>    NA
    ## 566         facility  285         <NA>     <NA>    NA
    ## 567           shares  285         <NA>     <NA>     1
    ## 568           sister  285         <NA>     <NA>    NA
    ## 569        teenagers  285         <NA>     <NA>    NA
    ## 570                e  284         <NA>     <NA>    NA
    ## 571           failed  283         <NA> negative    -2
    ## 572            words  283        anger     <NA>    NA
    ## 573            words  283     negative     <NA>    NA
    ## 574              ama  282         <NA>     <NA>    NA
    ## 575        deepfakes  282         <NA>     <NA>    NA
    ## 576               ll  282         <NA>     <NA>    NA
    ## 577            claim  281         <NA>     <NA>    NA
    ## 578            keanu  280         <NA>     <NA>    NA
    ## 579           passed  280         <NA>     <NA>    NA
    ## 580      approaching  279 anticipation     <NA>    NA
    ## 581            jokes  279         <NA>     <NA>     2
    ## 582           memory  279         <NA>     <NA>    NA
    ## 583           scores  279         <NA>     <NA>    NA
    ## 584           warner  279         <NA>     <NA>    NA
    ## 585            stone  278        anger     <NA>    NA
    ## 586            stone  278     negative     <NA>    NA
    ## 587          economy  277        trust     <NA>    NA
    ## 588             firm  277         <NA>     <NA>    NA
    ## 589         possibly  277         <NA>     <NA>    NA
    ## 590               vs  277         <NA>     <NA>    NA
    ## 591          funding  276         <NA>     <NA>    NA
    ## 592          library  276     positive     <NA>    NA
    ## 593             mike  276         <NA>     <NA>    NA
    ## 594         theatres  276         <NA>     <NA>    NA
    ## 595            topic  275         <NA>     <NA>    NA
    ## 596              ago  274         <NA>     <NA>    NA
    ## 597           photos  273         <NA>     <NA>    NA
    ## 598           seeing  273         <NA>     <NA>    NA
    ## 599             page  272         <NA>     <NA>    NA
    ## 600             told  272         <NA>     <NA>    NA
    ## 601          unveils  272         <NA>     <NA>    NA
    ## 602           nearly  271         <NA>     <NA>    NA
    ## 603       resolution  271         <NA>     <NA>    NA
    ## 604        subtitles  271         <NA>     <NA>    NA
    ## 605           change  270         fear     <NA>    NA
    ## 606              feb  270         <NA>     <NA>    NA
    ## 607          pirated  270         <NA>     <NA>    NA
    ## 608            panel  269         <NA>     <NA>    NA
    ## 609          showing  269         <NA>     <NA>    NA
    ## 610              ted  269         <NA>     <NA>    NA
    ## 611            hates  268         <NA> negative    -3
    ## 612        hollywood  268         <NA>     <NA>    NA
    ## 613           letter  268 anticipation     <NA>    NA
    ## 614          offered  268         <NA>     <NA>    NA
    ## 615            shown  268         <NA>     <NA>    NA
    ## 616          turnout  268         <NA>     <NA>    NA
    ## 617         admitted  267         <NA>     <NA>    -1
    ## 618      billionaire  267         <NA>     <NA>    NA
    ## 619             dead  267         <NA> negative    -3
    ## 620            hands  267         <NA>     <NA>    NA
    ## 621          tactics  266         fear     <NA>    NA
    ## 622          tactics  266        trust     <NA>    NA
    ## 623        democrats  265         <NA>     <NA>    NA
    ## 624            swear  265     positive     <NA>    -2
    ## 625            swear  265        trust     <NA>    -2
    ## 626             mark  264         <NA>     <NA>    NA
    ## 627            peter  264         <NA>     <NA>    NA
    ## 628              alt  263         <NA>     <NA>    NA
    ## 629           energy  263         <NA>     <NA>    NA
    ## 630        americans  262         <NA>     <NA>    NA
    ## 631          grandma  262         <NA>     <NA>    NA
    ## 632           israel  262         <NA>     <NA>    NA
    ## 633            offer  262     positive     <NA>    NA
    ## 634             show  262        trust     <NA>    NA
    ## 635              til  262         <NA>     <NA>    NA
    ## 636            calls  261 anticipation     <NA>    NA
    ## 637            calls  261     negative     <NA>    NA
    ## 638            calls  261        trust     <NA>    NA
    ## 639            makes  261         <NA>     <NA>    NA
    ## 640              u.s  261         <NA>     <NA>    NA
    ## 641       vaccinated  261         <NA>     <NA>    NA
    ## 642             made  260         <NA>     <NA>    NA
    ## 643              sit  260         <NA>     <NA>    NA
    ## 644           border  259         <NA>     <NA>    NA
    ## 645         describe  259         <NA>     <NA>    NA
    ## 646           across  258         <NA>     <NA>    NA
    ## 647          banning  258         <NA>     <NA>    NA
    ## 648            spent  258     negative     <NA>    NA
    ## 649         citizens  257         <NA>     <NA>    NA
    ## 650    congressional  257         <NA>     <NA>    NA
    ## 651          context  257         <NA>     <NA>    NA
    ## 652            joker  257          joy negative    NA
    ## 653            joker  257     positive negative    NA
    ## 654            joker  257     surprise negative    NA
    ## 655         cultural  256         <NA>     <NA>    NA
    ## 656            angry  255        anger negative    -3
    ## 657            angry  255      disgust negative    -3
    ## 658            angry  255     negative negative    -3
    ## 659             bank  255        trust     <NA>    NA
    ## 660          address  254         <NA>     <NA>    NA
    ## 661             body  254         <NA>     <NA>    NA
    ## 662         together  254         <NA>     <NA>    NA
    ## 663      vaccination  254         <NA>     <NA>    NA
    ## 664         becoming  253         <NA>     <NA>    NA
    ## 665          parents  252         <NA>     <NA>    NA
    ## 666       scientific  252     positive     <NA>    NA
    ## 667       scientific  252        trust     <NA>    NA
    ## 668          calling  251         <NA>     <NA>    NA
    ## 669         deadline  251         <NA>     <NA>    NA
    ## 670            drama  251         <NA>     <NA>    NA
    ## 671             fake  251     negative negative    -3
    ## 672        askreddit  250         <NA>     <NA>    NA
    ## 673             hair  250         <NA>     <NA>    NA
    ## 674        illnesses  250         <NA>     <NA>    -2
    ## 675             says  250         <NA>     <NA>    NA
    ## 676             vote  250        anger     <NA>    NA
    ## 677             vote  250 anticipation     <NA>    NA
    ## 678             vote  250          joy     <NA>    NA
    ## 679             vote  250     negative     <NA>    NA
    ## 680             vote  250     positive     <NA>    NA
    ## 681             vote  250      sadness     <NA>    NA
    ## 682             vote  250     surprise     <NA>    NA
    ## 683             vote  250        trust     <NA>    NA
    ## 684             feet  249         <NA>     <NA>    NA
    ## 685         minister  249         <NA>     <NA>    NA
    ## 686              sex  249 anticipation     <NA>    NA
    ## 687              sex  249          joy     <NA>    NA
    ## 688              sex  249     positive     <NA>    NA
    ## 689              sex  249        trust     <NA>    NA
    ## 690            today  249         <NA>     <NA>    NA
    ## 691          charlie  248         <NA>     <NA>    NA
    ## 692        filmmaker  248         <NA>     <NA>    NA
    ## 693         releases  248         <NA>     <NA>    NA
    ## 694             wasn  248         <NA>     <NA>    NA
    ## 695         democrat  247         <NA>     <NA>    NA
    ## 696             lack  247     negative negative    -2
    ## 697              one  247         <NA>     <NA>    NA
    ## 698            weird  247      disgust negative    -2
    ## 699            weird  247     negative negative    -2
    ## 700        president  246     positive     <NA>    NA
    ## 701        president  246        trust     <NA>    NA
    ## 702             rich  246         <NA> positive     2
    ## 703            scene  246         <NA>     <NA>    NA
    ## 704           friend  245          joy     <NA>    NA
    ## 705           friend  245     positive     <NA>    NA
    ## 706           friend  245        trust     <NA>    NA
    ## 707           driven  244         <NA>     <NA>    NA
    ## 708        generated  244         <NA>     <NA>    NA
    ## 709          outside  244         <NA>     <NA>    NA
    ## 710       production  244 anticipation     <NA>    NA
    ## 711       production  244     positive     <NA>    NA
    ## 712           voters  244         <NA>     <NA>    NA
    ## 713              add  243         <NA>     <NA>    NA
    ## 714           family  243         <NA>     <NA>    NA
    ## 715       presidents  243         <NA>     <NA>    NA
    ## 716        buildings  242         <NA>     <NA>    NA
    ## 717            civil  242     positive     <NA>    NA
    ## 718                h  242         <NA>     <NA>    NA
    ## 719          waiting  242         <NA>     <NA>    NA
    ## 720         december  241         <NA>     <NA>    NA
    ## 721            doesn  241         <NA>     <NA>    NA
    ## 722           recent  241         <NA>     <NA>    NA
    ## 723           sunday  241         <NA>     <NA>    NA
    ## 724          support  241         <NA> positive     2
    ## 725         attacked  240         <NA>     <NA>    -1
    ## 726       government  240         fear     <NA>    NA
    ## 727       government  240     negative     <NA>    NA
    ## 728         accurate  239     positive positive    NA
    ## 729         accurate  239        trust positive    NA
    ## 730        christian  239         <NA>     <NA>    NA
    ## 731            image  239         <NA>     <NA>    NA
    ## 732             mail  239 anticipation     <NA>    NA
    ## 733       reportedly  239         <NA>     <NA>    NA
    ## 734    significantly  239         <NA>     <NA>    NA
    ## 735          suicide  239        anger negative    -2
    ## 736          suicide  239         fear negative    -2
    ## 737          suicide  239     negative negative    -2
    ## 738          suicide  239      sadness negative    -2
    ## 739       democratic  238         <NA>     <NA>    NA
    ## 740         identify  238         <NA>     <NA>    NA
    ## 741       manchester  238         <NA>     <NA>    NA
    ## 742           number  238         <NA>     <NA>    NA
    ## 743       techniques  238         <NA>     <NA>    NA
    ## 744            woman  238         <NA>     <NA>    NA
    ## 745        barcelona  237         <NA>     <NA>    NA
    ## 746          beating  237        anger     <NA>    -1
    ## 747          beating  237         fear     <NA>    -1
    ## 748          beating  237     negative     <NA>    -1
    ## 749          beating  237      sadness     <NA>    -1
    ## 750          chelsea  237         <NA>     <NA>    NA
    ## 751         increase  237     positive     <NA>     1
    ## 752           papers  237         <NA>     <NA>    NA
    ## 753             alex  236         <NA>     <NA>    NA
    ## 754             base  236        trust     <NA>    NA
    ## 755        hilarious  236          joy positive     2
    ## 756        hilarious  236     positive positive     2
    ## 757        hilarious  236     surprise positive     2
    ## 758              npc  236         <NA>     <NA>    NA
    ## 759              cut  235         <NA>     <NA>    -1
    ## 760              isn  235         <NA>     <NA>    NA
    ## 761          justice  235     positive     <NA>     2
    ## 762          justice  235        trust     <NA>     2
    ## 763           openly  234         <NA> positive    NA
    ## 764           reveal  234         <NA>     <NA>    NA
    ## 765              row  234        anger     <NA>    NA
    ## 766              row  234     negative     <NA>    NA
    ## 767          million  233         <NA>     <NA>    NA
    ## 768            fully  232     positive     <NA>    NA
    ## 769            fully  232        trust     <NA>    NA
    ## 770             hear  232         <NA>     <NA>    NA
    ## 771             hits  232         <NA>     <NA>    NA
    ## 772           iconic  232         <NA>     <NA>    NA
    ## 773        candidate  231     positive     <NA>    NA
    ## 774             kong  231         <NA>     <NA>    NA
    ## 775            thank  231         <NA> positive     2
    ## 776        correctly  230         <NA> positive    NA
    ## 777          decides  230         <NA>     <NA>    NA
    ## 778          monthly  230         <NA>     <NA>    NA
    ## 779            prime  230     positive     <NA>    NA
    ## 780          written  230         <NA>     <NA>    NA
    ## 781             amid  229         <NA>     <NA>    NA
    ## 782             burn  229         <NA> negative    NA
    ## 783          elected  229         <NA>     <NA>    NA
    ## 784          factory  229         <NA>     <NA>    NA
    ## 785            heads  229         <NA>     <NA>    NA
    ## 786           skynet  229         <NA>     <NA>    NA
    ## 787             aren  228         <NA>     <NA>    NA
    ## 788        discusses  228         <NA>     <NA>    NA
    ## 789              lot  228         <NA>     <NA>    NA
    ## 790           messed  228         <NA> negative    -2
    ## 791         promises  228         <NA> positive     1
    ## 792        superhero  228         <NA>     <NA>    NA
    ## 793               ya  228         <NA>     <NA>    NA
    ## 794           camera  227         <NA>     <NA>    NA
    ## 795         declares  227         <NA>     <NA>    NA
    ## 796           racist  227         <NA> negative    -3
    ## 797            still  227         <NA>     <NA>    NA
    ## 798              bad  226        anger negative    -3
    ## 799              bad  226      disgust negative    -3
    ## 800              bad  226         fear negative    -3
    ## 801              bad  226     negative negative    -3
    ## 802              bad  226      sadness negative    -3
    ## 803       characters  226         <NA>     <NA>    NA
    ## 804              old  226         <NA>     <NA>    NA
    ## 805            right  226         <NA> positive    NA
    ## 806            steve  226         <NA>     <NA>    NA
    ## 807        supported  226     positive positive     2
    ## 808            wrote  226         <NA>     <NA>    NA
    ## 809            later  225         <NA>     <NA>    NA
    ## 810          systems  225         <NA>     <NA>    NA
    ## 811     accidentally  224     surprise     <NA>    -2
    ## 812             hong  224         <NA>     <NA>    NA
    ## 813           placed  224         <NA>     <NA>    NA
    ## 814            press  224         <NA>     <NA>    NA
    ## 815          trailer  224         <NA>     <NA>    NA
    ## 816               tv  224         <NA>     <NA>    NA
    ## 817         american  223         <NA>     <NA>    NA
    ## 818        continues  223         <NA>     <NA>    NA
    ## 819       corruption  223      disgust negative    NA
    ## 820       corruption  223     negative negative    NA
    ## 821          despite  223         <NA>     <NA>    NA
    ## 822           france  223         <NA>     <NA>    NA
    ## 823            plane  223         <NA>     <NA>    NA
    ## 824             sell  223         <NA>     <NA>    NA
    ## 825           comedy  222         <NA>     <NA>     1
    ## 826            least  222         <NA>     <NA>    NA
    ## 827            posts  222         <NA>     <NA>    NA
    ## 828          threats  222         <NA> negative    -2
    ## 829          vehicle  222         <NA>     <NA>    NA
    ## 830               fc  221         <NA>     <NA>    NA
    ## 831            spike  221         fear     <NA>    NA
    ## 832           street  221         <NA>     <NA>    NA
    ## 833            danny  220         <NA>     <NA>    NA
    ## 834             held  220         <NA>     <NA>    NA
    ## 835         patterns  220         <NA>     <NA>    NA
    ## 836          accused  219        anger     <NA>    -2
    ## 837          accused  219         fear     <NA>    -2
    ## 838          accused  219     negative     <NA>    -2
    ## 839          british  219         <NA>     <NA>    NA
    ## 840          burning  219         <NA> negative    NA
    ## 841       california  219         <NA>     <NA>    NA
    ## 842              cdc  219         <NA>     <NA>    NA
    ## 843        favourite  219         <NA>     <NA>    NA
    ## 844      information  219     positive     <NA>    NA
    ## 845              put  219         <NA>     <NA>    NA
    ## 846                s  219         <NA>     <NA>    NA
    ## 847             cats  218         <NA>     <NA>    NA
    ## 848             cell  218         <NA>     <NA>    NA
    ## 849           doctor  218     positive     <NA>    NA
    ## 850           doctor  218        trust     <NA>    NA
    ## 851         industry  218         <NA>     <NA>    NA
    ## 852           really  218         <NA>     <NA>    NA
    ## 853            takes  218         <NA>     <NA>    NA
    ## 854             wins  218         <NA> positive     4
    ## 855              wow  218         <NA> positive     4
    ## 856          casting  217         <NA>     <NA>    NA
    ## 857            tesla  217         <NA>     <NA>    NA
    ## 858              top  217 anticipation positive     2
    ## 859              top  217     positive positive     2
    ## 860              top  217        trust positive     2
    ## 861             nice  216         <NA> positive     3
    ## 862            since  216         <NA>     <NA>    NA
    ## 863             yeah  216         <NA>     <NA>     1
    ## 864             area  215         <NA>     <NA>    NA
    ## 865          hackers  215         <NA>     <NA>    NA
    ## 866           vaxxer  215         <NA>     <NA>    NA
    ## 867             bros  214         <NA>     <NA>    NA
    ## 868           daniel  214         <NA>     <NA>    NA
    ## 869          federal  214         <NA>     <NA>    NA
    ## 870        musicians  214         <NA>     <NA>    NA
    ## 871              pet  214     negative     <NA>    NA
    ## 872          teacher  214     positive     <NA>    NA
    ## 873          teacher  214        trust     <NA>    NA
    ## 874      antibiotics  213     positive     <NA>    NA
    ## 875             life  213         <NA>     <NA>    NA
    ## 876            march  213     positive     <NA>    NA
    ## 877        perfectly  213         <NA> positive     3
    ## 878        allegedly  212         <NA>     <NA>    NA
    ## 879            badly  212     negative negative    -3
    ## 880            badly  212      sadness negative    -3
    ## 881       experience  212         <NA>     <NA>    NA
    ## 882            flash  212         <NA>     <NA>    NA
    ## 883            guide  212     positive     <NA>    NA
    ## 884            guide  212        trust     <NA>    NA
    ## 885           poster  212         <NA>     <NA>    NA
    ## 886             race  212         <NA>     <NA>    NA
    ## 887             woke  212         <NA>     <NA>    NA
    ## 888          comment  211         <NA>     <NA>    NA
    ## 889            issue  211         <NA> negative    NA
    ## 890          results  211         <NA>     <NA>    NA
    ## 891              yes  211         <NA>     <NA>     1
    ## 892          covered  210         <NA>     <NA>    NA
    ## 893           escape  210 anticipation     <NA>    -1
    ## 894           escape  210         fear     <NA>    -1
    ## 895           escape  210     negative     <NA>    -1
    ## 896           escape  210     positive     <NA>    -1
    ## 897            older  210      sadness     <NA>    NA
    ## 898            treat  210        anger     <NA>    NA
    ## 899            treat  210 anticipation     <NA>    NA
    ## 900            treat  210      disgust     <NA>    NA
    ## 901            treat  210         fear     <NA>    NA
    ## 902            treat  210          joy     <NA>    NA
    ## 903            treat  210     negative     <NA>    NA
    ## 904            treat  210     positive     <NA>    NA
    ## 905            treat  210      sadness     <NA>    NA
    ## 906            treat  210     surprise     <NA>    NA
    ## 907            treat  210        trust     <NA>    NA
    ## 908             best  209         <NA> positive     3
    ## 909      enforcement  209     negative     <NA>    NA
    ## 910             gets  209         <NA>     <NA>    NA
    ## 911       officially  209         <NA>     <NA>    NA
    ## 912        translate  209         <NA>     <NA>    NA
    ## 913           agency  208         <NA>     <NA>    NA
    ## 914          brought  208         <NA>     <NA>    NA
    ## 915             cage  208     negative     <NA>    NA
    ## 916             cage  208      sadness     <NA>    NA
    ## 917         decision  208         <NA>     <NA>    NA
    ## 918          private  208         <NA>     <NA>    NA
    ## 919     surveillance  208         fear     <NA>    NA
    ## 920          climate  207         <NA>     <NA>    NA
    ## 921          cutting  207        anger     <NA>    -1
    ## 922          cutting  207      disgust     <NA>    -1
    ## 923          cutting  207         fear     <NA>    -1
    ## 924          cutting  207     negative     <NA>    -1
    ## 925          cutting  207      sadness     <NA>    -1
    ## 926            elect  207     positive     <NA>    NA
    ## 927            elect  207        trust     <NA>    NA
    ## 928            haven  207     positive     <NA>    NA
    ## 929            haven  207        trust     <NA>    NA
    ## 930           leaves  207         <NA>     <NA>    NA
    ## 931           taiwan  207         <NA>     <NA>    NA
    ## 932           update  207         <NA>     <NA>    NA
    ## 933          becomes  206         <NA>     <NA>    NA
    ## 934            allow  205         <NA>     <NA>     1
    ## 935          foreign  205     negative     <NA>    NA
    ## 936           office  205         <NA>     <NA>    NA
    ## 937             bill  204         <NA>     <NA>    NA
    ## 938          captain  204     positive     <NA>    NA
    ## 939            girls  204         <NA>     <NA>    NA
    ## 940          housing  204         <NA>     <NA>    NA
    ## 941          posting  204         <NA>     <NA>    NA
    ## 942        reporting  204         <NA>     <NA>    NA
    ## 943             star  204 anticipation     <NA>    NA
    ## 944             star  204          joy     <NA>    NA
    ## 945             star  204     positive     <NA>    NA
    ## 946             star  204        trust     <NA>    NA
    ## 947       subreddits  204         <NA>     <NA>    NA
    ## 948          vaxxers  204         <NA>     <NA>    NA
    ## 949             year  204         <NA>     <NA>    NA
    ## 950        defending  203     positive     <NA>    NA
    ## 951           gotten  203         <NA>     <NA>    NA
    ## 952            jones  203         <NA>     <NA>    NA
    ## 953                l  203         <NA>     <NA>    NA
    ## 954            blame  202        anger negative    -2
    ## 955            blame  202      disgust negative    -2
    ## 956            blame  202     negative negative    -2
    ## 957             coal  202         <NA>     <NA>    NA
    ## 958          cosplay  202         <NA>     <NA>    NA
    ## 959         governor  202        trust     <NA>    NA
    ## 960             jedi  202         <NA>     <NA>    NA
    ## 961           korean  202         <NA>     <NA>    NA
    ## 962       revolution  202        anger     <NA>    NA
    ## 963       revolution  202 anticipation     <NA>    NA
    ## 964       revolution  202         fear     <NA>    NA
    ## 965       revolution  202     negative     <NA>    NA
    ## 966       revolution  202     positive     <NA>    NA
    ## 967       revolution  202      sadness     <NA>    NA
    ## 968       revolution  202     surprise     <NA>    NA
    ## 969         superman  202          joy     <NA>    NA
    ## 970         superman  202     positive     <NA>    NA
    ## 971         superman  202        trust     <NA>    NA
    ## 972            union  202         <NA>     <NA>    NA
    ## 973           actors  201         <NA>     <NA>    NA
    ## 974            asked  201         <NA>     <NA>    NA
    ## 975           leaked  201         <NA>     <NA>    -1
    ## 976             real  201     positive     <NA>    NA
    ## 977             real  201        trust     <NA>    NA
    ## 978             seat  201         <NA>     <NA>    NA
    ## 979           status  201     positive     <NA>    NA
    ## 980            women  201         <NA>     <NA>    NA
    ## 981           year's  201         <NA>     <NA>    NA
    ## 982        community  200     positive     <NA>    NA
    ## 983             date  200         <NA>     <NA>    NA
    ## 984          falling  200     negative negative    -1
    ## 985          falling  200      sadness negative    -1
    ## 986          seattle  200         <NA>     <NA>    NA
    ## 987           united  200     positive     <NA>     1
    ## 988           united  200        trust     <NA>     1
    ## 989               us  200         <NA>     <NA>    NA
    ## 990        available  199         <NA> positive    NA
    ## 991           better  199         <NA> positive     2
    ## 992               co  199         <NA>     <NA>    NA
    ## 993         criminal  199        anger negative    -3
    ## 994         criminal  199      disgust negative    -3
    ## 995         criminal  199         fear negative    -3
    ## 996         criminal  199     negative negative    -3
    ## 997         software  199         <NA>     <NA>    NA
    ## 998           sweden  199         <NA>     <NA>    NA
    ## 999            bring  198         <NA>     <NA>    NA
    ## 1000          secure  198         <NA> positive     2
    ## 1001         belgium  197         <NA>     <NA>    NA
    ## 1002           cases  197         <NA>     <NA>    NA
    ## 1003       decisions  197         <NA>     <NA>    NA
    ## 1004            draw  197         <NA>     <NA>    NA
    ## 1005         reached  197         <NA>     <NA>     1
    ## 1006           beast  196        anger     <NA>    NA
    ## 1007           beast  196         fear     <NA>    NA
    ## 1008           beast  196     negative     <NA>    NA
    ## 1009         feeling  196        anger     <NA>     1
    ## 1010         feeling  196 anticipation     <NA>     1
    ## 1011         feeling  196      disgust     <NA>     1
    ## 1012         feeling  196         fear     <NA>     1
    ## 1013         feeling  196          joy     <NA>     1
    ## 1014         feeling  196     negative     <NA>     1
    ## 1015         feeling  196     positive     <NA>     1
    ## 1016         feeling  196      sadness     <NA>     1
    ## 1017         feeling  196     surprise     <NA>     1
    ## 1018         feeling  196        trust     <NA>     1
    ## 1019           feels  196         <NA>     <NA>    NA
    ## 1020           front  196         <NA>     <NA>    NA
    ## 1021        infected  196         <NA> negative    -2
    ## 1022          movies  196         <NA>     <NA>    NA
    ## 1023             bat  195         <NA>     <NA>    NA
    ## 1024             cnn  195         <NA>     <NA>    NA
    ## 1025          reddit  195         <NA>     <NA>    NA
    ## 1026       animation  194         <NA>     <NA>    NA
    ## 1027            free  194         <NA> positive     1
    ## 1028        historic  194         <NA>     <NA>    NA
    ## 1029            iowa  194         <NA>     <NA>    NA
    ## 1030            cost  193         <NA>     <NA>    NA
    ## 1031          legend  193         <NA>     <NA>    NA
    ## 1032           mayor  193     positive     <NA>    NA
    ## 1033        military  193         fear     <NA>    NA
    ## 1034             say  193         <NA>     <NA>    NA
    ## 1035            slow  193         <NA> negative    NA
    ## 1036           years  193         <NA>     <NA>    NA
    ## 1037       analytica  192         <NA>     <NA>    NA
    ## 1038            asks  192         <NA>     <NA>    NA
    ## 1039           bruce  192         <NA>     <NA>    NA
    ## 1040           grade  192         <NA>     <NA>    NA
    ## 1041        greatest  192         <NA> positive     3
    ## 1042            just  192         <NA>     <NA>    NA
    ## 1043            line  192         <NA>     <NA>    NA
    ## 1044       operation  192         fear     <NA>    NA
    ## 1045       operation  192        trust     <NA>    NA
    ## 1046          people  192         <NA>     <NA>    NA
    ## 1047           trade  192        trust     <NA>    NA
    ## 1048           twice  192         <NA>     <NA>    NA
    ## 1049          acting  191         <NA>     <NA>    NA
    ## 1050             gas  191         <NA>     <NA>    NA
    ## 1051          ground  191        trust     <NA>    NA
    ## 1052          issues  191         <NA> negative    NA
    ## 1053          method  191         <NA>     <NA>    NA
    ## 1054        robotics  191         <NA>     <NA>    NA
    ## 1055         science  191         <NA>     <NA>    NA
    ## 1056        thunberg  191         <NA>     <NA>    NA
    ## 1057         chicken  190         fear     <NA>    NA
    ## 1058          highly  190         <NA>     <NA>    NA
    ## 1059         planned  190         <NA>     <NA>    NA
    ## 1060       preparing  190         <NA>     <NA>    NA
    ## 1061          trolls  190         <NA>     <NA>    NA
    ## 1062      candidates  189         <NA>     <NA>    NA
    ## 1063       connected  189         <NA>     <NA>    NA
    ## 1064           daily  189 anticipation     <NA>    NA
    ## 1065      parliament  189        trust     <NA>    NA
    ## 1066          prices  189         <NA>     <NA>    NA
    ## 1067          soviet  189         <NA>     <NA>    NA
    ## 1068          almost  188         <NA>     <NA>    NA
    ## 1069           black  188     negative     <NA>    NA
    ## 1070           black  188      sadness     <NA>    NA
    ## 1071          chance  188     surprise     <NA>     2
    ## 1072     complaining  188         <NA> negative    NA
    ## 1073       diagnosed  188         <NA>     <NA>    NA
    ## 1074         hillary  188         <NA>     <NA>    NA
    ## 1075         illegal  188        anger negative    -3
    ## 1076         illegal  188      disgust negative    -3
    ## 1077         illegal  188         fear negative    -3
    ## 1078         illegal  188     negative negative    -3
    ## 1079         illegal  188      sadness negative    -3
    ## 1080         popular  188         <NA> positive     3
    ## 1081         release  188         <NA>     <NA>    NA
    ## 1082             ads  187         <NA>     <NA>    NA
    ## 1083           karma  187         <NA>     <NA>    NA
    ## 1084       predicted  187         <NA>     <NA>    NA
    ## 1085          rating  187        anger     <NA>    NA
    ## 1086          rating  187         fear     <NA>    NA
    ## 1087          rating  187     negative     <NA>    NA
    ## 1088          rating  187      sadness     <NA>    NA
    ## 1089          rights  187         <NA>     <NA>    NA
    ## 1090           state  187         <NA>     <NA>    NA
    ## 1091           tired  187     negative negative    -2
    ## 1092             yet  187         <NA>     <NA>    NA
    ## 1093           abuse  186        anger negative    -3
    ## 1094           abuse  186      disgust negative    -3
    ## 1095           abuse  186         fear negative    -3
    ## 1096           abuse  186     negative negative    -3
    ## 1097           abuse  186      sadness negative    -3
    ## 1098        allowing  186         <NA>     <NA>    NA
    ## 1099         dealing  186         <NA>     <NA>    NA
    ## 1100            leak  186     negative negative    -1
    ## 1101         moments  186         <NA>     <NA>    NA
    ## 1102             app  185         <NA>     <NA>    NA
    ## 1103            bomb  185        anger negative    -1
    ## 1104            bomb  185         fear negative    -1
    ## 1105            bomb  185     negative negative    -1
    ## 1106            bomb  185      sadness negative    -1
    ## 1107            bomb  185     surprise negative    -1
    ## 1108         country  185         <NA>     <NA>    NA
    ## 1109          detail  185         <NA>     <NA>    NA
    ## 1110             eye  185         <NA>     <NA>    NA
    ## 1111          honest  185        anger positive     2
    ## 1112          honest  185      disgust positive     2
    ## 1113          honest  185         fear positive     2
    ## 1114          honest  185          joy positive     2
    ## 1115          honest  185     positive positive     2
    ## 1116          honest  185      sadness positive     2
    ## 1117          honest  185        trust positive     2
    ## 1118          flight  184         <NA>     <NA>    NA
    ## 1119          fossil  184         <NA>     <NA>    NA
    ## 1120           grand  184         <NA> positive     3
    ## 1121           greta  184         <NA>     <NA>    NA
    ## 1122            king  184     positive     <NA>    NA
    ## 1123            nerf  184         <NA>     <NA>    NA
    ## 1124         profile  184         <NA>     <NA>    NA
    ## 1125       supporter  184          joy positive     1
    ## 1126       supporter  184     positive positive     1
    ## 1127       supporter  184        trust positive     1
    ## 1128       companies  183         <NA>     <NA>    NA
    ## 1129         storage  183         <NA>     <NA>    NA
    ## 1130         treated  183         <NA>     <NA>    NA
    ## 1131           warns  183         <NA>     <NA>    -2
    ## 1132          boston  182         <NA>     <NA>    NA
    ## 1133        claiming  182         <NA>     <NA>    NA
    ## 1134             end  182         <NA>     <NA>    NA
    ## 1135        messages  182         <NA>     <NA>    NA
    ## 1136         putting  182         <NA>     <NA>    NA
    ## 1137            tech  182         <NA>     <NA>    NA
    ## 1138            wave  182         <NA>     <NA>    NA
    ## 1139           basis  181         <NA>     <NA>    NA
    ## 1140         changes  181         <NA>     <NA>    NA
    ## 1141             law  181        trust     <NA>    NA
    ## 1142            meme  181         <NA>     <NA>    NA
    ## 1143         message  181         <NA>     <NA>    NA
    ## 1144            past  181         <NA>     <NA>    NA
    ## 1145         popcorn  181         <NA>     <NA>    NA
    ## 1146          states  181         <NA>     <NA>    NA
    ## 1147            want  181         <NA>     <NA>     1
    ## 1148          latest  180         <NA>     <NA>    NA
    ## 1149            meet  180         <NA>     <NA>    NA
    ## 1150           paint  180         <NA>     <NA>    NA
    ## 1151           sorry  180         <NA> negative    -1
    ## 1152        timeline  180         <NA>     <NA>    NA
    ## 1153          bought  179         <NA>     <NA>    NA
    ## 1154        continue  179 anticipation     <NA>    NA
    ## 1155        continue  179     positive     <NA>    NA
    ## 1156        continue  179        trust     <NA>    NA
    ## 1157         costume  179         <NA>     <NA>    NA
    ## 1158          freaks  179         <NA> negative    NA
    ## 1159          offers  179         <NA>     <NA>    NA
    ## 1160        required  179         <NA>     <NA>    NA
    ## 1161   controversial  178        anger negative    -2
    ## 1162   controversial  178     negative negative    -2
    ## 1163         germany  178         <NA>     <NA>    NA
    ## 1164         largest  178         <NA>     <NA>    NA
    ## 1165           named  178         <NA>     <NA>    NA
    ## 1166            pics  178         <NA>     <NA>    NA
    ## 1167           tells  178         <NA>     <NA>    NA
    ## 1168         anymore  177         <NA>     <NA>    NA
    ## 1169        compared  177         <NA>     <NA>    NA
    ## 1170        daughter  177          joy     <NA>    NA
    ## 1171        daughter  177     positive     <NA>    NA
    ## 1172            fine  177         <NA> positive     2
    ## 1173             pic  177         <NA>     <NA>    NA
    ## 1174        redditor  177         <NA>     <NA>    NA
    ## 1175      appreciate  176         <NA> positive     2
    ## 1176            away  176         <NA>     <NA>    NA
    ## 1177             buy  176         <NA>     <NA>    NA
    ## 1178         control  176         <NA>     <NA>    NA
    ## 1179          ethics  176     positive     <NA>    NA
    ## 1180           first  176         <NA>     <NA>    NA
    ## 1181            give  176         <NA>     <NA>    NA
    ## 1182          hidden  176     negative     <NA>    NA
    ## 1183           known  176         <NA>     <NA>    NA
    ## 1184            last  176         <NA>     <NA>    NA
    ## 1185         leaders  176         <NA>     <NA>    NA
    ## 1186           moved  176         <NA>     <NA>    NA
    ## 1187         prepare  176 anticipation     <NA>    NA
    ## 1188         prepare  176     positive     <NA>    NA
    ## 1189             saw  176         <NA>     <NA>    NA
    ## 1190         service  176         <NA>     <NA>    NA
    ## 1191          things  176         <NA>     <NA>    NA
    ## 1192          voting  176         <NA>     <NA>    NA
    ## 1193      widespread  176     positive     <NA>    NA
    ## 1194        bullshit  175         <NA> negative    -4
    ## 1195           bunch  175         <NA>     <NA>    NA
    ## 1196           fresh  175         <NA> positive     1
    ## 1197            hall  175         <NA>     <NA>    NA
    ## 1198           irish  175         <NA>     <NA>    NA
    ## 1199          russia  175         <NA>     <NA>    NA
    ## 1200          brexit  174         <NA>     <NA>    NA
    ## 1201            iron  174     positive     <NA>    NA
    ## 1202            iron  174        trust     <NA>    NA
    ## 1203       offensive  174        anger negative    NA
    ## 1204       offensive  174      disgust negative    NA
    ## 1205       offensive  174     negative negative    NA
    ## 1206          palace  174         <NA>     <NA>    NA
    ## 1207         panther  174         <NA>     <NA>    NA
    ## 1208            wife  174         <NA>     <NA>    NA
    ## 1209           asian  173         <NA>     <NA>    NA
    ## 1210       basically  173         <NA>     <NA>    NA
    ## 1211          debate  173     positive     <NA>    NA
    ## 1212              id  173         <NA>     <NA>    NA
    ## 1213            lion  173         fear     <NA>    NA
    ## 1214            lion  173     positive     <NA>    NA
    ## 1215        original  173         <NA>     <NA>    NA
    ## 1216         reveals  173         <NA>     <NA>    NA
    ## 1217           visit  173     positive     <NA>    NA
    ## 1218      destroying  172        anger     <NA>    -3
    ## 1219      destroying  172         fear     <NA>    -3
    ## 1220      destroying  172     negative     <NA>    -3
    ## 1221      destroying  172      sadness     <NA>    -3
    ## 1222            full  172     positive     <NA>    NA
    ## 1223         legally  172         <NA>     <NA>     1
    ## 1224          pursue  172         <NA>     <NA>    NA
    ## 1225         reports  172         <NA>     <NA>    NA
    ## 1226         resigns  172         <NA>     <NA>    -1
    ## 1227            uefa  172         <NA>     <NA>    NA
    ## 1228           korea  171         <NA>     <NA>    NA
    ## 1229          months  171         <NA>     <NA>    NA
    ## 1230       outbreaks  171         <NA>     <NA>    NA
    ## 1231            rise  171         <NA>     <NA>    NA
    ## 1232             don  170     positive     <NA>    NA
    ## 1233             don  170        trust     <NA>    NA
    ## 1234          effect  170         <NA>     <NA>    NA
    ## 1235           oscar  170         <NA>     <NA>    NA
    ## 1236            took  170         <NA>     <NA>    NA
    ## 1237           trial  170         <NA>     <NA>    NA
    ## 1238       beginning  169         <NA>     <NA>    NA
    ## 1239       dystopian  169         <NA>     <NA>    NA
    ## 1240          fourth  169         <NA>     <NA>    NA
    ## 1241             mad  169        anger negative    -3
    ## 1242             mad  169      disgust negative    -3
    ## 1243             mad  169         fear negative    -3
    ## 1244             mad  169     negative negative    -3
    ## 1245             mad  169      sadness negative    -3
    ## 1246            odds  169         <NA>     <NA>    NA
    ## 1247        pictures  169         <NA>     <NA>    NA
    ## 1248         sanders  169         <NA>     <NA>    NA
    ## 1249             see  169         <NA>     <NA>    NA
    ## 1250            sets  169         <NA>     <NA>    NA
    ## 1251             won  169         <NA> positive     3
    ## 1252          decade  168         <NA>     <NA>    NA
    ## 1253          giving  168     positive     <NA>    NA
    ## 1254             met  168         <NA>     <NA>    NA
    ## 1255          monday  168         <NA>     <NA>    NA
    ## 1256         premier  168     positive positive    NA
    ## 1257        problems  168         <NA> negative    -2
    ## 1258  representative  168         <NA>     <NA>    NA
    ## 1259         stephen  168         <NA>     <NA>    NA
    ## 1260           tweet  168         <NA>     <NA>    NA
    ## 1261        expected  167 anticipation     <NA>    NA
    ## 1262            kept  167         <NA>     <NA>    NA
    ## 1263          posted  167         <NA>     <NA>    NA
    ## 1264            food  166          joy     <NA>    NA
    ## 1265            food  166     positive     <NA>    NA
    ## 1266            food  166        trust     <NA>    NA
    ## 1267          former  166         <NA>     <NA>    NA
    ## 1268            goes  166         <NA>     <NA>    NA
    ## 1269             run  166         <NA>     <NA>    NA
    ## 1270         scandal  166         fear negative    -3
    ## 1271         scandal  166     negative negative    -3
    ## 1272           world  166         <NA>     <NA>    NA
    ## 1273   automatically  165         <NA>     <NA>    NA
    ## 1274          bayern  165         <NA>     <NA>    NA
    ## 1275          brings  165         <NA>     <NA>    NA
    ## 1276           chair  165         <NA>     <NA>    NA
    ## 1277        coverage  165         <NA>     <NA>    NA
    ## 1278        european  165         <NA>     <NA>    NA
    ## 1279       cambridge  164         <NA>     <NA>    NA
    ## 1280        database  164         <NA>     <NA>    NA
    ## 1281            guys  164         <NA>     <NA>    NA
    ## 1282       integrity  164     positive     <NA>     2
    ## 1283       integrity  164        trust     <NA>     2
    ## 1284          mental  164         <NA>     <NA>    NA
    ## 1285         monster  164         fear negative    NA
    ## 1286         monster  164     negative negative    NA
    ## 1287        publicly  164         <NA>     <NA>    NA
    ## 1288         attempt  163 anticipation     <NA>    NA
    ## 1289            city  163         <NA>     <NA>    NA
    ## 1290          comics  163         <NA>     <NA>    NA
    ## 1291           nancy  163         <NA>     <NA>    NA
    ## 1292             per  163         <NA>     <NA>    NA
    ## 1293          person  163         <NA>     <NA>    NA
    ## 1294         quickly  163         <NA>     <NA>    NA
    ## 1295             sat  163         <NA>     <NA>    NA
    ## 1296           alter  162         <NA>     <NA>    NA
    ## 1297          gaming  162         <NA>     <NA>    NA
    ## 1298       influence  162     negative     <NA>    NA
    ## 1299       influence  162     positive     <NA>    NA
    ## 1300         matters  162         <NA>     <NA>     1
    ## 1301        projects  162         <NA>     <NA>    NA
    ## 1302          winner  162 anticipation positive     4
    ## 1303          winner  162          joy positive     4
    ## 1304          winner  162     positive positive     4
    ## 1305          winner  162     surprise positive     4
    ## 1306         citizen  161     positive     <NA>    NA
    ## 1307          claims  161         <NA>     <NA>    NA
    ## 1308      conspiracy  161         fear negative    -3
    ## 1309          delete  161         <NA>     <NA>    NA
    ## 1310          ending  161         <NA>     <NA>    NA
    ## 1311             lab  161         <NA>     <NA>    NA
    ## 1312         missing  161         fear     <NA>    -2
    ## 1313         missing  161     negative     <NA>    -2
    ## 1314         missing  161      sadness     <NA>    -2
    ## 1315        powerful  161        anger positive     2
    ## 1316        powerful  161 anticipation positive     2
    ## 1317        powerful  161      disgust positive     2
    ## 1318        powerful  161         fear positive     2
    ## 1319        powerful  161          joy positive     2
    ## 1320        powerful  161     positive positive     2
    ## 1321        powerful  161        trust positive     2
    ## 1322            save  161          joy     <NA>     2
    ## 1323            save  161     positive     <NA>     2
    ## 1324            save  161        trust     <NA>     2
    ## 1325           study  161     positive     <NA>    NA
    ## 1326            time  161 anticipation     <NA>    NA
    ## 1327             bar  160         <NA>     <NA>    NA
    ## 1328            fuel  160         <NA>     <NA>    NA
    ## 1329          morgan  160         <NA>     <NA>    NA
    ## 1330       spreading  160         <NA>     <NA>    NA
    ## 1331      threatened  160         <NA>     <NA>    -2
    ## 1332               u  160         <NA>     <NA>    NA
    ## 1333           white  160 anticipation     <NA>    NA
    ## 1334           white  160          joy     <NA>    NA
    ## 1335           white  160     positive     <NA>    NA
    ## 1336           white  160        trust     <NA>    NA
    ## 1337          afraid  159         fear negative    -2
    ## 1338          afraid  159     negative negative    -2
    ## 1339           clash  159        anger negative    -2
    ## 1340           clash  159     negative negative    -2
    ## 1341              dc  159         <NA>     <NA>    NA
    ## 1342          marvel  159     positive positive     3
    ## 1343          marvel  159     surprise positive     3
    ## 1344            news  159         <NA>     <NA>    NA
    ## 1345            wars  159         <NA>     <NA>    NA
    ## 1346        absolute  158     positive     <NA>    NA
    ## 1347     blockbuster  158         <NA> positive     3
    ## 1348          bridge  158         <NA>     <NA>    NA
    ## 1349    contribution  158         <NA> positive    NA
    ## 1350           dress  158         <NA>     <NA>    NA
    ## 1351            laws  158         <NA>     <NA>    NA
    ## 1352         morning  158         <NA>     <NA>    NA
    ## 1353            name  158         <NA>     <NA>    NA
    ## 1354      republican  158         <NA>     <NA>    NA
    ## 1355          ruined  158        anger negative    -2
    ## 1356          ruined  158      disgust negative    -2
    ## 1357          ruined  158         fear negative    -2
    ## 1358          ruined  158     negative negative    -2
    ## 1359          ruined  158      sadness negative    -2
    ## 1360         strange  158         <NA> negative    -1
    ## 1361            club  157         <NA>     <NA>    NA
    ## 1362       interview  157         <NA>     <NA>    NA
    ## 1363             man  157         <NA>     <NA>    NA
    ## 1364          newest  157         <NA>     <NA>    NA
    ## 1365            post  157         <NA>     <NA>    NA
    ## 1366           prove  157     positive     <NA>    NA
    ## 1367            song  157         <NA>     <NA>    NA
    ## 1368            tree  157        anger     <NA>    NA
    ## 1369            tree  157 anticipation     <NA>    NA
    ## 1370            tree  157      disgust     <NA>    NA
    ## 1371            tree  157          joy     <NA>    NA
    ## 1372            tree  157     positive     <NA>    NA
    ## 1373            tree  157     surprise     <NA>    NA
    ## 1374            tree  157        trust     <NA>    NA
    ## 1375          warren  157         <NA>     <NA>    NA
    ## 1376           adult  156         <NA>     <NA>    NA
    ## 1377         classic  156     positive positive    NA
    ## 1378            copy  156     negative     <NA>    NA
    ## 1379            girl  156         <NA>     <NA>    NA
    ## 1380         holding  156         <NA>     <NA>    NA
    ## 1381        internet  156         <NA>     <NA>    NA
    ## 1382            lost  156     negative negative    -3
    ## 1383            lost  156      sadness negative    -3
    ## 1384        marriage  156 anticipation     <NA>    NA
    ## 1385        marriage  156          joy     <NA>    NA
    ## 1386        marriage  156     positive     <NA>    NA
    ## 1387        marriage  156        trust     <NA>    NA
    ## 1388            rest  156     positive     <NA>    NA
    ## 1389        suddenly  156     surprise     <NA>    NA
    ## 1390        targeted  156         <NA>     <NA>    NA
    ## 1391             act  155         <NA>     <NA>    NA
    ## 1392         counter  155         <NA>     <NA>    NA
    ## 1393           equal  155         <NA>     <NA>    NA
    ## 1394           month  155         <NA>     <NA>    NA
    ## 1395          random  155         <NA>     <NA>    NA
    ## 1396         stopped  155         <NA>     <NA>    -1
    ## 1397      supporters  155         <NA>     <NA>     1
    ## 1398           truck  155        trust     <NA>    NA
    ## 1399            user  155         <NA>     <NA>    NA
    ## 1400         wedding  155         <NA>     <NA>    NA
    ## 1401            also  154         <NA>     <NA>    NA
    ## 1402       cinematic  154         <NA>     <NA>    NA
    ## 1403            deal  154 anticipation     <NA>    NA
    ## 1404            deal  154          joy     <NA>    NA
    ## 1405            deal  154     positive     <NA>    NA
    ## 1406            deal  154     surprise     <NA>    NA
    ## 1407            deal  154        trust     <NA>    NA
    ## 1408         reverse  154         <NA>     <NA>    NA
    ## 1409           round  154         <NA>     <NA>    NA
    ## 1410           wrong  154     negative negative    -2
    ## 1411          ensues  153         <NA>     <NA>    NA
    ## 1412          exists  153         <NA>     <NA>    NA
    ## 1413               f  153         <NA>     <NA>    NA
    ## 1414         fucking  153         <NA> negative    -4
    ## 1415             men  153         <NA>     <NA>    NA
    ## 1416      originally  153         <NA>     <NA>    NA
    ## 1417          others  153         <NA>     <NA>    NA
    ## 1418               t  153         <NA>     <NA>    NA
    ## 1419      terminator  153         <NA>     <NA>    NA
    ## 1420         today's  153         <NA>     <NA>    NA
    ## 1421           truth  153     positive     <NA>    NA
    ## 1422           truth  153        trust     <NA>    NA
    ## 1423         asshole  152        anger     <NA>    -4
    ## 1424         asshole  152      disgust     <NA>    -4
    ## 1425         asshole  152     negative     <NA>    -4
    ## 1426          become  152         <NA>     <NA>    NA
    ## 1427            boys  152         <NA>     <NA>    NA
    ## 1428             etc  152         <NA>     <NA>    NA
    ## 1429          filter  152         <NA>     <NA>    NA
    ## 1430          leader  152     positive     <NA>    NA
    ## 1431          leader  152        trust     <NA>    NA
    ## 1432        national  152         <NA>     <NA>    NA
    ## 1433              pa  152         <NA>     <NA>    NA
    ## 1434           title  152     positive     <NA>    NA
    ## 1435           title  152        trust     <NA>    NA
    ## 1436       campaigns  151         <NA>     <NA>    NA
    ## 1437      commission  151        trust     <NA>    NA
    ## 1438      constantly  151        trust     <NA>    NA
    ## 1439        download  151         <NA>     <NA>    NA
    ## 1440   investigation  151 anticipation     <NA>    NA
    ## 1441            week  151         <NA>     <NA>    NA
    ## 1442          budget  150        trust     <NA>    NA
    ## 1443        thriller  150         <NA>     <NA>    NA
    ## 1444         tonight  150         <NA>     <NA>    NA
    ## 1445           ahead  149     positive     <NA>    NA
    ## 1446        announce  149         <NA>     <NA>    NA
    ## 1447          attack  149        anger negative    -1
    ## 1448          attack  149         fear negative    -1
    ## 1449          attack  149     negative negative    -1
    ## 1450        children  149         <NA>     <NA>    NA
    ## 1451         council  149 anticipation     <NA>    NA
    ## 1452         council  149     positive     <NA>    NA
    ## 1453         council  149        trust     <NA>    NA
    ## 1454        director  149     positive     <NA>    NA
    ## 1455        director  149        trust     <NA>    NA
    ## 1456        hundreds  149         <NA>     <NA>    NA
    ## 1457          nation  149        trust     <NA>    NA
    ## 1458            poll  149        trust     <NA>    NA
    ## 1459            said  149         <NA>     <NA>    NA
    ## 1460          turned  149         <NA>     <NA>    NA
    ## 1461            bond  148         <NA>     <NA>    NA
    ## 1462         correct  148         <NA> positive    NA
    ## 1463            dave  148         <NA>     <NA>    NA
    ## 1464      discussing  148         <NA>     <NA>    NA
    ## 1465            five  148         <NA>     <NA>    NA
    ## 1466          health  148         <NA>     <NA>    NA
    ## 1467       scientist  148 anticipation     <NA>    NA
    ## 1468       scientist  148     positive     <NA>    NA
    ## 1469       scientist  148        trust     <NA>    NA
    ## 1470           upset  148        anger negative    -2
    ## 1471           upset  148     negative negative    -2
    ## 1472           upset  148      sadness negative    -2
    ## 1473            fair  147     positive positive     2
    ## 1474             gem  147          joy positive    NA
    ## 1475             gem  147     positive positive    NA
    ## 1476          island  147         <NA>     <NA>    NA
    ## 1477            plus  147         <NA>     <NA>    NA
    ## 1478         raising  147         <NA>     <NA>    NA
    ## 1479         remains  147      disgust     <NA>    NA
    ## 1480         remains  147         fear     <NA>    NA
    ## 1481         remains  147     negative     <NA>    NA
    ## 1482         remains  147     positive     <NA>    NA
    ## 1483         remains  147        trust     <NA>    NA
    ## 1484     republicans  147         <NA>     <NA>    NA
    ## 1485          second  147         <NA>     <NA>    NA
    ## 1486           squad  147         <NA>     <NA>    NA
    ## 1487      unexpected  147 anticipation negative    NA
    ## 1488      unexpected  147         fear negative    NA
    ## 1489      unexpected  147          joy negative    NA
    ## 1490      unexpected  147     negative negative    NA
    ## 1491      unexpected  147     positive negative    NA
    ## 1492      unexpected  147     surprise negative    NA
    ## 1493           wuhan  147         <NA>     <NA>    NA
    ## 1494         capital  146         <NA>     <NA>    NA
    ## 1495        facebook  146         <NA>     <NA>    NA
    ## 1496           japan  146         <NA>     <NA>    NA
    ## 1497             let  146         <NA>     <NA>    NA
    ## 1498         muslims  146         <NA>     <NA>    NA
    ## 1499        convince  145 anticipation     <NA>     1
    ## 1500        convince  145     positive     <NA>     1
    ## 1501        convince  145        trust     <NA>     1
    ## 1502          forgot  145         <NA>     <NA>    NA
    ## 1503            game  145         <NA>     <NA>    NA
    ## 1504          hitler  145         <NA>     <NA>    NA
    ## 1505         primary  145     positive     <NA>    NA
    ## 1506               y  145         <NA>     <NA>    NA
    ## 1507           bills  144         <NA>     <NA>    NA
    ## 1508         debates  144         <NA>     <NA>    NA
    ## 1509            four  144         <NA>     <NA>    NA
    ## 1510            grab  144        anger     <NA>    NA
    ## 1511            grab  144     negative     <NA>    NA
    ## 1512             gun  144        anger     <NA>    -1
    ## 1513             gun  144         fear     <NA>    -1
    ## 1514             gun  144     negative     <NA>    -1
    ## 1515           house  144         <NA>     <NA>    NA
    ## 1516        illinois  144         <NA>     <NA>    NA
    ## 1517               m  144         <NA>     <NA>    NA
    ## 1518         members  144         <NA>     <NA>    NA
    ## 1519           never  144         <NA>     <NA>    NA
    ## 1520        produced  144         <NA>     <NA>    NA
    ## 1521         running  144         <NA>     <NA>    NA
    ## 1522          slowly  144         <NA> negative    NA
    ## 1523           storm  144        anger     <NA>    NA
    ## 1524           storm  144     negative     <NA>    NA
    ## 1525       thousands  144         <NA>     <NA>    NA
    ## 1526      understand  144         <NA>     <NA>    NA
    ## 1527         unknown  144 anticipation negative    NA
    ## 1528         unknown  144         fear negative    NA
    ## 1529         unknown  144     negative negative    NA
    ## 1530           awful  143        anger negative    -3
    ## 1531           awful  143      disgust negative    -3
    ## 1532           awful  143         fear negative    -3
    ## 1533           awful  143     negative negative    -3
    ## 1534           awful  143      sadness negative    -3
    ## 1535        choosing  143         <NA>     <NA>    NA
    ## 1536           clubs  143         <NA>     <NA>    NA
    ## 1537       copyright  143         <NA>     <NA>    NA
    ## 1538        creators  143         <NA>     <NA>    NA
    ## 1539       criticism  143        anger negative    -2
    ## 1540       criticism  143     negative negative    -2
    ## 1541       criticism  143      sadness negative    -2
    ## 1542              go  143         <NA>     <NA>    NA
    ## 1543            left  143         <NA>     <NA>    NA
    ## 1544       legendary  143     positive positive    NA
    ## 1545             now  143         <NA>     <NA>    NA
    ## 1546           order  143         <NA>     <NA>    NA
    ## 1547       political  143         <NA>     <NA>    NA
    ## 1548        saturday  143         <NA>     <NA>    NA
    ## 1549           taken  143         <NA>     <NA>    NA
    ## 1550             tax  143     negative     <NA>    NA
    ## 1551             tax  143      sadness     <NA>    NA
    ## 1552            cast  142         <NA>     <NA>    NA
    ## 1553      complaints  142         <NA> negative    NA
    ## 1554         dollars  142         <NA>     <NA>    NA
    ## 1555          female  142     positive     <NA>    NA
    ## 1556          locked  142         <NA>     <NA>    NA
    ## 1557       marijuana  142         <NA>     <NA>    NA
    ## 1558          police  142         fear     <NA>    NA
    ## 1559          police  142     positive     <NA>    NA
    ## 1560          police  142        trust     <NA>    NA
    ## 1561       professor  142     positive     <NA>    NA
    ## 1562       professor  142        trust     <NA>    NA
    ## 1563         refuses  142         <NA> negative    NA
    ## 1564          showed  142         <NA>     <NA>    NA
    ## 1565       workforce  142         <NA>     <NA>    NA
    ## 1566            gold  141     positive positive    NA
    ## 1567            hero  141 anticipation positive     2
    ## 1568            hero  141          joy positive     2
    ## 1569            hero  141     positive positive     2
    ## 1570            hero  141     surprise positive     2
    ## 1571            hero  141        trust positive     2
    ## 1572              ii  141         <NA>     <NA>    NA
    ## 1573            mass  141         <NA>     <NA>    NA
    ## 1574             ran  141         <NA>     <NA>    NA
    ## 1575        received  141     positive     <NA>    NA
    ## 1576          search  141         <NA>     <NA>    NA
    ## 1577          strong  141         <NA> positive     2
    ## 1578        supplies  141     positive     <NA>    NA
    ## 1579            term  141         <NA>     <NA>    NA
    ## 1580           trust  141        trust positive     1
    ## 1581           voter  141         <NA>     <NA>    NA
    ## 1582         website  141         <NA>     <NA>    NA
    ## 1583       worldnews  141         <NA>     <NA>    NA
    ## 1584            adam  140         <NA>     <NA>    NA
    ## 1585      autonomous  140         <NA> positive    NA
    ## 1586             pay  140 anticipation     <NA>    -1
    ## 1587             pay  140          joy     <NA>    -1
    ## 1588             pay  140     positive     <NA>    -1
    ## 1589             pay  140        trust     <NA>    -1
    ## 1590        producer  140     positive     <NA>    NA
    ## 1591           proof  140        trust     <NA>    NA
    ## 1592           scale  140         <NA>     <NA>    NA
    ## 1593         started  140         <NA>     <NA>    NA
    ## 1594         watches  140         <NA>     <NA>    NA
    ## 1595           water  140         <NA>     <NA>    NA
    ## 1596           alive  139 anticipation     <NA>     1
    ## 1597           alive  139          joy     <NA>     1
    ## 1598           alive  139     positive     <NA>     1
    ## 1599           alive  139        trust     <NA>     1
    ## 1600           enjoy  139 anticipation positive     2
    ## 1601           enjoy  139          joy positive     2
    ## 1602           enjoy  139     positive positive     2
    ## 1603           enjoy  139        trust positive     2
    ## 1604            hate  139        anger negative    -3
    ## 1605            hate  139      disgust negative    -3
    ## 1606            hate  139         fear negative    -3
    ## 1607            hate  139     negative negative    -3
    ## 1608            hate  139      sadness negative    -3
    ## 1609           knows  139         <NA>     <NA>    NA
    ## 1610        magazine  139         <NA>     <NA>    NA
    ## 1611        mourinho  139         <NA>     <NA>    NA
    ## 1612        policies  139         <NA>     <NA>    NA
    ## 1613           pride  139          joy positive    NA
    ## 1614           pride  139     positive positive    NA
    ## 1615          profit  139         <NA>     <NA>    NA
    ## 1616            punk  139         <NA> negative    NA
    ## 1617        straight  139         <NA>     <NA>     1
    ## 1618          worker  139         <NA>     <NA>    NA
    ## 1619           coach  138        trust     <NA>    NA
    ## 1620        election  138         <NA>     <NA>    NA
    ## 1621       electoral  138         <NA>     <NA>    NA
    ## 1622         experts  138         <NA>     <NA>    NA
    ## 1623          father  138        trust     <NA>    NA
    ## 1624           gotta  138         <NA>     <NA>    NA
    ## 1625            kiss  138 anticipation     <NA>     2
    ## 1626            kiss  138          joy     <NA>     2
    ## 1627            kiss  138     positive     <NA>     2
    ## 1628            kiss  138     surprise     <NA>     2
    ## 1629             map  138         <NA>     <NA>    NA
    ## 1630           paper  138         <NA>     <NA>    NA
    ## 1631           piece  138         <NA>     <NA>    NA
    ## 1632           shuts  138         <NA>     <NA>    NA
    ## 1633         texting  138         <NA>     <NA>    NA
    ## 1634         wearing  138         <NA>     <NA>    NA
    ## 1635          berlin  137         <NA>     <NA>    NA
    ## 1636           cross  137        anger     <NA>    NA
    ## 1637           cross  137         fear     <NA>    NA
    ## 1638           cross  137     negative     <NA>    NA
    ## 1639           cross  137      sadness     <NA>    NA
    ## 1640            didn  137         <NA>     <NA>    NA
    ## 1641        examples  137         <NA>     <NA>    NA
    ## 1642       influenza  137     negative     <NA>    NA
    ## 1643            long  137 anticipation     <NA>    NA
    ## 1644          reason  137     positive     <NA>    NA
    ## 1645         refused  137     negative negative    -2
    ## 1646         refused  137      sadness negative    -2
    ## 1647        released  137         <NA>     <NA>    NA
    ## 1648       socialism  137      disgust     <NA>    NA
    ## 1649       socialism  137         fear     <NA>    NA
    ## 1650            town  137         <NA>     <NA>    NA
    ## 1651       unpopular  137      disgust negative    NA
    ## 1652       unpopular  137     negative negative    NA
    ## 1653       unpopular  137      sadness negative    NA
    ## 1654             win  137         <NA> positive     4
    ## 1655         alleged  136         <NA>     <NA>    NA
    ## 1656        avengers  136         <NA>     <NA>    NA
    ## 1657          became  136         <NA>     <NA>    NA
    ## 1658            dies  136         <NA> negative    NA
    ## 1659          dinner  136     positive     <NA>    NA
    ## 1660             era  136         <NA>     <NA>    NA
    ## 1661             son  136         <NA>     <NA>    NA
    ## 1662            trey  136         <NA>     <NA>    NA
    ## 1663        ultimate  136 anticipation     <NA>    NA
    ## 1664        ultimate  136      sadness     <NA>    NA
    ## 1665            dumb  135     negative negative    -3
    ## 1666       employees  135         <NA>     <NA>    NA
    ## 1667          enough  135         <NA> positive    NA
    ## 1668            exit  135         <NA>     <NA>    NA
    ## 1669         feature  135     positive     <NA>    NA
    ## 1670         finally  135 anticipation     <NA>    NA
    ## 1671         finally  135      disgust     <NA>    NA
    ## 1672         finally  135          joy     <NA>    NA
    ## 1673         finally  135     positive     <NA>    NA
    ## 1674         finally  135     surprise     <NA>    NA
    ## 1675         finally  135        trust     <NA>    NA
    ## 1676         liberal  135     negative     <NA>    NA
    ## 1677         liberal  135     positive     <NA>    NA
    ## 1678        patients  135         <NA>     <NA>    NA
    ## 1679     performance  135         <NA>     <NA>    NA
    ## 1680         prevent  135         fear     <NA>    -1
    ## 1681            pull  135     positive     <NA>    NA
    ## 1682         remakes  135         <NA>     <NA>    NA
    ## 1683          sequel  135 anticipation     <NA>    NA
    ## 1684           short  135         <NA>     <NA>    NA
    ## 1685           young  135 anticipation     <NA>    NA
    ## 1686           young  135          joy     <NA>    NA
    ## 1687           young  135     positive     <NA>    NA
    ## 1688           young  135     surprise     <NA>    NA
    ## 1689         alcohol  134         <NA>     <NA>    NA
    ## 1690          avatar  134     positive     <NA>    NA
    ## 1691         endgame  134         <NA>     <NA>    NA
    ## 1692     explanation  134         <NA>     <NA>    NA
    ## 1693           fired  134         <NA>     <NA>    -2
    ## 1694           goods  134     positive     <NA>    NA
    ## 1695           views  134         <NA>     <NA>    NA
    ## 1696          bernie  133         <NA>     <NA>    NA
    ## 1697            fans  133         <NA> positive    NA
    ## 1698       franchise  133         <NA>     <NA>    NA
    ## 1699           kinda  133         <NA>     <NA>    NA
    ## 1700            till  133         <NA>     <NA>    NA
    ## 1701            type  133         <NA>     <NA>    NA
    ## 1702          weapon  133         <NA>     <NA>    NA
    ## 1703        accounts  132        trust     <NA>    NA
    ## 1704          arrest  132     negative     <NA>    -2
    ## 1705             ben  132         <NA>     <NA>    NA
    ## 1706           books  132         <NA>     <NA>    NA
    ## 1707            call  132         <NA>     <NA>    NA
    ## 1708          cinema  132         <NA>     <NA>    NA
    ## 1709         killing  132        anger negative    -3
    ## 1710         killing  132         fear negative    -3
    ## 1711         killing  132     negative negative    -3
    ## 1712         killing  132      sadness negative    -3
    ## 1713           ronda  132         <NA>     <NA>    NA
    ## 1714      scientists  132         <NA>     <NA>    NA
    ## 1715           tasks  132         <NA>     <NA>    NA
    ## 1716            urge  132         <NA>     <NA>    NA
    ## 1717            aged  131         <NA>     <NA>    NA
    ## 1718          broken  131        anger negative    -1
    ## 1719          broken  131         fear negative    -1
    ## 1720          broken  131     negative negative    -1
    ## 1721          broken  131      sadness negative    -1
    ## 1722       everybody  131         <NA>     <NA>    NA
    ## 1723          please  131         <NA>     <NA>     1
    ## 1724            puts  131         <NA>     <NA>    NA
    ## 1725          regret  131     negative negative    -2
    ## 1726          regret  131      sadness negative    -2
    ## 1727            york  131         <NA>     <NA>    NA
    ## 1728             day  130         <NA>     <NA>    NA
    ## 1729         disease  130        anger     <NA>    NA
    ## 1730         disease  130      disgust     <NA>    NA
    ## 1731         disease  130         fear     <NA>    NA
    ## 1732         disease  130     negative     <NA>    NA
    ## 1733         disease  130      sadness     <NA>    NA
    ## 1734            dude  130         <NA>     <NA>    NA
    ## 1735        response  130         <NA>     <NA>    NA
    ## 1736             sub  130         <NA>     <NA>    NA
    ## 1737       suspended  130         <NA>     <NA>    -1
    ## 1738            wild  130     negative negative    NA
    ## 1739            wild  130     surprise negative    NA
    ## 1740           birth  129 anticipation     <NA>    NA
    ## 1741           birth  129         fear     <NA>    NA
    ## 1742           birth  129          joy     <NA>    NA
    ## 1743           birth  129     positive     <NA>    NA
    ## 1744           birth  129        trust     <NA>    NA
    ## 1745           brain  129         <NA>     <NA>    NA
    ## 1746         corrupt  129     negative negative    NA
    ## 1747         extreme  129         <NA>     <NA>    NA
    ## 1748            fear  129        anger negative    -2
    ## 1749            fear  129         fear negative    -2
    ## 1750            fear  129     negative negative    -2
    ## 1751            john  129      disgust     <NA>    NA
    ## 1752            john  129     negative     <NA>    NA
    ## 1753          nobody  129         <NA>     <NA>    NA
    ## 1754           pulls  129         <NA>     <NA>    NA
    ## 1755            self  129         <NA>     <NA>    NA
    ## 1756           share  129 anticipation     <NA>     1
    ## 1757           share  129          joy     <NA>     1
    ## 1758           share  129     positive     <NA>     1
    ## 1759           share  129        trust     <NA>     1
    ## 1760           shows  129         <NA>     <NA>    NA
    ## 1761            used  129         <NA>     <NA>    NA
    ## 1762        accuracy  128         <NA>     <NA>    NA
    ## 1763          arnold  128         <NA>     <NA>    NA
    ## 1764        bringing  128         <NA>     <NA>    NA
    ## 1765            case  128         fear     <NA>    NA
    ## 1766            case  128     negative     <NA>    NA
    ## 1767            case  128      sadness     <NA>    NA
    ## 1768       champions  128         <NA>     <NA>    NA
    ## 1769        confirms  128         <NA>     <NA>    NA
    ## 1770         exactly  128         <NA>     <NA>    NA
    ## 1771          fights  128         <NA>     <NA>    NA
    ## 1772           group  128         <NA>     <NA>    NA
    ## 1773         painted  128         <NA>     <NA>    NA
    ## 1774          public  128 anticipation     <NA>    NA
    ## 1775          public  128     positive     <NA>    NA
    ## 1776        responds  128         <NA>     <NA>    NA
    ## 1777         sequels  128         <NA>     <NA>    NA
    ## 1778           small  128     negative     <NA>    NA
    ## 1779        violence  128        anger     <NA>    -3
    ## 1780        violence  128         fear     <NA>    -3
    ## 1781        violence  128     negative     <NA>    -3
    ## 1782        violence  128      sadness     <NA>    -3
    ## 1783         warning  128         fear negative    -3
    ## 1784        actually  127         <NA>     <NA>    NA
    ## 1785          bottle  127         <NA>     <NA>    NA
    ## 1786          choose  127         <NA>     <NA>    NA
    ## 1787           chris  127         <NA>     <NA>    NA
    ## 1788        cleaning  127     positive     <NA>    NA
    ## 1789          driver  127         <NA>     <NA>    NA
    ## 1790             far  127         <NA>     <NA>    NA
    ## 1791           harry  127        anger     <NA>    NA
    ## 1792           harry  127     negative     <NA>    NA
    ## 1793           harry  127      sadness     <NA>    NA
    ## 1794           james  127         <NA>     <NA>    NA
    ## 1795        juventus  127         <NA>     <NA>    NA
    ## 1796        memories  127         <NA>     <NA>    NA
    ## 1797            poor  127         <NA> negative    -2
    ## 1798            site  127         <NA>     <NA>    NA
    ## 1799          switch  127         <NA>     <NA>    NA
    ## 1800             war  127         fear     <NA>    -2
    ## 1801             war  127     negative     <NA>    -2
    ## 1802            wcgw  127         <NA>     <NA>    NA
    ## 1803     advertising  126         <NA>     <NA>    NA
    ## 1804         african  126         <NA>     <NA>    NA
    ## 1805           board  126 anticipation     <NA>    NA
    ## 1806       childhood  126          joy     <NA>    NA
    ## 1807       childhood  126     positive     <NA>    NA
    ## 1808           covid  126         <NA>     <NA>    NA
    ## 1809            door  126         <NA>     <NA>    NA
    ## 1810           liked  126         <NA> positive     2
    ## 1811            love  126          joy positive     3
    ## 1812            love  126     positive positive     3
    ## 1813         payment  126     negative     <NA>    NA
    ## 1814        reminder  126         <NA>     <NA>    NA
    ## 1815         shouldn  126         <NA>     <NA>    NA
    ## 1816         working  126     positive     <NA>    NA
    ## 1817         arrival  125 anticipation     <NA>    NA
    ## 1818          beyond  125         <NA>     <NA>    NA
    ## 1819        everyone  125         <NA>     <NA>    NA
    ## 1820          german  125         <NA>     <NA>    NA
    ## 1821             got  125         <NA>     <NA>    NA
    ## 1822         hearing  125         fear     <NA>    NA
    ## 1823         hearing  125     negative     <NA>    NA
    ## 1824            help  125         <NA>     <NA>     2
    ## 1825            july  125         <NA>     <NA>    NA
    ## 1826           leads  125         <NA> positive    NA
    ## 1827             lol  125         <NA>     <NA>     3
    ## 1828         married  125         <NA>     <NA>    NA
    ## 1829           shirt  125         <NA>     <NA>    NA
    ## 1830        tomorrow  125 anticipation     <NA>    NA
    ## 1831          trying  125         <NA>     <NA>    NA
    ## 1832         twitter  125         <NA>     <NA>    NA
    ## 1833         attacks  124         <NA> negative    -1
    ## 1834          critic  124     negative negative    -2
    ## 1835            fast  124         <NA> positive    NA
    ## 1836         meaning  124         <NA>     <NA>    NA
    ## 1837             new  124         <NA>     <NA>    NA
    ## 1838         provide  124     positive     <NA>    NA
    ## 1839         provide  124        trust     <NA>    NA
    ## 1840             two  124         <NA>     <NA>    NA
    ## 1841            come  123         <NA>     <NA>    NA
    ## 1842          either  123         <NA>     <NA>    NA
    ## 1843             hot  123        anger positive    NA
    ## 1844           idiot  123      disgust negative    -3
    ## 1845           idiot  123     negative negative    -3
    ## 1846        language  123         <NA>     <NA>    NA
    ## 1847           legal  123     positive     <NA>     1
    ## 1848           legal  123        trust     <NA>     1
    ## 1849            make  123         <NA>     <NA>    NA
    ## 1850              nc  123         <NA>     <NA>    NA
    ## 1851           owned  123         <NA>     <NA>    NA
    ## 1852        properly  123         <NA> positive    NA
    ## 1853              re  123         <NA>     <NA>    NA
    ## 1854             rob  123        anger     <NA>    -2
    ## 1855             rob  123      disgust     <NA>    -2
    ## 1856             rob  123         fear     <NA>    -2
    ## 1857             rob  123     negative     <NA>    -2
    ## 1858             rob  123      sadness     <NA>    -2
    ## 1859         selling  123         <NA>     <NA>    NA
    ## 1860           story  123         <NA>     <NA>    NA
    ## 1861          system  123        trust     <NA>    NA
    ## 1862             toy  123         <NA>     <NA>    NA
    ## 1863         veteran  123     positive     <NA>    NA
    ## 1864         veteran  123        trust     <NA>    NA
    ## 1865         victory  123 anticipation positive    NA
    ## 1866         victory  123          joy positive    NA
    ## 1867         victory  123     positive positive    NA
    ## 1868         victory  123        trust positive    NA
    ## 1869           alien  122      disgust     <NA>    NA
    ## 1870           alien  122         fear     <NA>    NA
    ## 1871           alien  122     negative     <NA>    NA
    ## 1872       beautiful  122          joy positive     3
    ## 1873       beautiful  122     positive positive     3
    ## 1874        football  122 anticipation     <NA>    NA
    ## 1875        football  122          joy     <NA>    NA
    ## 1876        football  122     positive     <NA>    NA
    ## 1877      historical  122         <NA>     <NA>    NA
    ## 1878          insult  122        anger negative    -2
    ## 1879          insult  122      disgust negative    -2
    ## 1880          insult  122     negative negative    -2
    ## 1881          insult  122      sadness negative    -2
    ## 1882          insult  122     surprise negative    -2
    ## 1883             low  122         <NA>     <NA>    NA
    ## 1884          murder  122        anger negative    -2
    ## 1885          murder  122      disgust negative    -2
    ## 1886          murder  122         fear negative    -2
    ## 1887          murder  122     negative negative    -2
    ## 1888          murder  122      sadness negative    -2
    ## 1889          murder  122     surprise negative    -2
    ## 1890           roast  122         <NA>     <NA>    NA
    ## 1891      television  122         <NA>     <NA>    NA
    ## 1892        campaign  121         <NA>     <NA>    NA
    ## 1893       different  121         <NA>     <NA>    NA
    ## 1894            epic  121     positive     <NA>    NA
    ## 1895           fears  121         <NA> negative    NA
    ## 1896           force  121        anger     <NA>    NA
    ## 1897           force  121         fear     <NA>    NA
    ## 1898           force  121     negative     <NA>    NA
    ## 1899          ignore  121     negative negative    -1
    ## 1900            less  121         <NA>     <NA>    NA
    ## 1901    manipulation  121        anger negative    -1
    ## 1902    manipulation  121         fear negative    -1
    ## 1903    manipulation  121     negative negative    -1
    ## 1904        millions  121         <NA>     <NA>    NA
    ## 1905        opinions  121         <NA>     <NA>    NA
    ## 1906      prediction  121 anticipation     <NA>    NA
    ## 1907           promo  121         <NA>     <NA>    NA
    ## 1908          rigged  121         <NA>     <NA>    -1
    ## 1909            take  121         <NA>     <NA>    NA
    ## 1910         appears  120         <NA>     <NA>    NA
    ## 1911        business  120         <NA>     <NA>    NA
    ## 1912          cancer  120        anger negative    -1
    ## 1913          cancer  120      disgust negative    -1
    ## 1914          cancer  120         fear negative    -1
    ## 1915          cancer  120     negative negative    -1
    ## 1916          cancer  120      sadness negative    -1
    ## 1917          center  120     positive     <NA>    NA
    ## 1918          center  120        trust     <NA>    NA
    ## 1919          county  120        trust     <NA>    NA
    ## 1920        drinking  120     negative     <NA>    NA
    ## 1921       elections  120         <NA>     <NA>    NA
    ## 1922           every  120         <NA>     <NA>    NA
    ## 1923          higher  120         <NA>     <NA>    NA
    ## 1924            keep  120         <NA>     <NA>    NA
    ## 1925            move  120         <NA>     <NA>    NA
    ## 1926    presidential  120         <NA>     <NA>    NA
    ## 1927        sleeping  120         <NA>     <NA>    NA
    ## 1928           store  120 anticipation     <NA>    NA
    ## 1929           store  120     positive     <NA>    NA
    ## 1930        surprise  120         fear     <NA>    NA
    ## 1931        surprise  120          joy     <NA>    NA
    ## 1932        surprise  120     positive     <NA>    NA
    ## 1933        surprise  120     surprise     <NA>    NA
    ## 1934          thrown  120         <NA>     <NA>    NA
    ## 1935             try  120         <NA>     <NA>    NA
    ## 1936           drops  119         <NA>     <NA>    NA
    ## 1937            feel  119         <NA>     <NA>    NA
    ## 1938         filming  119         <NA>     <NA>    NA
    ## 1939          flying  119         fear     <NA>    NA
    ## 1940          flying  119     positive     <NA>    NA
    ## 1941            look  119         <NA>     <NA>    NA
    ## 1942           party  119         <NA>     <NA>    NA
    ## 1943         program  119         <NA>     <NA>    NA
    ## 1944            push  119         <NA>     <NA>    NA
    ## 1945      reasonable  119         <NA> positive    NA
    ## 1946          rising  119 anticipation     <NA>    NA
    ## 1947          rising  119          joy     <NA>    NA
    ## 1948          rising  119     positive     <NA>    NA
    ## 1949            role  119         <NA>     <NA>    NA
    ## 1950            shut  119         <NA>     <NA>    NA
    ## 1951         station  119         <NA>     <NA>    NA
    ## 1952         allowed  118         <NA>     <NA>    NA
    ## 1953         assault  118        anger negative    NA
    ## 1954         assault  118         fear negative    NA
    ## 1955         assault  118     negative negative    NA
    ## 1956             due  118         <NA>     <NA>    NA
    ## 1957             god  118 anticipation     <NA>     1
    ## 1958             god  118         fear     <NA>     1
    ## 1959             god  118          joy     <NA>     1
    ## 1960             god  118     positive     <NA>     1
    ## 1961             god  118        trust     <NA>     1
    ## 1962          growth  118     positive     <NA>     2
    ## 1963        hedgehog  118         <NA>     <NA>    NA
    ## 1964           lines  118         fear     <NA>    NA
    ## 1965            nazi  118         <NA>     <NA>    NA
    ## 1966            near  118         <NA>     <NA>    NA
    ## 1967       nominated  118         <NA>     <NA>    NA
    ## 1968           north  118         <NA>     <NA>    NA
    ## 1969          player  118     negative     <NA>    NA
    ## 1970        politics  118        anger     <NA>    NA
    ## 1971          potter  118         <NA>     <NA>    NA
    ## 1972            ship  118 anticipation     <NA>    NA
    ## 1973           solid  118     positive positive     2
    ## 1974          august  117     positive     <NA>    NA
    ## 1975            days  117         <NA>     <NA>    NA
    ## 1976         deleted  117         <NA>     <NA>    NA
    ## 1977           drive  117         <NA>     <NA>    NA
    ## 1978            face  117         <NA>     <NA>    NA
    ## 1979          fellow  117     positive     <NA>    NA
    ## 1980          fellow  117        trust     <NA>    NA
    ## 1981      healthcare  117         <NA>     <NA>    NA
    ## 1982           hopes  117         <NA>     <NA>     2
    ## 1983         leading  117        trust positive    NA
    ## 1984            much  117         <NA>     <NA>    NA
    ## 1985            nasa  117         <NA>     <NA>    NA
    ## 1986            nine  117         <NA>     <NA>    NA
    ## 1987          poland  117         <NA>     <NA>    NA
    ## 1988         posters  117         <NA>     <NA>    NA
    ## 1989           rolls  117         <NA>     <NA>    NA
    ## 1990        security  117         <NA>     <NA>    NA
    ## 1991       seriously  117         <NA>     <NA>    NA
    ## 1992        spoilers  117         <NA>     <NA>    NA
    ## 1993            arms  116         <NA>     <NA>    NA
    ## 1994       australia  116         <NA>     <NA>    NA
    ## 1995          deadly  116        anger negative    NA
    ## 1996          deadly  116      disgust negative    NA
    ## 1997          deadly  116         fear negative    NA
    ## 1998          deadly  116     negative negative    NA
    ## 1999          deadly  116      sadness negative    NA
    ## 2000            done  116         <NA>     <NA>    NA
    ## 2001           falls  116         <NA> negative    NA
    ## 2002        favorite  116          joy positive     2
    ## 2003        favorite  116     positive positive     2
    ## 2004        favorite  116        trust positive     2
    ## 2005       institute  116        trust     <NA>    NA
    ## 2006          invest  116         <NA>     <NA>    NA
    ## 2007          madrid  116         <NA>     <NA>    NA
    ## 2008          signed  116         <NA>     <NA>    NA
    ## 2009       standards  116         <NA>     <NA>    NA
    ## 2010       arguments  115        anger     <NA>    NA
    ## 2011          causes  115         <NA>     <NA>    NA
    ## 2012         chances  115         <NA>     <NA>     2
    ## 2013          cruise  115         <NA>     <NA>    NA
    ## 2014       currently  115         <NA>     <NA>    NA
    ## 2015             gay  115         <NA>     <NA>    NA
    ## 2016          images  115         <NA>     <NA>    NA
    ## 2017     intelligent  115     positive positive     2
    ## 2018     intelligent  115        trust positive     2
    ## 2019         minutes  115         <NA>     <NA>    NA
    ## 2020           photo  115         <NA>     <NA>    NA
    ## 2021           power  115         <NA>     <NA>    NA
    ## 2022             pre  115         <NA>     <NA>    NA
    ## 2023           votes  115         <NA>     <NA>    NA
    ## 2024            came  114         <NA>     <NA>    NA
    ## 2025          circle  114         <NA>     <NA>    NA
    ## 2026   conservatives  114         <NA>     <NA>    NA
    ## 2027          damage  114        anger negative    -3
    ## 2028          damage  114      disgust negative    -3
    ## 2029          damage  114     negative negative    -3
    ## 2030          damage  114      sadness negative    -3
    ## 2031         deserve  114        anger     <NA>    NA
    ## 2032         deserve  114 anticipation     <NA>    NA
    ## 2033         deserve  114     positive     <NA>    NA
    ## 2034         deserve  114        trust     <NA>    NA
    ## 2035         driving  114         <NA>     <NA>    NA
    ## 2036            goal  114         <NA>     <NA>    NA
    ## 2037      impossible  114     negative negative    NA
    ## 2038      impossible  114      sadness negative    NA
    ## 2039            lady  114         <NA>     <NA>    NA
    ## 2040         predict  114 anticipation     <NA>    NA
    ## 2041            rare  114         <NA>     <NA>    NA
    ## 2042         seconds  114         <NA>     <NA>    NA
    ## 2043            team  114        trust     <NA>    NA
    ## 2044        trending  114         <NA>     <NA>    NA
    ## 2045            uses  114         <NA>     <NA>    NA
    ## 2046          wonder  114         <NA> positive    NA
    ## 2047            able  113         <NA>     <NA>    NA
    ## 2048         details  113         <NA>     <NA>    NA
    ## 2049      girlfriend  113         <NA>     <NA>    NA
    ## 2050            harm  113         fear negative    -2
    ## 2051            harm  113     negative negative    -2
    ## 2052            jail  113         fear     <NA>    NA
    ## 2053            jail  113     negative     <NA>    NA
    ## 2054            jail  113      sadness     <NA>    NA
    ## 2055             kid  113         <NA>     <NA>    NA
    ## 2056           light  113         <NA>     <NA>    NA
    ## 2057          played  113         <NA>     <NA>    NA
    ## 2058             psa  113         <NA>     <NA>    NA
    ## 2059          taking  113         <NA>     <NA>    NA
    ## 2060           toxic  113      disgust negative    NA
    ## 2061           toxic  113     negative negative    NA
    ## 2062       according  112         <NA>     <NA>    NA
    ## 2063           aware  112         <NA>     <NA>    NA
    ## 2064  discrimination  112        anger negative    NA
    ## 2065  discrimination  112      disgust negative    NA
    ## 2066  discrimination  112         fear negative    NA
    ## 2067  discrimination  112     negative negative    NA
    ## 2068  discrimination  112      sadness negative    NA
    ## 2069             get  112         <NA>     <NA>    NA
    ## 2070        handling  112         <NA>     <NA>    NA
    ## 2071         mystery  112 anticipation negative    NA
    ## 2072         mystery  112     surprise negative    NA
    ## 2073       redditors  112         <NA>     <NA>    NA
    ## 2074        research  112         <NA>     <NA>    NA
    ## 2075      researcher  112         <NA>     <NA>    NA
    ## 2076           roles  112         <NA>     <NA>    NA
    ## 2077          scared  112         <NA> negative    -2
    ## 2078          single  112         <NA>     <NA>    NA
    ## 2079      technology  112     positive     <NA>    NA
    ## 2080         villain  112         fear     <NA>    NA
    ## 2081         villain  112     negative     <NA>    NA
    ## 2082             bio  111         <NA>     <NA>    NA
    ## 2083          career  111 anticipation     <NA>    NA
    ## 2084          career  111     positive     <NA>    NA
    ## 2085           cough  111      disgust     <NA>    NA
    ## 2086           cough  111     negative     <NA>    NA
    ## 2087               d  111         <NA>     <NA>    NA
    ## 2088            fail  111         <NA> negative    -2
    ## 2089        february  111         <NA>     <NA>    NA
    ## 2090        feelings  111         <NA>     <NA>    NA
    ## 2091        finances  111         <NA>     <NA>    NA
    ## 2092            hard  111         <NA> negative    -1
    ## 2093        moderate  111     positive     <NA>    NA
    ## 2094        operator  111         <NA>     <NA>    NA
    ## 2095       residents  111         <NA>     <NA>    NA
    ## 2096         weapons  111         <NA>     <NA>    NA
    ## 2097           chill  110         <NA> negative    NA
    ## 2098          choice  110     positive     <NA>    NA
    ## 2099            grow  110 anticipation     <NA>    NA
    ## 2100            grow  110          joy     <NA>    NA
    ## 2101            grow  110     positive     <NA>    NA
    ## 2102            grow  110        trust     <NA>    NA
    ## 2103     impeachment  110     negative     <NA>    NA
    ## 2104         officer  110     positive     <NA>    NA
    ## 2105         officer  110        trust     <NA>    NA
    ## 2106   relationships  110         <NA>     <NA>    NA
    ## 2107       something  110         <NA>     <NA>    NA
    ## 2108        symptoms  110         <NA> negative    NA
    ## 2109           alice  109         <NA>     <NA>    NA
    ## 2110         animals  109         <NA>     <NA>    NA
    ## 2111        attempts  109         <NA>     <NA>    NA
    ## 2112               b  109         <NA>     <NA>    NA
    ## 2113       christmas  109         <NA>     <NA>    NA
    ## 2114            cops  109         <NA>     <NA>    NA
    ## 2115         culture  109     positive     <NA>    NA
    ## 2116         general  109     positive     <NA>    NA
    ## 2117         general  109        trust     <NA>    NA
    ## 2118           hotel  109         <NA>     <NA>    NA
    ## 2119        includes  109         <NA>     <NA>    NA
    ## 2120             may  109         <NA>     <NA>    NA
    ## 2121          places  109         <NA>     <NA>    NA
    ## 2122           seats  109         <NA>     <NA>    NA
    ## 2123           stick  109         <NA>     <NA>    NA
    ## 2124            wise  109     positive positive    NA
    ## 2125           based  108         <NA>     <NA>    NA
    ## 2126            clip  108         <NA>     <NA>    NA
    ## 2127       democracy  108     positive     <NA>    NA
    ## 2128        electing  108         <NA>     <NA>    NA
    ## 2129           happy  108 anticipation positive     3
    ## 2130           happy  108          joy positive     3
    ## 2131           happy  108     positive positive     3
    ## 2132           happy  108        trust positive     3
    ## 2133      infections  108         <NA> negative    NA
    ## 2134         johnson  108         <NA>     <NA>    NA
    ## 2135          middle  108         <NA>     <NA>    NA
    ## 2136        outbreak  108         <NA> negative    NA
    ## 2137            park  108         <NA>     <NA>    NA
    ## 2138        starring  108         <NA>     <NA>    NA
    ## 2139         talking  108         <NA>     <NA>    NA
    ## 2140           teams  108         <NA>     <NA>    NA
    ## 2141         trained  108         <NA>     <NA>    NA
    ## 2142            word  108     positive     <NA>    NA
    ## 2143            word  108        trust     <NA>    NA
    ## 2144            born  107         <NA>     <NA>    NA
    ## 2145         changed  107         <NA>     <NA>    NA
    ## 2146         dangers  107         <NA>     <NA>    NA
    ## 2147           favor  107         <NA> positive     2
    ## 2148             hat  107         <NA>     <NA>    NA
    ## 2149            hide  107         fear     <NA>    -1
    ## 2150            hmrb  107         <NA>     <NA>    NA
    ## 2151         learned  107         <NA>     <NA>    NA
    ## 2152           magic  107         <NA> positive    NA
    ## 2153            mask  107         <NA>     <NA>    NA
    ## 2154        multiple  107         <NA>     <NA>    NA
    ## 2155         replace  107         <NA>     <NA>    NA
    ## 2156        advanced  106     positive positive     1
    ## 2157          canada  106         <NA>     <NA>    NA
    ## 2158    conservative  106         <NA> negative    NA
    ## 2159            crew  106        trust     <NA>    NA
    ## 2160        employee  106         <NA>     <NA>    NA
    ## 2161              eu  106         <NA>     <NA>    NA
    ## 2162           going  106         <NA>     <NA>    NA
    ## 2163       hospitals  106         <NA>     <NA>    NA
    ## 2164           lease  106         <NA>     <NA>    NA
    ## 2165          period  106         <NA>     <NA>    NA
    ## 2166         preview  106         <NA>     <NA>    NA
    ## 2167       realistic  106         <NA> positive    NA
    ## 2168         require  106         <NA>     <NA>    NA
    ## 2169           taste  106         <NA>     <NA>    NA
    ## 2170           tries  106         <NA>     <NA>    NA
    ## 2171            uber  106         <NA>     <NA>    NA
    ## 2172          wanted  106         <NA>     <NA>    NA
    ## 2173             age  105         <NA>     <NA>    NA
    ## 2174           block  105         <NA>     <NA>    -1
    ## 2175         episode  105         <NA>     <NA>    NA
    ## 2176        everyday  105         <NA>     <NA>    NA
    ## 2177            form  105         <NA>     <NA>    NA
    ## 2178           human  105         <NA>     <NA>    NA
    ## 2179         imagine  105         <NA>     <NA>    NA
    ## 2180      incredibly  105         <NA> positive    NA
    ## 2181           mixed  105         <NA>     <NA>    NA
    ## 2182          notice  105         <NA>     <NA>    NA
    ## 2183         parties  105         <NA>     <NA>    NA
    ## 2184           place  105         <NA>     <NA>    NA
    ## 2185        realized  105         <NA>     <NA>    NA
    ## 2186           skill  105         <NA> positive    NA
    ## 2187       spielberg  105         <NA>     <NA>    NA
    ## 2188            test  105         <NA>     <NA>    NA
    ## 2189         weekend  105         <NA>     <NA>    NA
    ## 2190           weeks  105         <NA>     <NA>    NA
    ## 2191          affect  104         <NA>     <NA>    NA
    ## 2192             bus  104         <NA>     <NA>    NA
    ## 2193     individuals  104         <NA>     <NA>    NA
    ## 2194             key  104         <NA>     <NA>    NA
    ## 2195         signing  104         <NA>     <NA>    NA
    ## 2196            stop  104         <NA>     <NA>    -1
    ## 2197          allows  103         <NA>     <NA>    NA
    ## 2198           award  103 anticipation positive     3
    ## 2199           award  103          joy positive     3
    ## 2200           award  103     positive positive     3
    ## 2201           award  103     surprise positive     3
    ## 2202           award  103        trust positive     3
    ## 2203        disaster  103        anger negative    -2
    ## 2204        disaster  103      disgust negative    -2
    ## 2205        disaster  103         fear negative    -2
    ## 2206        disaster  103     negative negative    -2
    ## 2207        disaster  103      sadness negative    -2
    ## 2208        disaster  103     surprise negative    -2
    ## 2209            gary  103         <NA>     <NA>    NA
    ## 2210        hospital  103         fear     <NA>    NA
    ## 2211        hospital  103      sadness     <NA>    NA
    ## 2212        hospital  103        trust     <NA>    NA
    ## 2213           hours  103         <NA>     <NA>    NA
    ## 2214    intelligence  103         fear positive    NA
    ## 2215    intelligence  103          joy positive    NA
    ## 2216    intelligence  103     positive positive    NA
    ## 2217    intelligence  103        trust positive    NA
    ## 2218       literally  103         <NA>     <NA>    NA
    ## 2219         massive  103         <NA>     <NA>    NA
    ## 2220         measles  103      disgust     <NA>    NA
    ## 2221         measles  103         fear     <NA>    NA
    ## 2222         measles  103     negative     <NA>    NA
    ## 2223         measles  103      sadness     <NA>    NA
    ## 2224           plays  103         <NA>     <NA>    NA
    ## 2225         protest  103         <NA> negative    -2
    ## 2226          review  103         <NA>     <NA>    NA
    ## 2227          thanks  103         <NA>     <NA>     2
    ## 2228         theatre  103         <NA>     <NA>    NA
    ## 2229         world's  103         <NA>     <NA>    NA
    ## 2230      apocalypse  102         <NA> negative    NA
    ## 2231          battle  102        anger     <NA>    -1
    ## 2232          battle  102     negative     <NA>    -1
    ## 2233        deepfake  102         <NA>     <NA>    NA
    ## 2234            even  102         <NA>     <NA>    NA
    ## 2235          indeed  102         <NA>     <NA>    NA
    ## 2236          lights  102         <NA>     <NA>    NA
    ## 2237       qualified  102     positive positive    NA
    ## 2238       qualified  102        trust positive    NA
    ## 2239         reality  102         <NA>     <NA>    NA
    ## 2240        recorded  102         <NA>     <NA>    NA
    ## 2241        regional  102         <NA>     <NA>    NA
    ## 2242          saying  102         <NA>     <NA>    NA
    ## 2243           three  102         <NA>     <NA>    NA
    ## 2244            tiny  102         <NA>     <NA>    NA
    ## 2245           turns  102         <NA>     <NA>    NA
    ## 2246         cameras  101         <NA>     <NA>    NA
    ## 2247           china  101         <NA>     <NA>    NA
    ## 2248           ghost  101         fear     <NA>    -1
    ## 2249            lead  101     positive positive    NA
    ## 2250        majority  101          joy     <NA>    NA
    ## 2251        majority  101     positive     <NA>    NA
    ## 2252        majority  101        trust     <NA>    NA
    ## 2253            must  101         <NA>     <NA>    NA
    ## 2254             non  101         <NA>     <NA>    NA
    ## 2255         produce  101         <NA>     <NA>    NA
    ## 2256          rabbit  101         <NA>     <NA>    NA
    ## 2257         regular  101         <NA>     <NA>    NA
    ## 2258  representation  101         <NA>     <NA>    NA
    ## 2259           shots  101         <NA>     <NA>    NA
    ## 2260         winning  101 anticipation positive     4
    ## 2261         winning  101      disgust positive     4
    ## 2262         winning  101          joy positive     4
    ## 2263         winning  101     positive positive     4
    ## 2264         winning  101      sadness positive     4
    ## 2265         winning  101     surprise positive     4
    ## 2266         winning  101        trust positive     4
    ## 2267         applied  100         <NA>     <NA>    NA
    ## 2268          behind  100         <NA>     <NA>    NA
    ## 2269    hospitalized  100         <NA>     <NA>    NA
    ## 2270        stealing  100      disgust negative    NA
    ## 2271        stealing  100         fear negative    NA
    ## 2272        stealing  100     negative negative    NA
    ## 2273            task  100     positive     <NA>    NA
    ## 2274         whoever  100         <NA>     <NA>    NA
    ## 2275       algorithm   99         <NA>     <NA>    NA
    ## 2276      algorithms   99         <NA>     <NA>    NA
    ## 2277      completely   99     positive     <NA>    NA
    ## 2278       customers   99         <NA>     <NA>    NA
    ## 2279              de   99         <NA>     <NA>    NA
    ## 2280         decided   99         <NA>     <NA>    NA
    ## 2281          double   99         <NA>     <NA>    NA
    ## 2282            else   99         <NA>     <NA>    NA
    ## 2283        entirely   99         <NA>     <NA>    NA
    ## 2284       extremely   99         <NA>     <NA>    NA
    ## 2285          facial   99         <NA>     <NA>    NA
    ## 2286         freedom   99          joy positive     2
    ## 2287         freedom   99     positive positive     2
    ## 2288         freedom   99        trust positive     2
    ## 2289             mom   99         <NA>     <NA>    NA
    ## 2290           money   99        anger     <NA>    NA
    ## 2291           money   99 anticipation     <NA>    NA
    ## 2292           money   99          joy     <NA>    NA
    ## 2293           money   99     positive     <NA>    NA
    ## 2294           money   99     surprise     <NA>    NA
    ## 2295           money   99        trust     <NA>    NA
    ## 2296        province   99         <NA>     <NA>    NA
    ## 2297         receive   99         <NA>     <NA>    NA
    ## 2298     researchers   99         <NA>     <NA>    NA
    ## 2299            ride   99         <NA>     <NA>    NA
    ## 2300          safety   99         <NA>     <NA>     1
    ## 2301      apparently   98         <NA>     <NA>    NA
    ## 2302          forget   98     negative     <NA>    -1
    ## 2303     gatekeeping   98         <NA>     <NA>    NA
    ## 2304            hope   98 anticipation     <NA>     2
    ## 2305            hope   98          joy     <NA>     2
    ## 2306            hope   98     positive     <NA>     2
    ## 2307            hope   98     surprise     <NA>     2
    ## 2308            hope   98        trust     <NA>     2
    ## 2309            liga   98         <NA>     <NA>    NA
    ## 2310         physics   98     positive     <NA>    NA
    ## 2311           track   98 anticipation     <NA>    NA
    ## 2312       volunteer   98 anticipation     <NA>    NA
    ## 2313       volunteer   98         fear     <NA>    NA
    ## 2314       volunteer   98          joy     <NA>    NA
    ## 2315       volunteer   98     positive     <NA>    NA
    ## 2316       volunteer   98        trust     <NA>    NA
    ## 2317             way   98         <NA>     <NA>    NA
    ## 2318       computers   97         <NA>     <NA>    NA
    ## 2319            dear   97     positive     <NA>     2
    ## 2320         finance   97         <NA>     <NA>    NA
    ## 2321           games   97         <NA>     <NA>    NA
    ## 2322            high   97         <NA>     <NA>    NA
    ## 2323           point   97         <NA>     <NA>    NA
    ## 2324        romantic   97 anticipation positive    NA
    ## 2325        romantic   97          joy positive    NA
    ## 2326        romantic   97     positive positive    NA
    ## 2327        romantic   97        trust positive    NA
    ## 2328             six   97         <NA>     <NA>    NA
    ## 2329       solutions   97         <NA>     <NA>     1
    ## 2330           sonic   97         <NA>     <NA>    NA
    ## 2331           tried   97         <NA>     <NA>    NA
    ## 2332       diversity   96         <NA>     <NA>    NA
    ## 2333         exposed   96     negative     <NA>    -1
    ## 2334             fan   96         <NA>     <NA>     3
    ## 2335          inside   96         <NA>     <NA>    NA
    ## 2336          lawyer   96        anger     <NA>    NA
    ## 2337          lawyer   96      disgust     <NA>    NA
    ## 2338          lawyer   96         fear     <NA>    NA
    ## 2339          lawyer   96     negative     <NA>    NA
    ## 2340            play   96         <NA>     <NA>    NA
    ## 2341           split   96     negative negative    NA
    ## 2342            spot   96         <NA>     <NA>    NA
    ## 2343         spotted   96         <NA>     <NA>    NA
    ## 2344      absolutely   95         <NA>     <NA>    NA
    ## 2345           basic   95         <NA>     <NA>    NA
    ## 2346          closer   95         <NA>     <NA>    NA
    ## 2347        fighting   95        anger     <NA>    NA
    ## 2348        fighting   95     negative     <NA>    NA
    ## 2349             mit   95         <NA>     <NA>    NA
    ## 2350           owner   95         <NA>     <NA>    NA
    ## 2351         quality   95         <NA>     <NA>    NA
    ## 2352             tom   95         <NA>     <NA>    NA
    ## 2353           trash   95      disgust negative    NA
    ## 2354           trash   95     negative negative    NA
    ## 2355           trash   95      sadness negative    NA
    ## 2356           whats   95         <NA>     <NA>    NA
    ## 2357            bans   94         <NA>     <NA>    NA
    ## 2358        believes   94         <NA>     <NA>    NA
    ## 2359        canadian   94         <NA>     <NA>    NA
    ## 2360         chinese   94         <NA>     <NA>    NA
    ## 2361       contracts   94         <NA>     <NA>    NA
    ## 2362         dropped   94         <NA>     <NA>    NA
    ## 2363        drowning   94         <NA> negative    NA
    ## 2364       effective   94     positive positive     2
    ## 2365       effective   94        trust positive     2
    ## 2366           given   94         <NA>     <NA>    NA
    ## 2367     governments   94         <NA>     <NA>    NA
    ## 2368         husband   94         <NA>     <NA>    NA
    ## 2369      individual   94         <NA>     <NA>    NA
    ## 2370        invented   94         <NA>     <NA>    NA
    ## 2371          longer   94         <NA>     <NA>    NA
    ## 2372          market   94         <NA>     <NA>    NA
    ## 2373        negative   94     negative negative    -2
    ## 2374        negative   94      sadness negative    -2
    ## 2375            next   94         <NA>     <NA>    NA
    ## 2376        positive   94         <NA> positive     2
    ## 2377        pressure   94     negative     <NA>    -1
    ## 2378       recording   94         <NA>     <NA>    NA
    ## 2379          sounds   94         <NA>     <NA>    NA
    ## 2380          spoken   94         <NA>     <NA>    NA
    ## 2381            tell   94         <NA>     <NA>    NA
    ## 2382         trump's   94         <NA>     <NA>    NA
    ## 2383           alert   93         <NA>     <NA>    -1
    ## 2384           build   93     positive     <NA>    NA
    ## 2385        changing   93         <NA>     <NA>    NA
    ## 2386            damn   93        anger negative    -4
    ## 2387            damn   93      disgust negative    -4
    ## 2388            damn   93     negative negative    -4
    ## 2389            debt   93     negative negative    -2
    ## 2390            debt   93      sadness negative    -2
    ## 2391           gives   93         <NA>     <NA>    NA
    ## 2392          guilty   93        anger negative    -3
    ## 2393          guilty   93     negative negative    -3
    ## 2394          guilty   93      sadness negative    -3
    ## 2395            hell   93        anger negative    -4
    ## 2396            hell   93      disgust negative    -4
    ## 2397            hell   93         fear negative    -4
    ## 2398            hell   93     negative negative    -4
    ## 2399            hell   93      sadness negative    -4
    ## 2400            home   93         <NA>     <NA>    NA
    ## 2401           nobel   93         <NA>     <NA>    NA
    ## 2402         reasons   93         <NA>     <NA>    NA
    ## 2403            rock   93     positive     <NA>    NA
    ## 2404           works   93         <NA> positive    NA
    ## 2405         worried   93     negative negative    -3
    ## 2406         worried   93      sadness negative    -3
    ## 2407         another   92         <NA>     <NA>    NA
    ## 2408             bid   92         <NA>     <NA>    NA
    ## 2409       brazilian   92         <NA>     <NA>    NA
    ## 2410           cause   92         <NA>     <NA>    NA
    ## 2411             die   92         fear negative    -3
    ## 2412             die   92     negative negative    -3
    ## 2413             die   92      sadness negative    -3
    ## 2414         dressed   92         <NA>     <NA>    NA
    ## 2415         earlier   92         <NA>     <NA>    NA
    ## 2416           event   92         <NA>     <NA>    NA
    ## 2417       excellent   92          joy positive     3
    ## 2418       excellent   92     positive positive     3
    ## 2419       excellent   92        trust positive     3
    ## 2420         finding   92         <NA>     <NA>    NA
    ## 2421             kit   92         <NA>     <NA>    NA
    ## 2422       published   92         <NA>     <NA>    NA
    ## 2423             rip   92         <NA> negative    NA
    ## 2424           songs   92         <NA>     <NA>    NA
    ## 2425           towns   92         <NA>     <NA>    NA
    ## 2426         writing   92         <NA>     <NA>    NA
    ## 2427             ysk   92         <NA>     <NA>    NA
    ## 2428          argues   91         <NA>     <NA>    NA
    ## 2429           brown   91         <NA>     <NA>    NA
    ## 2430          corona   91         <NA>     <NA>    NA
    ## 2431           crazy   91        anger negative    -2
    ## 2432           crazy   91         fear negative    -2
    ## 2433           crazy   91     negative negative    -2
    ## 2434           crazy   91      sadness negative    -2
    ## 2435          create   91          joy     <NA>    NA
    ## 2436          create   91     positive     <NA>    NA
    ## 2437          demand   91        anger     <NA>    -1
    ## 2438          demand   91     negative     <NA>    -1
    ## 2439         enemies   91         <NA> negative    -2
    ## 2440       existence   91     positive     <NA>    NA
    ## 2441         florida   91         <NA>     <NA>    NA
    ## 2442            huge   91         <NA>     <NA>     1
    ## 2443     immediately   91 anticipation     <NA>    NA
    ## 2444     immediately   91     negative     <NA>    NA
    ## 2445     immediately   91     positive     <NA>    NA
    ## 2446       important   91     positive positive     2
    ## 2447       important   91        trust positive     2
    ## 2448         michael   91         <NA>     <NA>    NA
    ## 2449     politicians   91         <NA>     <NA>    NA
    ## 2450     recognition   91         <NA>     <NA>    NA
    ## 2451         someone   91         <NA>     <NA>    NA
    ## 2452           tests   91         <NA>     <NA>    NA
    ## 2453            true   91          joy     <NA>     2
    ## 2454            true   91     positive     <NA>     2
    ## 2455            true   91        trust     <NA>     2
    ## 2456           broke   90         fear negative    -1
    ## 2457           broke   90     negative negative    -1
    ## 2458           broke   90      sadness negative    -1
    ## 2459       committee   90        trust     <NA>    NA
    ## 2460         defense   90        anger     <NA>    NA
    ## 2461         defense   90 anticipation     <NA>    NA
    ## 2462         defense   90         fear     <NA>    NA
    ## 2463         defense   90     positive     <NA>    NA
    ## 2464              et   90         <NA>     <NA>    NA
    ## 2465          events   90         <NA>     <NA>    NA
    ## 2466      explaining   90         <NA>     <NA>    NA
    ## 2467           fever   90         fear negative    NA
    ## 2468            hand   90         <NA>     <NA>    NA
    ## 2469             job   90     positive     <NA>    NA
    ## 2470          killed   90         <NA> negative    -3
    ## 2471            like   90         <NA> positive     2
    ## 2472            raid   90        anger     <NA>    NA
    ## 2473            raid   90         fear     <NA>    NA
    ## 2474            raid   90     negative     <NA>    NA
    ## 2475            raid   90     surprise     <NA>    NA
    ## 2476           ready   90 anticipation positive    NA
    ## 2477      university   90 anticipation     <NA>    NA
    ## 2478      university   90     positive     <NA>    NA
    ## 2479            auto   89         <NA>     <NA>    NA
    ## 2480         average   89         <NA>     <NA>    NA
    ## 2481            baby   89          joy     <NA>    NA
    ## 2482            baby   89     positive     <NA>    NA
    ## 2483      capitalism   89         <NA>     <NA>    NA
    ## 2484           class   89         <NA>     <NA>    NA
    ## 2485         ethical   89     positive positive     2
    ## 2486           found   89          joy     <NA>    NA
    ## 2487           found   89     positive     <NA>    NA
    ## 2488           found   89        trust     <NA>    NA
    ## 2489           gonna   89         <NA>     <NA>    NA
    ## 2490            good   89 anticipation positive     3
    ## 2491            good   89          joy positive     3
    ## 2492            good   89     positive positive     3
    ## 2493            good   89     surprise positive     3
    ## 2494            good   89        trust positive     3
    ## 2495          google   89         <NA>     <NA>    NA
    ## 2496          labour   89         <NA>     <NA>    NA
    ## 2497           local   89         <NA>     <NA>    NA
    ## 2498         problem   89         fear negative    -2
    ## 2499         problem   89     negative negative    -2
    ## 2500         problem   89      sadness negative    -2
    ## 2501             psg   89         <NA>     <NA>    NA
    ## 2502          remove   89        anger     <NA>    NA
    ## 2503          remove   89         fear     <NA>    NA
    ## 2504          remove   89     negative     <NA>    NA
    ## 2505          remove   89      sadness     <NA>    NA
    ## 2506            riot   89        anger     <NA>    -2
    ## 2507            riot   89         fear     <NA>    -2
    ## 2508            riot   89     negative     <NA>    -2
    ## 2509          salary   89 anticipation     <NA>    NA
    ## 2510          salary   89          joy     <NA>    NA
    ## 2511          salary   89     positive     <NA>    NA
    ## 2512          salary   89        trust     <NA>    NA
    ## 2513             san   89         <NA>     <NA>    NA
    ## 2514       statement   89     positive     <NA>    NA
    ## 2515       statement   89        trust     <NA>    NA
    ## 2516           value   89         <NA>     <NA>    NA
    ## 2517           video   89         <NA>     <NA>    NA
    ## 2518           worse   89         fear negative    -3
    ## 2519           worse   89     negative negative    -3
    ## 2520           worse   89      sadness negative    -3
    ## 2521           y'all   89         <NA>     <NA>    NA
    ## 2522     accusations   88         <NA> negative    -2
    ## 2523             aka   88         <NA>     <NA>    NA
    ## 2524          called   88         <NA>     <NA>    NA
    ## 2525            clue   88 anticipation     <NA>    NA
    ## 2526             cmv   88         <NA>     <NA>    NA
    ## 2527       explained   88         <NA>     <NA>    NA
    ## 2528          global   88         <NA>     <NA>    NA
    ## 2529       halloween   88         <NA>     <NA>    NA
    ## 2530             hmc   88         <NA>     <NA>    NA
    ## 2531           indie   88         <NA>     <NA>    NA
    ## 2532           leave   88     negative     <NA>    -1
    ## 2533           leave   88      sadness     <NA>    -1
    ## 2534           leave   88     surprise     <NA>    -1
    ## 2535          little   88         <NA>     <NA>    NA
    ## 2536        minority   88     negative     <NA>    NA
    ## 2537              nj   88         <NA>     <NA>    NA
    ## 2538          online   88         <NA>     <NA>    NA
    ## 2539          polish   88     positive     <NA>    NA
    ## 2540           radio   88     positive     <NA>    NA
    ## 2541          scream   88        anger negative    -2
    ## 2542          scream   88      disgust negative    -2
    ## 2543          scream   88         fear negative    -2
    ## 2544          scream   88     negative negative    -2
    ## 2545          scream   88     surprise negative    -2
    ## 2546         somehow   88         <NA>     <NA>    NA
    ## 2547           spain   88         <NA>     <NA>    NA
    ## 2548       structure   88     positive     <NA>    NA
    ## 2549       structure   88        trust     <NA>    NA
    ## 2550          turkey   88         <NA>     <NA>    NA
    ## 2551         without   88         <NA>     <NA>    NA
    ## 2552       automated   87         <NA>     <NA>    NA
    ## 2553       conscious   87         <NA>     <NA>    NA
    ## 2554      developers   87         <NA>     <NA>    NA
    ## 2555         doctors   87         <NA>     <NA>    NA
    ## 2556          forced   87         fear     <NA>    -1
    ## 2557          forced   87     negative     <NA>    -1
    ## 2558         january   87         <NA>     <NA>    NA
    ## 2559             leg   87         <NA>     <NA>    NA
    ## 2560            read   87         <NA>     <NA>    NA
    ## 2561         respect   87 anticipation positive    NA
    ## 2562         respect   87          joy positive    NA
    ## 2563         respect   87     positive positive    NA
    ## 2564         respect   87        trust positive    NA
    ## 2565           serie   87         <NA>     <NA>    NA
    ## 2566          social   87         <NA>     <NA>    NA
    ## 2567          trashy   87      disgust negative    NA
    ## 2568          trashy   87     negative negative    NA
    ## 2569      washington   87         <NA>     <NA>    NA
    ## 2570         already   86         <NA>     <NA>    NA
    ## 2571             bag   86         <NA>     <NA>    NA
    ## 2572          defend   86         fear     <NA>    NA
    ## 2573          defend   86     positive     <NA>    NA
    ## 2574          direct   86         <NA>     <NA>    NA
    ## 2575       following   86         <NA>     <NA>    NA
    ## 2576            kids   86         <NA>     <NA>    NA
    ## 2577          nature   86         <NA>     <NA>    NA
    ## 2578        proposes   86         <NA>     <NA>    NA
    ## 2579       purchased   86         <NA>     <NA>    NA
    ## 2580       secretary   86         <NA>     <NA>    NA
    ## 2581           signs   86         <NA>     <NA>    NA
    ## 2582          simple   86         <NA>     <NA>    NA
    ## 2583          soccer   86         <NA>     <NA>    NA
    ## 2584         spreads   86         <NA>     <NA>    NA
    ## 2585      successful   86 anticipation positive     3
    ## 2586      successful   86          joy positive     3
    ## 2587      successful   86     positive positive     3
    ## 2588      successful   86        trust positive     3
    ## 2589     traditional   86     positive     <NA>    NA
    ## 2590             use   86         <NA>     <NA>    NA
    ## 2591           whose   86         <NA>     <NA>    NA
    ## 2592               c   85         <NA>     <NA>    NA
    ## 2593           chief   85         <NA>     <NA>    NA
    ## 2594         created   85         <NA>     <NA>    NA
    ## 2595            fire   85         fear     <NA>    -2
    ## 2596           gates   85         <NA>     <NA>    NA
    ## 2597         helping   85         <NA> positive     2
    ## 2598          liking   85          joy positive    NA
    ## 2599          liking   85     positive positive    NA
    ## 2600          liking   85        trust positive    NA
    ## 2601  misinformation   85         <NA>     <NA>    -2
    ## 2602         network   85 anticipation     <NA>    NA
    ## 2603        physical   85         <NA>     <NA>    NA
    ## 2604          poorly   85     negative negative    NA
    ## 2605         protect   85     positive positive     1
    ## 2606         realize   85         <NA>     <NA>    NA
    ## 2607           saved   85         <NA>     <NA>     2
    ## 2608         version   85         <NA>     <NA>    NA
    ## 2609        anything   84         <NA>     <NA>    NA
    ## 2610         brother   84     positive     <NA>    NA
    ## 2611         brother   84        trust     <NA>    NA
    ## 2612            cars   84         <NA>     <NA>    NA
    ## 2613         central   84         <NA>     <NA>    NA
    ## 2614           death   84        anger negative    -2
    ## 2615           death   84 anticipation negative    -2
    ## 2616           death   84      disgust negative    -2
    ## 2617           death   84         fear negative    -2
    ## 2618           death   84     negative negative    -2
    ## 2619           death   84      sadness negative    -2
    ## 2620           death   84     surprise negative    -2
    ## 2621           dying   84        anger negative    NA
    ## 2622           dying   84      disgust negative    NA
    ## 2623           dying   84         fear negative    NA
    ## 2624           dying   84     negative negative    NA
    ## 2625           dying   84      sadness negative    NA
    ## 2626           earth   84         <NA>     <NA>    NA
    ## 2627           fraud   84        anger negative    -4
    ## 2628           fraud   84     negative negative    -4
    ## 2629               j   84         <NA>     <NA>    NA
    ## 2630            lose   84        anger negative    NA
    ## 2631            lose   84      disgust negative    NA
    ## 2632            lose   84         fear negative    NA
    ## 2633            lose   84     negative negative    NA
    ## 2634            lose   84      sadness negative    NA
    ## 2635            lose   84     surprise negative    NA
    ## 2636          mother   84 anticipation     <NA>    NA
    ## 2637          mother   84          joy     <NA>    NA
    ## 2638          mother   84     negative     <NA>    NA
    ## 2639          mother   84     positive     <NA>    NA
    ## 2640          mother   84      sadness     <NA>    NA
    ## 2641          mother   84        trust     <NA>    NA
    ## 2642     progressive   84     positive positive    NA
    ## 2643           using   84         <NA>     <NA>    NA
    ## 2644          agenda   83         <NA>     <NA>    NA
    ## 2645          design   83         <NA>     <NA>    NA
    ## 2646          dreams   83         <NA>     <NA>     1
    ## 2647           fight   83        anger     <NA>    -1
    ## 2648           fight   83         fear     <NA>    -1
    ## 2649           fight   83     negative     <NA>    -1
    ## 2650         healthy   83     positive positive     2
    ## 2651         holiday   83 anticipation     <NA>    NA
    ## 2652         holiday   83          joy     <NA>    NA
    ## 2653         holiday   83     positive     <NA>    NA
    ## 2654       infantino   83         <NA>     <NA>    NA
    ## 2655          likely   83         <NA>     <NA>    NA
    ## 2656           media   83         <NA>     <NA>    NA
    ## 2657         nothing   83         <NA>     <NA>    NA
    ## 2658           purge   83         fear     <NA>    NA
    ## 2659           purge   83     negative     <NA>    NA
    ## 2660          rather   83         <NA>     <NA>    NA
    ## 2661            soon   83         <NA>     <NA>    NA
    ## 2662           theme   83         <NA>     <NA>    NA
    ## 2663       condition   82         <NA>     <NA>    NA
    ## 2664            cube   82        trust     <NA>    NA
    ## 2665           david   82         <NA>     <NA>    NA
    ## 2666    disappointed   82        anger negative    -2
    ## 2667    disappointed   82      disgust negative    -2
    ## 2668    disappointed   82     negative negative    -2
    ## 2669    disappointed   82      sadness negative    -2
    ## 2670              dz   82         <NA>     <NA>    NA
    ## 2671             fun   82 anticipation positive     4
    ## 2672             fun   82          joy positive     4
    ## 2673             fun   82     positive positive     4
    ## 2674           jesus   82         <NA>     <NA>     1
    ## 2675           major   82     positive     <NA>    NA
    ## 2676           maker   82         <NA>     <NA>    NA
    ## 2677          manage   82     positive     <NA>    NA
    ## 2678          manage   82        trust     <NA>    NA
    ## 2679        purchase   82         <NA>     <NA>    NA
    ## 2680        recently   82         <NA>     <NA>    NA
    ## 2681       suspected   82         <NA>     <NA>    -1
    ## 2682          tricks   82         <NA>     <NA>    NA
    ## 2683           worry   82 anticipation negative    -3
    ## 2684           worry   82         fear negative    -3
    ## 2685           worry   82     negative negative    -3
    ## 2686           worry   82      sadness negative    -3
    ## 2687           write   82         <NA>     <NA>    NA
    ## 2688         account   81        trust     <NA>    NA
    ## 2689           admit   81         <NA>     <NA>    -1
    ## 2690           along   81         <NA>     <NA>    NA
    ## 2691         awesome   81         <NA> positive     4
    ## 2692             big   81         <NA>     <NA>     1
    ## 2693          breaks   81         <NA> negative    NA
    ## 2694         china's   81         <NA>     <NA>    NA
    ## 2695     criticizing   81         <NA> negative    -2
    ## 2696        fighters   81         <NA>     <NA>    NA
    ## 2697              ft   81         <NA>     <NA>    NA
    ## 2698         grenade   81         fear     <NA>    NA
    ## 2699         grenade   81     negative     <NA>    NA
    ## 2700            hire   81 anticipation     <NA>    NA
    ## 2701            hire   81          joy     <NA>    NA
    ## 2702            hire   81     positive     <NA>    NA
    ## 2703            hire   81        trust     <NA>    NA
    ## 2704            hype   81 anticipation negative    NA
    ## 2705            hype   81     negative negative    NA
    ## 2706            jump   81          joy     <NA>    NA
    ## 2707            jump   81     positive     <NA>    NA
    ## 2708       knowledge   81     positive     <NA>    NA
    ## 2709           paris   81         <NA>     <NA>    NA
    ## 2710    professional   81     positive     <NA>    NA
    ## 2711    professional   81        trust     <NA>    NA
    ## 2712            sign   81         <NA>     <NA>    NA
    ## 2713        standard   81         <NA>     <NA>    NA
    ## 2714           tower   81     positive     <NA>    NA
    ## 2715          action   80     positive     <NA>    NA
    ## 2716           agree   80     positive     <NA>     1
    ## 2717           color   80         <NA>     <NA>    NA
    ## 2718          coming   80 anticipation     <NA>    NA
    ## 2719              ex   80         <NA>     <NA>    NA
    ## 2720             fly   80         <NA>     <NA>    NA
    ## 2721            joke   80     negative negative     2
    ## 2722        medicare   80         <NA>     <NA>    NA
    ## 2723              op   80         <NA>     <NA>    NA
    ## 2724           panic   80         fear negative    -3
    ## 2725           panic   80     negative negative    -3
    ## 2726        products   80         <NA>     <NA>    NA
    ## 2727          return   80         <NA>     <NA>    NA
    ## 2728         stories   80         <NA>     <NA>    NA
    ## 2729       treatment   80         <NA>     <NA>    NA
    ## 2730              uk   80         <NA>     <NA>    NA
    ## 2731              un   80         <NA>     <NA>    NA
    ## 2732             van   80         <NA>     <NA>    NA
    ## 2733           waste   80      disgust negative    -1
    ## 2734           waste   80     negative negative    -1
    ## 2735          winter   80         <NA>     <NA>    NA
    ## 2736             air   79         <NA>     <NA>    NA
    ## 2737             amp   79         <NA>     <NA>    NA
    ## 2738           aside   79         <NA>     <NA>    NA
    ## 2739        generate   79         <NA>     <NA>    NA
    ## 2740           heart   79         <NA>     <NA>    NA
    ## 2741             hiv   79         <NA>     <NA>    NA
    ## 2742           holes   79         <NA>     <NA>    NA
    ## 2743       investing   79         <NA>     <NA>    NA
    ## 2744         lawsuit   79        anger     <NA>    -2
    ## 2745         lawsuit   79      disgust     <NA>    -2
    ## 2746         lawsuit   79         fear     <NA>    -2
    ## 2747         lawsuit   79     negative     <NA>    -2
    ## 2748         lawsuit   79      sadness     <NA>    -2
    ## 2749         lawsuit   79     surprise     <NA>    -2
    ## 2750            list   79         <NA>     <NA>    NA
    ## 2751           loves   79         <NA> positive    NA
    ## 2752            need   79         <NA>     <NA>    NA
    ## 2753           newly   79         <NA>     <NA>    NA
    ## 2754       pneumonia   79         fear     <NA>    NA
    ## 2755       pneumonia   79     negative     <NA>    NA
    ## 2756          points   79         <NA>     <NA>    NA
    ## 2757            rent   79         <NA>     <NA>    NA
    ## 2758        romanian   79         <NA>     <NA>    NA
    ## 2759         trilogy   79         <NA>     <NA>    NA
    ## 2760          tweets   79         <NA>     <NA>    NA
    ## 2761          visual   79         <NA>     <NA>    NA
    ## 2762          amazon   78         <NA>     <NA>    NA
    ## 2763            book   78         <NA>     <NA>    NA
    ## 2764            code   78         <NA>     <NA>    NA
    ## 2765            dogs   78         <NA>     <NA>    NA
    ## 2766        features   78         <NA>     <NA>    NA
    ## 2767         glasses   78         <NA>     <NA>    NA
    ## 2768            gone   78         <NA>     <NA>    NA
    ## 2769           guess   78     surprise     <NA>    NA
    ## 2770          hoping   78         <NA>     <NA>     2
    ## 2771        infinity   78 anticipation     <NA>    NA
    ## 2772        infinity   78          joy     <NA>    NA
    ## 2773        infinity   78     positive     <NA>    NA
    ## 2774        infinity   78        trust     <NA>    NA
    ## 2775           parma   78         <NA>     <NA>    NA
    ## 2776           peace   78 anticipation positive     2
    ## 2777           peace   78          joy positive     2
    ## 2778           peace   78     positive positive     2
    ## 2779           peace   78        trust positive     2
    ## 2780        replaced   78         <NA>     <NA>    NA
    ## 2781          saving   78         <NA>     <NA>    NA
    ## 2782        secretly   78         <NA>     <NA>    NA
    ## 2783             set   78         <NA>     <NA>    NA
    ## 2784            sure   78         <NA>     <NA>    NA
    ## 2785         survive   78     positive     <NA>    NA
    ## 2786             tim   78         <NA>     <NA>    NA
    ## 2787           train   78         <NA>     <NA>    NA
    ## 2788         virtual   78         <NA>     <NA>    NA
    ## 2789           virus   78     negative negative    NA
    ## 2790          anyone   77         <NA>     <NA>    NA
    ## 2791       celebrate   77         <NA> positive     3
    ## 2792     coronavirus   77         <NA>     <NA>    NA
    ## 2793            cuts   77         <NA>     <NA>    -1
    ## 2794          desert   77        anger negative    NA
    ## 2795          desert   77      disgust negative    NA
    ## 2796          desert   77         fear negative    NA
    ## 2797          desert   77     negative negative    NA
    ## 2798          desert   77      sadness negative    NA
    ## 2799          detect   77     positive     <NA>    NA
    ## 2800         edition   77 anticipation     <NA>    NA
    ## 2801           exist   77         <NA>     <NA>    NA
    ## 2802             ice   77         <NA>     <NA>    NA
    ## 2803            matt   77         <NA>     <NA>    NA
    ## 2804           moves   77         <NA>     <NA>    NA
    ## 2805          paying   77         <NA>     <NA>    NA
    ## 2806            runs   77         <NA>     <NA>    NA
    ## 2807           score   77 anticipation     <NA>    NA
    ## 2808           score   77          joy     <NA>    NA
    ## 2809           score   77     positive     <NA>    NA
    ## 2810           score   77     surprise     <NA>    NA
    ## 2811           stole   77         <NA> negative    NA
    ## 2812        surgical   77         <NA>     <NA>    NA
    ## 2813     underground   77         <NA>     <NA>    NA
    ## 2814             via   77         <NA>     <NA>    NA
    ## 2815           ain't   76         <NA>     <NA>    NA
    ## 2816             art   76 anticipation     <NA>    NA
    ## 2817             art   76          joy     <NA>    NA
    ## 2818             art   76     positive     <NA>    NA
    ## 2819             art   76      sadness     <NA>    NA
    ## 2820             art   76     surprise     <NA>    NA
    ## 2821          asleep   76         <NA>     <NA>    NA
    ## 2822           close   76         <NA>     <NA>    NA
    ## 2823          degree   76     positive     <NA>    NA
    ## 2824            idea   76         <NA>     <NA>    NA
    ## 2825              im   76         <NA>     <NA>    NA
    ## 2826            know   76         <NA>     <NA>    NA
    ## 2827           level   76     positive     <NA>    NA
    ## 2828           level   76        trust     <NA>    NA
    ## 2829      population   76     positive     <NA>    NA
    ## 2830             pro   76         <NA>     <NA>    NA
    ## 2831          shadow   76         <NA>     <NA>    NA
    ## 2832         sitting   76         <NA>     <NA>    NA
    ## 2833          starts   76         <NA>     <NA>    NA
    ## 2834          strike   76        anger negative    -1
    ## 2835          strike   76     negative negative    -1
    ## 2836      supporting   76     positive positive     1
    ## 2837      supporting   76        trust positive     1
    ## 2838            west   76         <NA>     <NA>    NA
    ## 2839           whole   76         <NA>     <NA>    NA
    ## 2840      considered   75         <NA>     <NA>    NA
    ## 2841          figure   75         <NA>     <NA>    NA
    ## 2842            guns   75         <NA>     <NA>    NA
    ## 2843        interest   75     positive     <NA>     1
    ## 2844        position   75         <NA>     <NA>    NA
    ## 2845        probably   75         <NA>     <NA>    NA
    ## 2846          quotes   75         <NA>     <NA>    NA
    ## 2847           stunt   75         <NA> negative    NA
    ## 2848         testing   75         <NA>     <NA>    NA
    ## 2849          tinder   75         <NA>     <NA>    NA
    ## 2850        vaccines   75         <NA>     <NA>    NA
    ## 2851         western   75         <NA>     <NA>    NA
    ## 2852         willing   75         <NA> positive    NA
    ## 2853            wind   75         <NA>     <NA>    NA
    ## 2854            work   75         <NA> positive    NA
    ## 2855          agreed   74     positive     <NA>     1
    ## 2856          agreed   74        trust     <NA>     1
    ## 2857            care   74         <NA>     <NA>     2
    ## 2858          course   74         <NA>     <NA>    NA
    ## 2859           freak   74         <NA> negative    NA
    ## 2860         looking   74         <NA>     <NA>    NA
    ## 2861          pretty   74 anticipation positive     1
    ## 2862          pretty   74          joy positive     1
    ## 2863          pretty   74     positive positive     1
    ## 2864          pretty   74        trust positive     1
    ## 2865            rage   74        anger negative    -2
    ## 2866            rage   74     negative negative    -2
    ## 2867            send   74         <NA>     <NA>    NA
    ## 2868            soul   74         <NA>     <NA>    NA
    ## 2869           speed   74         <NA>     <NA>    NA
    ## 2870           steps   74         <NA>     <NA>    NA
    ## 2871         student   74         <NA>     <NA>    NA
    ## 2872           texas   74         <NA>     <NA>    NA
    ## 2873         develop   73 anticipation     <NA>    NA
    ## 2874         develop   73     positive     <NA>    NA
    ## 2875         digital   73         <NA>     <NA>    NA
    ## 2876            fits   73        anger     <NA>    NA
    ## 2877            fits   73     negative     <NA>    NA
    ## 2878         houston   73         <NA>     <NA>    NA
    ## 2879          humans   73         <NA>     <NA>    NA
    ## 2880         italian   73         <NA>     <NA>    NA
    ## 2881           kevin   73         <NA>     <NA>    NA
    ## 2882          lyrics   73         <NA>     <NA>    NA
    ## 2883             mid   73         <NA>     <NA>    NA
    ## 2884              ok   73         <NA>     <NA>    NA
    ## 2885             sea   73     positive     <NA>    NA
    ## 2886          spider   73      disgust     <NA>    NA
    ## 2887          spider   73         fear     <NA>    NA
    ## 2888         america   72         <NA>     <NA>    NA
    ## 2889           beach   72          joy     <NA>    NA
    ## 2890         certain   72         <NA>     <NA>     1
    ## 2891        entitled   72         <NA>     <NA>     1
    ## 2892         managed   72         <NA>     <NA>    NA
    ## 2893      management   72     positive     <NA>    NA
    ## 2894      management   72        trust     <NA>    NA
    ## 2895        northern   72         <NA>     <NA>    NA
    ## 2896        pandemic   72         fear     <NA>    NA
    ## 2897        pandemic   72     negative     <NA>    NA
    ## 2898        pandemic   72      sadness     <NA>    NA
    ## 2899          patent   72     positive     <NA>    NA
    ## 2900      politician   72         <NA>     <NA>    NA
    ## 2901       questions   72         <NA>     <NA>    NA
    ## 2902          reduce   72         <NA>     <NA>    NA
    ## 2903           rogue   72      disgust negative    NA
    ## 2904           rogue   72     negative negative    NA
    ## 2905        sporting   72         <NA>     <NA>    NA
    ## 2906        transfer   72         <NA>     <NA>    NA
    ## 2907            turn   72         <NA>     <NA>    NA
    ## 2908        universe   72         <NA>     <NA>    NA
    ## 2909         whether   72         <NA>     <NA>    NA
    ## 2910     apocalyptic   71         <NA> negative    -2
    ## 2911        approach   71         <NA>     <NA>    NA
    ## 2912            bots   71         <NA>     <NA>    NA
    ## 2913           brave   71         <NA> positive     2
    ## 2914         chapter   71         <NA>     <NA>    NA
    ## 2915            cool   71     positive positive     1
    ## 2916     development   71         <NA>     <NA>    NA
    ## 2917      discovered   71         <NA>     <NA>    NA
    ## 2918           dream   71         <NA>     <NA>     1
    ## 2919          dustin   71         <NA>     <NA>    NA
    ## 2920           elite   71 anticipation positive    NA
    ## 2921           elite   71          joy positive    NA
    ## 2922           elite   71     positive positive    NA
    ## 2923           elite   71        trust positive    NA
    ## 2924            hulu   71         <NA>     <NA>    NA
    ## 2925           jason   71         <NA>     <NA>    NA
    ## 2926              le   71         <NA>     <NA>    NA
    ## 2927        personal   71        trust     <NA>    NA
    ## 2928         players   71         <NA>     <NA>    NA
    ## 2929            risk   71 anticipation negative    -2
    ## 2930            risk   71         fear negative    -2
    ## 2931            risk   71     negative negative    -2
    ## 2932        scotland   71         <NA>     <NA>    NA
    ## 2933          sexual   71         <NA>     <NA>    NA
    ## 2934         spoiler   71     negative     <NA>    NA
    ## 2935         spoiler   71      sadness     <NA>    NA
    ## 2936         tuesday   71         <NA>     <NA>    NA
    ## 2937            wall   71         <NA>     <NA>    NA
    ## 2938            well   71         <NA> positive    NA
    ## 2939          castle   70         <NA>     <NA>    NA
    ## 2940             cop   70         fear     <NA>    NA
    ## 2941             cop   70        trust     <NA>    NA
    ## 2942           cover   70        trust     <NA>    NA
    ## 2943            drop   70         <NA>     <NA>    -1
    ## 2944          europe   70         <NA>     <NA>    NA
    ## 2945            find   70         <NA>     <NA>    NA
    ## 2946           fires   70         <NA>     <NA>    NA
    ## 2947          french   70         <NA>     <NA>    NA
    ## 2948           night   70         <NA>     <NA>    NA
    ## 2949       recognize   70         <NA>     <NA>    NA
    ## 2950           space   70         <NA>     <NA>    NA
    ## 2951             ten   70         <NA>     <NA>    NA
    ## 2952          andrew   69         <NA>     <NA>    NA
    ## 2953             can   69         <NA>     <NA>    NA
    ## 2954             cgi   69         <NA>     <NA>    NA
    ## 2955            chip   69         <NA>     <NA>    NA
    ## 2956    civilization   69     positive     <NA>    NA
    ## 2957    civilization   69        trust     <NA>    NA
    ## 2958        comments   69         <NA>     <NA>    NA
    ## 2959        computer   69         <NA>     <NA>    NA
    ## 2960          dating   69         <NA>     <NA>    NA
    ## 2961          decent   69     positive positive    NA
    ## 2962        exciting   69 anticipation positive     3
    ## 2963        exciting   69          joy positive     3
    ## 2964        exciting   69     positive positive     3
    ## 2965        exciting   69     surprise positive     3
    ## 2966      interested   69      disgust     <NA>     2
    ## 2967      interested   69     positive     <NA>     2
    ## 2968      interested   69      sadness     <NA>     2
    ## 2969      investment   69         <NA>     <NA>    NA
    ## 2970          iphone   69         <NA>     <NA>    NA
    ## 2971           italy   69         <NA>     <NA>    NA
    ## 2972            menu   69         <NA>     <NA>    NA
    ## 2973          modern   69         <NA> positive    NA
    ## 2974          parent   69         <NA>     <NA>    NA
    ## 2975           plans   69         <NA>     <NA>    NA
    ## 2976            size   69         <NA>     <NA>    NA
    ## 2977           stand   69         <NA>     <NA>    NA
    ## 2978           viral   69         <NA>     <NA>    NA
    ## 2979            wait   69 anticipation     <NA>    NA
    ## 2980            wait   69     negative     <NA>    NA
    ## 2981           youth   69        anger     <NA>    NA
    ## 2982           youth   69 anticipation     <NA>    NA
    ## 2983           youth   69         fear     <NA>    NA
    ## 2984           youth   69          joy     <NA>    NA
    ## 2985           youth   69     positive     <NA>    NA
    ## 2986           youth   69     surprise     <NA>    NA
    ## 2987           argue   68        anger     <NA>    NA
    ## 2988           argue   68     negative     <NA>    NA
    ## 2989          around   68         <NA>     <NA>    NA
    ## 2990        counties   68         <NA>     <NA>    NA
    ## 2991      definitely   68         <NA>     <NA>    NA
    ## 2992        epidemic   68        anger negative    NA
    ## 2993        epidemic   68 anticipation negative    NA
    ## 2994        epidemic   68      disgust negative    NA
    ## 2995        epidemic   68         fear negative    NA
    ## 2996        epidemic   68     negative negative    NA
    ## 2997        epidemic   68      sadness negative    NA
    ## 2998        epidemic   68     surprise negative    NA
    ## 2999          expect   68 anticipation     <NA>    NA
    ## 3000          expect   68     positive     <NA>    NA
    ## 3001          expect   68     surprise     <NA>    NA
    ## 3002          expect   68        trust     <NA>    NA
    ## 3003        freakout   68         <NA>     <NA>    NA
    ## 3004             joe   68         <NA>     <NA>    NA
    ## 3005           keeps   68         <NA>     <NA>    NA
    ## 3006           kills   68         <NA> negative    -3
    ## 3007           prize   68         <NA> positive    NA
    ## 3008             pve   68         <NA>     <NA>    NA
    ## 3009          school   68        trust     <NA>    NA
    ## 3010         society   68         <NA>     <NA>    NA
    ## 3011          spread   68         <NA>     <NA>    NA
    ## 3012          yang's   68         <NA>     <NA>    NA
    ## 3013        checking   67         <NA>     <NA>    NA
    ## 3014         creates   67         <NA>     <NA>    NA
    ## 3015      eventually   67         <NA>     <NA>    NA
    ## 3016           homes   67         <NA>     <NA>    NA
    ## 3017           index   67         <NA>     <NA>    NA
    ## 3018          league   67     positive     <NA>    NA
    ## 3019           lower   67     negative     <NA>    NA
    ## 3020           lower   67      sadness     <NA>    NA
    ## 3021            male   67         <NA>     <NA>    NA
    ## 3022       microsoft   67         <NA>     <NA>    NA
    ## 3023        november   67         <NA>     <NA>    NA
    ## 3024         powered   67         <NA>     <NA>    NA
    ## 3025        premiere   67         <NA>     <NA>    NA
    ## 3026         process   67         <NA>     <NA>    NA
    ## 3027         removed   67         <NA>     <NA>    NA
    ## 3028       rivalries   67         <NA>     <NA>    NA
    ## 3029          simply   67         <NA>     <NA>    NA
    ## 3030       technique   67         <NA>     <NA>    NA
    ## 3031         belongs   66         <NA>     <NA>    NA
    ## 3032         benefit   66     positive positive     2
    ## 3033           biden   66         <NA>     <NA>    NA
    ## 3034           check   66         <NA>     <NA>    NA
    ## 3035             con   66         <NA>     <NA>    NA
    ## 3036          creepy   66         <NA> negative    NA
    ## 3037          decide   66         <NA>     <NA>    NA
    ## 3038            edge   66         <NA>     <NA>    NA
    ## 3039            eric   66         <NA>     <NA>    NA
    ## 3040            gang   66        anger     <NA>    NA
    ## 3041            gang   66         fear     <NA>    NA
    ## 3042            gang   66     negative     <NA>    NA
    ## 3043        horrible   66        anger negative    -3
    ## 3044        horrible   66      disgust negative    -3
    ## 3045        horrible   66         fear negative    -3
    ## 3046        horrible   66     negative negative    -3
    ## 3047       instagram   66         <NA>     <NA>    NA
    ## 3048         instead   66         <NA>     <NA>    NA
    ## 3049              la   66         <NA>     <NA>    NA
    ## 3050         mayoral   66         <NA>     <NA>    NA
    ## 3051           names   66         <NA>     <NA>    NA
    ## 3052         nominee   66         <NA>     <NA>    NA
    ## 3053          result   66 anticipation     <NA>    NA
    ## 3054          robots   66         <NA>     <NA>    NA
    ## 3055           third   66         <NA>     <NA>    NA
    ## 3056            ugly   66      disgust negative    -3
    ## 3057            ugly   66     negative negative    -3
    ## 3058       universal   66         <NA>     <NA>    NA
    ## 3059     alternative   65         <NA>     <NA>    NA
    ## 3060          appear   65         <NA>     <NA>    NA
    ## 3061        argument   65        anger     <NA>    NA
    ## 3062        argument   65     negative     <NA>    NA
    ## 3063           blood   65         <NA>     <NA>    NA
    ## 3064         current   65         <NA>     <NA>    NA
    ## 3065           early   65         <NA>     <NA>    NA
    ## 3066         forward   65     positive     <NA>    NA
    ## 3067            hour   65         <NA>     <NA>    NA
    ## 3068          levels   65         <NA>     <NA>    NA
    ## 3069           lobby   65         <NA>     <NA>    -2
    ## 3070          making   65         <NA>     <NA>    NA
    ## 3071            part   65         <NA>     <NA>    NA
    ## 3072        playlist   65         <NA>     <NA>    NA
    ## 3073        reported   65         <NA>     <NA>    NA
    ## 3074         telling   65         <NA>     <NA>    NA
    ## 3075          valley   65         <NA>     <NA>    NA
    ## 3076        whatever   65         <NA>     <NA>    NA
    ## 3077               x   65         <NA>     <NA>    NA
    ## 3078           apple   64         <NA>     <NA>    NA
    ## 3079        building   64     positive     <NA>    NA
    ## 3080           clips   64         <NA>     <NA>    NA
    ## 3081            easy   64         <NA> positive     1
    ## 3082           loans   64         <NA>     <NA>    NA
    ## 3083         medical   64 anticipation     <NA>    NA
    ## 3084         medical   64         fear     <NA>    NA
    ## 3085         medical   64     positive     <NA>    NA
    ## 3086         medical   64        trust     <NA>    NA
    ## 3087               o   64         <NA>     <NA>    NA
    ## 3088           prior   64         <NA>     <NA>    NA
    ## 3089         sharing   64         <NA>     <NA>    NA
    ## 3090         smarter   64         <NA> positive     2
    ## 3091           stage   64         <NA>     <NA>    NA
    ## 3092        stranger   64         fear negative    NA
    ## 3093        stranger   64     negative negative    NA
    ## 3094         strikes   64         <NA>     <NA>    -1
    ## 3095          symbol   64         <NA>     <NA>    NA
    ## 3096           trend   64     positive     <NA>    NA
    ## 3097         victims   64         <NA>     <NA>    -3
    ## 3098            ai's   63         <NA>     <NA>    NA
    ## 3099         article   63         <NA>     <NA>    NA
    ## 3100             ask   63         <NA>     <NA>    NA
    ## 3101       dangerous   63         fear negative    NA
    ## 3102       dangerous   63     negative negative    NA
    ## 3103         demands   63         <NA>     <NA>    -1
    ## 3104       determine   63         <NA>     <NA>    NA
    ## 3105             dnc   63         <NA>     <NA>    NA
    ## 3106           enemy   63        anger negative    -2
    ## 3107           enemy   63      disgust negative    -2
    ## 3108           enemy   63         fear negative    -2
    ## 3109           enemy   63     negative negative    -2
    ## 3110          future   63         <NA>     <NA>    NA
    ## 3111           giant   63         fear     <NA>    NA
    ## 3112         matches   63         <NA>     <NA>    NA
    ## 3113             mma   63         <NA>     <NA>    NA
    ## 3114       necessary   63         <NA>     <NA>    NA
    ## 3115            pink   63         <NA>     <NA>    NA
    ## 3116           rises   63         <NA>     <NA>    NA
    ## 3117           shoot   63        anger     <NA>    -1
    ## 3118           shoot   63         fear     <NA>    -1
    ## 3119           shoot   63     negative     <NA>    -1
    ## 3120          tattoo   63         <NA>     <NA>    NA
    ## 3121         warfare   63        anger     <NA>    -2
    ## 3122         warfare   63         fear     <NA>    -2
    ## 3123         warfare   63     negative     <NA>    -2
    ## 3124         warfare   63      sadness     <NA>    -2
    ## 3125       wikipedia   63         <NA>     <NA>    NA
    ## 3126          actual   62     positive     <NA>    NA
    ## 3127           album   62         <NA>     <NA>    NA
    ## 3128           break   62     surprise negative    NA
    ## 3129        brothers   62         <NA>     <NA>    NA
    ## 3130          coffee   62         <NA>     <NA>    NA
    ## 3131         college   62         <NA>     <NA>    NA
    ## 3132          couldn   62         <NA>     <NA>    NA
    ## 3133            fall   62     negative negative    NA
    ## 3134            fall   62      sadness negative    NA
    ## 3135           honor   62     positive positive     2
    ## 3136           honor   62        trust positive     2
    ## 3137            jobs   62         <NA>     <NA>    NA
    ## 3138            late   62     negative     <NA>    NA
    ## 3139            late   62      sadness     <NA>    NA
    ## 3140           lives   62         <NA>     <NA>    NA
    ## 3141         mission   62         <NA>     <NA>    NA
    ## 3142         parking   62         <NA>     <NA>    NA
    ## 3143            sale   62         <NA>     <NA>    NA
    ## 3144           sense   62     positive     <NA>    NA
    ## 3145         singing   62         <NA>     <NA>    NA
    ## 3146        behavior   61         <NA>     <NA>    NA
    ## 3147         britain   61         <NA>     <NA>    NA
    ## 3148           cards   61         <NA>     <NA>    NA
    ## 3149          combat   61        anger     <NA>    -1
    ## 3150          combat   61         fear     <NA>    -1
    ## 3151          combat   61     negative     <NA>    -1
    ## 3152         concept   61         <NA>     <NA>    NA
    ## 3153            dark   61      sadness negative    NA
    ## 3154             dna   61         <NA>     <NA>    NA
    ## 3155            duty   61         <NA>     <NA>    NA
    ## 3156          engine   61         <NA>     <NA>    NA
    ## 3157         forever   61         <NA>     <NA>    NA
    ## 3158            hill   61         <NA>     <NA>    NA
    ## 3159            lied   61         <NA> negative    -2
    ## 3160            meta   61         <NA>     <NA>    NA
    ## 3161           nazis   61         <NA>     <NA>    NA
    ## 3162          nevada   61         <NA>     <NA>    NA
    ## 3163        reaction   61         <NA>     <NA>    NA
    ## 3164           scary   61         <NA> negative    -2
    ## 3165         bolivia   60         <NA>     <NA>    NA
    ## 3166           boost   60         <NA> positive     1
    ## 3167        carolina   60         <NA>     <NA>    NA
    ## 3168    conversation   60         <NA>     <NA>    NA
    ## 3169        deepmind   60         <NA>     <NA>    NA
    ## 3170        division   60         <NA>     <NA>    NA
    ## 3171          eating   60         <NA>     <NA>    NA
    ## 3172            evil   60        anger negative    -3
    ## 3173            evil   60      disgust negative    -3
    ## 3174            evil   60         fear negative    -3
    ## 3175            evil   60     negative negative    -3
    ## 3176            evil   60      sadness negative    -3
    ## 3177              ga   60         <NA>     <NA>    NA
    ## 3178          happen   60 anticipation     <NA>    NA
    ## 3179            hold   60         <NA>     <NA>    NA
    ## 3180       infection   60         fear negative    NA
    ## 3181       infection   60     negative negative    NA
    ## 3182            jack   60         <NA>     <NA>    NA
    ## 3183          matter   60         <NA>     <NA>     1
    ## 3184          member   60         <NA>     <NA>    NA
    ## 3185         playing   60         <NA>     <NA>    NA
    ## 3186          prince   60     positive     <NA>    NA
    ## 3187        princess   60     positive     <NA>    NA
    ## 3188          proper   60     positive positive    NA
    ## 3189          raised   60         <NA>     <NA>    NA
    ## 3190             red   60         <NA>     <NA>    NA
    ## 3191             rep   60         <NA>     <NA>    NA
    ## 3192           shark   60     negative negative    NA
    ## 3193            stay   60         <NA>     <NA>    NA
    ## 3194            tank   60         <NA> negative    NA
    ## 3195         turning   60         <NA>     <NA>    NA
    ## 3196         useless   60     negative negative    -2
    ## 3197            wake   60         <NA>     <NA>    NA
    ## 3198          asking   59         <NA>     <NA>    NA
    ## 3199         cartoon   59         <NA>     <NA>    NA
    ## 3200          crisis   59     negative negative    -3
    ## 3201         getting   59         <NA>     <NA>    NA
    ## 3202         happens   59         <NA>     <NA>    NA
    ## 3203         intense   59        anger negative     1
    ## 3204         intense   59      disgust negative     1
    ## 3205         intense   59         fear negative     1
    ## 3206         intense   59          joy negative     1
    ## 3207         intense   59     negative negative     1
    ## 3208         intense   59     positive negative     1
    ## 3209         intense   59     surprise negative     1
    ## 3210         intense   59        trust negative     1
    ## 3211             lie   59        anger negative    NA
    ## 3212             lie   59      disgust negative    NA
    ## 3213             lie   59     negative negative    NA
    ## 3214             lie   59      sadness negative    NA
    ## 3215            math   59         <NA>     <NA>    NA
    ## 3216          needed   59         <NA>     <NA>    NA
    ## 3217            rick   59         <NA>     <NA>    NA
    ## 3218            sees   59         <NA>     <NA>    NA
    ## 3219         serious   59         <NA>     <NA>    NA
    ## 3220           smith   59        trust     <NA>    NA
    ## 3221            text   59         <NA>     <NA>    NA
    ## 3222        training   59         <NA>     <NA>    NA
    ## 3223         workers   59         <NA>     <NA>    NA
    ## 3224         actions   58         <NA>     <NA>    NA
    ## 3225           bonus   58 anticipation positive    NA
    ## 3226           bonus   58          joy positive    NA
    ## 3227           bonus   58     positive positive    NA
    ## 3228           bonus   58     surprise positive    NA
    ## 3229             cat   58         <NA>     <NA>    NA
    ## 3230          common   58         <NA>     <NA>    NA
    ## 3231      depression   58     negative negative    NA
    ## 3232      depression   58      sadness negative    NA
    ## 3233          emails   58         <NA>     <NA>    NA
    ## 3234           enter   58         <NA>     <NA>    NA
    ## 3235           extra   58     positive     <NA>    NA
    ## 3236         friends   58         <NA>     <NA>    NA
    ## 3237         granted   58     positive     <NA>     1
    ## 3238            imdb   58         <NA>     <NA>    NA
    ## 3239          killer   58         <NA> negative    NA
    ## 3240       landslide   58         fear     <NA>    NA
    ## 3241       landslide   58     negative     <NA>    NA
    ## 3242       landslide   58      sadness     <NA>    NA
    ## 3243            lies   58         <NA> negative    NA
    ## 3244           looks   58         <NA>     <NA>    NA
    ## 3245          motion   58 anticipation     <NA>    NA
    ## 3246        neighbor   58 anticipation     <NA>    NA
    ## 3247        neighbor   58     positive     <NA>    NA
    ## 3248        neighbor   58        trust     <NA>    NA
    ## 3249          policy   58        trust     <NA>    NA
    ## 3250           punch   58        anger negative    NA
    ## 3251           punch   58         fear negative    NA
    ## 3252           punch   58     negative negative    NA
    ## 3253           punch   58      sadness negative    NA
    ## 3254           punch   58     surprise negative    NA
    ## 3255            shot   58        anger     <NA>    NA
    ## 3256            shot   58         fear     <NA>    NA
    ## 3257            shot   58     negative     <NA>    NA
    ## 3258            shot   58      sadness     <NA>    NA
    ## 3259            shot   58     surprise     <NA>    NA
    ## 3260            side   58         <NA>     <NA>    NA
    ## 3261        thinking   58         <NA>     <NA>    NA
    ## 3262          throws   58         <NA>     <NA>    NA
    ## 3263          ticket   58         <NA>     <NA>    NA
    ## 3264          unless   58         <NA>     <NA>    NA
    ## 3265       abandoned   57        anger     <NA>    -2
    ## 3266       abandoned   57         fear     <NA>    -2
    ## 3267       abandoned   57     negative     <NA>    -2
    ## 3268       abandoned   57      sadness     <NA>    -2
    ## 3269            bout   57        anger     <NA>    NA
    ## 3270            bout   57     negative     <NA>    NA
    ## 3271         closing   57         <NA>     <NA>    NA
    ## 3272          domain   57         <NA>     <NA>    NA
    ## 3273           drunk   57         <NA> negative    -2
    ## 3274       education   57         <NA>     <NA>    NA
    ## 3275           emoji   57         <NA>     <NA>    NA
    ## 3276       emotional   57         <NA>     <NA>    NA
    ## 3277             eu4   57         <NA>     <NA>    NA
    ## 3278         existed   57         <NA>     <NA>    NA
    ## 3279          expert   57     positive     <NA>    NA
    ## 3280          expert   57        trust     <NA>    NA
    ## 3281            gave   57         <NA>     <NA>    NA
    ## 3282      incredible   57         <NA> positive    NA
    ## 3283     interesting   57     positive positive     2
    ## 3284           likes   57         <NA> positive     2
    ## 3285          listen   57         <NA>     <NA>    NA
    ## 3286          martin   57         <NA>     <NA>    NA
    ## 3287            paid   57         <NA>     <NA>    NA
    ## 3288            plan   57 anticipation     <NA>    NA
    ## 3289       reference   57         <NA>     <NA>    NA
    ## 3290            road   57         <NA>     <NA>    NA
    ## 3291           robot   57         <NA>     <NA>    NA
    ## 3292           sales   57         <NA>     <NA>    NA
    ## 3293           scott   57         <NA>     <NA>    NA
    ## 3294            seem   57         <NA>     <NA>    NA
    ## 3295           seems   57         <NA>     <NA>    NA
    ## 3296           south   57         <NA>     <NA>    NA
    ## 3297        specific   57         <NA>     <NA>    NA
    ## 3298          adults   56         <NA>     <NA>    NA
    ## 3299        annoying   56        anger negative    -2
    ## 3300        annoying   56     negative negative    -2
    ## 3301            bird   56         <NA>     <NA>    NA
    ## 3302       bloomberg   56         <NA>     <NA>    NA
    ## 3303           cells   56         <NA>     <NA>    NA
    ## 3304      convention   56     positive     <NA>    NA
    ## 3305           cured   56         <NA>     <NA>    NA
    ## 3306         embassy   56         <NA>     <NA>    NA
    ## 3307       explosion   56         fear     <NA>    NA
    ## 3308       explosion   56     negative     <NA>    NA
    ## 3309       explosion   56     surprise     <NA>    NA
    ## 3310         fiction   56         <NA> negative    NA
    ## 3311          filmed   56         <NA>     <NA>    NA
    ## 3312        freaking   56         <NA> negative    NA
    ## 3313        google's   56         <NA>     <NA>    NA
    ## 3314              hi   56         <NA>     <NA>    NA
    ## 3315     independent   56         <NA>     <NA>    NA
    ## 3316           kinds   56         <NA>     <NA>    NA
    ## 3317        possible   56         <NA>     <NA>    NA
    ## 3318       potential   56         <NA>     <NA>    NA
    ## 3319           potus   56         <NA>     <NA>    NA
    ## 3320            ruin   56         fear negative    -2
    ## 3321            ruin   56     negative negative    -2
    ## 3322            ruin   56      sadness negative    -2
    ## 3323            rule   56         fear     <NA>    NA
    ## 3324            rule   56        trust     <NA>    NA
    ## 3325          season   56         <NA>     <NA>    NA
    ## 3326         senator   56         <NA>     <NA>    NA
    ## 3327          speech   56     positive     <NA>    NA
    ## 3328           agent   55         <NA>     <NA>    NA
    ## 3329           alone   55         <NA>     <NA>    -2
    ## 3330      artificial   55         <NA>     <NA>    NA
    ## 3331       challenge   55        anger     <NA>    -1
    ## 3332       challenge   55         fear     <NA>    -1
    ## 3333       challenge   55     negative     <NA>    -1
    ## 3334        concerns   55         <NA> negative    NA
    ## 3335             dem   55         <NA>     <NA>    NA
    ## 3336     documentary   55         <NA>     <NA>    NA
    ## 3337            iran   55         <NA>     <NA>    NA
    ## 3338            lots   55         <NA>     <NA>    NA
    ## 3339           lying   55        anger negative    NA
    ## 3340           lying   55      disgust negative    NA
    ## 3341           lying   55     negative negative    NA
    ## 3342           masks   55         <NA>     <NA>    NA
    ## 3343           means   55         <NA>     <NA>    NA
    ## 3344         meeting   55         <NA>     <NA>    NA
    ## 3345           milan   55         <NA>     <NA>    NA
    ## 3346       recommend   55     positive positive     2
    ## 3347       recommend   55        trust positive     2
    ## 3348          spring   55         <NA>     <NA>    NA
    ## 3349          within   55         <NA>     <NA>    NA
    ## 3350         ability   54     positive     <NA>     2
    ## 3351        accident   54         fear     <NA>    -2
    ## 3352        accident   54     negative     <NA>    -2
    ## 3353        accident   54      sadness     <NA>    -2
    ## 3354        accident   54     surprise     <NA>    -2
    ## 3355              ca   54         <NA>     <NA>    NA
    ## 3356         damaged   54         <NA> negative    NA
    ## 3357        district   54         <NA>     <NA>    NA
    ## 3358         drawing   54         <NA>     <NA>    NA
    ## 3359      especially   54         <NA>     <NA>    NA
    ## 3360         finland   54         <NA>     <NA>    NA
    ## 3361         instant   54         <NA>     <NA>    NA
    ## 3362          living   54         <NA>     <NA>    NA
    ## 3363              mp   54         <NA>     <NA>    NA
    ## 3364           needs   54         <NA>     <NA>    NA
    ## 3365      quarantine   54         fear     <NA>    NA
    ## 3366            quit   54     negative     <NA>    NA
    ## 3367        remember   54         <NA>     <NA>    NA
    ## 3368           rogan   54         <NA>     <NA>    NA
    ## 3369        soldiers   54         <NA>     <NA>    NA
    ## 3370           super   54         <NA> positive     3
    ## 3371         viruses   54         <NA>     <NA>    NA
    ## 3372        whenever   54         <NA>     <NA>    NA
    ## 3373        assuming   53         <NA>     <NA>    NA
    ## 3374             bit   53         <NA>     <NA>    NA
    ## 3375          buying   53         <NA>     <NA>    NA
    ## 3376       clearview   53         <NA>     <NA>    NA
    ## 3377         figures   53         <NA>     <NA>    NA
    ## 3378         footage   53         <NA>     <NA>    NA
    ## 3379          harder   53         <NA>     <NA>    NA
    ## 3380          indian   53         <NA>     <NA>    NA
    ## 3381       insurance   53         <NA>     <NA>    NA
    ## 3382            kind   53          joy     <NA>     2
    ## 3383            kind   53     positive     <NA>     2
    ## 3384            kind   53        trust     <NA>     2
    ## 3385            loss   53        anger negative    -3
    ## 3386            loss   53         fear negative    -3
    ## 3387            loss   53     negative negative    -3
    ## 3388            loss   53      sadness negative    -3
    ## 3389       meanwhile   53         <NA>     <NA>    NA
    ## 3390              mr   53         <NA>     <NA>    NA
    ## 3391           msnbc   53         <NA>     <NA>    NA
    ## 3392            ones   53         <NA>     <NA>    NA
    ## 3393        painting   53         <NA>     <NA>    NA
    ## 3394        previous   53         <NA>     <NA>    NA
    ## 3395         several   53         <NA>     <NA>    NA
    ## 3396        thoughts   53         <NA>     <NA>    NA
    ## 3397           added   52         <NA>     <NA>    NA
    ## 3398         besides   52         <NA>     <NA>    NA
    ## 3399             car   52         <NA>     <NA>    NA
    ## 3400          closed   52         <NA>     <NA>    NA
    ## 3401        creative   52     positive positive     2
    ## 3402       emergency   52         fear negative    -2
    ## 3403       emergency   52     negative negative    -2
    ## 3404       emergency   52      sadness negative    -2
    ## 3405       emergency   52     surprise negative    -2
    ## 3406           helps   52         <NA>     <NA>     2
    ## 3407            link   52         <NA>     <NA>    NA
    ## 3408            miss   52         <NA> negative    -2
    ## 3409         romance   52 anticipation     <NA>     2
    ## 3410         romance   52         fear     <NA>     2
    ## 3411         romance   52          joy     <NA>     2
    ## 3412         romance   52     positive     <NA>     2
    ## 3413         romance   52      sadness     <NA>     2
    ## 3414         romance   52     surprise     <NA>     2
    ## 3415         romance   52        trust     <NA>     2
    ## 3416          screen   52         <NA>     <NA>    NA
    ## 3417        separate   52         <NA>     <NA>    NA
    ## 3418            sick   52      disgust negative    -2
    ## 3419            sick   52     negative negative    -2
    ## 3420            sick   52      sadness negative    -2
    ## 3421           sound   52         <NA>     <NA>    NA
    ## 3422        starting   52         <NA>     <NA>    NA
    ## 3423           surge   52     surprise     <NA>    NA
    ## 3424           think   52         <NA>     <NA>    NA
    ## 3425         violent   52        anger negative    -3
    ## 3426         violent   52      disgust negative    -3
    ## 3427         violent   52         fear negative    -3
    ## 3428         violent   52     negative negative    -3
    ## 3429         violent   52     surprise negative    -3
    ## 3430            yang   52         <NA>     <NA>    NA
    ## 3431           among   51         <NA>     <NA>    NA
    ## 3432        critical   51         <NA> negative    NA
    ## 3433         dislike   51        anger negative    -2
    ## 3434         dislike   51      disgust negative    -2
    ## 3435         dislike   51     negative negative    -2
    ## 3436       hampshire   51         <NA>     <NA>    NA
    ## 3437           maybe   51         <NA>     <NA>    NA
    ## 3438          mostly   51         <NA>     <NA>    NA
    ## 3439       neighbors   51         <NA>     <NA>    NA
    ## 3440           puppy   51 anticipation     <NA>    NA
    ## 3441           puppy   51     positive     <NA>    NA
    ## 3442           puppy   51        trust     <NA>    NA
    ## 3443          summer   51         <NA>     <NA>    NA
    ## 3444     translation   51        trust     <NA>    NA
    ## 3445          barack   50         <NA>     <NA>    NA
    ## 3446          caught   50         <NA>     <NA>    NA
    ## 3447           crime   50        anger negative    -3
    ## 3448           crime   50     negative negative    -3
    ## 3449        designed   50         <NA>     <NA>    NA
    ## 3450      developing   50         <NA>     <NA>    NA
    ## 3451            fact   50        trust     <NA>    NA
    ## 3452         fighter   50         <NA>     <NA>    NA
    ## 3453           floor   50         <NA>     <NA>    NA
    ## 3454        included   50     positive     <NA>    NA
    ## 3455        inspired   50          joy     <NA>     2
    ## 3456        inspired   50     positive     <NA>     2
    ## 3457        inspired   50     surprise     <NA>     2
    ## 3458        inspired   50        trust     <NA>     2
    ## 3459            kick   50        anger     <NA>    NA
    ## 3460            kick   50     negative     <NA>    NA
    ## 3461     libertarian   50         <NA>     <NA>    NA
    ## 3462         manager   50         <NA>     <NA>    NA
    ## 3463       positions   50         <NA>     <NA>    NA
    ## 3464         regards   50         <NA>     <NA>    NA
    ## 3465        religion   50        trust     <NA>    NA
    ## 3466       screening   50         <NA>     <NA>    NA
    ## 3467         thought   50 anticipation     <NA>    NA
    ## 3468            tony   50         <NA>     <NA>    NA
    ## 3469         towards   50         <NA>     <NA>    NA
    ## 3470          worked   50         <NA> positive    NA
    ## 3471           bored   49         <NA> negative    -2
    ## 3472     considering   49         <NA>     <NA>    NA
    ## 3473        europe's   49         <NA>     <NA>    NA
    ## 3474             gov   49         <NA>     <NA>    NA
    ## 3475         illness   49         fear negative    -2
    ## 3476         illness   49     negative negative    -2
    ## 3477         illness   49      sadness negative    -2
    ## 3478       impeached   49         <NA>     <NA>    NA
    ## 3479      inevitable   49         <NA> negative    NA
    ## 3480          models   49         <NA>     <NA>    NA
    ## 3481        overview   49         <NA>     <NA>    NA
    ## 3482     perspective   49         <NA>     <NA>    NA
    ## 3483          picked   49         <NA>     <NA>    NA
    ## 3484       situation   49         <NA>     <NA>    NA
    ## 3485          sports   49         <NA>     <NA>    NA
    ## 3486          tackle   49        anger     <NA>    NA
    ## 3487          tackle   49     surprise     <NA>    NA
    ## 3488           truly   49         <NA>     <NA>    NA
    ## 3489           worth   49     positive positive     2
    ## 3490              yo   49         <NA>     <NA>    NA
    ## 3491         amazing   48         <NA> positive     4
    ## 3492     appreciated   48         <NA> positive     2
    ## 3493           blind   48     negative negative    -1
    ## 3494           cloud   48         <NA> negative    NA
    ## 3495      difficulty   48        anger negative    NA
    ## 3496      difficulty   48         fear negative    NA
    ## 3497      difficulty   48     negative negative    NA
    ## 3498      difficulty   48      sadness negative    NA
    ## 3499              fi   48         <NA>     <NA>    NA
    ## 3500            loan   48         <NA>     <NA>    NA
    ## 3501        machines   48         <NA>     <NA>    NA
    ## 3502            open   48         <NA>     <NA>    NA
    ## 3503        pentagon   48         <NA>     <NA>    NA
    ## 3504        predicts   48         <NA>     <NA>    NA
    ## 3505             sci   48         <NA>     <NA>    NA
    ## 3506         spanish   48         <NA>     <NA>    NA
    ## 3507            tier   48         <NA>     <NA>    NA
    ## 3508   understanding   48     positive     <NA>    NA
    ## 3509   understanding   48        trust     <NA>    NA
    ## 3510        upcoming   48         <NA>     <NA>    NA
    ## 3511         walmart   48         <NA>     <NA>    NA
    ## 3512    announcement   47 anticipation     <NA>    NA
    ## 3513            blue   47      sadness     <NA>    NA
    ## 3514             boy   47      disgust     <NA>    NA
    ## 3515             boy   47     negative     <NA>    NA
    ## 3516        contract   47         <NA>     <NA>    NA
    ## 3517        exchange   47     positive     <NA>    NA
    ## 3518        exchange   47        trust     <NA>    NA
    ## 3519              fa   47         <NA>     <NA>    NA
    ## 3520       fantastic   47         <NA> positive     4
    ## 3521            gain   47 anticipation positive     2
    ## 3522            gain   47          joy positive     2
    ## 3523            gain   47     positive positive     2
    ## 3524             gap   47     negative     <NA>    NA
    ## 3525           grows   47         <NA>     <NA>    NA
    ## 3526          income   47 anticipation     <NA>    NA
    ## 3527          income   47          joy     <NA>    NA
    ## 3528          income   47     negative     <NA>    NA
    ## 3529          income   47     positive     <NA>    NA
    ## 3530          income   47      sadness     <NA>    NA
    ## 3531          income   47        trust     <NA>    NA
    ## 3532         ireland   47         <NA>     <NA>    NA
    ## 3533        promised   47         <NA> positive     1
    ## 3534          refuse   47     negative negative    -2
    ## 3535            seek   47 anticipation     <NA>    NA
    ## 3536        survival   47         <NA> positive    NA
    ## 3537       violating   47         <NA>     <NA>    -2
    ## 3538            wide   47         <NA>     <NA>    NA
    ## 3539              ai   46         <NA>     <NA>    NA
    ## 3540           built   46         <NA>     <NA>    NA
    ## 3541           comic   46         <NA>     <NA>    NA
    ## 3542             dog   46         <NA>     <NA>    NA
    ## 3543          dollar   46         <NA>     <NA>    NA
    ## 3544            dont   46         <NA>     <NA>    NA
    ## 3545           false   46         <NA> negative    NA
    ## 3546            fool   46      disgust negative    -2
    ## 3547            fool   46     negative negative    -2
    ## 3548           large   46         <NA>     <NA>    NA
    ## 3549         machina   46         <NA>     <NA>    NA
    ## 3550           man's   46         <NA>     <NA>    NA
    ## 3551           music   46          joy     <NA>    NA
    ## 3552           music   46     positive     <NA>    NA
    ## 3553           music   46      sadness     <NA>    NA
    ## 3554         october   46         <NA>     <NA>    NA
    ## 3555            rain   46         <NA>     <NA>    NA
    ## 3556           raise   46         <NA>     <NA>    NA
    ## 3557       regarding   46         <NA>     <NA>    NA
    ## 3558    registration   46         <NA>     <NA>    NA
    ## 3559           stock   46         <NA>     <NA>    NA
    ## 3560        students   46         <NA>     <NA>    NA
    ## 3561         success   46 anticipation positive     2
    ## 3562         success   46          joy positive     2
    ## 3563         success   46     positive positive     2
    ## 3564        touching   46         <NA>     <NA>    NA
    ## 3565            zone   46         <NA>     <NA>    NA
    ## 3566      accurately   45         <NA> positive    NA
    ## 3567        canceled   45         <NA>     <NA>    NA
    ## 3568            card   45         <NA>     <NA>    NA
    ## 3569         company   45         <NA>     <NA>    NA
    ## 3570        complain   45        anger negative    -2
    ## 3571        complain   45     negative negative    -2
    ## 3572        complain   45      sadness negative    -2
    ## 3573         divided   45         <NA>     <NA>    NA
    ## 3574        dropping   45         <NA>     <NA>    NA
    ## 3575          easter   45         <NA>     <NA>    NA
    ## 3576         efforts   45         <NA>     <NA>    NA
    ## 3577            ends   45         <NA>     <NA>    NA
    ## 3578        evidence   45         <NA>     <NA>    NA
    ## 3579            fish   45         <NA>     <NA>    NA
    ## 3580      fluminense   45         <NA>     <NA>    NA
    ## 3581           focus   45     positive     <NA>    NA
    ## 3582           laugh   45          joy     <NA>     1
    ## 3583           laugh   45     positive     <NA>     1
    ## 3584           laugh   45     surprise     <NA>     1
    ## 3585           moral   45        anger     <NA>    NA
    ## 3586           moral   45     positive     <NA>    NA
    ## 3587           moral   45        trust     <NA>    NA
    ## 3588           pedro   45         <NA>     <NA>    NA
    ## 3589        prepared   45 anticipation     <NA>     1
    ## 3590        prepared   45     positive     <NA>     1
    ## 3591        prepared   45        trust     <NA>     1
    ## 3592            sinn   45         <NA>     <NA>    NA
    ## 3593            skip   45     negative     <NA>    NA
    ## 3594            wear   45     negative     <NA>    NA
    ## 3595            wear   45        trust     <NA>    NA
    ## 3596             ali   44         <NA>     <NA>    NA
    ## 3597         anybody   44         <NA>     <NA>    NA
    ## 3598          biopic   44         <NA>     <NA>    NA
    ## 3599            buff   44         <NA>     <NA>    NA
    ## 3600       confident   44          joy positive     2
    ## 3601       confident   44     positive positive     2
    ## 3602       confident   44        trust positive     2
    ## 3603       corporate   44         <NA>     <NA>    NA
    ## 3604            farm   44 anticipation     <NA>    NA
    ## 3605         harmful   44        anger negative    -2
    ## 3606         harmful   44      disgust negative    -2
    ## 3607         harmful   44         fear negative    -2
    ## 3608         harmful   44     negative negative    -2
    ## 3609         harmful   44      sadness negative    -2
    ## 3610             idk   44         <NA>     <NA>    NA
    ## 3611            june   44         <NA>     <NA>    NA
    ## 3612           links   44         <NA>     <NA>    NA
    ## 3613         machine   44        trust     <NA>    NA
    ## 3614           might   44         <NA>     <NA>    NA
    ## 3615         minimum   44     negative     <NA>    NA
    ## 3616        missions   44         <NA>     <NA>    NA
    ## 3617           model   44     positive     <NA>    NA
    ## 3618           nurse   44     positive     <NA>    NA
    ## 3619           nurse   44        trust     <NA>    NA
    ## 3620         opinion   44         <NA>     <NA>    NA
    ## 3621   parliamentary   44         <NA>     <NA>    NA
    ## 3622           phone   44         <NA>     <NA>    NA
    ## 3623          planet   44         <NA>     <NA>    NA
    ## 3624      protection   44         <NA> positive    NA
    ## 3625        roasting   44         <NA>     <NA>    NA
    ## 3626          secret   44        trust     <NA>    NA
    ## 3627      automation   43         <NA>     <NA>    NA
    ## 3628          babies   43         <NA>     <NA>    NA
    ## 3629           cycle   43         <NA>     <NA>    NA
    ## 3630            data   43         <NA>     <NA>    NA
    ## 3631           doors   43         <NA>     <NA>    NA
    ## 3632           dutch   43         <NA>     <NA>    NA
    ## 3633     engineering   43         <NA>     <NA>    NA
    ## 3634           entry   43         <NA>     <NA>    NA
    ## 3635          garage   43         <NA>     <NA>    NA
    ## 3636           glass   43         <NA>     <NA>    NA
    ## 3637        honestly   43         <NA>     <NA>    NA
    ## 3638              oc   43         <NA>     <NA>    NA
    ## 3639         outcome   43     positive     <NA>    NA
    ## 3640          remain   43         <NA>     <NA>    NA
    ## 3641          skills   43         <NA>     <NA>    NA
    ## 3642          though   43         <NA>     <NA>    NA
    ## 3643            beat   42         <NA>     <NA>    NA
    ## 3644      brasileiro   42         <NA>     <NA>    NA
    ## 3645          breast   42         <NA>     <NA>    NA
    ## 3646      cavalinhos   42         <NA>     <NA>    NA
    ## 3647            cold   42     negative negative    NA
    ## 3648         contact   42     positive     <NA>    NA
    ## 3649       developed   42         <NA>     <NA>    NA
    ## 3650            heat   42         <NA>     <NA>    NA
    ## 3651          impact   42         <NA>     <NA>    NA
    ## 3652          length   42         <NA>     <NA>    NA
    ## 3653       mandatory   42         <NA>     <NA>    -1
    ## 3654            mean   42         <NA>     <NA>    NA
    ## 3655      medication   42         <NA>     <NA>    NA
    ## 3656      presidency   42         <NA>     <NA>    NA
    ## 3657            sent   42         <NA>     <NA>    NA
    ## 3658       sometimes   42         <NA>     <NA>    NA
    ## 3659           sport   42         <NA>     <NA>    NA
    ## 3660              st   42         <NA>     <NA>    NA
    ## 3661          wishes   42         <NA>     <NA>     1
    ## 3662          zombie   42         <NA> negative    NA
    ## 3663              ad   41         <NA>     <NA>    NA
    ## 3664      biological   41         <NA>     <NA>    NA
    ## 3665      businesses   41         <NA>     <NA>    NA
    ## 3666           chaos   41        anger negative    -2
    ## 3667           chaos   41         fear negative    -2
    ## 3668           chaos   41     negative negative    -2
    ## 3669           chaos   41      sadness negative    -2
    ## 3670    distribution   41         <NA>     <NA>    NA
    ## 3671       expecting   41 anticipation     <NA>    NA
    ## 3672            kung   41         <NA>     <NA>    NA
    ## 3673        lockdown   41         <NA>     <NA>    NA
    ## 3674           metal   41         <NA>     <NA>    NA
    ## 3675        networks   41         <NA>     <NA>    NA
    ## 3676          neural   41         <NA>     <NA>    NA
    ## 3677             pvp   41         <NA>     <NA>    NA
    ## 3678           quick   41         <NA>     <NA>    NA
    ## 3679         related   41        trust     <NA>    NA
    ## 3680            shop   41         <NA>     <NA>    NA
    ## 3681             sun   41 anticipation     <NA>    NA
    ## 3682             sun   41          joy     <NA>    NA
    ## 3683             sun   41     positive     <NA>    NA
    ## 3684             sun   41     surprise     <NA>    NA
    ## 3685             sun   41        trust     <NA>    NA
    ## 3686          threat   41        anger negative    -2
    ## 3687          threat   41         fear negative    -2
    ## 3688          threat   41     negative negative    -2
    ## 3689        websites   41         <NA>     <NA>    NA
    ## 3690       affecting   40         <NA>     <NA>    NA
    ## 3691          amount   40         <NA>     <NA>    NA
    ## 3692           beats   40         <NA>     <NA>    NA
    ## 3693            bias   40        anger negative    -1
    ## 3694            bias   40     negative negative    -1
    ## 3695           boris   40         <NA>     <NA>    NA
    ## 3696            camp   40         <NA>     <NA>    NA
    ## 3697            cure   40         <NA> positive    NA
    ## 3698           dance   40          joy     <NA>    NA
    ## 3699           dance   40     positive     <NA>    NA
    ## 3700           dance   40        trust     <NA>    NA
    ## 3701            dare   40 anticipation     <NA>    NA
    ## 3702            dare   40        trust     <NA>    NA
    ## 3703          easily   40         <NA>     <NA>    NA
    ## 3704           ebola   40         <NA>     <NA>    NA
    ## 3705            foot   40         <NA>     <NA>    NA
    ## 3706        friendly   40 anticipation positive     2
    ## 3707        friendly   40          joy positive     2
    ## 3708        friendly   40     positive positive     2
    ## 3709        friendly   40        trust positive     2
    ## 3710          injury   40        anger negative    -2
    ## 3711          injury   40         fear negative    -2
    ## 3712          injury   40     negative negative    -2
    ## 3713          injury   40      sadness negative    -2
    ## 3714          insane   40        anger negative    -2
    ## 3715          insane   40         fear negative    -2
    ## 3716          insane   40     negative negative    -2
    ## 3717         insight   40         <NA>     <NA>    NA
    ## 3718        movement   40         <NA>     <NA>    NA
    ## 3719            nose   40      disgust     <NA>    NA
    ## 3720             nyc   40         <NA>     <NA>    NA
    ## 3721           polls   40         <NA>     <NA>    NA
    ## 3722         purpose   40         <NA>     <NA>    NA
    ## 3723 recommendations   40         <NA> positive    NA
    ## 3724         reminds   40         <NA>     <NA>    NA
    ## 3725          sparks   40         <NA>     <NA>    NA
    ## 3726        standing   40     positive     <NA>    NA
    ## 3727          tested   40         <NA>     <NA>    NA
    ## 3728           types   40         <NA>     <NA>    NA
    ## 3729            warn   40 anticipation     <NA>    -2
    ## 3730            warn   40         fear     <NA>    -2
    ## 3731            warn   40     negative     <NA>    -2
    ## 3732            warn   40     surprise     <NA>    -2
    ## 3733            warn   40        trust     <NA>    -2
    ## 3734              ac   39         <NA>     <NA>    NA
    ## 3735          answer   39         <NA>     <NA>    NA
    ## 3736        birthday   39 anticipation     <NA>    NA
    ## 3737        birthday   39          joy     <NA>    NA
    ## 3738        birthday   39     positive     <NA>    NA
    ## 3739        birthday   39     surprise     <NA>    NA
    ## 3740          checks   39         <NA>     <NA>    NA
    ## 3741           count   39     positive     <NA>    NA
    ## 3742           count   39        trust     <NA>    NA
    ## 3743            deep   39         <NA>     <NA>    NA
    ## 3744             flu   39         fear     <NA>    -2
    ## 3745             flu   39     negative     <NA>    -2
    ## 3746           goals   39         <NA>     <NA>    NA
    ## 3747          helped   39         <NA> positive    NA
    ## 3748            host   39         <NA>     <NA>    NA
    ## 3749        humanity   39          joy     <NA>    NA
    ## 3750        humanity   39     positive     <NA>    NA
    ## 3751        humanity   39        trust     <NA>    NA
    ## 3752   international   39         <NA>     <NA>    NA
    ## 3753         knowing   39     positive     <NA>    NA
    ## 3754          master   39     positive positive    NA
    ## 3755           perks   39         <NA>     <NA>    NA
    ## 3756           rates   39         <NA>     <NA>    NA
    ## 3757            room   39         <NA>     <NA>    NA
    ## 3758            solo   39         <NA>     <NA>    NA
    ## 3759         youtube   39         <NA>     <NA>    NA
    ## 3760            army   38         <NA>     <NA>    NA
    ## 3761        deserves   38         <NA>     <NA>    NA
    ## 3762            dick   38         <NA> negative    -4
    ## 3763      difference   38         <NA>     <NA>    NA
    ## 3764        directed   38         <NA>     <NA>    NA
    ## 3765         editing   38         <NA>     <NA>    NA
    ## 3766      equivalent   38         <NA>     <NA>    NA
    ## 3767    implications   38         <NA>     <NA>    NA
    ## 3768           labor   38 anticipation     <NA>    NA
    ## 3769           labor   38          joy     <NA>    NA
    ## 3770           labor   38     positive     <NA>    NA
    ## 3771           labor   38     surprise     <NA>    NA
    ## 3772           labor   38        trust     <NA>    NA
    ## 3773           learn   38     positive     <NA>    NA
    ## 3774             max   38         <NA>     <NA>    NA
    ## 3775     multiplayer   38         <NA>     <NA>    NA
    ## 3776            paul   38         <NA>     <NA>    NA
    ## 3777          plague   38      disgust negative    NA
    ## 3778          plague   38         fear negative    NA
    ## 3779          plague   38     negative negative    NA
    ## 3780          plague   38      sadness negative    NA
    ## 3781         project   38         <NA>     <NA>    NA
    ## 3782           santa   38         <NA>     <NA>    NA
    ## 3783              sc   38         <NA>     <NA>    NA
    ## 3784           sleep   38         <NA>     <NA>    NA
    ## 3785           talks   38         <NA>     <NA>    NA
    ## 3786           teeth   38         <NA>     <NA>    NA
    ## 3787          tracks   38         <NA>     <NA>    NA
    ## 3788         wanting   38     negative     <NA>    NA
    ## 3789         wanting   38      sadness     <NA>    NA
    ## 3790          warned   38 anticipation negative    -2
    ## 3791          warned   38         fear negative    -2
    ## 3792          warned   38     surprise negative    -2
    ## 3793          church   37 anticipation     <NA>    NA
    ## 3794          church   37          joy     <NA>    NA
    ## 3795          church   37     positive     <NA>    NA
    ## 3796          church   37        trust     <NA>    NA
    ## 3797          deaths   37         <NA>     <NA>    NA
    ## 3798         english   37         <NA>     <NA>    NA
    ## 3799        flamengo   37         <NA>     <NA>    NA
    ## 3800          forces   37         <NA>     <NA>    NA
    ## 3801        obsolete   37         <NA> negative    -2
    ## 3802            pain   37         fear negative    -2
    ## 3803            pain   37     negative negative    -2
    ## 3804            pain   37      sadness negative    -2
    ## 3805       replacing   37         <NA>     <NA>    NA
    ## 3806          runner   37         <NA>     <NA>    NA
    ## 3807    specifically   37         <NA>     <NA>    NA
    ## 3808        teaching   37         <NA>     <NA>    NA
    ## 3809         anxiety   36        anger negative    -2
    ## 3810         anxiety   36 anticipation negative    -2
    ## 3811         anxiety   36         fear negative    -2
    ## 3812         anxiety   36     negative negative    -2
    ## 3813         anxiety   36      sadness negative    -2
    ## 3814            ball   36         <NA>     <NA>    NA
    ## 3815          carbon   36         <NA>     <NA>    NA
    ## 3816         causing   36         <NA>     <NA>    NA
    ## 3817          charge   36         <NA>     <NA>    NA
    ## 3818      contribute   36     positive     <NA>    NA
    ## 3819          couple   36         <NA>     <NA>    NA
    ## 3820       expensive   36         <NA> negative    NA
    ## 3821     killstreaks   36         <NA>     <NA>    NA
    ## 3822           knife   36         <NA> negative    NA
    ## 3823         license   36         <NA>     <NA>    NA
    ## 3824           lived   36         <NA>     <NA>    NA
    ## 3825            mine   36         <NA>     <NA>    NA
    ## 3826         mistake   36     negative negative    -2
    ## 3827         mistake   36      sadness negative    -2
    ## 3828         natural   36         <NA>     <NA>     1
    ## 3829        progress   36 anticipation positive     2
    ## 3830        progress   36          joy positive     2
    ## 3831        progress   36     positive positive     2
    ## 3832        question   36     positive     <NA>    NA
    ## 3833         reading   36     positive     <NA>    NA
    ## 3834         similar   36         <NA>     <NA>    NA
    ## 3835     singularity   36         <NA>     <NA>    NA
    ## 3836            snap   36         <NA>     <NA>    NA
    ## 3837         species   36         <NA>     <NA>    NA
    ## 3838           spend   36         <NA>     <NA>    NA
    ## 3839           stars   36         <NA>     <NA>    NA
    ## 3840        throwing   36         <NA>     <NA>    NA
    ## 3841            ways   36         <NA>     <NA>    NA
    ## 3842         classes   35         <NA>     <NA>    NA
    ## 3843        creating   35         <NA>     <NA>    NA
    ## 3844            cute   35     positive positive     2
    ## 3845           dates   35         <NA>     <NA>    NA
    ## 3846     experiences   35         <NA>     <NA>    NA
    ## 3847           holds   35         <NA>     <NA>    NA
    ## 3848         however   35         <NA>     <NA>    NA
    ## 3849         injured   35         fear     <NA>    -2
    ## 3850         injured   35     negative     <NA>    -2
    ## 3851         injured   35      sadness     <NA>    -2
    ## 3852      interviews   35         <NA>     <NA>    NA
    ## 3853       mentioned   35         <NA>     <NA>    NA
    ## 3854         noticed   35         <NA>     <NA>    NA
    ## 3855            pets   35         <NA>     <NA>    NA
    ## 3856       primaries   35         <NA>     <NA>    NA
    ## 3857         request   35         <NA>     <NA>    NA
    ## 3858       requiring   35         <NA>     <NA>    NA
    ## 3859            safe   35          joy positive     1
    ## 3860            safe   35     positive positive     1
    ## 3861            safe   35        trust positive     1
    ## 3862      soundtrack   35         <NA>     <NA>    NA
    ## 3863          toilet   35      disgust     <NA>    NA
    ## 3864          toilet   35     negative     <NA>    NA
    ## 3865           touch   35         <NA>     <NA>    NA
    ## 3866        activity   34         <NA>     <NA>    NA
    ## 3867         adopted   34         <NA>     <NA>    NA
    ## 3868          advice   34        trust     <NA>    NA
    ## 3869          afford   34     positive positive    NA
    ## 3870         consent   34         <NA>     <NA>     2
    ## 3871       convinced   34        trust     <NA>     1
    ## 3872             dad   34         <NA>     <NA>    NA
    ## 3873        disabled   34         fear negative    NA
    ## 3874        disabled   34     negative negative    NA
    ## 3875        disabled   34      sadness negative    NA
    ## 3876             e.g   34         <NA>     <NA>    NA
    ## 3877        expenses   34     negative     <NA>    NA
    ## 3878       financial   34         <NA>     <NA>    NA
    ## 3879         fixture   34     positive     <NA>    NA
    ## 3880             gym   34         <NA>     <NA>    NA
    ## 3881         keeping   34         <NA>     <NA>    NA
    ## 3882          losing   34        anger negative    -3
    ## 3883          losing   34     negative negative    -3
    ## 3884          losing   34      sadness negative    -3
    ## 3885       narrative   34         <NA>     <NA>    NA
    ## 3886         ongoing   34 anticipation     <NA>    NA
    ## 3887              pc   34         <NA>     <NA>    NA
    ## 3888         present   34 anticipation     <NA>    NA
    ## 3889         present   34          joy     <NA>    NA
    ## 3890         present   34     positive     <NA>    NA
    ## 3891         present   34     surprise     <NA>    NA
    ## 3892         present   34        trust     <NA>    NA
    ## 3893          source   34         <NA>     <NA>    NA
    ## 3894           table   34         <NA>     <NA>    NA
    ## 3895       threatens   34         <NA>     <NA>    -2
    ## 3896         tickets   34         <NA>     <NA>    NA
    ## 3897         trouble   34         <NA> negative    -2
    ## 3898             vax   34         <NA>     <NA>    NA
    ## 3899           aaron   33         <NA>     <NA>    NA
    ## 3900            bear   33        anger     <NA>    NA
    ## 3901            bear   33         fear     <NA>    NA
    ## 3902            beta   33         <NA>     <NA>    NA
    ## 3903          cancel   33     negative     <NA>    -1
    ## 3904          cancel   33      sadness     <NA>    -1
    ## 3905         capable   33         <NA> positive     1
    ## 3906         clearly   33         <NA> positive     1
    ## 3907        economic   33         <NA>     <NA>    NA
    ## 3908             egg   33         <NA>     <NA>    NA
    ## 3909           empty   33         <NA>     <NA>    -1
    ## 3910          faster   33         <NA> positive    NA
    ## 3911             hey   33         <NA>     <NA>    NA
    ## 3912          hopper   33         <NA>     <NA>    NA
    ## 3913           india   33         <NA>     <NA>    NA
    ## 3914            land   33     positive     <NA>    NA
    ## 3915               n   33         <NA>     <NA>    NA
    ## 3916     opportunity   33 anticipation     <NA>     2
    ## 3917     opportunity   33     positive     <NA>     2
    ## 3918             ps4   33         <NA>     <NA>    NA
    ## 3919            rate   33         <NA>     <NA>    NA
    ## 3920        register   33         <NA>     <NA>    NA
    ## 3921     responsible   33     positive     <NA>     2
    ## 3922     responsible   33        trust     <NA>     2
    ## 3923           rocky   33         <NA> negative    NA
    ## 3924             sad   33         <NA> negative    -2
    ## 3925        somebody   33         <NA>     <NA>    NA
    ## 3926        supposed   33         <NA>     <NA>    NA
    ## 3927     switzerland   33         <NA>     <NA>    NA
    ## 3928          taught   33        trust     <NA>    NA
    ## 3929            toll   33         <NA> negative    NA
    ## 3930             web   33         <NA>     <NA>    NA
    ## 3931       appointed   32         <NA>     <NA>    NA
    ## 3932             arm   32         <NA>     <NA>    NA
    ## 3933             ass   32     negative     <NA>    -4
    ## 3934      background   32         <NA>     <NA>    NA
    ## 3935             bee   32        anger     <NA>    NA
    ## 3936             bee   32         fear     <NA>    NA
    ## 3937      comparison   32         <NA>     <NA>    NA
    ## 3938         contest   32         <NA>     <NA>    NA
    ## 3939       criminals   32         <NA>     <NA>    -3
    ## 3940              dr   32         <NA>     <NA>    NA
    ## 3941            drug   32         <NA>     <NA>    NA
    ## 3942          handle   32         <NA>     <NA>    NA
    ## 3943          immune   32         <NA>     <NA>     1
    ## 3944            info   32         <NA>     <NA>    NA
    ## 3945         monitor   32         <NA>     <NA>    NA
    ## 3946           mouth   32     surprise     <NA>    NA
    ## 3947           noise   32     negative negative    NA
    ## 3948           pixar   32         <NA>     <NA>    NA
    ## 3949    relationship   32         <NA>     <NA>    NA
    ## 3950           speak   32         <NA>     <NA>    NA
    ## 3951            step   32         <NA>     <NA>    NA
    ## 3952            cash   31        anger     <NA>    NA
    ## 3953            cash   31 anticipation     <NA>    NA
    ## 3954            cash   31         fear     <NA>    NA
    ## 3955            cash   31          joy     <NA>    NA
    ## 3956            cash   31     positive     <NA>    NA
    ## 3957            cash   31        trust     <NA>    NA
    ## 3958      everywhere   31         <NA>     <NA>    NA
    ## 3959           funds   31         <NA>     <NA>    NA
    ## 3960           intro   31         <NA>     <NA>    NA
    ## 3961        japanese   31         <NA>     <NA>    NA
    ## 3962        planning   31 anticipation     <NA>    NA
    ## 3963        planning   31     positive     <NA>    NA
    ## 3964        planning   31        trust     <NA>    NA
    ## 3965            prop   31     positive     <NA>    NA
    ## 3966          script   31     positive     <NA>    NA
    ## 3967      simulation   31         <NA>     <NA>    NA
    ## 3968          stream   31         <NA>     <NA>    NA
    ## 3969         windows   31         <NA>     <NA>    NA
    ## 3970     association   30        trust     <NA>    NA
    ## 3971    billionaires   30         <NA>     <NA>    NA
    ## 3972      blockchain   30         <NA>     <NA>    NA
    ## 3973       budgeting   30         <NA>     <NA>    NA
    ## 3974           clear   30         <NA> positive     1
    ## 3975         concern   30         <NA> negative    NA
    ## 3976    consequences   30         <NA>     <NA>    NA
    ## 3977          credit   30     positive     <NA>    NA
    ## 3978          credit   30        trust     <NA>    NA
    ## 3979         eastern   30         <NA>     <NA>    NA
    ## 3980             fit   30         <NA>     <NA>     1
    ## 3981           heavy   30         <NA>     <NA>    NA
    ## 3982      innovation   30     positive positive     1
    ## 3983            join   30     positive     <NA>     1
    ## 3984        learning   30     positive     <NA>    NA
    ## 3985            lets   30         <NA>     <NA>    NA
    ## 3986           minor   30         <NA>     <NA>    NA
    ## 3987           steal   30        anger negative    -2
    ## 3988           steal   30         fear negative    -2
    ## 3989           steal   30     negative negative    -2
    ## 3990           steal   30      sadness negative    -2
    ## 3991          strain   30         <NA> negative    NA
    ## 3992          travel   30         <NA>     <NA>    NA
    ## 3993             ubi   30         <NA>     <NA>    NA
    ## 3994         younger   30     positive     <NA>    NA
    ## 3995       alexander   29         <NA>     <NA>    NA
    ## 3996         angeles   29         <NA>     <NA>    NA
    ## 3997       attention   29     positive     <NA>    NA
    ## 3998      australian   29         <NA>     <NA>    NA
    ## 3999    corporations   29         <NA>     <NA>    NA
    ## 4000            cult   29         fear     <NA>    NA
    ## 4001            cult   29     negative     <NA>    NA
    ## 4002        elective   29         <NA>     <NA>    NA
    ## 4003           email   29         <NA>     <NA>    NA
    ## 4004           folks   29         <NA>     <NA>    NA
    ## 4005            gene   29         <NA>     <NA>    NA
    ## 4006          looked   29         <NA>     <NA>    NA
    ## 4007            lord   29      disgust     <NA>    NA
    ## 4008            lord   29     negative     <NA>    NA
    ## 4009            lord   29     positive     <NA>    NA
    ## 4010            lord   29        trust     <NA>    NA
    ## 4011             los   29         <NA>     <NA>    NA
    ## 4012            mall   29         <NA>     <NA>    NA
    ## 4013       minecraft   29         <NA>     <NA>    NA
    ## 4014         recover   29         <NA> positive    NA
    ## 4015         savings   29     positive positive    NA
    ## 4016            skin   29         <NA>     <NA>    NA
    ## 4017             spy   29         <NA>     <NA>    NA
    ## 4018            suck   29     negative negative    -3
    ## 4019            tool   29         <NA>     <NA>    NA
    ## 4020          topics   29         <NA>     <NA>    NA
    ## 4021         vaccine   29     positive     <NA>    NA
    ## 4022          videos   29         <NA>     <NA>    NA
    ## 4023         warming   29         <NA>     <NA>    NA
    ## 4024       agreement   28     positive     <NA>     1
    ## 4025       agreement   28        trust     <NA>     1
    ## 4026         ancient   28     negative     <NA>    NA
    ## 4027      confidence   28         fear positive     2
    ## 4028      confidence   28          joy positive     2
    ## 4029      confidence   28     positive positive     2
    ## 4030      confidence   28        trust positive     2
    ## 4031          cutest   28         <NA>     <NA>    NA
    ## 4032             eat   28     positive     <NA>    NA
    ## 4033        endorses   28         <NA> positive     2
    ## 4034            ford   28         <NA>     <NA>    NA
    ## 4035      generation   28         <NA>     <NA>    NA
    ## 4036              gf   28         <NA>     <NA>    NA
    ## 4037       nightmare   28         fear negative    NA
    ## 4038       nightmare   28     negative negative    NA
    ## 4039   technological   28         <NA>     <NA>    NA
    ## 4040         traffic   28         <NA>     <NA>    NA
    ## 4041              tx   28         <NA>     <NA>    NA
    ## 4042          unique   28     positive     <NA>    NA
    ## 4043          unique   28     surprise     <NA>    NA
    ## 4044           voice   28         <NA>     <NA>    NA
    ## 4045         zealand   28         <NA>     <NA>    NA
    ## 4046        articles   27         <NA>     <NA>    NA
    ## 4047            bowl   27         <NA>     <NA>    NA
    ## 4048       complaint   27        anger negative    NA
    ## 4049       complaint   27     negative negative    NA
    ## 4050        detected   27         <NA>     <NA>    NA
    ## 4051       discovery   27     positive     <NA>    NA
    ## 4052           faith   27 anticipation positive     1
    ## 4053           faith   27          joy positive     1
    ## 4054           faith   27     positive positive     1
    ## 4055           faith   27        trust positive     1
    ## 4056         genetic   27         <NA>     <NA>    NA
    ## 4057            hasn   27         <NA>     <NA>    NA
    ## 4058              mw   27         <NA>     <NA>    NA
    ## 4059              ny   27         <NA>     <NA>    NA
    ## 4060            path   27         <NA>     <NA>    NA
    ## 4061            pete   27         <NA>     <NA>    NA
    ## 4062         quantum   27         <NA>     <NA>    NA
    ## 4063             ray   27         <NA>     <NA>    NA
    ## 4064           react   27        anger     <NA>    NA
    ## 4065           react   27         fear     <NA>    NA
    ## 4066          refund   27         <NA> positive    NA
    ## 4067       resulting   27         <NA>     <NA>    NA
    ## 4068           throw   27         <NA>     <NA>    NA
    ## 4069          titles   27         <NA>     <NA>    NA
    ## 4070    unemployment   27         <NA>     <NA>    -2
    ## 4071       buttigieg   26         <NA>     <NA>    NA
    ## 4072          laptop   26         <NA>     <NA>    NA
    ## 4073      mainstream   26         <NA>     <NA>    NA
    ## 4074        medicine   26         <NA>     <NA>    NA
    ## 4075          moving   26         <NA>     <NA>    NA
    ## 4076      nomination   26     positive     <NA>    NA
    ## 4077             pop   26     negative     <NA>    NA
    ## 4078             pop   26     surprise     <NA>    NA
    ## 4079        resident   26     positive     <NA>    NA
    ## 4080     soundtracks   26         <NA>     <NA>    NA
    ## 4081           style   26         <NA>     <NA>    NA
    ## 4082        treating   26         <NA>     <NA>    NA
    ## 4083               w   26         <NA>     <NA>    NA
    ## 4084       wondering   26         <NA>     <NA>    NA
    ## 4085          assume   25         <NA>     <NA>    NA
    ## 4086          bigger   25         <NA>     <NA>    NA
    ## 4087           blade   25         <NA>     <NA>    NA
    ## 4088          brains   25     positive     <NA>    NA
    ## 4089        catching   25         <NA>     <NA>    NA
    ## 4090           crash   25         fear negative    -2
    ## 4091           crash   25     negative negative    -2
    ## 4092           crash   25      sadness negative    -2
    ## 4093           crash   25     surprise negative    -2
    ## 4094              fb   25         <NA>     <NA>    NA
    ## 4095           fixed   25        trust     <NA>    NA
    ## 4096         forcing   25         <NA>     <NA>    NA
    ## 4097             irl   25         <NA>     <NA>    NA
    ## 4098           limit   25         <NA> negative    NA
    ## 4099          mirror   25         <NA>     <NA>    NA
    ## 4100      predicting   25         <NA>     <NA>    NA
    ## 4101      recovering   25         <NA>     <NA>    NA
    ## 4102           roads   25         <NA>     <NA>    NA
    ## 4103            scam   25         <NA> negative    -2
    ## 4104         subject   25     negative     <NA>    NA
    ## 4105            tips   25         <NA>     <NA>    NA
    ## 4106          artist   24         <NA>     <NA>    NA
    ## 4107         balance   24     positive     <NA>    NA
    ## 4108            cant   24         <NA>     <NA>    NA
    ## 4109         complex   24         <NA> negative    NA
    ## 4110     environment   24         <NA>     <NA>    NA
    ## 4111             gop   24         <NA>     <NA>    NA
    ## 4112         include   24     positive     <NA>    NA
    ## 4113         overall   24         <NA>     <NA>    NA
    ## 4114         plastic   24         <NA>     <NA>    NA
    ## 4115     predictions   24         <NA>     <NA>    NA
    ## 4116        refusing   24         <NA> negative    -2
    ## 4117             rid   24         <NA>     <NA>    NA
    ## 4118        strategy   24         <NA>     <NA>    NA
    ## 4119           taxes   24         <NA>     <NA>    NA
    ## 4120          trials   24         <NA>     <NA>    NA
    ## 4121         ukraine   24         <NA>     <NA>    NA
    ## 4122         usually   24         <NA>     <NA>    NA
    ## 4123          vision   24 anticipation     <NA>     1
    ## 4124          vision   24     positive     <NA>     1
    ## 4125            wage   24         <NA>     <NA>    NA
    ## 4126         weather   24         <NA>     <NA>    NA
    ## 4127        yanggang   24         <NA>     <NA>    NA
    ## 4128        affected   23         <NA>     <NA>    -1
    ## 4129           blues   23         fear     <NA>    NA
    ## 4130           blues   23     negative     <NA>    NA
    ## 4131           blues   23      sadness     <NA>    NA
    ## 4132       computing   23         <NA>     <NA>    NA
    ## 4133          easier   23         <NA> positive    NA
    ## 4134     endorsement   23         <NA> positive     2
    ## 4135         focused   23         <NA>     <NA>     2
    ## 4136            gate   23        trust     <NA>    NA
    ## 4137         lindsey   23         <NA>     <NA>    NA
    ## 4138       marketing   23         <NA>     <NA>    NA
    ## 4139        pregnant   23         <NA>     <NA>    NA
    ## 4140         roundup   23         <NA>     <NA>    NA
    ## 4141          silent   23         <NA> positive    NA
    ## 4142         stomach   23      disgust     <NA>    NA
    ## 4143            sued   23         <NA> negative    NA
    ## 4144           tools   23         <NA>     <NA>    NA
    ## 4145          turing   23         <NA>     <NA>    NA
    ## 4146            wish   23         <NA>     <NA>     1
    ## 4147            apps   22         <NA>     <NA>    NA
    ## 4148            bugs   22         <NA> negative    NA
    ## 4149      collection   22         <NA>     <NA>    NA
    ## 4150           couch   22      sadness     <NA>    NA
    ## 4151       country's   22         <NA>     <NA>    NA
    ## 4152          custom   22         <NA>     <NA>    NA
    ## 4153        defeated   22     negative positive    -2
    ## 4154        defeated   22      sadness positive    -2
    ## 4155       electable   22         <NA>     <NA>    NA
    ## 4156            fell   22     negative negative    NA
    ## 4157            fell   22      sadness negative    NA
    ## 4158            file   22         <NA>     <NA>    NA
    ## 4159             fix   22         <NA>     <NA>    NA
    ## 4160            gift   22 anticipation     <NA>     2
    ## 4161            gift   22          joy     <NA>     2
    ## 4162            gift   22     positive     <NA>     2
    ## 4163            gift   22     surprise     <NA>     2
    ## 4164          graham   22         <NA>     <NA>    NA
    ## 4165              gt   22         <NA>     <NA>    NA
    ## 4166             hip   22         <NA>     <NA>    NA
    ## 4167               k   22         <NA>     <NA>    NA
    ## 4168            mode   22         <NA>     <NA>    NA
    ## 4169         respond   22         <NA>     <NA>    NA
    ## 4170           scare   22        anger negative    -2
    ## 4171           scare   22 anticipation negative    -2
    ## 4172           scare   22         fear negative    -2
    ## 4173           scare   22     negative negative    -2
    ## 4174           scare   22     surprise negative    -2
    ## 4175         section   22         <NA>     <NA>    NA
    ## 4176          steven   22         <NA>     <NA>    NA
    ## 4177           stuff   22         <NA>     <NA>    NA
    ## 4178          theory   22 anticipation     <NA>    NA
    ## 4179          theory   22        trust     <NA>    NA
    ## 4180      thunberg's   22         <NA>     <NA>    NA
    ## 4181              vr   22         <NA>     <NA>    NA
    ## 4182          window   22         <NA>     <NA>    NA
    ## 4183             a.i   21         <NA>     <NA>    NA
    ## 4184      additional   21         <NA>     <NA>    NA
    ## 4185          africa   21         <NA>     <NA>    NA
    ## 4186           audio   21         <NA>     <NA>    NA
    ## 4187          cities   21         <NA>     <NA>    NA
    ## 4188             cod   21         <NA>     <NA>    NA
    ## 4189      comparable   21         <NA>     <NA>    NA
    ## 4190       completed   21         <NA>     <NA>    NA
    ## 4191        customer   21     positive     <NA>    NA
    ## 4192            devs   21         <NA>     <NA>    NA
    ## 4193        electric   21          joy     <NA>    NA
    ## 4194        electric   21     positive     <NA>    NA
    ## 4195        electric   21     surprise     <NA>    NA
    ## 4196              er   21         <NA>     <NA>    NA
    ## 4197     financially   21         <NA>     <NA>    NA
    ## 4198            folk   21         <NA>     <NA>    NA
    ## 4199           greek   21         <NA>     <NA>    NA
    ## 4200          limits   21         <NA> negative    -1
    ## 4201       operators   21         <NA>     <NA>    NA
    ## 4202       returning   21         <NA>     <NA>    NA
    ## 4203         setting   21         <NA>     <NA>    NA
    ## 4204           uncle   21         <NA>     <NA>    NA
    ## 4205            vice   21     negative negative    NA
    ## 4206          watson   21         <NA>     <NA>    NA
    ## 4207          aliens   20         <NA>     <NA>    NA
    ## 4208       assistant   20         <NA>     <NA>    NA
    ## 4209            boss   20         <NA>     <NA>    NA
    ## 4210           buddy   20 anticipation     <NA>    NA
    ## 4211           buddy   20          joy     <NA>    NA
    ## 4212           buddy   20     positive     <NA>    NA
    ## 4213           buddy   20        trust     <NA>    NA
    ## 4214          danger   20         fear negative    -2
    ## 4215          danger   20     negative negative    -2
    ## 4216          danger   20      sadness negative    -2
    ## 4217       diagnosis   20 anticipation     <NA>    NA
    ## 4218       diagnosis   20         fear     <NA>    NA
    ## 4219       diagnosis   20     negative     <NA>    NA
    ## 4220       diagnosis   20        trust     <NA>    NA
    ## 4221          fields   20         <NA>     <NA>    NA
    ## 4222             hop   20         <NA>     <NA>    NA
    ## 4223          kitten   20          joy     <NA>    NA
    ## 4224          kitten   20     positive     <NA>    NA
    ## 4225          kitten   20        trust     <NA>    NA
    ## 4226            mini   20         <NA>     <NA>    NA
    ## 4227         nuclear   20         <NA>     <NA>    NA
    ## 4228          nvidia   20         <NA>     <NA>    NA
    ## 4229         ontario   20         <NA>     <NA>    NA
    ## 4230          retail   20         <NA>     <NA>    NA
    ## 4231            roth   20         <NA>     <NA>    NA
    ## 4232         seeking   20         <NA>     <NA>    NA
    ## 4233         sticker   20         <NA>     <NA>    NA
    ## 4234        trailers   20         <NA>     <NA>    NA
    ## 4235          useful   20         <NA> positive     2
    ## 4236       apartment   19         <NA>     <NA>    NA
    ## 4237         channel   19         <NA>     <NA>    NA
    ## 4238            dawn   19 anticipation positive    NA
    ## 4239            dawn   19          joy positive    NA
    ## 4240            dawn   19     positive positive    NA
    ## 4241            dawn   19     surprise positive    NA
    ## 4242            dawn   19        trust positive    NA
    ## 4243          device   19         <NA>     <NA>    NA
    ## 4244    electability   19         <NA>     <NA>    NA
    ## 4245         exhibit   19         <NA>     <NA>    NA
    ## 4246           hired   19         <NA>     <NA>    NA
    ## 4247           ideas   19         <NA>     <NA>    NA
    ## 4248     inspiration   19 anticipation positive     2
    ## 4249     inspiration   19          joy positive     2
    ## 4250     inspiration   19     positive positive     2
    ## 4251      particular   19         <NA>     <NA>    NA
    ## 4252          region   19         <NA>     <NA>    NA
    ## 4253        reminded   19         <NA>     <NA>    NA
    ## 4254        scenario   19         <NA>     <NA>    NA
    ## 4255         schools   19         <NA>     <NA>    NA
    ## 4256           solve   19         <NA>     <NA>     1
    ## 4257           terms   19         <NA>     <NA>    NA
    ## 4258           avian   18         <NA>     <NA>    NA
    ## 4259             bbc   18         <NA>     <NA>    NA
    ## 4260            buys   18         <NA>     <NA>    NA
    ## 4261     celebrating   18 anticipation     <NA>     3
    ## 4262     celebrating   18          joy     <NA>     3
    ## 4263     celebrating   18     positive     <NA>     3
    ## 4264        chairman   18     positive     <NA>    NA
    ## 4265        chairman   18        trust     <NA>    NA
    ## 4266         choices   18         <NA>     <NA>    NA
    ## 4267        confused   18         <NA> negative    -2
    ## 4268          except   18         <NA>     <NA>    NA
    ## 4269         hostile   18        anger negative    -2
    ## 4270         hostile   18      disgust negative    -2
    ## 4271         hostile   18         fear negative    -2
    ## 4272         hostile   18     negative negative    -2
    ## 4273              ma   18         <NA>     <NA>    NA
    ## 4274           memes   18         <NA>     <NA>    NA
    ## 4275              nh   18         <NA>     <NA>    NA
    ## 4276           parts   18         <NA>     <NA>    NA
    ## 4277       portrayed   18         <NA>     <NA>    NA
    ## 4278          raises   18         <NA>     <NA>    NA
    ## 4279        spending   18         <NA>     <NA>    NA
    ## 4280        thailand   18         <NA>     <NA>    NA
    ## 4281        tracking   18         <NA>     <NA>    NA
    ## 4282             aoc   17         <NA>     <NA>    NA
    ## 4283     authorities   17         <NA>     <NA>    NA
    ## 4284    breakthrough   17         <NA> positive     3
    ## 4285           catch   17     surprise     <NA>    NA
    ## 4286           cheap   17     negative negative    NA
    ## 4287       concerned   17         fear negative    NA
    ## 4288       concerned   17      sadness negative    NA
    ## 4289              iw   17         <NA>     <NA>    NA
    ## 4290            jazz   17         <NA>     <NA>    NA
    ## 4291            lego   17         <NA>     <NA>    NA
    ## 4292       listening   17         <NA>     <NA>    NA
    ## 4293        material   17         <NA>     <NA>    NA
    ## 4294             ops   17         <NA>     <NA>    NA
    ## 4295            spec   17         <NA>     <NA>    NA
    ## 4296         startup   17         <NA>     <NA>    NA
    ## 4297         suggest   17        trust     <NA>    NA
    ## 4298           swine   17      disgust     <NA>    NA
    ## 4299           swine   17     negative     <NA>    NA
    ## 4300            tale   17     positive     <NA>    NA
    ## 4301             td2   17         <NA>     <NA>    NA
    ## 4302          urgent   17 anticipation negative    -1
    ## 4303          urgent   17         fear negative    -1
    ## 4304          urgent   17     negative negative    -1
    ## 4305          urgent   17     surprise negative    -1
    ## 4306         artists   16         <NA>     <NA>    NA
    ## 4307         compete   16         <NA>     <NA>    NA
    ## 4308     competition   16 anticipation     <NA>    NA
    ## 4309     competition   16     negative     <NA>    NA
    ## 4310      conditions   16         <NA>     <NA>    NA
    ## 4311         drivers   16         <NA>     <NA>    NA
    ## 4312         effects   16         <NA>     <NA>    NA
    ## 4313        employer   16         <NA>     <NA>    NA
    ## 4314              en   16         <NA>     <NA>    NA
    ## 4315        extended   16         <NA>     <NA>    NA
    ## 4316            hang   16         <NA> negative    NA
    ## 4317             mix   16         <NA>     <NA>    NA
    ## 4318         musical   16        anger     <NA>    NA
    ## 4319         musical   16 anticipation     <NA>    NA
    ## 4320         musical   16          joy     <NA>    NA
    ## 4321         musical   16     positive     <NA>    NA
    ## 4322         musical   16      sadness     <NA>    NA
    ## 4323         musical   16     surprise     <NA>    NA
    ## 4324         musical   16        trust     <NA>    NA
    ## 4325           naked   16         <NA>     <NA>    NA
    ## 4326         product   16         <NA>     <NA>    NA
    ## 4327     programming   16         <NA>     <NA>    NA
    ## 4328      retirement   16 anticipation     <NA>    NA
    ## 4329      retirement   16         fear     <NA>    NA
    ## 4330      retirement   16          joy     <NA>    NA
    ## 4331      retirement   16     negative     <NA>    NA
    ## 4332      retirement   16     positive     <NA>    NA
    ## 4333      retirement   16      sadness     <NA>    NA
    ## 4334      retirement   16        trust     <NA>    NA
    ## 4335     suggestions   16         <NA>     <NA>    NA
    ## 4336         toronto   16         <NA>     <NA>    NA
    ## 4337        anywhere   15         <NA>     <NA>    NA
    ## 4338          clause   15         <NA>     <NA>    NA
    ## 4339    construction   15         <NA>     <NA>    NA
    ## 4340      controlled   15         <NA>     <NA>    NA
    ## 4341           dirty   15      disgust negative    -2
    ## 4342           dirty   15     negative negative    -2
    ## 4343           drugs   15         <NA>     <NA>    NA
    ## 4344            edit   15         <NA>     <NA>    NA
    ## 4345       evolution   15     positive     <NA>    NA
    ## 4346            flat   15         <NA>     <NA>    NA
    ## 4347            gear   15     positive     <NA>    NA
    ## 4348     investments   15         <NA>     <NA>    NA
    ## 4349            mers   15         <NA>     <NA>    NA
    ## 4350          mobile   15 anticipation     <NA>    NA
    ## 4351           novel   15         <NA>     <NA>     2
    ## 4352         numbers   15     positive     <NA>    NA
    ## 4353       obviously   15         <NA>     <NA>    NA
    ## 4354     personality   15         <NA>     <NA>    NA
    ## 4355      principles   15         <NA>     <NA>    NA
    ## 4356        property   15         <NA>     <NA>    NA
    ## 4357      references   15         <NA>     <NA>    NA
    ## 4358       resources   15          joy     <NA>    NA
    ## 4359       resources   15     positive     <NA>    NA
    ## 4360       resources   15        trust     <NA>    NA
    ## 4361     significant   15         <NA> positive     1
    ## 4362           smart   15         <NA> positive     1
    ## 4363     application   14         <NA>     <NA>    NA
    ## 4364        champion   14 anticipation positive    NA
    ## 4365        champion   14          joy positive    NA
    ## 4366        champion   14     positive positive    NA
    ## 4367        champion   14        trust positive    NA
    ## 4368         devices   14         <NA>     <NA>    NA
    ## 4369      employment   14         <NA>     <NA>    NA
    ## 4370           error   14     negative negative    -2
    ## 4371           error   14      sadness negative    -2
    ## 4372        feedback   14         <NA>     <NA>    NA
    ## 4373           field   14         <NA>     <NA>    NA
    ## 4374        graduate   14         <NA>     <NA>    NA
    ## 4375         hawkins   14         <NA>     <NA>    NA
    ## 4376        legality   14         <NA>     <NA>    NA
    ## 4377     medications   14         <NA>     <NA>    NA
    ## 4378         mention   14         <NA>     <NA>    NA
    ## 4379              mi   14         <NA>     <NA>    NA
    ## 4380         patient   14 anticipation positive    NA
    ## 4381         patient   14     positive positive    NA
    ## 4382        portrait   14         <NA>     <NA>    NA
    ## 4383          square   14         <NA>     <NA>    NA
    ## 4384           sweet   14 anticipation positive     2
    ## 4385           sweet   14          joy positive     2
    ## 4386           sweet   14     positive positive     2
    ## 4387           sweet   14     surprise positive     2
    ## 4388           sweet   14        trust positive     2
    ## 4389           xpost   14         <NA>     <NA>    NA
    ## 4390              al   13         <NA>     <NA>    NA
    ## 4391       awareness   13         <NA>     <NA>    NA
    ## 4392        benefits   13         <NA> positive     2
    ## 4393             bot   13         <NA>     <NA>    NA
    ## 4394          builds   13         <NA>     <NA>    NA
    ## 4395  cryptocurrency   13         <NA>     <NA>    NA
    ## 4396           items   13         <NA>     <NA>    NA
    ## 4397         partner   13     positive     <NA>    NA
    ## 4398         perform   13         <NA>     <NA>    NA
    ## 4399          sample   13         <NA>     <NA>    NA
    ## 4400            sars   13         <NA>     <NA>    NA
    ## 4401           screw   13         <NA>     <NA>    NA
    ## 4402            sort   13         <NA>     <NA>    NA
    ## 4403          supply   13     positive     <NA>    NA
    ## 4404     threatening   13        anger negative    -2
    ## 4405     threatening   13      disgust negative    -2
    ## 4406     threatening   13         fear negative    -2
    ## 4407     threatening   13     negative negative    -2
    ## 4408           debit   12         <NA>     <NA>    NA
    ## 4409        diseases   12         <NA>     <NA>    NA
    ## 4410        eligible   12     positive     <NA>    NA
    ## 4411          launch   12 anticipation     <NA>    NA
    ## 4412          launch   12     positive     <NA>    NA
    ## 4413         matched   12         <NA>     <NA>    NA
    ## 4414      operations   12         <NA>     <NA>    NA
    ## 4415              wa   12         <NA>     <NA>    NA
    ## 4416          annual   11         <NA>     <NA>    NA
    ## 4417            apes   11         <NA>     <NA>    NA
    ## 4418    applications   11         <NA>     <NA>    NA
    ## 4419         comfort   11 anticipation positive     2
    ## 4420         comfort   11          joy positive     2
    ## 4421         comfort   11     positive positive     2
    ## 4422         comfort   11        trust positive     2
    ## 4423          cooper   11         <NA>     <NA>    NA
    ## 4424      disability   11     negative     <NA>    NA
    ## 4425      disability   11      sadness     <NA>    NA
    ## 4426         endless   11        anger     <NA>    NA
    ## 4427         endless   11         fear     <NA>    NA
    ## 4428         endless   11          joy     <NA>    NA
    ## 4429         endless   11     negative     <NA>    NA
    ## 4430         endless   11     positive     <NA>    NA
    ## 4431         endless   11      sadness     <NA>    NA
    ## 4432         endless   11        trust     <NA>    NA
    ## 4433      enviroment   11         <NA>     <NA>    NA
    ## 4434             fee   11        anger     <NA>    NA
    ## 4435             fee   11     negative     <NA>    NA
    ## 4436            h1n1   11         <NA>     <NA>    NA
    ## 4437             hoa   11         <NA>     <NA>    NA
    ## 4438        landlord   11         <NA>     <NA>    NA
    ## 4439         options   11         <NA>     <NA>    NA
    ## 4440             ost   11         <NA>     <NA>    NA
    ## 4441         podcast   11         <NA>     <NA>    NA
    ## 4442          rental   11         <NA>     <NA>    NA
    ## 4443        sentient   11         <NA>     <NA>    NA
    ## 4444      suggestion   11         <NA>     <NA>    NA
    ## 4445           teach   11          joy     <NA>    NA
    ## 4446           teach   11     positive     <NA>    NA
    ## 4447           teach   11     surprise     <NA>    NA
    ## 4448           teach   11        trust     <NA>    NA
    ## 4449          tenant   11     positive     <NA>    NA
    ## 4450          trends   11         <NA>     <NA>    NA
    ## 4451          advise   10     positive     <NA>    NA
    ## 4452          advise   10        trust     <NA>    NA
    ## 4453          animal   10         <NA>     <NA>    NA
    ## 4454        automate   10         <NA>     <NA>    NA
    ## 4455             bug   10      disgust negative    NA
    ## 4456             bug   10         fear negative    NA
    ## 4457             bug   10     negative negative    NA
    ## 4458     congressman   10        trust     <NA>    NA
    ## 4459         deposit   10         <NA>     <NA>    NA
    ## 4460          effort   10     positive     <NA>    NA
    ## 4461           firms   10         <NA>     <NA>    NA
    ## 4462            grad   10         <NA>     <NA>    NA
    ## 4463              hr   10         <NA>     <NA>    NA
    ## 4464              mo   10         <NA>     <NA>    NA
    ## 4465        platform   10         <NA>     <NA>    NA
    ## 4466      programmed   10         <NA>     <NA>    NA
    ## 4467        remedies   10         <NA>     <NA>    NA
    ## 4468         screwed   10        anger negative    -2
    ## 4469         screwed   10     negative negative    -2
    ## 4470        southern   10         <NA>     <NA>    NA
    ## 4471   contributions    9         <NA>     <NA>    NA
    ## 4472              fl    9         <NA>     <NA>    NA
    ## 4473            fmla    9         <NA>     <NA>    NA
    ## 4474            fund    9         <NA>     <NA>    NA
    ## 4475             ibm    9         <NA>     <NA>    NA
    ## 4476          michel    9         <NA>     <NA>    NA
    ## 4477         realism    9         <NA>     <NA>    NA
    ## 4478       selection    9         <NA>     <NA>    NA
    ## 4479       workplace    9         <NA>     <NA>    NA
    ## 4480         chatbot    8         <NA>     <NA>    NA
    ## 4481            east    8         <NA>     <NA>    NA
    ## 4482      electronic    8         <NA>     <NA>    NA
    ## 4483          estate    8         <NA>     <NA>    NA
    ## 4484         harvard    8         <NA>     <NA>    NA
    ## 4485             ira    8         <NA>     <NA>    NA
    ## 4486           solar    8         <NA>     <NA>    NA
    ## 4487             ais    7         <NA>     <NA>    NA
    ## 4488        detector    7         <NA>     <NA>    NA
    ## 4489          drones    7         <NA> negative    NA
    ## 4490        emotions    7         <NA>     <NA>    NA
    ## 4491           frame    7         <NA>     <NA>    NA
    ## 4492             hsa    7         <NA>     <NA>    NA
    ## 4493            visa    7         <NA>     <NA>    NA
    ## 4494          weiwei    7         <NA>     <NA>    NA
    ## 4495          enviro    6         <NA>     <NA>    NA
    ## 4496         expense    6         <NA>     <NA>    NA
    ## 4497       frontline    6         <NA>     <NA>    NA
    ## 4498          johnny    6         <NA>     <NA>    NA
    ## 4499       molecular    6         <NA>     <NA>    NA
    ## 4500       featuring    5         <NA>     <NA>    NA
    ## 4501              il    5         <NA>     <NA>    NA
    ## 4502     interactive    5         <NA>     <NA>    NA
    ## 4503           kitty    5         <NA>     <NA>    NA
    ## 4504          learns    5         <NA>     <NA>    NA
    ## 4505             rap    5         <NA>     <NA>    NA
    ## 4506             wei    5         <NA>     <NA>    NA
    ## 4507              du    4         <NA>     <NA>    NA
    ## 4508       engineers    4         <NA>     <NA>    NA
    ## 4509            feat    4 anticipation positive    NA
    ## 4510            feat    4          joy positive    NA
    ## 4511            feat    4     positive positive    NA
    ## 4512            feat    4     surprise positive    NA
    ## 4513            j'ai    4         <NA>     <NA>    NA
    ## 4514              se    3         <NA>     <NA>    NA
    ## 4515         spotify    3         <NA>     <NA>    NA
    ## 4516            j'en    2         <NA>     <NA>    NA
    ## 4517           marre    2         <NA>     <NA>    NA
    ## 4518             pas    2         <NA>     <NA>    NA
    ## 4519            pego    2         <NA>     <NA>    NA
    ## 4520        symphony    2 anticipation     <NA>    NA
    ## 4521        symphony    2          joy     <NA>    NA
    ## 4522        symphony    2     positive     <NA>    NA
    ## 4523              te    2         <NA>     <NA>    NA
    ## 4524             pup    1         <NA>     <NA>    NA
    ## 4525            serj    1         <NA>     <NA>    NA
    ## 4526         tankian    1         <NA>     <NA>    NA
    ## 4527             jav    0         <NA>     <NA>    NA

``` r
bing_word_counts <- sent_reviews %>%  
  filter(!is.na(bing))

bing_word_counts %>%  
  filter(mean > 20) %>%  
  mutate(mean = ifelse(bing == "negative", -mean, mean)) %>%  
  mutate(word = reorder(word, mean)) %>%  
  ggplot(aes(word, mean, fill = bing)) +  
  geom_col() +  coord_flip() +  
  labs(y = "number of hits")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

``` r
df4<-cleaned_tokens %>%
  mutate(count=1)

df5<-ddply(df4,c("word"),numcolwise(sum))

df6<-df5 %>%    
  spread(key = word, value = count)

df6[is.na(df6)] <- 0
```

## Machine Learning

10. Run logistic Regression to see any relation existing.

Some of the posts has over 10,000 comments, to have a better view, data
has been scaled for this part.

From the result, it can be observed that some of the words will increase
the number of comments of the post, but this result is not good enough.

``` r
df6$num_comments<-df6$num_comments/1000

df6$num_comments<-ifelse(df6$num_comments>10,10,df6$num_comments)
  
df6$num_comments<-floor(df6$num_comments)

bank.df <- df6
bank.df$SalePrice<-df6$num_comments

selected.var <- c(1:length(df6))

set.seed(2)
train.index <- sample(c(1:dim(bank.df)[1]), dim(bank.df)[1]*0.6)
train.df <- bank.df[train.index, selected.var]
valid.df <- bank.df[-train.index, selected.var]

car.lm <- lm(num_comments ~ ., data = train.df)

options(scipen = 999)
summary(car.lm)   # Get model summary
```

    ## 
    ## Call:
    ## lm(formula = num_comments ~ ., data = train.df)
    ## 
    ## Residuals:
    ## ALL 2070 residuals are 0: no residual degrees of freedom!
    ## 
    ## Coefficients: (1382 not defined because of singularities)
    ##                               Estimate Std. Error t value Pr(>|t|)
    ## (Intercept)      0.0000000000004480879         NA      NA       NA
    ## a.i                                 NA         NA      NA       NA
    ## aaron           -0.0000000000000378517         NA      NA       NA
    ## abandoned                           NA         NA      NA       NA
    ## ability                             NA         NA      NA       NA
    ## able             0.1111111111111019040         NA      NA       NA
    ## absolute         0.0833333333333030196         NA      NA       NA
    ## absolutely       0.0789473684210456883         NA      NA       NA
    ## abuse                               NA         NA      NA       NA
    ## ac              -0.0000000000000372718         NA      NA       NA
    ## academy          0.4117647058823473705         NA      NA       NA
    ## accept           0.3000000000000336287         NA      NA       NA
    ## access                              NA         NA      NA       NA
    ## accident                            NA         NA      NA       NA
    ## accidentally     0.1818181818182053322         NA      NA       NA
    ## according        0.0909090909090544130         NA      NA       NA
    ## account          0.0740740740740632592         NA      NA       NA
    ## accounts         0.1249999999999815703         NA      NA       NA
    ## accuracy                            NA         NA      NA       NA
    ## accurate         0.2307692307691474320         NA      NA       NA
    ## accurately                          NA         NA      NA       NA
    ## accusations     -0.0000000000001085888         NA      NA       NA
    ## accused                             NA         NA      NA       NA
    ## across           0.1639344262295009957         NA      NA       NA
    ## act                                 NA         NA      NA       NA
    ## acting           0.1499999999999934996         NA      NA       NA
    ## action           0.0749999999999976796         NA      NA       NA
    ## actions          0.0555555555555377056         NA      NA       NA
    ## activist                            NA         NA      NA       NA
    ## activity        -0.0000000000000254264         NA      NA       NA
    ## actor            0.1562500000000136557         NA      NA       NA
    ## actors           0.1851851851851822883         NA      NA       NA
    ## actress          0.5555555555555636849         NA      NA       NA
    ## actual                              NA         NA      NA       NA
    ## actually                            NA         NA      NA       NA
    ## ad               0.0312500000000173889         NA      NA       NA
    ## adam             0.1052631578947037655         NA      NA       NA
    ## adaptation       0.2727272727271637942         NA      NA       NA
    ## add              0.2173913043477873619         NA      NA       NA
    ## added                               NA         NA      NA       NA
    ## additional      -0.0000000000000390770         NA      NA       NA
    ## address          0.2105263157894323445         NA      NA       NA
    ## administration   0.5454545454545663974         NA      NA       NA
    ## admit                               NA         NA      NA       NA
    ## admitted         0.1818181818181367482         NA      NA       NA
    ## adopted                             NA         NA      NA       NA
    ## ads                                 NA         NA      NA       NA
    ## adult            0.1052631578947018642         NA      NA       NA
    ## adults          -0.0000000000000316705         NA      NA       NA
    ## advance          0.3571428571427439080         NA      NA       NA
    ## advanced         0.0937500000000147382         NA      NA       NA
    ## advertising                         NA         NA      NA       NA
    ## advice                              NA         NA      NA       NA
    ## advise          -0.0000000000000354345         NA      NA       NA
    ## affect                              NA         NA      NA       NA
    ## affected        -0.0000000000000484340         NA      NA       NA
    ## affecting       -0.0000000000001225031         NA      NA       NA
    ## afford                              NA         NA      NA       NA
    ## afraid                              NA         NA      NA       NA
    ## africa                              NA         NA      NA       NA
    ## african          0.1111111111111312139         NA      NA       NA
    ## age              0.0921052631578974657         NA      NA       NA
    ## aged                                NA         NA      NA       NA
    ## agency           0.1999999999999308997         NA      NA       NA
    ## agenda                              NA         NA      NA       NA
    ## agent            0.0476190476190595791         NA      NA       NA
    ## agents                              NA         NA      NA       NA
    ## ago              0.1052631578947426649         NA      NA       NA
    ## agree            0.0476190476190230944         NA      NA       NA
    ## agreed                              NA         NA      NA       NA
    ## agreement                           NA         NA      NA       NA
    ## ahead                               NA         NA      NA       NA
    ## ai                                  NA         NA      NA       NA
    ## `ai's`                              NA         NA      NA       NA
    ## `ain't`         -0.0000000000001196366         NA      NA       NA
    ## air              0.0588235294117486623         NA      NA       NA
    ## airport          0.3846153846153630984         NA      NA       NA
    ## ais             -0.0000000000000414641         NA      NA       NA
    ## aka                                 NA         NA      NA       NA
    ## al              -0.0000000000000928098         NA      NA       NA
    ## album                               NA         NA      NA       NA
    ## albums           0.2727272727272549990         NA      NA       NA
    ## alcohol          0.0909090909090467247         NA      NA       NA
    ## alert            0.0769230769230447031         NA      NA       NA
    ## alex                                NA         NA      NA       NA
    ## alexander       -0.0000000000000342738         NA      NA       NA
    ## algorithm                           NA         NA      NA       NA
    ## algorithms       0.0816326530612145468         NA      NA       NA
    ## ali             -0.0000000000000310644         NA      NA       NA
    ## alice            0.0666666666666943658         NA      NA       NA
    ## alien                               NA         NA      NA       NA
    ## aliens                              NA         NA      NA       NA
    ## alive                               NA         NA      NA       NA
    ## alleged                             NA         NA      NA       NA
    ## allegedly                           NA         NA      NA       NA
    ## allow            0.2000000000000136391         NA      NA       NA
    ## allowed          0.1176470588235296738         NA      NA       NA
    ## allowing         0.1428571428570840907         NA      NA       NA
    ## allows           0.0952380952380680462         NA      NA       NA
    ## almost           0.1052631578947351015         NA      NA       NA
    ## alone            0.0370370370370210408         NA      NA       NA
    ## along            0.0740740740740566950         NA      NA       NA
    ## already          0.0813953488372247402         NA      NA       NA
    ## also             0.0584795321637429505         NA      NA       NA
    ## alt              0.2631578947368060639         NA      NA       NA
    ## alter            0.0909090909090204680         NA      NA       NA
    ## alternative                         NA         NA      NA       NA
    ## always           0.0943396226415237071         NA      NA       NA
    ## ama              0.2777777777777807322         NA      NA       NA
    ## amazing                             NA         NA      NA       NA
    ## amazon                              NA         NA      NA       NA
    ## america                             NA         NA      NA       NA
    ## american         0.0641025641025632364         NA      NA       NA
    ## americans        0.0970873786407654427         NA      NA       NA
    ## amid             0.2121212121212295854         NA      NA       NA
    ## among            0.0322580645161031701         NA      NA       NA
    ## amount          -0.0000000000000341586         NA      NA       NA
    ## amp                                 NA         NA      NA       NA
    ## analysis         0.2439024390244153295         NA      NA       NA
    ## analytica        0.1818181818181146547         NA      NA       NA
    ## ancient                             NA         NA      NA       NA
    ## andrew                              NA         NA      NA       NA
    ## angeles          0.0000000000000006165         NA      NA       NA
    ## angry                               NA         NA      NA       NA
    ## animal          -0.0000000000000250002         NA      NA       NA
    ## animals                             NA         NA      NA       NA
    ## animated                            NA         NA      NA       NA
    ## animation        0.1874999999999626132         NA      NA       NA
    ## anime                               NA         NA      NA       NA
    ## announce         0.0769230769230483113         NA      NA       NA
    ## announced        0.2812500000000208722         NA      NA       NA
    ## announcement    -0.0000000000000092728         NA      NA       NA
    ## announces        0.3333333333332754167         NA      NA       NA
    ## annoying        -0.0000000000000340926         NA      NA       NA
    ## annual          -0.0000000000001636733         NA      NA       NA
    ## another                             NA         NA      NA       NA
    ## answer                              NA         NA      NA       NA
    ## answers          0.4545454545455049344         NA      NA       NA
    ## anti             0.0917431192660438377         NA      NA       NA
    ## antibiotics      0.1764705882352745892         NA      NA       NA
    ## anxiety                             NA         NA      NA       NA
    ## anybody          0.0370370370370144489         NA      NA       NA
    ## anymore                             NA         NA      NA       NA
    ## anyone                              NA         NA      NA       NA
    ## anything         0.0789473684210486304         NA      NA       NA
    ## anyway           0.3529411764705448484         NA      NA       NA
    ## anywhere        -0.0000000000000553261         NA      NA       NA
    ## aoc             -0.0000000000000309350         NA      NA       NA
    ## apart                               NA         NA      NA       NA
    ## apartment       -0.0000000000000335516         NA      NA       NA
    ## apes            -0.0000000000000329413         NA      NA       NA
    ## apocalypse       0.0714285714285395890         NA      NA       NA
    ## apocalyptic     -0.0000000000000348051         NA      NA       NA
    ## app                                 NA         NA      NA       NA
    ## apparently       0.0810810810810569937         NA      NA       NA
    ## appear                              NA         NA      NA       NA
    ## appears                             NA         NA      NA       NA
    ## apple                               NA         NA      NA       NA
    ## application                         NA         NA      NA       NA
    ## applications    -0.0000000000000303497         NA      NA       NA
    ## applied         -0.0000000000000436217         NA      NA       NA
    ## apply                               NA         NA      NA       NA
    ## appointed        0.0000000000000204731         NA      NA       NA
    ## appreciate                          NA         NA      NA       NA
    ## appreciated     -0.0000000000000117684         NA      NA       NA
    ## approach                            NA         NA      NA       NA
    ## approaching      0.1999999999999587941         NA      NA       NA
    ## apps                                NA         NA      NA       NA
    ## april            0.3157894736841728722         NA      NA       NA
    ## area                                NA         NA      NA       NA
    ## aren                                NA         NA      NA       NA
    ## argue            0.0555555555555323904         NA      NA       NA
    ## argues          -0.0000000000000631663         NA      NA       NA
    ## arguing          0.2307692307691980860         NA      NA       NA
    ## argument                            NA         NA      NA       NA
    ## arguments        0.1052631578946721241         NA      NA       NA
    ## arm             -0.0000000000000778991         NA      NA       NA
    ## arms                                NA         NA      NA       NA
    ## army                                NA         NA      NA       NA
    ## arnold           0.0833333333332924864         NA      NA       NA
    ## around           0.0645161290322577019         NA      NA       NA
    ## arrest           0.1249999999999909656         NA      NA       NA
    ## arrested                            NA         NA      NA       NA
    ## arrival          0.0833333333333289850         NA      NA       NA
    ## arrived          0.6923076923076152411         NA      NA       NA
    ## arsenal                             NA         NA      NA       NA
    ## art              0.0684931506849237870         NA      NA       NA
    ## article                             NA         NA      NA       NA
    ## articles        -0.0000000000000221411         NA      NA       NA
    ## artificial                          NA         NA      NA       NA
    ## artist           0.0188679245282965292         NA      NA       NA
    ## artists                             NA         NA      NA       NA
    ## asian            0.1562499999999922007         NA      NA       NA
    ## asians           0.3636363636363156848         NA      NA       NA
    ## aside                               NA         NA      NA       NA
    ## ask                                 NA         NA      NA       NA
    ## asked            0.1886792452830213518         NA      NA       NA
    ## asking           0.0540540540540263501         NA      NA       NA
    ## askmen           0.2999999999998183564         NA      NA       NA
    ## askreddit                           NA         NA      NA       NA
    ## asks             0.1874999999999486799         NA      NA       NA
    ## asleep                              NA         NA      NA       NA
    ## ass             -0.0000000000000325958         NA      NA       NA
    ## assault          0.0999999999999660744         NA      NA       NA
    ## asshole          0.0909090909090663063         NA      NA       NA
    ## assistant       -0.0000000000000408044         NA      NA       NA
    ## association     -0.0000000000000620369         NA      NA       NA
    ## assume          -0.0000000000000490593         NA      NA       NA
    ## assuming                            NA         NA      NA       NA
    ## attack                              NA         NA      NA       NA
    ## attacked                            NA         NA      NA       NA
    ## attacks                             NA         NA      NA       NA
    ## attempt                             NA         NA      NA       NA
    ## attempted                           NA         NA      NA       NA
    ## attempts                            NA         NA      NA       NA
    ## attention                           NA         NA      NA       NA
    ## attorney                            NA         NA      NA       NA
    ## audience                            NA         NA      NA       NA
    ## audio           -0.0000000000000731659         NA      NA       NA
    ## august                              NA         NA      NA       NA
    ## australia        0.0952380952380598444         NA      NA       NA
    ## australian       0.0285714285714186167         NA      NA       NA
    ## authorities     -0.0000000000001062607         NA      NA       NA
    ## auto             0.0666666666666345664         NA      NA       NA
    ## automate                            NA         NA      NA       NA
    ## automated        0.0714285714285525092         NA      NA       NA
    ## automatically                       NA         NA      NA       NA
    ## automation                          NA         NA      NA       NA
    ## autonomous       0.1333333333334033866         NA      NA       NA
    ## available        0.1951219512195075667         NA      NA       NA
    ## avatar           0.0769230769230642847         NA      NA       NA
    ## avengers         0.1071428571428600929         NA      NA       NA
    ## average                             NA         NA      NA       NA
    ## avian           -0.0000000000000623873         NA      NA       NA
    ## avoid            0.1923076923076908751         NA      NA       NA
    ## award            0.0714285714285312484         NA      NA       NA
    ## awards                              NA         NA      NA       NA
    ## aware                               NA         NA      NA       NA
    ## awareness                           NA         NA      NA       NA
    ## away                                NA         NA      NA       NA
    ## awesome          0.0799999999999625455         NA      NA       NA
    ## awful                               NA         NA      NA       NA
    ## b                                   NA         NA      NA       NA
    ## babies          -0.0000000000000192469         NA      NA       NA
    ## baby                                NA         NA      NA       NA
    ## back             0.0438596491228051485         NA      NA       NA
    ## background      -0.0000000000000250484         NA      NA       NA
    ## bad                                 NA         NA      NA       NA
    ## badly                               NA         NA      NA       NA
    ## bag                                 NA         NA      NA       NA
    ## balance                             NA         NA      NA       NA
    ## ball            -0.0000000000000397612         NA      NA       NA
    ## ballot           0.3076923076923315792         NA      NA       NA
    ## ballots          0.4499999999999886868         NA      NA       NA
    ## ban              0.2083333333333539650         NA      NA       NA
    ## band             0.2777777777777775681         NA      NA       NA
    ## bank             0.2285714285714495198         NA      NA       NA
    ## banned                              NA         NA      NA       NA
    ## banning                             NA         NA      NA       NA
    ## bans                                NA         NA      NA       NA
    ## bar              0.1428571428571462354         NA      NA       NA
    ## barack          -0.0000000000000405209         NA      NA       NA
    ## barcelona        0.2142857142856745001         NA      NA       NA
    ## base             0.1999999999999492739         NA      NA       NA
    ## based                               NA         NA      NA       NA
    ## basic            0.0847457627118527873         NA      NA       NA
    ## basically        0.1666666666666413998         NA      NA       NA
    ## basis                               NA         NA      NA       NA
    ## bat              0.1764705882352396449         NA      NA       NA
    ## bathroom         0.2857142857142708769         NA      NA       NA
    ## batman                              NA         NA      NA       NA
    ## battle                              NA         NA      NA       NA
    ## bay              0.3333333333332602066         NA      NA       NA
    ## bayern           0.0999999999998916062         NA      NA       NA
    ## bbc              0.0000000000000840323         NA      NA       NA
    ## beach                               NA         NA      NA       NA
    ## bear            -0.0000000000000919998         NA      NA       NA
    ## beast            0.1666666666666082319         NA      NA       NA
    ## beat                                NA         NA      NA       NA
    ## beating          0.1999999999999733935         NA      NA       NA
    ## beats           -0.0000000000000183320         NA      NA       NA
    ## beautiful                           NA         NA      NA       NA
    ## became           0.1111111111110827943         NA      NA       NA
    ## become           0.0952380952380968288         NA      NA       NA
    ## becomes                             NA         NA      NA       NA
    ## becoming                            NA         NA      NA       NA
    ## bed              0.2222222222222352550         NA      NA       NA
    ## bee              0.0243902439023973155         NA      NA       NA
    ## beer             0.2999999999999471978         NA      NA       NA
    ## begin                               NA         NA      NA       NA
    ## beginning                           NA         NA      NA       NA
    ## behavior         0.0434782608695672979         NA      NA       NA
    ## behind           0.0897435897435963637         NA      NA       NA
    ## belgium          0.0999999999999713757         NA      NA       NA
    ## believe          0.0819672131147527322         NA      NA       NA
    ## believes                            NA         NA      NA       NA
    ## belongs         -0.0000000000000024116         NA      NA       NA
    ## ben                                 NA         NA      NA       NA
    ## benefit          0.0454545454545200109         NA      NA       NA
    ## benefits                            NA         NA      NA       NA
    ## berlin                              NA         NA      NA       NA
    ## bernie                              NA         NA      NA       NA
    ## besides         -0.0000000000000379162         NA      NA       NA
    ## best             0.0269541778975729736         NA      NA       NA
    ## beta             0.0000000000000201448         NA      NA       NA
    ## better                              NA         NA      NA       NA
    ## beyond                              NA         NA      NA       NA
    ## bias            -0.0000000000000659297         NA      NA       NA
    ## bid                                 NA         NA      NA       NA
    ## biden            0.0465116279069422642         NA      NA       NA
    ## big                                 NA         NA      NA       NA
    ## bigger          -0.0000000000000473786         NA      NA       NA
    ## biggest                             NA         NA      NA       NA
    ## bill             0.1010101010100937224         NA      NA       NA
    ## billion          0.2439024390244369511         NA      NA       NA
    ## billionaire                         NA         NA      NA       NA
    ## billionaires    -0.0000000000000364594         NA      NA       NA
    ## bills            0.1111111111110822253         NA      NA       NA
    ## bio              0.0769230769230611622         NA      NA       NA
    ## biological                          NA         NA      NA       NA
    ## biopic           0.0000000000000084295         NA      NA       NA
    ## bird                                NA         NA      NA       NA
    ## birds            0.7272727272727681491         NA      NA       NA
    ## birth                               NA         NA      NA       NA
    ## birthday         0.0384615384615310460         NA      NA       NA
    ## bit              0.0303030303030293047         NA      NA       NA
    ## bitcoin          0.3333333333333305393         NA      NA       NA
    ## black                               NA         NA      NA       NA
    ## blade           -0.0000000000000294390         NA      NA       NA
    ## blame            0.1428571428571126511         NA      NA       NA
    ## blind           -0.0000000000000315492         NA      NA       NA
    ## block            0.0999999999999665878         NA      NA       NA
    ## blockbuster      0.1333333333333285853         NA      NA       NA
    ## blockchain                          NA         NA      NA       NA
    ## blood                               NA         NA      NA       NA
    ## bloomberg        0.0545454545454416712         NA      NA       NA
    ## blue                                NA         NA      NA       NA
    ## blues                               NA         NA      NA       NA
    ## board            0.1186440677966102558         NA      NA       NA
    ## bob              0.2499999999999823475         NA      NA       NA
    ## bodies           0.4999999999999714118         NA      NA       NA
    ## body             0.2439024390243555718         NA      NA       NA
    ## bolivia          0.0526315789473416409         NA      NA       NA
    ## bolsonaro        0.9999999999998516742         NA      NA       NA
    ## bomb             0.1428571428571191460         NA      NA       NA
    ## bond             0.1111111111110882482         NA      NA       NA
    ## bonus            0.0499999999999805322         NA      NA       NA
    ## book                                NA         NA      NA       NA
    ## books            0.1206896551724004341         NA      NA       NA
    ## boost                               NA         NA      NA       NA
    ## border           0.2105263157894675385         NA      NA       NA
    ## bored            0.0434782608695395076         NA      NA       NA
    ## boris           -0.0000000000000309659         NA      NA       NA
    ## born             0.0833333333332876292         NA      NA       NA
    ## boss                                NA         NA      NA       NA
    ## boston                              NA         NA      NA       NA
    ## bot                                 NA         NA      NA       NA
    ## bots                                NA         NA      NA       NA
    ## bottle           0.1176470588234863890         NA      NA       NA
    ## bottom           0.2499999999999662215         NA      NA       NA
    ## bought           0.1707317073170714905         NA      NA       NA
    ## bout                                NA         NA      NA       NA
    ## bowl                                NA         NA      NA       NA
    ## box              0.1639344262295079069         NA      NA       NA
    ## boy              0.0454545454545505906         NA      NA       NA
    ## boyfriend        0.3999999999999720446         NA      NA       NA
    ## boys                                NA         NA      NA       NA
    ## brain            0.1162790697674345175         NA      NA       NA
    ## brains                              NA         NA      NA       NA
    ## brasileiro      -0.0000000000000822090         NA      NA       NA
    ## brave                               NA         NA      NA       NA
    ## brazil           0.2916666666666529184         NA      NA       NA
    ## brazilian                           NA         NA      NA       NA
    ## `break`                             NA         NA      NA       NA
    ## breaking                            NA         NA      NA       NA
    ## breaks           0.0499999999999877209         NA      NA       NA
    ## breakthrough    -0.0000000000000495154         NA      NA       NA
    ## breast          -0.0000000000000480559         NA      NA       NA
    ## brexit                              NA         NA      NA       NA
    ## bridge                              NA         NA      NA       NA
    ## bring            0.1587301587301584715         NA      NA       NA
    ## bringing                            NA         NA      NA       NA
    ## brings           0.1499999999999810651         NA      NA       NA
    ## britain          0.0000000000000344207         NA      NA       NA
    ## british          0.2058823529411610287         NA      NA       NA
    ## broke                               NA         NA      NA       NA
    ## broken           0.1212121212121340108         NA      NA       NA
    ## bros                                NA         NA      NA       NA
    ## brother                             NA         NA      NA       NA
    ## brothers         0.0454545454545515135         NA      NA       NA
    ## brought          0.1818181818181517917         NA      NA       NA
    ## brown            0.0769230769230475758         NA      NA       NA
    ## bruce            0.1666666666666507812         NA      NA       NA
    ## buddy           -0.0000000000000269323         NA      NA       NA
    ## budget                              NA         NA      NA       NA
    ## budgeting       -0.0000000000000301889         NA      NA       NA
    ## buff                                NA         NA      NA       NA
    ## bug             -0.0000000000000082622         NA      NA       NA
    ## bugs                                NA         NA      NA       NA
    ## build                               NA         NA      NA       NA
    ## building         0.0491803278688576206         NA      NA       NA
    ## buildings                           NA         NA      NA       NA
    ## builds          -0.0000000000000281076         NA      NA       NA
    ## built                               NA         NA      NA       NA
    ## bullet           0.2307692307692117972         NA      NA       NA
    ## bullshit         0.1481481481481158324         NA      NA       NA
    ## bunch                               NA         NA      NA       NA
    ## burn             0.2142857142856753883         NA      NA       NA
    ## burning          0.2142857142857371999         NA      NA       NA
    ## bus                                 NA         NA      NA       NA
    ## bush             0.4999999999999717448         NA      NA       NA
    ## business                            NA         NA      NA       NA
    ## businesses                          NA         NA      NA       NA
    ## buttigieg                           NA         NA      NA       NA
    ## buy              0.1492537313432838186         NA      NA       NA
    ## buying           0.0344827586206723005         NA      NA       NA
    ## buys            -0.0000000000000044802         NA      NA       NA
    ## c                0.0689655172413643353         NA      NA       NA
    ## ca               0.0425531914893468449         NA      NA       NA
    ## cage             0.1739130434782480417         NA      NA       NA
    ## california                          NA         NA      NA       NA
    ## call                                NA         NA      NA       NA
    ## called           0.0793650793650824138         NA      NA       NA
    ## calling          0.2325581395348869096         NA      NA       NA
    ## calls                               NA         NA      NA       NA
    ## cambridge                           NA         NA      NA       NA
    ## came                                NA         NA      NA       NA
    ## camera                              NA         NA      NA       NA
    ## cameras          0.0555555555555308361         NA      NA       NA
    ## camp            -0.0000000000000322085         NA      NA       NA
    ## campaign         0.0518134715025909479         NA      NA       NA
    ## campaigns                           NA         NA      NA       NA
    ## can              0.0087719298245613475         NA      NA       NA
    ## canada           0.0933333333333391107         NA      NA       NA
    ## canadian         0.0731707317073112401         NA      NA       NA
    ## cancel                              NA         NA      NA       NA
    ## canceled                            NA         NA      NA       NA
    ## cancelled        0.4285714285714176675         NA      NA       NA
    ## cancer           0.1190476190475894536         NA      NA       NA
    ## candidate        0.0571428571428549761         NA      NA       NA
    ## candidates                          NA         NA      NA       NA
    ## cant                                NA         NA      NA       NA
    ## capable                             NA         NA      NA       NA
    ## capital                             NA         NA      NA       NA
    ## capitalism                          NA         NA      NA       NA
    ## captain          0.1818181818181092424         NA      NA       NA
    ## car              0.0467289719626163141         NA      NA       NA
    ## carbon                              NA         NA      NA       NA
    ## card                                NA         NA      NA       NA
    ## cards                               NA         NA      NA       NA
    ## care             0.0699999999999915690         NA      NA       NA
    ## career           0.0882352941176235972         NA      NA       NA
    ## carolina                            NA         NA      NA       NA
    ## cars             0.0769230769230719036         NA      NA       NA
    ## cartoon         -0.0000000000000688670         NA      NA       NA
    ## case                                NA         NA      NA       NA
    ## cases            0.0990099009900947680         NA      NA       NA
    ## cash            -0.0000000000000045947         NA      NA       NA
    ## cast                                NA         NA      NA       NA
    ## casting          0.1904761904761516911         NA      NA       NA
    ## castle                              NA         NA      NA       NA
    ## cat              0.0550458715596311987         NA      NA       NA
    ## catch                               NA         NA      NA       NA
    ## catching                            NA         NA      NA       NA
    ## cats                                NA         NA      NA       NA
    ## caucus           0.2173913043478026830         NA      NA       NA
    ## caught                              NA         NA      NA       NA
    ## cause            0.0909090909090821270         NA      NA       NA
    ## caused           0.4000000000000061839         NA      NA       NA
    ## causes           0.1111111111110945904         NA      NA       NA
    ## causing         -0.0000000000000842452         NA      NA       NA
    ## cavalinhos                          NA         NA      NA       NA
    ## cdc                                 NA         NA      NA       NA
    ## celebrate       -0.0000000000000498616         NA      NA       NA
    ## celebrating                         NA         NA      NA       NA
    ## celebrity        0.3636363636363504348         NA      NA       NA
    ## cell             0.1538461538461367573         NA      NA       NA
    ## cells            0.0000000000000555618         NA      NA       NA
    ## censorship                          NA         NA      NA       NA
    ## center           0.1081081081081020079         NA      NA       NA
    ## central          0.0499999999999916067         NA      NA       NA
    ## century          0.3461538461538374856         NA      NA       NA
    ## ceo                                 NA         NA      NA       NA
    ## certain                             NA         NA      NA       NA
    ## cgi              0.0666666666665924057         NA      NA       NA
    ## chain                               NA         NA      NA       NA
    ## chair            0.1538461538461309563         NA      NA       NA
    ## chairman                            NA         NA      NA       NA
    ## challenge                           NA         NA      NA       NA
    ## champion        -0.0000000000000391443         NA      NA       NA
    ## champions        0.1249999999999700517         NA      NA       NA
    ## chance                              NA         NA      NA       NA
    ## chances                             NA         NA      NA       NA
    ## change           0.0431034482758636261         NA      NA       NA
    ## changed          0.0909090909090902455         NA      NA       NA
    ## changes                             NA         NA      NA       NA
    ## changing         0.0909090909090771448         NA      NA       NA
    ## channel                             NA         NA      NA       NA
    ## chaos                               NA         NA      NA       NA
    ## chapter         -0.0000000000000432571         NA      NA       NA
    ## character        0.0980392156862626429         NA      NA       NA
    ## characters                          NA         NA      NA       NA
    ## charge                              NA         NA      NA       NA
    ## charged                             NA         NA      NA       NA
    ## charges          0.3888888888888866746         NA      NA       NA
    ## charity          0.2999999999998989031         NA      NA       NA
    ## charlie          0.1818181818181386356         NA      NA       NA
    ## chatbot         -0.0000000000000345914         NA      NA       NA
    ## cheap           -0.0000000000000631604         NA      NA       NA
    ## check                               NA         NA      NA       NA
    ## checking        -0.0000000000000480566         NA      NA       NA
    ## checks          -0.0000000000000492466         NA      NA       NA
    ## chelsea          0.1818181818181351106         NA      NA       NA
    ## chicago          0.3999999999999463984         NA      NA       NA
    ## chicken                             NA         NA      NA       NA
    ## chief            0.0689655172413610879         NA      NA       NA
    ## child            0.1204819277108416797         NA      NA       NA
    ## childhood        0.0666666666666346913         NA      NA       NA
    ## children                            NA         NA      NA       NA
    ## chill                               NA         NA      NA       NA
    ## china                               NA         NA      NA       NA
    ## `china's`                           NA         NA      NA       NA
    ## chinese          0.0549450549450562764         NA      NA       NA
    ## chip                                NA         NA      NA       NA
    ## choice                              NA         NA      NA       NA
    ## choices                             NA         NA      NA       NA
    ## choose           0.1136363636363558055         NA      NA       NA
    ## choosing         0.1111111111110741761         NA      NA       NA
    ## chris            0.1249999999999780870         NA      NA       NA
    ## christian        0.2307692307691839584         NA      NA       NA
    ## christmas        0.1046511627906948350         NA      NA       NA
    ## church          -0.0000000000000238480         NA      NA       NA
    ## cia                                 NA         NA      NA       NA
    ## cinema           0.1228070175438460698         NA      NA       NA
    ## cinemas          0.3571428571428153509         NA      NA       NA
    ## cinematic        0.1249999999999608508         NA      NA       NA
    ## circle                              NA         NA      NA       NA
    ## cities                              NA         NA      NA       NA
    ## citizen          0.1379310344827534285         NA      NA       NA
    ## citizens                            NA         NA      NA       NA
    ## city             0.0757575757575686404         NA      NA       NA
    ## civil            0.2380952380952384428         NA      NA       NA
    ## civilization     0.0555555555555332439         NA      NA       NA
    ## claim            0.2777777777777515889         NA      NA       NA
    ## claiming                            NA         NA      NA       NA
    ## claims                              NA         NA      NA       NA
    ## clash            0.1176470588234982961         NA      NA       NA
    ## class            0.0784313725490135699         NA      NA       NA
    ## classes                             NA         NA      NA       NA
    ## classic          0.1555555555555741265         NA      NA       NA
    ## clause                              NA         NA      NA       NA
    ## clean                               NA         NA      NA       NA
    ## cleaning         0.0833333333332910153         NA      NA       NA
    ## clear           -0.0000000000000196026         NA      NA       NA
    ## clearly          0.0000000000000432460         NA      NA       NA
    ## clearview       -0.0000000000000850313         NA      NA       NA
    ## climate          0.0529100529100518524         NA      NA       NA
    ## clinton          0.2222222222222010601         NA      NA       NA
    ## clip             0.0555555555555271724         NA      NA       NA
    ## clips            0.0499999999999896916         NA      NA       NA
    ## close            0.0694444444444324849         NA      NA       NA
    ## closed                              NA         NA      NA       NA
    ## closer           0.0769230769230664774         NA      NA       NA
    ## closing         -0.0000000000000374859         NA      NA       NA
    ## cloud            0.0357142857142401515         NA      NA       NA
    ## club                                NA         NA      NA       NA
    ## clubs                               NA         NA      NA       NA
    ## clue             0.0833333333332790388         NA      NA       NA
    ## cmv              0.0109649122807013010         NA      NA       NA
    ## cnn              0.1428571428571085988         NA      NA       NA
    ## co               0.1860465116278981201         NA      NA       NA
    ## coach                               NA         NA      NA       NA
    ## coal             0.1666666666666407892         NA      NA       NA
    ## cod                                 NA         NA      NA       NA
    ## code                                NA         NA      NA       NA
    ## coffee                              NA         NA      NA       NA
    ## cold                                NA         NA      NA       NA
    ## collection       0.0000000000000171296         NA      NA       NA
    ## college          0.0588235294117589527         NA      NA       NA
    ## color                               NA         NA      NA       NA
    ## colorado         0.5882352941176035577         NA      NA       NA
    ## combat           0.0399999999999834030         NA      NA       NA
    ## come                                NA         NA      NA       NA
    ## comedy           0.1999999999999919620         NA      NA       NA
    ## comes                               NA         NA      NA       NA
    ## comfort                             NA         NA      NA       NA
    ## comic            0.0434782608695436709         NA      NA       NA
    ## comics           0.1000000000000248607         NA      NA       NA
    ## coming           0.0724637681159354119         NA      NA       NA
    ## comment          0.2051282051281977381         NA      NA       NA
    ## comments                            NA         NA      NA       NA
    ## commercial       0.4347826086956207425         NA      NA       NA
    ## commission       0.1363636363636428761         NA      NA       NA
    ## committee        0.0833333333332860887         NA      NA       NA
    ## common           0.0461538461538442135         NA      NA       NA
    ## community        0.1914893617021710959         NA      NA       NA
    ## companies                           NA         NA      NA       NA
    ## company          0.0421686746987931585         NA      NA       NA
    ## comparable      -0.0000000000000512687         NA      NA       NA
    ## compared         0.1739130434782321100         NA      NA       NA
    ## comparison      -0.0000000000000476847         NA      NA       NA
    ## compete         -0.0000000000000243109         NA      NA       NA
    ## competition     -0.0000000000000171684         NA      NA       NA
    ## complain        -0.0000000000000631040         NA      NA       NA
    ## complaining      0.1666666666666206664         NA      NA       NA
    ## complaint                           NA         NA      NA       NA
    ## complaints       0.0999999999999551664         NA      NA       NA
    ## complete         0.3448275862068737596         NA      NA       NA
    ## completed                           NA         NA      NA       NA
    ## completely                          NA         NA      NA       NA
    ## complex         -0.0000000000000190418         NA      NA       NA
    ## computer                            NA         NA      NA       NA
    ## computers        0.0833333333333154958         NA      NA       NA
    ## computing                           NA         NA      NA       NA
    ## con             -0.0000000000000365418         NA      NA       NA
    ## concept          0.0499999999999747660         NA      NA       NA
    ## concern         -0.0000000000000382495         NA      NA       NA
    ## concerned                           NA         NA      NA       NA
    ## concerns         0.0312499999999870659         NA      NA       NA
    ## condition                           NA         NA      NA       NA
    ## conditions                          NA         NA      NA       NA
    ## confidence      -0.0000000000000521458         NA      NA       NA
    ## confident       -0.0000000000000464226         NA      NA       NA
    ## confirmed                           NA         NA      NA       NA
    ## confirms         0.1025641025640900567         NA      NA       NA
    ## confused        -0.0000000000000410999         NA      NA       NA
    ## congress         0.2272727272727267933         NA      NA       NA
    ## congressional    0.1999999999999956812         NA      NA       NA
    ## congressman                         NA         NA      NA       NA
    ## connected        0.1666666666667045715         NA      NA       NA
    ## conscious                           NA         NA      NA       NA
    ## consent         -0.0000000000000264599         NA      NA       NA
    ## consequences    -0.0000000000000088558         NA      NA       NA
    ## conservative     0.1025641025640913195         NA      NA       NA
    ## conservatives    0.1034482758620478976         NA      NA       NA
    ## consider         0.4545454545454318818         NA      NA       NA
    ## considered                          NA         NA      NA       NA
    ## considering                         NA         NA      NA       NA
    ## conspiracy       0.1290322580645436312         NA      NA       NA
    ## constantly       0.0909090909090129046         NA      NA       NA
    ## construction                        NA         NA      NA       NA
    ## contact          0.0416666666666495530         NA      NA       NA
    ## content                             NA         NA      NA       NA
    ## contest         -0.0000000000000268436         NA      NA       NA
    ## context          0.1818181818181809906         NA      NA       NA
    ## continue                            NA         NA      NA       NA
    ## continues                           NA         NA      NA       NA
    ## contract         0.0303030303030048694         NA      NA       NA
    ## contracts                           NA         NA      NA       NA
    ## contribute      -0.0000000000000460928         NA      NA       NA
    ## contribution     0.1428571428571083490         NA      NA       NA
    ## contributions   -0.0000000000000202985         NA      NA       NA
    ## control                             NA         NA      NA       NA
    ## controlled      -0.0000000000000349814         NA      NA       NA
    ## controversial    0.1666666666666434260         NA      NA       NA
    ## controversy                         NA         NA      NA       NA
    ## convention      -0.0000000000001070748         NA      NA       NA
    ## conversation                        NA         NA      NA       NA
    ## convince                            NA         NA      NA       NA
    ## convinced       -0.0000000000000887894         NA      NA       NA
    ## cool                                NA         NA      NA       NA
    ## cooper                              NA         NA      NA       NA
    ## cop              0.0588235294118016477         NA      NA       NA
    ## cops             0.0624999999999585887         NA      NA       NA
    ## copy             0.1249999999999829720         NA      NA       NA
    ## copyright        0.1388888888888865358         NA      NA       NA
    ## corner                              NA         NA      NA       NA
    ## corona           0.0845070422535121185         NA      NA       NA
    ## coronavirus                         NA         NA      NA       NA
    ## corporate                           NA         NA      NA       NA
    ## corporations                        NA         NA      NA       NA
    ## correct          0.0833333333332955950         NA      NA       NA
    ## correctly                           NA         NA      NA       NA
    ## corrupt          0.0666666666666436841         NA      NA       NA
    ## corruption       0.2222222222221830468         NA      NA       NA
    ## cosplay          0.1538461538461040334         NA      NA       NA
    ## cost                                NA         NA      NA       NA
    ## costs                               NA         NA      NA       NA
    ## costume                             NA         NA      NA       NA
    ## couch                               NA         NA      NA       NA
    ## cough            0.0909090909090232435         NA      NA       NA
    ## couldn          -0.0000000000000542146         NA      NA       NA
    ## council          0.1379310344827203716         NA      NA       NA
    ## count                               NA         NA      NA       NA
    ## counted          0.2999999999999640177         NA      NA       NA
    ## counter                             NA         NA      NA       NA
    ## counties        -0.0000000000000392041         NA      NA       NA
    ## countries        0.1515151515151479389         NA      NA       NA
    ## country          0.0729927007299163155         NA      NA       NA
    ## `country's`     -0.0000000000000291331         NA      NA       NA
    ## county           0.1190476190476122409         NA      NA       NA
    ## couple           0.0294117647058682249         NA      NA       NA
    ## course           0.0526315789473790069         NA      NA       NA
    ## court            0.1492537313432709123         NA      NA       NA
    ## cover            0.0535714285714266325         NA      NA       NA
    ## coverage         0.1562499999999800715         NA      NA       NA
    ## covered          0.1739130434782436563         NA      NA       NA
    ## covid                               NA         NA      NA       NA
    ## crack            0.2727272727272471720         NA      NA       NA
    ## crash           -0.0000000000000374146         NA      NA       NA
    ## crazy            0.0769230769230679345         NA      NA       NA
    ## create                              NA         NA      NA       NA
    ## created          0.0845070422535221244         NA      NA       NA
    ## creates          0.0000000000000083676         NA      NA       NA
    ## creating         0.0312499999999833675         NA      NA       NA
    ## creative                            NA         NA      NA       NA
    ## creator                             NA         NA      NA       NA
    ## creators         0.0769230769230413447         NA      NA       NA
    ## credit           0.0285714285714225996         NA      NA       NA
    ## credits          0.2999999999999490297         NA      NA       NA
    ## creepy           0.0454545454545307731         NA      NA       NA
    ## crew             0.0833333333332739179         NA      NA       NA
    ## crime            0.0357142857142667969         NA      NA       NA
    ## criminal         0.1764705882353421462         NA      NA       NA
    ## criminals       -0.0000000000000552305         NA      NA       NA
    ## crisis                              NA         NA      NA       NA
    ## critic           0.0909090909090569665         NA      NA       NA
    ## critical                            NA         NA      NA       NA
    ## criticism        0.0833333333332912929         NA      NA       NA
    ## criticizing                         NA         NA      NA       NA
    ## critics                             NA         NA      NA       NA
    ## croatia          0.5882352941177125816         NA      NA       NA
    ## cross            0.0909090909090983917         NA      NA       NA
    ## crowd            0.5263157894736518738         NA      NA       NA
    ## cruise           0.0588235294117322866         NA      NA       NA
    ## cry                                 NA         NA      NA       NA
    ## crying                              NA         NA      NA       NA
    ## cryptocurrency  -0.0000000000000426511         NA      NA       NA
    ## cube            -0.0000000000000277856         NA      NA       NA
    ## cult                                NA         NA      NA       NA
    ## cultural                            NA         NA      NA       NA
    ## culture          0.0740740740740942899         NA      NA       NA
    ## cup              0.2564102564102461734         NA      NA       NA
    ## cure             0.0312499999999860945         NA      NA       NA
    ## cured           -0.0000000000000168263         NA      NA       NA
    ## current          0.0624999999999917288         NA      NA       NA
    ## currently                           NA         NA      NA       NA
    ## custom                              NA         NA      NA       NA
    ## customer        -0.0000000000000034246         NA      NA       NA
    ## customers        0.0909090909090633364         NA      NA       NA
    ## cut              0.2187499999999944211         NA      NA       NA
    ## cute            -0.0000000000000108660         NA      NA       NA
    ## cutest          -0.0000000000001633460         NA      NA       NA
    ## cuts                                NA         NA      NA       NA
    ## cutting          0.1428571428571170365         NA      NA       NA
    ## cycle                               NA         NA      NA       NA
    ## d                0.0987654320987688961         NA      NA       NA
    ## dad                                 NA         NA      NA       NA
    ## daily            0.1599999999999756339         NA      NA       NA
    ## damage           0.0909090909090732452         NA      NA       NA
    ## damaged                             NA         NA      NA       NA
    ## damn             0.0666666666666494018         NA      NA       NA
    ## dance            0.0322580645161145221         NA      NA       NA
    ## danger          -0.0000000000000320285         NA      NA       NA
    ## dangerous        0.0599999999999972500         NA      NA       NA
    ## dangers          0.0909090909090535526         NA      NA       NA
    ## daniel           0.1999999999999206857         NA      NA       NA
    ## danny            0.1818181818181946463         NA      NA       NA
    ## dare            -0.0000000000000348146         NA      NA       NA
    ## dark             0.0483870967741898972         NA      NA       NA
    ## data             0.0400000000000000633         NA      NA       NA
    ## database                            NA         NA      NA       NA
    ## date             0.1249999999999946293         NA      NA       NA
    ## dates                               NA         NA      NA       NA
    ## dating           0.0666666666666329844         NA      NA       NA
    ## daughter         0.1714285714285511686         NA      NA       NA
    ## dave             0.0833333333332589715         NA      NA       NA
    ## david            0.0666666666666625440         NA      NA       NA
    ## dawn                                NA         NA      NA       NA
    ## day                                 NA         NA      NA       NA
    ## days             0.0578034682080850182         NA      NA       NA
    ## dc               0.1379310344827437418         NA      NA       NA
    ## de               0.0925925925925881882         NA      NA       NA
    ## dead                                NA         NA      NA       NA
    ## deadline         0.2499999999999565348         NA      NA       NA
    ## deadly           0.1153846153846036504         NA      NA       NA
    ## deadpool                            NA         NA      NA       NA
    ## deal                                NA         NA      NA       NA
    ## dealing          0.1249999999999692329         NA      NA       NA
    ## dear                                NA         NA      NA       NA
    ## death                               NA         NA      NA       NA
    ## deaths                              NA         NA      NA       NA
    ## debate           0.1515151515151479944         NA      NA       NA
    ## debates                             NA         NA      NA       NA
    ## debit           -0.0000000000000705503         NA      NA       NA
    ## debt             0.0869565217391275735         NA      NA       NA
    ## decade                              NA         NA      NA       NA
    ## decades                             NA         NA      NA       NA
    ## december                            NA         NA      NA       NA
    ## decent                              NA         NA      NA       NA
    ## decide           0.0606060606060392290         NA      NA       NA
    ## decided          0.0930232558139470894         NA      NA       NA
    ## decides          0.1874999999999498457         NA      NA       NA
    ## decision         0.2051282051282199426         NA      NA       NA
    ## decisions                           NA         NA      NA       NA
    ## declare                             NA         NA      NA       NA
    ## declares                            NA         NA      NA       NA
    ## deep             0.0263157894736760559         NA      NA       NA
    ## deepfake         0.0769230769230371952         NA      NA       NA
    ## deepfakes        0.2592592592592151135         NA      NA       NA
    ## deepmind                            NA         NA      NA       NA
    ## defeat           0.4285714285713797533         NA      NA       NA
    ## defeated                            NA         NA      NA       NA
    ## defend           0.0769230769230410533         NA      NA       NA
    ## defending                           NA         NA      NA       NA
    ## defense                             NA         NA      NA       NA
    ## definitely       0.0454545454545435268         NA      NA       NA
    ## degree          -0.0000000000000363950         NA      NA       NA
    ## delete                              NA         NA      NA       NA
    ## deleted                             NA         NA      NA       NA
    ## dem             -0.0000000000000489978         NA      NA       NA
    ## demand                              NA         NA      NA       NA
    ## demands         -0.0000000000000283717         NA      NA       NA
    ## democracy                           NA         NA      NA       NA
    ## democrat                            NA         NA      NA       NA
    ## democratic                          NA         NA      NA       NA
    ## democrats        0.1587301587301567229         NA      NA       NA
    ## denied                              NA         NA      NA       NA
    ## department       0.5625000000000679456         NA      NA       NA
    ## deposit                             NA         NA      NA       NA
    ## depression      -0.0000000000000462341         NA      NA       NA
    ## describe                            NA         NA      NA       NA
    ## desert           0.0666666666666121954         NA      NA       NA
    ## deserve          0.0588235294117417581         NA      NA       NA
    ## deserves        -0.0000000000000605793         NA      NA       NA
    ## design           0.0749999999999934885         NA      NA       NA
    ## designed         0.0285714285714170103         NA      NA       NA
    ## despite          0.2195121951219413392         NA      NA       NA
    ## destroy          0.4166666666667239727         NA      NA       NA
    ## destroying                          NA         NA      NA       NA
    ## detail           0.1818181818181368314         NA      NA       NA
    ## details          0.0909090909090713023         NA      NA       NA
    ## detect           0.0645161290322257691         NA      NA       NA
    ## detected        -0.0000000000000482267         NA      NA       NA
    ## detector        -0.0000000000000406437         NA      NA       NA
    ## determine                           NA         NA      NA       NA
    ## develop                             NA         NA      NA       NA
    ## developed        0.0270270270270190627         NA      NA       NA
    ## developers                          NA         NA      NA       NA
    ## developing       0.0454545454545263738         NA      NA       NA
    ## development      0.0624999999999880096         NA      NA       NA
    ## develops                            NA         NA      NA       NA
    ## device          -0.0000000000000624132         NA      NA       NA
    ## devices                             NA         NA      NA       NA
    ## devs                                NA         NA      NA       NA
    ## diagnosed        0.1818181818181572040         NA      NA       NA
    ## diagnosis                           NA         NA      NA       NA
    ## dick                                NA         NA      NA       NA
    ## didn                                NA         NA      NA       NA
    ## die                                 NA         NA      NA       NA
    ## died                                NA         NA      NA       NA
    ## dies             0.1199999999999804695         NA      NA       NA
    ## difference       0.0238095238095123729         NA      NA       NA
    ## different        0.1123595505618043139         NA      NA       NA
    ## difficult        0.5882352941176369754         NA      NA       NA
    ## difficulty       0.0454545454545241187         NA      NA       NA
    ## digital          0.0588235294117592303         NA      NA       NA
    ## dinner           0.1333333333333033832         NA      NA       NA
    ## direct                              NA         NA      NA       NA
    ## directed        -0.0000000000000213504         NA      NA       NA
    ## director         0.1363636363636348547         NA      NA       NA
    ## directors        0.4999999999999752420         NA      NA       NA
    ## dirty                               NA         NA      NA       NA
    ## disability                          NA         NA      NA       NA
    ## disabled        -0.0000000000000083312         NA      NA       NA
    ## disappointed     0.0666666666665836488         NA      NA       NA
    ## disaster         0.0714285714285824297         NA      NA       NA
    ## discover                            NA         NA      NA       NA
    ## discovered                          NA         NA      NA       NA
    ## discovery       -0.0000000000000532005         NA      NA       NA
    ## discrimination   0.0909090909090496807         NA      NA       NA
    ## discuss          0.4347826086956338432         NA      NA       NA
    ## discusses                           NA         NA      NA       NA
    ## discussing       0.1176470588235041803         NA      NA       NA
    ## discussion       0.1333333333333405202         NA      NA       NA
    ## disease          0.1276595744680823208         NA      NA       NA
    ## diseases         0.0000000000000206060         NA      NA       NA
    ## dislike         -0.0000000000000405329         NA      NA       NA
    ## disney           0.0862068965517185787         NA      NA       NA
    ## distribution     0.0000000000000203316         NA      NA       NA
    ## district                            NA         NA      NA       NA
    ## diversity                           NA         NA      NA       NA
    ## divided                             NA         NA      NA       NA
    ## division                            NA         NA      NA       NA
    ## dna             -0.0000000000000137990         NA      NA       NA
    ## dnc                                 NA         NA      NA       NA
    ## doctor           0.1785714285714244398         NA      NA       NA
    ## doctors          0.0714285714285606693         NA      NA       NA
    ## documentary      0.0416666666666602944         NA      NA       NA
    ## doesn            0.2380952380952472969         NA      NA       NA
    ## dog              0.0434782608695607128         NA      NA       NA
    ## dogs                                NA         NA      NA       NA
    ## dollar           0.0312499999999817230         NA      NA       NA
    ## dollars                             NA         NA      NA       NA
    ## domain                              NA         NA      NA       NA
    ## domestic         0.5000000000000234257         NA      NA       NA
    ## don                                 NA         NA      NA       NA
    ## donald                              NA         NA      NA       NA
    ## done                                NA         NA      NA       NA
    ## dont            -0.0000000000000204466         NA      NA       NA
    ## door             0.1071428571428342941         NA      NA       NA
    ## doors           -0.0000000000000725852         NA      NA       NA
    ## double           0.0909090909090815996         NA      NA       NA
    ## download                            NA         NA      NA       NA
    ## dr              -0.0000000000000283674         NA      NA       NA
    ## drama            0.0854700854700888579         NA      NA       NA
    ## draw                                NA         NA      NA       NA
    ## drawing         -0.0000000000000438263         NA      NA       NA
    ## dream            0.0624999999999976824         NA      NA       NA
    ## dreams                              NA         NA      NA       NA
    ## dress                               NA         NA      NA       NA
    ## dressed                             NA         NA      NA       NA
    ## drinking         0.0833333333332649806         NA      NA       NA
    ## drive            0.1025641025641110815         NA      NA       NA
    ## driven                              NA         NA      NA       NA
    ## driver           0.1052631578947239577         NA      NA       NA
    ## drivers                             NA         NA      NA       NA
    ## driving                             NA         NA      NA       NA
    ## drones                              NA         NA      NA       NA
    ## drop             0.0399999999999772760         NA      NA       NA
    ## dropped          0.0769230769230413725         NA      NA       NA
    ## dropping                            NA         NA      NA       NA
    ## drops            0.0909090909090750909         NA      NA       NA
    ## drowning                            NA         NA      NA       NA
    ## drug             0.0285714285714142972         NA      NA       NA
    ## drugs                               NA         NA      NA       NA
    ## drunk                               NA         NA      NA       NA
    ## du              -0.0000000000000391943         NA      NA       NA
    ## dude             0.1034482758620337145         NA      NA       NA
    ## due              0.0714285714285728679         NA      NA       NA
    ## dumb             0.1153846153846102285         NA      NA       NA
    ## dustin          -0.0000000000000680337         NA      NA       NA
    ## dutch                               NA         NA      NA       NA
    ## duty             0.0000000000000114583         NA      NA       NA
    ## dying                               NA         NA      NA       NA
    ## dystopian                           NA         NA      NA       NA
    ## dz               0.0588235294116981264         NA      NA       NA
    ## e                0.2499999999999613365         NA      NA       NA
    ## e.g                                 NA         NA      NA       NA
    ## earlier          0.0526315789473584955         NA      NA       NA
    ## early            0.0634920634920613791         NA      NA       NA
    ## earth                               NA         NA      NA       NA
    ## easier                              NA         NA      NA       NA
    ## easily          -0.0000000000000346652         NA      NA       NA
    ## east            -0.0000000000000213592         NA      NA       NA
    ## easter          -0.0000000000000396888         NA      NA       NA
    ## eastern                             NA         NA      NA       NA
    ## easy             0.0588235294117521249         NA      NA       NA
    ## eat             -0.0000000000000195512         NA      NA       NA
    ## eating                              NA         NA      NA       NA
    ## ebola           -0.0000000000000196863         NA      NA       NA
    ## economic                            NA         NA      NA       NA
    ## economy          0.2608695652173722013         NA      NA       NA
    ## edge            -0.0000000000000361019         NA      NA       NA
    ## edit            -0.0000000000000745691         NA      NA       NA
    ## editing         -0.0000000000000094758         NA      NA       NA
    ## edition          0.0588235294117430765         NA      NA       NA
    ## education        0.0344827586206886139         NA      NA       NA
    ## effect           0.1578947368420862418         NA      NA       NA
    ## effective        0.0714285714285931156         NA      NA       NA
    ## effects         -0.0000000000000188511         NA      NA       NA
    ## effort          -0.0000000000000276686         NA      NA       NA
    ## efforts         -0.0000000000000446114         NA      NA       NA
    ## egg                                 NA         NA      NA       NA
    ## either                              NA         NA      NA       NA
    ## el                                  NA         NA      NA       NA
    ## elect                               NA         NA      NA       NA
    ## electability    -0.0000000000000077756         NA      NA       NA
    ## electable                           NA         NA      NA       NA
    ## elected          0.0208768267223362024         NA      NA       NA
    ## electing         0.0967741935483836385         NA      NA       NA
    ## election         0.0051679586563302179         NA      NA       NA
    ## elections        0.0187617260787988800         NA      NA       NA
    ## elective        -0.0000000000000225733         NA      NA       NA
    ## electoral                           NA         NA      NA       NA
    ## electric                            NA         NA      NA       NA
    ## electronic                          NA         NA      NA       NA
    ## elects                              NA         NA      NA       NA
    ## eligible        -0.0000000000000357952         NA      NA       NA
    ## elite           -0.0000000000000515375         NA      NA       NA
    ## elizabeth        0.2499999999999574507         NA      NA       NA
    ## elon                                NA         NA      NA       NA
    ## `else`                              NA         NA      NA       NA
    ## email                               NA         NA      NA       NA
    ## emails          -0.0000000000000456283         NA      NA       NA
    ## embassy         -0.0000000000000199603         NA      NA       NA
    ## emergency        0.0465116279069690830         NA      NA       NA
    ## emoji           -0.0000000000000555869         NA      NA       NA
    ## emotional        0.0499999999999834951         NA      NA       NA
    ## emotions                            NA         NA      NA       NA
    ## employee         0.0799999999999952555         NA      NA       NA
    ## employees        0.1249999999999961281         NA      NA       NA
    ## employer         0.0135135135135012168         NA      NA       NA
    ## employment                          NA         NA      NA       NA
    ## empty                               NA         NA      NA       NA
    ## en              -0.0000000000000516040         NA      NA       NA
    ## end              0.0684931506849194432         NA      NA       NA
    ## ended            0.4545454545454416517         NA      NA       NA
    ## endgame                             NA         NA      NA       NA
    ## ending                              NA         NA      NA       NA
    ## endless         -0.0000000000000374219         NA      NA       NA
    ## endorsement      0.0000000000000230627         NA      NA       NA
    ## endorses        -0.0000000000000235005         NA      NA       NA
    ## ends             0.0434782608695449824         NA      NA       NA
    ## enemies          0.0624999999999644312         NA      NA       NA
    ## enemy            0.0487804878048625976         NA      NA       NA
    ## energy                              NA         NA      NA       NA
    ## enforcement      0.1999999999999879374         NA      NA       NA
    ## engine           0.0588235294117321825         NA      NA       NA
    ## engineer         0.3076923076922555289         NA      NA       NA
    ## engineering                         NA         NA      NA       NA
    ## engineers       -0.0000000000000393366         NA      NA       NA
    ## england                             NA         NA      NA       NA
    ## english          0.0196078431372547525         NA      NA       NA
    ## enjoy            0.1320754716981065446         NA      NA       NA
    ## enjoying                            NA         NA      NA       NA
    ## enough                              NA         NA      NA       NA
    ## ensues                              NA         NA      NA       NA
    ## enter           -0.0000000000000145081         NA      NA       NA
    ## entertainment                       NA         NA      NA       NA
    ## entire                              NA         NA      NA       NA
    ## entirely                            NA         NA      NA       NA
    ## entitled         0.0666666666665787500         NA      NA       NA
    ## entry           -0.0000000000000216503         NA      NA       NA
    ## enviro                              NA         NA      NA       NA
    ## enviroment                          NA         NA      NA       NA
    ## environment     -0.0000000000000119053         NA      NA       NA
    ## epic             0.0999999999999751504         NA      NA       NA
    ## epidemic                            NA         NA      NA       NA
    ## episode          0.0999999999999877376         NA      NA       NA
    ## equal                               NA         NA      NA       NA
    ## equipment        0.5454545454544130756         NA      NA       NA
    ## equivalent                          NA         NA      NA       NA
    ## er              -0.0000000000000272653         NA      NA       NA
    ## era                                 NA         NA      NA       NA
    ## eric            -0.0000000000000290981         NA      NA       NA
    ## error           -0.0000000000000560667         NA      NA       NA
    ## escape                              NA         NA      NA       NA
    ## especially       0.0399999999999808703         NA      NA       NA
    ## estate          -0.0000000000000449057         NA      NA       NA
    ## et               0.0666666666666456964         NA      NA       NA
    ## etc              0.0952380952380905976         NA      NA       NA
    ## ethical          0.0624999999999732367         NA      NA       NA
    ## ethics           0.1176470588234833636         NA      NA       NA
    ## eu               0.0819672131147526489         NA      NA       NA
    ## eu4                                 NA         NA      NA       NA
    ## euro             0.1999999999999563238         NA      NA       NA
    ## europe                              NA         NA      NA       NA
    ## `europe's`       0.0434782608695599843         NA      NA       NA
    ## european         0.1111111111111117433         NA      NA       NA
    ## eve              0.5999999999999794387         NA      NA       NA
    ## even                                NA         NA      NA       NA
    ## event            0.0816326530612151158         NA      NA       NA
    ## events                              NA         NA      NA       NA
    ## eventually                          NA         NA      NA       NA
    ## ever                                NA         NA      NA       NA
    ## every            0.0334448160535121067         NA      NA       NA
    ## everybody                           NA         NA      NA       NA
    ## everyday         0.0624999999999740277         NA      NA       NA
    ## everyone                            NA         NA      NA       NA
    ## everything                          NA         NA      NA       NA
    ## everytime                           NA         NA      NA       NA
    ## everywhere                          NA         NA      NA       NA
    ## evidence         0.0370370370370198543         NA      NA       NA
    ## evil             0.0333333333333194828         NA      NA       NA
    ## evolution       -0.0000000000000462895         NA      NA       NA
    ## ex               0.0800000000000044287         NA      NA       NA
    ## exactly          0.1136363636363285495         NA      NA       NA
    ## example          0.3703703703703862815         NA      NA       NA
    ## examples                            NA         NA      NA       NA
    ## excellent                           NA         NA      NA       NA
    ## except          -0.0000000000000160351         NA      NA       NA
    ## exchange                            NA         NA      NA       NA
    ## excited          0.4545454545454638007         NA      NA       NA
    ## exciting                            NA         NA      NA       NA
    ## exclusive                           NA         NA      NA       NA
    ## exhibit         -0.0000000000000684164         NA      NA       NA
    ## exist                               NA         NA      NA       NA
    ## existed         -0.0000000000000321553         NA      NA       NA
    ## existence        0.0909090909090538718         NA      NA       NA
    ## exists                              NA         NA      NA       NA
    ## exit                                NA         NA      NA       NA
    ## expect                              NA         NA      NA       NA
    ## expected         0.1428571428571340785         NA      NA       NA
    ## expecting       -0.0000000000000470318         NA      NA       NA
    ## expense                             NA         NA      NA       NA
    ## expenses        -0.0000000000000367747         NA      NA       NA
    ## expensive                           NA         NA      NA       NA
    ## experience       0.1470588235294189028         NA      NA       NA
    ## experiences     -0.0000000000000117482         NA      NA       NA
    ## expert           0.0357142857142632511         NA      NA       NA
    ## experts          0.1290322580644887307         NA      NA       NA
    ## explain          0.2702702702702980408         NA      NA       NA
    ## explained        0.0869565217391024964         NA      NA       NA
    ## explaining      -0.0000000000000390164         NA      NA       NA
    ## explanation      0.0714285714285121803         NA      NA       NA
    ## explosion       -0.0000000000000475853         NA      NA       NA
    ## exposed                             NA         NA      NA       NA
    ## extended        -0.0000000000000354226         NA      NA       NA
    ## extra                               NA         NA      NA       NA
    ## extreme          0.1176470588234717896         NA      NA       NA
    ## extremely        0.0909090909090556065         NA      NA       NA
    ## eye                                 NA         NA      NA       NA
    ## eyes             0.2777777777777021284         NA      NA       NA
    ## f                0.1388888888889058260         NA      NA       NA
    ## fa                                  NA         NA      NA       NA
    ## face                                NA         NA      NA       NA
    ## facebook         0.0621118012422343310         NA      NA       NA
    ## faces            0.3030303030303288514         NA      NA       NA
    ## facial           0.0909090909090848054         NA      NA       NA
    ## facility         0.2727272727272423425         NA      NA       NA
    ## facing                              NA         NA      NA       NA
    ## fact                                NA         NA      NA       NA
    ## factor           0.3636363636363140195         NA      NA       NA
    ## factory          0.1999999999999615141         NA      NA       NA
    ## facts            0.6666666666666473118         NA      NA       NA
    ## fail             0.0714285714285453066         NA      NA       NA
    ## failed                              NA         NA      NA       NA
    ## fair             0.1372549019607965382         NA      NA       NA
    ## faith                               NA         NA      NA       NA
    ## fake             0.1041666666666641317         NA      NA       NA
    ## fall                                NA         NA      NA       NA
    ## falling          0.1999999999999463318         NA      NA       NA
    ## falls            0.0999999999999548889         NA      NA       NA
    ## false           -0.0000000000000276979         NA      NA       NA
    ## family           0.1111111111110990590         NA      NA       NA
    ## famous                              NA         NA      NA       NA
    ## fan              0.0909090909090826960         NA      NA       NA
    ## fans             0.1212121212121147346         NA      NA       NA
    ## fantastic                           NA         NA      NA       NA
    ## far                                 NA         NA      NA       NA
    ## farm            -0.0000000000000190705         NA      NA       NA
    ## fast                                NA         NA      NA       NA
    ## faster          -0.0000000000000254086         NA      NA       NA
    ## father                              NA         NA      NA       NA
    ## favor            0.0666666666666371199         NA      NA       NA
    ## favorite         0.0568181818181806922         NA      NA       NA
    ## favourite                           NA         NA      NA       NA
    ## fb              -0.0000000000000516262         NA      NA       NA
    ## fbi                                 NA         NA      NA       NA
    ## fc               0.2068965517241384000         NA      NA       NA
    ## fear             0.1276595744680646127         NA      NA       NA
    ## fears            0.1142857142856943259         NA      NA       NA
    ## feat            -0.0000000000000291879         NA      NA       NA
    ## feature          0.1199999999999895595         NA      NA       NA
    ## features        -0.0000000000000212188         NA      NA       NA
    ## featuring       -0.0000000000000362016         NA      NA       NA
    ## feb                                 NA         NA      NA       NA
    ## february         0.0909090909090683047         NA      NA       NA
    ## federal          0.2000000000000065059         NA      NA       NA
    ## fee             -0.0000000000000413578         NA      NA       NA
    ## feedback        -0.0000000000000136647         NA      NA       NA
    ## feel             0.0427350427350436865         NA      NA       NA
    ## feeling                             NA         NA      NA       NA
    ## feelings         0.0714285714285917001         NA      NA       NA
    ## feels            0.1794871794871766013         NA      NA       NA
    ## feet                                NA         NA      NA       NA
    ## fell                                NA         NA      NA       NA
    ## fellow           0.0999999999999772876         NA      NA       NA
    ## felt             0.2666666666666314689         NA      NA       NA
    ## female           0.1219512195121843778         NA      NA       NA
    ## fever            0.0714285714285399082         NA      NA       NA
    ## fi               0.0238095238095141735         NA      NA       NA
    ## fiction                             NA         NA      NA       NA
    ## fictional                           NA         NA      NA       NA
    ## field                               NA         NA      NA       NA
    ## fields          -0.0000000000000377367         NA      NA       NA
    ## fifa             0.2702702702702657334         NA      NA       NA
    ## fight                               NA         NA      NA       NA
    ## fighter          0.0384615384615155306         NA      NA       NA
    ## fighters         0.0526315789473349310         NA      NA       NA
    ## fighting         0.0784313725490136116         NA      NA       NA
    ## fights                              NA         NA      NA       NA
    ## figure           0.0666666666666567015         NA      NA       NA
    ## figures         -0.0000000000000598545         NA      NA       NA
    ## file            -0.0000000000000294757         NA      NA       NA
    ## filed                               NA         NA      NA       NA
    ## film             0.0357142857142806816         NA      NA       NA
    ## filmed                              NA         NA      NA       NA
    ## filming                             NA         NA      NA       NA
    ## filmmaker                           NA         NA      NA       NA
    ## films            0.1162790697674358220         NA      NA       NA
    ## filter           0.0909090909090601029         NA      NA       NA
    ## final            0.2222222222222247912         NA      NA       NA
    ## finally                             NA         NA      NA       NA
    ## finance          0.0499999999999766534         NA      NA       NA
    ## finances         0.0999999999999318240         NA      NA       NA
    ## financial                           NA         NA      NA       NA
    ## financially     -0.0000000000000308765         NA      NA       NA
    ## find                                NA         NA      NA       NA
    ## finding          0.0833333333333225457         NA      NA       NA
    ## finds                               NA         NA      NA       NA
    ## fine                                NA         NA      NA       NA
    ## finish           0.6666666666665820307         NA      NA       NA
    ## finished         0.4166666666666509200         NA      NA       NA
    ## finland         -0.0000000000000372570         NA      NA       NA
    ## fire             0.0677966101694750939         NA      NA       NA
    ## fired                               NA         NA      NA       NA
    ## fires                               NA         NA      NA       NA
    ## firm                                NA         NA      NA       NA
    ## firms                               NA         NA      NA       NA
    ## first                               NA         NA      NA       NA
    ## fish                                NA         NA      NA       NA
    ## fit             -0.0000000000000241379         NA      NA       NA
    ## fits            -0.0000000000000399360         NA      NA       NA
    ## five             0.1388888888888600848         NA      NA       NA
    ## fix              0.0196078431372339393         NA      NA       NA
    ## fixed           -0.0000000000000803565         NA      NA       NA
    ## fixture                             NA         NA      NA       NA
    ## fl                                  NA         NA      NA       NA
    ## flag                                NA         NA      NA       NA
    ## flamengo                            NA         NA      NA       NA
    ## flash                               NA         NA      NA       NA
    ## flat            -0.0000000000000411227         NA      NA       NA
    ## flight                              NA         NA      NA       NA
    ## flights          0.3333333333332961779         NA      NA       NA
    ## floor                               NA         NA      NA       NA
    ## florida                             NA         NA      NA       NA
    ## flu              0.0069881201956666040         NA      NA       NA
    ## fluminense                          NA         NA      NA       NA
    ## fly              0.0476190476190308867         NA      NA       NA
    ## flying           0.0666666666666383967         NA      NA       NA
    ## fmla            -0.0000000000000373582         NA      NA       NA
    ## focus                               NA         NA      NA       NA
    ## focused                             NA         NA      NA       NA
    ## folk                                NA         NA      NA       NA
    ## folks                               NA         NA      NA       NA
    ## follow           0.3461538461538419265         NA      NA       NA
    ## following        0.0833333333333182019         NA      NA       NA
    ## food                                NA         NA      NA       NA
    ## fool             0.0000000000000226877         NA      NA       NA
    ## foot            -0.0000000000000128207         NA      NA       NA
    ## footage                             NA         NA      NA       NA
    ## football         0.1219512195121915110         NA      NA       NA
    ## force                               NA         NA      NA       NA
    ## forced                              NA         NA      NA       NA
    ## forces          -0.0000000000000317925         NA      NA       NA
    ## forcing                             NA         NA      NA       NA
    ## ford                                NA         NA      NA       NA
    ## foreign          0.1960784313725381367         NA      NA       NA
    ## forever                             NA         NA      NA       NA
    ## forget                              NA         NA      NA       NA
    ## forgot           0.1249999999999982653         NA      NA       NA
    ## form                                NA         NA      NA       NA
    ## former           0.1578947368421015351         NA      NA       NA
    ## forward          0.0384615384615184241         NA      NA       NA
    ## fossil           0.1666666666666581642         NA      NA       NA
    ## found                               NA         NA      NA       NA
    ## founder          0.5999999999999583444         NA      NA       NA
    ## four             0.1428571428571383528         NA      NA       NA
    ## fourth                              NA         NA      NA       NA
    ## fox              0.3571428571428915122         NA      NA       NA
    ## frame                               NA         NA      NA       NA
    ## france           0.1587301587301557515         NA      NA       NA
    ## franchise                           NA         NA      NA       NA
    ## fraud            0.0681818181818058394         NA      NA       NA
    ## freak           -0.0000000000000638670         NA      NA       NA
    ## freaking                            NA         NA      NA       NA
    ## freakout         0.0399999999999827299         NA      NA       NA
    ## freaks                              NA         NA      NA       NA
    ## free             0.0465116279069801297         NA      NA       NA
    ## freedom                             NA         NA      NA       NA
    ## french           0.0649350649350564496         NA      NA       NA
    ## fresh                               NA         NA      NA       NA
    ## friday           0.2173913043478061802         NA      NA       NA
    ## friend                              NA         NA      NA       NA
    ## friendly                            NA         NA      NA       NA
    ## friends          0.0487804878048730267         NA      NA       NA
    ## front                               NA         NA      NA       NA
    ## frontline       -0.0000000000000372762         NA      NA       NA
    ## frozen           0.3333333333332797466         NA      NA       NA
    ## ft                                  NA         NA      NA       NA
    ## fuck             0.2040816326530607572         NA      NA       NA
    ## fucked           0.5263157894736876230         NA      NA       NA
    ## fucking          0.1333333333333217297         NA      NA       NA
    ## fuel             0.1538461538460659805         NA      NA       NA
    ## full             0.0884955752212359392         NA      NA       NA
    ## fully            0.2307692307692169320         NA      NA       NA
    ## fun              0.0740740740740725434         NA      NA       NA
    ## fund            -0.0000000000000261869         NA      NA       NA
    ## funding                             NA         NA      NA       NA
    ## funds                               NA         NA      NA       NA
    ## funny            0.2564102564102465065         NA      NA       NA
    ## future           0.0326797385620909639         NA      NA       NA
    ## ga                                  NA         NA      NA       NA
    ## gain                                NA         NA      NA       NA
    ## game                                NA         NA      NA       NA
    ## games                               NA         NA      NA       NA
    ## gaming                              NA         NA      NA       NA
    ## gang             0.0599999999999849890         NA      NA       NA
    ## gap             -0.0000000000000350116         NA      NA       NA
    ## garage          -0.0000000000000354334         NA      NA       NA
    ## gary             0.0999999999999251626         NA      NA       NA
    ## gas                                 NA         NA      NA       NA
    ## gate                                NA         NA      NA       NA
    ## gatekeeping                         NA         NA      NA       NA
    ## gates            0.0588235294117112478         NA      NA       NA
    ## gave                                NA         NA      NA       NA
    ## gay              0.1111111111111034722         NA      NA       NA
    ## gear                                NA         NA      NA       NA
    ## gem              0.1249999999999707873         NA      NA       NA
    ## gender           0.3999999999999850897         NA      NA       NA
    ## gene            -0.0000000000000353995         NA      NA       NA
    ## general                             NA         NA      NA       NA
    ## generally        0.8333333333333026172         NA      NA       NA
    ## generate         0.0399999999999688313         NA      NA       NA
    ## generated        0.1587301587301513106         NA      NA       NA
    ## generation                          NA         NA      NA       NA
    ## genetic         -0.0000000000000386709         NA      NA       NA
    ## genre            0.4999999999999614197         NA      NA       NA
    ## genuinely        0.9999999999999740208         NA      NA       NA
    ## george                              NA         NA      NA       NA
    ## georgia          0.4761904761904617311         NA      NA       NA
    ## german                              NA         NA      NA       NA
    ## germany                             NA         NA      NA       NA
    ## get              0.0139470013946996155         NA      NA       NA
    ## gets             0.0523560209424042342         NA      NA       NA
    ## getting                             NA         NA      NA       NA
    ## gf              -0.0000000000000357613         NA      NA       NA
    ## ghost            0.0999999999999761496         NA      NA       NA
    ## giant            0.0370370370370160379         NA      NA       NA
    ## gift            -0.0000000000000251739         NA      NA       NA
    ## girl             0.0877192982455964471         NA      NA       NA
    ## girlfriend       0.0882352941176315908         NA      NA       NA
    ## girls            0.1851851851851748776         NA      NA       NA
    ## give                                NA         NA      NA       NA
    ## given                               NA         NA      NA       NA
    ## gives                               NA         NA      NA       NA
    ## giving                              NA         NA      NA       NA
    ## glass           -0.0000000000000242575         NA      NA       NA
    ## glasses                             NA         NA      NA       NA
    ## global                              NA         NA      NA       NA
    ## go               0.0317460317460322439         NA      NA       NA
    ## goal                                NA         NA      NA       NA
    ## goals           -0.0000000000000411797         NA      NA       NA
    ## god              0.1063829787233863955         NA      NA       NA
    ## goes                                NA         NA      NA       NA
    ## going                               NA         NA      NA       NA
    ## gold             0.1176470588235080245         NA      NA       NA
    ## golden                              NA         NA      NA       NA
    ## gone             0.0769230769230583311         NA      NA       NA
    ## gonna                               NA         NA      NA       NA
    ## good             0.0256410256410237701         NA      NA       NA
    ## goods                               NA         NA      NA       NA
    ## google           0.0432900432900445858         NA      NA       NA
    ## `google's`       0.0333333333333190526         NA      NA       NA
    ## gop             -0.0000000000000138791         NA      NA       NA
    ## got              0.0303030303030309041         NA      NA       NA
    ## gotta            0.0952380952380673107         NA      NA       NA
    ## gotten                              NA         NA      NA       NA
    ## gov                                 NA         NA      NA       NA
    ## government                          NA         NA      NA       NA
    ## governments                         NA         NA      NA       NA
    ## governor                            NA         NA      NA       NA
    ## grab                                NA         NA      NA       NA
    ## grad            -0.0000000000000500852         NA      NA       NA
    ## grade                               NA         NA      NA       NA
    ## graduate                            NA         NA      NA       NA
    ## graham                              NA         NA      NA       NA
    ## grand                               NA         NA      NA       NA
    ## grandma          0.2499999999999476807         NA      NA       NA
    ## granted         -0.0000000000000718569         NA      NA       NA
    ## great            0.0671140939597322933         NA      NA       NA
    ## greatest                            NA         NA      NA       NA
    ## greek                               NA         NA      NA       NA
    ## green            0.2702702702702436954         NA      NA       NA
    ## grenade                             NA         NA      NA       NA
    ## greta            0.0243309002433072415         NA      NA       NA
    ## grossing         0.6999999999999918510         NA      NA       NA
    ## ground                              NA         NA      NA       NA
    ## group                               NA         NA      NA       NA
    ## grow                                NA         NA      NA       NA
    ## growing          0.2941176470588390823         NA      NA       NA
    ## grows                               NA         NA      NA       NA
    ## growth                              NA         NA      NA       NA
    ## gt              -0.0000000000000395986         NA      NA       NA
    ## guess                               NA         NA      NA       NA
    ## guide                               NA         NA      NA       NA
    ## guilty           0.0476190476190044912         NA      NA       NA
    ## gun              0.1428571428571343838         NA      NA       NA
    ## guns             0.0555555555555082986         NA      NA       NA
    ## guy              0.0555555555555529296         NA      NA       NA
    ## guys             0.0892857142857056973         NA      NA       NA
    ## gym             -0.0000000000000078585         NA      NA       NA
    ## h                                   NA         NA      NA       NA
    ## h1n1                                NA         NA      NA       NA
    ## hack             0.3999999999999495626         NA      NA       NA
    ## hacked           0.3333333333333091675         NA      NA       NA
    ## hackers          0.2083333333333158566         NA      NA       NA
    ## hacking          0.2916666666666442587         NA      NA       NA
    ## hair             0.2307692307692054134         NA      NA       NA
    ## half             0.1818181818181688059         NA      NA       NA
    ## hall                                NA         NA      NA       NA
    ## halloween        0.0666666666666686780         NA      NA       NA
    ## hampshire        0.0000000000000190204         NA      NA       NA
    ## hand             0.0869565217391087553         NA      NA       NA
    ## handle          -0.0000000000000380992         NA      NA       NA
    ## handling         0.0769230769230519751         NA      NA       NA
    ## hands            0.2499999999999796274         NA      NA       NA
    ## hang            -0.0000000000000085077         NA      NA       NA
    ## happen                              NA         NA      NA       NA
    ## happened                            NA         NA      NA       NA
    ## happening        0.3846153846153796962         NA      NA       NA
    ## happens                             NA         NA      NA       NA
    ## happy            0.0967741935483829446         NA      NA       NA
    ## hard                                NA         NA      NA       NA
    ## harder          -0.0000000000000304844         NA      NA       NA
    ## harm             0.1111111111110918287         NA      NA       NA
    ## harmful         -0.0000000000000476686         NA      NA       NA
    ## harry                               NA         NA      NA       NA
    ## harvard         -0.0000000000000095676         NA      NA       NA
    ## hasn            -0.0000000000000260679         NA      NA       NA
    ## hat                                 NA         NA      NA       NA
    ## hate             0.1030927835051519970         NA      NA       NA
    ## hated            0.9999999999999813483         NA      NA       NA
    ## hates            0.2307692307691531219         NA      NA       NA
    ## haven                               NA         NA      NA       NA
    ## hawkins          0.0000000000000005015         NA      NA       NA
    ## hd                                  NA         NA      NA       NA
    ## head             0.1666666666666503094         NA      NA       NA
    ## heads            0.1874999999999736600         NA      NA       NA
    ## health           0.0826446280991734450         NA      NA       NA
    ## healthcare       0.1052631578947161445         NA      NA       NA
    ## healthy                             NA         NA      NA       NA
    ## hear                                NA         NA      NA       NA
    ## heard            0.2325581395348735592         NA      NA       NA
    ## hearing          0.0909090909090790183         NA      NA       NA
    ## heart            0.0624999999999655553         NA      NA       NA
    ## heat                                NA         NA      NA       NA
    ## heavy           -0.0000000000000263610         NA      NA       NA
    ## hedgehog         0.0769230769230417194         NA      NA       NA
    ## held                                NA         NA      NA       NA
    ## hell                                NA         NA      NA       NA
    ## help                                NA         NA      NA       NA
    ## helped                              NA         NA      NA       NA
    ## helping                             NA         NA      NA       NA
    ## helps            0.0454545454545230362         NA      NA       NA
    ## hero             0.1304347826086679762         NA      NA       NA
    ## hey              0.0303030303030167106         NA      NA       NA
    ## hi                                  NA         NA      NA       NA
    ## hidden           0.1599999999999907330         NA      NA       NA
    ## hide                                NA         NA      NA       NA
    ## high             0.0763358778625943674         NA      NA       NA
    ## higher           0.1052631578947158530         NA      NA       NA
    ## highest                             NA         NA      NA       NA
    ## highly           0.1666666666666460350         NA      NA       NA
    ## hilarious                           NA         NA      NA       NA
    ## hill            -0.0000000000000345815         NA      NA       NA
    ## hillary          0.1666666666666636043         NA      NA       NA
    ## hip                                 NA         NA      NA       NA
    ## hire             0.0714285714286396894         NA      NA       NA
    ## hired           -0.0000000000000360615         NA      NA       NA
    ## historic         0.1818181818181081599         NA      NA       NA
    ## historical       0.0624999999999726052         NA      NA       NA
    ## history          0.1020408163265216078         NA      NA       NA
    ## hit              0.1086956521739142340         NA      NA       NA
    ## hitler                              NA         NA      NA       NA
    ## hits             0.1999999999999723665         NA      NA       NA
    ## hiv                                 NA         NA      NA       NA
    ## hmc             -0.0000000000000685830         NA      NA       NA
    ## hmrb                                NA         NA      NA       NA
    ## hoa                                 NA         NA      NA       NA
    ## hold                                NA         NA      NA       NA
    ## holding          0.1538461538461149691         NA      NA       NA
    ## holds           -0.0000000000000506294         NA      NA       NA
    ## hole                                NA         NA      NA       NA
    ## holes                               NA         NA      NA       NA
    ## holiday          0.0499999999999747313         NA      NA       NA
    ## hollywood                           NA         NA      NA       NA
    ## home             0.0534759358288745129         NA      NA       NA
    ## homecoming       0.4999999999999524825         NA      NA       NA
    ## homes           -0.0000000000000459811         NA      NA       NA
    ## honest           0.1428571428571250856         NA      NA       NA
    ## honestly                            NA         NA      NA       NA
    ## hong             0.1515151515151435813         NA      NA       NA
    ## honor                               NA         NA      NA       NA
    ## hop                                 NA         NA      NA       NA
    ## hope             0.0923076923076911054         NA      NA       NA
    ## hopes                               NA         NA      NA       NA
    ## hoping          -0.0000000000000321744         NA      NA       NA
    ## hopper          -0.0000000000000296912         NA      NA       NA
    ## horrible                            NA         NA      NA       NA
    ## horror           0.0699300699300675899         NA      NA       NA
    ## hospital         0.0967741935483860810         NA      NA       NA
    ## hospitalized     0.0909090909090407850         NA      NA       NA
    ## hospitals                           NA         NA      NA       NA
    ## host                                NA         NA      NA       NA
    ## hostile                             NA         NA      NA       NA
    ## hot                                 NA         NA      NA       NA
    ## hotel            0.1034482758620570014         NA      NA       NA
    ## hour                                NA         NA      NA       NA
    ## hours            0.0919540229885022764         NA      NA       NA
    ## house            0.0558659217877078285         NA      NA       NA
    ## housing                             NA         NA      NA       NA
    ## houston                             NA         NA      NA       NA
    ## however         -0.0000000000000277354         NA      NA       NA
    ## hr              -0.0000000000000431564         NA      NA       NA
    ## hsa             -0.0000000000000167482         NA      NA       NA
    ## huge                                NA         NA      NA       NA
    ## hulu                                NA         NA      NA       NA
    ## human            0.0406504065040643944         NA      NA       NA
    ## humanity         0.0338983050847375331         NA      NA       NA
    ## humans           0.0703124999999978212         NA      NA       NA
    ## hundreds                            NA         NA      NA       NA
    ## hurt                                NA         NA      NA       NA
    ## husband          0.0555555555555346733         NA      NA       NA
    ## hype                                NA         NA      NA       NA
    ## ibm             -0.0000000000000137611         NA      NA       NA
    ## ice              0.0555555555555264716         NA      NA       NA
    ## iconic                              NA         NA      NA       NA
    ## id                                  NA         NA      NA       NA
    ## idea                                NA         NA      NA       NA
    ## ideas                               NA         NA      NA       NA
    ## identify                            NA         NA      NA       NA
    ## idiot            0.0666666666666118485         NA      NA       NA
    ## idk                                 NA         NA      NA       NA
    ## ignore           0.0714285714285411155         NA      NA       NA
    ## ii                                  NA         NA      NA       NA
    ## il              -0.0000000000000254450         NA      NA       NA
    ## ill              0.2857142857142913606         NA      NA       NA
    ## illegal          0.1777777777777592716         NA      NA       NA
    ## illinois         0.0833333333332936799         NA      NA       NA
    ## illness                             NA         NA      NA       NA
    ## illnesses        0.1999999999999421685         NA      NA       NA
    ## im                                  NA         NA      NA       NA
    ## image            0.1886792452830021449         NA      NA       NA
    ## images           0.1142857142856969627         NA      NA       NA
    ## imagine          0.0909090909090668614         NA      NA       NA
    ## imdb             0.0476190476190362574         NA      NA       NA
    ## immediately                         NA         NA      NA       NA
    ## immune                              NA         NA      NA       NA
    ## impact           0.0408163265306018611         NA      NA       NA
    ## impeached       -0.0000000000000648675         NA      NA       NA
    ## impeachment      0.0882352941176429290         NA      NA       NA
    ## implications    -0.0000000000000739703         NA      NA       NA
    ## important        0.0909090909090696647         NA      NA       NA
    ## impossible       0.0769230769230612316         NA      NA       NA
    ## impressive       0.2999999999999574118         NA      NA       NA
    ## improve          0.2631578947368635735         NA      NA       NA
    ## include         -0.0000000000000242282         NA      NA       NA
    ## included        -0.0000000000000595063         NA      NA       NA
    ## includes                            NA         NA      NA       NA
    ## including                           NA         NA      NA       NA
    ## income                              NA         NA      NA       NA
    ## increase                            NA         NA      NA       NA
    ## incredible       0.0476190476190309075         NA      NA       NA
    ## incredibly                          NA         NA      NA       NA
    ## indeed           0.0833333333333084320         NA      NA       NA
    ## independent                         NA         NA      NA       NA
    ## index                               NA         NA      NA       NA
    ## india           -0.0000000000000215982         NA      NA       NA
    ## indian                              NA         NA      NA       NA
    ## indiana                             NA         NA      NA       NA
    ## indie            0.0769230769230687950         NA      NA       NA
    ## individual       0.0555555555555248270         NA      NA       NA
    ## individuals      0.0555555555555162367         NA      NA       NA
    ## industrial       0.2727272727272287978         NA      NA       NA
    ## industry                            NA         NA      NA       NA
    ## inevitable      -0.0000000000000459733         NA      NA       NA
    ## infantino       -0.0000000000000522217         NA      NA       NA
    ## infected         0.1724137931034503968         NA      NA       NA
    ## infection        0.0370370370370057614         NA      NA       NA
    ## infections       0.0555555555555417996         NA      NA       NA
    ## infinity         0.0526315789473418769         NA      NA       NA
    ## influence                           NA         NA      NA       NA
    ## influenza        0.1351351351351426089         NA      NA       NA
    ## info                                NA         NA      NA       NA
    ## information      0.1612903225806621710         NA      NA       NA
    ## infrastructure   0.3999999999999483413         NA      NA       NA
    ## injured         -0.0000000000000456965         NA      NA       NA
    ## injury                              NA         NA      NA       NA
    ## innovation      -0.0000000000000298750         NA      NA       NA
    ## insane          -0.0000000000000850062         NA      NA       NA
    ## inside           0.0843373493975835553         NA      NA       NA
    ## insight         -0.0000000000000435617         NA      NA       NA
    ## inspiration     -0.0000000000000360138         NA      NA       NA
    ## inspired         0.0294117647058921398         NA      NA       NA
    ## instagram        0.0344827586206592276         NA      NA       NA
    ## instant         -0.0000000000000610168         NA      NA       NA
    ## instantly                           NA         NA      NA       NA
    ## instead          0.0608695652173879068         NA      NA       NA
    ## institute                           NA         NA      NA       NA
    ## insult                              NA         NA      NA       NA
    ## insurance                           NA         NA      NA       NA
    ## integrity                           NA         NA      NA       NA
    ## intelligence                        NA         NA      NA       NA
    ## intelligent                         NA         NA      NA       NA
    ## intense         -0.0000000000000376192         NA      NA       NA
    ## interactive     -0.0000000000000294885         NA      NA       NA
    ## interest         0.0526315789473604800         NA      NA       NA
    ## interested       0.0689655172413637246         NA      NA       NA
    ## interesting      0.0526315789473593421         NA      NA       NA
    ## interfere        0.2999999999999495848         NA      NA       NA
    ## interference                        NA         NA      NA       NA
    ## international                       NA         NA      NA       NA
    ## internet                            NA         NA      NA       NA
    ## interview        0.1568627450980318305         NA      NA       NA
    ## interviews       0.0000000000000020360         NA      NA       NA
    ## intro           -0.0000000000000186485         NA      NA       NA
    ## invented                            NA         NA      NA       NA
    ## invest           0.1034482758620521164         NA      NA       NA
    ## investigation    0.1363636363636107351         NA      NA       NA
    ## investing       -0.0000000000000909489         NA      NA       NA
    ## investment                          NA         NA      NA       NA
    ## investments     -0.0000000000000338784         NA      NA       NA
    ## involved                            NA         NA      NA       NA
    ## iowa                                NA         NA      NA       NA
    ## ip               0.2999999999999816147         NA      NA       NA
    ## iphone                              NA         NA      NA       NA
    ## ira             -0.0000000000000184022         NA      NA       NA
    ## iran                                NA         NA      NA       NA
    ## ireland          0.0357142857142702663         NA      NA       NA
    ## irish            0.1666666666666590246         NA      NA       NA
    ## irl             -0.0000000000000430521         NA      NA       NA
    ## iron                                NA         NA      NA       NA
    ## island                              NA         NA      NA       NA
    ## isn              0.2121212121211926149         NA      NA       NA
    ## israel           0.2499999999999718836         NA      NA       NA
    ## issue            0.1914893617021109495         NA      NA       NA
    ## issues                              NA         NA      NA       NA
    ## italian                             NA         NA      NA       NA
    ## italy            0.0595238095238076678         NA      NA       NA
    ## items           -0.0000000000000313934         NA      NA       NA
    ## iw              -0.0000000000000389900         NA      NA       NA
    ## j                                   NA         NA      NA       NA
    ## `j'ai`                              NA         NA      NA       NA
    ## `j'en`          -0.0000000000000333032         NA      NA       NA
    ## jack                                NA         NA      NA       NA
    ## jail             0.0769230769230404843         NA      NA       NA
    ## james            0.1111111111110993088         NA      NA       NA
    ## january                             NA         NA      NA       NA
    ## japan            0.1379310344827338330         NA      NA       NA
    ## japanese         0.0303030303030173941         NA      NA       NA
    ## jason                               NA         NA      NA       NA
    ## jav                                 NA         NA      NA       NA
    ## jazz                                NA         NA      NA       NA
    ## jedi                                NA         NA      NA       NA
    ## jesus                               NA         NA      NA       NA
    ## job                                 NA         NA      NA       NA
    ## jobs                                NA         NA      NA       NA
    ## joe              0.0487804878048677185         NA      NA       NA
    ## john                                NA         NA      NA       NA
    ## johnny                              NA         NA      NA       NA
    ## johnson          0.1034482758620525605         NA      NA       NA
    ## join            -0.0000000000000173908         NA      NA       NA
    ## joke                                NA         NA      NA       NA
    ## joker            0.1694915254237323310         NA      NA       NA
    ## jokes            0.1999999999999295952         NA      NA       NA
    ## jones            0.1666666666666223595         NA      NA       NA
    ## judge                               NA         NA      NA       NA
    ## judges           0.3999999999999734324         NA      NA       NA
    ## july             0.0769230769230345446         NA      NA       NA
    ## jump             0.0454545454545224673         NA      NA       NA
    ## june            -0.0000000000000257930         NA      NA       NA
    ## just             0.0128040973111405807         NA      NA       NA
    ## justice          0.2272727272727096681         NA      NA       NA
    ## juventus                            NA         NA      NA       NA
    ## k               -0.0000000000000431555         NA      NA       NA
    ## kansas           0.9999999999999121814         NA      NA       NA
    ## karma            0.1764705882352772259         NA      NA       NA
    ## keanu            0.2499999999999843459         NA      NA       NA
    ## keep             0.0943396226415100514         NA      NA       NA
    ## keeping          0.0222222222222118529         NA      NA       NA
    ## keeps            0.0476190476190270842         NA      NA       NA
    ## kept                                NA         NA      NA       NA
    ## kevin            0.0416666666666421770         NA      NA       NA
    ## key              0.0714285714285578521         NA      NA       NA
    ## khabib                              NA         NA      NA       NA
    ## kick            -0.0000000000000157242         NA      NA       NA
    ## kid              0.0999999999999905409         NA      NA       NA
    ## kids             0.0808080808080771651         NA      NA       NA
    ## kill             0.1369863013698704168         NA      NA       NA
    ## killed           0.0833333333333180215         NA      NA       NA
    ## killer           0.0555555555555294137         NA      NA       NA
    ## killing          0.1111111111110819200         NA      NA       NA
    ## kills            0.0606060606060476945         NA      NA       NA
    ## killstreaks                         NA         NA      NA       NA
    ## kind                                NA         NA      NA       NA
    ## kinda            0.1333333333333139858         NA      NA       NA
    ## kinds                               NA         NA      NA       NA
    ## king                                NA         NA      NA       NA
    ## kiss             0.0999999999999183209         NA      NA       NA
    ## kit             -0.0000000000000658083         NA      NA       NA
    ## kitten          -0.0000000000000336159         NA      NA       NA
    ## kitty                               NA         NA      NA       NA
    ## knew             0.3703703703703584704         NA      NA       NA
    ## knife           -0.0000000000000478692         NA      NA       NA
    ## know             0.0258397932816528987         NA      NA       NA
    ## knowing         -0.0000000000000319510         NA      NA       NA
    ## knowledge        0.0740740740740577219         NA      NA       NA
    ## known            0.1621621621621164577         NA      NA       NA
    ## knows                               NA         NA      NA       NA
    ## kong             0.1562499999999994449         NA      NA       NA
    ## korea                               NA         NA      NA       NA
    ## korean           0.1923076923076798839         NA      NA       NA
    ## kung            -0.0000000000000349912         NA      NA       NA
    ## l                                   NA         NA      NA       NA
    ## la               0.0655737704917968844         NA      NA       NA
    ## lab                                 NA         NA      NA       NA
    ## labor           -0.0000000000000245737         NA      NA       NA
    ## labour           0.0799999999999990719         NA      NA       NA
    ## lack             0.2285714285714155192         NA      NA       NA
    ## lady             0.0937499999999791417         NA      NA       NA
    ## land                                NA         NA      NA       NA
    ## landlord        -0.0000000000000169666         NA      NA       NA
    ## landslide                           NA         NA      NA       NA
    ## language                            NA         NA      NA       NA
    ## languages        0.3999999999999284128         NA      NA       NA
    ## laptop          -0.0000000000000712192         NA      NA       NA
    ## large            0.0344827586206686576         NA      NA       NA
    ## largest          0.1499999999999847566         NA      NA       NA
    ## last                                NA         NA      NA       NA
    ## late             0.0454545454545343536         NA      NA       NA
    ## later                               NA         NA      NA       NA
    ## latest                              NA         NA      NA       NA
    ## laugh                               NA         NA      NA       NA
    ## laughing                            NA         NA      NA       NA
    ## launch          -0.0000000000000478844         NA      NA       NA
    ## launches         0.2799999999999951972         NA      NA       NA
    ## law              0.1176470588235273979         NA      NA       NA
    ## laws                                NA         NA      NA       NA
    ## lawsuit          0.0624999999999722097         NA      NA       NA
    ## lawyer           0.0588235294117355270         NA      NA       NA
    ## le               0.0555555555555282896         NA      NA       NA
    ## lead             0.0847457627118547024         NA      NA       NA
    ## leader           0.1351351351351202656         NA      NA       NA
    ## leaders                             NA         NA      NA       NA
    ## leading          0.0909090909090771865         NA      NA       NA
    ## leads            0.1111111111110955202         NA      NA       NA
    ## league           0.0619469026548616422         NA      NA       NA
    ## leak             0.0999999999999474087         NA      NA       NA
    ## leaked           0.1999999999999817757         NA      NA       NA
    ## learn                               NA         NA      NA       NA
    ## learned          0.1052631578947232777         NA      NA       NA
    ## learning                            NA         NA      NA       NA
    ## learns          -0.0000000000000228587         NA      NA       NA
    ## lease            0.0869565217391084500         NA      NA       NA
    ## least            0.1369863013698561227         NA      NA       NA
    ## leave                               NA         NA      NA       NA
    ## leaves                              NA         NA      NA       NA
    ## leaving                             NA         NA      NA       NA
    ## led              0.3888888888888721307         NA      NA       NA
    ## lee                                 NA         NA      NA       NA
    ## left             0.0769230769230698636         NA      NA       NA
    ## leg              0.0833333333332902243         NA      NA       NA
    ## legal                               NA         NA      NA       NA
    ## legality        -0.0000000000000444480         NA      NA       NA
    ## legally                             NA         NA      NA       NA
    ## legend           0.1578947368420873243         NA      NA       NA
    ## legendary                           NA         NA      NA       NA
    ## legitimate                          NA         NA      NA       NA
    ## lego            -0.0000000000000208594         NA      NA       NA
    ## length                              NA         NA      NA       NA
    ## less             0.1199999999999943889         NA      NA       NA
    ## let              0.0757575757575729425         NA      NA       NA
    ## lets                                NA         NA      NA       NA
    ## letter                              NA         NA      NA       NA
    ## level            0.0641025641025595033         NA      NA       NA
    ## levels                              NA         NA      NA       NA
    ## liberal                             NA         NA      NA       NA
    ## libertarian     -0.0000000000000392866         NA      NA       NA
    ## library          0.2727272727272557762         NA      NA       NA
    ## license                             NA         NA      NA       NA
    ## lie                                 NA         NA      NA       NA
    ## lied                                NA         NA      NA       NA
    ## lies                                NA         NA      NA       NA
    ## life             0.0389105058365753834         NA      NA       NA
    ## lifetime         0.3529411764705597809         NA      NA       NA
    ## liga             0.0833333333332914872         NA      NA       NA
    ## light            0.0930232558139437032         NA      NA       NA
    ## lights                              NA         NA      NA       NA
    ## like             0.0096061479346771800         NA      NA       NA
    ## liked                               NA         NA      NA       NA
    ## likely           0.0757575757575574271         NA      NA       NA
    ## likes            0.0294117647058649082         NA      NA       NA
    ## liking                              NA         NA      NA       NA
    ## limit           -0.0000000000000336891         NA      NA       NA
    ## limits                              NA         NA      NA       NA
    ## lindsey                             NA         NA      NA       NA
    ## line             0.1639344262294946675         NA      NA       NA
    ## lines                               NA         NA      NA       NA
    ## link             0.0285714285714574710         NA      NA       NA
    ## links           -0.0000000000000236620         NA      NA       NA
    ## lion                                NA         NA      NA       NA
    ## list             0.0731707317073091862         NA      NA       NA
    ## listen                              NA         NA      NA       NA
    ## listening       -0.0000000000000223880         NA      NA       NA
    ## literally        0.0810810810810854016         NA      NA       NA
    ## little           0.0704225352112604341         NA      NA       NA
    ## live                                NA         NA      NA       NA
    ## lived           -0.0000000000000304207         NA      NA       NA
    ## liverpool        0.3999999999999321876         NA      NA       NA
    ## lives            0.0555555555555091798         NA      NA       NA
    ## living           0.0434782608695549397         NA      NA       NA
    ## ll               0.2666666666666689389         NA      NA       NA
    ## loan             0.0322580645161154658         NA      NA       NA
    ## loans            0.0416666666666364802         NA      NA       NA
    ## lobby                               NA         NA      NA       NA
    ## local            0.0873786407766935502         NA      NA       NA
    ## location         0.2999999999999773959         NA      NA       NA
    ## lockdown        -0.0000000000000146915         NA      NA       NA
    ## locked                              NA         NA      NA       NA
    ## lol              0.0952380952380618567         NA      NA       NA
    ## london           0.4999999999999787947         NA      NA       NA
    ## long                                NA         NA      NA       NA
    ## longer           0.0882352941176451772         NA      NA       NA
    ## look             0.0657894736842044581         NA      NA       NA
    ## looked                              NA         NA      NA       NA
    ## looking                             NA         NA      NA       NA
    ## looks            0.0563380281690164311         NA      NA       NA
    ## lord            -0.0000000000000231599         NA      NA       NA
    ## los                                 NA         NA      NA       NA
    ## lose                                NA         NA      NA       NA
    ## loses            0.2857142857142623282         NA      NA       NA
    ## losing                              NA         NA      NA       NA
    ## loss             0.0454545454545110736         NA      NA       NA
    ## lost             0.1470588235294030544         NA      NA       NA
    ## lot              0.1136363636363573876         NA      NA       NA
    ## lots            -0.0000000000000270039         NA      NA       NA
    ## loud             0.5882352941175341687         NA      NA       NA
    ## love             0.0613496932515327775         NA      NA       NA
    ## loved            0.4347826086957045089         NA      NA       NA
    ## loves            0.0645161290322436576         NA      NA       NA
    ## low                                 NA         NA      NA       NA
    ## lower           -0.0000000000000171392         NA      NA       NA
    ## lying           -0.0000000000000388749         NA      NA       NA
    ## lyrics           0.0476190476190261128         NA      NA       NA
    ## m                0.0584795321637412505         NA      NA       NA
    ## ma              -0.0000000000000297716         NA      NA       NA
    ## machina                             NA         NA      NA       NA
    ## machine                             NA         NA      NA       NA
    ## machines         0.0333333333333325973         NA      NA       NA
    ## mad              0.1538461538461366462         NA      NA       NA
    ## made                                NA         NA      NA       NA
    ## madrid           0.1153846153846004169         NA      NA       NA
    ## magazine                            NA         NA      NA       NA
    ## magic            0.0666666666666334562         NA      NA       NA
    ## mail                                NA         NA      NA       NA
    ## main             0.1538461538461553535         NA      NA       NA
    ## mainstream      -0.0000000000000326069         NA      NA       NA
    ## major            0.0769230769230686701         NA      NA       NA
    ## majority         0.0857142857142753900         NA      NA       NA
    ## make             0.0241545893719795381         NA      NA       NA
    ## maker           -0.0000000000000631586         NA      NA       NA
    ## makes            0.0714285714285675805         NA      NA       NA
    ## making                              NA         NA      NA       NA
    ## male             0.0454545454545330768         NA      NA       NA
    ## mall            -0.0000000000000209074         NA      NA       NA
    ## man              0.0246913580246903852         NA      NA       NA
    ## `man's`         -0.0000000000000432111         NA      NA       NA
    ## manage          -0.0000000000000348339         NA      NA       NA
    ## managed          0.0714285714285402412         NA      NA       NA
    ## management       0.0499999999999794220         NA      NA       NA
    ## manager          0.0408163265306023051         NA      NA       NA
    ## manchester                          NA         NA      NA       NA
    ## mandatory       -0.0000000000000530235         NA      NA       NA
    ## manipulation                        NA         NA      NA       NA
    ## many             0.0595238095238068768         NA      NA       NA
    ## map              0.1219512195121568721         NA      NA       NA
    ## march                               NA         NA      NA       NA
    ## marijuana                           NA         NA      NA       NA
    ## mark                                NA         NA      NA       NA
    ## market           0.0930232558139477972         NA      NA       NA
    ## marketing                           NA         NA      NA       NA
    ## marre                               NA         NA      NA       NA
    ## marriage                            NA         NA      NA       NA
    ## married                             NA         NA      NA       NA
    ## martin                              NA         NA      NA       NA
    ## marvel                              NA         NA      NA       NA
    ## mask                                NA         NA      NA       NA
    ## masks                               NA         NA      NA       NA
    ## mass             0.1250000000000179856         NA      NA       NA
    ## massive          0.0952380952380968010         NA      NA       NA
    ## master          -0.0000000000000200407         NA      NA       NA
    ## match            0.0917431192660501660         NA      NA       NA
    ## matched         -0.0000000000000473971         NA      NA       NA
    ## matches          0.0555555555555198172         NA      NA       NA
    ## material        -0.0000000000000291876         NA      NA       NA
    ## math            -0.0000000000000366477         NA      NA       NA
    ## matrix           0.3999999999999792610         NA      NA       NA
    ## matt                                NA         NA      NA       NA
    ## matter           0.0512820512820323579         NA      NA       NA
    ## matters                             NA         NA      NA       NA
    ## max                                 NA         NA      NA       NA
    ## may                                 NA         NA      NA       NA
    ## maybe            0.0384615384615323019         NA      NA       NA
    ## mayor            0.1739130434782512613         NA      NA       NA
    ## mayoral         -0.0000000000000684380         NA      NA       NA
    ## mcconnell        0.4374999999999096834         NA      NA       NA
    ## mcgregor                            NA         NA      NA       NA
    ## mean                                NA         NA      NA       NA
    ## meaning                             NA         NA      NA       NA
    ## means            0.0384615384615205821         NA      NA       NA
    ## meanwhile       -0.0000000000000346019         NA      NA       NA
    ## measles                             NA         NA      NA       NA
    ## meddling                            NA         NA      NA       NA
    ## media            0.0684931506849349447         NA      NA       NA
    ## medical          0.0612244897959114617         NA      NA       NA
    ## medicare        -0.0000000000000206009         NA      NA       NA
    ## medication      -0.0000000000000393479         NA      NA       NA
    ## medications                         NA         NA      NA       NA
    ## medicine        -0.0000000000000161646         NA      NA       NA
    ## meet                                NA         NA      NA       NA
    ## meeting          0.0370370370370192714         NA      NA       NA
    ## megathread       0.3846153846153881339         NA      NA       NA
    ## member                              NA         NA      NA       NA
    ## members          0.1176470588235134923         NA      NA       NA
    ## meme             0.1632653061224508817         NA      NA       NA
    ## memes           -0.0000000000000222513         NA      NA       NA
    ## memories                            NA         NA      NA       NA
    ## memory                              NA         NA      NA       NA
    ## men              0.0909090909090878585         NA      NA       NA
    ## mental                              NA         NA      NA       NA
    ## mention         -0.0000000000000458476         NA      NA       NA
    ## mentioned        0.0000000000000005516         NA      NA       NA
    ## menu                                NA         NA      NA       NA
    ## mers                                NA         NA      NA       NA
    ## mess             0.5714285714285263218         NA      NA       NA
    ## message          0.1590909090908981804         NA      NA       NA
    ## messages                            NA         NA      NA       NA
    ## messed                              NA         NA      NA       NA
    ## met                                 NA         NA      NA       NA
    ## meta             0.0454545454544994856         NA      NA       NA
    ## metal            0.0217391304347718459         NA      NA       NA
    ## method           0.1764705882352708699         NA      NA       NA
    ## mexican                             NA         NA      NA       NA
    ## mexico           0.4999999999999356071         NA      NA       NA
    ## mi                                  NA         NA      NA       NA
    ## michael                             NA         NA      NA       NA
    ## michel                              NA         NA      NA       NA
    ## michigan                            NA         NA      NA       NA
    ## microsoft        0.0666666666666524965         NA      NA       NA
    ## mid                                 NA         NA      NA       NA
    ## middle           0.0869565217391244233         NA      NA       NA
    ## midterm                             NA         NA      NA       NA
    ## might            0.0357142857142800779         NA      NA       NA
    ## mike             0.2499999999999759914         NA      NA       NA
    ## mil              0.4545454545453754824         NA      NA       NA
    ## milan            0.0476190476190299153         NA      NA       NA
    ## military         0.1842105263157671202         NA      NA       NA
    ## million          0.0833333333333306225         NA      NA       NA
    ## millions         0.0967741935483784899         NA      NA       NA
    ## mind                                NA         NA      NA       NA
    ## mine                                NA         NA      NA       NA
    ## minecraft                           NA         NA      NA       NA
    ## mini            -0.0000000000000353708         NA      NA       NA
    ## minimum         -0.0000000000000356859         NA      NA       NA
    ## minister         0.2222222222221779120         NA      NA       NA
    ## minnesota        0.3636363636363268981         NA      NA       NA
    ## minor           -0.0000000000000199492         NA      NA       NA
    ## minority                            NA         NA      NA       NA
    ## minute                              NA         NA      NA       NA
    ## minutes          0.1111111111111052763         NA      NA       NA
    ## mirror                              NA         NA      NA       NA
    ## misinformation                      NA         NA      NA       NA
    ## miss            -0.0000000000000192380         NA      NA       NA
    ## missed           0.2941176470588153236         NA      NA       NA
    ## missing          0.1304347826086878770         NA      NA       NA
    ## mission                             NA         NA      NA       NA
    ## missions        -0.0000000000000521408         NA      NA       NA
    ## mistake                             NA         NA      NA       NA
    ## mit                                 NA         NA      NA       NA
    ## mitch            0.4705882352940828905         NA      NA       NA
    ## mix             -0.0000000000000289255         NA      NA       NA
    ## mixed                               NA         NA      NA       NA
    ## mma              0.0508474576271210274         NA      NA       NA
    ## mo              -0.0000000000000452648         NA      NA       NA
    ## mobile                              NA         NA      NA       NA
    ## mod              0.3461538461538399836         NA      NA       NA
    ## mode                                NA         NA      NA       NA
    ## model            0.0333333333333119333         NA      NA       NA
    ## models           0.0434782608695522613         NA      NA       NA
    ## moderate         0.0769230769230489497         NA      NA       NA
    ## modern           0.0657894736842030703         NA      NA       NA
    ## mods                                NA         NA      NA       NA
    ## molecular       -0.0000000000000323191         NA      NA       NA
    ## mom              0.0816326530612109802         NA      NA       NA
    ## moment                              NA         NA      NA       NA
    ## moments          0.1538461538461305678         NA      NA       NA
    ## monday           0.1249999999999716893         NA      NA       NA
    ## money                               NA         NA      NA       NA
    ## monitor         -0.0000000000000340186         NA      NA       NA
    ## monster          0.1176470588235049436         NA      NA       NA
    ## month            0.1282051282051197283         NA      NA       NA
    ## monthly                             NA         NA      NA       NA
    ## months           0.1333333333333236170         NA      NA       NA
    ## moon                                NA         NA      NA       NA
    ## moore                               NA         NA      NA       NA
    ## moral           -0.0000000000000294338         NA      NA       NA
    ## morgan           0.0999999999999565126         NA      NA       NA
    ## morning          0.1428571428571316915         NA      NA       NA
    ## moscow           0.3703703703703489780         NA      NA       NA
    ## mostly           0.0322580645161098176         NA      NA       NA
    ## mother                              NA         NA      NA       NA
    ## motion           0.0499999999999766256         NA      NA       NA
    ## mourinho         0.0999999999999578310         NA      NA       NA
    ## mouth           -0.0000000000000337683         NA      NA       NA
    ## move                                NA         NA      NA       NA
    ## moved            0.1739130434782451273         NA      NA       NA
    ## movement                            NA         NA      NA       NA
    ## moves                               NA         NA      NA       NA
    ## movie                               NA         NA      NA       NA
    ## moviepass        0.3636363636363225682         NA      NA       NA
    ## movies           0.0075357950263749149         NA      NA       NA
    ## moving           0.0227272727272613898         NA      NA       NA
    ## mp                                  NA         NA      NA       NA
    ## mr                                  NA         NA      NA       NA
    ## msnbc                               NA         NA      NA       NA
    ## much                                NA         NA      NA       NA
    ## multiplayer                         NA         NA      NA       NA
    ## multiple                            NA         NA      NA       NA
    ## mum                                 NA         NA      NA       NA
    ## murder                              NA         NA      NA       NA
    ## music                               NA         NA      NA       NA
    ## musical                             NA         NA      NA       NA
    ## musicians        0.1999999999999404754         NA      NA       NA
    ## musk                                NA         NA      NA       NA
    ## muslim           0.7272727272726777770         NA      NA       NA
    ## muslims                             NA         NA      NA       NA
    ## must                                NA         NA      NA       NA
    ## mw              -0.0000000000000135144         NA      NA       NA
    ## mystery                             NA         NA      NA       NA
    ## n               -0.0000000000000247672         NA      NA       NA
    ## naked                               NA         NA      NA       NA
    ## name             0.0900900900900878099         NA      NA       NA
    ## named                               NA         NA      NA       NA
    ## names            0.0357142857142710643         NA      NA       NA
    ## nancy                               NA         NA      NA       NA
    ## narrative                           NA         NA      NA       NA
    ## nasa             0.0909090909090293636         NA      NA       NA
    ## nation                              NA         NA      NA       NA
    ## national                            NA         NA      NA       NA
    ## nations          0.2307692307691694700         NA      NA       NA
    ## native                              NA         NA      NA       NA
    ## natural                             NA         NA      NA       NA
    ## nature                              NA         NA      NA       NA
    ## nazi             0.1052631578947043067         NA      NA       NA
    ## nazis           -0.0000000000000323216         NA      NA       NA
    ## nc                                  NA         NA      NA       NA
    ## near             0.1076923076922998018         NA      NA       NA
    ## nearly           0.2325581395348733371         NA      NA       NA
    ## necessary        0.0588235294117200255         NA      NA       NA
    ## need             0.0304878048780459592         NA      NA       NA
    ## needed           0.0416666666666561450         NA      NA       NA
    ## needs                               NA         NA      NA       NA
    ## negative         0.0588235294117362278         NA      NA       NA
    ## neighbor        -0.0000000000000372848         NA      NA       NA
    ## neighbors                           NA         NA      NA       NA
    ## nerf                                NA         NA      NA       NA
    ## net                                 NA         NA      NA       NA
    ## netflix          0.0641025641025606968         NA      NA       NA
    ## network                             NA         NA      NA       NA
    ## networks                            NA         NA      NA       NA
    ## neural                              NA         NA      NA       NA
    ## neutrality       0.5999999999999959810         NA      NA       NA
    ## nevada           0.0454545454545256036         NA      NA       NA
    ## never            0.0442477876106170884         NA      NA       NA
    ## new              0.0102986611740471731         NA      NA       NA
    ## newest                              NA         NA      NA       NA
    ## newly                               NA         NA      NA       NA
    ## news                                NA         NA      NA       NA
    ## newspaper        0.1999999999999471645         NA      NA       NA
    ## `next`                              NA         NA      NA       NA
    ## nh                                  NA         NA      NA       NA
    ## nice                                NA         NA      NA       NA
    ## night            0.0649350649350635412         NA      NA       NA
    ## nightmare       -0.0000000000000460021         NA      NA       NA
    ## nine             0.0909090909090501664         NA      NA       NA
    ## nj               0.0588235294117291918         NA      NA       NA
    ## nobel            0.0588235294117463656         NA      NA       NA
    ## nobody                              NA         NA      NA       NA
    ## noise           -0.0000000000000259842         NA      NA       NA
    ## nominated        0.0624999999999825001         NA      NA       NA
    ## nomination      -0.0000000000000294810         NA      NA       NA
    ## nominee                             NA         NA      NA       NA
    ## non                                 NA         NA      NA       NA
    ## none             0.7692307692307354117         NA      NA       NA
    ## normal                              NA         NA      NA       NA
    ## north            0.1081081081080901146         NA      NA       NA
    ## northern         0.0454545454545343744         NA      NA       NA
    ## nose                                NA         NA      NA       NA
    ## note             0.3333333333333098336         NA      NA       NA
    ## notes            0.5384615384614951372         NA      NA       NA
    ## nothing          0.0819672131147493738         NA      NA       NA
    ## notice                              NA         NA      NA       NA
    ## noticed         -0.0000000000000511691         NA      NA       NA
    ## novel           -0.0000000000000143568         NA      NA       NA
    ## november         0.0384615384615209152         NA      NA       NA
    ## now                                 NA         NA      NA       NA
    ## npc                                 NA         NA      NA       NA
    ## nuclear         -0.0000000000000157995         NA      NA       NA
    ## number                              NA         NA      NA       NA
    ## numbers                             NA         NA      NA       NA
    ## nurse           -0.0000000000000220902         NA      NA       NA
    ## nvidia          -0.0000000000000494567         NA      NA       NA
    ## ny                                  NA         NA      NA       NA
    ## nyc              0.0357142857142715084         NA      NA       NA
    ## o                                   NA         NA      NA       NA
    ## obama                               NA         NA      NA       NA
    ## obsolete        -0.0000000000000199887         NA      NA       NA
    ## obviously       -0.0000000000000459141         NA      NA       NA
    ## oc               0.0370370370370259952         NA      NA       NA
    ## october         -0.0000000000000274528         NA      NA       NA
    ## odds             0.1666666666666391794         NA      NA       NA
    ## offensive                           NA         NA      NA       NA
    ## offer                               NA         NA      NA       NA
    ## offered          0.2142857142856971209         NA      NA       NA
    ## offering                            NA         NA      NA       NA
    ## offers                              NA         NA      NA       NA
    ## office           0.0709219858155876043         NA      NA       NA
    ## officer          0.0833333333333144827         NA      NA       NA
    ## official                            NA         NA      NA       NA
    ## officially                          NA         NA      NA       NA
    ## officials                           NA         NA      NA       NA
    ## often            0.2941176470587962277         NA      NA       NA
    ## oh                                  NA         NA      NA       NA
    ## ohio                                NA         NA      NA       NA
    ## oil              0.4999999999999634182         NA      NA       NA
    ## ok                                  NA         NA      NA       NA
    ## okay                                NA         NA      NA       NA
    ## old              0.0289017341040447538         NA      NA       NA
    ## older                               NA         NA      NA       NA
    ## one              0.0147928994082831642         NA      NA       NA
    ## ones             0.0270270270270113744         NA      NA       NA
    ## ongoing         -0.0000000000000296350         NA      NA       NA
    ## online                              NA         NA      NA       NA
    ## ontario                             NA         NA      NA       NA
    ## op                                  NA         NA      NA       NA
    ## open             0.0470588235294064697         NA      NA       NA
    ## opening          0.2777777777777669654         NA      NA       NA
    ## openly                              NA         NA      NA       NA
    ## opens            0.9090909090908644297         NA      NA       NA
    ## operation        0.1538461538461087241         NA      NA       NA
    ## operations                          NA         NA      NA       NA
    ## operator         0.0666666666666361207         NA      NA       NA
    ## operators       -0.0000000000000527261         NA      NA       NA
    ## opinion                             NA         NA      NA       NA
    ## opinions         0.0909090909090747995         NA      NA       NA
    ## opportunity      0.0000000000000033997         NA      NA       NA
    ## opposition       0.4499999999999606537         NA      NA       NA
    ## ops              0.0153846153846115778         NA      NA       NA
    ## option           0.3571428571428210130         NA      NA       NA
    ## options         -0.0000000000000185567         NA      NA       NA
    ## order            0.1399999999999833877         NA      NA       NA
    ## original         0.1333333333333427129         NA      NA       NA
    ## originally       0.0999999999999562350         NA      NA       NA
    ## oscar            0.1612903225806308627         NA      NA       NA
    ## oscars           0.4166666666666447028         NA      NA       NA
    ## ost             -0.0000000000000545426         NA      NA       NA
    ## others                              NA         NA      NA       NA
    ## otherwise                           NA         NA      NA       NA
    ## outbreak                            NA         NA      NA       NA
    ## outbreaks        0.1599999999999719424         NA      NA       NA
    ## outcome                             NA         NA      NA       NA
    ## outrage          0.2999999999999580225         NA      NA       NA
    ## outside          0.1639344262294898935         NA      NA       NA
    ## overall         -0.0000000000000459812         NA      NA       NA
    ## overrated        0.4545454545454047368         NA      NA       NA
    ## overview                            NA         NA      NA       NA
    ## owned                               NA         NA      NA       NA
    ## owner                               NA         NA      NA       NA
    ## pa               0.1249999999999781425         NA      NA       NA
    ## page             0.2564102564102299642         NA      NA       NA
    ## paid                                NA         NA      NA       NA
    ## pain                                NA         NA      NA       NA
    ## paint                               NA         NA      NA       NA
    ## painted                             NA         NA      NA       NA
    ## painting        -0.0000000000000365406         NA      NA       NA
    ## palace           0.0999999999999542921         NA      NA       NA
    ## pandemic         0.0681818181818058949         NA      NA       NA
    ## panel                               NA         NA      NA       NA
    ## panic                               NA         NA      NA       NA
    ## panther                             NA         NA      NA       NA
    ## paper                               NA         NA      NA       NA
    ## papers           0.1999999999999652611         NA      NA       NA
    ## parasite                            NA         NA      NA       NA
    ## parent          -0.0000000000000450368         NA      NA       NA
    ## parents          0.1639344262295047427         NA      NA       NA
    ## paris                               NA         NA      NA       NA
    ## park             0.0999999999999904021         NA      NA       NA
    ## parking                             NA         NA      NA       NA
    ## parliament       0.1818181818181728582         NA      NA       NA
    ## parliamentary   -0.0000000000000441556         NA      NA       NA
    ## parma                               NA         NA      NA       NA
    ## part                                NA         NA      NA       NA
    ## particular      -0.0000000000000297849         NA      NA       NA
    ## parties                             NA         NA      NA       NA
    ## partner         -0.0000000000000256375         NA      NA       NA
    ## parts           -0.0000000000000226404         NA      NA       NA
    ## party            0.0628930817610068998         NA      NA       NA
    ## pas                                 NA         NA      NA       NA
    ## pass             0.2727272727272449515         NA      NA       NA
    ## passed                              NA         NA      NA       NA
    ## past                                NA         NA      NA       NA
    ## patch            0.2857142857142648262         NA      NA       NA
    ## patent                              NA         NA      NA       NA
    ## path            -0.0000000000000549119         NA      NA       NA
    ## patient                             NA         NA      NA       NA
    ## patients                            NA         NA      NA       NA
    ## patterns         0.1999999999999511890         NA      NA       NA
    ## paul                                NA         NA      NA       NA
    ## pay                                 NA         NA      NA       NA
    ## paying           0.0588235294117541857         NA      NA       NA
    ## payment                             NA         NA      NA       NA
    ## pc              -0.0000000000000734340         NA      NA       NA
    ## peace                               NA         NA      NA       NA
    ## pedro           -0.0000000000000514151         NA      NA       NA
    ## pego                                NA         NA      NA       NA
    ## pennsylvania                        NA         NA      NA       NA
    ## pentagon        -0.0000000000000085260         NA      NA       NA
    ## people                              NA         NA      NA       NA
    ## `people's`       0.2999999999999276024         NA      NA       NA
    ## per              0.1388888888888737405         NA      NA       NA
    ## percent          0.5625000000000076605         NA      NA       NA
    ## perfect                             NA         NA      NA       NA
    ## perfectly                           NA         NA      NA       NA
    ## perform         -0.0000000000000415835         NA      NA       NA
    ## performance      0.1199999999999740441         NA      NA       NA
    ## period           0.0909090909090507909         NA      NA       NA
    ## perks                               NA         NA      NA       NA
    ## person                              NA         NA      NA       NA
    ## personal                            NA         NA      NA       NA
    ## personality                         NA         NA      NA       NA
    ## personally                          NA         NA      NA       NA
    ## perspective                         NA         NA      NA       NA
    ## pet              0.1764705882352690103         NA      NA       NA
    ## pete                                NA         NA      NA       NA
    ## peter                               NA         NA      NA       NA
    ## pets            -0.0000000000000210654         NA      NA       NA
    ## phone                               NA         NA      NA       NA
    ## photo            0.1126760563380288099         NA      NA       NA
    ## photos                              NA         NA      NA       NA
    ## physical         0.0555555555556067199         NA      NA       NA
    ## physics                             NA         NA      NA       NA
    ## pic              0.1578947368420978437         NA      NA       NA
    ## pick             0.2380952380952290337         NA      NA       NA
    ## picked                              NA         NA      NA       NA
    ## pics             0.1499999999999843958         NA      NA       NA
    ## picture          0.1123595505617957790         NA      NA       NA
    ## pictures         0.1521739130434672516         NA      NA       NA
    ## piece            0.1315789473684089717         NA      NA       NA
    ## pink            -0.0000000000000511031         NA      NA       NA
    ## piracy                              NA         NA      NA       NA
    ## pirate           0.3999999999999839795         NA      NA       NA
    ## pirated                             NA         NA      NA       NA
    ## pirates          0.4117647058823244444         NA      NA       NA
    ## pixar           -0.0000000000000728383         NA      NA       NA
    ## place                               NA         NA      NA       NA
    ## placed           0.2142857142856830210         NA      NA       NA
    ## places                              NA         NA      NA       NA
    ## plague                              NA         NA      NA       NA
    ## plan                                NA         NA      NA       NA
    ## plane            0.2173913043478061802         NA      NA       NA
    ## planet                              NA         NA      NA       NA
    ## planned          0.1874999999999727163         NA      NA       NA
    ## planning        -0.0000000000000233403         NA      NA       NA
    ## plans            0.0540540540540320538         NA      NA       NA
    ## plant            0.3571428571428332255         NA      NA       NA
    ## plastic                             NA         NA      NA       NA
    ## platform        -0.0000000000000237903         NA      NA       NA
    ## play             0.0684931506849305316         NA      NA       NA
    ## played           0.0980392156862674030         NA      NA       NA
    ## player                              NA         NA      NA       NA
    ## players                             NA         NA      NA       NA
    ## playing                             NA         NA      NA       NA
    ## playlist                            NA         NA      NA       NA
    ## plays            0.0624999999999689484         NA      NA       NA
    ## please           0.0518134715025896711         NA      NA       NA
    ## plot             0.2083333333333277915         NA      NA       NA
    ## plus                                NA         NA      NA       NA
    ## pm               0.3913043478260688768         NA      NA       NA
    ## pneumonia       -0.0000000000000383125         NA      NA       NA
    ## podcast         -0.0000000000000347767         NA      NA       NA
    ## point            0.0886075949367313570         NA      NA       NA
    ## points           0.0769230769230591499         NA      NA       NA
    ## poland           0.0799999999999833622         NA      NA       NA
    ## police                              NA         NA      NA       NA
    ## policies                            NA         NA      NA       NA
    ## policy           0.0526315789473604245         NA      NA       NA
    ## polish           0.0689655172413447121         NA      NA       NA
    ## political                           NA         NA      NA       NA
    ## politician       0.0370370370370203816         NA      NA       NA
    ## politicians      0.0799999999999913558         NA      NA       NA
    ## politics                            NA         NA      NA       NA
    ## poll                                NA         NA      NA       NA
    ## polling          0.3333333333333249882         NA      NA       NA
    ## polls            0.0270270270270158604         NA      NA       NA
    ## poor                                NA         NA      NA       NA
    ## poorly          -0.0000000000000344989         NA      NA       NA
    ## pop              0.0131578947368365499         NA      NA       NA
    ## popcorn                             NA         NA      NA       NA
    ## popular          0.1587301587301486738         NA      NA       NA
    ## population                          NA         NA      NA       NA
    ## porn             0.3103448275861906414         NA      NA       NA
    ## portrait                            NA         NA      NA       NA
    ## portrayed       -0.0000000000000516132         NA      NA       NA
    ## position                            NA         NA      NA       NA
    ## positions       -0.0000000000000453240         NA      NA       NA
    ## positive                            NA         NA      NA       NA
    ## possible         0.0522388059701452412         NA      NA       NA
    ## possibly                            NA         NA      NA       NA
    ## post             0.0395256916996026900         NA      NA       NA
    ## posted           0.1599999999999903721         NA      NA       NA
    ## poster                              NA         NA      NA       NA
    ## posters          0.1153846153845835831         NA      NA       NA
    ## posting                             NA         NA      NA       NA
    ## posts                               NA         NA      NA       NA
    ## potential        0.0526315789473606119         NA      NA       NA
    ## potentially      0.3333333333333085013         NA      NA       NA
    ## potter           0.0909090909090518318         NA      NA       NA
    ## potus           -0.0000000000000408485         NA      NA       NA
    ## power                               NA         NA      NA       NA
    ## powered                             NA         NA      NA       NA
    ## powerful         0.1470588235293942003         NA      NA       NA
    ## pre                                 NA         NA      NA       NA
    ## predict                             NA         NA      NA       NA
    ## predicted                           NA         NA      NA       NA
    ## predicting      -0.0000000000000238395         NA      NA       NA
    ## prediction       0.1111111111110837657         NA      NA       NA
    ## predictions                         NA         NA      NA       NA
    ## predicts         0.0416666666666503996         NA      NA       NA
    ## pregnant        -0.0000000000000436035         NA      NA       NA
    ## premier          0.1379310344827451018         NA      NA       NA
    ## premiere        -0.0000000000000500627         NA      NA       NA
    ## prepare                             NA         NA      NA       NA
    ## prepared        -0.0000000000000377552         NA      NA       NA
    ## preparing        0.1666666666665741481         NA      NA       NA
    ## present         -0.0000000000000237218         NA      NA       NA
    ## presidency       0.0344827586206749234         NA      NA       NA
    ## president        0.0230414746543789392         NA      NA       NA
    ## presidential     0.0390624999999983416         NA      NA       NA
    ## presidents       0.2222222222221870158         NA      NA       NA
    ## press            0.1818181818181723863         NA      NA       NA
    ## pressure         0.0833333333332881149         NA      NA       NA
    ## pretty                              NA         NA      NA       NA
    ## prevent          0.1176470588235190989         NA      NA       NA
    ## preview                             NA         NA      NA       NA
    ## previous        -0.0000000000000152879         NA      NA       NA
    ## price                               NA         NA      NA       NA
    ## prices           0.1538461538461395328         NA      NA       NA
    ## pride            0.0909090909090526228         NA      NA       NA
    ## primaries                           NA         NA      NA       NA
    ## primary          0.1190476190476132540         NA      NA       NA
    ## prime                               NA         NA      NA       NA
    ## prince          -0.0000000000000293133         NA      NA       NA
    ## princess        -0.0000000000000481230         NA      NA       NA
    ## principles                          NA         NA      NA       NA
    ## prior           -0.0000000000000484945         NA      NA       NA
    ## prison                              NA         NA      NA       NA
    ## privacy          0.4166666666666501428         NA      NA       NA
    ## private                             NA         NA      NA       NA
    ## prize                               NA         NA      NA       NA
    ## pro              0.0576923076922982517         NA      NA       NA
    ## probably         0.0606060606060552301         NA      NA       NA
    ## problem                             NA         NA      NA       NA
    ## problems                            NA         NA      NA       NA
    ## process          0.0499999999999895736         NA      NA       NA
    ## produce          0.0588235294117367968         NA      NA       NA
    ## produced                            NA         NA      NA       NA
    ## producer         0.0769230769230384998         NA      NA       NA
    ## product         -0.0000000000000285306         NA      NA       NA
    ## production       0.2285714285714132155         NA      NA       NA
    ## products                            NA         NA      NA       NA
    ## professional     0.0555555555555313635         NA      NA       NA
    ## professor        0.0909090909090509852         NA      NA       NA
    ## profile          0.1666666666666440366         NA      NA       NA
    ## profit                              NA         NA      NA       NA
    ## program          0.0909090909090712052         NA      NA       NA
    ## programmed                          NA         NA      NA       NA
    ## programming     -0.0000000000000302048         NA      NA       NA
    ## progress        -0.0000000000000226784         NA      NA       NA
    ## progressive                         NA         NA      NA       NA
    ## project          0.0196078431372446529         NA      NA       NA
    ## projects         0.0833333333333203669         NA      NA       NA
    ## promised        -0.0000000000000363057         NA      NA       NA
    ## promises         0.2272727272727264602         NA      NA       NA
    ## promo            0.0833333333332972048         NA      NA       NA
    ## proof                               NA         NA      NA       NA
    ## prop                                NA         NA      NA       NA
    ## propaganda       0.4761904761904334760         NA      NA       NA
    ## proper          -0.0000000000000422306         NA      NA       NA
    ## properly         0.0624999999999781911         NA      NA       NA
    ## property        -0.0000000000000161652         NA      NA       NA
    ## proposes        -0.0000000000000446750         NA      NA       NA
    ## protect                             NA         NA      NA       NA
    ## protection      -0.0000000000000249746         NA      NA       NA
    ## protest                             NA         NA      NA       NA
    ## protesters                          NA         NA      NA       NA
    ## protesting       0.3684210526315168099         NA      NA       NA
    ## protests         0.2857142857142560000         NA      NA       NA
    ## proud                               NA         NA      NA       NA
    ## prove            0.1199999999999842581         NA      NA       NA
    ## provide          0.0909090909090696370         NA      NA       NA
    ## province         0.0588235294117254379         NA      NA       NA
    ## ps4                                 NA         NA      NA       NA
    ## psa                                 NA         NA      NA       NA
    ## psg                                 NA         NA      NA       NA
    ## public                              NA         NA      NA       NA
    ## publicly         0.1111111111110820032         NA      NA       NA
    ## published        0.0499999999999759040         NA      NA       NA
    ## pull                                NA         NA      NA       NA
    ## pulls            0.0909090909090479321         NA      NA       NA
    ## punch                               NA         NA      NA       NA
    ## punk                                NA         NA      NA       NA
    ## pup             -0.0000000000000431287         NA      NA       NA
    ## puppy                               NA         NA      NA       NA
    ## purchase         0.0526315789473437018         NA      NA       NA
    ## purchased        0.0833333333333119569         NA      NA       NA
    ## purge            0.0714285714285596979         NA      NA       NA
    ## purpose                             NA         NA      NA       NA
    ## pursue                              NA         NA      NA       NA
    ## push                                NA         NA      NA       NA
    ## put              0.0793650793650754610         NA      NA       NA
    ## putin            0.6249999999999671374         NA      NA       NA
    ## puts             0.1111111111110710120         NA      NA       NA
    ## putting                             NA         NA      NA       NA
    ## pve             -0.0000000000000438759         NA      NA       NA
    ## pvp             -0.0000000000000220766         NA      NA       NA
    ## qualified        0.0769230769230417055         NA      NA       NA
    ## quality          0.0943396226414972561         NA      NA       NA
    ## quantum         -0.0000000000000161054         NA      NA       NA
    ## quarantine       0.0303030303030232886         NA      NA       NA
    ## quarantined      0.4347826086956295133         NA      NA       NA
    ## queen            0.3571428571428234555         NA      NA       NA
    ## question                            NA         NA      NA       NA
    ## questions        0.0684931506849244948         NA      NA       NA
    ## quick                               NA         NA      NA       NA
    ## quickly                             NA         NA      NA       NA
    ## quietly                             NA         NA      NA       NA
    ## quit            -0.0000000000000463888         NA      NA       NA
    ## quite                               NA         NA      NA       NA
    ## quote            0.4545454545454348794         NA      NA       NA
    ## quotes           0.0666666666666297369         NA      NA       NA
    ## r                0.0161030595813199603         NA      NA       NA
    ## rabbit           0.0999999999999807709         NA      NA       NA
    ## race             0.1298701298701312457         NA      NA       NA
    ## racism           0.3076923076922865041         NA      NA       NA
    ## racist           0.2040816326530538460         NA      NA       NA
    ## radio            0.0666666666666368979         NA      NA       NA
    ## rage             0.0624999999999547931         NA      NA       NA
    ## raid                                NA         NA      NA       NA
    ## rain                                NA         NA      NA       NA
    ## raise            0.0285714285714136450         NA      NA       NA
    ## raised          -0.0000000000000367157         NA      NA       NA
    ## raises          -0.0000000000000446227         NA      NA       NA
    ## raising          0.0909090909090660565         NA      NA       NA
    ## rally            0.4285714285714061211         NA      NA       NA
    ## ran                                 NA         NA      NA       NA
    ## random                              NA         NA      NA       NA
    ## ranking          0.3999999999999504507         NA      NA       NA
    ## rap             -0.0000000000000195924         NA      NA       NA
    ## rare             0.0999999999999756778         NA      NA       NA
    ## rate                                NA         NA      NA       NA
    ## rated                               NA         NA      NA       NA
    ## rates                               NA         NA      NA       NA
    ## rather                              NA         NA      NA       NA
    ## rating                              NA         NA      NA       NA
    ## ray                                 NA         NA      NA       NA
    ## re               0.0418410041841002014         NA      NA       NA
    ## reach                               NA         NA      NA       NA
    ## reached          0.0999999999999706818         NA      NA       NA
    ## react           -0.0000000000000341089         NA      NA       NA
    ## reaction                            NA         NA      NA       NA
    ## read                                NA         NA      NA       NA
    ## reading          0.0344827586206736397         NA      NA       NA
    ## ready                               NA         NA      NA       NA
    ## real                                NA         NA      NA       NA
    ## realism         -0.0000000000000310503         NA      NA       NA
    ## realistic        0.0833333333333209497         NA      NA       NA
    ## reality          0.0877192982456086179         NA      NA       NA
    ## realize                             NA         NA      NA       NA
    ## realized         0.0555555555555367481         NA      NA       NA
    ## really                              NA         NA      NA       NA
    ## reason           0.1269841269841210096         NA      NA       NA
    ## reasonable                          NA         NA      NA       NA
    ## reasons                             NA         NA      NA       NA
    ## receive          0.0499999999999767575         NA      NA       NA
    ## received         0.1176470588235147968         NA      NA       NA
    ## recent                              NA         NA      NA       NA
    ## recently         0.0740740740740624265         NA      NA       NA
    ## recognition      0.0869565217391195522         NA      NA       NA
    ## recognize        0.0624999999999784200         NA      NA       NA
    ## recommend        0.0370370370370177865         NA      NA       NA
    ## recommendations                     NA         NA      NA       NA
    ## record           0.3225806451612702741         NA      NA       NA
    ## recorded         0.0624999999999700587         NA      NA       NA
    ## recording        0.0833333333332923892         NA      NA       NA
    ## records          0.4285714285714074534         NA      NA       NA
    ## recover                             NA         NA      NA       NA
    ## recovering                          NA         NA      NA       NA
    ## red              0.0487804878048671703         NA      NA       NA
    ## reddit                              NA         NA      NA       NA
    ## redditor         0.1666666666666303531         NA      NA       NA
    ## redditors                           NA         NA      NA       NA
    ## reduce                              NA         NA      NA       NA
    ## reeves           0.2857142857142489500         NA      NA       NA
    ## reference        0.0526315789473576143         NA      NA       NA
    ## references      -0.0000000000000298799         NA      NA       NA
    ## referendum       0.5714285714285413098         NA      NA       NA
    ## refund          -0.0000000000000339200         NA      NA       NA
    ## refuse                              NA         NA      NA       NA
    ## refused                             NA         NA      NA       NA
    ## refuses          0.1071428571428382076         NA      NA       NA
    ## refusing                            NA         NA      NA       NA
    ## regarding                           NA         NA      NA       NA
    ## regards         -0.0000000000000327807         NA      NA       NA
    ## region                              NA         NA      NA       NA
    ## regional                            NA         NA      NA       NA
    ## register        -0.0000000000000227763         NA      NA       NA
    ## registered                          NA         NA      NA       NA
    ## registration                        NA         NA      NA       NA
    ## regret           0.0909090909091253147         NA      NA       NA
    ## regular          0.0869565217391128770         NA      NA       NA
    ## related                             NA         NA      NA       NA
    ## relationship                        NA         NA      NA       NA
    ## relationships                       NA         NA      NA       NA
    ## release                             NA         NA      NA       NA
    ## released         0.1250000000000000278         NA      NA       NA
    ## releases                            NA         NA      NA       NA
    ## relevant                            NA         NA      NA       NA
    ## religion        -0.0000000000000401079         NA      NA       NA
    ## remade           1.0000000000000426326         NA      NA       NA
    ## remain          -0.0000000000000365171         NA      NA       NA
    ## remains          0.0833333333332709758         NA      NA       NA
    ## remake                              NA         NA      NA       NA
    ## remakes          0.0909090909090526367         NA      NA       NA
    ## remedies        -0.0000000000000399969         NA      NA       NA
    ## remember         0.0468749999999935885         NA      NA       NA
    ## reminded                            NA         NA      NA       NA
    ## reminder         0.1111111111110829053         NA      NA       NA
    ## reminds                             NA         NA      NA       NA
    ## remove                              NA         NA      NA       NA
    ## removed                             NA         NA      NA       NA
    ## renewable        0.4545454545454146733         NA      NA       NA
    ## rent             0.0399999999999823067         NA      NA       NA
    ## rental                              NA         NA      NA       NA
    ## rep             -0.0000000000000601971         NA      NA       NA
    ## replace                             NA         NA      NA       NA
    ## replaced                            NA         NA      NA       NA
    ## replacing       -0.0000000000000236285         NA      NA       NA
    ## report           0.1388888888888857309         NA      NA       NA
    ## reported                            NA         NA      NA       NA
    ## reportedly       0.2142857142856683106         NA      NA       NA
    ## reporting        0.1538461538461224631         NA      NA       NA
    ## reports          0.1590909090908937118         NA      NA       NA
    ## representation                      NA         NA      NA       NA
    ## representative   0.1176470588235050546         NA      NA       NA
    ## representatives  0.3076923076922705724         NA      NA       NA
    ## republican       0.1521739130434590637         NA      NA       NA
    ## republicans                         NA         NA      NA       NA
    ## request         -0.0000000000000163446         NA      NA       NA
    ## require                             NA         NA      NA       NA
    ## required                            NA         NA      NA       NA
    ## requiring       -0.0000000000000402025         NA      NA       NA
    ## research         0.1063829787233993573         NA      NA       NA
    ## researcher       0.0909090909090484178         NA      NA       NA
    ## researchers      0.0987654320987648854         NA      NA       NA
    ## resident                            NA         NA      NA       NA
    ## residents        0.1111111111110903854         NA      NA       NA
    ## resigns          0.0999999999999515027         NA      NA       NA
    ## resolution       0.2666666666666501762         NA      NA       NA
    ## resources       -0.0000000000000046139         NA      NA       NA
    ## respect                             NA         NA      NA       NA
    ## respond         -0.0000000000000278114         NA      NA       NA
    ## responds         0.0833333333332959558         NA      NA       NA
    ## response         0.1228070175438487621         NA      NA       NA
    ## responsible     -0.0000000000000397244         NA      NA       NA
    ## rest             0.1463414634146550652         NA      NA       NA
    ## restaurant       0.4666666666666339225         NA      NA       NA
    ## restrictions     0.2727272727272297415         NA      NA       NA
    ## result                              NA         NA      NA       NA
    ## resulting       -0.0000000000000461324         NA      NA       NA
    ## results          0.0806451612903188314         NA      NA       NA
    ## retail          -0.0000000000000518436         NA      NA       NA
    ## retirement      -0.0000000000000271370         NA      NA       NA
    ## return           0.0606060606060465010         NA      NA       NA
    ## returning       -0.0000000000000092300         NA      NA       NA
    ## returns          0.2499999999999694411         NA      NA       NA
    ## reveal           0.2307692307691878442         NA      NA       NA
    ## revealed                            NA         NA      NA       NA
    ## reveals                             NA         NA      NA       NA
    ## reverse                             NA         NA      NA       NA
    ## review                              NA         NA      NA       NA
    ## reviews                             NA         NA      NA       NA
    ## revolution       0.1666666666666646035         NA      NA       NA
    ## rich                                NA         NA      NA       NA
    ## rick                                NA         NA      NA       NA
    ## rid             -0.0000000000000262179         NA      NA       NA
    ## ride             0.0909090909090671528         NA      NA       NA
    ## ridiculous       1.0000000000000173195         NA      NA       NA
    ## rigged           0.0769230769230316996         NA      NA       NA
    ## rigging          0.4166666666666202223         NA      NA       NA
    ## right            0.0362318840579693505         NA      NA       NA
    ## rights           0.1515151515151496320         NA      NA       NA
    ## ring             0.4615384615384383604         NA      NA       NA
    ## riot                                NA         NA      NA       NA
    ## rip              0.0769230769230409700         NA      NA       NA
    ## rise                                NA         NA      NA       NA
    ## rises                               NA         NA      NA       NA
    ## rising           0.0833333333332758192         NA      NA       NA
    ## risk                                NA         NA      NA       NA
    ## rivalries       -0.0000000000000348420         NA      NA       NA
    ## road                                NA         NA      NA       NA
    ## roads                               NA         NA      NA       NA
    ## roast            0.1123595505617954321         NA      NA       NA
    ## roasted          0.4285714285713870808         NA      NA       NA
    ## roasting        -0.0000000000000189470         NA      NA       NA
    ## roasts                              NA         NA      NA       NA
    ## rob                                 NA         NA      NA       NA
    ## robert                              NA         NA      NA       NA
    ## robot                               NA         NA      NA       NA
    ## robotics         0.1818181818181651421         NA      NA       NA
    ## robots           0.0631578947368372917         NA      NA       NA
    ## rock             0.0714285714285713969         NA      NA       NA
    ## rocky           -0.0000000000000411748         NA      NA       NA
    ## rogan                               NA         NA      NA       NA
    ## rogue            0.0588235294117371160         NA      NA       NA
    ## role                                NA         NA      NA       NA
    ## roles            0.0666666666666397290         NA      NA       NA
    ## roll             0.2777777777778014934         NA      NA       NA
    ## rolls            0.0909090909090247978         NA      NA       NA
    ## romance         -0.0000000000000469537         NA      NA       NA
    ## romanian         0.0624999999999787809         NA      NA       NA
    ## romantic         0.0952380952380781909         NA      NA       NA
    ## ronda            0.0999999999999569150         NA      NA       NA
    ## room                                NA         NA      NA       NA
    ## roth            -0.0000000000000143150         NA      NA       NA
    ## rotten                              NA         NA      NA       NA
    ## round                               NA         NA      NA       NA
    ## roundup                             NA         NA      NA       NA
    ## row                                 NA         NA      NA       NA
    ## ruin            -0.0000000000000246863         NA      NA       NA
    ## ruined           0.1578947368420723640         NA      NA       NA
    ## rule             0.0434782608695424497         NA      NA       NA
    ## rules            0.1785714285713961291         NA      NA       NA
    ## run              0.0900900900900881985         NA      NA       NA
    ## runner          -0.0000000000000425296         NA      NA       NA
    ## running                             NA         NA      NA       NA
    ## runs                                NA         NA      NA       NA
    ## russia                              NA         NA      NA       NA
    ## russian          0.1063829787233987467         NA      NA       NA
    ## russians         0.4545454545453761486         NA      NA       NA
    ## s                                   NA         NA      NA       NA
    ## sad             -0.0000000000000228657         NA      NA       NA
    ## safe             0.0303030303030150383         NA      NA       NA
    ## safety                              NA         NA      NA       NA
    ## said             0.0735294117647024431         NA      NA       NA
    ## salary           0.0689655172413567996         NA      NA       NA
    ## sale                                NA         NA      NA       NA
    ## sales                               NA         NA      NA       NA
    ## sample                              NA         NA      NA       NA
    ## samsung                             NA         NA      NA       NA
    ## san              0.0624999999999708775         NA      NA       NA
    ## sanders          0.0584795321637411325         NA      NA       NA
    ## santa                               NA         NA      NA       NA
    ## sars            -0.0000000000000160499         NA      NA       NA
    ## sat              0.1538461538461204370         NA      NA       NA
    ## saturday                            NA         NA      NA       NA
    ## save                                NA         NA      NA       NA
    ## saved            0.0434782608695578540         NA      NA       NA
    ## saving           0.0399999999999827438         NA      NA       NA
    ## savings         -0.0000000000000002734         NA      NA       NA
    ## saw              0.1075268817204276528         NA      NA       NA
    ## say              0.0492610837438391125         NA      NA       NA
    ## saying           0.0909090909090831401         NA      NA       NA
    ## says             0.0341296928327600446         NA      NA       NA
    ## sc               0.0312499999999926968         NA      NA       NA
    ## scale            0.1111111111110851674         NA      NA       NA
    ## scam                                NA         NA      NA       NA
    ## scandal          0.1538461538461171063         NA      NA       NA
    ## scare           -0.0000000000000343275         NA      NA       NA
    ## scared                              NA         NA      NA       NA
    ## scary                               NA         NA      NA       NA
    ## scenario        -0.0000000000000290208         NA      NA       NA
    ## scene            0.0724637681159378405         NA      NA       NA
    ## scenes                              NA         NA      NA       NA
    ## school                              NA         NA      NA       NA
    ## schools                             NA         NA      NA       NA
    ## schwarzenegger   0.2727272727272321284         NA      NA       NA
    ## sci              0.0263157894736767428         NA      NA       NA
    ## science          0.1492537313432668877         NA      NA       NA
    ## scientific       0.2307692307692270628         NA      NA       NA
    ## scientist                           NA         NA      NA       NA
    ## scientists       0.0970873786407723122         NA      NA       NA
    ## score                               NA         NA      NA       NA
    ## scores                              NA         NA      NA       NA
    ## scotland        -0.0000000000000513212         NA      NA       NA
    ## scott            0.0384615384615380057         NA      NA       NA
    ## scream          -0.0000000000000426565         NA      NA       NA
    ## screen           0.0277777777777893538         NA      NA       NA
    ## screening        0.0416666666666472563         NA      NA       NA
    ## screw           -0.0000000000000452663         NA      NA       NA
    ## screwed         -0.0000000000000373417         NA      NA       NA
    ## script                              NA         NA      NA       NA
    ## se              -0.0000000000000292824         NA      NA       NA
    ## sea              0.0499999999999803310         NA      NA       NA
    ## search           0.1372549019607788856         NA      NA       NA
    ## season           0.0526315789473658022         NA      NA       NA
    ## seat                                NA         NA      NA       NA
    ## seats                               NA         NA      NA       NA
    ## seattle          0.1578947368420745845         NA      NA       NA
    ## second           0.0869565217391247702         NA      NA       NA
    ## seconds          0.0833333333333198256         NA      NA       NA
    ## secret                              NA         NA      NA       NA
    ## secretary                           NA         NA      NA       NA
    ## secretly        -0.0000000000000409931         NA      NA       NA
    ## section         -0.0000000000000289606         NA      NA       NA
    ## secure                              NA         NA      NA       NA
    ## security         0.1038961038961132705         NA      NA       NA
    ## see                                 NA         NA      NA       NA
    ## seeing                              NA         NA      NA       NA
    ## seek                                NA         NA      NA       NA
    ## seeking         -0.0000000000000702535         NA      NA       NA
    ## seem             0.0444444444444334411         NA      NA       NA
    ## seems            0.0533333333333154969         NA      NA       NA
    ## seen             0.0564971751412297399         NA      NA       NA
    ## sees            -0.0000000000000317511         NA      NA       NA
    ## selection       -0.0000000000000364752         NA      NA       NA
    ## self             0.1282051282051248908         NA      NA       NA
    ## sell             0.2121212121211950852         NA      NA       NA
    ## selling          0.0833333333333211301         NA      NA       NA
    ## semi             0.8333333333332826331         NA      NA       NA
    ## senate           0.1923076923076804390         NA      NA       NA
    ## senator                             NA         NA      NA       NA
    ## send             0.0399999999999802319         NA      NA       NA
    ## sends            0.3076923076922731259         NA      NA       NA
    ## sense            0.0606060606060483953         NA      NA       NA
    ## sent             0.0227272727272623785         NA      NA       NA
    ## sentient        -0.0000000000000346612         NA      NA       NA
    ## separate        -0.0000000000000380292         NA      NA       NA
    ## sequel                              NA         NA      NA       NA
    ## sequels          0.1176470588235026676         NA      NA       NA
    ## serie            0.0833333333333134418         NA      NA       NA
    ## series           0.1562499999999990286         NA      NA       NA
    ## serious          0.0540540540540503101         NA      NA       NA
    ## seriously        0.1071428571428382354         NA      NA       NA
    ## serj            -0.0000000000000320308         NA      NA       NA
    ## service          0.1698113207547080894         NA      NA       NA
    ## services                            NA         NA      NA       NA
    ## set              0.0680272108843421458         NA      NA       NA
    ## sets             0.1304347826086760809         NA      NA       NA
    ## setting         -0.0000000000000169488         NA      NA       NA
    ## several          0.0312499999999792839         NA      NA       NA
    ## sex              0.1562499999999931999         NA      NA       NA
    ## sexual                              NA         NA      NA       NA
    ## shadow                              NA         NA      NA       NA
    ## share            0.1136363636363534046         NA      NA       NA
    ## shares           0.2499999999998719080         NA      NA       NA
    ## sharing          0.0416666666666525645         NA      NA       NA
    ## shark           -0.0000000000000461761         NA      NA       NA
    ## sheriff                             NA         NA      NA       NA
    ## ship                                NA         NA      NA       NA
    ## shirt            0.1052631578947320068         NA      NA       NA
    ## shit                                NA         NA      NA       NA
    ## shoot            0.0476190476190269801         NA      NA       NA
    ## shooting                            NA         NA      NA       NA
    ## shop                                NA         NA      NA       NA
    ## short            0.1216216216216075841         NA      NA       NA
    ## shot             0.0515463917525760540         NA      NA       NA
    ## shots            0.0892857142857109987         NA      NA       NA
    ## shouldn          0.0666666666666397428         NA      NA       NA
    ## show             0.0473933649289097828         NA      NA       NA
    ## showed                              NA         NA      NA       NA
    ## showing                             NA         NA      NA       NA
    ## shown            0.2592592592592239953         NA      NA       NA
    ## shows            0.0632911392405031137         NA      NA       NA
    ## shut                                NA         NA      NA       NA
    ## shuts            0.0999999999999408723         NA      NA       NA
    ## sick             0.0487804878048744839         NA      NA       NA
    ## side                                NA         NA      NA       NA
    ## sign             0.0655737704917952885         NA      NA       NA
    ## signatures                          NA         NA      NA       NA
    ## signed                              NA         NA      NA       NA
    ## significant     -0.0000000000000390060         NA      NA       NA
    ## significantly    0.2307692307691828204         NA      NA       NA
    ## signing          0.0909090909090482929         NA      NA       NA
    ## signs            0.0624999999999853936         NA      NA       NA
    ## silent                              NA         NA      NA       NA
    ## similar          0.0263157894736723123         NA      NA       NA
    ## simple                              NA         NA      NA       NA
    ## simply           0.0624999999999850606         NA      NA       NA
    ## simulation      -0.0000000000000213285         NA      NA       NA
    ## since            0.0943396226415028766         NA      NA       NA
    ## singing         -0.0000000000000416316         NA      NA       NA
    ## single           0.1111111111111068028         NA      NA       NA
    ## singularity     -0.0000000000000465562         NA      NA       NA
    ## sinn            -0.0000000000000332451         NA      NA       NA
    ## sister           0.2352941176470327855         NA      NA       NA
    ## sit              0.1999999999999531597         NA      NA       NA
    ## site             0.1249999999999906602         NA      NA       NA
    ## sites            0.2999999999999757860         NA      NA       NA
    ## sitting          0.0434782608695445313         NA      NA       NA
    ## situation                           NA         NA      NA       NA
    ## six                                 NA         NA      NA       NA
    ## size                                NA         NA      NA       NA
    ## skill                               NA         NA      NA       NA
    ## skills                              NA         NA      NA       NA
    ## skin            -0.0000000000000407439         NA      NA       NA
    ## skip            -0.0000000000000407245         NA      NA       NA
    ## skynet           0.1999999999999555189         NA      NA       NA
    ## sleep                               NA         NA      NA       NA
    ## sleeping                            NA         NA      NA       NA
    ## slow             0.1874999999999742428         NA      NA       NA
    ## slowly                              NA         NA      NA       NA
    ## small            0.1184210526315776224         NA      NA       NA
    ## smart           -0.0000000000000114499         NA      NA       NA
    ## smarter          0.0434782608695380712         NA      NA       NA
    ## smith           -0.0000000000000130250         NA      NA       NA
    ## smoke            0.7692307692307255307         NA      NA       NA
    ## snap            -0.0000000000000589304         NA      NA       NA
    ## soccer                              NA         NA      NA       NA
    ## social           0.0804597701149342565         NA      NA       NA
    ## socialism        0.0833333333332885867         NA      NA       NA
    ## society          0.0666666666666618918         NA      NA       NA
    ## software                            NA         NA      NA       NA
    ## solar           -0.0000000000000230336         NA      NA       NA
    ## sold             0.4999999999999785172         NA      NA       NA
    ## soldiers                            NA         NA      NA       NA
    ## solid            0.0769230769230409145         NA      NA       NA
    ## solo                                NA         NA      NA       NA
    ## solution         0.4444444444444420328         NA      NA       NA
    ## solutions                           NA         NA      NA       NA
    ## solve           -0.0000000000000033622         NA      NA       NA
    ## somebody        -0.0000000000000261882         NA      NA       NA
    ## somehow                             NA         NA      NA       NA
    ## someone          0.0483091787439593398         NA      NA       NA
    ## something                           NA         NA      NA       NA
    ## sometimes        0.0249999999999880665         NA      NA       NA
    ## son              0.1162790697674318807         NA      NA       NA
    ## song             0.0751879699248083067         NA      NA       NA
    ## songs                               NA         NA      NA       NA
    ## sonic                               NA         NA      NA       NA
    ## sony             0.5624999999999231726         NA      NA       NA
    ## soon             0.0714285714285646106         NA      NA       NA
    ## sorry                               NA         NA      NA       NA
    ## sort            -0.0000000000000177312         NA      NA       NA
    ## soul             0.0499999999999844527         NA      NA       NA
    ## sound            0.0416666666666631533         NA      NA       NA
    ## sounds           0.0769230769230473677         NA      NA       NA
    ## soundtrack                          NA         NA      NA       NA
    ## soundtracks     -0.0000000000000463293         NA      NA       NA
    ## source                              NA         NA      NA       NA
    ## sources          0.2499999999999091838         NA      NA       NA
    ## south            0.0555555555555519998         NA      NA       NA
    ## southern        -0.0000000000000321548         NA      NA       NA
    ## soviet           0.1538461538461295963         NA      NA       NA
    ## space            0.0677966101694836287         NA      NA       NA
    ## spain            0.0555555555555325431         NA      NA       NA
    ## spanish                             NA         NA      NA       NA
    ## sparks          -0.0000000000000459458         NA      NA       NA
    ## speak           -0.0000000000000260661         NA      NA       NA
    ## speaking                            NA         NA      NA       NA
    ## spec                                NA         NA      NA       NA
    ## special                             NA         NA      NA       NA
    ## species          0.0000000000000189717         NA      NA       NA
    ## specific         0.0370370370370204094         NA      NA       NA
    ## specifically    -0.0000000000000420753         NA      NA       NA
    ## speech           0.0545454545454434198         NA      NA       NA
    ## speed                               NA         NA      NA       NA
    ## spend                               NA         NA      NA       NA
    ## spending                            NA         NA      NA       NA
    ## spent                               NA         NA      NA       NA
    ## spider                              NA         NA      NA       NA
    ## spielberg        0.0909090909090553151         NA      NA       NA
    ## spike                               NA         NA      NA       NA
    ## split                               NA         NA      NA       NA
    ## spoiler          0.0624999999999892447         NA      NA       NA
    ## spoilers                            NA         NA      NA       NA
    ## spoken                              NA         NA      NA       NA
    ## sport            0.0000000000000398343         NA      NA       NA
    ## sporting                            NA         NA      NA       NA
    ## sports           0.0303030303030147399         NA      NA       NA
    ## spot                                NA         NA      NA       NA
    ## spotify         -0.0000000000000399001         NA      NA       NA
    ## spotted                             NA         NA      NA       NA
    ## spread           0.0568181818181741835         NA      NA       NA
    ## spreading        0.1351351351351220975         NA      NA       NA
    ## spreads          0.0689655172413631834         NA      NA       NA
    ## spring          -0.0000000000000435476         NA      NA       NA
    ## spy             -0.0000000000000441385         NA      NA       NA
    ## squad            0.1304347826086890150         NA      NA       NA
    ## square                              NA         NA      NA       NA
    ## st              -0.0000000000000329801         NA      NA       NA
    ## staff                               NA         NA      NA       NA
    ## stage                               NA         NA      NA       NA
    ## stand            0.0588235294117560314         NA      NA       NA
    ## standard                            NA         NA      NA       NA
    ## standards        0.0999999999999502814         NA      NA       NA
    ## standing        -0.0000000000000314263         NA      NA       NA
    ## stands           0.2999999999999233835         NA      NA       NA
    ## star                                NA         NA      NA       NA
    ## starring         0.0937499999999829997         NA      NA       NA
    ## stars           -0.0000000000000247254         NA      NA       NA
    ## start                               NA         NA      NA       NA
    ## started          0.1290322580645142381         NA      NA       NA
    ## starting         0.0363636363636261895         NA      NA       NA
    ## starts           0.0666666666666539676         NA      NA       NA
    ## startup                             NA         NA      NA       NA
    ## state                               NA         NA      NA       NA
    ## statement        0.0555555555555256944         NA      NA       NA
    ## states                              NA         NA      NA       NA
    ## station          0.1025641025640916942         NA      NA       NA
    ## status           0.1764705882352670674         NA      NA       NA
    ## stay                                NA         NA      NA       NA
    ## staying          0.2857142857142881409         NA      NA       NA
    ## steal           -0.0000000000000345692         NA      NA       NA
    ## stealing         0.0833333333332874487         NA      NA       NA
    ## step                                NA         NA      NA       NA
    ## stephen                             NA         NA      NA       NA
    ## steps                               NA         NA      NA       NA
    ## steve                               NA         NA      NA       NA
    ## steven                              NA         NA      NA       NA
    ## stick                               NA         NA      NA       NA
    ## sticker         -0.0000000000000344146         NA      NA       NA
    ## still            0.0423728813559303558         NA      NA       NA
    ## stock            0.0249999999999878757         NA      NA       NA
    ## stole           -0.0000000000000419357         NA      NA       NA
    ## stolen           0.2631578947368238275         NA      NA       NA
    ## stomach         -0.0000000000000136020         NA      NA       NA
    ## stone                               NA         NA      NA       NA
    ## stop             0.0621118012422330126         NA      NA       NA
    ## stopped          0.1499999999999789835         NA      NA       NA
    ## storage          0.1499999999999777067         NA      NA       NA
    ## store            0.1136363636363499768         NA      NA       NA
    ## stories                             NA         NA      NA       NA
    ## storm                               NA         NA      NA       NA
    ## story            0.0746268656716383288         NA      NA       NA
    ## straight                            NA         NA      NA       NA
    ## strain          -0.0000000000000187474         NA      NA       NA
    ## strange                             NA         NA      NA       NA
    ## stranger         0.0563380281690086665         NA      NA       NA
    ## strategy                            NA         NA      NA       NA
    ## stream          -0.0000000000000290949         NA      NA       NA
    ## streaming        0.1694915254237363833         NA      NA       NA
    ## street           0.1999999999999765299         NA      NA       NA
    ## streets          0.3999999999999402367         NA      NA       NA
    ## strike                              NA         NA      NA       NA
    ## strikes                             NA         NA      NA       NA
    ## strong           0.1351351351351268437         NA      NA       NA
    ## structure                           NA         NA      NA       NA
    ## struggling                          NA         NA      NA       NA
    ## stuck                               NA         NA      NA       NA
    ## student          0.0693069306930645251         NA      NA       NA
    ## students                            NA         NA      NA       NA
    ## studio           0.2592592592592423695         NA      NA       NA
    ## studios          0.4347826086956320113         NA      NA       NA
    ## study                               NA         NA      NA       NA
    ## stuff           -0.0000000000000102616         NA      NA       NA
    ## stunt            0.0555555555555314398         NA      NA       NA
    ## stupid                              NA         NA      NA       NA
    ## style            0.0249999999999884620         NA      NA       NA
    ## sub                                 NA         NA      NA       NA
    ## subject          0.0000000000000012271         NA      NA       NA
    ## subreddit                           NA         NA      NA       NA
    ## subreddits                          NA         NA      NA       NA
    ## subtitles        0.2631578947368181653         NA      NA       NA
    ## success          0.0000000000000002006         NA      NA       NA
    ## successful       0.0833333333332952064         NA      NA       NA
    ## suck            -0.0000000000000369571         NA      NA       NA
    ## suddenly         0.1304347826086832418         NA      NA       NA
    ## sue              0.2380952380952188474         NA      NA       NA
    ## sued                                NA         NA      NA       NA
    ## suggest         -0.0000000000000361158         NA      NA       NA
    ## suggestion      -0.0000000000000302292         NA      NA       NA
    ## suggestions     -0.0000000000000143676         NA      NA       NA
    ## suggests         0.4545454545454111761         NA      NA       NA
    ## suicide          0.2380952380952229275         NA      NA       NA
    ## suit             0.4615384615384257594         NA      NA       NA
    ## summer          -0.0000000000000245337         NA      NA       NA
    ## sun                                 NA         NA      NA       NA
    ## sunday                              NA         NA      NA       NA
    ## super            0.0499999999999942851         NA      NA       NA
    ## superhero        0.2222222222222038079         NA      NA       NA
    ## superman         0.1666666666667180885         NA      NA       NA
    ## supplies         0.0769230769230418443         NA      NA       NA
    ## supply          -0.0000000000000402872         NA      NA       NA
    ## support                             NA         NA      NA       NA
    ## supported                           NA         NA      NA       NA
    ## supporter                           NA         NA      NA       NA
    ## supporters                          NA         NA      NA       NA
    ## supporting       0.0434782608695567854         NA      NA       NA
    ## supposed        -0.0000000000000142304         NA      NA       NA
    ## supreme          0.5555555555554350100         NA      NA       NA
    ## sure             0.0674157303370732652         NA      NA       NA
    ## surge                               NA         NA      NA       NA
    ## surgery          0.2799999999999934763         NA      NA       NA
    ## surgical                            NA         NA      NA       NA
    ## surprise                            NA         NA      NA       NA
    ## surveillance     0.1851851851851663011         NA      NA       NA
    ## survey           0.4166666666666324348         NA      NA       NA
    ## survival                            NA         NA      NA       NA
    ## survive                             NA         NA      NA       NA
    ## suspected        0.0740740740740580272         NA      NA       NA
    ## suspended                           NA         NA      NA       NA
    ## swear                               NA         NA      NA       NA
    ## sweden           0.1666666666666410668         NA      NA       NA
    ## swedish                             NA         NA      NA       NA
    ## sweet           -0.0000000000000358052         NA      NA       NA
    ## swine            0.0099009900990046727         NA      NA       NA
    ## switch                              NA         NA      NA       NA
    ## switzerland     -0.0000000000000449135         NA      NA       NA
    ## symbol          -0.0000000000000378611         NA      NA       NA
    ## symphony                            NA         NA      NA       NA
    ## symptoms         0.0909090909090815164         NA      NA       NA
    ## system                              NA         NA      NA       NA
    ## systems          0.2040816326530185132         NA      NA       NA
    ## t                0.0204918032786875100         NA      NA       NA
    ## table           -0.0000000000000459865         NA      NA       NA
    ## tackle          -0.0000000000000053088         NA      NA       NA
    ## tactics          0.2499999999999958089         NA      NA       NA
    ## taiwan                              NA         NA      NA       NA
    ## take                                NA         NA      NA       NA
    ## taken            0.1363636363636246407         NA      NA       NA
    ## takes                               NA         NA      NA       NA
    ## taking           0.1097560975609692818         NA      NA       NA
    ## tale            -0.0000000000000335694         NA      NA       NA
    ## talk                                NA         NA      NA       NA
    ## talking          0.0952380952380814660         NA      NA       NA
    ## talks            0.0243902439024278085         NA      NA       NA
    ## tank             0.0526315789473499329         NA      NA       NA
    ## tankian         -0.0000000000000317640         NA      NA       NA
    ## target           0.2631578947368202193         NA      NA       NA
    ## targeted                            NA         NA      NA       NA
    ## targets                             NA         NA      NA       NA
    ## task             0.0588235294117376156         NA      NA       NA
    ## tasks                               NA         NA      NA       NA
    ## taste                               NA         NA      NA       NA
    ## tattoo          -0.0000000000000409198         NA      NA       NA
    ## taught                              NA         NA      NA       NA
    ## tax              0.1323529411764505226         NA      NA       NA
    ## taxes           -0.0000000000000159370         NA      NA       NA
    ## td2             -0.0000000000000395327         NA      NA       NA
    ## te              -0.0000000000000199965         NA      NA       NA
    ## teach                               NA         NA      NA       NA
    ## teacher                             NA         NA      NA       NA
    ## teaching                            NA         NA      NA       NA
    ## team             0.0961538461538437861         NA      NA       NA
    ## teams            0.0833333333332931664         NA      NA       NA
    ## teaser           0.4615384615383739120         NA      NA       NA
    ## tech                                NA         NA      NA       NA
    ## technique                           NA         NA      NA       NA
    ## techniques       0.1999999999999595712         NA      NA       NA
    ## technological   -0.0000000000000440047         NA      NA       NA
    ## technologies     0.2857142857142910830         NA      NA       NA
    ## technology                          NA         NA      NA       NA
    ## ted              0.1818181818181495435         NA      NA       NA
    ## teen             0.3809523809523622795         NA      NA       NA
    ## teenagers        0.2727272727272217479         NA      NA       NA
    ## teeth           -0.0000000000000356303         NA      NA       NA
    ## television       0.0909090909090688737         NA      NA       NA
    ## tell             0.0874999999999961087         NA      NA       NA
    ## telling                             NA         NA      NA       NA
    ## tells                               NA         NA      NA       NA
    ## ten             -0.0000000000000382678         NA      NA       NA
    ## tenant                              NA         NA      NA       NA
    ## term                                NA         NA      NA       NA
    ## terminator       0.1499999999999871436         NA      NA       NA
    ## terms           -0.0000000000000171988         NA      NA       NA
    ## terrible         0.2325581395348735592         NA      NA       NA
    ## tesla                               NA         NA      NA       NA
    ## test                                NA         NA      NA       NA
    ## tested           0.0384615384615261888         NA      NA       NA
    ## testing                             NA         NA      NA       NA
    ## tests                               NA         NA      NA       NA
    ## texas                               NA         NA      NA       NA
    ## text             0.0454545454545422778         NA      NA       NA
    ## texting          0.0769230769230408867         NA      NA       NA
    ## thailand        -0.0000000000000360541         NA      NA       NA
    ## thank            0.2142857142857007846         NA      NA       NA
    ## thanks                              NA         NA      NA       NA
    ## thanksgiving     0.7272727272726122738         NA      NA       NA
    ## the_donald       0.8333333333333540205         NA      NA       NA
    ## theater          0.0465116279069739125         NA      NA       NA
    ## theaters                            NA         NA      NA       NA
    ## theatre          0.0921052631578885839         NA      NA       NA
    ## theatres                            NA         NA      NA       NA
    ## theme                               NA         NA      NA       NA
    ## theory           0.0208333333333237600         NA      NA       NA
    ## thing                               NA         NA      NA       NA
    ## things                              NA         NA      NA       NA
    ## think            0.0200400801603197697         NA      NA       NA
    ## thinking                            NA         NA      NA       NA
    ## thinks           0.1587301587301535588         NA      NA       NA
    ## third                               NA         NA      NA       NA
    ## though           0.0263157894736705499         NA      NA       NA
    ## thought          0.0416666666666625912         NA      NA       NA
    ## thoughts         0.0510204081632648423         NA      NA       NA
    ## thousands        0.1333333333333195925         NA      NA       NA
    ## thread           0.0714285714285703283         NA      NA       NA
    ## threat           0.0357142857142754497         NA      NA       NA
    ## threatened       0.1304347826086748041         NA      NA       NA
    ## threatening     -0.0000000000000246811         NA      NA       NA
    ## threatens       -0.0000000000000350553         NA      NA       NA
    ## threats                             NA         NA      NA       NA
    ## three            0.0909090909090759791         NA      NA       NA
    ## thriller                            NA         NA      NA       NA
    ## throughout                          NA         NA      NA       NA
    ## throw           -0.0000000000000297521         NA      NA       NA
    ## throwing                            NA         NA      NA       NA
    ## thrown                              NA         NA      NA       NA
    ## throws                              NA         NA      NA       NA
    ## thunberg                            NA         NA      NA       NA
    ## `thunberg's`    -0.0000000000000223874         NA      NA       NA
    ## ticket           0.0555555555555298022         NA      NA       NA
    ## tickets         -0.0000000000000184445         NA      NA       NA
    ## tier            -0.0000000000000317963         NA      NA       NA
    ## til                                 NA         NA      NA       NA
    ## till             0.0909090909090614629         NA      NA       NA
    ## tim              0.0588235294117361376         NA      NA       NA
    ## time             0.0173913043478253065         NA      NA       NA
    ## timeline         0.0999999999999543338         NA      NA       NA
    ## times                               NA         NA      NA       NA
    ## tinder           0.0714285714285709666         NA      NA       NA
    ## tiny                                NA         NA      NA       NA
    ## tips            -0.0000000000000143336         NA      NA       NA
    ## tired            0.1599999999999788536         NA      NA       NA
    ## title            0.1449275362318922789         NA      NA       NA
    ## titles          -0.0000000000000292425         NA      NA       NA
    ## today            0.0429184549356197431         NA      NA       NA
    ## `today's`                           NA         NA      NA       NA
    ## together         0.2380952380952262026         NA      NA       NA
    ## toilet          -0.0000000000000815224         NA      NA       NA
    ## told                                NA         NA      NA       NA
    ## toll                                NA         NA      NA       NA
    ## tom              0.0731707317073045788         NA      NA       NA
    ## tomorrow         0.1086956521739002868         NA      NA       NA
    ## tonight                             NA         NA      NA       NA
    ## tony                                NA         NA      NA       NA
    ## took                                NA         NA      NA       NA
    ## tool                                NA         NA      NA       NA
    ## tools                               NA         NA      NA       NA
    ## top                                 NA         NA      NA       NA
    ## topic                               NA         NA      NA       NA
    ## topics                              NA         NA      NA       NA
    ## toronto         -0.0000000000000306188         NA      NA       NA
    ## total            0.3478260869565020785         NA      NA       NA
    ## totally          0.4166666666666494767         NA      NA       NA
    ## touch           -0.0000000000000265644         NA      NA       NA
    ## touching        -0.0000000000000531221         NA      NA       NA
    ## toward           0.7999999999999491962         NA      NA       NA
    ## towards          0.0344827586206731401         NA      NA       NA
    ## tower                               NA         NA      NA       NA
    ## town             0.1320754716980948873         NA      NA       NA
    ## towns            0.0769230769230463685         NA      NA       NA
    ## toxic                               NA         NA      NA       NA
    ## toy              0.1111111111110848898         NA      NA       NA
    ## track                               NA         NA      NA       NA
    ## tracking        -0.0000000000000160832         NA      NA       NA
    ## tracks                              NA         NA      NA       NA
    ## trade                               NA         NA      NA       NA
    ## traditional                         NA         NA      NA       NA
    ## traffic         -0.0000000000000516928         NA      NA       NA
    ## trailer          0.0840336134453740918         NA      NA       NA
    ## trailers                            NA         NA      NA       NA
    ## train                               NA         NA      NA       NA
    ## trained          0.0740740740740806897         NA      NA       NA
    ## training         0.0476190476190475956         NA      NA       NA
    ## transfer         0.0714285714285386036         NA      NA       NA
    ## transform                           NA         NA      NA       NA
    ## translate        0.1818181818181509868         NA      NA       NA
    ## translation     -0.0000000000000334872         NA      NA       NA
    ## trash            0.0666666666666543561         NA      NA       NA
    ## trashy           0.0526315789473442083         NA      NA       NA
    ## travel                              NA         NA      NA       NA
    ## treat            0.1739130434782129864         NA      NA       NA
    ## treated                             NA         NA      NA       NA
    ## treating        -0.0000000000000413764         NA      NA       NA
    ## treatment        0.0476190476190255160         NA      NA       NA
    ## tree                                NA         NA      NA       NA
    ## trek             0.2727272727272309627         NA      NA       NA
    ## trend                               NA         NA      NA       NA
    ## trending                            NA         NA      NA       NA
    ## trends          -0.0000000000000240708         NA      NA       NA
    ## trey                                NA         NA      NA       NA
    ## trial            0.1481481481481317919         NA      NA       NA
    ## trials          -0.0000000000000944879         NA      NA       NA
    ## tricks                              NA         NA      NA       NA
    ## tried            0.0749999999999857864         NA      NA       NA
    ## tries            0.0857142857142664388         NA      NA       NA
    ## trilogy                             NA         NA      NA       NA
    ## trip                                NA         NA      NA       NA
    ## troll            0.1999999999999587663         NA      NA       NA
    ## trolls           0.1666666666666909991         NA      NA       NA
    ## trouble                             NA         NA      NA       NA
    ## truck            0.1304347826086851569         NA      NA       NA
    ## true             0.0793650793650731712         NA      NA       NA
    ## truly           -0.0000000000000220560         NA      NA       NA
    ## trump                               NA         NA      NA       NA
    ## `trump's`                           NA         NA      NA       NA
    ## trust            0.1290322580645077433         NA      NA       NA
    ## truth            0.1333333333333036330         NA      NA       NA
    ## try                                 NA         NA      NA       NA
    ## trying           0.0653594771241800959         NA      NA       NA
    ## tuesday          0.0416666666666476518         NA      NA       NA
    ## turing          -0.0000000000000430621         NA      NA       NA
    ## turkey          -0.0000000000000416450         NA      NA       NA
    ## turn                                NA         NA      NA       NA
    ## turned           0.1475409836065559310         NA      NA       NA
    ## turning         -0.0000000000000381065         NA      NA       NA
    ## turnout                             NA         NA      NA       NA
    ## turns                               NA         NA      NA       NA
    ## tv                                  NA         NA      NA       NA
    ## tweet            0.1499999999999775957         NA      NA       NA
    ## tweets                              NA         NA      NA       NA
    ## twice            0.1333333333332990811         NA      NA       NA
    ## twitter                             NA         NA      NA       NA
    ## two                                 NA         NA      NA       NA
    ## tx              -0.0000000000000213971         NA      NA       NA
    ## type             0.1081081081080872697         NA      NA       NA
    ## types           -0.0000000000000357285         NA      NA       NA
    ## u                                   NA         NA      NA       NA
    ## u.s              0.0564971751412337159         NA      NA       NA
    ## uber                                NA         NA      NA       NA
    ## ubi             -0.0000000000000150303         NA      NA       NA
    ## uefa             0.0909090909090396748         NA      NA       NA
    ## ufc              0.1075268817204235172         NA      NA       NA
    ## ugly            -0.0000000000000450041         NA      NA       NA
    ## uk                                  NA         NA      NA       NA
    ## ukraine         -0.0000000000000165209         NA      NA       NA
    ## ultimate                            NA         NA      NA       NA
    ## un                                  NA         NA      NA       NA
    ## uncle                               NA         NA      NA       NA
    ## underground                         NA         NA      NA       NA
    ## understand                          NA         NA      NA       NA
    ## understanding   -0.0000000000000231663         NA      NA       NA
    ## unemployment    -0.0000000000000161875         NA      NA       NA
    ## unexpected       0.0999999999999552774         NA      NA       NA
    ## union            0.1764705882352797239         NA      NA       NA
    ## unique          -0.0000000000000370453         NA      NA       NA
    ## united           0.0746268656716364276         NA      NA       NA
    ## universal                           NA         NA      NA       NA
    ## universe         0.0624999999999838185         NA      NA       NA
    ## university       0.0869565217391208706         NA      NA       NA
    ## unknown                             NA         NA      NA       NA
    ## unless                              NA         NA      NA       NA
    ## unpopular        0.0769230769230379863         NA      NA       NA
    ## unveils          0.1818181818181496545         NA      NA       NA
    ## upcoming         0.0465116279069705332         NA      NA       NA
    ## update           0.1904761904761682612         NA      NA       NA
    ## updated          0.4615384615384313660         NA      NA       NA
    ## updates                             NA         NA      NA       NA
    ## upon             0.3225806451612868719         NA      NA       NA
    ## upset            0.1176470588234568154         NA      NA       NA
    ## urge             0.0909090909090453231         NA      NA       NA
    ## urgent                              NA         NA      NA       NA
    ## us               0.0154083204930658656         NA      NA       NA
    ## usa              0.1204819277108388625         NA      NA       NA
    ## usage            0.6999999999999048095         NA      NA       NA
    ## use                                 NA         NA      NA       NA
    ## used                                NA         NA      NA       NA
    ## useful          -0.0000000000000226776         NA      NA       NA
    ## useless          0.0555555555555229189         NA      NA       NA
    ## user             0.1086956521739100429         NA      NA       NA
    ## users                               NA         NA      NA       NA
    ## uses                                NA         NA      NA       NA
    ## using            0.0387596899224807306         NA      NA       NA
    ## usually         -0.0000000000000241977         NA      NA       NA
    ## v                                   NA         NA      NA       NA
    ## vaccinated                          NA         NA      NA       NA
    ## vaccination      0.2222222222221986732         NA      NA       NA
    ## vaccinations                        NA         NA      NA       NA
    ## vaccine          0.0247933884297495896         NA      NA       NA
    ## vaccines         0.0571428571428487728         NA      NA       NA
    ## valley          -0.0000000000000321944         NA      NA       NA
    ## value            0.0666666666666490965         NA      NA       NA
    ## values           0.5454545454545179917         NA      NA       NA
    ## van                                 NA         NA      NA       NA
    ## vax             -0.0000000000000397245         NA      NA       NA
    ## vaxxer                              NA         NA      NA       NA
    ## vaxxers                             NA         NA      NA       NA
    ## ve                                  NA         NA      NA       NA
    ## vehicle                             NA         NA      NA       NA
    ## version                             NA         NA      NA       NA
    ## veteran          0.0909090909090292110         NA      NA       NA
    ## via              0.0526315789473565179         NA      NA       NA
    ## vice            -0.0000000000000277813         NA      NA       NA
    ## victim                              NA         NA      NA       NA
    ## victims         -0.0000000000000385738         NA      NA       NA
    ## victory          0.1199999999999778327         NA      NA       NA
    ## video            0.0401606425702786610         NA      NA       NA
    ## videos           0.0163934426229446206         NA      NA       NA
    ## view             0.3125000000000014433         NA      NA       NA
    ## views            0.1199999999999831618         NA      NA       NA
    ## villain          0.0588235294117376017         NA      NA       NA
    ## violating                           NA         NA      NA       NA
    ## violence                            NA         NA      NA       NA
    ## violent                             NA         NA      NA       NA
    ## viral                               NA         NA      NA       NA
    ## virginia         0.2941176470588092728         NA      NA       NA
    ## virtual          0.0399999999999888223         NA      NA       NA
    ## virus            0.0549450549450544515         NA      NA       NA
    ## viruses          0.0370370370370193894         NA      NA       NA
    ## visa            -0.0000000000000594520         NA      NA       NA
    ## vision                              NA         NA      NA       NA
    ## visit            0.1562499999999776290         NA      NA       NA
    ## visual          -0.0000000000000413419         NA      NA       NA
    ## voice                               NA         NA      NA       NA
    ## volunteer                           NA         NA      NA       NA
    ## vote                                NA         NA      NA       NA
    ## voted            0.1492537313432700519         NA      NA       NA
    ## voter                               NA         NA      NA       NA
    ## voters           0.1315789473684156330         NA      NA       NA
    ## votes            0.1052631578947229724         NA      NA       NA
    ## voting           0.0520833333333310944         NA      NA       NA
    ## vr              -0.0000000000000257133         NA      NA       NA
    ## vs                                  NA         NA      NA       NA
    ## w                                   NA         NA      NA       NA
    ## wa              -0.0000000000000474656         NA      NA       NA
    ## wage            -0.0000000000000249568         NA      NA       NA
    ## wait                                NA         NA      NA       NA
    ## waiting          0.2249999999999880707         NA      NA       NA
    ## wake             0.0476190476190393244         NA      NA       NA
    ## walk                                NA         NA      NA       NA
    ## walking                             NA         NA      NA       NA
    ## wall                                NA         NA      NA       NA
    ## walmart                             NA         NA      NA       NA
    ## want             0.0369003690036866158         NA      NA       NA
    ## wanted                              NA         NA      NA       NA
    ## wanting                             NA         NA      NA       NA
    ## wants                               NA         NA      NA       NA
    ## war              0.0877192982456100750         NA      NA       NA
    ## warfare          0.0588235294117516183         NA      NA       NA
    ## warming         -0.0000000000000230047         NA      NA       NA
    ## warn                                NA         NA      NA       NA
    ## warned                              NA         NA      NA       NA
    ## warner           0.2352941176470330908         NA      NA       NA
    ## warning          0.1025641025641026577         NA      NA       NA
    ## warns                               NA         NA      NA       NA
    ## warren           0.1481481481481266294         NA      NA       NA
    ## wars                                NA         NA      NA       NA
    ## washington       0.0714285714285553819         NA      NA       NA
    ## wasn             0.1818181818181716369         NA      NA       NA
    ## waste           -0.0000000000000386134         NA      NA       NA
    ## watch                               NA         NA      NA       NA
    ## watched          0.1041666666666639096         NA      NA       NA
    ## watches                             NA         NA      NA       NA
    ## watching         0.0495049504950435676         NA      NA       NA
    ## water            0.1249999999999902717         NA      NA       NA
    ## watson          -0.0000000000000454200         NA      NA       NA
    ## wave             0.1612903225806305296         NA      NA       NA
    ## way              0.0338983050847438058         NA      NA       NA
    ## ways             0.0192307692307582406         NA      NA       NA
    ## wcgw                                NA         NA      NA       NA
    ## weapon           0.1249999999999814593         NA      NA       NA
    ## weapons          0.0909090909090778387         NA      NA       NA
    ## wear             0.0285714285714179714         NA      NA       NA
    ## wearing          0.1052631578947069296         NA      NA       NA
    ## weather         -0.0000000000000221197         NA      NA       NA
    ## web             -0.0000000000000302342         NA      NA       NA
    ## website                             NA         NA      NA       NA
    ## websites        -0.0000000000000249238         NA      NA       NA
    ## wedding          0.1249999999999711203         NA      NA       NA
    ## week             0.0624999999999993477         NA      NA       NA
    ## weekend          0.0999999999999838796         NA      NA       NA
    ## weeks            0.1016949152542264700         NA      NA       NA
    ## wei             -0.0000000000000163539         NA      NA       NA
    ## weird            0.1999999999999950984         NA      NA       NA
    ## weiwei                              NA         NA      NA       NA
    ## welcome          0.3846153846153764211         NA      NA       NA
    ## well             0.0692307692307692624         NA      NA       NA
    ## went             0.1190476190476034285         NA      NA       NA
    ## west                                NA         NA      NA       NA
    ## western                             NA         NA      NA       NA
    ## whatever                            NA         NA      NA       NA
    ## whats                               NA         NA      NA       NA
    ## whenever        -0.0000000000000320784         NA      NA       NA
    ## whether          0.0606060606060538909         NA      NA       NA
    ## white                               NA         NA      NA       NA
    ## whoever                             NA         NA      NA       NA
    ## whole                               NA         NA      NA       NA
    ## whose           -0.0000000000000420242         NA      NA       NA
    ## wide            -0.0000000000000325496         NA      NA       NA
    ## widespread                          NA         NA      NA       NA
    ## wife                                NA         NA      NA       NA
    ## wikipedia       -0.0000000000000382928         NA      NA       NA
    ## wild                                NA         NA      NA       NA
    ## willing          0.0666666666666300978         NA      NA       NA
    ## win                                 NA         NA      NA       NA
    ## wind             0.0666666666666542729         NA      NA       NA
    ## window          -0.0000000000000560661         NA      NA       NA
    ## windows                             NA         NA      NA       NA
    ## wing                                NA         NA      NA       NA
    ## winner           0.1515151515151383910         NA      NA       NA
    ## winning          0.0816326530612144774         NA      NA       NA
    ## wins             0.1052631578947316737         NA      NA       NA
    ## winter                              NA         NA      NA       NA
    ## wise             0.0999999999999415662         NA      NA       NA
    ## wish                                NA         NA      NA       NA
    ## wishes                              NA         NA      NA       NA
    ## within           0.0392156862745010951         NA      NA       NA
    ## without          0.0568181818181787771         NA      NA       NA
    ## woke             0.1538461538461198541         NA      NA       NA
    ## woman            0.0694444444444366205         NA      NA       NA
    ## women                               NA         NA      NA       NA
    ## won                                 NA         NA      NA       NA
    ## wonder                              NA         NA      NA       NA
    ## wondering       -0.0000000000000330923         NA      NA       NA
    ## word             0.0943396226415003647         NA      NA       NA
    ## words                               NA         NA      NA       NA
    ## work             0.0300300300300297071         NA      NA       NA
    ## worked                              NA         NA      NA       NA
    ## worker                              NA         NA      NA       NA
    ## workers          0.0483870967741855257         NA      NA       NA
    ## workforce        0.0909090909090484178         NA      NA       NA
    ## working          0.1136363636363630081         NA      NA       NA
    ## workplace       -0.0000000000000388537         NA      NA       NA
    ## works            0.0862068965517156366         NA      NA       NA
    ## world                               NA         NA      NA       NA
    ## `world's`                           NA         NA      NA       NA
    ## worldnews        0.0909090909090496946         NA      NA       NA
    ## worldwide        0.3703703703703567496         NA      NA       NA
    ## worried                             NA         NA      NA       NA
    ## worry            0.0555555555555276581         NA      NA       NA
    ## worse            0.0769230769230699052         NA      NA       NA
    ## worst            0.0819672131147491795         NA      NA       NA
    ## worth            0.0468749999999982583         NA      NA       NA
    ## wow              0.1666666666666274943         NA      NA       NA
    ## write                               NA         NA      NA       NA
    ## writers                             NA         NA      NA       NA
    ## writing          0.0476190476190213319         NA      NA       NA
    ## written          0.2195121951219491940         NA      NA       NA
    ## wrong            0.0917431192660509431         NA      NA       NA
    ## wrote            0.2068965517241225793         NA      NA       NA
    ## wtf                                 NA         NA      NA       NA
    ## wuhan            0.0781249999999995698         NA      NA       NA
    ## x                                   NA         NA      NA       NA
    ## xpost           -0.0000000000000456284         NA      NA       NA
    ## y                                   NA         NA      NA       NA
    ## `y'all`          0.0499999999999808931         NA      NA       NA
    ## ya                                  NA         NA      NA       NA
    ## yang                                NA         NA      NA       NA
    ## `yang's`                            NA         NA      NA       NA
    ## yanggang        -0.0000000000000166804         NA      NA       NA
    ## yeah             0.1818181818182215692         NA      NA       NA
    ## year             0.0201612903225765021         NA      NA       NA
    ## `year's`                            NA         NA      NA       NA
    ## years                               NA         NA      NA       NA
    ## yes                                 NA         NA      NA       NA
    ## yesterday                           NA         NA      NA       NA
    ## yet                                 NA         NA      NA       NA
    ## yo              -0.0000000000000520443         NA      NA       NA
    ## york                                NA         NA      NA       NA
    ## young            0.1228070175438578659         NA      NA       NA
    ## younger                             NA         NA      NA       NA
    ## youth            0.0666666666666306945         NA      NA       NA
    ## youtube                             NA         NA      NA       NA
    ## ysk              0.0294117647058813568         NA      NA       NA
    ## zealand         -0.0000000000000418143         NA      NA       NA
    ## zero                                NA         NA      NA       NA
    ## zombie          -0.0000000000000320331         NA      NA       NA
    ## zone                                NA         NA      NA       NA
    ## zuckerberg                          NA         NA      NA       NA
    ## 
    ## Residual standard error: NaN on 0 degrees of freedom
    ## Multiple R-squared:      1,  Adjusted R-squared:    NaN 
    ## F-statistic:   NaN on 2069 and 0 DF,  p-value: NA

11. Run Deep learning models in H2O.

<!-- end list -->

``` r
df6<-df5 %>%    
  spread(key = word, value = count)

df6[is.na(df6)] <- 0

#split dataset
which_train <- sample(x = c(TRUE, FALSE), size = nrow(df6),
                      replace = TRUE, prob = c(0.8, 0.2))

recc_data_train <- df6[which_train, ]
recc_data_valid <- df6[!which_train, ]

length(recc_data_train[[1]])
```

    ## [1] 2752

``` r
length(recc_data_valid[[1]])
```

    ## [1] 698

``` r
x_train_processed_tbl <- recc_data_train %>% select(-num_comments)
y_train_processed_tbl <- recc_data_train %>% select(num_comments) 
x_test_processed_tbl  <- recc_data_valid

h2o.init(nthreads = -1)
```

    ## 
    ## H2O is not running yet, starting it now...
    ## 
    ## Note:  In case of errors look at the following log files:
    ##     C:\Users\anush\AppData\Local\Temp\RtmpS8dM7V/h2o_anush_started_from_r.out
    ##     C:\Users\anush\AppData\Local\Temp\RtmpS8dM7V/h2o_anush_started_from_r.err
    ## 
    ## 
    ## Starting H2O JVM and connecting: . Connection successful!
    ## 
    ## R is connected to the H2O cluster: 
    ##     H2O cluster uptime:         4 seconds 251 milliseconds 
    ##     H2O cluster timezone:       America/Chicago 
    ##     H2O data parsing timezone:  UTC 
    ##     H2O cluster version:        3.28.0.2 
    ##     H2O cluster version age:    2 months and 18 days  
    ##     H2O cluster name:           H2O_started_from_R_anush_bbm871 
    ##     H2O cluster total nodes:    1 
    ##     H2O cluster total memory:   0.97 GB 
    ##     H2O cluster total cores:    8 
    ##     H2O cluster allowed cores:  8 
    ##     H2O cluster healthy:        TRUE 
    ##     H2O Connection ip:          localhost 
    ##     H2O Connection port:        54321 
    ##     H2O Connection proxy:       NA 
    ##     H2O Internal Security:      FALSE 
    ##     H2O API Extensions:         Amazon S3, Algos, AutoML, Core V3, TargetEncoder, Core V4 
    ##     R Version:                  R version 3.6.2 (2019-12-12)

``` r
h2o.clusterInfo()
```

    ## R is connected to the H2O cluster: 
    ##     H2O cluster uptime:         4 seconds 442 milliseconds 
    ##     H2O cluster timezone:       America/Chicago 
    ##     H2O data parsing timezone:  UTC 
    ##     H2O cluster version:        3.28.0.2 
    ##     H2O cluster version age:    2 months and 18 days  
    ##     H2O cluster name:           H2O_started_from_R_anush_bbm871 
    ##     H2O cluster total nodes:    1 
    ##     H2O cluster total memory:   0.97 GB 
    ##     H2O cluster total cores:    8 
    ##     H2O cluster allowed cores:  8 
    ##     H2O cluster healthy:        TRUE 
    ##     H2O Connection ip:          localhost 
    ##     H2O Connection port:        54321 
    ##     H2O Connection proxy:       NA 
    ##     H2O Internal Security:      FALSE 
    ##     H2O API Extensions:         Amazon S3, Algos, AutoML, Core V3, TargetEncoder, Core V4 
    ##     R Version:                  R version 3.6.2 (2019-12-12)

``` r
data_h2o <- as.h2o(
  bind_cols(y_train_processed_tbl, x_train_processed_tbl),
  destination_frame= "train.hex" #destination_frame is optional
)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |======================================================================| 100%

``` r
new_data_h2o <- as.h2o(
  x_test_processed_tbl,
  destination_frame= "test.hex" #destination_frame is optional
)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |======================================================================| 100%

``` r
h2o.ls()
```

    ##         key
    ## 1  test.hex
    ## 2 train.hex

``` r
splits <- h2o.splitFrame(data = data_h2o,
                         ratios = c(0.7, 0.15), # 70/15/15 split
                         seed = 1234)
train_h2o <- splits[[1]] # from training data
valid_h2o <- splits[[2]] # from training data
test_h2o <- splits[[3]] # from training data


y <- "num_comments" # column name for outcome
x <- setdiff(names(train_h2o), y) # column names for predictors
m1 <- h2o.deeplearning(
  model_id = "dl_model_first",
  x = x,
  y = y,
  training_frame = train_h2o,
  validation_frame = valid_h2o, ## validation dataset: used for scoring and
  ## early stopping
  #activation="Rectifier", ## default
  #hidden=c(200,200), ## default: 2 hidden layers, 200 neurons each
  epochs = 1 ## one pass over the training data
)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |=======                                                               |  10%  |                                                                              |=====================                                                 |  30%  |                                                                              |===================================                                   |  50%  |                                                                              |=================================================                     |  70%  |                                                                              |===============================================================       |  90%  |                                                                              |======================================================================| 100%

``` r
summary(m1)
```

    ## Model Details:
    ## ==============
    ## 
    ## H2ORegressionModel: deeplearning
    ## Model Key:  dl_model_first 
    ## Status of Neuron Layers: predicting num_comments, regression, gaussian distribution, Quadratic loss, 429,201 weights/biases, 5.2 MB, 2,108 training samples, mini-batch size 1
    ##   layer units      type dropout       l1       l2 mean_rate rate_rms momentum
    ## 1     1  1943     Input  0.00 %       NA       NA        NA       NA       NA
    ## 2     2   200 Rectifier  0.00 % 0.000000 0.000000  0.102066 0.070021 0.000000
    ## 3     3   200 Rectifier  0.00 % 0.000000 0.000000  0.004942 0.009833 0.000000
    ## 4     4     1    Linear      NA 0.000000 0.000000  0.000190 0.000105 0.000000
    ##   mean_weight weight_rms mean_bias bias_rms
    ## 1          NA         NA        NA       NA
    ## 2    0.008857   0.030908  0.488814 0.012677
    ## 3   -0.004732   0.069751  0.993381 0.006080
    ## 4   -0.000633   0.088509  0.004458 0.000000
    ## 
    ## H2ORegressionMetrics: deeplearning
    ## ** Reported on training data. **
    ## ** Metrics reported on full training frame **
    ## 
    ## MSE:  199500451
    ## RMSE:  14124.46
    ## MAE:  8336.767
    ## RMSLE:  NaN
    ## Mean Residual Deviance :  199500451
    ## 
    ## 
    ## H2ORegressionMetrics: deeplearning
    ## ** Reported on validation data. **
    ## ** Metrics reported on full validation frame **
    ## 
    ## MSE:  242488651
    ## RMSE:  15572.05
    ## MAE:  7551.954
    ## RMSLE:  1.794801
    ## Mean Residual Deviance :  242488651
    ## 
    ## 
    ## 
    ## 
    ## Scoring History: 
    ##             timestamp   duration training_speed  epochs iterations     samples
    ## 1 2020-04-08 15:33:45  0.000 sec             NA 0.00000          0    0.000000
    ## 2 2020-04-08 15:33:46  7.253 sec    256 obs/sec 0.10968          1  213.000000
    ## 3 2020-04-08 15:33:58 19.270 sec    261 obs/sec 1.08548         11 2108.000000
    ##   training_rmse training_deviance training_mae training_r2 validation_rmse
    ## 1            NA                NA           NA          NA              NA
    ## 2   19722.19533   388964988.78998  13476.59033    -0.76335     15724.16665
    ## 3   14124.46286   199500451.14259   8336.76661     0.09558     15572.04711
    ##   validation_deviance validation_mae validation_r2
    ## 1                  NA             NA            NA
    ## 2     247249416.72997     8401.16457      -0.02648
    ## 3     242488651.26250     7551.95381      -0.00671
    ## 
    ## Variable Importances: (Extract with `h2o.varimp`) 
    ## =================================================
    ## 
    ## Variable Importances: 
    ##    variable relative_importance scaled_importance percentage
    ## 1     house            1.000000          1.000000   0.000610
    ## 2   matters            0.997848          0.997848   0.000608
    ## 3       sun            0.980669          0.980669   0.000598
    ## 4   dealing            0.980455          0.980455   0.000598
    ## 5 minnesota            0.978845          0.978845   0.000597
    ## 
    ## ---
    ##      variable relative_importance scaled_importance percentage
    ## 1938   taiwan            0.688293          0.688293   0.000420
    ## 1939    beach            0.681338          0.681338   0.000415
    ## 1940   matter            0.676494          0.676494   0.000413
    ## 1941     copy            0.672807          0.672807   0.000410
    ## 1942      kit            0.667892          0.667892   0.000407
    ## 1943    truck            0.647182          0.647182   0.000395

``` r
prediction_h2o_dl <- h2o.predict(m1,
                                 newdata = new_data_h2o)
```

    ##   |                                                                              |                                                                      |   0%  |                                                                              |======================================================================| 100%

``` r
prediction_dl_tbl <- tibble(
  SK_ID_CURR = x_test_processed_tbl$num_comments,
  TARGET = as.vector(prediction_h2o_dl$predict)
)

h2o.shutdown(prompt = F)
```

    ## [1] TRUE

From the result, RMSE was bad, I am getting same prediction for each
word. Since the dataset become bigger and bigger, the word set used is
getting bigger as well. That could be one of the reasons why this is not
working. The number of dataset might be one of the reason too.

12. Compare to Youtube

It seems people use equal number of positive and negative titles in
Reddit, but its different from youtube.

``` r
application_train <- read.csv("CAvideos.csv",na=c("",NA,"-1"))





df <- application_train[,c("video_id","title","channel_title",
                           "tags","views","likes","dislikes","comment_count","description")]
df <- df %>% mutate(title = as.character(title))
tokens <- df %>%  
  unnest_tokens(output = word, input = title) 


sw = get_stopwords()
cleaned_tokens <- tokens %>%  
  filter(!word %in% sw$word)
nums <- cleaned_tokens %>%   
  filter(str_detect(word, "^[0-9]")) %>%   
  select(word) %>% unique()



cleaned_tokens <- cleaned_tokens %>%   
  filter(!word %in% nums$word)

cleaned_tokens <- cleaned_tokens %>%   
  filter(!grepl("[^\u0001-\u007F]+",cleaned_tokens$word))

rare <- cleaned_tokens %>%   
  dplyr::count(word) %>%  
  filter(n<10) %>%  
  select(word) %>% unique() 

cleaned_tokens <- cleaned_tokens %>%   
  filter(!word %in% rare$word) 


sent_reviews1 = cleaned_tokens %>%   
  left_join(get_sentiments("nrc")) %>%  
  rename(replace=c("sentiment"="nrc")) %>%
  left_join(get_sentiments("bing")) %>%  
  rename(replace=c("sentiment"="bing")) %>%
  left_join(get_sentiments("afinn")) %>%  
  rename(replace=c("value"="affin"))


bing_word_counts1 <- sent_reviews1 %>%  
  filter(!is.na(bing)) %>%  
  dplyr::count(word,bing, sort = TRUE) 


high_hit_word_list<- cleaned_tokens %>%
  ddply("word",summarize,mean=round(mean(views))) %>%
  arrange(desc(mean))


sent_reviews = high_hit_word_list %>%   
  left_join(get_sentiments("nrc")) %>%  
  rename(replace=c("sentiment"="nrc")) %>%
  left_join(get_sentiments("bing")) %>%  
  rename(replace=c("sentiment"="bing")) %>%
  left_join(get_sentiments("afinn")) %>%  
  rename(replace=c("value"="affin"))


bing_word_counts <- sent_reviews %>%  
  filter(!is.na(bing))

bing_word_counts %>%  
  filter(mean > 2000000) %>%  
  mutate(mean = ifelse(bing == "negative", -mean, mean)) %>%  
  mutate(word = reorder(word, mean)) %>%  
  ggplot(aes(word, mean, fill = bing)) +  
  geom_col() +  coord_flip() +  
  labs(y = "number of hits")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
bing_word_counts %>%  
  filter(mean > 10000) %>%  
  mutate(mean = ifelse(bing == "negative", -mean, mean)) %>%  
  mutate(word = reorder(word, mean)) %>%  
  ggplot(aes(word, mean, fill = bing)) +  
  geom_col() +  coord_flip() +  
  labs(y = "Contribution to sentiment")
```

![](Reddit_Sentiment_Analysis-Report_files/figure-gfm/unnamed-chunk-21-2.png)<!-- -->

## Summary

For the first part: For president Trump, it seems people use mostly
negative comments on him, the most used words are :violent, strike,
blame, abuse, attack, lie, suspicious, bomb, emergency…. On question
that can come to mind is how to identify negative comment embedded in
negative comments? It can be seen that there are not so much embedded
negative commentd. Those words are directly related to what
Mr. President did last year.

For the second part: There are lots more negative words in YouTube
titles compare to Reddit. One may think the reason could be: in youtube,
people will look at this video just because the title, not too many
people will read video description and comment before watching, But in
reddit, if you going to leave a review, you must have something
interesting in your post text, not the title. So the title shows a
natural tone overall.

I coudn’t find any relationship between titles and number of comments.
But it might be possible to find something in YouTube titles, because in
the plot, it is obvious that some negative words will attract more video
hits. But that was not so obvious in Reddit.
