# Comix Downloader

High-performance CLI tool to download manga, manhwa, and comics from **comix.to** as high-quality **CBZ, PDF, and EPUB** files with ComicInfo.xml metadata.

---

## Features
- **CBZ, PDF & EPUB Output**: Download chapters as `.cbz` archives, `.pdf` files, or `.epub` books.
- **Media Server Integration**: Generates `ComicInfo.xml` metadata for Komga, Kavita, and Calibre.
- **Interactive In-CLI Search**: Search directly from terminal with interactive number selection.
- **Trending & Library Sync**: Download trending titles or sync your reading list.
- **Aria2 Acceleration**: Auto-detects `aria2c` for high-speed multi-connection downloads.
- **Security Emulation**: Uses `comix_signer.js` to compute security tokens seamlessly.

---

## Installation & Usage

1. **Install requirements:**
   ```bash
   pip install -r requirements.txt
   ```
2. **Search and download:**
   ```bash
   python main.py search "Solo Leveling"
   ```
3. **Download directly by URL:**
   ```bash
   python main.py https://comix.to/title/example-comic -c 1-5 --cbz
   ```

---

## License
MIT License.
