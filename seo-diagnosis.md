# SEO Technical Troubleshooting

## Scenario: New Website Not Indexing After Sitemap Submission

**Example Site:** workcity.com

---

## 1. Crawlability Tests

### Check Site Accessibility
- Test if site loads: `https://workcity.com`
- Verify SSL certificate is valid
- Check DNS points to correct server

### Google Search Console Tests
1. Go to **URL Inspection** → Enter homepage URL
2. Check status: "URL is on Google" or "Not in index"
3. Click **"Test Live URL"** → View crawled page
4. Compare rendered page to actual site

**Red Flags:**
- Page doesn't render correctly for Googlebot
- JavaScript/CSS not loading
- Timeout errors

---

## 2. Canonical Tag Audit

### Check Each Page
Right-click → View Page Source → Search for `rel="canonical"`

**Correct:**
```html
<link rel="canonical" href="https://workcity.com/about/" />
```
(Each page points to itself)

**Wrong - Blocks Indexing:**
```html
<link rel="canonical" href="https://example.com/about/" />
```
(Points to different domain)

**Fix in WordPress:**
- Install Yoast SEO or Rank Math
- Plugins auto-add self-referencing canonicals
- Verify in plugin settings

---

## 3. Robots.txt & No-Index Audit

### Check robots.txt
Visit: `https://workcity.com/robots.txt`

**Correct:**
```
User-agent: *
Allow: /

Sitemap: https://workcity.com/sitemap.xml
```

**Wrong (Blocks Everything):**
```
User-agent: *
Disallow: /
```

**Fix:** Edit robots.txt file or in WordPress:
- Settings → Reading
- Uncheck "Discourage search engines from indexing this site"

### Check for No-Index Tags
View Page Source → Search for `noindex`

**If found:**
```html
<meta name="robots" content="noindex" />
```

**Remove it** via:
- WordPress: Settings → Reading → Uncheck blocking option
- Manually: Delete the meta tag from `<head>`

---

## 4. Sitemap Structure Issues

### Verify Sitemap Exists
Go to: `https://workcity.com/sitemap.xml`

Should show XML code with URLs like:
```xml
<url>
  <loc>https://workcity.com/about/</loc>
  <lastmod>2025-01-15</lastmod>
</url>
```

### Common Problems
- URLs point to wrong domain (staging/old site)
- Using `http://` instead of `https://`
- Duplicate URLs listed
- Sitemap doesn't exist (404 error)

### Generate Sitemap (WordPress)
1. Install Yoast SEO or Rank Math
2. Enable XML Sitemaps in plugin settings
3. Submit to Search Console: Sitemaps → Add sitemap URL
4. Verify "Success" status

---

## 5. Page Speed Indexing Blockers

### Quick Check
Run: [Google PageSpeed Insights](https://pagespeed.web.dev)
- Score below 50 = Poor (may affect crawling)
- Pages timing out (30+ seconds) = Google may skip them

### Common Speed Issues
- Server returning 503 errors during crawls
- JavaScript blocking page rendering
- Hosting server insufficient resources

**Quick Fixes:**
- Install caching plugin (WP Rocket, WP Super Cache)
- Optimize images (Smush, ShortPixel)
- Upgrade hosting if consistently slow

**Note:** Speed rarely the main cause—fix robots.txt and no-index first.

---

## 6. Search Console Debugging

### Check Coverage Report
1. Search Console → **Coverage**
2. Review sections:
   - **Valid** = Indexed ✓
   - **Excluded** = Not indexed (investigate why)
   - **Error** = Problems found

### If 0 Valid Pages
Check **Excluded** section for reasons:
- "Excluded by noindex tag" → Remove no-index
- "Blocked by robots.txt" → Fix robots.txt
- "Duplicate, Google chose different canonical" → Fix canonicals

### Request Indexing
1. URL Inspection → Enter fixed page URL
2. Click **"Request Indexing"**
3. Google re-crawls within hours
4. Limit: 10-20 important pages (don't spam)

### Check Crawl Stats
Search Console → **Settings** → Check:
- Pages crawled per day
- Crawl errors
- Low crawl activity on new sites is normal

---

## Diagnostic Flowchart

```
1. Site loads in browser?
   NO → Fix hosting/DNS/SSL
   YES → Continue

2. Check robots.txt
   Blocks crawling? → Fix: Allow /
   Allows crawling → Continue

3. Check for no-index tags
   Found? → Remove no-index
   Not found → Continue

4. Check canonicals
   Wrong/missing? → Fix to self-reference
   Correct → Continue

5. Check sitemap
   Missing/broken? → Generate and submit
   Valid → Continue

6. Search Console Coverage
   Shows errors? → Fix specific errors
   Shows excluded? → Remove blocks
   Shows valid pages → Indexing working
```

---

## Quick Checklist

- [ ] Site accessible in browser
- [ ] robots.txt allows crawling
- [ ] No `noindex` meta tags
- [ ] Self-referencing canonicals on all pages
- [ ] Valid sitemap at `/sitemap.xml`
- [ ] Sitemap submitted to Search Console
- [ ] Search Console shows no crawl errors
- [ ] Coverage report shows "Valid" pages

**If all checked:** Wait 2-4 weeks for full indexing.
**If any unchecked:** That's your issue—fix it.

---

## Indexing Timeline

**New Site:**
- Week 1: Homepage appears
- Weeks 2-4: 20-50% indexed
- Month 2-3: 80%+ indexed

**After Fixing Issues:**
- Days 2-5: Re-indexing starts
- Week 2: Full coverage restored

If no indexing after 60 days → Revisit all checks above.