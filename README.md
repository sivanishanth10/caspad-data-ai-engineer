# What Shoppers Say About Fit

## 1. The problem in my own words

An online women's clothing retailer is seeing increasing product returns. The main question is whether low-rated reviews are strongly related to size and fit problems.

This project analyzes customer reviews to identify:

* Data quality and consistency issues
* Departments and classes with lower ratings
* Patterns in ratings across reviewer age groups
* The main issues mentioned in low-rated reviews
* Whether size/fit is a significant contributor to negative customer feedback

The goal is to provide evidence-based recommendations for improving product pages, sizing information, and product presentation.

---

## 2. Assumptions

* A review with a rating of 1, 2, or 3 is treated as a low-rated review.
* Missing review text is treated as an empty string during data cleaning.
* Missing product category values are represented as `Unknown`.
* Product information is maintained at one row per `Clothing ID`.
* Reviewer age bands are defined as:

  * ≤20
  * 21–30
  * 31–40
  * 41–50
  * 51–60
  * 61+
* For the LLM analysis, one main issue is assigned to each review using the fixed labels specified in the project.
* The LLM output was manually checked on a random sample of 30 reviews.

---

## 3. Data source and how to run

### Data source

The dataset used is the **Women’s E-Commerce Clothing Reviews** dataset from Kaggle.

The dataset contains **23,486 reviews**.

The downloaded raw CSV is stored locally under:

```text
data/Womens Clothing E-Commerce Clothing Reviews.csv
```

The raw dataset is not required to be committed to GitHub.

### Main tools and packages

* Python
* Pandas
* Matplotlib
* Jupyter Notebook
* Groq API
* python-dotenv

### Project structure

```text
Caspad_Project/
│
├── data/
│   └── Womens Clothing E-Commerce Clothing Reviews.csv
│
├── notebooks/
│   ├── 01_part_a.ipynb
│   └── 03_part-c.ipynb
│
├── outputs/
│   ├── products.csv
│   ├── cleaned_reviews.csv
│   ├── department_summary.csv
│   ├── class_summary.csv
│   ├── low_rating_counts_by_class.csv
│   ├── age_band_summary.csv
│   ├── llm_tagged_193_reviews.csv
│   ├── manual_30_check.csv
│   └── issue_mix_by_department.csv
│
├── .env
├── .gitignore
├── requirements.txt
└── README.md
```

### How to run

1. Download the dataset from Kaggle.
2. Place the CSV inside the `data/` directory.
3. Install the required Python packages.
4. For Part C, add the Groq API key to `.env`:

```text
GROQ_API_KEY=your_api_key
```

5. Run the notebooks in the following order:

   * `01_part_a.ipynb`
   * Part B analysis cells
   * `03_part-c.ipynb`

The notebooks generate the required CSV outputs inside the `outputs/` directory.

---

## 4. Part A: Cleaning rules, affected rows, and checks

### Cleaning rules

The dataset contains 23,486 review records.

Missing review text was replaced with an empty string:

```python
df["Review Text"] = df["Review Text"].fillna("")
```

Missing product category values were replaced with `Unknown`:

```python
for col in ["Division Name", "Department Name", "Class Name"]:
    df[col] = df[col].fillna("Unknown")
```

### Missing values found

| Column          | Missing rows |
| --------------- | -----------: |
| Review Text     |          845 |
| Title           |        3,810 |
| Division Name   |           14 |
| Department Name |           14 |
| Class Name      |           14 |

The project analysis does not require the missing review titles to be filled, so the `Title` column was retained without artificial values.

### Product and review tables

The original dataset contains **23,486 review rows**.

A separate product table was created at the `Clothing ID` grain.

* Review table: **23,486 rows**
* Product table: **1,206 unique Clothing IDs**

During the product consistency check, one Clothing ID had conflicting category information:

* Clothing ID: `1119`
* Department: `Jackets`
* Class values: `Jackets` and `Outerwear`

This conflict was identified and retained as a documented data-quality issue rather than silently changing the source information.

### Validation checks

Three required checks were performed:

1. Each Clothing ID was checked for conflicting Department/Class values.
2. Ratings were checked to ensure they were between 1 and 5.
3. Recommended IND was checked to ensure it contained only 0 and 1.

Results:

* Conflicting Clothing IDs: **1**
* Invalid ratings: **0**
* Invalid Recommended IND values: **0**

---

## 5. Part B: Key numbers and charts

### Department-level results

