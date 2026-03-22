<div align="center">

# ✦ SonicBoard

### **Download any Spotify playlist as MP3 320k - straight to your PC**
*Full metadata · Synced lyrics · Album art · Genre tags · Smart audio matching*

<br/>

[![Open in Colab](https://img.shields.io/badge/Open%20in-Google%20Colab-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/drive/1QVfcnIhHYnbGBeilF8-IROJ5g81zk21U?usp=sharing)
[![Spotify API](https://img.shields.io/badge/Spotify-Web%20API%20v2026-1DB954?style=for-the-badge&logo=spotify&logoColor=white)](https://developer.spotify.com/documentation/web-api)
[![License: MIT](https://img.shields.io/badge/License-MIT-0ea5e9?style=for-the-badge)](LICENSE)

<br/>

---

</div>

## ✦ What is SonicBoard?

**SonicBoard** is a Google Colab notebook that turns any Spotify playlist into a clean, fully-tagged MP3 collection - downloaded as a ZIP directly to your computer. No CLI setup. No local Python environment. No subscriptions. Just paste, run, and receive.

| Feature | Details |
|---|---|
| 🎵 **Audio Quality** | MP3 at 320 kbps via `yt-dlp` |
| 🏷️ **Metadata** | Title, Artist, Album Artist, Album, Year, Track №, Disc №, ISRC, Spotify URL, Cover Art, Genre |
| 📝 **Lyrics** | Synced `.lrc` preferred; 4-source cascade: LRCLib → lyrics.ovh → Genius → syncedlyrics |
| 🎭 **No Lyrics Fallback** | Animated placeholder `.lrc` embedded into MP3 - works in any folder |
| 🎯 **Smart Audio Matching** | Scores up to 8 candidates before downloading; rejects karaoke, instrumentals, covers, tributes automatically |
| 🔇 **Audio Track Guard** | `bestaudio[acodec!=none]` format filter prevents silent/background-only alternate streams |
| 🧹 **Clean Audio** | SponsorBlock removes intros, outros, self-promos; duration filter rejects extended mixes |
| 🎼 **Genre Tags** | Spotify artist data with automatic Last.fm fallback (Spotify genres empty since 2025) |
| 📁 **Output** | `Playlist Name/MP3/` and `Playlist Name/LRC/` inside a single ZIP |
| ☁️ **Platform** | Google Colab - runs entirely in the cloud, no local install |

<br/>

---

## 🚀 Quick Start

### 1 - Get Spotify API Credentials *(free, 2 min)*

1. Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard) and log in
2. Click **Create App**
3. Set Redirect URI to `https://open.spotify.com`
4. Under **APIs used**, check ✅ **Web API** → **Save**
5. Copy your **Client ID** and **Client Secret**

> ⚠️ **February 2026 API Note:** Spotify now requires an active **Premium** subscription on the account that owns the API app. Dev Mode apps will return a `403` for non-Premium accounts.

### 2 - Get a Last.fm API Key *(optional but recommended)*

Spotify has silently emptied most artist genre data since early 2025. SonicBoard uses Last.fm as a fallback to keep genre tags accurate.

1. Go to [last.fm/api/account/create](https://www.last.fm/api/account/create)
2. Create a free API account
3. Copy your **API Key** into Cell 2

> If left blank, genre tags will only populate when Spotify returns data directly.

### 3 - Open the Notebook

[![Open in Colab](https://img.shields.io/badge/Open%20in-Google%20Colab-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/drive/1QVfcnIhHYnbGBeilF8-IROJ5g81zk21U?usp=sharing)

### 4 - Run

| Cell | Action |
|---|---|
| **Cell 1** | Installs all dependencies (`spotipy`, `yt-dlp`, `mutagen`, `ffmpeg`, `syncedlyrics`, etc.) |
| **Cell 2** | Paste your **Spotify Client ID**, **Client Secret**, and optional **Last.fm API Key** |
| **Cell 3** | Paste your **Spotify playlist URL** |
| **Cell 4** | Runs the full downloader - sit back |

`Runtime → Run all` and you're done. A ZIP file will download to your PC automatically when finished.

<br/>

---

## 📂 Output Structure

```
Playlist Name/
├── MP3/
│   ├── Blinding Lights - The Weeknd.mp3
│   ├── Levitating - Dua Lipa.mp3
│   └── ...
└── LRC/
    ├── Blinding Lights - The Weeknd.lrc
    ├── Levitating - Dua Lipa.lrc
    └── ...
```

Each `.mp3` file is embedded with:
- **ID3v2.3 tags**: Title, Artist, Album Artist, Album, Year, Track Number, Disc Number
- **ISRC** (`TSRC`) and Spotify URL (`WOAF`)
- **Album Art** (`APIC`) - fetched from Spotify's CDN
- **Inline Lyrics** (`USLT`) - lyrics always embedded inside the MP3 itself, so they work in any folder regardless of where the `.lrc` file is
- **Genre** (`TCON`) - from Spotify artist data, with Last.fm fallback
- **Comment** (`COMM`) - download date and ISRC for reference

<br/>

---

## 🔧 How It Works

```
Spotify API ──► Playlist metadata + track info (title, artist, ISRC, cover, genre…)
                    │
                    ▼
           YouTube Smart Matching (yt-dlp)
           ┌───────────────────────────────────────────────────────┐
           │  Phase 1 - Candidate scoring                          │
           │    • Fetch up to 8 candidates via extract_flat        │
           │    • Score each: title words + artist name match      │
           │    • Hard -100 penalty: karaoke / instrumental /      │
           │      cover / tribute / backing track / no vocals      │
           │    • Bonus: official / audio / vevo signals           │
           │    • Duration gate: ±35–45 s window                   │
           │    • Download the highest-scoring candidate directly  │
           │                                                       │
           │  Phase 2 - Plain-query fallback (if Phase 1 fails)    │
           │    • Simple ytsearch1 on "Artist Title"               │
           │    • Wider duration window (±45–60 s)                 │
           │                                                       │
           │  Always: bestaudio[acodec!=none] format filter        │
           │          prevents silent alternate audio streams      │
           └───────────────────────────────────────────────────────┘
                    │
                    ▼
           Lyrics Cascade
           ┌───────────────────────────────────────────────┐
           │  1. LRCLib      - synced .lrc (best quality)  │
           │  2. lyrics.ovh  - plain text, no key needed   │
           │  3. Genius      - scraping, Hindi/non-English │
           │  4. syncedlyrics - Musixmatch, Apple, NetEase │
           │  ✗  None found  - funny animated placeholder  │
           └───────────────────────────────────────────────┘
                    │
                    ▼
           Genre Resolution
           ┌───────────────────────────────────────────────┐
           │  1. Spotify artist.genres                     │
           │  2. Last.fm artist.getTopTags (fallback)      │
           └───────────────────────────────────────────────┘
                    │
                    ▼
           mutagen ID3v2.3 embed (lyrics always go into USLT tag)
                    │
                    ▼
           ZIP  ──►  files.download()  ──►  Your PC
```

<br/>

---

## 🎯 Smart Audio Matching

The previous approach searched YouTube and blindly downloaded the first result that passed a duration check. This caused two recurring problems: karaoke/instrumental versions being downloaded instead of originals, and YouTube's multi-track audio system sometimes exposing a background-music-only stream as the "best" audio format.

SonicBoard now uses a two-phase matching system:

**Phase 1 - Scored candidate selection**

Instead of downloading the first result, the script fetches up to 8 candidates using `extract_flat` (fast - no audio downloaded yet) and scores every result before committing to any download.

| Signal | Score |
|---|---|
| Title words matching the track | +3 per word |
| Artist name words matching | +2 per word |
| "official", "audio", "vevo", "music video" in title | +2 each |
| `karaoke`, `instrumental`, `cover`, `tribute`, `backing track`, `no vocals`, `minus one`, `remake`, `rendition` | **−100 (hard penalty)** |

Only the highest-scoring candidate that also passes the duration gate (`±35 s`) gets downloaded - directly by URL, not via another search. If the winning score is deeply negative (all candidates are non-originals), the phase fails cleanly rather than downloading something wrong.

**Phase 2 - Plain fallback**

If Phase 1 finds nothing, a simple `Artist Title` query is used as a last resort with a slightly wider duration window.

**Audio track guard**

The format selector was changed from `bestaudio/best` to `bestaudio[acodec!=none]/bestaudio/best`. This prevents yt-dlp from picking a YouTube multi-track alternate stream (often a background-music-only dub track) that can surface as the top-ranked audio format.

<br/>

---

## 📝 Lyrics System

SonicBoard tries four sources in order, stopping at the first hit:

| Priority | Source | Type | Notes |
|---|---|---|---|
| 1 | **LRCLib** | Synced + Plain | `/api/get` by duration; `/api/search` fallback with best-match picking |
| 2 | **lyrics.ovh** | Plain | Free REST API, no key, no scraping |
| 3 | **Genius** | Plain | Scraping fallback; strong Hindi/non-English coverage |
| 4 | **syncedlyrics** | Synced + Plain | Tries Musixmatch, Apple Music, NetEase; bypasses anti-scrape |

When **no source finds lyrics**, a fun animated placeholder `.lrc` is written instead of leaving the file empty. The placeholder is also embedded directly into the MP3's `USLT` tag - so it shows up correctly in your music player regardless of whether the `.lrc` file is in the same folder as the MP3 or not.

<br/>

---

## 🎼 Genre System

Spotify has largely stopped returning genre data for artists since early 2025. SonicBoard handles this with a two-layer approach:

1. Fetch genres from `sp.artist()` - used if Spotify returns any
2. Fall back to **Last.fm `artist.getTopTags`** - filters out noise tags (e.g. `seen live`, `favorites`, `beautiful`) and only keeps tags with a community count ≥ 10

Results are cached per artist to avoid redundant API calls across the same playlist.

<br/>

---

## 🛠️ Tech Stack

| Library | Role |
|---|---|
| [`spotipy`](https://spotipy.readthedocs.io/) | Spotify Web API client - playlist & artist data |
| [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) | YouTube audio extraction & candidate scoring |
| [`ffmpeg`](https://ffmpeg.org/) | Audio transcoding to MP3 320k |
| [`mutagen`](https://mutagen.readthedocs.io/) | ID3v2.3 metadata embedding |
| [`requests`](https://requests.readthedocs.io/) | HTTP - cover art, lyrics APIs |
| [`beautifulsoup4`](https://www.crummy.com/software/BeautifulSoup/) | HTML parsing for Genius lyrics scraping |
| [`syncedlyrics`](https://github.com/moisesmorillo/syncedlyrics) | Lyrics package - Musixmatch, Apple Music, NetEase |
| [`LRCLib`](https://lrclib.net/) | Primary lyrics source (synced + plain) |
| [`lyrics.ovh`](https://lyrics.ovh/) | Secondary lyrics source (plain, no key) |
| [`Last.fm API`](https://www.last.fm/api) | Genre fallback (`artist.getTopTags`) |

<br/>

---

## ⚠️ Spotify API Compatibility

This notebook is updated for the **February 2026 Spotify Web API changes**:

| Change | Old | Fixed |
|---|---|---|
| Playlist items endpoint | `sp.playlist_tracks()` → `/tracks` | `sp.playlist_items()` → `/items` ✅ |
| Track key in response | `item['track']` only | `item.get('track') or item.get('item')` ✅ |
| Dev Mode requirement | Any account | **Premium required** on app-owner account |
| Artist genre data | Usually populated | Mostly empty - Last.fm fallback added ✅ |

<br/>

---

## 🙋 FAQ

**Q: Is this legal?**
SonicBoard downloads audio from YouTube, not from Spotify directly. Usage is subject to YouTube's Terms of Service. This tool is intended for personal, offline use of content you have rights to access.

**Q: Will it work on free Spotify accounts?**
The Spotify API credentials (Client ID/Secret) need to belong to a **Premium** account owner as of February 2026. The playlist itself can be public.

**Q: What if a track fails to download?**
The script will skip it, log it as `failed`, and continue. A summary is printed at the end. You can re-run the notebook - already downloaded files are skipped automatically.

**Q: Can I use this for albums or single tracks?**
Currently playlists only. Album and track support is a planned enhancement - PRs welcome.

**Q: Why did I get an instrumental or karaoke version before?**
The old script downloaded the first YouTube result that matched the duration, with no check of what the video actually was. The new smart matching system scores up to 8 candidates and applies a hard penalty to any result whose title contains karaoke, instrumental, cover, tribute, or similar keywords. See the [Smart Audio Matching](#-smart-audio-matching) section for full details.

**Q: What if lyrics aren't found?**
All four sources are tried in order (LRCLib → lyrics.ovh → Genius → syncedlyrics). If all fail, a fun animated placeholder `.lrc` is created and embedded into the MP3 so it always displays correctly in your music player.

**Q: Why do lyrics work from any folder?**
Lyrics are embedded directly inside the MP3 file via the `USLT` ID3 tag. Your player reads them from the file itself - it doesn't need to find a matching `.lrc` file in the same folder. The separate `.lrc` file in the `LRC/` folder is a bonus for players that support external synced lyrics.

**Q: Do I need a Last.fm API key?**
It's optional but recommended. Without it, genre tags will only appear when Spotify returns data directly, which has been rare since early 2025.

<br/>

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

<br/>

---

<div align="center">

Made with 🎧 and too much chai

*If SonicBoard saved you time, consider starring the repo ⭐*

</div>
