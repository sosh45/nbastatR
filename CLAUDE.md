# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Overview

nbastatR is an R package that provides a unified interface to NBA data from multiple sources:
- **NBA Stats API** (stats.nba.com) - Primary source for official statistics
- **Basketball-Reference.com** - Historical data, awards, rankings
- **HoopsHype** - Salary information
- **RealGM** - Transactions, rosters
- **nbadraft.net** - Mock drafts
- **Basketball Insiders** - Salary cap data

## Development Commands

```r
# Generate documentation and NAMESPACE from Roxygen2 comments
devtools::document()

# Build the package
devtools::build()

# Run R CMD check
devtools::check()

# Install locally
devtools::install()

# Run all tests
devtools::test()

# Run a single test file
testthat::test_file("tests/testthat/test-nbastatR.R")
```

## Code Architecture

### Core Files (by function)

| File | Purpose |
|------|---------|
| `R/new_stats.R` | NBA Stats API interface (largest file, ~18k lines) |
| `R/bref.R` | Basketball-Reference scraping |
| `R/new_ish.R` | Additional API helpers |
| `R/utils.R` | Shared utilities including `make_url()` for API URL construction |
| `R/resolver.R` | Player name resolution across data sources |

### Key Patterns

**Memoization**: Most API functions use `memoise()` to cache responses:
```r
function_name <- memoise::memoise(function(...) { ... })
```

**Parallel Processing**: Functions support `future`/`furrr` for batch operations:
```r
library(future)
plan(multisession)
game_logs(seasons = 2010:2019)  # Runs in parallel
```

**Internal Functions**: Prefixed with `.` (e.g., `.curl_chinazi()`, `.get_current_season()`)

**Name Resolution**: Player names differ across sources. `resolver.R` and `.resolve_bref_players()` in `bref.R` handle mapping.

### Naming Conventions

- Exported functions: `snake_case` (e.g., `game_logs()`, `box_scores()`)
- Internal functions: `.dot_prefix` (e.g., `.get_current_season()`)
- Section headers in files: `# section_name ---`

## Testing

Tests are in `tests/testthat/`. Current coverage is minimal. Tests make live API calls, so they require network access.

## API Notes

The NBA Stats API requires specific headers. The package handles this via custom curl wrappers. If API calls fail, check if endpoints have changed - the NBA occasionally updates their API structure.
