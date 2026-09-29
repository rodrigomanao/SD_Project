# Distributed Web Page Indexing and Search System

A distributed web indexing and search system implemented in Java using RMI (Remote Method Invocation). The system consists of multiple components that work together to index and search web pages in a coordinated and distributed manner.

---

## Requirements

- **Java JDK 11 or higher**
- **Maven 3.6+**
- **Operating System:** macOS, Linux, or Windows
- **Internet connection** (for indexing web pages)

---

## Installation

### 1. Clone the repository
```bash
git clone <repository-url>
cd SD_Project
```

### 2. Compile the project
```bash
make clean
```
Or using Maven directly:
```bash
mvn clean compile
```

---

## Running the System

### Option 1: Run all components automatically
```bash
make run-backend
```
This command starts all system components in the following order:
1. Gateway (default port: 8183)
2. URL Queue (default port: 8181)
3. Barrel 1 (default port: 8186)
4. Barrel 2 (default port: 8182)
5. Downloader
6. Client

### Option 2: Run components individually

#### Gateway
```bash
make run-g
```
Or with arguments:
```bash
java -cp target/classes gateway.Gateway <port> <name> <host>
```

#### URL Queue
```bash
make run-q
```
Or with arguments:
```bash
java -cp target/classes queue.URLQueue <port> <name> <host>
```

#### Barrel (Storage Barrel)
```bash
make run-b1    # Barrel 1
make run-b2    # Barrel 2
```
Or with arguments:
```bash
java -cp target/classes barrel.IndexStorageBarrel <port> <name> <host>
```

#### Downloader
```bash
make run-d
```
Or with arguments:
```bash
java -cp target/classes downloader.Downloader <gatewayPort> <queuePort> <gatewayHost> <queueHost>
```

#### Client
```bash
make run-c
```
Or with arguments:
```bash
java -cp target/classes client.Client <gatewayPort> <gatewayHost>
```

#### Spring Boot REST API (Web Server)
```bash
make run-api
```
The API will be available at `http://localhost:8080`.

**Main endpoints:**
- `GET/POST /api/search?q=query` - Search
- `GET /api/barrels/active` - Active barrels
- `GET /api/health` - Health check

---

## Configuration

The default settings can be changed in:
```
src/main/resources/Config.properties
```

### Configuration example
```properties
# Gateway
gateway.host=127.0.0.1
gateway.port=8183
gateway.name=gateway

# URL Queue
queue.host=127.0.0.1
queue.port=8181
queue.name=urlqueue

# Barrel1
barrel1.host=127.0.0.1
barrel1.port=8186
barrel1.name=barrel1
```

**Note:** If command-line arguments are provided, they take precedence over the settings in the configuration file.

---

## Stopping the System

### macOS/Linux:
```bash
make stop-backend
```

### Windows:
Close each terminal manually or use `Ctrl+C` in each window.

---

## System Architecture

### Components

#### 1. **Gateway** (Entry point)
- Central communication point for the system
- Performs load balancing between active barrels
- Manages automatic barrel failover
- Aggregates search results from multiple barrels
- Maintains system statistics

#### 2. **Index Storage Barrels** (Distributed storage)
- Store the inverted index (word → URLs)
- Automatically synchronize data with one another
- Dynamically remove stop words using IQR (Interquartile Range)
- Store progress on disk for recovery after failures
- Can be added or removed dynamically

#### 3. **Downloader** (Indexing workers)
- Download web pages and extract their content
- Tokenize text and send words to the barrels
- Extract links and add them to the URL queue
- Process pages asynchronously
- Support automatic retries in case of errors

#### 4. **URL Queue**
- Manages URLs to be processed (`BlockingDeque`)
- Prevents duplicate URLs
- Persists its state to barrels when shutting down
- Supports URL prioritization

#### 5. **Client** (User interface)
- Command-line interface
- Allows searches for one or multiple words
- Displays pages ordered by relevance
- Allows users to add URLs manually
- Displays system statistics

#### 6. **Spring Boot REST API** (Web interface)
- REST API for integration with a React frontend
- JSON endpoints for search and management
- CORS enabled for local development
- Health checks and monitoring
- Default port: 8080

---

## Features

