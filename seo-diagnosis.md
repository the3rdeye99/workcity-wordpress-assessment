# SEO Technical Troubleshooting: Website Not Indexing

**Scenario:** A new Clean & Shine brand website is not indexing even after sitemap submission.

**Objective:** Complete diagnostic framework to identify and resolve indexing issues.

**Status:** Production troubleshooting guide  
**Created:** November 25, 2025  
**For:** WordPress SEO Assessment

---

## Table of Contents

1. [Quick Diagnosis Checklist](#quick-diagnosis-checklist)
2. [Crawlability Tests](#crawlability-tests)
3. [Canonical Tags Audit](#canonical-tags-audit)
4. [Robots.txt & No-Index Audit](#robotstxt--no-index-audit)
5. [Sitemap Structure Issues](#sitemap-structure-issues)
6. [Page Speed Indexing Blockers](#page-speed-indexing-blockers)
7. [Search Console Debugging](#search-console-debugging-steps)
8. [Index Status Monitoring](#index-status-monitoring)
9. [Recovery Action Plan](#recovery-action-plan)

---

## Quick Diagnosis Checklist

Use this checklist to identify the likely cause:

```
□ CRITICAL BLOCKERS (Fix immediately if checked)
  □ robots.txt blocking all crawlers (Disallow: /)
  □ Noindex meta tag on all pages
  □ No Search Console property verified
  □ 5xx errors on homepage
  □ All links are JavaScript only (no HTML links)

□ MAJOR ISSUES (Usually prevent indexing)
  □ Canonical pointing to different domain
  □ Infinite redirect loops
  □ Paywall/login required on homepage
  □ Robots.txt blocking sitemap
  □ Sitemap.xml file not accessible (404)
  □ Core Web Vitals failing (LCP >4s, CLS >0.25, FID >100ms)

□ CONTRIBUTING FACTORS (May slow or block indexing)
  □ New domain (less than 30 days)
  □ Very slow page load (>5 seconds)
  □ Duplicate content across domains
  □ Soft 404 responses
  □ Too many redirect chains (>2)
  □ CSS/JS blocked by robots.txt
  □ Incorrect XML sitemap format
```

---

## Crawlability Tests

### 1. Homepage Accessibility Test

**Goal:** Verify Google can access and render your homepage.

**Method A: Direct URL Test (Search Console)**

1. Go to Google Search Console
2. Click on the property for cleanandshine.com
3. Click **URL Inspection**
4. Enter: `https://cleanandshine.com/`
5. Click **Test live URL**

**Expected Result:**
- ✅ "URL is available to Google"
- ✅ Last crawled: Within 7 days
- ✅ Coverage: Indexed or valid in test

**If Failed:**
- ⚠️ "Discovered - currently not indexed" → Go to [Sitemap Issues](#sitemap-structure-issues)
- ❌ "URL not found (404)" → Homepage is returning 404 error
- ❌ "Redirect error" → Fix redirect chains
- ❌ "Not available to Google" → robots.txt blocking

**Diagnosis Code:**
```powershell
# Test homepage HTTP status
$response = Invoke-WebRequest -Uri "https://cleanandshine.com/" -Method HEAD
$response.StatusCode  # Should be 200
```

**Expected Output:** `200` (success)  
**Problem Codes:**
- `301/302` = Redirect (check destination)
- `401/403` = Authentication required
- `404` = Page not found
- `5xx` = Server error

---

### 2. Robots.txt Accessibility Test

**Goal:** Ensure robots.txt is valid and not blocking crawlers.

**Manual Check:**

```
https://cleanandshine.com/robots.txt
```

**Expected Format:**
```
User-agent: *
Allow: /
Disallow: /wp-admin/
Disallow: /wp-includes/
Disallow: /wp-json/
Disallow: /wp-login.php

Sitemap: https://cleanandshine.com/sitemap.xml
```

**Critical Issues to Find:**

❌ **BLOCKER: Disallow: /all pages**
```
User-agent: *
Disallow: /  ← This blocks ALL pages!
```
**Fix:** Remove or comment out. Should be `Allow: /`

❌ **BLOCKER: Disallow: Sitemap line**
```
Disallow: /sitemap
```
**Fix:** Remove any blocks on /sitemap path

❌ **PROBLEM: No Sitemap directive**
```
# Missing: Sitemap: https://cleanandshine.com/sitemap.xml
```
**Fix:** Add: `Sitemap: https://cleanandshine.com/sitemap.xml`

❌ **PROBLEM: Wrong User-agent**
```
User-agent: Googlebot-Image  ← Too specific!
Disallow: /
```
**Fix:** Use `User-agent: *` for all bots or `User-agent: Googlebot`

**WordPress robots.txt Location:**

In WordPress, robots.txt is generated dynamically if file doesn't exist physically:

```
# Physical location (if created):
/robots.txt

# If not present, WordPress generates from settings:
Settings > Reading > Search Engine Visibility
```

**Check WordPress Setting:**
1. Go to: `Dashboard > Settings > Reading`
2. Look for: "Search Engine Visibility"
3. ⚠️ If checked "Discourage search engines..." → **UNCHECK THIS**
4. Save changes

**Diagnostic Code:**
```php
<?php
// Check if WordPress is discouraging search engines
$discourage = get_option('blog_public');
echo $discourage ? 'INDEXED' : 'DISCOURAGING SEARCH ENGINES (PROBLEM!)';
?>
```

---

### 3. Crawl Budget Test

**Goal:** Verify Google has sufficient crawl budget for your site.

**Check in Search Console:**

1. Go to **Settings** (left sidebar)
2. Click **Crawl Stats** (under "About this property")
3. Review last 90 days:
   - **Requests per day** (should be >0)
   - **KB downloaded per day**
   - **Response time**

**Analysis:**

✅ **Good Signs:**
- Requests per day: 10+
- Response time: <500ms
- Consistent daily crawling
- No spike in errors

⚠️ **Warning Signs:**
- Requests per day: 0-5 (very low)
- Response time: >2000ms (slow)
- Sudden drop in requests
- Spike in 5xx errors

❌ **Critical Problems:**
- Requests per day: 0 (no crawling!)
- Response time: >5000ms (extremely slow)
- Consistent high error rates (>50%)

**If Crawl Budget is Zero:**

This means Google is not crawling your site. Causes:
1. Robots.txt blocking everything
2. Site returns 5xx errors
3. Site is completely unavailable
4. New domain (less than 2 weeks old)

**Resolution:**
- Fix robots.txt (see [Robots.txt Audit](#robotstxt--no-index-audit))
- Fix server errors
- Request crawl in Search Console (see [Request Indexing](#request-indexing))

---

### 4. Page Rendering Test

**Goal:** Verify Google can render JavaScript on your pages.

**Method A: Search Console URL Inspection**

1. In URL Inspection, click **Screenshot**
2. Compare "Googlebot" screenshot vs browser screenshot
3. If different → JavaScript not rendering for Google

**Expected:** Screenshots should be nearly identical

**Problem:** Blank areas in Googlebot screenshot indicate:
- CSS/JS blocked by robots.txt
- Dynamic content not loading
- JavaScript errors during render

**Method B: Manual Testing**

```bash
# Fetch page as Googlebot
curl -H "User-Agent: Mozilla/5.0 (Linux; Android 6.0.1; Nexus 5X Build/MMB29K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/W.X.Y.Z Mobile Safari/537.36 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://cleanandshine.com/
```

**Check for:**
- ✅ Main content present (not blank)
- ✅ All hero images loaded
- ✅ Navigation menus visible
- ✅ Contact forms rendered
- ✅ Schema markup present

---

## Canonical Tags Audit

### 1. Homepage Canonical Check

**Location to Check:**
```html
<head>
  <link rel="canonical" href="https://cleanandshine.com/" />
</head>
```

**Expected Values:**
```html
<!-- Option 1: Self-referential (recommended) -->
<link rel="canonical" href="https://cleanandshine.com/" />

<!-- Option 2: Explicit homepage -->
<link rel="canonical" href="https://cleanandshine.com/index.html" />
```

❌ **PROBLEM: Canonical to different domain**
```html
<link rel="canonical" href="https://oldsite.com/" />
```
**Impact:** Google won't index your site; will index old domain instead.  
**Fix:** Change to cleanandshine.com

❌ **PROBLEM: Canonical to HTTP (site is HTTPS)**
```html
<link rel="canonical" href="http://cleanandshine.com/" />
```
**Impact:** Duplicate content; split authority between protocols.  
**Fix:** Use HTTPS: `href="https://cleanandshine.com/"`

❌ **PROBLEM: Canonical to non-existent page**
```html
<link rel="canonical" href="https://cleanandshine.com/nonexistent/" />
```
**Impact:** Google follows canonical to 404; won't index.  
**Fix:** Point to valid page URL

**Diagnostic Code (WordPress):**

```php
<?php
// Check canonical tag on homepage
$canonical = wp_get_canonical_url();
echo 'Canonical: ' . esc_url($canonical);

// Verify it matches site URL
$site_url = home_url('/');
if ($canonical === $site_url) {
    echo ' ✅ CORRECT';
} else {
    echo ' ⚠️ MISMATCH! Expected: ' . esc_url($site_url);
}
?>
```

### 2. Audit All Pages for Canonical Issues

**WordPress Plugin Method:**

1. Install **Yoast SEO** (if not already)
2. Go to: `Yoast SEO > Tools > Import & Export`
3. Click **Export All Canonicals**
4. Review exported CSV for issues:
   - All should be HTTPS
   - All should be same domain
   - None should point to different URLs

**Manual Audit (cURL method):**

```bash
#!/bin/bash
# Test canonicals on all pages

for url in \
    "https://cleanandshine.com/" \
    "https://cleanandshine.com/about/" \
    "https://cleanandshine.com/services/" \
    "https://cleanandshine.com/contact/"; do
  
  echo "URL: $url"
  canonical=$(curl -s "$url" | grep -oP 'rel="canonical" href="\K[^"]+' || echo "NOT FOUND")
  echo "Canonical: $canonical"
  echo ""
done
```

### 3. HTTP to HTTPS Redirect Check

**Test redirect chain:**

```powershell
# Test if HTTP redirects to HTTPS
Invoke-WebRequest -Uri "http://cleanandshine.com/" -MaximumRedirection 0 | Select-Object StatusCode, Location
```

**Expected Result:**
```
StatusCode Location
----------- --------
301 https://cleanandshine.com/
```

**Problem Scenarios:**

❌ **Too many redirects (>2)**
```
http → https → www → non-www → /index.html → /
```
**Fix:** Consolidate to one redirect: `http://cleanandshine.com → https://cleanandshine.com`

---

## Robots.txt & No-Index Audit

### 1. Search for Noindex Meta Tag

**Goal:** Ensure noindex meta tag is NOT present on pages you want indexed.

**Location to Check (Page Head):**

```html
<!-- ❌ CRITICAL BLOCKER if present -->
<meta name="robots" content="noindex" />
<!-- or -->
<meta name="googlebot" content="noindex" />
```

**Why This Blocks Indexing:**
Explicitly tells Google: "Do not index this page"

**How to Find It:**

**Method A: Browser DevTools**
1. Visit page
2. Press F12 to open DevTools
3. Go to **Elements** tab
4. Press Ctrl+F to search
5. Search: `noindex`
6. If found anywhere → **REMOVE IT**

**Method B: View Page Source**
1. Right-click page → **View Page Source**
2. Press Ctrl+F
3. Search: `noindex`

**Method C: cURL**
```bash
curl -s https://cleanandshine.com/ | grep -i "noindex"
```

**If Noindex Found: Where to Remove It**

**WordPress Settings:**

1. Go to: `Settings > Reading`
2. Check: "Search Engine Visibility"
3. ⚠️ If checked → **UNCHECK** (this adds noindex)
4. Save changes

**Yoast SEO Plugin:**
1. Edit page
2. Look for: **Yoast SEO Meta Box** (bottom of page)
3. Click **Advanced** tab
4. Set: "Allow search engines to show this post in search results?" = **YES**
5. Save page

**All in One SEO:**
1. Edit page
2. Scroll to: **All in One SEO**
3. Under: **Advanced Tab**
4. Set: "Robots Meta" = **NOT SET** (or blank)
5. Save

---

### 2. Robots.txt Directive Audit

**WordPress robots.txt Setup:**

Since WordPress generates robots.txt dynamically, edit it in Settings:

1. Navigate to: `/wp-admin`
2. Go to: **Tools > Registered Settings** (or create physical file)
3. Recommended content:

```
User-agent: *
Allow: /
Disallow: /wp-admin/
Disallow: /wp-includes/
Disallow: /wp-login.php
Disallow: /wp-signup.php
Disallow: /wp-trackback.php
Disallow: /?s=
Disallow: /search/
Disallow: /cart/
Disallow: /checkout/

# Block access logs
Disallow: /access_log

# Allow search engines to crawl assets
Allow: /*.css$
Allow: /*.js$
Allow: /*.jpg$
Allow: /*.jpeg$
Allow: /*.png$
Allow: /*.gif$
Allow: /*.webp$

Sitemap: https://cleanandshine.com/sitemap.xml
Sitemap: https://cleanandshine.com/sitemap-posts.xml
Sitemap: https://cleanandshine.com/sitemap-pages.xml
```

**Critical Settings:**

✅ **CORRECT:**
```
User-agent: *
Allow: /
```

❌ **WRONG - Blocks everything:**
```
User-agent: *
Disallow: /
```

❌ **WRONG - Blocks CSS/JS:**
```
Disallow: /*.css$
Disallow: /*.js$
```

---

### 3. No-Index on Important Pages

**Check These Critical Pages:**

```
https://cleanandshine.com/               Homepage
https://cleanandshine.com/about/          About page
https://cleanandshine.com/services/       Services
https://cleanandshine.com/contact/        Contact
https://cleanandshine.com/blog/          Blog
```

**For Each Page:**

1. View page source (Ctrl+U)
2. Search (Ctrl+F): `noindex`
3. If found: ❌ PROBLEM - Must remove
4. If not found: ✅ OK - Page can be indexed

**To Remove Noindex from Single Page (WordPress):**

**Using Yoast SEO:**
1. Edit the page/post
2. Scroll to: **Yoast SEO** section
3. Click: **Advanced** tab
4. Find: "Allow search engines to show this post in search results?"
5. Set to: **Default** or **Yes**
6. Click **Update**

**Using All in One SEO:**
1. Edit the page/post
2. Scroll to: **All in One SEO** section
3. Find: **Robots Meta**
4. Select: **Inherit** or leave blank
5. Click **Update**

---

## Sitemap Structure Issues

### 1. Sitemap XML Validation

**Goal:** Ensure sitemap.xml is valid and accessible.

**Step 1: Check if sitemap exists**

Navigate to:
```
https://cleanandshine.com/sitemap.xml
```

**Expected Result:**
- ✅ Returns XML content (not 404 error)
- ✅ Shows proper XML structure
- ✅ Lists all pages to index

**If 404 Error:**

**Cause:** Sitemap plugin not activated or not configured.

**Fix (WordPress):**

1. Install **Yoast SEO** plugin (most reliable)
2. Go to: `Yoast SEO > Tools > Import & Export`
3. Check: **Enable XML sitemaps** ✓
4. Go to: `Yoast SEO > Sitemaps`
5. Verify sitemaps are generated:
   - `sitemap.xml` (index)
   - `sitemap-posts.xml`
   - `sitemap-pages.xml`
6. Test: https://cleanandshine.com/sitemap.xml

**Alternative (Google XML Sitemaps plugin):**
1. Install: **Google XML Sitemaps**
2. Go to: `XML-Sitemap > Sitemap Settings`
3. Check: **Post types** (at least Pages and Posts)
4. Click: **Build the sitemap**
5. Test: https://cleanandshine.com/sitemap.xml

---

### 2. Sitemap Format Validation

**Correct XML Structure:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://cleanandshine.com/</loc>
    <lastmod>2025-11-25T00:00:00+00:00</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://cleanandshine.com/services/</loc>
    <lastmod>2025-11-24T12:00:00+00:00</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

**Critical Requirements:**

✅ **MUST HAVE:**
- XML declaration: `<?xml version="1.0" encoding="UTF-8"?>`
- Proper namespace: `xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"`
- Closing `</urlset>` tag
- URLs in `<loc>` tags

❌ **COMMON ERRORS:**

**Error: Missing XML declaration**
```xml
<!-- ❌ WRONG - Missing first line -->
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
```
**Fix:** Add: `<?xml version="1.0" encoding="UTF-8"?>`

**Error: Wrong namespace**
```xml
<!-- ❌ WRONG -->
<urlset xmlns="http://www.sitemaps.org/sitemap/0.9">
                ↑ missing 'schemas'
```
**Fix:** Use: `http://www.sitemaps.org/schemas/sitemap/0.9`

**Error: Unescaped special characters**
```xml
<!-- ❌ WRONG - Ampersand not escaped -->
<loc>https://cleanandshine.com/?page=1&sort=price</loc>

<!-- ✅ CORRECT - Ampersand escaped -->
<loc>https://cleanandshine.com/?page=1&amp;sort=price</loc>
```

---

### 3. Sitemap Size & Limits

**Rules:**

- Maximum 50,000 URLs per sitemap file
- Maximum 50 MB per file (uncompressed)
- If over limits: Create multiple sitemaps + index file

**Example Multi-Sitemap Structure:**

```
sitemap.xml (index file)
  ├─ sitemap-posts.xml (URLs 1-50,000)
  ├─ sitemap-pages.xml (URLs 50,001-100,000)
  └─ sitemap-products.xml (additional)
```

**Index file format:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://cleanandshine.com/sitemap-posts.xml</loc>
    <lastmod>2025-11-25T00:00:00+00:00</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://cleanandshine.com/sitemap-pages.xml</loc>
    <lastmod>2025-11-25T00:00:00+00:00</lastmod>
  </sitemap>
</sitemapindex>
```

**Check Sitemap Size:**

```powershell
# Check sitemap file size
$sitemap = Invoke-WebRequest -Uri "https://cleanandshine.com/sitemap.xml"
$size_mb = ([xml]$sitemap.Content).SelectNodes("//url").Count
Write-Host "Total URLs in sitemap: $size_mb"
```

---

### 4. Sitemap Submission & Registration

**Step 1: Verify in Search Console**

1. Go to: **Google Search Console > Sitemaps** (left sidebar)
2. Click: **New Sitemap**
3. Enter: `https://cleanandshine.com/sitemap.xml`
4. Click: **Submit**

**Expected Result:**
- Status: **Success** or **Submitted**
- Last submitted: Today
- Issues discovered: 0 (ideally)

**Step 2: Check Submission Status**

Return to **Sitemaps** page, click your sitemap:

✅ **Healthy Status:**
- Status: `Success`
- Discovered URLs: >0
- Indexed URLs: >0
- Errors: 0

⚠️ **Warning Signs:**
- Discovered URLs: 0
- Indexed URLs: 0 (but discovered > 0)
- Errors: 1+ (click to see details)

❌ **Critical Problems:**
- Status: `Error`
- Errors: Multiple
- No discovered URLs

**If Sitemap Has Errors:**

1. Click on sitemap in Search Console
2. Click on the error count
3. Note specific error message
4. Common errors:
   - `404 (Not Found)` - URL doesn't exist
   - `Crawl rate` - Too slow, reduce URLs
   - `Duplicate` - Same URL in multiple sitemaps
   - `Redirect` - URL redirects elsewhere

---

## Page Speed Indexing Blockers

### 1. Core Web Vitals Analysis

**Why This Matters:** Pages with poor speed metrics are deprioritized for indexing.

**Three Core Metrics:**

| Metric | Good | Needs Improvement | Poor |
|--------|------|------------------|------|
| **LCP** (Largest Contentful Paint) | <2.5s | 2.5s-4s | >4s ❌ |
| **FID** (First Input Delay) | <100ms | 100ms-300ms | >300ms ❌ |
| **CLS** (Cumulative Layout Shift) | <0.1 | 0.1-0.25 | >0.25 ❌ |

**Check Your Metrics:**

**Method A: Search Console**
1. Go to: **Experience > Core Web Vitals**
2. View: Mobile and Desktop reports
3. See: "Good", "Needs Improvement", "Poor"

**Method B: Google PageSpeed Insights**
1. Visit: https://pagespeed.web.dev/
2. Enter: `https://cleanandshine.com`
3. Review scores for Mobile & Desktop
4. See: Performance, Accessibility, Best Practices, SEO scores

**Method C: Chrome DevTools**
1. Press F12 on the page
2. Go to **Lighthouse** tab
3. Click **Analyze page load**
4. View: Performance score (target: 90+)

---

### 2. Optimize Page Speed

**Quick Wins (implement first):**

**Enable Caching:**
```php
// WordPress functions.php
// Add object caching
define('WP_CACHE', true);
```

**Install Caching Plugin:**
- **WP Super Cache** (easiest)
- **W3 Total Cache** (most powerful)
- **LiteSpeed Cache** (if on LiteSpeed server)

**Compress Images:**

1. Install **Imagify** or **ShortPixel**
2. Go to: `Imagify > Bulk Optimization`
3. Compress all images
4. Set quality: **Recommended** (usually 82%)

**Minify CSS/JavaScript:**

Using Yoast SEO:
1. Go to: `Yoast SEO > Settings > Advanced`
2. Enable: **Minify CSS** ✓
3. Enable: **Minify JavaScript** ✓
4. Save

**Lazy Load Images:**

```php
// Add to functions.php
add_filter('wp_lazy_loading_enabled', '__return_true');
```

Or use plugin: **Lazy Load by WP Rocket**

---

### 3. Server Response Time

**Test Server Response Time:**

```bash
# Using curl
time curl -o /dev/null -s -w "%{time_total}\n" https://cleanandshine.com/
```

**Expected:** <0.5 seconds  
**Acceptable:** <1 second  
**Problem:** >2 seconds ❌

**If Response Time is Slow:**

1. **Check Server Load:**
   - Access cPanel or hosting control panel
   - View CPU usage (should be <50%)
   - View Memory usage (should be <70%)

2. **Optimize Database:**
   ```php
   // WordPress database optimization plugin
   // Install: WP-Optimize or Advanced Database Cleaner
   ```

3. **Enable CDN:**
   - Use **Cloudflare** (free tier available)
   - Automatically caches and serves from edge locations
   - Can reduce response time by 50%+

---

## Search Console Debugging Steps

### 1. Property Setup & Verification

**Step 1: Create or Verify Property**

1. Go to: https://search.google.com/search-console
2. Click: **+ Create property**
3. Choose: **URL prefix** (recommended)
4. Enter: `https://cleanandshine.com`
5. Click: **Continue**

**Step 2: Verify Ownership**

Choose verification method:

**Method A: HTML file upload** (most reliable)
1. Download verification HTML file from Google
2. Upload to: `/public_html/` (root directory)
3. Click **Verify** in Search Console

**Method B: HTML meta tag**
1. Copy meta tag from Google
2. Add to WordPress theme header:
   ```html
   <!-- wp-content/themes/workcity/header.php -->
   <head>
   <?php wp_head(); ?>
   <!-- Paste here -->
   <meta name="google-site-verification" content="XXXXX" />
   </head>
   ```
3. Click **Verify** in Search Console

**Method C: Google Analytics**
If you have GA4 connected:
1. Click **Verify via Google Analytics**
2. Requires GA property linked to domain

---

### 2. Index Coverage Report Analysis

**Navigation:** `Search Console > Coverage`

**Report Breakdown:**

| Status | Meaning | Action |
|--------|---------|--------|
| **Indexed** | ✅ Page is in Google Index | Good! Monitor for issues |
| **Excluded** | ⚠️ Google knows page but didn't index | Investigate why |
| **Error** | ❌ Google encountered error | Fix immediately |

**Common Coverage Issues:**

**Issue: "Excluded - Noindex tag"**
- Cause: Noindex meta tag on page
- Fix: Remove noindex (see [Noindex Audit](#robotstxt--no-index-audit))

**Issue: "Excluded - Blocked by robots.txt"**
- Cause: robots.txt preventing access
- Fix: Allow in robots.txt (see [Robots.txt Audit](#robotstxt--no-index-audit))

**Issue: "Excluded - Redirect"**
- Cause: URL redirects to another location
- Fix: Update sitemap to final destination URL

**Issue: "Excluded - Duplicate"**
- Cause: URL has preferred canonical elsewhere
- Fix: Verify canonical is correct

**Issue: "Error - 404 Not Found"**
- Cause: URL doesn't exist on server
- Fix: Remove from sitemap or restore page

---

### 3. Request Indexing for New Pages

**Method A: URL Inspection Tool**

1. In Search Console, click **URL Inspection**
2. Enter: URL of page to index
3. Click: **Request Indexing**
4. Wait for processing (can take 1-2 days)

**Method B: Bulk Request (Sitemap)**

1. Submit sitemap (already submitted)
2. Wait 24-48 hours
3. New pages should appear in index

**Method C: Direct Links**

Add links to indexed pages pointing to new pages:
- Link from homepage
- Link from navigation menu
- Link from blog/news section

Google crawls linked pages faster.

---

### 4. Performance Report Analysis

**Navigation:** `Search Console > Performance`

**Key Metrics to Monitor:**

1. **Total Clicks:** Total clicks from search results
2. **Total Impressions:** Times page appeared in search
3. **Average CTR:** Click-through rate (should be 3-10%)
4. **Average Position:** Average ranking (1-10, 11-50, 51-100, 100+)

**What Good Performance Looks Like:**

```
Homepage:
  Position: 1-5
  Clicks: 10+ per day
  Impressions: 100+ per day
  CTR: 5-15%

Service pages:
  Position: 5-15
  Clicks: 2+ per day
  Impressions: 20+ per day
  CTR: 2-5%
```

**What Bad Performance Looks Like:**

```
Homepage:
  Position: 50+
  Clicks: 0-1 per day
  Impressions: 20-30 per day
  CTR: <1%
  ↑ This means page is ranking low despite being indexed
```

---

### 5. Search Analytics Debugging

**Common Issues:**

**Problem: Page indexed but not ranking**

1. Check **Performance > Top pages**
2. If page shows Position: 50+ with low clicks
3. Likely causes:
   - Keyword too competitive
   - Low-quality content vs competitors
   - Not enough backlinks
   - Low page authority

**Solution:**
- Optimize content for target keyword
- Add backlinks (guest posts, citations)
- Improve on-page SEO (headings, internal links)
- Wait 2-4 weeks for re-ranking

**Problem: Page disappeared from index**

1. Go to **Coverage** report
2. Filter for the URL
3. Check status:
   - If "Error" → Fix error
   - If "Excluded" → Remove exclusion
   - If not listed → Resubmit sitemap

---

## Index Status Monitoring

### 1. Set Up Alerts for Index Changes

**In Search Console:**

1. Go to **Settings > Email preferences**
2. Check: "Email coverage issues" ✓
3. Check: "Email mobile usability issues" ✓
4. Check: "Email indexing issues" ✓
5. Save

You'll receive emails when:
- Pages stop being indexed
- New errors appear
- Coverage changes significantly

---

### 2. Create Index Monitoring Dashboard

**Track Over Time:**

```
Date         | Indexed URLs | Errors | Trend
2025-11-25   | 25          | 0      | Starting
2025-12-02   | 50          | 0      | ✅ Growing
2025-12-09   | 95          | 2      | ⚠️ Slight issues
2025-12-16   | 98          | 0      | ✅ Recovered
```

**Export Data from Search Console:**

1. Go to **Coverage** or **Performance**
2. Click **Download** icon
3. Choose date range (last 90 days)
4. Save as CSV

**Analyze Trends:**
- Indexing should be stable or growing
- Errors should remain at 0 or decrease
- If indexing decreases → Investigate immediately

---

### 3. Weekly Monitoring Checklist

**Every Monday (quick check):**

```
□ Search Console
  □ Check Coverage page (Errors: 0?)
  □ Check Performance page (Impressions growing?)
  □ Check for new issues email

□ Google PageSpeed Insights
  □ Run on homepage
  □ Performance score still 85+?

□ Site Health
  □ Verify site loads quickly
  □ Check for any error messages in logs
  □ Verify contact form works

□ Ranking Check
  □ Google: "site:cleanandshine.com"
  □ Should show 50+ indexed pages
  □ Check homepage in top position

□ Backlink Check
  □ Use Google Search Console > Links
  □ Or use: ahrefs.com / moz.com / semrush.com
  □ Should have 5+ quality backlinks
```

---

## Recovery Action Plan

### For "Not Indexing" Scenario

**Phase 1: Emergency Fixes (Do Immediately)**

Priority: **🔴 CRITICAL**

```
□ Check WordPress "Discourage search engines" setting
  □ Settings > Reading
  □ ❌ UNCHECK "Discourage search engines"
  □ Save changes

□ Verify robots.txt allows indexing
  □ https://cleanandshine.com/robots.txt
  □ Should NOT have: Disallow: /
  □ Should have: Sitemap: https://cleanandshine.com/sitemap.xml

□ Check for noindex meta tags
  □ View page source (Ctrl+U)
  □ Search for "noindex"
  □ Remove if found on homepage/important pages

□ Verify site is accessible
  □ Test: https://cleanandshine.com
  □ Should return 200 status code
  □ Should load in <3 seconds
  □ Homepage should be fully visible
```

**Phase 2: Validation (Next 24 Hours)**

Priority: **🟠 HIGH**

```
□ Verify sitemap.xml is valid
  □ Access: https://cleanandshine.com/sitemap.xml
  □ Should show XML content (not 404)
  □ Should list all important pages

□ Submit sitemap to Search Console
  □ Go to: Sitemaps section
  □ Submit: https://cleanandshine.com/sitemap.xml
  □ Wait for processing (24-48 hours)

□ Request indexing via URL Inspection
  □ Search Console > URL Inspection
  □ Enter homepage URL
  □ Click "Request Indexing"
  □ Repeat for 5-10 key pages

□ Check Search Console for errors
  □ Go to: Coverage page
  □ Note any errors/excluded items
  □ Begin fixing based on error type
```

**Phase 3: Optimization (Days 2-7)**

Priority: **🟡 MEDIUM**

```
□ Optimize page speed
  □ Install WP Super Cache plugin
  □ Compress images with Imagify
  □ Run Google PageSpeed check
  □ Performance score should be 85+

□ Verify canonical tags
  □ Homepage canonical should point to itself
  □ No canonicals pointing to other domains
  □ No HTTP vs HTTPS mismatches

□ Create internal links
  □ Homepage links to top 5 pages
  □ Navigation menu links to all main pages
  □ Footer links to contact/about pages

□ Add schema markup
  □ Organization schema for homepage
  □ Article schema for blog posts
  □ Service schema for service pages
  □ Validate with Google Rich Results Test

□ Submit via Google Search Console
  □ Upload sitemap
  □ Request indexing for key pages
  □ Monitor for errors daily
```

**Phase 4: Monitoring (Week 2+)**

Priority: 🔵 NORMAL

```
□ Monitor index growth
  □ Check Search Console daily
  □ Should see steady index growth
  □ Track indexed URL count

□ Monitor search rankings
  □ Check: site:cleanandshine.com (should increase)
  □ Monitor homepage in Google Search
  □ Should appear in top 3 within 2-4 weeks

□ Monitor errors
  □ Check Search Console Coverage page
  □ Should stay at 0 errors
  □ If new errors appear → fix immediately

□ Build backlinks
  □ Reach out for guest post opportunities
  □ Get business citations from directories
  □ Add to local business listings
  □ Goal: 5-10 quality backlinks

□ Publish content
  □ Blog post per week (minimum)
  □ Service page improvements
  □ Case studies/testimonials
  □ Video content (ranks well)
```

---

## Quick Reference: Common Solutions

| Problem | Diagnosis | Solution | Time to Fix |
|---------|-----------|----------|------------|
| **Not indexed at all** | Check robots.txt | Remove `Disallow: /` | 5 minutes |
| **Noindex blocking** | View page source | Remove noindex meta tag | 10 minutes |
| **Site speed slow** | PageSpeed Insights <50 | Enable caching, compress images | 1-2 hours |
| **Redirect loop** | Test with curl | Fix redirect chain | 15 minutes |
| **Sitemap 404** | Access sitemap URL | Generate with Yoast SEO | 30 minutes |
| **Canonical wrong** | Check <head> | Update to correct domain | 10 minutes |
| **Search Console errors** | Check Coverage | Fix errors per type | 30 mins - 2 hrs |
| **New site (30 days)** | Check domain age | Submit sitemap, add links | Wait 2-4 weeks |

---

## FAQ: Indexing Issues

**Q: How long does indexing take?**
- New site: 2-4 weeks
- Existing site: 1-7 days for updates
- Fastest: Content linked from homepage (1-2 days)

**Q: Why is my homepage not indexed but others are?**
- Usually: Homepage has special issue (noindex, robots.txt block, canonical to different domain)
- Fix: Check homepage specifically using steps above

**Q: Can I force Google to index my pages?**
- Limited control: Use "Request Indexing" in Search Console (max 10x/month for free tier)
- Better approach: Get quality backlinks, publish regularly, improve page speed

**Q: What if nothing works?**
- Check: https://support.google.com/webmasters/
- Post in: Google Search Central Community
- Contact: Support via Search Console (Premium Support)

---

## Conclusion

**Indexing is critical for SEO success.** Follow this diagnostic framework systematically:

1. ✅ Start with Quick Diagnosis Checklist
2. ✅ Run Crawlability Tests (verify Google can access site)
3. ✅ Audit Robots.txt & Noindex (remove blockers)
4. ✅ Validate Sitemap (ensure it's correct and submitted)
5. ✅ Check Page Speed (fix critical issues)
6. ✅ Use Search Console (request indexing, monitor progress)
7. ✅ Execute Recovery Plan (follow phases in order)
8. ✅ Monitor Weekly (track progress over time)

**Expected Timeline to Full Resolution:**
- Days 1-2: Emergency fixes
- Days 3-7: Validation & optimization
- Days 8-14: Monitoring & link building
- Week 3-4: See initial indexing growth
- Month 2-3: Full site indexed with good rankings

**Success Indicator:** All 50+ pages indexed + homepage ranking position 1-3 for target keywords.