| Department | Average Rating | Share Recommended |
| ---------- | -------------: | ----------------: |
| Bottoms    |           4.29 |             85.1% |
| Intimate   |           4.28 |             85.0% |
| Jackets    |           4.26 |             83.6% |
| Tops       |           4.17 |             81.5% |
| Dresses    |           4.15 |             80.8% |
| Trend      |           3.82 |             73.9% |

**Key observation:** Trend has the lowest average rating and recommendation rate among the departments, while Bottoms and Intimate have the strongest overall results.

### Classes with the most 1–2 star ratings

| Class    | Low-rating count |
| -------- | ---------------: |
| Dresses  |              689 |
| Knits    |              506 |
| Blouses  |              348 |
| Sweaters |              155 |
| Pants    |              124 |

**Key observation:** Dresses, Knits, and Blouses account for the largest numbers of 1–2 star reviews and should receive particular attention.

### Reviewer age bands

| Age Band | Average Rating |
| -------- | -------------: |
| ≤20      |           4.32 |
| 21–30    |           4.19 |
| 31–40    |           4.17 |
| 41–50    |           4.17 |
| 51–60    |           4.25 |
| 61+      |           4.29 |

**Key observation:** Average ratings are lowest for the 31–50 age groups and somewhat higher among the youngest and oldest groups.

### Charts

The analysis includes charts for:

* Average rating by department
* Share recommended by department
* Classes with the most 1–2 star ratings
* Average rating by reviewer age band
* Average rating by class
* Share recommended by class

---

## 6. Part C: LLM tagging and validation

### Sampling

There were **5,278 reviews rated 1–3 stars**.

A random sample of **200** low-rated reviews was selected using a fixed random seed.

Of these:

* 200 reviews were sampled
* 7 had missing review text
* 193 reviews had usable text and were sent for LLM classification

### LLM classification

The LLM was asked to assign:

**Main issue**

* size/fit
* fabric/quality
* looks different from photo
* comfort
* price
* other

**Fit**

* small
* large
* neither

The same structured prompt format was used for the reviews.

The LLM output was constrained to:

text
issue|fit


The raw LLM output was saved before final parsing and validation.

### Manual validation

A random sample of 30 reviews was manually checked against the LLM predictions.

Results:

| Metric         |   Accuracy |
| -------------- | ---------: |
| Issue accuracy | 96.67% |
| Fit accuracy   | 90.00% |
| Exact accuracy | 90.00% |

These results provide a manual estimate of classification quality on the validation sample.

### Issue mix

Across the 193 classified reviews, the dominant issue was size/fit.

The issue mix was also broken down by department and saved to:

```text
outputs/issue_mix_by_department.csv
```

The strongest signal is that size/fit problems are substantially more common than the other issue categories in the low-rated sample.

---

## 7. Limitations and what next

### Limitations

* The LLM analysis used 193 reviews with usable text rather than all 5,278 low-rated reviews.
* The manual accuracy check used only 30 reviews, so the reported accuracy should not be treated as a population-wide accuracy estimate.
* A review may mention multiple problems, but the classification assigns one main issue.
* The dataset is historical and may not represent current customer behavior.
* The analysis identifies associations in customer feedback but does not prove that a particular issue directly causes product returns.

### What I would do next

If more time and production data were available, I would:

1. Compare the identified issues with actual return reasons.
2. Analyze issue patterns at the individual product level.
3. Increase the manual validation sample.
4. Monitor issue trends over time after product-page or sizing changes.

---

## 8. How I used AI tools

AI tools were used as development and productivity support during the project.

### ChatGPT

I used ChatGPT to:

* Clarify project requirements and organize the implementation steps.
* Review Python/Pandas logic during development.
* Help troubleshoot coding and API-related issues.
* Improve explanations and documentation.
* Discuss potential edge cases and validation approaches.

### Claude

I used Claude as a second AI-assisted development resource for:

* Reviewing implementation approaches.
* Cross-checking code and reasoning.
* Helping identify potential issues during development.
* Improving clarity of technical explanations.

### LLM used for the project analysis

For Part C, the **Groq API with `openai/gpt-oss-20b`** was used to classify the selected low-rated customer reviews.

The classification prompt and structured output format were kept consistent across the sample.

Importantly, the LLM-generated classifications were not treated as automatically correct. A manually reviewed sample of 30 reviews was used to measure issue, fit, and exact-match accuracy.

---

## 9. Optional extra

Not completed.

The optional step of running the LLM tagger across all low-rated reviews was not included because the required Parts A, B, and C were prioritized.