### Search
- **Single-word search:** Returns all URLs containing the word
- **Multiple-word search:** Returns URLs containing ALL words (intersection)
- **Reference-based sorting:** Results are sorted by the number of incoming links (popularity)
- **Detailed information:** Title, URL, and description (snippet) for each page

### Indexing
- **Distributed indexing:** Multiple barrels process pages in parallel
- **Dynamic stop words:** Automatic removal of very common words using IQR
- **Persistence:** Indexes are automatically saved to disk
- **Synchronization:** New barrels receive data from existing barrels

### Statistics
- Number of active barrels
- Index size for each barrel
- Average response time per barrel
- Top 10 most frequent searches
- Most referenced pages

---

## Usage Example

### 1. Start the system
```bash
make run-backend
make run-api
```

### 2. Choose an option from the Client menu:
```
CLIENT MENU
============================================
1. Add URL for indexing
2. Search for a word
3. View statistics
4. View the list of links for a page
0. Exit
============================================
```

### 3. Add a URL for indexing:
```
Choose an option (1-5): 1
Enter the URL (http:// or https://): https://en.wikipedia.org/wiki/Java
URL successfully added to the queue!
```

### 4. Search for words:
```
Choose an option (1-5): 2
Word(s) to search: java programming
Found 42 result(s) for: [java, programming]
------------------------------------------------------------
[1] - 15 Reference(s)
Java (programming language)
URL: https://en.wikipedia.org/wiki/Java_(programming_language)
    Java is a high-level, class-based, object-oriented programming language...
------------------------------------------------------------
```

### 5. View statistics:
```
==================================================
Choose an option (1-5): 3

TOP 10 SEARCHES
------------------------------
No searches have been recorded yet

ACTIVE BARRELS
------------------------------
Active Barrel -> barrel1
Port: 8182
Host: 127.0.0.1
Index: 0

Active Barrel -> barrel2
Port: 8184
Host: 127.0.0.1
Index: 0


REGISTERED BARRELS
------------------------------
Registered Barrel -> barrel2:8184:127.0.0.1
Registered Barrel -> barrel1:8182:127.0.0.1

AVERAGE RESPONSE TIME PER BARREL
------------------------------
No response time recorded.
...
```

---

## File Structure

```
SD_Project/
├── src/main/java/
│   ├── barrel/              # Barrel logic
│   ├── client/              # Client interface
│   ├── common/              # Shared classes (Utils, Config, etc.)
│   ├── downloader/          # Indexing workers
│   ├── gateway/             # Gateway and connections
│   └── queue/               # URL queue
├── src/main/resources/
│   └── Config.cfg           # System configuration
├── Makefile                 # Build and execution commands
├── pom.xml                  # Maven configuration
└── README.md                # This file
```

---

## Testing

To test the system, you can use URLs such as:
- `https://en.wikipedia.org/wiki/Main_Page`
- `https://eden.dei.uc.pt/~rbarbosa/sd/`
- Any public web page

---

## Troubleshooting

### Error: "Address already in use"
- Check whether a component is already running on the same port
- Run `make stop-all` and try again

### Error: "Connection refused"
- Make sure the Gateway is running before the other components
- Check the host and port settings in `Config.properties`

### Barrels do not synchronize
- Check that all barrels are registered with the Gateway
- Confirm that the ports and hosts are correct

### Stop words remove important words
- The system learns dynamically after 5,000 indexed pages
- It only removes words that are outliers across multiple consecutive cycles

---

## Important Notes

- **Persistence:** Indexes are automatically saved when barrels shut down
- **Failover:** The system continues to operate even if a barrel fails
- **Scalability:** Additional barrels and downloaders can be added dynamically
- **Stop Words:** Stop words are discovered in a distributed manner using statistical analysis (IQR)

---

## Javadoc

To generate and view the project's Javadoc documentation:

### Generate Javadoc

#### Using Maven:
```bash
mvn javadoc:javadoc
```

### Open the documentation
```bash
cd target/reports/apidocs

# macOS/Linux
open index.html

# Windows
start index.html
```

---

## Authors

- Henrique Diz
- Rodrigo Manão
- João Francisco

---

## License

Academic project - University of Coimbra

---
