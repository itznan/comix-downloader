# Comix Downloader

[![CI](https://github.com/itznan/comix-downloader/actions/workflows/ci.yml/badge.svg)](https://github.com/itznan/comix-downloader/actions/workflows/ci.yml)
![Python Versions](https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13-blue.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

High-performance Python CLI downloader and scraper for **comix.to** manga, manhwa & webtoons. Features in-CLI interactive search, account reading list sync, automated `ComicInfo.xml` metadata generation, and fast compilation to **CBZ, PDF, and EPUB**.

---

## Features

- **Document Formats**: Export chapters as `.cbz` comic book archives, `.pdf` documents, or `.epub` digital books.
- **Media Server ComicInfo.xml**: Automatically creates standard `ComicInfo.xml` metadata in every series directory for **Komga**, **Kavita**, and **Calibre**.
- **Interactive Terminal Search**: Search directly from the terminal (`search <query>`) with instant number selection.
- **Trending & Discovery**: Discover daily, weekly, or monthly trending titles (`trending`).
- **Account Follows & Library Sync**: Scan your Comix.to bookmarks (`--sync`) and download newly released chapters.
- **Bookmark & Library Export**: Export reading lists to MyAnimeList (MAL XML), AniList (JSON), or CSV.
- **Smart Deduplication**: Automatically selects the highest-voted scanlation group to prevent duplicate chapter numbers.
- **Flexible Ranges**: Specify `-c 1-10`, `-c 1,3,5`, `-c latest`, or `-c 20+`.
- **Aria2 Acceleration**: Auto-detects and leverages `aria2c` for high-throughput parallel downloads.
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
python main.py search "Solo Leveling"
```

### 2. Download by Title URL or Slug
```bash
# Download chapters 1 to 10
python main.py https://comix.to/title/example-comic -c 1-10

# Download all chapters as CBZ archives (standard for comic readers)
python main.py https://comix.to/title/example-comic --cbz

# Download all chapters as PDF and merge into a single volume
python main.py https://comix.to/title/example-comic -m
```

### 3. Resume from a Specific Chapter
Provide a chapter URL to automatically download from that chapter onwards:
```bash
python main.py https://comix.to/title/example-comic/12345-chapter-20 --from-here
```

### 4. Sync Account Library
Keep your personal collection up to date by downloading newly released unread chapters:
```bash
python main.py --sync --unread-only
```

### 5. Export Bookmarks
Export your reading list to your favorite anime/manga tracker:
```bash
python main.py --export-bookmarks mal
```

---

## Command-Line Options

| Option | Short | Description | Default |
|---|---|---|---|
| `target` | | Comic URL / slug, or `search`, `trending`, `collection`, `sync` | *Required* |
| `--format` | | Document format (`pdf`, `cbz`, `epub`, `both`) | `pdf` |
| `--cbz` | | Shortcut to export chapters as `.cbz` archives | `False` |
| `--epub` | | Shortcut to export chapters as `.epub` books | `False` |
| `--chapters` | `-c` | Chapters to download (`all`, `1-5`, `1,3,5`, `latest`, `10+`) | `all` |
| `--merge` | `-m` | Merge chapters into one combined PDF or CBZ | `False` |
| `--sync` | | Check reading list and download new/missing chapters | `False` |
| `--unread-only` | | Only download unread chapters when syncing | `False` |
| `--folder` | | Filter reading list by folder (`reading`, `completed`, etc.) | `None` |
| `--export-bookmarks`| | Export reading list (`mal`, `anilist`, `csv`, `json`) | `mal` |
| `--trending` | | Browse trending titles (`--days 1`, `7`, `30`) | `False` |
| `--output` | `-o` | Output directory | `./downloads/{Title}` |
| `--lang` | `-l` | Filter chapters by language | `en` |
| `--threads` | `-t` | Concurrent download threads | `8` |
| `--aria2` | | Force use aria2c for accelerated downloading | Auto-detected |
| `--cookies` | | Path to cookie file for private/restricted titles | Auto-discovered |
| `--no-comicinfo` | | Disable generating ComicInfo.xml metadata | `False` |

---

## Cloudflare & Private Manga Access

When Comix.to enables Cloudflare verification or when accessing private folders/follows, provide a cookie file export:

1. Export cookies from your browser for `comix.to` using a standard browser extension (e.g., *Get cookies.txt LOCALLY*).
2. Save it as `comix.to_cookies.txt` in the project root, or pass it via:
   ```bash
   python main.py search "Solo Leveling" --cookies comix.to_cookies.txt
   ```

---

## Testing

Run unit tests and verification with coverage:

```bash
pytest -v --cov=src
```

Lint with Ruff:

```bash
ruff check .
```

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
