# Problem Solving & Technical Post-Mortem: Black Clover Download

This document details the issues encountered while downloading the first 10 chapters of **Black Clover** from **Comix.to** using `comix-downloader`, the root causes discovered, and the solutions implemented to resolve them.

---

## 1. Executive Summary

| Issue # | Problem Description | Root Cause | Solution Implemented |
|---|---|---|---|
| **#1** | **Cloudflare Challenge 403 Block** | Stale `cf_clearance` cookie token & premature exit bug in profile setup | Fixed condition in `test/open_browser_profile.py` requiring valid homepage title; refreshed cookies using Chrome |
| **#2** | **HTTP 404 on Title Download** | Passed numeric `id` (`41888`) instead of alphanumeric `hid` (`rrleg`) | Extracted canonical slug `rrleg-black-clover` from search API response metadata |
| **#3** | **Image CDN 403 Failures (Aria2 / Urllib)** | Hardcoded `Referer: https://comix.to` in image requests blocked by CDNs | Modified `src/pdf.py` to only send `Referer` for `comix.to` hosts, omitting it for external CDNs |
| **#4** | **Cover Poster Download Blocked** | `static.comix.to` protected by Cloudflare; `urllib` returned 403 | Enhanced `src/metadata.py` to download covers via `curl_cffi` with active cookies & TLS impersonation |

---

## 2. Problem Breakdown & Solutions

### Problem 1: Cloudflare Turnstile Challenge (HTTP 403 Forbidden)

#### Symptom
Running search or download commands failed with:
```text
❌ Cloudflare Challenge Active (HTTP 403 Forbidden)
Comix.to requires valid Cloudflare cookies.
Please update 'comix.to_cookies.txt' or run 'python -m src.cookies' to inspect.
```

#### Investigation & Root Cause
1. Running `python -m src.cookies` confirmed that the existing `comix.to_cookies.txt` was rejected by Cloudflare (`HTTP 403`).
2. Cloudflare Turnstile binds `cf_clearance` to the client's public IP address, TLS fingerprint, and exact `User-Agent`.
3. In `test/open_browser_profile.py`, the detection polling loop had a logic bug:
   ```python
   # Previous buggy logic
   has_cf = any(c.get("name") == "cf_clearance" for c in cookies if "comix.to" in c.get("domain", ""))
   is_homepage = "moment" not in title.lower() and len(title) > 0

   if has_cf or (is_homepage and len(cookies) > 0):
       cf_found, cookie_file = save_cookies_to_file(cookies)
       break
   ```
   Because `has_cf` was already in the persistent browser profile from an expired previous session, `has_cf or ...` immediately evaluated to `True` while the page title was still `"Just a moment..."` (the Turnstile challenge page). It wrote stale cookies and exited prematurely without waiting for the challenge to complete.

#### Solution
1. Fixed `test/open_browser_profile.py` to strictly enforce that the real homepage has loaded:
   ```python
   # Fixed logic
   if is_homepage and (has_cf or len(cookies) > 0):
       cf_found, cookie_file = save_cookies_to_file(cookies)
       break
   ```
2. Executed browser session to solve Turnstile and save fresh clearance cookies to `comix.to_cookies.txt`.
3. Re-ran `python -m src.cookies`, verifying `✅ SUCCESS: Cookies are valid! Cloudflare bypassed successfully. (HTTP 200)`.

> **Note**: To keep the project lightweight and dependency-free, the Playwright daemon has been removed in favor of direct cookie import (`comix.to_cookies.txt`) and exact User-Agent matching (`--ua`).

---

### Problem 2: HTTP 404 on Title Download (`41888-black-clover`)

#### Symptom
Attempting to download via `python main.py 41888-black-clover -c 1-10` resulted in:
```text
[*] Accessing manga page: https://comix.to/title/41888-black-clover
❌ Error: HTTP Error 404: Not Found
```

#### Investigation & Root Cause
- The CLI search table displays the comic's internal database ID (`41888`).
- Comix.to URL routes require the alphanumeric Hash ID (`hid`), not the numeric ID.
- Inspecting the raw API search response:
  ```json
  {
    "id": 41888,
    "hid": "rrleg",
    "title": "Black Clover",
    "url": "/title/rrleg-black-clover"
  }
  ```

#### Solution
- Targeted the canonical URL slug `rrleg-black-clover` (`https://comix.to/title/rrleg-black-clover`), which loaded the series metadata and all 1,752 chapter releases correctly.

---

### Problem 3: Image CDNs Blocking `Referer: https://comix.to` (Aria2 / Urllib 403)

#### Symptom
During chapter image downloads:
1. `aria2c` threw multiple errors:
   ```text
   [ERROR] CUID#22 - Download aborted. URI=https://447.minimalhomestore.site/...
   -> [HttpSkipResponseCommand.cc:239] errorCode=22 The response status is not successful. status=403
   ```
