# Steam Project - Exploratory Data Analysis (EDA)

The goal of this project is to conduct a global analysis of the games available on Steam's marketplace in order to better understand the videogames ecosystem and today's trends. We want to understand what factors affect the popularity or sales of a video game to better prepare the release of a new revolutionary videogame. 

## DATABRICKS ACCES
URL for Databricks notebook with outputs : https://dbc-4ce177e6-a8bb.cloud.databricks.com/browse/folders/256899025860107?o=4499755976569818

## Overview

This project conducts a comprehensive **Exploratory Data Analysis (EDA)** of the Steam gaming platform, examining a dataset of **55,000+ games** to understand market dynamics and identify key success factors.

The analysis provides actionable insights for game publishers, developers, and business strategists looking to optimize game releases on Steam's marketplace.



## Objectives

The analysis addresses the following business questions:

- **Publisher Analysis:** What characterizes successful publishers? Is size correlated with quality?
- **Game Performance:** Which games perform best? What makes a game successful?
- **Market Segmentation:** How do genres, categories, and tags influence success?
- **Pricing Strategy:** What pricing model is optimal? How do discounts impact sales perception?
- **Language & Accessibility:** How important is localization? Which languages matter?
- **Technical Requirements:** What platforms are essential? Is multi-platform support necessary?
- **Temporal Trends:** How has the market evolved? Are there seasonal patterns?



## Dataset

### Source & Format
- **Location:** S3 bucket (`s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json`)
- **Format:** Nested JSON structure with complex data types (StructType, ArrayType)
- **Size:** 55,000+ games with 2 columns: `id` and `data` (nested)

### Key Variables

| Category | Variables |
|----------|-----------|
| **Identifiers** | appid, name, publisher, developer |
| **Content** | genres, categories, tags, type |
| **Engagement** | positive reviews, negative reviews, ccu (concurrent users) |
| **Pricing** | price, initialprice, discount |
| **Metadata** | languages, platforms, required_age, release_date, website |
| **Platform Support** | windows, mac, linux |

### Data Quality
- **Total Games:** 55,000+
- **Publishers:** 10,000+ (58% with only 1 game)
- **Null Values:** Minimal for standard columns; tags have high NULLs (expected — absence ≠ missing)
- **Duplicates:** No complete duplicates; 261 games with same name but different appid (versions)



## Project Structure

```
Steam_project/
├── Steam_project_no_output.ipynb      # Main analysis notebook
├── README.md                           # This file
└── requirements.txt
```



## Data Preparation

### Step 1: Importing & Exploring
- Loaded JSON from S3 using PySpark
- Analyzed nested schema structure
- Identified 3 levels of nesting (data → appid, categories, platforms, tags, etc.)

### Step 2: Flattening Nested Structure
**Challenge:** JSON data contained nested StructTypes (platforms, tags) and ArrayTypes (categories)

**Solution:**
- Created `get_nested_columns()` function to recursively extract all nested fields
- Used dot notation for StructTypes (e.g., `data.platforms.windows`)
- Handled special characters in column names (e.g., tags like "1980s") with backtick escaping

### Step 3: NULL Value Analysis

**Key Insight:** Different column types require different NULL handling strategies:

| Type | Interpretation | Action |
|------|---|---|
| **Classical Variables** | Missing data → NULL % analysis | Drop if >80% |
| **Structured Tags** | Absent feature → 0 (not missing) | Replace NULL with 0 |
| **Array Columns** | Explode and count frequency | Keep NULL-aware in analysis |

**Result:** No missing data in standard columns; 228 rare tags (<1% frequency) removed.

### Step 4: Type Conversions
- **Prices:** String → Float (price, initialprice, discount)
- **Age:** String → Integer (required_age)
- **Dates:** String → Date (release_date), extracted year/month

### Step 5: Feature Engineering
Created derived columns:
- **total_reviews** = positive + negative reviews
- **rating_score** = positive / total_reviews (null if no reviews)
- **has_discount** = boolean flag for discounted games
- **bayesian_score** = weighted rating accounting for review volume


## Analysis Sections

### I. Publications Analysis

#### 1.1 Publisher Characteristics

