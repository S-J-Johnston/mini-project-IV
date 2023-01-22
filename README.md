# Mini-project IV

### [Assignment](assignment.md)

## Project/Goals
The purpose of this project is to develop and deploy an automated loan approval model. The model should be able to assess loan applications based on prepoulated fields and provide a decision on whether to approve or deny a loan and be consistent with historical performance.

## Hypothesis
There are a number of features that are unavailable to us that would typically be used in loan approvals. Namely debt service coverage ratio, existing liabilities, Credit score, property valuation or other collateral and the business rules that most financial institutions have in place. In lieu of these the following features may have an impact on the outcome:
1. Applicatants with a credit history that meets specifications are more likely to be granted a loan. The pass specifcations are unknown to us but we can see whether or not it has passed.
2. The lower the ratio of the combined income to the loan amount divided by the term the more likely the loan is to be approved. 

## EDA 
1. Loan Amount and Applicant Income: As expected there is a long right hand tail to the distibution of both incomes and loan amounts.

    ![Alt text](images/Screenshot%202023-01-21%20at%2015.10.56.png)

2. There is a gender inbalance in the number of male and female applicants (4:1). The cause is not known but if this were to go to production the model should be built using a sample that is representative of the target market. Is our target market 80% male?

    ![Gender](images/Screenshot%202023-01-20%20at%2013.13.46.png)

3.  Approval Rate: Based on the sample data approvals account for 70% of all loan applications. Given that the sample size is limited at 614, this may cause some bias towards approvals over denials in our model. Resampling or increasing sample size may be required.

    ![Alt text](images/Screenshot%202023-01-20%20at%2013.14.15.png)


## Process
(fill in what you did during EDA, cleaning, feature engineering, modeling, deployment, testing)
### Feature Engineering:
- Applicant and Co-applicant income were combined to create combined_income. This was used throughout the analysis instead of the individual income parts.
- Monthly income: Estimated loan payment per month was calculated by dividing combined income by the loan payment divided by the term length. This was a vague attempt at quantifying the applicants abilty to repay their loan. This obviously does not take into account liabilities or interest rate.
- One Hot Encoding was generated for the following features:
    - Property Area
    - Dependents
    - Loan Amount Term
    - Gender
    - Education
    - Self Employed 
    - Credit History
    - Married
    - Loan Status 

### Cleaning
- Missing Values:
    - Missing values in features where there was a heavy skew towards a single category within them, were replaced with the mode. This was applied to "Married", "Gender", "Dependents" and "Self Employed". 
    - For missing values in "credit history" a different tact was taken as it was believed this was more influential in the final assessment and a greater proportion of missing values were present (8%). Forward fill was used in this case.
    - Missing values in loan amount and income were dropped as row is useless without them.
- Outlier Removal:
    - Income, loan amount and loan to income ratio were both skewed to the right due to folks who had applied for large loans or who had exceptionally high incomes as demostated in the image above. As the sample size is rather small the emphasis was on saving as many of the rows as possible rather than excluding anything that did not fit with the general population. Several reported incomes appeared to be exceptionally high (>30K a month), it was assumed that these incomes were mis-reported as annual incomes. After all, why would you apply for a 100k loan if you make 480K a year. These were identified and divided by 12 to move them to monthly values.
    Secondly, log transform was applied to combined income, loan amount and loan to income ratio to shift the distribution towards normal. After which outlier removal was applied using +- 1.5 x IQR. This romoved a combined toal of 83 rows. Initially the outlier removal was completed first then the logtransform but that exclued almost twice as many rows. The distrbution after the cleaning is shown below:

        ![Alt text](images/Screenshot%202023-01-21%20at%2015.19.04.png)

### Modelling and Results
GridSearchCV was used to test multiple feature reduction techniques and classification models with varying hyper parameters. The classifcation models tested were:
- Random Forest
- SVC
- K Nearest Neighbors
- Gradient Boosting

Feature reduct techniques were:
- PCA
- Select K Best

For details on the range of hyperparameters tested refer to the notebook.
Without feature reduction the best performing model was SVC (C=1.0). The best test accuracy achieved was 80%. Confusion matrix is shown below:

![Alt text](images/Screenshot%202023-01-21%20at%2017.12.10.png)

With feature reduction it was possible to achieve a max accuracy of 84% with K Best using SVC (C=1.0) and k=3. Confusion matrix is below.

![Alt text](images/Screenshot%202023-01-21%20at%2017.17.21.png)

In both cases the model performs well at classifying True Positives but has difficulty distinguishing True Negatives. This may be related to the class imbalance on declined applications.

### Deployment
Unfortuantely it was not possible to get the model deployed before the project deadline. In the coming days I will try to get this completed and update the report as required.

## Demo
TBD

## Challanges 
1. There were some time constraints that prevented me from getting the model deployed on time.

## Future Goals
1. When carrying out the model development what I was attempting to do was run a GridSearchCV using both a range or feature reduction techniques and a range of classifiers with their respective ranges of hyperparameters. I was not able to get them to work within the same grid. Instead I picked the best performing feature reduction technique and then used those hyperparameters with a range of classifiers and their ranges of hyper parameters. In short it would have been nice to get that to run as a single block of code.
2. Deployment... it was unfortunate I could not get it completed and will have to go back to it at a later point.