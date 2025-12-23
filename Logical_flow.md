# LLM Stats Interactive Web Scraper - Logical Flow

## 📚 Libraries and Dependencies Used

### System Libraries
- **`<windows.h>`** - Windows API functions
- **`<winhttp.h>`** - WinHTTP API for HTTP/HTTPS requests
- **`winhttp.lib`** - Windows HTTP Services library (linked)

### Standard C++ Libraries
- **`<iostream>`** - Console input/output
- **`<fstream>`** - File input/output operations
- **`<vector>`** - Dynamic arrays for data storage
- **`<string>`** - String manipulation
- **`<iomanip>`** - Output formatting
- **`<algorithm>`** - Sorting and data transformation
- **`<thread>`** - Threading support for delays
- **`<chrono>`** - Time utilities for retry delays

### Third-Party Libraries
- **`json.hpp`** (nlohmann/json) - Modern JSON parsing library for C++

---

## 🔄 Main Program Flow

```mermaid
flowchart TD
    Start([Program Start]) --> Init[Display Welcome Banner]
    Init --> Menu{Display Main Menu}
    
    Menu -->|Option 1| APIFlow[Fetch Model Leaderboard API]
    Menu -->|Option 2| HTMLFlow[Discover Leaderboards HTML]
    Menu -->|Option 3| ViewFiles[View Exported Files]
    Menu -->|Option 4| Exit([Exit Program])
    
    APIFlow --> FetchAPI[FetchAndDisplayModels]
    HTMLFlow --> ScrapeHTML[ScrapeAndDisplayLeaderboards]
    ViewFiles --> ListFiles[List Generated Files]
    
    FetchAPI --> DataProcess[Process & Rank Data]
    DataProcess --> Export[Export Results]
    Export --> Menu
    
    ScrapeHTML --> ParseHTML[Parse HTML Links]
    ParseHTML --> DisplayLinks[Display Found Links]
    DisplayLinks --> Menu
    
    ListFiles --> Menu
```

---

## 🌐 Data Fetching Flow (HTTP Request)

```mermaid
flowchart TD
    Start([FetchData Function]) --> Init[Initialize Attempt Counter]
    Init --> Connect[Create WinHTTP Session]
    
    Connect --> Retry{Attempts < MAX_RETRIES?}
    Retry -->|No| Failed([Return Empty String])
    Retry -->|Yes| Inc[Increment Attempt Counter]
    
    Inc --> Wait{First Attempt?}
    Wait -->|No| Delay[Sleep RETRY_DELAY_MS]
    Wait -->|Yes| OpenConn[WinHttpConnect to Domain]
    Delay --> OpenConn
    
    OpenConn --> Request[WinHttpOpenRequest GET]
    Request --> Send[WinHttpSendRequest]
    Send --> Response[WinHttpReceiveResponse]
    
    Response --> Check{Response OK?}
    Check -->|No| Retry
    Check -->|Yes| Read[Read Data Loop]
    
    Read --> Query[WinHttpQueryDataAvailable]
    Query --> HasData{Data Available?}
    HasData -->|Yes| ReadChunk[WinHttpReadData]
    ReadChunk --> Append[Append to Response Buffer]
    Append --> Progress[Display Download Progress]
    Progress --> Query
    
    HasData -->|No| Success([Return Response String])
```

---

## 📊 API Data Processing Flow

```mermaid
flowchart TD
    Start([FetchAndDisplayModels]) --> Fetch[Call FetchData for API]
    Fetch --> Check{Data Retrieved?}
    Check -->|No| Error([Display Error & Return])
    Check -->|Yes| Parse[Parse JSON Response]
    
    Parse --> Loop[For Each Model in JSON]
    Loop --> Extract[Extract Model Data:<br/>- name<br/>- organization<br/>- gpqa_score<br/>- input_price]
    Extract --> Price[Parse Price String to Numeric Value]
    Price --> Store[Store in ModelEntry Vector]
    Store --> Loop
    
    Store --> UserInput[Prompt User for Display Count]
    UserInput --> Sort[Create Three Ranking Lists]
    
    Sort --> Rank1[Performance Ranking:<br/>Sort by GPQA Score DESC]
    Sort --> Rank2[Price Ranking:<br/>Sort by Price ASC]
    Sort --> Rank3[Value Ranking:<br/>Sort by Performance/Price DESC]
    
    Rank1 --> Display[Display All Rankings in Console]
    Rank2 --> Display
    Rank3 --> Display
    
    Display --> ExportFiles[Export to Multiple Formats]
    
    ExportFiles --> MD[Export to Markdown<br/>report.md]
    ExportFiles --> HTML[Export to HTML<br/>leaderboard.html]
    ExportFiles --> CSV1[Export to CSV<br/>leaderboard_performance.csv]
    ExportFiles --> CSV2[Export to CSV<br/>leaderboard_price.csv]
    ExportFiles --> CSV3[Export to CSV<br/>leaderboard_value.csv]
    ExportFiles --> JSON[Export to JSON<br/>leaderboard_all.json]
    
    MD --> Stats[Display Statistics Summary]
    HTML --> Stats
    CSV1 --> Stats
    CSV2 --> Stats
    CSV3 --> Stats
    JSON --> Stats
    
    Stats --> End([Return to Main Menu])
```

