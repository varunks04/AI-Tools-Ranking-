# Quick Reference Guide - Enhanced Features

## 🎯 User Input Feature

When you run the scraper and select option 1 (Fetch Model Leaderboard), you'll be prompted:

```
How many top models to display? (Enter number, default 20):
```

**Examples:**
- Enter `10` to see top 10 models in each ranking
- Enter `50` to see top 50 models
- Press Enter (leave blank) for default 20 models
- Enter any number up to the total available models

---

## 📊 Three Ranking Tables

### 1️⃣ Performance Ranking (GPQA High → Low)
Shows the best performing models based on GPQA benchmark scores.

**Example Output:**
```
==================================================================================
  RANKING #1: TOP MODELS BY PERFORMANCE (GPQA Score - High to Low)
==================================================================================

Rank  Model                        Organization  GPQA
----------------------------------------------------------------------------------
1     GPT-5.2 Pro                  OpenAI        0.932
2     GPT-5.2                      OpenAI        0.924
3     Gemini 3 Pro                 Google        0.919
...
```

### 2️⃣ Price Ranking (Low → High)
Shows models sorted by price from cheapest to most expensive.

**Example Output:**
```
==================================================================================
  RANKING #2: TOP MODELS BY PRICE (Low to High)
==================================================================================

Rank  Model                        Organization  GPQA      Price
----------------------------------------------------------------------------------
1     Free Model 1                 Provider A    0.750     Free
2     Budget Model 2               Provider B    0.780     $0.50
3     Affordable Model 3           Provider C    0.800     $1.00
...
```

### 3️⃣ Best Value Ranking (Performance-to-Price Ratio)
Shows models with the best balance of performance and cost.

**Formula:** Value Score = GPQA Score / (Price + 0.01)

**Example Output:**
```
==================================================================================
  RANKING #3: BEST VALUE MODELS (Performance-to-Price Ratio)
==================================================================================

Rank  Model                        Organization  GPQA      Price
----------------------------------------------------------------------------------
1     Great Value Model            Provider X    0.850     $2.00
2     Budget Performer             Provider Y    0.800     $1.50
3     Efficient Model              Provider Z    0.820     $2.50
...
```

---

## 📈 Statistics Summary

After displaying all rankings, you'll see a summary:

```
==================================================================================
  STATISTICS SUMMARY
==================================================================================
Total Models Analyzed: 250
Top Performer: GPT-5.2 Pro (GPQA: 0.932)
Best Price: Free Model 1 (Free)
Best Value: Great Value Model (Performance/Price)
==================================================================================
```

---

## 📁 Export Files Generated

### Text Format
**File:** `output.txt`
- Contains all 3 rankings in formatted tables
- Human-readable
- Easy to share via email or documents

### CSV Formats (Spreadsheet Compatible)
**File:** `leaderboard_performance.csv`
```csv
Rank,Model,Organization,GPQA Score,Input Price
1,"GPT-5.2 Pro","OpenAI",0.932,"$10.00"
2,"GPT-5.2","OpenAI",0.924,"$8.00"
```

**File:** `leaderboard_price.csv`
```csv
Rank,Model,Organization,GPQA Score,Input Price,Price Value
1,"Free Model","Provider",0.750,"Free",0.0000
2,"Budget Model","Provider",0.780,"$0.50",0.5000
```

**File:** `leaderboard_value.csv`
```csv
Rank,Model,Organization,GPQA Score,Input Price,Value Score
1,"Value Model","Provider",0.850,"$2.00",425.00
2,"Efficient Model","Provider",0.820,"$2.50",328.00
```

### JSON Format (Machine Readable)
**File:** `leaderboard_all.json`
```json
{
  "metadata": {
    "generated": "Dec 23 2025 18:00:00",
    "total_models": 250
  },
  "performance_ranking": [
    {"rank":1,"model":"GPT-5.2 Pro","organization":"OpenAI","gpqa_score":0.932,"input_price":"$10.00"},
    ...
  ],
  "price_ranking": [
    {"rank":1,"model":"Free Model","organization":"Provider","gpqa_score":0.750,"input_price":"Free","price_value":0.0},
    ...
  ],
  "value_ranking": [
    {"rank":1,"model":"Value Model","organization":"Provider","gpqa_score":0.850,"input_price":"$2.00","value_score":425.00},
    ...
  ]
}
```

---

## 🎮 Usage Workflow

1. **Start the program**
   ```bash
   ./scraper.exe
   ```

2. **Select Option 1** from menu
   ```
   Select option [1-4]: 1
   ```

3. **Enter display count**
   ```
   How many top models to display? (Enter number, default 20): 30
   ```

4. **View results** - You'll see:
   - Progress indicators during download
   - 3 different ranking tables (top 30 in each)
   - Statistics summary
   - Export confirmation messages

5. **Check exported files** - All data saved in:
   - `output.txt` - All rankings
   - `leaderboard_performance.csv` - Performance ranking
   - `leaderboard_price.csv` - Price ranking
   - `leaderboard_value.csv` - Value ranking
   - `leaderboard_all.json` - Comprehensive JSON

---

## 💡 Tips

- **Large Datasets:** Enter a high number (e.g., 100) to see more models
- **Quick Overview:** Use default 20 for a quick overview
- **Spreadsheet Analysis:** Open CSV files in Excel/Google Sheets for filtering and sorting
- **Programming:** Use the JSON file for programmatic analysis
- **Sharing:** Use output.txt for easy sharing of all rankings

---

## 🔍 Understanding the Rankings

### When to use Performance Ranking:
- You need the absolute best model regardless of cost
- Benchmarking your application against top performers
- Research and comparison purposes

### When to use Price Ranking:
- Budget is your primary concern
- Looking for free or low-cost alternatives
- Cost-sensitive deployments

### When to use Value Ranking:
- You want the best bang for your buck
- Balancing performance with budget constraints
- Finding efficient models for production use

---

**Version:** 2.0  
**Last Updated:** December 23, 2025
