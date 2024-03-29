# It's Elementary! : Modeling Global Primary School Completion Rates
#### A Linear regression model for primary completion rates

We were tasked with the challenge of selecting a topic to explore, finding data around that topic, and creating a linear regression model to predict some variable in the data. We focused on global education, deciding to attempt to model primary school completion rates. Education is, after all, the key to an individual's agency. With greater education comes a greater understanding of the world around you and a greater ability to try and solve the problems you see. Anyone can go into a foreign country and try and fix what they perceieve to be a problem, but by increasing the education level of citizens in that country, agency is placed in the hands of the people to improve what _they_ believe needs to be fixed. It was our hope that our model might shed light on what factors are important to primary school completion. 

We started by gathering our data and cleaning it, doing some exploratory data analysis to uncover important relationships our model would need to capture, and finally, created our model. Along the way we made some mathematical break throughs, sometimes questioning if we had broken all of math, but always resolving that math was not, in fact, broken! We'll discuss those fun break throughs here, in the hopes that others won't make the same mistakes that we did.

## Data: Sourcing and Cleaning
We used data from World Bank and UNICEF's 'State of the World's Children' report. We began by looking at the following variables:
- Child employment rates
- Proportion of GDP spent on education 
- Population density
- Urban population
- Proportion of agricultural land
- Adolescent birth rate
- Improved sanitation (percent of the population with access to UNICEF's take on "improved sanitation")
- Immunization (an average of different rates for different vaccines administered)
- Average support in learning from fathers (the proportion of fathers that take an active role in their childrens' education -- reading to them, singing with them, playing with them, etc.)
- Region
- Income relative to other countries (a categorical scale, according to World Bank)

Unfortunately, the data on child employment rates, proportion of GDP spend on education, and average support in learning from fathers were not complete enough to use or accurately fill in missing values, so they had to be dropped from the data set. 

Once we settled on the data we were using, we averaged values from 2000 to 2019 in our World Bank data, since UNICEF's data was an average of those years. Then, we took the  immunization rates in UNICEF's data (for polio vaccines, MMR, Hepetitis, etc.) and averaged those into one average immunization rate, since our domain knowledge was not extensive enough to understand the difference between each of these vaccines. Finally, we joined the World Bank and UNICEF data on country names, having to drop countries that weren't in both (there were not very many) and correct differences in names for the same country (such as "Bahamas, The" in World Bank data, versus just "Bahamas" in UNICEF data). 

## Exploratory Data Analysis
The distribution of our target variable was left skewed and looked like:
<p align='center'>
  Distribution of Target Variable
</p>
<p align='center'>
<img src='Images/target_distribution.png'>
</p>
We started by looking at a correlation matrix of each variable against primary school completion rates to see if there were any obvious variables to include in our model.

![Correlation Matrix](Images/First_Correlation_Matrix.png 'correlation matrix')

We started off with a linear regression model that included all of our variables. Our r-squared value was .693, which wasn't too bad, but our p-values for each feature were extremely variable. It was clear that this model was far too naive and we needed to dive into the nuances of the data.

![Our First Model](Images/First_Naive_Model.png 'first model with all variable')

Next, we looked at the effects of region, given the varying p-values we saw for each region's effect on the model. After looking at joint plots for all of our variables, colored by region, we noticed that most countries' performance weren't being determined by the region they were in. When we examined the distribution of income in each region, we realized that a more reliable predictor of the patterns we were seeing was country income.

![Income pairplot](Images/Income_Pairplot.png 'pairplot grouped by income')

So, in our next model we dropped the regional column. We saw a lower r-squared value, but the p-values improved quite a bit, leading us to believe that we improved our model overall, but still needed more predictive power.

![Second Model](Images/Second_Model.png 'second model without region')

We delved into the country income data, thinking that maybe there was an interaction term at play. Looking at urban population and relative country income's higher p-values, we figured there may be an interaction there that can improve our p-values. As it turned out, the relationship between urban population and primary school completion was only significant when the income group was low income (we used a confidence level of 95%; all other p-values were greater than or equal to .05).

![Income and Urban Population Interaction](Images/Income_Urban_Interaction.png 'motivation for low income and urban interaction term')
<p align='center'>
  Distribution of Income Groups
</p>
<p align='center'>
  <img src='Images/income_pie.png'>
</p>
We created an interaction term with low income x average urban population to extract this relationship. We also hypothesized that the agricultural land and immunization rates may interact, and observed that, when included in our final model, that interaction had a p-value of .032, and therefore was indeed significant.

![Final Model](Images/Final_Model.png 'Final Model with interaction terms')

We did it! We arrived at our final model, using urban population, agricultural land, adolescent birth rate, improved sanitation, immunization average, low income (categorical), low income x urban population, and agricultural land x immunization average as our independent variables. Unfortunately, though our residual plot tended to become less and less patterned as we refined our model, we still observed a somewhat substantial linear trend in our final residual plot. Though we tried transformations on the data that were skewed (primary school completion, for example, is right skewed, and so we used a log transformation to adjust for this), we were forced to accept that there were more complex transformations needed that we didn't have time to work out in our allotted 3 days, and so our residual plots were not as irregularly distributed as we would have hoped.

## A Mathematical Journey
Once we settled on our model, we standardized each independent variable so that we could compare the effects to see what features matter the most to primary school completion rates. We figured that standardizing the model would be simple enough and wouldn't change the p values, since the only things changing would be the scale of the variables, not the actual relationships they had with primary school completion. However, once we ran the model again, we found different p values! What was going on? As it turns out, we were calculating the interaction terms _after_ standardizing the variables. This effectively standardized the interaction term twice! In other (more mathy) words, if we let f be the scaling function, x be one independent variable, and y be the other independent variable, then f(xy) is what we were going for, but f(x)f(y) = f<sup>2</sup>(xy) is what we got! 

That was only one half of our journey for a deeper understanding of stats. The other half began when we were thinking about interaction terms and multicolinearity -- why is multicolinearity not a problem when you have interaction terms? Don't two variables, X and Y, perfectly predict the interaction XY? Earlier in the week, we had spent time talking about dropping one dummy column when representing a categorical variable in your model. If you have some category, C, with three different possibilities, and represent C with dummy variables, C<sub>1</sub>, C<sub>2</sub>, and C<sub>3</sub>, one of those dummy variables is redundant, and so you drop it in your model so that you don't have multicolinearity muddling your results. This is because C<sub>1</sub> + C<sub>2</sub> tells you exactly what C<sub>3</sub> will be -- if C<sub>1</sub> + C<sub>2</sub> = 1, then C<sub>3</sub> = 0, and if C<sub>1</sub> + C<sub>2</sub> = 0, then C<sub>3</sub> = 1. However, interaction terms are not the same. X + Y can tell you _something_ about XY, but it doesn't tell the whole story. Therefore, including all three variables -- X, Y, and XY -- doesn't include redundant information, and so you aren't dealing with an issue of multicolinearity. This understanding also gave us a glimpse into a more systematic way of uncovering good interaction terms (two dimensional interactions only) -- if you look at the correlation between X and XY and between Y and XY, you can determine how meaningful XY is by looking at the difference in those two correlations. The more different they are (perhaps the first is -.76 and the second is .47), the more 'interesting' their interaction is. This makes sense, because if one is .65 and the other is .7, then both X and Y increase with XY, and so XY isn't telling you much more than X and Y do on their own. 

Wow! Math is neat. 

## Final Conclusions
When we look at the coefficients of our final model (dependent variables were standardized so the coefficients would be directly comparable), we see that improved santitation, immunization rates, and having a larger proportion of agricultural land had the largest positive impact on primary school completion. On the other hand, being a low income country, having a high adolescent birth rate, and being both low income and having a high urban population all contribute the most negatively to primary school completion rate. 

## Next Steps
In the future, we'd would explore further interaction terms, as well as try to reduce the bias seen in the residual plot by adjusting for the skew in primary school completion rates. We would also like to explore ways to fill in missing data for the variables we had to drop, such as father support. One possible way to do this is sklearn's multivariate imputation. 
