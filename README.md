# Comix Downloader

[![CI](https://github.com/itznan/comix-downloader/actions/workflows/ci.yml/badge.svg)](https://github.com/itznan/comix-downloader/actions/workflows/ci.yml)
[![CodeQL](https://github.com/itznan/comix-downloader/actions/workflows/codeql.yml/badge.svg)](https://github.com/itznan/comix-downloader/actions/workflows/codeql.yml)
![Python Versions](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13-blue.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

High-performance Python CLI downloader, scraper, and local REST API server for **comix.to** manga, manhwa & webtoons. Features in-CLI interactive search, account reading list sync, automated `ComicInfo.xml` metadata generation, interactive Swagger UI, and fast compilation to **CBZ, PDF, and EPUB**.

---

## Features

- **Multi-Format Export**: Export chapters as `.cbz` comic book archives, `.pdf` documents, or `.epub` digital books.
- **Media Server Ready**: Automatically creates standard `ComicInfo.xml` metadata and saves official high-resolution `cover.jpg` posters in every series directory for **Komga**, **Kavita**, and **Calibre**.
- **Interactive Terminal Search**: Search directly from the terminal (`search <query>`) with instant number selection and rich table rendering.
- **Trending & Discovery**: Discover daily, weekly, or monthly trending titles with filtering by genres, demographics, and status (`trending`).
- **Account Follows & Library Sync**: Scan your Comix.to bookmarks (`--sync`, `following`) and download newly released chapters.
- **Bookmark & Library Export**: Export reading lists directly to MyAnimeList (MAL XML), AniList (JSON), or CSV.
- **Curated Collections**: Batch download or preview entire user-curated reading lists (`collection <id>`).
- **Scanlation Group Selection**: View all scanlation teams for a title (`--list-groups`) and select preferred scanlators (`-g <group>`). Smart deduplication automatically selects the highest-voted group to prevent duplicate chapter numbers.
- **Flexible Chapter Ranges**: Specify `-c 1-10`, `-c 1,3,5`, `-c latest`, or `-c 20+`.
- **Resume Downloads**: Provide a chapter URL to download from that chapter onwards (`--from-here`).
- **Aria2 Acceleration**: Auto-detects and leverages `aria2c` for high-throughput parallel downloads with multi-threaded Python fallback.
- **Embedded Web Server & Swagger UI**: Run a local FastAPI web service (`server`) with interactive API documentation at `/docs` and `/redoc`.
- **Client Security Emulation**: Dynamic Node.js security VM bridge (`comix_signer.js`) for signature generation and token decryption.

---

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/itznan/comix-downloader.git
   cd comix-downloader
   ```

2. **Install Python dependencies:**
   ```bash
   pip install -r requirements.txt
   ```
   *(Ensure Node.js 18+ is installed on your system for client security VM computation).*

---

## Quick Start & Usage Examples

### 1. Interactive Search
Search Comix.to directly from your terminal and choose a title from the interactive prompt:
```bash
python main.py search "Chainsaw Man"
```
Filter search by comic type, publication status, and genres:
```bash
python main.py search "Leveling" --type manhwa --status finished --sort views_7d:desc
```

### 2. Download by Title URL or Slug
```bash
# Download chapters 1 to 10
python main.py https://comix.to/title/69l57-chainsaw-man -c 1-10

# Download all chapters as CBZ comic book archives
python main.py https://comix.to/title/69l57-chainsaw-man --cbz

# Download all chapters as EPUB digital books
python main.py https://comix.to/title/69l57-chainsaw-man --epub

# Download specific chapters and merge into a single volume PDF
python main.py https://comix.to/title/69l57-chainsaw-man -c 1-10 --merge
```

### 3. Resume from a Specific Chapter
Provide a chapter URL to automatically download from that chapter onwards:
```bash
python main.py https://comix.to/title/69l57-chainsaw-man/12345-chapter-20 --from-here
```

### 4. Sync Account Library & Bookmarks
Check your reading list and download newly released chapters:
```bash
python main.py sync
# Or only download unread chapters released after your last read chapter:
python main.py --sync --unread-only
```
List and interactively pick from your followed titles:
```bash
python main.py following
# Filter by reading status folder:
python main.py following --folder reading
```

### 5. Export Bookmarks
Export your reading list to your favorite anime/manga tracker:
```bash
python main.py --export-bookmarks mal      # MyAnimeList XML
python main.py --export-bookmarks anilist  # AniList JSON
python main.py --export-bookmarks csv      # Standard CSV
```

### 6. Browse Trending Titles
```bash
python main.py trending --limit 10
python main.py trending --days 7 --limit 10
python main.py trending --trend-type follows --days 7
```

### 7. Download Curated Collections
```bash
# Preview collection items without downloading
python main.py collection 123 --dry-run

# Batch download entire collection
python main.py collection https://comix.to/collection/123-best-action
```

### 8. List Scanlation Groups
```bash
python main.py https://comix.to/title/69l57-chainsaw-man --list-groups
# Filter downloads to a specific group:
python main.py https://comix.to/title/69l57-chainsaw-man -g "MangaPlus" -c 1-10
```

### 9. View Reading History
```bash
python main.py history
```

### 10. Start Local Web Server & Swagger UI
Start the local FastAPI service and open interactive API documentation in your browser:
```bash
python main.py server --port 8000
```
- **Swagger UI**: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- **ReDoc**: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)
- **OpenAPI Schema**: [http://127.0.0.1:8000/openapi.json](http://127.0.0.1:8000/openapi.json)

---

## Cloudflare & Cookie Authentication

Comix.to uses Cloudflare Turnstile protection. To download protected titles, access your private reading list, or bypass challenge checks, provide your browser session cookies:

### Step 1: Export Cookies
1. Open [comix.to](https://comix.to) in your browser (Chrome, Edge, Firefox, or mobile browser) on the same network.
2. Export your cookies for `comix.to` using a standard browser extension (such as *Get cookies.txt LOCALLY*) or copy the cookie header string.
3. Save the file as `comix.to_cookies.txt` in the project root directory (auto-discovered), or pass it via `--cookies <path>`.

### Step 2: Match Your Browser's User-Agent
Cloudflare Turnstile clearance tokens (`cf_clearance`) are cryptographically bound to your browser's exact `User-Agent`. If you exported cookies from a mobile browser or different browser version:

1. In your browser DevTools Console (F12), check your exact user agent:
   ```javascript
   navigator.userAgent
   ```
2. Pass it directly via the `--ua` command-line flag:
   ```bash
   python main.py following --ua "<your navigator.userAgent>"
   ```
   Or set it as an environment variable in your terminal session:
   - **PowerShell**:
     ```powershell
     $env:USER_AGENT = "<your navigator.userAgent>"
     ```
   - **Command Prompt (CMD)**:
     ```cmd
     set USER_AGENT=<your navigator.userAgent>
     ```
   - **Bash / Linux / macOS**:
     ```bash
     export USER_AGENT="<your navigator.userAgent>"
     ```

### Step 3: Validate Cookies Before Running
Verify that your cookie file and User-Agent bypass Cloudflare:
```bash
python -m src.cookies
# Or with a custom User-Agent:
python -m src.cookies --ua "<your navigator.userAgent>"
```

---

## Command-Line Options Reference

| Option | Short | Description | Default |
|---|---|---|---|
| `target` | | Comic URL/slug, or command: `search`, `trending`, `collection`, `groups`, `sync`, `export`, `following`, `history`, `server` | *None* |
| `search_query` | | Search keyword, collection ID/URL, or export format | `[]` |
| `--ua`, `--user-agent` | | Custom User-Agent matching the browser session that exported cookies | Environment / Default |
| `--format` | | Output document format (`pdf`, `cbz`, `epub`, `both`) | `pdf` |
| `--cbz` | | Shortcut to export chapters as `.cbz` comic archives | `False` |
| `--epub` | | Shortcut to export chapters as `.epub` digital books | `False` |
| `--chapters` | `-c` | Chapters to download (`all`, `1-5`, `1,3,5`, `latest`, `10+`) | `all` |
| `--merge` | `-m` | Merge all downloaded chapters into a single volume file | `False` |
| `--sync` | | Check reading list and download new/missing chapters | `False` |
| `--unread-only` | | Only download unread chapters when syncing | `False` |
| `--folder` | | Filter reading list by folder (`reading`, `completed`, `paused`, `dropped`, `planning`) | `None` |
| `--export-bookmarks`| | Export reading list (`mal`, `anilist`, `csv`, `json`) | `mal` |
| `--trending` | | Browse top/trending titles | `False` |
| `--days` | | Time window for trending/top titles: `1`, `7`, or `30` | `1` |
| `--trend-type` | | Discovery ranking mode: `trending` or `follows` | `trending` |
| `--auto-download` | | Automatically batch download all discovered trending titles | `False` |
| `--collection` | | Batch download a curated collection by URL or ID | `None` |
| `--dry-run` | | Preview items without downloading (for sync or collections) | `False` |
| `--type` | | Filter search by comic type (`manga`, `manhwa`, `manhua`, `other`) | `None` |
| `--status` | | Filter search by status (`releasing`, `finished`, `on_hiatus`, `discontinued`) | `None` |
| `--genre`, `--genres` | | Filter search by genre (e.g. `action,fantasy`) | `None` |
| `--demographic` | | Filter search by demographic (`shounen`, `seinen`, `shoujo`, `josei`) | `None` |
| `--sort` | | Sort order (e.g. `views_7d:desc`, `chapter_updated_at:desc`, `score:desc`) | `None` |
| `--limit` | | Number of search or trending results to return | `10` |
| `--no-interactive` | | Print search or following results without interactive prompt | `False` |
| `--output` | `-o` | Output directory for downloaded chapters | `./downloads/{Title}` |
| `--lang` | `-l` | Filter chapters by language code | `en` |
| `--group` | `-g` | Filter chapters by scanlation group name or ID | `None` |
| `--list-groups` | | Display all scanlation groups available for the title | `False` |
| `--from-here` | | When a chapter URL is provided, download all chapters from that point onwards | `False` |
| `--threads` | `-t` | Number of concurrent image download threads | `8` |
| `--aria2` | | Force use `aria2c` for accelerated downloading | Auto-detected |
| `--no-aria2` | | Disable `aria2c` and use standard multi-threaded downloader | `False` |
| `--keep-images` | | Keep raw extracted image files after archive/PDF creation | `False` |
| `--cover` | | Download official cover art as `cover.jpg` and embed in PDF/CBZ | `True` |
| `--no-cover` | | Disable downloading and embedding cover art | `False` |
| `--cover-first` | | Insert cover art as the first page of every chapter document | `False` |
| `--no-comicinfo` | | Disable generating `ComicInfo.xml` metadata file | `False` |
| `--cookies` | | Path to Netscape or key=value cookies file | Auto-discovered |
| `--port` | | Web server port for Swagger UI | `8000` |
| `--host` | | Web server host interface | `127.0.0.1` |

---

## Testing & Quality

Run unit test suite with code coverage:
```bash
pytest -v --cov=src
```

Verify all 31 REST API endpoints & CLI routes:
```bash
python scripts/verify_all_commands.py
```

Lint with Ruff:
```bash
ruff check .
```

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