---

## 🕷️ HTML Web Scraping Flow

```mermaid
flowchart TD
    Start([ScrapeAndDisplayLeaderboards]) --> Fetch[Call FetchData for HTML]
    Fetch --> Check{HTML Retrieved?}
    Check -->|No| Error([Display Error & Return])
    Check -->|Yes| Search[Search for Anchor Tags]
    
    Search --> Find[Find '<a' Tag]
    Find --> Exists{Tag Found?}
    Exists -->|No| Results{Links Found?}
    Exists -->|Yes| CheckHref[Extract href Attribute]
    
    CheckHref --> Filter{Contains<br/>'leaderboard' or<br/>'arena'?}
    Filter -->|No| Find
    Filter -->|Yes| ExtractText[Extract Link Text]
    
    ExtractText --> Clean[Clean HTML Tags from Text]
    Clean --> Duplicate{Already in List?}
    Duplicate -->|Yes| Find
    Duplicate -->|No| Add[Add to Found Links]
    Add --> Display[Display Link Info]
    Display --> Find
    
    Results -->|No| NoLinks([Display No Links Found])
    Results -->|Yes| Summary[Display Summary Count]
    NoLinks --> End([Return to Main Menu])
    Summary --> End
```

---

## 🗂️ Data Export Structure

```mermaid
flowchart LR
    Source[Parsed Model Data] --> Sorter{Data Sorter}
    
    Sorter --> PerfRank[Performance Ranking<br/>by GPQA Score]
    Sorter --> PriceRank[Price Ranking<br/>by Input Price]
    Sorter --> ValueRank[Value Ranking<br/>by Performance/Price Ratio]
    
    PerfRank --> Export{Export Engine}
    PriceRank --> Export
    ValueRank --> Export
    
    Export --> MD[📄 report.md<br/>Markdown Tables]
    Export --> HTML[🌐 leaderboard.html<br/>Interactive Web Page]
    Export --> CSV1[📊 leaderboard_performance.csv]
    Export --> CSV2[📊 leaderboard_price.csv]
    Export --> CSV3[📊 leaderboard_value.csv]
    Export --> JSON[🔧 leaderboard_all.json<br/>Complete Data Structure]
```

---

## 📋 Data Structures

### ModelEntry Structure
```cpp
struct ModelEntry {
    std::string name;              // Model name (e.g., "GPT-5.2 Pro")
    std::string organization;      // Organization (e.g., "OpenAI")
    double gpqa_score;            // GPQA benchmark score (0.0 - 1.0)
    std::string input_price_str;   // Price string representation
    double input_price_val;        // Parsed numeric price for sorting
}
```

---

## 🔑 Key Functions

### 1. **FetchData(domain, path)**
- **Purpose**: HTTP/HTTPS data fetcher with retry mechanism
- **Returns**: String containing response body
- **Features**: 
  - Retry up to MAX_RETRIES (3) times
  - Progress indicator during download
  - RAII pattern for resource management

### 2. **ParsePrice(priceStr)**
- **Purpose**: Extract numeric value from price strings
- **Returns**: Double value of price
- **Handles**: 
  - "Free" strings → 0.0
  - Invalid formats → 999999.0 (high penalty)
  - Numeric extraction with decimal support

### 3. **FetchAndDisplayModels()**
- **Purpose**: Main API data processing workflow
- **Actions**:
  1. Fetch JSON from API
  2. Parse and store models
  3. Create three rankings
  4. Display to console
  5. Export to 6 file formats
  6. Show statistics

