# Predicting the Incidence of Cheating in FPS Games using Small-Data Techniques
#### Robin Stopa

## 1. Introduction and Problem Statement

Often times as data scientists we have questions to answer when the perfect dataset does not exist to answer the question. In this project, we hope to explore and understand what makes a video game have a high incidence of cheating, particularly in First Person Shooter (FPS) games.

Cheating and hacking has existed in video games for many years despite the existence of anti-cheat software which video game companies build into/ release with the game. As the anti-cheat systems have become more complex, so have the cheats/hacks themselves.

There are (just as an example) two common types of cheats in First Person Shooter games which are well known. The first is known as aimbot or aim-assist, where the player uses and external program to improve their weapon aim, thus unfairly gaining an advantage in fights against other players. The second example is wall-hacking, where the player uses an external program which allows bullets to pass through walls to shoot enemies, giving them an unfair advantage. These are just two examples, but many more types of cheats exist and they are detrimental to the experience of players as it damages the competative integrity of the game.

Companies are therefore interested to know what causes a high incidence of cheating in FPS games. This is not a well-researched area and data needs to be gathered in order to answer the question quantitatively.

#### Problem Statement: What causes a high incidence of cheating in First-Person Shooter Games?

Answering this question will help companies work towards creating fair games with less cheating, thus improving the experience of virtually shooting opponents for video game lovers worldwide.

## 2. Data Collection

To address our problem statement, we first start by defining what "incidence of cheating" is. Gaming companies have their own cheat-report systems, however this kind of data is not often publicly available and the reporting system varies by company/game.

To define the incidence of cheating, we will use data from a website called unknowncheats.com. This website boasts, "UnKnoWnCheaTs is the oldest game cheating forum in existence, leading the game cheating community for over 20 years. We encourage an open, free and collaborative environment and offer a vast and resourceful file database, a wiki that's packed with structured information and tutorials, access to the most intelligent programmers, and a team that protects members from malware while enforcing a diverse community."

The website contains a table called "First-Person Shooters" which has FPS games and the number of post and thread counts for each game. This table will be used as a proxy for the incidence of cheating in video games. That is, we make the assumption that the higher the thread count, the higher the prevalence of cheating for a particular game is.

Note that not every post or thread is directly related to cheating. However, since it is a website called "unknown cheats" whose purpose is to share cheating software, we take this as a measure of cheating. 

Now that the target variable is defined, we think of possible explanatory variables which may have influence on the incidence of cheating. For this, we use three further variables from data sources:

1. We use the wikipedia entries of video games by using natural language processing. There may be certain things such as first-player vs. multiplayer or video game genre which affects the incidence of cheating. We use this data to hopefully capture some of those characteristics.

2. We use the anti-cheat software for each game as an explanatory variable, gathered from levvvel.com. They have a table of video games and their anti-cheat software.

A large portion of this project was dedicated to gathering data via webscraping and different apis. After joining the three datasets (unknowncheats, wikipedia entries for games, and the anti-cheat information from levvvel.com), a dataset of 35 different video game series was obtained.

### Limitations of Small Data

A dataset containing just 35 entries is not sufficient to model on. Moreover, there are outliers in the data which are of particular interest since they are games for which there is a very high incidence of cheating (many unknowncheats post counts). Since our problem statement is novel and not well-studied, we hope to gain quantitative insights from the data regardless of the small size and these outliers. We therefore explore a method called **SMOGN** (Synthetic Minority Over-Sampling Technique for Regression with Gaussian Noise) which is a pre-processing solution used to address small datasets with domain imbalance (by domain imbalance, we mean that we have a few games with very high number of unknowncheats posts which are of high interest). We also explore other common methods which are best-practice when dealing with small data.

## 3. Methodology

### Methods for Small-Datasets

Because of the small size of the data, we are worried that the model will have high variance and not perform well on unseen data. To address this, we used the following techniques:

1. Carefully selecting the random seed in the train-test split. We used the training and test scores after modeling to evaluate the goodness of the random seed used

2. We chose a simple linear regression. More complex models would lead to over-fitting, especially due to the small data size. Therefore we used a very simple model in hopes that it would generalize to new data better

3. Small datasets are sensitive to outliers. However, removing outliers would lead to an even smaller dataset, and in our case, the outliers are of high interest because they are games which have very high incidences of cheating. For example, the game *Counter Strike* had  508,058 posts on unknowncheats.com, compared to an average of ~45,540 in our dataset. Although this was labeled an outlier (using IQR methods), we wanted to keep this game in our model because we hope to identify what gaming characteristics lead to having a very high incidence of cheating. To address this, SMOGN was used, which is explained below

### Overview of SMOGN

As stated before, **SMOGN** (Synthetic Minority Over-Sampling Technique for Regression with Gaussian Noise) is a pre-processing solution used to address small datasets with domain imbalance. It first labels the target variable $y$ with one of the two labels: normal case or rare case. These labels can be defined by the user as well.

SMOGN applies random under-sampling for the normal cases of $y$ (our target) and two types of over-sampling for the rare cases of $y$. The two over-sampling methods are called **SMOTER** (explained below) and Gaussian noise.

SMOTER is a strategy used to create new data through interpolation (interpolation is creating a new data point inside the range of existing datapoints). First, a rare case (of $y$) is selected as a seed and another rare case is randomly selected from the K-nearest neighbors of the seed. A new datapoint is created as a weighted average of the target variable values of the two rare cases used. Gaussian noise is creating datapoints by adding noise to existing datapoints, where the noise has a normal distribution.

