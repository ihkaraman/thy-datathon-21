[![N|Solid](https://raw.githubusercontent.com/ihkaraman/ihkaraman/main/images/turkish_airlines-logo.png)](https://www.turkishairlines.com/)
# Travel Datathon 2021 
### Data Butchers - Second place winner

[![N|Solid](https://raw.githubusercontent.com/ihkaraman/ihkaraman/main/images/thy_datathon_presentation.JPG)](https://www.teknofest.org/duyurular-286.html/)

In Travel Datathon 2021, there were 3 different projects; The Next Destination Prediction, Flight Reason Prediction and Cargo Delay Prediction. Detailed solution approaches to these projects are presented below.

### 1. THE NEXT DESTINATION PROJECT

In this project, last 5 years flight data is used to suggest next destination alternatives for customers. Dataset contains 115 million entries for 36 million unique customers.
##### Data Preprocessing
In this step, miswritten city names are corrected, variable types are changed to minimize memory usage. Besideds of null vlaues, the rows that have same arrival and departure city are dropped. Also, ‘season’ feature is created by converting the departure time to season.
##### Modeling
Markov Model is used for solution approach, cities are modeled as states, flights are the transitions. 3 different transition matrices are used:
- General transition matrix contains all the flights of all the customers
- Seasonal transition matrix contains the flights of customers for seasons
- Individual transition matrix contains customer’s flights.

In the model, the matrices are used to determine probabilities for all cities for each customer to suggest new destinations. Weighted sum of the 3 matrices is taken and final probabilities are calculated. Weights are given to matrices based on the number of flights for each customer.

Based on this model, desired number of cities can be predicted accordingly. In transition matrices, all probabilities are calculated for all cities and the ones have the highest probabilities are chosen. 7 is set as the number of city suggestions for each customer. And, if a city is suggested, its alternative cities are filtered and not added to the suggestion list.

##### Results
Samples are taken from the validation dataset and metrics are calculated as 0.73 mean and 0.03 standard deviation. Accuracy is calculated as correct (takes 1) if the arrival city is in the suggested cities, if not false (takes 0).

### 2. FLIGHT REASON PROJECT REPORT

In this project, goal is to predict flight reason of the customers. Target variable has 4 classes: BUSINESS, LEISURE, SECOND HOME and STUDENT. There are two datasets available, first, PNR contains information about reservations and TICKET contains information about tickets. Also, an additional dataset, airport country pairs, is also used to add a new feature. 
##### Data Preprocessing
In the PNR dataset, there are 7656315 entries and if there is no information for the first flight, these rows are dropped. Duplicates, nulls and outliers are dropped and some of the rows have conflicting information, these rows are dropped as well. Correlation is calculated between features and some columns are dropped.
##### Feature Engineering
PNR and TICKET datasets are merged and new features are created. A wide range of and promising features are added to the dataset. For example, by using additional dataset binary features are added as if nationality of the ticket owner is the same as the departure country and the arrival country.

-pnr_create_month             : The month the PNR was created in  GMT time type.
-pnr_create_weekday	    : The weekday the PNR was created in  GMT time type.
-flight_month_1	    : The month of the first flight
-flight_weekday_1               : The weekday of the first flight
-flight_month_2	    : The month of the second flight
-flight_weekday_2	    : The weekday of the second flight
-flight_duration_1	    : The flight duration of the first flight
-flight_duration_2	    : The flight duration of the second flight
-arrival_interval_1               : Arrival intervals of first flight (05:00-11:00, 11:00-18:00, 18:00-05:00) 
-arrival_interval_2	    : Arrival intervals of second flight (05:00-11:00, 11:00-18:00, 18:00-05:00)
-diff_pnr_firstflight              : number of days between first flight and PNR creation date
-diff_second_firstflight	    : number of days between second flight and first flight
-pnr_workhour_interval     : Is PNR creation hour in workhours or not? (Workhours 08:00-18:00)
-num_of_flights                   : Number of flights for each PNR
-dep_nat_flag	    : if nationality of the ticket owner is the same as the departure country
-arr_nat_flag		    : if nationality of the ticket owner is the same as the arrival country

##### Modeling
After solving the class imbalance problem by random undersampling for target variable, data is split as train and test and a pipeline is created to convey all the data preprocessing steps. Numerical, categorical str and categorical int are the main categories for features in pipeline structure. If the data is naturally null like, if the ticket does not have a second flight, constant values are imputed on the other hand, if the data is null because missingness, median imputation (median is the most promising one among mean, mode, median imputation) is used.

22 different classifier models from sklearn library are applied and the best 5 of them are used for further improvements. In order to improve these scores, hyperparameter tuning procedure is operated. After some trials, voting and stacking classifiers are applied with the different combinations of best tuned classifiers. The best overall score is obtained from stacking classifier consisting of LGBM, SVC, Logistic Regression and Random Forest Classifier. 0.95 f1-score is obtained as the best score 

##### Model results:  Stacking Classifier

              precision    recall  f1-score   support
    accuracy       0.95      0.95      0.95       846


### 3. CARGO DELAY PROJECT REPORT

In this project, it is expected to predict whether there will be a delay in cargo flights with the dataset containing some cargo flights between 2017-2021. 
##### Data Preprocessing
Missing values, outliers, conflicts, nulls are detected and removed from the dataset. Correlation analysis is conducted and some columns are removed.
##### Feature Engineering 
New Features are created by using existing features:
Flight_duration: Difference in minutes between departure and arrival times
arr_month: Month of the arrival date
arr_weekday : Weekday of the arrival date
arr_day : Day of the month of the arrival date
expected_arrival_interval : interval number of the arrival number (interval distance is 30 mins)
expected_departure_interval : interval number of the departure number (interval distance is 30 mins)
##### Modeling
After solving the data imbalance problem, pipeline structure is constructed to apply all preprocessing steps. 22 different classification models from sklearn library are applied to choose top classifiers. The best 3 of them are chosen for further improvements based on f1-score.
In order to improve these scores, hyperparameter tuning procedure is operated for top classifiers and ensemble classifiers are constructed. Different variations of voting classifiers and stacking classifiers are used as more complex models and top scores are obtained with stacking classifier. Extra Tree Classifier, Random Forest and K-NN with their optimal parameters are used as estimators in the stacking classifier to get final scores, which is 0.80.

##### Cargo Delay Reason Analysis
After predicting whether there will be a delay or not, delay reasons are predicted as well. There are more than 350 reasons and nearly all of them occur rarely. Therefore, it is almost impossible to predict all the 350 labels with insufficient data and simpler methods should be adopted. 
When delay codes are checked, 93Z and 89Z are the majority codes in the data and corresponds to 65% in total. Remaining codes are gathered in ‘others’ category to simplify the modelling. Multi output classification is trained and 0.75 of f1-score is obtained in average. 
