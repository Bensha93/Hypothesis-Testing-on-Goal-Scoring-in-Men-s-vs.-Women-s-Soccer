
# Hypothesis Testing with Men's and Women's Soccer Matches

You’re working as a sports journalist at a major online sports media company, specializing in soccer analysis and reporting. You’ve been watching both men’s and women’s international soccer matches for a number of years, and your gut instinct tells you that more goals are scored in women’s international football matches than men’s. This would make an interesting investigative article that your subscribers are bound to love, but you’ll need to perform a valid statistical hypothesis test to be sure!

While scoping this project, you acknowledge that the sport has changed a lot over the years, and performances likely vary a lot depending on the tournament. Therefore, you decide to limit the data used in the analysis to only official FIFA World Cup matches (not including qualifiers) since January 1st, 2002.

You create two datasets containing the results of every official men's and women's international football match since the 19th century, which you scraped from a reliable online source. This data is stored in two CSV files: **women_results.csv** and **men_results.csv**.

The question you are trying to determine the answer to is:

**Are more goals scored in women's international soccer matches than men's?**

### Assumptions:
- **Significance Level**: 10%
- **Null Hypothesis (H₀)**: The mean number of goals scored in women's international soccer matches is the same as men's.
- **Alternative Hypothesis (H₁)**: The mean number of goals scored in women's international soccer matches is greater than men's.

## Libraries Used:
- **Pandas**: for data manipulation
- **Matplotlib**: for data visualization
- **Pingouin**: for statistical tests
- **SciPy**: for statistical tests

## Approach

### 1. Import Libraries

```python
import pandas as pd
import matplotlib.pyplot as plt
import pingouin
from scipy.stats import mannwhitneyu
```

### 2. Load Men’s and Women’s Datasets

```python
men = pd.read_csv("men_results.csv")
women = pd.read_csv("women_results.csv")
```

### 3. Filter the Data for the Time Range and Tournament

```python
men["date"] = pd.to_datetime(men["date"])
men_subset = men[(men["date"] > "2002-01-01") & (men["tournament"].isin(["FIFA World Cup"]))]

women["date"] = pd.to_datetime(women["date"])
women_subset = women[(women["date"] > "2002-01-01") & (women["tournament"].isin(["FIFA World Cup"]))]

```

### 4. Create Group and Goals Scored Columns

```python
men_subset["group"] = "men"
women_subset["group"] = "women"
men_subset["goals_scored"] = men_subset["home_score"] + men_subset["away_score"]
women_subset["goals_scored"] = women_subset["home_score"] + women_subset["away_score"]
```

### 5. Determine Normality Using Histograms

```python
men_subset["goals_scored"].hist()
plt.show()
plt.clf()
```

### 6. Wilcoxon-Mann-Whitney Test

Since the data is not normally distributed, we perform a **Wilcoxon-Mann-Whitney test** to compare the number of goals scored between men's and women's matches.

```python
# Combine women's and men's data and calculate goals scored in each match
both = pd.concat([women_subset, men_subset], axis=0, ignore_index=True)

# Transform the data for the pingouin Mann-Whitney U t-test/Wilcoxon-Mann-Whitney test
both_subset = both[["goals_scored", "group"]]
both_subset_wide = both_subset.pivot(columns="group", values="goals_scored")

# Perform right-tailed Wilcoxon-Mann-Whitney test with pingouin
results_pg = pingouin.mwu(x=both_subset_wide["women"],
                          y=both_subset_wide["men"],
                          alternative="greater")
```

### 7. Alternative SciPy Solution

```python
# Perform right-tailed Wilcoxon-Mann-Whitney test with scipy
results_scipy = mannwhitneyu(x=women_subset["goals_scored"],
                             y=men_subset["goals_scored"],
                             alternative="greater")
```

### 8. Determine Hypothesis Test Result Using Significance Level

```python
# Extract p-value as a float
p_val = results_pg["p-val"].values[0]

# Determine hypothesis test result using sig. level
if p_val <= 0.01:
    result = "rejected"
else:
    result = "fail to reject"

result_dict = {"p_val": p_val, "result": result}

print(f"The significance level of 10% exceeds the resulting probability value of {result_dict['p_val']}. This implies that the hypothesis should be {result_dict['result']}.")
```

### 9. Conclusion

**Result**:
- The significance level of 10% exceeds the resulting probability value of **0.005106609825443641**.
- This implies that the null hypothesis **should be rejected**.

This confirms that, based on the data, **more goals are scored in women's international soccer matches** than in men's FIFA World Cup matches since 2002.

## Visualizations

- **Goal Scored Distribution for Men’s Matches**  
  ![Men's Goal Distribution](./images/men_goals_distribution.png)

- **Goal Scored Distribution for Women’s Matches**  
  ![Women's Goal Distribution](./images/women_goals_distribution.png)

## Dataset

- **men_results.csv**: Contains results of men’s FIFA World Cup matches since 1900.
- **women_results.csv**: Contains results of women’s FIFA World Cup matches since 1900.

