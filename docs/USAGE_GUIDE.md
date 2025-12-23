# CrossBench Usage Guide

## 🚀 Quick Start

### Running CrossBench
```bash
# Navigate to project directory
cd "Task 6 Create a program for interactive web scrapping"

# Run the executable
./bin/scraper.exe

# Or compile and run
g++ -o bin/scraper.exe src/scraper.cpp -lwinhttp -std=c++17 -O2
./bin/scraper.exe
```

The program automatically:
1. Fetches latest data from api.zeroeval.com
2. Applies bias-adjusted scoring algorithms
3. Generates rankings across 10 specialized views
4. Exports to HTML, CSV, JSON, and text formats
5. Opens the interactive dashboard

---

## 🎨 Interactive Dashboard

### Accessing the Dashboard
After running the program, open:
```
output/leaderboard.html
```
in any modern web browser (Chrome, Firefox, Edge, Safari).

### Dashboard Features

#### 📑 Tab Navigation
The dashboard provides 10 specialized leaderboard views:

1. **Overall** - Comprehensive bias-adjusted ranking
   - Weighted composite of all metrics
   - Fair comparison across different model types
   
2. **Best Value** - Performance per dollar
   - Identifies cost-effective models
   - Ideal for budget-conscious selection
   
3. **Coding** - Software development capability
   - Programming benchmarks
   - Code generation quality
   
4. **Image Gen** - Visual generation quality
   - Image coherence and aesthetics
   - Prompt adherence
   
5. **Video Gen** - Temporal visual synthesis
   - Motion consistency
   - Physics simulation
   
6. **Speed** - Token generation throughput
   - Tokens per second
   - Latency performance
   
7. **Confidence** - Data reliability
   - Multi-benchmark validation
   - Signal strength
   
8. **Enterprise** - Business readiness
   - SLA guarantees
   - Organizational maturity
   
9. **Open Source** - Publicly available models
   - Community-driven models
   - Self-hostable options
   
10. **Ecosystem** - Market analytics
    - Organization market share
    - Performance by provider

#### 🖱️ Interactive Features

**Column Header Tooltips:**
Hover over any column header to see:
- What the metric measures
- How it's calculated
- What values mean
- Context specific to current tab

**Example Tooltips:**
- **Overall Tab + SCORE column**: "Composite score: Weighted average of reasoning, coding, creative, confidence, and price metrics (0-100 scale)"
- **Coding Tab + METRICS column**: "Coding Capability: Benchmark performance"
- **Speed Tab + SCORE column**: "Speed Score: Normalized throughput performance (0-100 scale)"

**Sort Controls:**
- Default: Authoritative ranking (with recency tie-breaker)
- Price: Low to High
- Speed: High to Low  
- Confidence: High to Low
### Overall Score Formula
```
Overall = (Core Score × 0.40) 
        + (Coding Score × 0.20)
        + (Creative Score × 0.15)
        + (Confidence × 0.15)
        + (Price Factor × 0.10)
```
All normalized to 0-100 scale.

### Value Score Formula
```
Value = (Performance Score)² / (1 + Price/10)
```
Higher is better. Rewards excellent performance while considering cost.

### Recency Tie-Breaker
When two models have scores within 0.5 points:
- Newer model (fewer days since release) ranks higher
- Ensures fairness and recognizes recent improvements

---

## 📁 Export Files Generated

### HTML Dashboard
**File:** `output/leaderboard.html`
- Interactive web interface
- 10 specialized tabs
- Hover tooltips
- Sort controls
- Ecosystem charts
- **Recommended for exploration**

### CSV Files (Spreadsheet Compatible)
**File:** `data/leaderboard_performance.csv`
```csv
Rank,Model,Organization,Score,Coding,Creative,Price,Confidence
1,GPT-5.2 Pro,OpenAI,90.5,88.1,96.9,10.00,92.5
```

**File:** `data/leaderboard_value.csv`
```csv
Rank,Model,Organization,Value Score,Performance,Price
1,Great Value Model,Provider X,425.0,85.0,2.00
```

