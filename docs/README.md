# CrossBench - AI Model Leaderboard Aggregator

## 📋 Overview
**A Bias-Adjusted Aggregation of Multiple AI Leaderboards to Help You Compare Models Faster and Make Informed Decisions**

CrossBench is an advanced C++ web scraping and analytics tool that fetches, aggregates, and analyzes AI model performance data from multiple authoritative benchmarks. It provides comprehensive rankings with bias-adjusted scoring, interactive visualizations, and multi-format exports.

## 🎯 Mission
CrossBench addresses the challenge of comparing AI models across fragmented leaderboards by:
- Aggregating data from multiple authoritative sources
- Applying bias-adjusted scoring algorithms
- Providing intuitive visualizations for quick decision-making
- Enabling easy model comparison across 10+ performance dimensions

## ✨ Features

### Core Functionality
- **Live API Data Fetching**: Real-time model performance from api.zeroeval.com
- **Multi-Tab Leaderboards**: 10 specialized views (Overall, Coding, Speed, Value, Enterprise, etc.)
- **Interactive Dashboard**: Beautiful HTML interface with dark mode and responsive design
- **Bias-Adjusted Rankings**: Fair scoring with recency tie-breakers and multi-source validation
- **Ecosystem Analytics**: Market share analysis and organizational performance tracking

### Advanced Features
- **Dynamic Tooltips**: Hover over column headers for detailed metric explanations that change per tab
- **Confidence Scoring**: Data reliability indicators based on multi-benchmark validation
- **Multi-Format Export**:
  - 📊 CSV format (Performance, Value, Price leaderboards)
  - 🔧 JSON format (Complete dataset with metadata)
  - 📄 Text format (Legacy plain-text rankings)
  - 🌐 HTML Dashboard (Interactive web interface)
- **Enhanced Error Handling**: Retry mechanism with exponential backoff
- **Performance Sorting**: Multiple sort modes (Default, Price, Speed, Confidence)

## 🛠️ Technical Details

### Dependencies
- **WinHTTP**: Windows HTTP Services for network requests
- **nlohmann/json**: Modern JSON library for C++ (json.hpp)
- **Windows SDK**: Required for WinHTTP functionality
- **Chart.js**: JavaScript charting library for visualizations
- **TailwindCSS**: Utility-first CSS framework for UI

### Build Requirements
- C++17 or higher
- Windows OS
- Visual Studio 2019+ or MinGW-w64 compiler
- winhttp.lib (automatically linked)

### Compilation
```bash
# Using g++
g++ -o scraper.exe src/scraper.cpp -lwinhttp -std=c++17 -O2

# Using MSVC
cl /EHsc /std:c++17 /O2 src/scraper.cpp winhttp.lib
```

## 🚀 Usage

### Running the Program
```bash
./bin/scraper.exe
```

### Output Files
The program automatically generates:
- `output/leaderboard.html` - **Main interactive dashboard** (Open in browser)
- `data/leaderboard_all.json` - Complete dataset with all models
- `data/leaderboard_performance.csv` - Performance-focused rankings
- `data/leaderboard_value.csv` - Best value models
- `data/leaderboard_price.csv` - Price-sorted listings
- `data/leaderboard_all.txt` - Legacy text format

### Dashboard Features
Open `output/leaderboard.html` in any modern browser to access:
- **10 Specialized Tabs**: Overall, Value, Coding, Image, Video, Speed, Confidence, Enterprise, Open Source, Ecosystem
- **Interactive Tooltips**: Hover over column headers for metric explanations
- **Sort Controls**: Multiple sorting options per view
- **Confidence Bars**: Visual reliability indicators
- **Ecosystem Charts**: Market share and organizational analytics

## 📊 Data Structure

### ModelEntity
```cpp
class ModelEntity {
    std::string name;                    // Model name (e.g., "GPT-5.2")
    std::string organization;            // Organization (e.g., "OpenAI")
    std::set<Modality> modalities;       // Text, Image, Video capabilities
    PerformanceMetrics metrics;          // Detailed performance data
    RankScores ranks;                    // Computed rankings per tab
    std::vector<Signal> signals;         // Multi-source validation signals
    double final_score;                  // Aggregated performance score
    double confidence_score;             // Data reliability (0-100)
    std::string confidence_reason;       // Explanation of confidence level
};
```

### PerformanceMetrics
```cpp
struct PerformanceMetrics {
    double reasoning_score;          // Core reasoning capability (0-1)
    double coding_score;            // Programming benchmark (0-1)
    double creative_score;          // Creative/generation quality (0-1)
    double price_input_1m;          // Cost per 1M input tokens
    double tokens_per_sec;          // Throughput speed
    int last_updated_days_ago;      // Freshness indicator
    bool is_open_source;            // Weight availability
    bool is_enterprise_ready;       // SLA/reliability
    // ... additional metrics
};
```

### RankScores (All on 0-100 scale)
```cpp
struct RankScores {
    double overall;      // Composite index
    double value;        // Performance/price ratio
    double coding;       // Programming capability
    double image;        // Image generation
    double video;        // Video generation
    double speed;        // Token throughput
    double confidence;   // Data reliability
    double enterprise;   // Business readiness
};
```

## 🔧 Configuration