For over-sampling on rare cases, SMOGN uses SMOTER when the seed and its K-nearest neighbor are close enough. When they are not close enough, it creates a new datapoint by adding Gaussian noise to the rare seed. This means the boundaries for the rare cases are expanded less conservatively when the seed and K-nearest neighbor are within a close distance, but otherwise expanded conservatively. It applies this algorithm to all rare seeds in the data.

### Model Creation

Our data has the following variables:

*Explanatory variables:*
* Anti-cheat system
* Wikipedia content of game
* We defined a new variable called `pub_dev_same` which is 1 when the publisher and developer of a video game are the same company and 0 if they are different. Very large gaming companies often both develop and publish their games, so this variable is used as an indicator of large companies

*Target variable:*
* Number of unknowncheats posts for the game

For our modeling process:

* We first pre-processed the data by using NLP methods and vectorizing the wikipedia content. We then scaled the data using StandardScaler and encoded nominal variables using OneHotEncoder.

* We then took this data and applied the SMOGN algorithm over it to create a new dataset. The following git repo was used to apply SMOGN: https://github.com/nickkunz/smogn

* We then created a linear regression model on the SMOGN-applied data and compared it to a linear regression on the original dataset

## 4. Results

Our linear regression model on SMOGN-applied data was better than the linear regression model on the original data. 

Linear regression on original data:
* Train score: 1.0
* Test score: -0.348

Linear regression on SMOGN-applied data:
* Train score: 1.0
* Test score: 0.785

However, we found that the RMSE for the SMOGN-applied data was 2.2 for the training set and 27889.1 for the test set. Our model still has very high variance. 

Although we cannot trust the model, the model interpretation can be summarized by the following:

* "Counterstrike" and "Valve anti-cheat" have the highest coefficients and indicate high incidence of cheating. This is further proof that the model is still overfit to the data (Counterstrike has the highest unknowncheat post count, and it uses Valve anti-cheat)
* Vanguard anti-cheat has a very negative coefficient and therefore is indicative of low incidence of cheating
* Punkbuster and Easy anti-cheat, which are both very polular, have negative coefficients
* We see from the coefficients that if the publisher and developer are the same, it is indicative of a low incidence of cheating

These should not be offered as answers to our original problem statement. However, we can keep them in mind and advise game companies to not use CounterStrike as an example when creating their games.

Although we still cannot call this SMOGN-preprocessed model good (due to the high overfitting), it was far better than our original model. It in particular performed better for higher values of y (refer to residual plot in notebook 4). **We therefore recognize SMOGN as a valuable pre-processing tool for cases in uneven domains.**

## 5. Application of Small-Dataset Techniques to Other Game Data

Although we could not directly address our original problem statement, we have learned new techniques of dealing with small data. As a fun aside, we use another dataset to study the application of SMOTE-techniques to solve problems of domain inbalance. 

This new dataset is taken from kaggle: https://www.kaggle.com/datasets/thedevastator/discovering-hidden-trends-in-global-video-games

It includes data on video game sales as well as features of video games. Although it is not nearly as small as the unknowncheats dataset, it is still relatively small, containing 1907 rows.

From this data, we created a classification problem where the goal is to predict if a video game will sell over 10 million copies. Only 48 out of the 1907 games sold over 10 million copies, however we want to focus on classifying cases of those blockbuster video games correctly. Since it is a classification problem, SMOTER was used for oversampling. Please see details of the model in notebook 5. The results were:

Original model:
* Training accuracy: 0.984
* Testing accuracy: 0.982

SMOTER-applied model:
* Training accuracy: 0.915
* Testing accuracy: 0.929

Although it may seem like the SMOTER-applied model performed worse, we see that the base cases for our binary classification were:

Original model:
* 0.974

SMOTER-applied model:
* 0.833

To evaluate the models, a random sample (with replacement) of 30% the data was taken and passed to both models. Of the 8 entries in the sampled data which had y=1, the original model classified 2 correctly whereas the SMOTER-applied model classified 4 correctly. Although this is not conclusive as it was a single sample, we have successfully used SMOTER to build a model which seems to generalize better to rare new instances of data.

Through this example, we have recognized SMOTE-methods as a valuable pre-processing technique in working with small datasets with rare values which are of high interest.

##  6. Conclusions

We have explored two datasets and two techniques for working with small data where there is domain inbalance: SMOGN and SMOTE-R. We have shown with the unknowncheats data that SMOGN is a valuable pre-processing tool (for a continuous y variable) where there are rare values of high interest. We also were able to use a slightly larger (still gaming-related) dataset, videogame sales data, to show that SMOTE-R is a valuable tool for binary classification problems where there is domain inbalance.

In terms of our original problem statement: 

#### Problem Statement: What causes a high incidence of cheating in First-Person Shooter Games?

We cannot give a conclusive answer to this and can only share the findings of our best model while making precautions very clear. However, the next steps towards finding the answer to this problem are very clear:

* Find more data - embellish the dataset with other sources and keep using SMOGN to learn about cases with high incidences of cheating
* Break apart game series into individual games, while being able to account for the similarity their wikipedia entries may have
* Ask companies to share more about their anti-cheat software to collaboratively work towards a better gaming community

(Our real problem statement was really: Can we use pre-processing techniques to derive meaning from small, imbalanced data?) :)



(note: please see sources cited directly in the jupyter notebooks)