2. The multithreaded `urllib` fallback also failed with `HTTP Error 403: Forbidden`, causing 0 images to be downloaded and resulting in `Finished downloading! 0 PDF files saved`.

#### Investigation & Root Cause
In `src/pdf.py`, both `download_single_image()` and `download_images_aria2c()` were appending:
```python
lines.append(f"  header=Referer: {BASE_URL}")
```
Comix.to uses third-party CDN mirrors for manga pages (e.g., `*.minimalhomestore.site`, `*.wanderingsoulblog.site`). Testing header variations against these CDNs revealed:
- `Referer: https://comix.to` -> `HTTP 403 Forbidden` (explicit anti-hotlinking / referrer block)
- No `Referer` header -> `HTTP 200 OK` (full image downloaded successfully)

#### Solution
Updated `src/pdf.py` so that `Referer` is only set if the target URL actually belongs to `comix.to`:
```python
# src/pdf.py - download_single_image
headers = {
    "User-Agent": USER_AGENT,
}
if "comix.to" in img_url:
    headers["Referer"] = BASE_URL
```
```python
# src/pdf.py - download_images_aria2c
lines.append(f"  header=User-Agent: {USER_AGENT}")
if "comix.to" in url:
    lines.append(f"  header=Referer: {BASE_URL}")
```
Once updated, `aria2c` downloaded all pages at maximum speed (~1.5 MB/s) with zero errors.

---

### Problem 4: Series Cover Poster Failing to Download

#### Symptom
The `cover.jpg` file was missing from the download output folder because `download_cover()` failed silently.

#### Investigation & Root Cause
- Poster images are hosted on `static.comix.to` (e.g., `https://static.comix.to/0048/i/1/4d/6a9ce28d23f0a.jpg`).
- `static.comix.to` is behind Cloudflare protection and rejects raw `urllib` requests with `HTTP 403 Forbidden`.
- It requires valid Cloudflare cookies and TLS fingerprint impersonation.

#### Solution
Enhanced `download_cover()` in `src/metadata.py` to first attempt downloading with `curl_cffi` using `impersonate="chrome"` and the active session cookies:
```python
# src/metadata.py - download_cover
try:
    from curl_cffi import requests as cffi_requests
    from .cookies import resolve_cookies
    cookie_str = resolve_cookies()
    cookie_dict = (
        {k.strip(): v.strip() for k, v in [c.split("=", 1) for c in cookie_str.split("; ") if "=" in c]}
        if cookie_str else None
    )
    resp = cffi_requests.get(poster_url, headers=headers, cookies=cookie_dict, impersonate="chrome", timeout=20)
    if resp.status_code == 200 and len(resp.content) > 500:
        save_file.write_bytes(resp.content)
        return True
except Exception:
    pass
```
The official series cover (`cover.jpg`, 52 KB) was successfully downloaded and embedded into the PDFs and `ComicInfo.xml`.

---

## 3. Verification & Results

### Download Verification
Ran:
```bash
python -u main.py rrleg-black-clover -c 1-10
```
All 10 chapters completed successfully:

```text
[OK] Finished downloading! 10 PDF files saved in:
    E:\NAN\Github\comixloop\comix-downloader\downloads\Black Clover
```

Files produced in `downloads/Black Clover/`:
- `cover.jpg` (Official poster art)
- `ComicInfo.xml` (Metadata for Komga/Kavita/Calibre)
- `Black Clover - Ch 001 - Page 1 The Boy's Vow [MangaPlus].pdf` (58.1 MB)
- `Black Clover - Ch 002 - Page 2 The Magic Knights Entrance Exam [MangaPlus].pdf` (25.8 MB)
- `Black Clover - Ch 003 - Page 3 The Road to the Wizard King [MangaPlus].pdf` (24.4 MB)
- `Black Clover - Ch 004 - Page 4 The Black Bulls [MangaPlus].pdf` (18.0 MB)
- `Black Clover - Ch 005 - Page 5 The Other New Member [MangaPlus].pdf` (19.8 MB)
- `Black Clover - Ch 006 - Page 6 Go! Go! First Mission [MangaPlus].pdf` (19.1 MB)
- `Black Clover - Ch 007 - Chapter 7 - Volume 1 (DigitalMangaFan) [VIZ Media].pdf` (25.2 MB)
- `Black Clover - Ch 008 - Page 8 Those Who Protect [MangaPlus].pdf` (7.5 MB)
- `Black Clover - Ch 009 - Page 9 The Boy's Vow Part 2 [MangaPlus].pdf` (6.9 MB)
- `Black Clover - Ch 010 - Chapter 10 - Volume 2 (DigitalMangaFan) [VIZ Media].pdf` (25.7 MB)

### Test Suite & Lint Verification
- **Unit Tests**: `pytest` -> `99 passed in 2.21s`
- **Linter**: `ruff check .` -> `All checks passed!`