**Key Metrics:**
- **Total Publishers:** 10,000+
- **Top Publisher:** Big Fish Games (422 games)
- **Median:** 1 game per publisher
- **Market Concentration:** Top 5 publishers = ~700 games (1.3% of catalog)

**Insight:** Highly fragmented market where one-hit publishers dominate, yet quality can emerge from anywhere.

#### 1.2 Publisher Ratings (Bayesian Scoring)

**Formula:**
$$\text{Bayesian Score} = \frac{v}{v+m}R + \frac{m}{v+m}C$$

Where:
- R = average rating
- v = number of reviews
- m = 80th percentile of reviews (threshold for reliability)
- C = global average rating

**Top 5 Publishers by Bayesian Score:**
1. Adamgryu (100% score, 1 game)
2. Igara Studio (100%, 1 game)
3. Studio Minus (100%, 1 game)
4. poncle (99%, 1 game)
5. HIKARI FIELD (98%, 1 game)

**Insight:** Small publishers with ONE highly-polished game outrank large publishers with many titles.

#### 1.3 Temporal Trends

**Release Evolution (1997–2022):**
- **1997–2013:** Steady growth (2 → 500 games/year)
- **2013–2021:** Exponential boom (500 → 8,823 games/year, peak in 2021)
- **2022:** Stabilization (~7,400 games/year)

**Monthly Distribution:**
- **Peak:** October (5,500+ releases) — pre-holiday strategy
- **Lowest:** January, December, June (holiday/slowdown effects)

---

### II. Games Analysis

#### 2.1 Best-Rated Games

**Top 5 by Bayesian Score:**
1. **People Playground** (Studio Minus) — 10,000+ positive reviews
2. **Aseprite** (Igara Studio) — Niche tool, high engagement
3. **Portal 2** (Valve) — AAA benchmark
4. **A Short Hike** (adamgryu) — Indie success story
5. **Vampire Survivors** (poncle) — Viral indie hit

**Pattern:** Mix of indie darlings and AAA classics; quality transcends publisher size.

#### 2.2 Language & Localization

**Language Distribution:**
- **English-only:** 86% of games (de facto standard)
- **Top additional languages:** Russian (5.6%), Simplified Chinese, Japanese

**Localization Impact:**
- Games with **1–2 languages:** Low reviews (median ~100)
- Games with **10+ languages:** 10x more reviews
- **Correlation:** More languages = higher Bayesian score

**Insight:** Language accessibility is both a commercial strategy AND a quality signal.

---

### III. Genres, Categories & Tags Analysis

#### 3.1 Genre Distribution

**Top 5 Genres (% of games):**
1. **Indie** — 25% (dominant category)
2. **Action** — 15%
3. **Casual** — 14%
4. **Adventure** — 13%
5. **Strategy** — 7%

**Top Genre Combinations:**
- Indie + Action (most common)
- Indie + Casual
- Indie + Action + Adventure

**Insight:** Indie games define the Steam ecosystem.

#### 3.2 Quality by Genre

**Average Bayesian Score by Genre:** 0.73–0.75 (across all genres)

**Conclusion:** Genre does NOT determine quality. A well-made Casual game scores as well as a well-made RPG.

#### 3.3 Categories

**Top 5 Categories (% of games):**
1. **Single-player** — 27.1%
2. **Steam Achievements** — 14.2%
3. **Steam Cloud** — 7.4%
4. **Full controller support** — 6.2%
5. **Multi-player** — 6.0%

**Insight:** Players expect single-player as baseline; achievements/cloud are nice-to-haves.

#### 3.4 Tags

**Most Frequent Tags:**
- Indie, Singleplayer, Action, Casual, Adventure (>20% of games)

**Tags Removed:** 228 rare tags (<1% frequency) were filtered out in data preparation.

---

### IV. Prices Analysis

#### 4.1 Pricing Distribution

**Free Games:** ~2% of catalog (700+ games)

**Paid Games Distribution:**
- **Median Price:** 599 (currency units)
- **99th Percentile:** 4,999
- **Tail:** Long-tail distribution with outliers (collector editions)

**Price Segmentation (using percentiles):**
| Segment | Range | % of Games |
|---------|-------|-----------|
| Very Low | 0–25th %ile | 25% |
| Low | 25–50th %ile | 25% |
| Mid | 50–75th %ile | 25% |
| High | 75–90th %ile | 15% |
| Premium | 90–100th %ile | 10% |