### Constants (configurable in Config namespace)
```cpp
namespace Config {
    const std::wstring API_DOMAIN = L"api.zeroeval.com";
    const std::wstring API_PATH = L"/leaderboard/models/full?justCanonicals=true";
    const int MAX_RETRIES = 3;
    const int RETRY_DELAY_MS = 2000;
    const std::string OUTPUT_DIR = "output";
    const std::string DATA_DIR = "data";
    
    // Ranking Weights (customizable)
    namespace Weights {
        constexpr double OVERALL_CORE = 0.40;
        constexpr double OVERALL_CODING = 0.20;
        constexpr double OVERALL_CREATIVE = 0.15;
        constexpr double OVERALL_CONFIDENCE = 0.15;
        constexpr double OVERALL_PRICE = 0.10;
    }
}
```

## 🎯 Key Features & Improvements

✅ **Bias-Adjusted Scoring**: Multi-source validation prevents single-benchmark bias
✅ **10 Leaderboard Views**: Specialized rankings for different use cases
✅ **Interactive Tooltips**: Context-aware explanations for each metric
✅ **Confidence Scoring**: Transparency in data reliability
✅ **Ecosystem Analytics**: Market insights and organizational trends
✅ **Responsive UI**: Beautiful dark mode interface with hover effects
✅ **Recency Tie-Breaker**: Fair ranking for models with similar scores
✅ **Multi-Format Export**: CSV, JSON, HTML, and text outputs
✅ **Real-Time Data**: Always fetches latest benchmark results
✅ **Error Resilience**: Automatic retry with exponential backoff

## 📈 Performance

- **Network Requests**: Efficient WinHTTP with retry logic
- **Memory Management**: RAII pattern ensures zero memory leaks
- **Response Handling**: Tested with 1MB+ JSON responses
- **Parsing Speed**: Optimized JSON parsing with nlohmann library
- **Dashboard Rendering**: Client-side rendering for instant tab switching
- **Data Processing**: Handles 250+ models in real-time

## 🐛 Error Handling

CrossBench handles various error scenarios gracefully:
- Network connection failures with automatic retry
- HTTP request errors (4xx, 5xx status codes)
- JSON parsing errors with detailed diagnostics
- File I/O errors with fallback mechanisms
- Missing or invalid data fields

## 🔒 Security Considerations

- Uses HTTPS for all network communication
- Validates JSON data structure before processing
- No sensitive data storage or transmission
- Safe string handling prevents buffer overflows
- Read-only data fetching (no POST/PUT operations)

## 📝 Sample Output

```
=== CrossBench - AI Model Leaderboard Aggregator ===
A Bias-Adjusted Aggregation of Multiple AI Leaderboards
Live Data Source: api.zeroeval.com
All Metrics Computed Dynamically

[Network] Connecting to api.zeroeval.com...
[Network] Request sent, awaiting response...
[Network] Downloaded 145.2 KB
[SUCCESS] Data fetched successfully

[Processing] Parsing JSON response...
[Processing] Found 250 models
[Processing] Applying bias-adjusted scoring...
[Processing] Computing rankings across 10 views...

[Export] Generating HTML dashboard...
[Export] Exporting CSV files...
[Export] Exporting JSON dataset...

✓ Pipeline Complete
  Dashboard: output/leaderboard.html
  Data Files: data/leaderboard_*.{csv,json}
```

## 🌟 Dashboard Screenshots

The interactive HTML dashboard features:
- **Header**: CrossBench branding with descriptive tagline
- **Tab Navigation**: 10 specialized leaderboard views
- **Data Table**: Sortable rankings with confidence bars
- **Tooltips**: Hover over column headers for metric explanations
- **Ecosystem Charts**: Doughnut and bar charts for market analysis

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Report bugs and issues
- Suggest new features or improvements
- Submit pull requests
- Improve documentation

## 📜 License

This project is open-source and available for educational and research purposes.

## 👨‍💻 Version History

- **v8.5** (2025) - CrossBench rebrand, enhanced tooltips, improved UI/UX
- **v8.0** (2025) - Multi-tab leaderboards, bias-adjusted scoring, ecosystem analytics
- **v7.0** (2024) - Confidence scoring, HTML dashboard, interactive visualizations
- **v6.0** (2024) - Multi-format export, performance optimizations
- **v1.0** (2024) - Initial release with basic API scraping

## 🔗 Resources

- [WinHTTP Documentation](https://learn.microsoft.com/en-us/windows/win32/winhttp/winhttp-start-page)
- [nlohmann/json Library](https://github.com/nlohmann/json)
- [Chart.js Documentation](https://www.chartjs.org/docs/)
- [TailwindCSS](https://tailwindcss.com/)
- [ZeroEval API](https://api.zeroeval.com)

## 🎓 Use Cases

CrossBench is ideal for:
- **ML Researchers**: Compare models for specific tasks
- **Enterprise Teams**: Evaluate models for production deployment
- **Cost Optimization**: Find best value models for your budget
- **Market Analysis**: Track AI ecosystem trends
- **Model Selection**: Make data-driven decisions quickly

---

**CrossBench** - Making AI model comparison faster, fairer, and more informed.

**Note**: This tool aggregates publicly available benchmark data for educational and research purposes. Always verify results and respect data source terms of service.