**File:** `data/leaderboard_price.csv`
```csv
Rank,Model,Organization,Price,Performance Score
1,Free Model,Provider,Free,75.0
2,Budget Model,Provider,0.50,78.0
```

### JSON Format (Machine Readable)
**File:** `data/leaderboard_all.json`
```json
{
  "ecosystem": {
    "OpenAI": 12.51,
    "Google": 10.35,
    "Anthropic": 6.77
  },
  "models": [
    {
      "name": "GPT-5.2",
      "org": "OpenAI",
      "metrics": {
        "score": 92.4,
        "coding": 78.5,
        "creative": 100.0,
        "price": 8.0,
        "speed": 100.0
      },
      "ranks": {
        "overall": 90.1,
        "value": 1.46,
        "coding": 74.9,
        "speed": 62.0
      },
      "meta": {
        "confidence": 92.5,
        "is_enterprise": true,
        "is_open_source": false
      }
    }
  ]
}
```

### Text Format (Legacy)
**File:** `data/leaderboard_all.txt`
```
AI LEADERBOARD V8.5 (Fixed)
------------------
Rank 1: GPT-5.2 - OpenAI (92.40)
Rank 2: Gemini 3 Pro - Google (91.90)
...
```

---

## 🎮 Typical Workflow

1. **Run the executable**
   ```bash
   ./bin/scraper.exe
   ```

2. **Wait for processing** (typically 5-10 seconds)
   ```
   [Network] Connecting to api.zeroeval.com...
   [Processing] Found 250 models
   [Export] Generating files...
   ✓ Pipeline Complete
   ```

3. **Open the dashboard**
   - Locate `output/leaderboard.html`
   - Double-click or open in browser
   - Explore different tabs
   - Hover over headers for explanations

4. **Export for analysis**
   - Use CSV files for Excel/Google Sheets
   - Use JSON for programmatic access
   - Share text file for quick reference

---

## 💡 Pro Tips

### For Quick Decisions
- **Overall Tab**: Best general-purpose models
- **Value Tab**: Cost-effective choices
- **Confidence Filter**: Sort by confidence for reliable data

### For Specific Use Cases
- **Coding Tab**: Programming assistants
- **Image/Video Tabs**: Content generation
- **Speed Tab**: Real-time applications
- **Enterprise Tab**: Production deployments

### For Research
- **Ecosystem Tab**: Market trends
- **JSON Export**: Programmatic analysis
- **Confidence Scores**: Data quality assessment

---

## 🔍 Understanding Metrics

### When to prioritize different views:
**Overall Ranking:**
- General-purpose model selection
- Balanced performance across dimensions
- When you need versatility

**Value Ranking:**
- Budget constraints
- Cost optimization
- Startup/small team scenarios

**Coding Ranking:**
- Software development assistants
- Code generation tasks
- Technical documentation

**Speed Ranking:**
- Real-time applications
- High-throughput requirements
- Latency-sensitive use cases

**Enterprise Ranking:**
- Production deployments
- Mission-critical applications
- SLA requirements

**Open Source Ranking:**
- Self-hosting requirements
- Customization needs
- Privacy-focused deployments

---

## 🛠️ Troubleshooting

### Common Issues

**Issue:** Program fails to connect
```
[ERROR] Connection failed
```
**Solution:** Check internet connection and firewall settings

**Issue:** Dashboard not rendering correctly
**Solution:** Use a modern browser (Chrome 90+, Firefox 88+, Edge 90+)

**Issue:** Missing output files
**Solution:** Check `output/` and `data/` directories exist and have write permissions

**Issue:** JSON parsing error
**Solution:** API data format may have changed - check for updates

---

## 📞 Support

For issues, questions, or feature requests:
- Check the main [README.md](README.md) for detailed documentation
- Review console output for error messages
- Ensure all dependencies are properly installed

---

**CrossBench** - Making AI model comparison faster, fairer, and more informed.
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
