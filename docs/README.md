# LLM Stats Interactive Web Scraper

## 📋 Overview
An advanced C++ web scraping tool designed to fetch and analyze Large Language Model (LLM) statistics and leaderboards. This interactive program scrapes data from multiple sources and provides comprehensive rankings with export capabilities.

## ✨ Features

### Core Functionality
- **API Data Fetching**: Retrieves model performance data from api.zeroeval.com
- **HTML Web Scraping**: Discovers leaderboard pages from llm-stats.com
- **Interactive Menu System**: User-friendly CLI interface with visual enhancements
- **Retry Mechanism**: Automatic retry with configurable attempts (MAX_RETRIES = 3)
- **RAII Pattern**: Safe resource management with WinHTTP handles

### Advanced Features
- **Progress Indicators**: Real-time download progress display
- **Multi-Format Export**:
  - 📄 Text format (`output.txt`)
  - 📊 CSV format (`leaderboard.csv`)
  - 🔧 JSON format (`leaderboard.json`)
- **Enhanced Error Handling**: Detailed error messages with color-coded logging
- **Performance Sorting**: Ranks models by GPQA scores
- **Price Parsing**: Intelligent price extraction and normalization

## 🛠️ Technical Details

### Dependencies
- **WinHTTP**: Windows HTTP Services for network requests
- **nlohmann/json**: Modern JSON library for C++ (json.hpp)
- **Windows SDK**: Required for WinHTTP functionality

### Build Requirements
- C++11 or higher
- Windows OS
- Visual Studio or MinGW compiler
- winhttp.lib (automatically linked)

### Compilation
```bash
# Using g++
g++ -o scraper.exe scraper.cpp -lwinhttp -std=c++11

# Using MSVC
cl /EHsc scraper.cpp winhttp.lib
```

## 🚀 Usage

### Running the Program
```bash
./scraper.exe
```

### Menu Options
1. **Fetch Model Leaderboard (API)** - Downloads and displays top LLM models
2. **Discover Leaderboards (HTML)** - Scrapes available leaderboard links
3. **View Exported Files** - Lists all generated export files
4. **Exit** - Cleanly exits the application

### Output Files
- `output.txt` - Formatted table of all models
- `leaderboard.csv` - CSV export for spreadsheet applications
- `leaderboard.json` - JSON format for programmatic access

## 📊 Data Structure

### ModelEntry
```cpp
struct ModelEntry {
    std::string name;              // Model name (e.g., "GPT-5.2 Pro")
    std::string organization;      // Organization (e.g., "OpenAI")
    double gpqa_score;            // GPQA benchmark score
    std::string input_price_str;   // Price string representation
    double input_price_val;        // Parsed price value for sorting
};
```

## 🔧 Configuration

### Constants (can be modified in source)
```cpp
const std::wstring API_DOMAIN = L"api.zeroeval.com";
const std::wstring API_PATH = L"/leaderboard/models/full?justCanonicals=true";
const std::wstring WEB_DOMAIN = L"llm-stats.com";
const int MAX_RETRIES = 3;
const int RETRY_DELAY_MS = 1000;
```

## 🎯 Key Improvements from v1.0

✅ **Enhanced Logging**: Color-coded, structured log messages
✅ **Export Functionality**: CSV and JSON export added
✅ **Progress Tracking**: Real-time download progress indicators
✅ **Better Error Handling**: Detailed error descriptions
✅ **Improved UI**: Beautiful ASCII-art menu interface
✅ **Code Organization**: Better structured with clear sections
✅ **Resource Management**: Improved RAII patterns

## 📈 Performance

- **Network Requests**: Asynchronous with retry logic
- **Memory Management**: RAII ensures no memory leaks
- **Response Size**: Handles large responses (tested up to 1MB+)
- **Parsing Speed**: Efficient JSON parsing with nlohmann/json

## 🐛 Error Handling

The application handles various error scenarios:
- Network connection failures
- HTTP request errors
- JSON parsing errors
- File I/O errors
- Invalid user input

## 🔒 Security Considerations

- Uses HTTPS for all network requests
- Validates downloaded data before parsing
- No sensitive data is stored or transmitted
- Safe string handling to prevent buffer overflows

## 📝 Sample Output

```
┌─────────────────────────────────────────┐
│           MAIN MENU                     │
├─────────────────────────────────────────┤
│ 1. Fetch Model Leaderboard (API)       │
│ 2. Discover Leaderboards (HTML)        │
│ 3. View Exported Files                  │
│ 4. Exit                                 │
└─────────────────────────────────────────┘

[INFO] Connecting to api.zeroeval.com...
[INFO] Sending request...
[DOWNLOAD] 145 KB received...
[SUCCESS] Download complete (145 KB)
Found 250 models available.

Rank  Model                         Organization  GPQA
------------------------------------------------------
1     GPT-5.2 Pro                  OpenAI        0.932
2     GPT-5.2                      OpenAI        0.924
...
```

## 🤝 Contributing

Feel free to submit issues, fork the repository, and create pull requests for any improvements.

## 📜 License

This project is open-source and available for educational purposes.

## 👨‍💻 Version History

- **v2.0** (2025) - Enhanced version with exports, progress tracking, and improved UI
- **v1.0** (2024) - Initial release with basic scraping functionality

## 🔗 Resources

- [WinHTTP Documentation](https://learn.microsoft.com/en-us/windows/win32/winhttp/winhttp-start-page)
- [nlohmann/json Library](https://github.com/nlohmann/json)
- [LLM Stats Website](https://llm-stats.com)

---

**Note**: This tool is for educational and research purposes. Always respect website terms of service and robots.txt when scraping.