### 4. **ScrapeAndDisplayLeaderboards()**
- **Purpose**: HTML parsing for leaderboard links
- **Method**: 
  - Search for `<a>` tags
  - Filter by href patterns
  - Extract and clean link text
  - Display discovered links

### 5. **ExportToHTML(perfRank, priceRank, valueRank)**
- **Purpose**: Generate interactive HTML report
- **Features**:
  - Tab-based navigation
  - Color-coded rankings
  - Responsive design
  - Dark theme styling

---

## 🎯 Data Scraping Methods

### Method 1: API-Based Scraping
```
Source: api.zeroeval.com/leaderboard/models/full
Method: RESTful API call
Format: JSON response
Data:  - Model names
       - Organization names  
       - GPQA scores
       - Input prices
```

### Method 2: HTML Web Scraping
```
Source: llm-stats.com
Method: HTML parsing
Format: Raw HTML
Data:  - Leaderboard links
       - Page navigation structure
       - Available arena links
```

---

## 🔄 Sorting Algorithms

### Performance Ranking
```
Sort Criteria: GPQA Score (Descending)
Algorithm: std::sort with lambda comparator
Formula: a.gpqa_score > b.gpqa_score
```

### Price Ranking
```
Sort Criteria: Input Price (Ascending)
Algorithm: std::sort with lambda comparator
Formula: a.input_price_val < b.input_price_val
Tie-Breaker: Higher GPQA score wins
```

### Value Ranking
```
Sort Criteria: Performance-to-Price Ratio (Descending)
Algorithm: std::sort with lambda comparator
Formula: (GPQA Score) / (Price + 0.01)
Note: +0.01 prevents division by zero
```

---

## 🎨 Output Formats

| Format | File Name | Purpose |
|--------|-----------|---------|
| **Markdown** | `report.md` | Human-readable summary with tables |
| **HTML** | `leaderboard.html` | Interactive web-based report with tabs |
| **CSV** | `leaderboard_performance.csv` | Performance ranking for Excel/Sheets |
| **CSV** | `leaderboard_price.csv` | Price ranking for Excel/Sheets |
| **CSV** | `leaderboard_value.csv` | Value ranking for Excel/Sheets |
| **JSON** | `leaderboard_all.json` | Complete structured data for APIs |

---

## ⚙️ Configuration Constants

```cpp
const std::wstring API_DOMAIN = L"api.zeroeval.com";
const std::wstring API_PATH = L"/leaderboard/models/full?justCanonicals=true";
const std::wstring WEB_DOMAIN = L"llm-stats.com";
const std::wstring WEB_PATH = L"/";
const int MAX_RETRIES = 3;
const int RETRY_DELAY_MS = 1000;
```

---

## 🛡️ Error Handling

### Network Errors
- Automatic retry mechanism (up to 3 attempts)
- 1-second delay between retries
- Detailed error logging with error codes

### JSON Parsing Errors
- Try-catch blocks for parse errors
- Null value handling for missing fields
- Default values for invalid data

### File I/O Errors
- Check file stream status before writing
- Silent failure with console notification
- RAII pattern ensures file handles are closed

---

## 📈 Program Execution Flow Summary

1. **Initialization** → Display welcome banner and menu
2. **User Selection** → Choose operation from menu
3. **Data Fetching** → HTTP request with retry logic
4. **Data Processing** → Parse JSON/HTML and extract data
5. **Data Ranking** → Sort by performance, price, and value
6. **Display Results** → Show rankings in formatted tables
7. **Export Data** → Generate 6 different output files
8. **Statistics** → Display summary information
9. **Loop Back** → Return to menu for next operation

---

## 🎯 Key Features

✅ **Interactive Menu System** - User-friendly CLI interface  
✅ **Dual Scraping Methods** - API and HTML scraping  
✅ **Retry Mechanism** - Handles network failures gracefully  
✅ **Multi-Format Export** - 6 different output formats  
✅ **Color-Coded Output** - ANSI colors for better readability  
✅ **Progress Indicators** - Real-time download progress  
✅ **RAII Pattern** - Safe resource management  
✅ **Error Handling** - Comprehensive error checking  
✅ **Data Validation** - Price parsing and null handling  

---

*Generated for LLM Stats Interactive Web Scraper v2.0*
