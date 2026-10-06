# What Shoppers Say About Fit

## Project Overview

This project analyzes women's clothing e-commerce reviews to understand whether low customer ratings are related to size and fit issues.

The analysis uses the **Women’s E-Commerce Clothing Reviews** dataset containing 23,486 reviews. The project covers:

- Data cleaning and validation
- Product and review table creation
- Rating and recommendation analysis
- Low-rating analysis by product class
- Rating analysis by reviewer age band
- LLM-based classification of issues in low-rated reviews
- Manual validation of 30 LLM classifications
- Recommendations for product and listing improvements

## Tools Used

- Python
- Pandas
- Jupyter Notebook
- Groq API
- LLM: `openai/gpt-oss-20b`
- Matplotlib
- CSV files for outputs

## Part A — Data Preparation and Validation

### Dataset

The raw dataset contains **23,486 review records** and 11 columns.

### Table Design

The data was separated into two tables:

**Product table**

- Clothing ID
- Division Name
- Department Name
- Class Name

Result: **1,402 product records**

**Review table**

- Review ID
- Clothing ID
- Age
- Title
- Review Text
- Rating
- Recommended IND
- Positive Feedback Count

Result: **23,486 review records**

### Cleaning Rules

- Missing `Review Text` values were replaced with an empty string because the review text is used for text analysis.
- Missing `Division Name`, `Department Name`, and `Class Name` values were replaced with `Unknown`.
- Missing `Title` values were retained because the title was not required for the main analysis.
- No review rows were removed during cleaning.

### Validation Checks

**Clothing ID consistency**

Each Clothing ID was checked to determine whether it mapped to a single department and class.

One conflict was found:

- Clothing ID **1119** has more than one Department Name and Class Name.

This conflict was retained and reported rather than silently changing the source data.

**Rating validation**

- Expected range: 1–5
- Invalid ratings found: **0**

**Recommended IND validation**

- Expected values: 0 or 1
- Invalid values found: **0**

### Part A Outputs

- `products.csv`
- `cleaned_reviews.csv`

## Part B — Rating and Recommendation Analysis

### 1. Average Rating and Recommendation Share

The average rating and percentage of recommended reviews were calculated for each department and class.

At the department level:

| Department | Average Rating | Recommended (%) |
| ---------- | -------------: | --------------: |
| Bottoms    |           4.29 |          85.13% |
| Dresses    |           4.15 |          80.82% |
| Intimate   |           4.28 |          85.01% |
| Jackets    |           4.26 |          83.62% |
| Tops       |           4.17 |          81.52% |
| Trend      |           3.82 |          73.95% |

The **Trend** department has the lowest average rating and lowest recommendation share among the departments.

### 2. Classes with the Most 1–2 Star Ratings

The largest numbers of 1–2 star reviews were found in:

| Class    | 1–2 Star Reviews |
| -------- | ---------------: |
| Dresses  |              689 |
| Knits    |              506 |
| Blouses  |              348 |
| Sweaters |              155 |
| Pants    |              124 |

These are counts rather than percentages, so they reflect the volume of low-rated reviews.

### 3. Rating by Reviewer Age Band

| Age Band | Average Rating | Review Count |
| -------- | -------------: | -----------: |
| ≤20      |           4.32 |          152 |
| 21–30    |           4.19 |        3,186 |
| 31–40    |           4.17 |        7,912 |
| 41–50    |           4.17 |        5,908 |
| 51–60    |           4.25 |        3,891 |
| 61+      |           4.29 |        2,437 |

The largest number of reviews comes from the **31–40** age band.

### Charts

The following charts were created:

1. Average Rating by Department
2. Share Recommended by Department
3. Low-Rating Count by Class
4. Average Rating by Age Band

### Part B Outputs

- `department_summary.csv`
- `class_summary.csv`
- `low_rating_counts_by_class.csv`
- `age_band_summary.csv`

## Part C — LLM Review Classification

### Sample Selection

There were **5,278 reviews rated 3 or below** in the dataset.

A random sample of **200 low-rated reviews** was selected using:

```python
low_rated.sample(n=200, random_state=42)
```

Of these 200 reviews:

- **193** contained review text and were sent to the LLM.
- **7** had missing review text and were excluded from LLM classification.

### Classification Labels

Each review was classified into exactly one main issue:

- `size/fit`
- `fabric/quality`
- `looks different from photo`
- `comfort`
- `price`
- `other`

The LLM was also asked to identify whether the item runs:

- `small`
- `large`
- `neither`

### LLM Method

The Groq API was used with the model:

`openai/gpt-oss-20b`

A single consistent prompt was used for all reviews. The model was instructed to return only:

```text
issue|fit
```

Temperature was set to `0` to make the classifications more consistent.

The raw LLM responses were retained in `llm_tagged_193_reviews.csv`.

### Manual Validation

A random sample of **30 reviews** was manually checked against the review text.

Results:

- **Issue accuracy: 83.33% (25/30)**
- **Fit accuracy: 86.67% (26/30)**
- **Overall exact-match accuracy: 80.0**

## Recommendations

Based on the rating analysis and the LLM classification of low-rated reviews, I would prioritize the following three product/listing changes:

### 1. Improve Size and Fit Guidance

Size/fit was the dominant issue in the low-rated review sample, particularly for Tops and Dresses.

**Recommended change:**

- Provide clearer fit descriptions.
- Include garment measurements.
- State whether an item tends to run small, large, or true to size.

### 2. Add More Detailed Product Fit Information

Reviews frequently mentioned specific fit problems involving length, bust, waist, armholes, and overall proportions.

**Recommended change:**

- Show model height and size worn.
- Provide key garment measurements.
- Add specific fit notes such as slim, relaxed, oversized, runs small, or runs large.

### 3. Improve Fabric and Quality Information

Fabric/quality was the second-largest issue category in the sample, with the highest counts in Tops and Dresses.

**Recommended change:**

- Make fabric composition more visible.
- Describe thickness, stretch, and feel.
- Include relevant care or shrinkage information where applicable.

## Project Structure

```text
Caspad_Project/
│
├── data/
│   └── Womens Clothing E-Commerce Reviews.csv
│
├── notebooks/
│   ├── 01_part_a.ipynb
│   ├── 02_part_b.ipynb
│   └── 03_part_c.ipynb
│
├── outputs/
│   ├── products.csv
│   ├── cleaned_reviews.csv
│   ├── department_summary.csv
│   ├── class_summary.csv
│   ├── low_rating_counts_by_class.csv
│   ├── age_band_summary.csv
│   ├── llm_tagged_193_reviews.csv
│   └── manual_30_check.csv
│
├── README.md
├── requirements.txt
├── .env
└── .gitignore
```

## How to Reproduce

1. Place the source CSV file in the `data/` folder.
2. Run `01_part_a.ipynb` for data preparation and validation.
3. Run `02_part_b.ipynb` for rating, recommendation, low-rating, and age-band analysis.
4. Run `03_part_c.ipynb` for low-rated review sampling and LLM classification.
5. The generated CSV results are saved in the `outputs/` folder.

The Groq API key is stored in the `.env` file and is not included in the project outputs or source code.
