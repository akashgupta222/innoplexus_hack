# innoplexus_hack
Approach and codes for innoplexus hackathon (2nd Position)

## Problem
The problem involved predicting the articles which a publication will cite based on its content. 

## Dataset 
- We had been provided with disjoint sets in which some articles cited the others from within that set only. There were total 18 sets and the important thing to note it that the train and test sets had different sets of articles (so no article from train can cite an article from test). 
- This disjoint presentation of sets made the challenge a bit tricky as normal tf-idf feature columns would probably overfit as they would learn the exact words and those might not be present in other sets. 

## Data Transformations to convert to Machine Learning Problem
- This was a tricky part to which I finally reached after long trail with other formations. 
- For each article(lets call it in_article) in a set of size k, form (k-1) pairs with all other articles(lets call it out_article) in that set. Hence, for each set we have a total of k(k-1) pairs. 
- For each pair of articles, have the output variable as 1 if the article in_article cites article out_article, else have it as 0. 
- Now we have total of sum(m*ki*(ki-1)) data points where m is the number of sets and ki is the size of each set and a standard machine learning problem where we need to predcit the output given in_article and out_article. 

## Feature Engineering 
- Here we need to be careful not to have any features which can be exclusize for a set so we cant simple have the tf-idf vectors for in_article's title and out_article's title. 
- So I decided to have the cosine distance between in_article title CountVector and out_article title CountVector. 
- Similarly I had a feature for the cosine distance between in_article abstract Tf-idfVector and out_article tabstract Tf-idfVector. 
- Also I noticed that many authors tend to cite previous papers by themselves and by co authors, so to capture it I included a feature to calculate the intersection of authors of the in_article and out_article as well as the union of these authors and have a value of intersection-over-union of authors. 
- To capture the difference in publishing dates, I had a feature for the no of months between the in_article publishing date and out_article publishing date. Having months instead of dates helped to prevent overfitting. 

## Classification (finally time to call model.fit ) 
- Used a standard LightGBM model with some parameter tuning to precent overfitting, but most of the work has been done in previous steps. You can see the code for the exact parameters I used. 

## Prediction and final output
- The model outputs a proabability score for each pair of articles. 
- To translate it into which articles an in_article will cite, I grouped by a article and got the scores for all articles in that set and sorted according to score. 
- For each in article : 
  - If there exists some out_articles with score greater than 0.5, output all those
  - Else if there exists some out_articles with score greater than 0.4, output all those
  - Else if there exists some out_articles with score greater than 0.3, output all those
  - Else if there exists some out_articles with score greater than 0.2, output all those
  - Else if there exists some out_articles with score greater than 0.1, output all those
  - Else output top 4 scoring articles 
  
- I also observed that if the article's pmid-1 is present in the same set, an article always cites it (probably a continuation of previous work), so if the output doesn't have pmid-1 and pmid-1 is present in the same set, add it to output. 