**Insight:** Over 50% of paid games are low-to-mid priced, indicating volume strategy dominates.

#### 4.2 Discounts

**Discount Prevalence:**
- Only **~10% of games** have active discounts
- Most common discounts: 50%, 80%, 90% (round numbers)

**Discount by Price Segment:**
| Segment | % Discounted |
|---------|------------|
| Very Low | 45% |
| Low | 20% |
| Mid | 10% |
| High | 8% |
| Premium | <7% |

**Insight:** Discounts are a volume strategy for cheap games, NOT a premium positioning tool.

#### 4.3 Temporal Price Evolution

**Early Years (1997–2005):** Volatile (1500 → 500 → 1500)
**2006–2022:** Stable median; occasional spikes in premium segment (2012, 2013, 2019)

**Conclusion:** Market pricing matured after 2006; AAA titles drive 95th percentile.

---

### V. Technologies Analysis

#### 5.1 Platform Support

**Distribution:**
- **Windows only** — 74.1% (dominant)
- **Windows + Mac + Linux** — 12.2% (full multi-platform)
- **Windows + Mac** — 10.7%
- **Other combinations** — <3%
- **Mac only / Linux only** — <0.05% (negligible)

**Platform Coverage:**
- **Windows:** 99.97% of all games
- **Mac:** 22.9% (only with Windows)
- **Linux:** 12.2% (only with Windows + Mac)

**Insight:** Windows is non-negotiable. Multi-platform is optional and seen in ~25% of titles.

#### 5.2 Platform & Quality

**Average Rating by Platform Support:**
- **Windows games:** 73.0%
- **Mac games:** 74.2%
- **Linux games:** 74.5%

**Caveat:** Selection effect — larger, more successful games are more likely to receive Mac/Linux ports.



## Key Findings

### Market Structure
- Highly fragmented (10,000+ publishers, 58% with 1 game)  
- Exponential growth (2013–2021), then stabilization  
- October is the peak release month (holiday season strategy)

### Success Factors
- **Quality > Quantity** — Small studios with polished games outperform volume publishers  
- **Genre agnostic** — Quality score consistent across all genres (~0.73)  
- **Localization matters** — Games in multiple languages get 3x more engagement  

### Market Segmentation
- **Indie is dominant** — 25% of all games, defines Steam culture  
- **Indie + Action/Casual** — Most popular combinations  
- **Single-player expected** — 27% of games; achievements valued

### Pricing Insights
- **Long-tail distribution** — Median = 599; most games cheap-to-mid range  
- **Discounts are rare** — Only 10% of games; primarily for cheap titles  
- **Free games niche** — Only 2% of catalog; separate business model

### Technical Reality
- **Windows mandatory** — 99.97% support essential; no exclusivity elsewhere  
- **Multi-platform optional** — Feature of successful titles, not driver  
- **Mac/Linux afterthought** — Always bundled with Windows support



## Recommendations for New Game Release

### 1. **Prioritize Quality Over Volume**
- Focus on ONE well-crafted game rather than multiple mediocre ones
- Best-rated publishers (Adamgryu, poncle) have single, polished releases
- Quality compounds over time (better reviews → more visibility → more sales)

### 2. **Select Winning Genre Combination**
- **Primary choice:** Indie + Action or Indie + Casual
- These combinations represent >40% of all games and have established player demand
- Avoid oversaturated niches; test with existing player communities first

### 3. **Time Release Strategically**
- **Release month:** October (peak visibility, pre-holiday consumer spending)
- Avoid January, June, December (seasonal slowdowns)
- Plan marketing 2–3 months in advance

### 4. **Invest in Localization**
- **Minimum:** English (non-negotiable)
- **High ROI:** Russian (5.6% of market), Simplified Chinese, Japanese
- Games with 3+ languages see significantly higher engagement

### 5. **Develop for Windows First, Others Later**
- **Target:** Windows from launch (99.97% requirement)
- **Optional:** Add Mac + Linux post-launch if successful
- Multi-platform adds <2% to addressable market initially

