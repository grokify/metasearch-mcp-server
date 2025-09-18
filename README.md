# Multi-Search MCP Server

[![Build Status][build-status-svg]][build-status-url]
[![Lint Status][lint-status-svg]][lint-status-url]
[![Go Report Card][goreport-svg]][goreport-url]
[![Docs][docs-godoc-svg]][docs-godoc-url]
[![Visualization][viz-svg]][viz-url]
[![License][license-svg]][license-url]

A Model Context Protocol (MCP) server implementation in Go that provides Google search functionality via multiple search engine APIs through a plugin-based architecture. Currently supporting:

1. Serper
2. SerpApi

## Features

This server provides comprehensive Google search capabilities through the following tools:

- **`google_search`** - General web search
- **`google_search_news`** - News articles search
- **`google_search_images`** - Image search
- **`google_search_videos`** - Video search
- **`google_search_places`** - Places and location search
- **`google_search_maps`** - Maps search
- **`google_search_reviews`** - Reviews search
- **`google_search_shopping`** - Product/shopping search
- **`google_search_scholar`** - Academic papers search
- **`google_search_lens`** - Visual search
- **`google_search_autocomplete`** - Search suggestions
- **`webpage_scrape`** - Web page content extraction

## Supported Search Engines

### Serper API (Default)
- **API**: [serper.dev](https://serper.dev)
- **Environment Variable**: `SERPER_API_KEY`
- **All tools supported**

### SerpAPI
- **API**: [serpapi.com](https://serpapi.com)
- **Environment Variable**: `SERPAPI_API_KEY`
- **Most tools supported** (note: `google_search_lens` falls back to image search)

## Prerequisites

1. **Go 1.24.5 or later**
2. **API Key** for your chosen search engine:
   - **Serper API Key** - Get one from [serper.dev](https://serper.dev)
   - **SerpAPI Key** - Get one from [serpapi.com](https://serpapi.com)

## Installation

### Option 1: Install via `go install`

```bash
go install github.com/grokify/metasearch-mcp-server@latest
```

### Option 2: Build from source

1. Clone or download this repository
2. Set your API key(s) as environment variables:
   ```bash
   # For Serper (default)
   export SERPER_API_KEY="your_serper_api_key_here"
   
   # For SerpAPI (optional)
   export SERPAPI_API_KEY="your_serpapi_key_here"
   
   # Choose which engine to use (optional, defaults to "serper")
   export SEARCH_ENGINE="serper"  # or "serpapi"
   ```
3. Install dependencies:
   ```bash
   go mod tidy
   ```
4. Build the server:
   ```bash
   go build -o multi-search-mcp-server
   ```

## Usage

### Running the Server

```bash
# Use default engine (Serper)
./multi-search-mcp-server

# Or explicitly choose an engine
SEARCH_ENGINE=serpapi ./multi-search-mcp-server
```

The server runs using stdio transport and follows the MCP specification.

### Tool Parameters

All search tools accept the following parameters:

- **`query`** (required) - The search query string
- **`location`** (optional) - Search location/region
- **`language`** (optional) - Language code (e.g., "en", "es", "fr")
- **`country`** (optional) - Country code (e.g., "us", "uk", "ca")
- **`num_results`** (optional) - Number of results to return (1-100, default: 10)

The `webpage_scrape` tool accepts:
- **`url`** (required) - The URL to scrape

### Example Tool Calls

#### Basic Web Search
```json
{
  "name": "google_search",
  "arguments": {
    "query": "artificial intelligence trends 2024",
    "num_results": 5
  }
}
```

#### Location-specific News Search
```json
{
  "name": "google_search_news",
  "arguments": {
    "query": "climate change",
    "location": "New York",
    "language": "en",
    "country": "us"
  }
}
```

#### Web Scraping
```json
{
  "name": "webpage_scrape",
  "arguments": {
    "url": "https://example.com/article"
  }
}
```

## Configuration with MCP Clients

### Claude Desktop

Add to your Claude Desktop configuration file:

```json
{
  "mcpServers": {
    "multi-search": {
      "command": "/path/to/multi-search-mcp-server",
      "env": {
        "SEARCH_ENGINE": "serper",
        "SERPER_API_KEY": "your_serper_api_key_here",
        "SERPAPI_API_KEY": "your_serpapi_key_here"
      }
    }
  }
}
```

Or for a specific engine only:

```json
{
  "mcpServers": {
    "search-serper": {
      "command": "/path/to/multi-search-mcp-server",
      "env": {
        "SEARCH_ENGINE": "serper",
        "SERPER_API_KEY": "your_serper_api_key_here"
      }
    },
    "search-serpapi": {
      "command": "/path/to/multi-search-mcp-server",
      "env": {
        "SEARCH_ENGINE": "serpapi",
        "SERPAPI_API_KEY": "your_serpapi_key_here"
      }
    }
  }
}
```

### Other MCP Clients

This server is compatible with any MCP-compliant client. Configure it according to your client's documentation, ensuring the appropriate API key environment variables are set.

## API Response Format

All search tools return JSON responses containing:
- Search results with titles, URLs, and snippets
- Knowledge graph information (when available)
- Related searches and suggestions
- Metadata about the search

The `webpage_scrape` tool returns:
- Extracted text content
- Page metadata
- Structured data (when available)

## Error Handling

The server provides detailed error messages for:
- Missing or invalid API keys
- Invalid search parameters
- Network connectivity issues
- API rate limits or quota exceeded
- Invalid URLs for scraping

## Architecture

This server uses a plugin-based architecture with the external [`github.com/grokify/metasearch`](https://github.com/grokify/metasearch) package:

```
/
└── main.go                      # Main server and tool registration
```

### Metasearch Package

The server leverages the external [`github.com/grokify/metasearch`](https://github.com/grokify/metasearch) package which provides:

- **Core interfaces** (`Engine`, `Registry`) for implementing search engines
- **Common types** (`SearchParams`, `ScrapeParams`, `SearchResult`)
- **Engine implementations** for various search providers
- **Registry management** for discovering and selecting engines

### Adding New Search Engines

To add a new search engine, contribute to the [`github.com/grokify/metasearch`](https://github.com/grokify/metasearch) package:

1. Fork the metasearch repository
2. Create a new engine implementation following the existing patterns
3. Submit a pull request to the metasearch repository

### Using the Metasearch Package

The [`github.com/grokify/metasearch`](https://github.com/grokify/metasearch) package can be used in your own projects:

```go
import "github.com/grokify/metasearch"

registry := metasearch.NewRegistry()
engine, err := metasearch.GetDefaultEngine(registry)
if err != nil {
    log.Fatal(err)
}

result, err := engine.Search(ctx, metasearch.SearchParams{
    Query: "golang web scraping",
})
```

## Development

### Building from Source

```bash
git clone <repository-url>
cd metasearch-mcp-server
go mod tidy
go build -o multi-search-mcp-server
```

### Testing

```bash
go test ./...
```

### Testing with Different Engines

```bash
# Test with Serper
SEARCH_ENGINE=serper SERPER_API_KEY=your_key ./multi-search-mcp-server

# Test with SerpAPI  
SEARCH_ENGINE=serpapi SERPAPI_API_KEY=your_key ./multi-search-mcp-server
```

## License

This project is licensed under the MIT License.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues related to:
- **Serper API**: Contact [serper.dev](https://serper.dev)
- **SerpAPI**: Contact [serpapi.com](https://serpapi.com)
- **MCP Specification**: See [Model Context Protocol documentation](https://modelcontextprotocol.io)
- **This Implementation**: Open an issue in this repository

 [build-status-svg]: https://github.com/grokify/metasearch-mcp-server/actions/workflows/ci.yaml/badge.svg?branch=main
 [build-status-url]: https://github.com/grokify/metasearch-mcp-server/actions/workflows/ci.yaml
 [lint-status-svg]: https://github.com/grokify/metasearch-mcp-server/actions/workflows/lint.yaml/badge.svg?branch=main
 [lint-status-url]: https://github.com/grokify/metasearch-mcp-server/actions/workflows/lint.yaml
 [goreport-svg]: https://goreportcard.com/badge/github.com/grokify/metasearch-mcp-server
 [goreport-url]: https://goreportcard.com/report/github.com/grokify/metasearch-mcp-server
 [docs-godoc-svg]: https://pkg.go.dev/badge/github.com/grokify/metasearch-mcp-server
 [docs-godoc-url]: https://pkg.go.dev/github.com/grokify/metasearch-mcp-server
 [viz-svg]: https://img.shields.io/badge/visualizaton-Go-blue.svg
 [viz-url]: https://mango-dune-07a8b7110.1.azurestaticapps.net/?repo=grokify%2Fmetasearch-mcp-server
 [loc-svg]: https://tokei.rs/b1/github/grokify/metasearch-mcp-server
 [repo-url]: https://github.com/grokify/metasearch-mcp-server
 [license-svg]: https://img.shields.io/badge/license-MIT-blue.svg
 [license-url]: https://github.com/grokify/metasearch-mcp-server/blob/master/LICENSE