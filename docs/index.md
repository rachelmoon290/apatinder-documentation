# APA Tinder for Dogs Web Application

## About the project

### 1. Introduction


### 2. Problem Statement
Every day at Austin Pets Alive!, there are about 40 people who visit the shelter to look for animals to adopt. However, there are only maximum 4 matchmakers on site each day, and it takes about 30 minutes to an hour for a matchmaker to talk to each visitor and match him/her with the best dogs. As a result, the average waiting time is about 53 minutes per visitor, and consequently about 26% of on-site visitors cancel or do not show up to their appointment, due to this bottleneck effect.

APA Tinder for Dogs is a web application that enables adopters

This project is part of Harvard's AC297r Capstone Project course, partnered with APA!.

### 3. Web Architecture



## Installation and setup (MacOS)
Please follow these steps below to install and set up all the necessary components to run our web application in a production environment.

### 1. MySQL Installation
1. **Download MySQL:** If MySQL is not already installed on your computer, download MySQL Community Server from this link: [https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/).

2. **Install MySQL.** Use "Legacy Password Encryption" option, for version compatibility.

3. **Initialize database and start server:** In System Preferences > MySQL, click on "Initialize Database." Set a password (this will be your password for the default root user), and use "Legacy Password Encryption." When it is complete, press "Start MySQL Server."

4. **Log into MySQL:** In terminal, type `mysql -u root -p`. Then type the password you set during database initialization (Step 3). Then you will be able to log into MySQL in the terminal.

5. (Optional) **Create another database user:** In MySQL type `GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';`. Replace `username` and `password` with the user information you want to create.

6. **Create database:** In MySQL, type `CREATE DATABASE apa_tinder;`. `apa_tinder` is the name of the database. Then type `USE apa_tinder;` to start using `apa_tinder` database.

7. **Import sql files into MySQL:** We used sql dump file from APA! to build and test our web application. To import the dump file into your local MySQL, in the terminal (not in MySQL) type `mysql -u root -p apa_tinder < /Users/directory/to/dumpfile/apadump.sql`. Use the directory path for your sql dump file. To easily get the directory path information, you can drag and drop your sql dump file into the terminal. Next, import `super_dog_table.sql` to your MySQL, using the same procedure.

### 2. Model Backend setup
1. Create `.env` file: Navigate to parent `src` folder. In terminal, type `touch .env`. In the `.env` file, populate the following variables with values. For example, for development and testing purposes, `DB_ENDPOINT=localhost`.
```
DB_ENDPOINT=
DB_USERNAME=
DB_PASSWORD=
JWT_TOKEN= # a private key. can be any string
```

2. Download required packages: Navigate to `src/model-server` folder. In terminal run `pip install -r requirements.txt`.

3. In terminal, type `./run.sh`. If the backend runs successfully, you will see `Running on localhost:xxxx` from the resulting message. In your web browser, navigate to `localhost:xxxx` to view a list of model backend REST API endpoints.

### 3. Web Frontend and Backend setup
1. Install yarn: in terminal, type `brew install yarn`.
2. Next, you need to run the web backend server (GraphQL). Navigate to `src/apa-match-server`. In terminal, run `yarn install`, and then run `yarn run dev`. Confirm that you get the message "listening on port yyyy."
3. Now you will be running the web frontend. Navigate to `src/apa-match-client`. In terminal, run `yarn install`, and then run `yarn start`. Check the port number (zzzz) from the message. In the web browser, navigate to `localhost:zzzz` to view the web application.

### 4. Jupyter notebook demo setup
1. Download required packages: Navigate to `notebooks` folder. In terminal run `pip install -r requirements.txt`. If theano in pymc3 package causes problems, try `conda install theano pygpu`, or follow instructions in http://deeplearning.net/software/theano_versions/dev/install_macos.html.


## Web Interface
The web serves two types of users: the adopters and the matchmakers (admin). Below we explain the current functionalities of our web application.

### User journey: adopters

![](img/userjourney_adopter.png)

When the users sign up, they will be asked to fill out several questions about their basic living environment information and animal preferences, and whether they are interested in fostering.

In the main home page, there is a featured dog, who is randomly selected from the 50 longest-stay dogs currently in the shelter. The user can add the dog to my favorites lists.

In the match page, the user can start getting matched to dogs. The recommendation model predicts the ranking for all available dogs in the shelter that match with the user's living environment conditions, and then gives 15 dogs as results. Out of 15 dogs, 10 dogs are the top ranked dogs, whereas the other 5 dogs are top ranked dogs from the long-stay dogs, so that even the long-stay dogs also have a chance to be shown to the users. After 15 swipes, the model then re-predicts the rankings on dogs not yet swiped, also incorporating previous user swipe behavior information. Pressing details button opens a new tab that shows dog details in the Austin Pets Alive! website. The favorited dogs list can be seen in the user's favorites page.

### User journey: matchmakers

![](img/userjourney_matchmaker.png)

The matchmakers can view the list of users who have signed up and used our application. They can also view specific user details, such as the user's environment condition, animal adoption preferences, and their favorited dog list. In the favorited dog list, the matchmakers are able to see the NLP generated summary of the dogs, and can also edit the summary if they wish.  


## Dog Recommendation Model
### Data Cleaning
Our team was given sql data from APA! that contains information about all dogs (~30,000 dogs) in the shelter. There are 147 columns (147 different features for dogs) in the data, but the majority of columns was filled with NaN values or did not seem to be important for our model. We decided to drop columns where more than 60% of their values consist of NaNs and drop columns not relevant to our model, such as 'AnimalCoverPhoto', 'Location', 'AnimalStatus', etc. We were left with 21 columns, as shown below.