### 6. **Price Competitively**
- **Target range:** 200–1000 (low-to-mid segment where 50% of paid games sit)
- Avoid premium pricing (>2000) unless AAA-quality or established franchise
- Use occasional discounts (50%) for volume boosts, not regular strategy

### 7. **Ensure Core Features**
- **Must-have:** Single-player support
- **Nice-to-have:** Steam Achievements, Cloud saves
- **Optional:** Controller support (6% of games have it, but valued)



## Technical Implementation

### Technology Stack
- **Platform:** Databricks with PySpark (Spark 3.x)
- **Language:** Python 3.8+
- **Data Source:** S3 (JSON format)
- **Processing:** PySpark SQL & DataFrame API

### Key Techniques Used

#### 1. Nested Data Flattening
```python
def get_nested_columns(schema, prefix=""):
    """Recursively extract all nested columns from schema"""
    columns = []
    for field in schema.fields:
        valid_field_name = f"`{field.name}`"
        name = f"{prefix}.{valid_field_name}" if prefix else valid_field_name
        
        if isinstance(field.dataType, StructType):
            columns += get_nested_columns(field.dataType, name)
        else:
            columns.append(name)
    return columns
```

#### 2. NULL Handling Strategy
- **Classical columns:** Standard NULL % analysis
- **Structured fields:** Replace NULL with 0 (represents absence)
- **Arrays:** Explode and count frequency separately

#### 3. Bayesian Rating (Laplace Smoothing)
Weights user ratings by review volume to prevent small-sample bias:
$$\text{Score} = \frac{v}{v+m}R + \frac{m}{v+m}C$$

#### 4. Date Parsing with Multiple Formats
```python
.withColumn(
    "parsed_date",
    F.coalesce(
        F.to_date(F.col("cleaned_dates"), "yyyy/MM/d"),
        F.to_date(F.col("cleaned_dates"), "yyyy/MM/dd"),
        F.to_date(F.col("cleaned_dates"), "yyyy/MM")
    )
)
```

#### 5. Quantile-based Segmentation
Used `approxQuantile()` to segment prices while respecting skewed distribution.



## Limitations

### Data Limitations
- **No Sales Revenue:** Analysis uses review count as proxy for popularity (imperfect correlation)
- **No Churn Data:** Cannot measure long-term success or player retention
- **Currency Unknown:** Prices are in unspecified units; absolute values not comparable to external benchmarks
- **Temporal Snapshot:** Dataset reflects a point in time; trends observed may not persist

### Methodological Limitations
- **Review Bias:** Only engaged players leave reviews; data skews toward polarized opinions
- **Self-Reported Metadata:** Genres/tags assigned by publishers (potential gaming of tags)
- **Selection Effects:** Multi-platform support appears in successful games, but causality unclear
- **Survivorship Bias:** Removed/delisted games are not in dataset

### Analysis Constraints
- **Early Years Sparse:** 1997–2005 based on <50 games/year; trends unreliable
- **Tags Redundant with Genres:** 228 tags removed due to rarity; may hide niche signals
- **Language Confounding:** Language support correlates with game size/budget, not a pure accessibility effect


## Data Privacy & Ethics

- **Personal Data:** None present (no user-level data)
- **Commercial Data:** Aggregate market metrics only
- **Source:** Public Steam marketplace data
- **Usage:** Academic/business intelligence only



## Tools & Technologies

| Component | Tool | Version |
|-----------|------|---------|
| **Compute** | Databricks | Runtime 11.x |
| **Processing** | Apache Spark | 3.2+ |
| **Language** | Python | 3.8+ |
| **Visualization** | Databricks Display | Built-in |
| **Storage** | AWS S3 | Native |
| **Format** | Parquet / CSV | For outputs |

### Dependencies
```python
import pyspark.sql.functions as F
from pyspark.sql.types import StructType, StringType, LongType, DoubleType, ArrayType
import datetime
```


---

### Author
Mounia Tonazzini Agricultural Engineer & Data Scientist | AI & AgriTech

Location: Montpellier, France
LinkedIn: www.linkedin.com/in/mounia-tonazzini
GitHub: Mounia-Agronomist-Datascientist

**Last Updated:** April 2026:
This project is part of the Jedha Bootcam certification 
