### Quick facts

* `gau` fetches **known URLs** for a domain from archives like Wayback Machine, Common Crawl, AlienVault OTX and URLScan. ([GitHub][1])

---

# 1) Install

You have two easy options on Kali:

A. From Kali repo (quick):

```bash
sudo apt update
sudo apt install getallurls
```

B. From source (Go) — gives latest:

```bash
# ensure Go is installed
go install github.com/lc/gau/v2/cmd/gau@latest
# make binary available if needed
sudo mv ~/go/bin/gau /usr/local/bin/
```

(You can also create a `~/.gau.toml` for config if you want). ([Kali Linux][2])

---

# 2) Basic usage

Fetch URLs for a domain:

```bash
gau example.com
```

Save output:

```bash
gau example.com > urls.txt
```

Feed domains from stdin (handy for scripts):

```bash
echo example.com | gau
cat domains.txt | gau
```

Include URLs from subdomains:

```bash
gau --subs example.com
```

(Useful when you already enumerated subdomains and want archive URLs for them). ([OSINT Team][3])

---

# 3) Common pipelines (real-world)

A. **Get unique JS files** (great for hunting secrets / endpoints in JS):

```bash
gau example.com \
  | grep -E '\.js([?#].*)?$' \
  | sed 's/#.*$//; s/\?.*$//' \
  | sort -u > gau_js.txt
```

B. **Get potential endpoints (strip query strings, dedupe)**:

```bash
gau example.com \
  | sed 's/#.*$//; s/\?.*$//' \
  | sort -u > gau_endpoints.txt
```

C. **Check which URLs are alive with httpx** (fast live filtering):

```bash
gau example.com | sort -u | httpx -silent -status-code -o gau_alive.txt
```

D. **Pipe into vulnerability scanners** (e.g., `gf` patterns, `nuclei`, `ffuf`):

```bash
gau example.com | sort -u | gf xss | tee xss_candidates.txt
gau example.com | sort -u | nuclei -t ~/nuclei-templates/ -o nuclei_results.txt
```

(Replace `gf xss` / `nuclei` usage to match your local setup).

---

# 4) Tips & best practices

* **Combine sources**: run both `gau` and `waybackurls` (and `wayback_machine_downloader` or `commoncrawl` tools) to maximize coverage.
* **Deduplicate & normalize**: remove fragments (`#...`) and query strings when you want canonical endpoints.
* **Filter by extension** (JS, PHP, JSON, etc.) with `grep -Ei '\.js$'` or regex.
* **Use subdomain lists**: when you have many subdomains, feed them into `gau --subs` to pull archive URLs across them. ([Medium][4])

---

# 5) Example full workflow (one-liner)

Find JS files, check alive, and scan for interesting patterns:

```bash
gau example.com --subs \
  | grep -E '\.js([?#].*)?$' \
  | sed 's/#.*$//; s/\?.*$//' \
  | sort -u \
  | httpx -silent -status-code -o alive_js.txt \
  && cat alive_js.txt | gf ssti | tee js_ssti_candidates.txt
```

---