1. AnimalSex
2. AnimalCurrentWeightPounds
3. AnimalAltered
4. AnimalAgeDays
5. AnimalBreed
6. AnimalColor
7. AgeAtIntake
8. LastEventType
9. FirstIntakeEventType
10. LastIntakeEventType
11. CurrentVolunteerCategory
12. CurrentBehaviorCategory
13. IsDistemper
14. IsDistemperWatch
15. IsMedicalConsult
16. IsCGC
17. IsTopDog
18. IsHWPos
19. IsHWTreatment
20. IsBehaviorConsult
21. DaysInShelter

We cleaned each column to make it ready for modeling. While the majority of columns needed minor changes like dropping few NaN values, we spent significant amount of time inspecting 'AnimalColor' and 'AnimalBreed' columns, since they are important features when determining the popularity of dogs.


### Feature Engineering

#### AnimalColor column
There were 292 unique animal colors for around 30,000 dogs in our data, where animal colors such as Blue/Black, Blue/Gold, and Blue/Grey were classified as different colors. We decided to choose top 10 popular colors, Black, Brown, Tan, White, Cream, Blue, Blond, Red, Grey, and Yellow, and grouped all dogs into those 10 colors.

Instead of doing one-hot encoding for 10 different colors, we manually created 10 new columns indicating whether each dog contains each color - IsBlack, IsBrown, IsTan, IsWhite, IsCream, IsBlue, IsBlond, IsRed, IsGrey, and IsYellow. For example, if the color of a dog is Black/Brown, it will have values 1 for IsBlack and IsBrown columns, and 0 for the other columns. This approach is better than one-hot encoding, since one-hot encoding allows only one input to have a value 1. If we used one-hot encoding, this dog would have had a value 1 for only IsBlack or IsBrown column and 0 for the rest of columns, so it is not an accurate representation of our data.  


#### AnimalBreed column
There were 1969 unique animal breeds for around 30,000 dogs in our data, where animal breeds such as Beagle/Bulldog, Beagle/Dachshund, and Beagle/Hound were classified as different breeds. We decided to choose top 40 popular breeds obtained from APA! website as different breeds.

Instead of doing one-hot encoding for 40 different breeds, we manually created 40 new columns indicating whether each dog contains each breed - IsDachshund, IsBeagle, IsBulldog, etc. For example if the breed of a dog is Beagle/Bulldog, it will have values 1 for IsBeagle and IsBulldog columns, and 0 for the other columns. Again, if we used one-hot encoding, this dog would have had a value 1 for only IsBeagle or IsBulldog column and 0 for the rest of columns, so it is not an accurate representation of our data.  

For other categorical columns like Animalsex, IsDistemper, IsCGC, etc, we performed one-hot encoding to make them ready for modeling. For continuous variables, we standardized them since different variables were at different scales.

### 1. Dog Popularity Model (Prior Model)

The response variable for our analysis is 'DaysInShelter'. The goal of the prior model is to figure out the best model that predicts 'DaysInShelter' and deliever the best set of coefficients of predictor variables to the Bayesian Posterior Model, so that it can start to be updated once user swipes into our system.

In order to find the best model, we have tried linear regression without polynomial terms, linear regression with polynomial degree 3 for 'AgeAtIntake', lassoCV with polynomial degree 3 for 'AgeAtIntake', ridgeCV with polynomial degree 3 for 'AgeAtIntake', and random forest using gridsearchCV. Expecting that 'AgeAtIntake' has a positive correlation with 'DaysInShelter', we would like to find out a non-linear relationship between 'AgeAtIntake' and 'DaysInShelter', by taking polynomial degree 3 for 'AgeAtIntake'.

Random forest using gridsearchCV produced the best test set $R^2$ score, followed by ridgeCV with polynomial degree 3 for 'AgeAtIntake'. We decided to use ridgeCV, since ridge regression provides values of coefficients that can be used as coefficients for the posterior model and the performance of the ridge regression model was nearly as good as the random forest model.



### 2. User Swipes Model (Posterior Model)





## Summary
In conclusion, we have a working web architecture that performs dog recommendations for potential adopters. Our web application will help improve dog adoption rate by recommending the right dogs that the adopters are more likely to adopt. Furthermore, our application helps publicize even the dogs that have stayed long in the shelter by featuring them on the main page, as well as matching users with long-stay dogs with highest ranking.

If the shelter visitors actually start using our matching application, it will reduce the time the matchmakers have to spend with each person, and also keep the visitors engaged while waiting for their appointment. We hope that our web application will consequentially help reduce the visitor's appointment cancellation rate.

## Future Work
Now that we have a working web application architecture, the next step is to collect real user swipes data and get feedback from the app users in order to evaluate and improve recommendation model performance. We can conduct testing such as A/B testing in order to determine whether our model results in higher dog adoption rate than random selection model. Furthermore, we plan to continue developing the application to suit APA!'s needs and work on the application rollout.

## About Us
We are a group of Masters and PhD candidate students from Harvard University School of Engineering and Applied Sciences (SEAS). This project is part of AC297r: Capstone project class, and we partnered with Austin Pets Alive! organization to work on this project.

## Acknowledgements
We would like to give special thanks to our capstone project instructors Dr. David Sondak and Patrick Ohiomoba, who guided our projects from beginning to end and were always super supportive. We also greatly thank Steve Porter, Director of Lifesaving Operations at Austin Pets Alive!, for partnering with our team and being very enthusiastic about our project.
