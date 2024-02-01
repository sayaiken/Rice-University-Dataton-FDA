# Rice University Dataton FDA Winnig Submission 

![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/rice_aiken8.png)


# Data Cleaning

## “Ptient Age”
We quickly see that not all age units are in years (Image 1) so all age values are transformed to actual years, according to what the age_unit is. Once this is done, age over 100 was explored. The range was between 100 -~900. Observations with age over 100 were removed. This is because there were very few observations over 100 (n = ~27) and we assume that the effect of age on health outcomes probably plateaus after a certain age. A value of 100 seems sensible given this hypothesis. 
## Image 1
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/Rice_Aiken1.png)

Caption: "Frequency Count of age_unitss"



Image 2 shows that kids 5 and under make up a notable percentage of the population just on their own. However, they still make up a small percentage of the population. Similar to those who are very old,  health effects are probably different for those who are young. Thus, we remove all patients under 5 years of age and use them for a separate analysis. This also makes our statistical analysis more interpretable since effects would not be considered for those groups that may have unique, group-level, effects. These group level effects could be accounted for with a multilevel analysis. However, such implementations for multinomial variables do not exist unless you are willing to use a Bayesian approach (which none of the team members are familiar with). 


## Image 2
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/Aiken_rice2.png)

Caption: "Age Distribution After Pre-processing"

## “Description”
Creating a lookup table of Description, the product categories, shows that some categories repeat. This is further emphasized by the fact that these categories have the same exact number of observations (the ‘count’ column). For example, consider “Fruit/Fruit Prod” in Image 3.  We see this category is repeated 3 times with the same exact count of observations, 1121. For all such cases the duplicates are removed.





## Image 3
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/aiken_rice3.png)

Caption: “Product” Values with Frequency Count

## “SEX”
Examining the values inside “sex” shows 1693 empty observations. These might as well be “Not Reported” along with “unknown.” Clearly, gender is not balanced (Image 4). We also decide to remove all “Not Reported” observations since we want to account for the effect of gender in our model. Furthermore, we observe that Females outpurchase males and that 2 products lead the number of reports and most of the reports are by women (see Image 5)
 
## Image 4
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/aiken_rice4.png)

Caption: "Provided Gender Values"







## Image 5
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/aiken_rice5.png)

Caption: "Product Category given Gender"

## Dates
The competition document states that we will analyze FDA from 2003 and onwards. However, our dataset contains observations before that date. We extracted the year from the date input and removed all observations before that date. After cleaning the dates, we plot the number of reports for each year. We clearly see a peak in 2021 (see Image 6). Unsurprisingly, the products leading these reports in 2021 are “Cosmetics” (n = 18347) and “Vit/Min/Prot/Unconv Diet(Human/Animal)” (n = 3560).







## Figure 6
![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/aiken_rice6.png)

Caption: "Number of Reports for Each Year"


## Repeated Measures Data
We notice that we have multiple inputs that belong to the same user. This is due to having SUSPECT and CONCOMITANT observations for the same person. However, this repeated measurements for the same user violates the independence assumption for many statistical models. Therefore, we decided to simply flag each user that had a CONCOMITANT report as well. This, of course, is not as detailed as modeling each user individually, by account, for each CONCOMITANT report that belongs to each user (also known as mixed effects modeling). But it gives us the option to still account for the effect of CONCOMITANT drugs in our analysis. For our analysis, this means we can use simpler models that have better interpretation and converge much faster. 
## Outcome Variable Transformation
The dataset we work with provides two columns that describe the specific outcome of the adverse effect for each product: (1) CASE_MEDDRA_PREFERRED_TERMS, which denotes the specific medical symptoms/afflictions experienced by the consumer, and (2) CASE_OUTCOME, which denote generic outcomes for the consumer. We first notice that for both features, a given product can assume one or more of the labels. We also notice that the CASE_OUTCOME feature is significantly imbalanced, with the label “Other Serious or Important Medical Event” representing more than half of the observations. Since the meaning of this majority label is ambiguous, we decide to use the specific medical labels provided in CASE_MEDDRA_PREFERRED_TERMS to facilitate a more explainable analysis.
Since the outcome features describe a multi-label problem, we decide to re-bin the combinations of labels so that the feature space has a smaller dimension. To achieve this dimensionality reduction, we implement a transformer-based pre-trained sentence embedding model, MiniLM, to embed and aggregate the values for the CASE_MEDDRA_PREFERRED_TERMS feature. The sentence embedding model enables a transformation from high-dimensional text space to a 384-dimension real vector space, encoding the semantic meaning of each medical symptom/affliction. The transformed feature, “pref_med_emb”, is the mean of the embeddings of the individual labels, and numerically represents the effects that a given product had on a given user. Using the K-Means algorithm, we cluster these effects into seven categories, represented by the “pref_med_cluster” feature. These categories will be used in the multinomial logistic regression analysis.
# Analyses
## Logistic Regression
We wanted to have interpretable results so we could make specific suggestions. To this end, we decided to use a Multinomial Logistic Regression since it would provide us parameters that we could then use to understand how age, gender, concomitant reports, and product type are associated with the likelihood of a specific health outcome. The outcome variables for our model were the resulting cluster from our text embedding clustering. See section “Outcome Variable.” The model specification is:
Y ~ Sex*DESCRIPTION + age_in_years + concomitant.
 
# Recommendations
## Exemption 4 Products
First, we note that many products were censored from the dataset, appearing only as “Exemption 4.” From the DESCRIPTION feature, we surmise that [...]. For future discussion, we ignore the Exemption 4 products and focus on the specific brand names and products for our recommendations.
## Johnson’s Baby Powder
Johnson’s Baby Powder by the statictic causes cancer especially infants
## Peanut Butter
Peanut butter is a common cause of allergy reaction
## Vitamins
A wide variety of vitamins and minerals from various sellers [...]. In the “CHOKING” cluster, vitamins make up about half (50.1%) of the products. Vitamins, minerals and other supplements often take the form of small capsules, so this observation is . We recommend the FDA encourage sellers to place greater emphasis on the choking hazards that their products inherently have.

# Kratom
Kratom is an herbal supplement that is used by some to alleviate pain and provide an energy boost [1]. The supplement is not regulated by the FDA. Our investigation shows that, upon subtracting out the Exemption 4-listed products, Kratom is the most frequently occuring product (18.6%) in our engineered “DEATH” outcome label.


![alt text](https://github.com/sayaiken/Rice-University-Dataton-FDA/blob/main/aiken_rice7.webp)

