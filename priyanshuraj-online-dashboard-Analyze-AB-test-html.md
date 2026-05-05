## Analyze A/B Test Results [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Analyze-A/B-Test-Results)

This project will assure you have mastered the subjects covered in the statistics lessons. The hope is to have this project be as comprehensive of these topics as possible. Good luck!

## Table of Contents [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Table-of-Contents)

- [Introduction](https://priyanshuraj.online/dashboard/Analyze_AB_test.html#intro)
- [Part I - Probability](https://priyanshuraj.online/dashboard/Analyze_AB_test.html#probability)
- [Part II - A/B Test](https://priyanshuraj.online/dashboard/Analyze_AB_test.html#ab_test)
- [Part III - Regression](https://priyanshuraj.online/dashboard/Analyze_AB_test.html#regression)

### Introduction [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Introduction)

A/B tests are very commonly performed by data analysts and data scientists. It is important that you get some practice working with the difficulties of these

For this project, you will be working to understand the results of an A/B test run by an e-commerce website. Your goal is to work through this notebook to help the company understand if they should implement the new page, keep the old page, or perhaps run the experiment longer to make their decision.

**As you work through this notebook, follow along in the classroom and answer the corresponding quiz questions associated with each question.** The labels for each classroom concept are provided for each question. This will assure you are on the right track as you work through the project, and you can feel more confident in your final submission meeting the criteria. As a final check, assure you meet all the criteria on the [RUBRIC](https://review.udacity.com/#!/projects/37e27304-ad47-4eb0-a1ab-8c12f60e43d0/rubric).

#### Part I - Probability [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Part-I---Probability)

To get started, let's import our libraries.

In \[1\]:

```
import pandas as pd
import numpy as np
import random
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
sns.set_theme()
#We are setting the seed to assure you get the same answers on quizzes as we set up
random.seed(42)
```

`1.` Now, read in the `ab_data.csv` data. Store it in `df`. **Use your dataframe to answer the questions in Quiz 1 of the classroom.**

a. Read in the dataset and take a look at the top few rows here:

In \[2\]:

```
df = pd.read_csv('Data/ab_data.csv')
df.head()
```

Out\[2\]:

|  | user\_id | timestamp | group | landing\_page | converted |
| --- | --- | --- | --- | --- | --- |
| 0 | 851104 | 2017-01-21 22:11:48.556739 | control | old\_page | 0 |
| 1 | 804228 | 2017-01-12 08:01:45.159739 | control | old\_page | 0 |
| 2 | 661590 | 2017-01-11 16:55:06.154213 | treatment | new\_page | 0 |
| 3 | 853541 | 2017-01-08 18:28:03.143765 | treatment | new\_page | 0 |
| 4 | 864975 | 2017-01-21 01:52:26.210827 | control | old\_page | 1 |

b. Use the below cell to find the number of rows in the dataset.

In \[3\]:

```
df.shape[0]
```

Out\[3\]:

```
294478
```

c. The number of unique users in the dataset.

In \[4\]:

```
df.user_id.nunique()
```

Out\[4\]:

```
290584
```

d. The proportion of users converted.

In \[5\]:

```
# Proportion converted - this works for 0s and 1s
df.converted.mean()
```

Out\[5\]:

```
0.11965919355605512
```

In \[6\]:

```
# # Proportion not converted - this works for 0s and 1s
# 1 - df.converted.mean()
```

In \[7\]:

```
# Another way to calculate (understanding formula "df.converted.mean()")
# Formula below is useful if column contain string or boolean value.
not_converted = df.query('converted == 0').user_id.count() #we could use nunique() insted of count
converted = df.query('converted == 1').user_id.count()   #we could use nunique() insted of count
proportion_converted = converted/df.shape[0]
not_converted, converted, proportion_converted
```

Out\[7\]:

```
(259241, 35237, 0.11965919355605512)
```

e. The number of times the `new_page` and `treatment` don't line up.

In \[8\]:

```
# Treatment doesn't align with the new page
no_aligment1 = df.query('group == "treatment" and landing_page == "old_page"').count().timestamp #Not align
# New page doesn't align with the treatment
no_aligment2 = df.query('group == "control" and landing_page == "new_page"').count().timestamp #Not align
# Check how many times do align
aligment3 = df.query('group == "treatment" and landing_page == "new_page"').count().timestamp #Align

no_aligment = no_aligment1 + no_aligment2
no_aligment
```

Out\[8\]:

```
3893
```

f. Do any of the rows have missing values?

In \[9\]:

```
df.isnull().sum()
```

Out\[9\]:

```
user_id         0
timestamp       0
group           0
landing_page    0
converted       0
dtype: int64
```

`2.` For the rows where **treatment** is not aligned with **new\_page** or **control** is not aligned with **old\_page**, we cannot be sure if this row truly received the new or old page. Use **Quiz 2** in the classroom to provide how we should handle these rows.

a. Now use the answer to the quiz to create a new dataset that meets the specifications from the quiz. Store your new dataframe in **df2**.

In \[10\]:

```
# Drop first condition
df_drop1 = df.drop(df[(df['group'] == 'treatment') & (df['landing_page'] == 'old_page')].index)
# Drop second condition
df2 = df_drop1.drop(df_drop1[(df_drop1['group'] == 'control') & (df_drop1['landing_page'] == 'new_page')].index)
```

In \[11\]:

```
# Double Check all of the correct rows were removed - this should be 0
df2[((df2['group'] == 'treatment') == (df2['landing_page'] == 'new_page')) == False].shape[0]
```

Out\[11\]:

```
0
```

`3.` Use **df2** and the cells below to answer questions for **Quiz3** in the classroom.

a. How many unique **user\_id** s are in **df2**?

In \[12\]:

```
df2.user_id.nunique()
```

Out\[12\]:

```
290584
```

b. There is one **user\_id** repeated in **df2**. What is it?

In \[13\]:

```
# How many duplicates in the dataset
df2.user_id.duplicated().sum()
```

Out\[13\]:

```
1
```

In \[14\]:

```
# Display the duplicates (displays both duplicates)
df2[df2.user_id.duplicated(keep=False)]
```

Out\[14\]:

|  | user\_id | timestamp | group | landing\_page | converted |
| --- | --- | --- | --- | --- | --- |
| 1899 | 773192 | 2017-01-09 05:37:58.781806 | treatment | new\_page | 0 |
| 2893 | 773192 | 2017-01-14 02:55:59.590927 | treatment | new\_page | 0 |

c. What is the row information for the repeat **user\_id**?

In \[15\]:

```
# Displays only one duplicate
# duplicate = df2[df2.user_id.duplicated(keep='first')]
# duplicate = df2[df2.user_id.duplicated(keep='last')]
duplicate = df2[df2.user_id.duplicated()]
duplicate
```

Out\[15\]:

|  | user\_id | timestamp | group | landing\_page | converted |
| --- | --- | --- | --- | --- | --- |
| 2893 | 773192 | 2017-01-14 02:55:59.590927 | treatment | new\_page | 0 |

In \[16\]:

```
# Check shape before droping the row
df2.shape
```

Out\[16\]:

```
(290585, 5)
```

d. Remove **one** of the rows with a duplicate **user\_id**, but keep your dataframe as **df2**.

In \[17\]:

```
# Drop the row by index
df2 = df2.drop([2893])
```

In \[18\]:

```
# Check if the drop is successful
df2.shape
```

Out\[18\]:

```
(290584, 5)
```

`4.` Use **df2** in the below cells to answer the quiz questions related to **Quiz 4** in the classroom.

a. What is the probability of an individual converting regardless of the page they receive?

In \[19\]:

```
converting_prob = df.converted.mean()
converting_prob
```

Out\[19\]:

```
0.11965919355605512
```

b. Given that an individual was in the `control` group, what is the probability they converted?

In \[20\]:

```
control_con_prob = df2.query('group == "control"').converted.mean()
control_con_prob
```

Out\[20\]:

```
0.1203863045004612
```

In \[21\]:

```
# This is the calculation to understand formula above and/or in case the converted is a boolean or other non-numerical value.
control_con_prob2 = df2.query('group == "control" & converted == 1 ').user_id.nunique() / df2.query('group == "control"').user_id.nunique()
control_con_prob2
```

Out\[21\]:

```
0.1203863045004612
```

c. Given that an individual was in the `treatment` group, what is the probability they converted?

In \[22\]:

```
treat_con_prob = df2.query('group == "treatment"').converted.mean()
treat_con_prob
```

Out\[22\]:

```
0.11880806551510564
```

d. What is the probability that an individual received the new page?

In \[23\]:

```
new_page_prob =  df2.query('landing_page == "new_page"').count().user_id / df2.shape[0]
new_page_prob
```

Out\[23\]:

```
0.5000619442226688
```

e. Consider your results from a. through d. above, and explain below whether you think there is sufficient evidence to say that the new treatment page leads to more conversions.

> **_Answer_**
>
> _There is an equal chance to get either a new page or old page: P(old) = P(new) = 0.5 = 50%. The probability to convert given an old page or a new page is the same, that is 0.12 or 12% (this probability is calculated from the data we have). We can calculate Bayes Rule posterior probability and get the result for both 0.5 or 50% (P(CON\|New\_Page) = 0.06/0.012 = 0.5 & P(CON\|Old\_Page) = 0.06/0.012 = 0.5). Based on these calculations we cannot say that there is sufficient evidence that a new treatment page leads to more conversions._

### Part II - A/B Test [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Part-II---A/B-Test)

Notice that because of the time stamp associated with each event, you could technically run a hypothesis test continuously as each observation was observed.

However, then the hard question is do you stop as soon as one page is considered significantly better than another or does it need to happen consistently for a certain amount of time? How long do you run to render a decision that neither page is better than another?

These questions are the difficult parts associated with A/B tests in general.

`1.` For now, consider you need to make the decision just based on all the data provided. If you want to assume that the old page is better unless the new page proves to be definitely better at a Type I error rate of 5%, what should your null and alternative hypotheses be? You can state your hypothesis in terms of words or in terms of **poldpold** and **pnewpnew**, which are the converted rates for the old and new pages.

> **_RESEARCH QUESTION: Does the experiment page drive higher traffic than the control page?_**
>
> **_H0H0: The new version of a page draws the same amount or less traffic than the old version of a page (new version is equal or worse than the old)._**
>
> **_H1H1: The new version of a page draws more traffic than the old version of a page (new version is better than the old version)._**

H0:pnew−pold≤0H0:pnew−pold≤0H1:pnew−pold>0H1:pnew−pold>0

> _pnewpnew and poldpold are the values for the old page and the new page, respectively._

`2.` Assume under the null hypothesis, pnewpnew and poldpold both have "true" success rates equal to the **converted** success rate regardless of page - that is pnewpnew and poldpold are equal. Furthermore, assume they are equal to the **converted** rate in **ab\_data.csv** regardless of the page.

Use a sample size for each page equal to the ones in **ab\_data.csv**. **_No sample size needed, we are using the whole ab\_data.csv dataset._**

Perform the sampling distribution for the difference in **converted** between the two pages over 10,000 iterations of calculating an estimate from the null.

Use the cells below to provide the necessary parts of this simulation. If this doesn't make complete sense right now, don't worry - you are going to work through the problems below to complete this problem. You can use **Quiz 5** in the classroom to make sure you are on the right track.

a. What is the **convert rate** for pnewpnew under the null?

In \[24\]:

```
p_new = df.converted.mean()
p_new
```

Out\[24\]:

```
0.11965919355605512
```

b. What is the **convert rate** for poldpold under the null?

In \[25\]:

```
p_old = df.converted.mean()
p_old
```

Out\[25\]:

```
0.11965919355605512
```

> **_Note_**
>
> _We assume that under the null hypothesis, p\_new and p\_old both have "true" success rates and therefore are equal to the converted success rate regardless of page - that is p\_new and p\_old are equal. Since they are both equal, we don't need to split into treatment types and consider all conversions together. Because we are using 0s and 1s to confirm conversion, it's possible to take the mean of this to find the rate (source: Udacity Knowledge FAQ)._

c. What is nnewnnew?

In \[26\]:

```
n_new = df2.query('landing_page == "new_page"').shape[0]
n_new
```

Out\[26\]:

```
145310
```

d. What is noldnold?

In \[27\]:

```
n_old = df2.query('landing_page == "old_page"').shape[0]
n_old
```

Out\[27\]:

```
145274
```

e. Simulate nnewnnew transactions with a convert rate of pnewpnew under the null. Store these nnewnnew 1's and 0's in **new\_page\_converted**.

In \[28\]:

```
new_page_converted = np.random.binomial(1, p_new, n_new)
new_page_converted
```

Out\[28\]:

```
array([0, 0, 0, ..., 0, 0, 1])
```

f. Simulate noldnold transactions with a convert rate of poldpold under the null. Store these noldnold 1's and 0's in **old\_page\_converted**.

In \[29\]:

```
old_page_converted = np.random.binomial(1, p_old, n_old)
old_page_converted
```

Out\[29\]:

```
array([0, 0, 0, ..., 0, 0, 0])
```

> **_Note_**
>
> _Here we stimulate the sample with the np.random.binomial method_
>
> _WHY: We stimulate this under the null hypothesis, to see how the mean of distribution looks like if it came from the null hypothesis. Then we calculate p-value (from actual) in order to reject or fail to reject the null hypothesis._
>
> _This is singular example for cell h where we stimulate for 10000 samples._
>
> _1 = trial size (0s and 1s)_
>
> _p\_new = probability of trial (calculated)_
>
> _n\_new = number of trials to run_

_because we are storing the value in form of 0s and 1s we use n=1: [np.random.binomial](https://stackoverflow.com/questions/27644617/difference-between-n-and-size-parameters-in-np-random-binomialn-p-size-1000)_

g. Find pnewpnew \- poldpold for your simulated values from part (e) and (f).

In \[30\]:

```
#This is the the stimulated mean difference under the null hypotesis.
p_diffs1 = new_page_converted.mean() - old_page_converted.mean()
p_diffs1
```

Out\[30\]:

```
-0.0013168229053714814
```

In \[31\]:

```
# This is the difference in acctual data (observed sample)
pdiff_actual = df2.query('group == "treatment"').converted.mean() - df2.query('group == "control"').converted.mean()
pdiff_actual
```

Out\[31\]:

```
-0.0015782389853555567
```

> **_Note_**
>
> _Now that we know the observed difference in this sample (dataset in our case), we have to see if this difference is significant and not just due to chance. Therefore, we will simulate 10,000 values and calculate the differences in proportions (pnewpnew \- poldpold)._

h. Simulate 10,000 pnewpnew \- poldpold values using this same process similarly to the one you calculated in parts **a. through g.** above. Store all 10,000 values in a numpy array called **p\_diffs**.

In \[32\]:

```
p_diffs = []
# No sample needed since we are using the whole dataset
# For loop is slower - using this computation to speed up the process much faster.
new_page_converted = np.random.binomial(n_new,𝑝_𝑛𝑒𝑤,10000)/n_new
old_page_converted = np.random.binomial(n_old,𝑝_old,10000)/n_old
p_diffs = new_page_converted - old_page_converted
p_diffs
```

Out\[32\]:

```
array([ 0.00183579,  0.0021796 ,  0.00181484, ...,  0.00096808,\
        0.0001561 , -0.00122045])
```

In \[33\]:

```
# Calculate the mean from the null
p_diffs_mean = p_diffs.mean()
p_diffs_mean
```

Out\[33\]:

```
2.370172572845339e-05
```

**_Note: Formula explained_**

if: _np.random.binomial(n\_new,𝑝\_𝑛𝑒𝑤,10000)_ -\> we get results how many times 1s appear in one trial.

if: _np.random.binomial(n\_new,𝑝\_𝑛𝑒𝑤,10000)/n\_new_ -\> we get a probabability of ocurrance of 1s.

n\_new = trial size (0s and 1s)

p\_new = probability event of interest occurs on any one trial (calculated)

10000 = number of times to run this experiment

_because we are caunting how many times 1s appear in one trial we use n=n\_new and divide with n/new to get the proportion: [np.random.binomial](https://stackoverflow.com/questions/27644617/difference-between-n-and-size-parameters-in-np-random-binomialn-p-size-1000)_

p\_diffs = then we calculate the difference of probability converted between new and old page. It should be 0, since we are calculating this distribution form null hypotesis which is H0:pnew−pold≤0H0:pnew−pold≤0

**_Note: Further understanding of distribution and number of trialsFormula explained_**

_Below is a graphical visualization of distribution of a trials and the difference between 10000 trials and 50 trials._
Source: [Stack Overflow](https://stackoverflow.com/questions/27644617/difference-between-n-and-size-parameters-in-np-random-binomialn-p-size-1000)\*

In \[34\]:

```
#new_page_converted = np.random.binomial(n_new,𝑝_𝑛𝑒𝑤,10000)/n_new
sns.histplot(new_page_converted, kde=True);
```

![](<Base64-Image-Removed>)

In \[35\]:

```
new_page_converted1 = np.random.binomial(n_new,𝑝_𝑛𝑒𝑤,50)/n_new
sns.histplot(new_page_converted1, kde=True);
```

![](<Base64-Image-Removed>)

> **_Note_**
>
> _When conducting hypothesis testing, we always simulate the null population and then compare to the observed statistic._

i. Plot a histogram of the **p\_diffs**. Does this plot look like what you expected? Use the matching problem in the classroom to assure you fully understand what was computed here.

> **_Note_**
>
> _This plot is expected - follow normal distribution (large number and normal distribution theory). Above are two plots that show what happen if the number of trials is low in comarisson with a large number of trials._

In \[36\]:

```
# view 95% confidence interval
low, upper = np.percentile(p_diffs, .05), np.percentile(p_diffs, 99.5)
```

In \[37\]:

```
plt.hist(p_diffs); #plot the distribution of 10,000 samples under null hypotesis
plt.axvline(pdiff_actual , color='blue', linewidth=2, linestyle='dashed', label='actual mean'); #plot the accutual observation
plt.axvline(p_diffs_mean, color='darkgray', linewidth=2, linestyle='dashed', label='null mean'); #plot the mean from the null
plt.axvline(low,  color='red', linewidth=2, label='lower boundry'); # lower boundry of 95% confidence interval
plt.axvline(upper,  color='red', linewidth=2, label='upper boundry'); # upper boundry of 95% confidence interval
plt.title('Distribution of differences');
plt.xlabel('differences');
plt.ylabel('number of occurrence')
plt.legend();
plt.legend();
```

![](<Base64-Image-Removed>)

> **blue area**: the distribution from the null (assuming the null is true)
>
> **dark blue dashed line**: the observed mean - actual mean (not from the null)
>
> **gray dashed line**: the null mean
>
> **red lines**: 95% confidence interval
>
> Now we need to calculate the area - our alternative hypothesis is: H1:pnew−pold>0H1:pnew−pold>0

j. What proportion of the **p\_diffs** are greater than the actual difference observed in **ab\_data.csv**?

In \[38\]:

```
# p_diffs > pdiff_actual
p_diffs = np.array(p_diffs)
null_value = np.random.normal(0, p_diffs.std(), p_diffs.size)

# Compute p-value
p_value = (null_value > pdiff_actual).mean()
p_value
```

Out\[38\]:

```
0.9061
```

k. In words, explain what you just computed in part **j.** What is this value called in scientific studies? What does this value mean in terms of whether or not there is a difference between the new and old pages?

> **Answer**
>
> Firstly we made 10,000 trials assuming p\_old and p\_new are equal (they are coming from a null hypothesis) and created a normal distribution of differences under this assumption. Next, we compared this distribution with the actual difference in our dataset to see how likely our null hypothesis is - this is a p-value. We use p-value to determine the statistical significance of our observed difference.
>
> In cell j we computed p-value for our statistics which is the observed difference in proportions.
>
> Firstly, we calculated by simulating the distribution under the null hypothesis and then finding the probability that our statistics came from this distribution. To simulate from the null we created a normal distribution centered at zero with the same standard deviation as sampling distribution and size. Next, we computed the p-value by finding the proportion of values in the null distribution that were greater than our observed difference.
>
> **_Formula explained:_**
>
> _np.random.normal -> Draw random samples from a normal (Gaussian) distribution_
>
> _loc = 0 -> Mean (“centre”) of the distribution (p\_diffs = 0)_
>
> _scale = p\_diffs.std() -> Standard deviation (spread or “width”) of the distribution_
>
> _size = p\_diffs.size -> Size of distribution._
>
> **_p-value of 0.9009 means that nearly all statistics came from a null (almost all ~ 90%); therefore, we fail to reject null hypothesis, meaning that alternative hypothesis is not true (new page is the same or worse than the old page.)_**

> **Note and additional resources**
>
> The p-value helps us make a decision. Because of the way we construct our assumptions, when calculated, the p-value tells us the probability of committing a Type I error if the null hypothesis is true. (A Type I error is when you incorrectly reject the null hypothesis - usually we would consider making Type I errors to be 'bad,' so we want to make as few of them as possible, and make this chance quite low)
>
> A low p-value is often considered to be less than 0.05 in business and research, and 0.01 in medicine, but it could be any value appropriate to the situation. That is, if you get a p-value that is 0.05, this means that there is a 5% chance that a statistic that you observed came from a population where the null hypothesis is true. With this reasoning, at low p-values we typically reject the null hypothesis. That is, we act on the assumption that the observed statistic came from a population where the alternate hypothesis is true.
> Source: [p-value](https://rebeccaebarnes.github.io/2018/05/01/what-is-a-p-value)

l. We could also use a built-in to achieve similar results. Though using the built-in might be easier to code, the above portions are a walkthrough of the ideas that are critical to correctly thinking about statistical significance. Fill in the below to calculate the number of conversions for each page, as well as the number of individuals who received each page. Let `n_old` and `n_new` refer the the number of rows associated with the old page and new pages, respectively.

In \[39\]:

```
convert_old = df2.query('group == "control" & converted == 1').user_id.count()
convert_new = df2.query('group == "treatment" & converted == 1').user_id.count()
n_old = df2.query('landing_page == "old_page"').shape[0]
n_new = df2.query('landing_page == "new_page"').shape[0]
```

m. Now use `stats.proportions_ztest` to compute your test statistic and p-value. [Here](https://docs.w3cub.com/statsmodels/generated/statsmodels.stats.proportion.proportions_ztest) is a helpful link on using the built in.

In \[40\]:

```
import statsmodels.api as sm
z_test, p_value = sm.stats.proportions_ztest([convert_new, convert_old], [n_new, n_old], alternative='larger')
z_test, p_value
```

Out\[40\]:

```
(-1.3109241984234394, 0.9050583127590245)
```

In \[41\]:

```
# Calculating critical value
# import library
from scipy.stats import norm
# Determine our critical value (upper bound og 95%)
p = 0.95
# Calculate
cval = norm.ppf(p)
cval
```

Out\[41\]:

```
1.6448536269514722
```

In \[42\]:

```
from IPython import display
display.Image("Resources/criticalvalue.png", width=500)
```

Out\[42\]:

![](<Base64-Image-Removed>)

_Source: [statisticshowto.com](https://www.statisticshowto.com/probability-and-statistics/find-critical-values/)_

n. What do the z-score and p-value you computed in the previous question mean for the conversion rates of the old and new pages? Do they agree with the findings in parts **j.** and **k.**?

> **Answer**
>
> Proportions z\_test build-in function did all the computation in a few lines of code that reflect what we did in Part II. P-value is the same as in Part II (in cell j).
> p-value and z-score computed in cell m agree with p-value computed in cell j, that is p-value of 0.905, meaning we fail to reject the null hypotesis and based on this computations we can conclude that the new page won't attrack more traffic.
>
> **Interpretation of p-value and z-value**
>
> The **Z-value** is a test that measures the difference between an observed statistic and its hypothesized population parameter in units of standard error. We can compare the Z-value to critical values of the standard normal distribution to determine whether to reject the null hypothesis. Z-score shows how many standard deviations away our observed (actual) difference is to the center. How many standard deviations away pdiff\_actual is from p\_diffs. In order to interpret z-score we look at the critcal value. Critical value for the 95% confidence interval (or alpha level of 0.05 or 5%) is 1.64. Our z-test is -3.11; therefore z-score value falls out of this critical value and we fail to reject the null hypotesis.
>
> The **p-value** is a probability that measures the evidence against the null hypothesis. A smaller p-value provides stronger evidence against the null hypothesis.

Source: [minitab.com](https://support.minitab.com/en-us/minitab/19/help-and-how-to/statistics/basic-statistics/how-to/1-sample-z/interpret-the-results/all-statistics-and-graphs/)

In \[43\]:

```
# calculating standard deviations
standart_deviation = np.std(p_diffs)
std1_low = 0 - standart_deviation*1
std1_high = 0 + standart_deviation*1
std2_low = 0 - standart_deviation*2
std2_high = 0 + standart_deviation*2
std3_low = 0 - standart_deviation*3
std3_high = 0 + standart_deviation*3

# visualizing
fig, ax = plt.subplots(figsize=(8, 6))
sns.histplot(p_diffs, bins=50, color="skyblue", kde=True);
plt.axvline(x=std1_low, color='blue', label='1 std');
plt.axvline(x=std1_high, color='blue');
plt.axvline(x=std2_low, color='green', label='2 std');
plt.axvline(x=std2_high, color='green');
plt.axvline(x=std3_low, color='orange', label='3 std');
plt.axvline(x=std3_high, color='orange');
plt.axvline(upper,  color='black', linewidth=2, linestyle='dashed',label='upper boundry 95%'); # upper boundry of 95% confidence interval
plt.title('Distribution of simulation');
plt.xlabel('differences');
plt.ylabel('count')

# Shade the are between the curve and alpha - where critical value is
kde_x, kde_y = ax.lines[0].get_data()
ax.fill_between(kde_x, kde_y, where=(kde_x>upper),
                interpolate=True, alpha=1, color='red', label='critical value')

# Shade the area between std-1 and std -2 where z-score is:
ax.axvspan(std1_low, std2_low, alpha=0.5, color='gray', label='z-score')

# This will shade all area from "upper" til the end of the chart - not used in this chart, but kept for the reference.
#ax.axvspan(upper, xlim[1], alpha=0.3, color='red', label='critical value')
# Get x-axis limit to shade the area
#xlim = ax.get_xlim()
#ax.margins(x=0)

plt.legend();
```

![](<Base64-Image-Removed>)

_z-score of -1.31 falls between -1st and -2nd standard deviation - shaded gray area_

_critical value of αα = 0.05 (95% confidence interval) - shaded red area_

**_Source: Udacity Knowledge FAQ & [z-score](https://www.statisticshowto.com/probability-and-statistics/z-score/) & [stacloverflow](https://stackoverflow.com/questions/46685453/how-to-fill-with-a-different-color-an-area-in-seaborn-distplot)_**

### Part III - A regression approach [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Part-III---A-regression-approach)

`1.` In this final part, you will see that the result you acheived in the previous A/B test can also be acheived by performing regression.

a. Since each row is either a conversion or no conversion, what type of regression should you be performing in this case?

> **Answer**
>
> Unlike linear regression (used for predicting quantiative response a continious numerical variable), logistic regression is used to predict a categorical response, a binary response with only two possible outcomes in our case a conversion vs. no conversion.

b. The goal is to use **statsmodels** to fit the regression model you specified in part **a.** to see if there is a significant difference in conversion based on which page a customer receives. However, you first need to create a column for the intercept, and create a dummy variable column for which page each user received. Add an **intercept** column, as well as an **ab\_page** column, which is 1 when an individual receives the **treatment** and 0 if **control**.

In \[44\]:

```
import statsmodels.api as sm
df2['intercept'] = 1
df2['ab_page'] = pd.get_dummies(df['landing_page'])['new_page']
```

> **_Note_**
>
> _Intercept == 1: initialize the value of the bias to 1 because it will be multiplied by the bias weights to produce the final bias value. If it was set to 0, it would always produce 0. If it was set to 5 it would scale the weights to much._
>
> _Source: Udacity Knowledge._

In \[45\]:

```
df2.head()
```

Out\[45\]:

|  | user\_id | timestamp | group | landing\_page | converted | intercept | ab\_page |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 851104 | 2017-01-21 22:11:48.556739 | control | old\_page | 0 | 1 | 0 |
| 1 | 804228 | 2017-01-12 08:01:45.159739 | control | old\_page | 0 | 1 | 0 |
| 2 | 661590 | 2017-01-11 16:55:06.154213 | treatment | new\_page | 0 | 1 | 1 |
| 3 | 853541 | 2017-01-08 18:28:03.143765 | treatment | new\_page | 0 | 1 | 1 |
| 4 | 864975 | 2017-01-21 01:52:26.210827 | control | old\_page | 1 | 1 | 0 |

c. Use **statsmodels** to import your regression model. Instantiate the model, and fit the model using the two columns you created in part **b.** to predict whether or not an individual converts.

In \[46\]:

```
log_mod = sm.Logit(df2['converted'], df2[['intercept', 'ab_page']])
results = log_mod.fit()
```

```
Optimization terminated successfully.
         Current function value: 0.366118
         Iterations 6
```

d. Provide the summary of your model below, and use it as necessary to answer the following questions.

In \[47\]:

```
results.summary()
```

Out\[47\]:

| Dep. Variable: | converted | No. Observations: | 290584 |
| Model: | Logit | Df Residuals: | 290582 |
| Method: | MLE | Df Model: | 1 |
| Date: | Tue, 04 May 2021 | Pseudo R-squ.: | 8.077e-06 |
| Time: | 16:54:33 | Log-Likelihood: | -1.0639e+05 |
| converged: | True | LL-Null: | -1.0639e+05 |
| Covariance Type: | nonrobust | LLR p-value: | 0.1899 |

Logit Regression Results

|  | coef | std err | z | P>\|z\| | \[0.025 | 0.975\] |
| intercept | -1.9888 | 0.008 | -246.669 | 0.000 | -2.005 | -1.973 |
| ab\_page | -0.0150 | 0.011 | -1.311 | 0.190 | -0.037 | 0.007 |

In \[48\]:

```
# Exponentiate each variable. Now each of these resulting value is the multiplicative change in the odds
np.exp(results.params)
```

Out\[48\]:

```
intercept    0.136863
ab_page      0.985123
dtype: float64
```

In \[49\]:

```
# Calculate the reciprocal -  with the values less than 1.
1/_
```

Out\[49\]:

```
intercept    7.306593
ab_page      1.015102
dtype: float64
```

e. What is the p-value associated with **ab\_page**? Why does it differ from the value you found in **Part II**?

**Hint**: What are the null and alternative hypotheses associated with your regression model, and how do they compare to the null and alternative hypotheses in the **Part II**?

> **Answer**
>
> p-value for ab\_page is 0.190. This p-value still indicates the same as p-value in Part II, that is we fail to reject the null hypotesis and based on this computations we can conclude that the new page won't attrack more traffic.
> The p-value differs because in Part II, we are doing a one-sided test since our null hypothesis is "p\_old - p\_new >= 0" in Part III, we are doing a two-sided test ("p\_old = p\_new"). For logistic regression we have two outputs possible "converted" or "not converted."

f. Now, you are considering other things that might influence whether or not an individual converts. Discuss why it is a good idea to consider other factors to add into your regression model. Are there any disadvantages to adding additional terms into your regression model?

> **Answer**
>
> Adding other features to the model can improve the model performance; however, we need to be carful with the interpretation. One potential side effect of having multicollinearity in the model is that the coeficent can be counter-intuitive. This happen if predictors are strongly corelated with one another. We can check for this correlation either with scatter plots or VIFs (variance inflation factors). In order to interpret the model more accurately we could remove at least one of highly correlated variable that are of least interest.

g. Now along with testing if the conversion rate changes for different pages, also add an effect based on which country a user lives. You will need to read in the **countries.csv** dataset and merge together your datasets on the approporiate rows. [Here](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.join.html) are the docs for joining tables.

Does it appear that country had an impact on conversion? Don't forget to create dummy variables for these country columns - **Hint: You will need two columns for the three dummy variables.** Provide the statistical output as well as a written response to answer this question.

In \[50\]:

```
countries_df = pd.read_csv('Data/countries.csv')
df_new = countries_df.set_index('user_id').join(df2.set_index('user_id'), how='inner')
df_new.tail()
```

Out\[50\]:

|  | country | timestamp | group | landing\_page | converted | intercept | ab\_page |
| --- | --- | --- | --- | --- | --- | --- | --- |
| user\_id |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 653118 | US | 2017-01-09 03:12:31.034796 | control | old\_page | 0 | 1 | 0 |
| 878226 | UK | 2017-01-05 15:02:50.334962 | control | old\_page | 0 | 1 | 0 |
| 799368 | UK | 2017-01-09 18:07:34.253935 | control | old\_page | 0 | 1 | 0 |
| 655535 | CA | 2017-01-09 13:30:47.524512 | treatment | new\_page | 0 | 1 | 1 |
| 934996 | UK | 2017-01-09 00:30:08.377677 | control | old\_page | 0 | 1 | 0 |

In \[51\]:

```
# Country dummies - check what values we have
df_new.country.value_counts()
```

Out\[51\]:

```
US    203619
UK     72466
CA     14499
Name: country, dtype: int64
```

In \[52\]:

```
### Create the necessary dummy variables
df_new[['CA','UK','US']] = pd.get_dummies(df_new['country'])
df_new.tail()
```

Out\[52\]:

|  | country | timestamp | group | landing\_page | converted | intercept | ab\_page | CA | UK | US |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| user\_id |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 653118 | US | 2017-01-09 03:12:31.034796 | control | old\_page | 0 | 1 | 0 | 0 | 0 | 1 |
| 878226 | UK | 2017-01-05 15:02:50.334962 | control | old\_page | 0 | 1 | 0 | 0 | 1 | 0 |
| 799368 | UK | 2017-01-09 18:07:34.253935 | control | old\_page | 0 | 1 | 0 | 0 | 1 | 0 |
| 655535 | CA | 2017-01-09 13:30:47.524512 | treatment | new\_page | 0 | 1 | 1 | 1 | 0 | 0 |
| 934996 | UK | 2017-01-09 00:30:08.377677 | control | old\_page | 0 | 1 | 0 | 0 | 1 | 0 |

In \[53\]:

```
# drop one column to get full rank
df_new = df_new.drop('US', axis=1)
```

In \[54\]:

```
### Fit Your Linear Model And Obtain the Results
df_new['intercept'] = 1
log_mod = sm.Logit(df_new['converted'], df_new[['intercept', 'CA', 'UK']])
results = log_mod.fit()
results.summary()
```

```
Optimization terminated successfully.
         Current function value: 0.366116
         Iterations 6
```

Out\[54\]:

| Dep. Variable: | converted | No. Observations: | 290584 |
| Model: | Logit | Df Residuals: | 290581 |
| Method: | MLE | Df Model: | 2 |
| Date: | Tue, 04 May 2021 | Pseudo R-squ.: | 1.521e-05 |
| Time: | 16:54:35 | Log-Likelihood: | -1.0639e+05 |
| converged: | True | LL-Null: | -1.0639e+05 |
| Covariance Type: | nonrobust | LLR p-value: | 0.1984 |

Logit Regression Results

|  | coef | std err | z | P>\|z\| | \[0.025 | 0.975\] |
| intercept | -1.9967 | 0.007 | -292.314 | 0.000 | -2.010 | -1.983 |
| CA | -0.0408 | 0.027 | -1.518 | 0.129 | -0.093 | 0.012 |
| UK | 0.0099 | 0.013 | 0.746 | 0.456 | -0.016 | 0.036 |

> **_Interpretation of the results - p-value_**
>
> Based on the p-value country doesn't have impact on conversion. None of the variables are statistical significant (p-value < 0.05). In logistic regression model summary we might use p-values to help us understand if a particular variable was significant and it's a great quick check to understand which relationship appear to be important. Furthermore we can interpret these coeficients to help us understand corelations.
>
> **_In order to interpret coeficient we need to exponentiate each:_**
>
> _The math.exp() method returns E raised to the power of x (Ex)._
> _'E' is the base of the natural system of logarithms (approximately 2.718282) and x is the number passed to it._ [www.w3schools](https://www.w3schools.com/python/ref_math_exp.asp)

In \[55\]:

```
# Exponentiate each variable. Now each of these resulting value is the multiplicative change in the odds
np.exp(results.params)
```

Out\[55\]:

```
intercept    0.135779
CA           0.960018
UK           1.009966
dtype: float64
```

In \[56\]:

```
# Calculate the reciprocal -  with the values less than 1.
1/_
```

Out\[56\]:

```
intercept    7.364925
CA           1.041647
UK           0.990133
dtype: float64
```

> **_Interpretation of the results - coeficient_**
>
> This results in multiplicative change in the odds of being in the one category of this value, holding all other variable constant.
>
> We can interpret results above if individual is from US is 1.01-times likely to convert than if they came form UK and individual is 1.04 likely to convert if they came form CA.

In \[57\]:

```
### Fit Your Linear Model And Obtain the Results
df_new['intercept'] = 1
log_mod2 = sm.Logit(df_new['converted'], df_new[['intercept', 'ab_page', 'CA', 'UK']])
results2 = log_mod2.fit()
results2.summary()
```

```
Optimization terminated successfully.
         Current function value: 0.366113
         Iterations 6
```

Out\[57\]:

| Dep. Variable: | converted | No. Observations: | 290584 |
| Model: | Logit | Df Residuals: | 290580 |
| Method: | MLE | Df Model: | 3 |
| Date: | Tue, 04 May 2021 | Pseudo R-squ.: | 2.323e-05 |
| Time: | 16:54:37 | Log-Likelihood: | -1.0639e+05 |
| converged: | True | LL-Null: | -1.0639e+05 |
| Covariance Type: | nonrobust | LLR p-value: | 0.1760 |

Logit Regression Results

|  | coef | std err | z | P>\|z\| | \[0.025 | 0.975\] |
| intercept | -1.9893 | 0.009 | -223.763 | 0.000 | -2.007 | -1.972 |
| ab\_page | -0.0149 | 0.011 | -1.307 | 0.191 | -0.037 | 0.007 |
| CA | -0.0408 | 0.027 | -1.516 | 0.130 | -0.093 | 0.012 |
| UK | 0.0099 | 0.013 | 0.743 | 0.457 | -0.016 | 0.036 |

In \[58\]:

```
# Exponentiate each variable. Now each of these resulting value is the multiplicative change in the odds
np.exp(results2.params)
```

Out\[58\]:

```
intercept    0.136795
ab_page      0.985168
CA           0.960062
UK           1.009932
dtype: float64
```

In \[59\]:

```
# Calculate the reciprocal -  with the values less than 1.
1/_
```

Out\[59\]:

```
intercept    7.310207
ab_page      1.015056
CA           1.041599
UK           0.990165
dtype: float64
```

**_Interpretation of the results_**

Adding more terms in this case did not change the model. None of the variables are statistical significant (p-value < 0.05) and coeficient stayed similar than in model above.
We can conclude that there is no significant p-value(all higher than 0.05) even after the addition of country dependent conversion and therefore we fail to reject the null. Company should stay on the old\_pages only as there's no enough evidence that the new\_pages are doing better.

h. Though you have now looked at the individual factors of country and page on conversion, we would now like to look at an interaction between page and country to see if there significant effects on conversion. Create the necessary additional columns, and fit the new model.

Provide the summary results, and your conclusions based on the results.

In \[60\]:

```
# adding interaction between page and country
df_new['CA_abpage'] = df_new.CA*df_new.ab_page
df_new['UK_abpage'] = df_new.UK*df_new.ab_page
```

_When we include higher order terms into our model we also need to include lower order terms. Mathematically, an interaction is created by multiplying two variables by one another and adding this term to our linear regression model. If the slope (vertical difference) between two variables is not the same, we consider adding interaction in our model if the slope is the same than we do not add an interaction._

In \[61\]:

```
### Fit Your Linear Model And Obtain the Results
df_new['intercept'] = 1
log_mod_int = sm.Logit(df_new['converted'], df_new[['intercept', 'CA', 'UK','ab_page', 'CA_abpage', 'UK_abpage']])
results_int = log_mod_int.fit()
results_int.summary()
```

```
Optimization terminated successfully.
         Current function value: 0.366109
         Iterations 6
```

Out\[61\]:

| Dep. Variable: | converted | No. Observations: | 290584 |
| Model: | Logit | Df Residuals: | 290578 |
| Method: | MLE | Df Model: | 5 |
| Date: | Tue, 04 May 2021 | Pseudo R-squ.: | 3.482e-05 |
| Time: | 16:54:39 | Log-Likelihood: | -1.0639e+05 |
| converged: | True | LL-Null: | -1.0639e+05 |
| Covariance Type: | nonrobust | LLR p-value: | 0.1920 |

Logit Regression Results

|  | coef | std err | z | P>\|z\| | \[0.025 | 0.975\] |
| intercept | -1.9865 | 0.010 | -206.344 | 0.000 | -2.005 | -1.968 |
| CA | -0.0175 | 0.038 | -0.465 | 0.642 | -0.091 | 0.056 |
| UK | -0.0057 | 0.019 | -0.306 | 0.760 | -0.043 | 0.031 |
| ab\_page | -0.0206 | 0.014 | -1.505 | 0.132 | -0.047 | 0.006 |
| CA\_abpage | -0.0469 | 0.054 | -0.872 | 0.383 | -0.152 | 0.059 |
| UK\_abpage | 0.0314 | 0.027 | 1.181 | 0.238 | -0.021 | 0.084 |

**_Interpretation of results_**
When the slopes for two variables no longer match we would want to add an interaction term between two variables (page and country) to our model. In this case the way ab page is related to the converted and is dependent from which country that individual is coming from.

Adding higher terms did not improve the model. Based on p-value for CA\_abpage and UK\_abpage is 0.383 and 0.238, respectively indicating that interactions are not significant and we would consider removing them from the model.
However it is essential to be aware of interactions since they can improve our models or even hurt if we do not add them and show significance.

**_Higher order terms - notes_**

Sometimes we would like to fit models where the response is not lineary related to the explanatory variable. We can do this with what are known as higher order terms. Higher order terms include quadratics, cubics and many other relationships. In order to add these terms to our linear models, we can simply multiply our columns by one another.

**_Additional notes - not part of the analysis_**

🎈logistic regression: confusion matrix, exponentiate each variable, VIF & multicollinearity

🎈 multiple linerar regression: VIF, scatter plots in multicollinearity

[VIF&multicollinearity](https://priyanshuraj.online/dashboard/Multicollinearity_And_VIFs_forReference.ipynb)

## Conclusions [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Conclusions)

Congratulations on completing the project!

### Gather Submission Materials [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Gather-Submission-Materials)

Once you are satisfied with the status of your Notebook, you should save it in a format that will make it easy for others to read. You can use the **File -> Download as -> HTML (.html)** menu to save your notebook as an .html file. If you are working locally and get an error about "No module name", then open a terminal and try installing the missing module using `pip install <module_name>` (don't include the "<" or ">" or any words following a period in the module name).

You will submit both your original Notebook and an HTML or PDF copy of the Notebook for review. There is no need for you to include any data files with your submission. If you made reference to other websites, books, and other resources to help you in solving tasks in the project, make sure that you document them. It is recommended that you either add a "Resources" section in a Markdown cell at the end of the Notebook report, or you can include a `readme.txt` file documenting your sources.

### Submit the Project [¶](https://priyanshuraj.online/dashboard/Analyze_AB_test.html\#Submit-the-Project)

When you're ready, click on the "Submit Project" button to go to the project submission page. You can submit your files as a .zip archive or you can link to a GitHub repository containing your project files. If you go with GitHub, note that your submission will be a snapshot of the linked repository at time of submission. It is recommended that you keep each project in a separate repository to avoid any potential confusion: if a reviewer gets multiple folders representing multiple projects, there might be confusion regarding what project is to be evaluated.

It can take us up to a week to grade the project, but in most cases it is much faster. You will get an email once your submission has been reviewed. If you are having any problems submitting your project or wish to check on the status of your submission, please email me at hi@priyanshuraj.online. In the meantime, you should feel free to continue on with your learning journey by beginning the next module in the program.

In \[ \]:

```

```