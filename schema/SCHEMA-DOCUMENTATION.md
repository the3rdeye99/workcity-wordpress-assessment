# JSON-LD Schema Markup Documentation

## Overview

This directory contains JSON-LD schema markup files for Clean & Shine, a professional cleaning services company based in Lagos, Nigeria. These schema files help search engines understand the business, improve search visibility, and enable rich snippets in search results.

---

## Files Included

### 1. **organization-schema.json**
**Type:** LocalBusiness Schema  
**Purpose:** Primary business information for local search optimization

**Key Information:**
- Business name: Clean & Shine
- Location: Lagos, Nigeria
- Services: Residential, Commercial, Deep Cleaning, Post-Construction, Maintenance
- Operating hours (Monday-Friday 8AM-6PM, Saturday 10AM-4PM)
- Phone number, email, and physical address
- Social media profiles (Facebook, Instagram, Twitter, LinkedIn, YouTube)
- Price range: ₦15,000 - ₦100,000

**Impact:**
- Enables Google Maps integration
- Shows business hours in search results
- Displays contact information
- Improves local SEO ranking

---

### 2. **founder-schema.json**
**Type:** Person Schema  
**Purpose:** Establish founder/key person identity for Knowledge Panel

**Key Information:**
- Name: Tobi Apata
- Title: Founder & CEO
- Portfolio: https://apata-oluwatobi-portfolio.vercel.app
- Social profiles: LinkedIn, GitHub, Twitter, Instagram
- Email and phone
- Skills and expertise
- Organization affiliation

**Impact:**
- Builds personal brand authority
- Improves Knowledge Panel chances
- Shows founder credibility
- Helps with brand entity recognition

---

### 3. **website-schema.json**
**Type:** WebSite Schema  
**Purpose:** Define website properties and search functionality

**Key Information:**
- Website URL: https://cleanandshine.com
- Search action template for site search
- Publisher information with logo
- Social media integration

**Impact:**
- Enables site search functionality in Google
- Improves website structure understanding
- Helps with branded search queries

---

### 4. **organization-full-schema.json**
**Type:** Organization Schema (Comprehensive)  
**Purpose:** Complete organization identity for Knowledge Graph

**Key Information:**
- Full organization details with founder
- Logo and cover image
- Founding date and location
- Area served
- All social profiles
- Services offered
- Price range

**Impact:**
- Builds Knowledge Graph entity
- Establishes organization authority
- Improves branded search results
- Shows in featured snippets

---

### 5. **services-schema.json**
**Type:** Service Schema with Offers  
**Purpose:** Document all service offerings and pricing

**Services Included:**
1. **Residential Cleaning** - ₦25,000
2. **Commercial Cleaning** - ₦50,000
3. **Deep Cleaning Services** - ₦40,000
4. **Post-Construction Cleaning** - ₦75,000
5. **Maintenance Cleaning** - ₦15,000

**Impact:**
- Shows services in search results
- Displays pricing information
- Enables service-specific queries
- Improves e-commerce integration

---

### 6. **reviews-schema.json**
**Type:** AggregateRating Schema  
**Purpose:** Display aggregate customer ratings

**Information:**
- Rating: 4.9/5 stars
- Review count: 156
- Best rating: 5
- Worst rating: 1

**Impact:**
- Shows star ratings in search results
- Increases click-through rate
- Builds trust and credibility
- Improves local search ranking

---

## How to Use These Schema Files

### Option 1: Add to WordPress Theme (functions.php)

```php
<?php
// Add JSON-LD schema markup to WordPress site
function add_json_ld_schema() {
    $schema_files = array(
        'organization-schema.json',
        'founder-schema.json',
        'website-schema.json',
        'services-schema.json',
        'reviews-schema.json'
    );
    
    foreach ($schema_files as $file) {
        $schema_path = get_template_directory() . '/schema/' . $file;
        if (file_exists($schema_path)) {
            echo '<script type="application/ld+json">' . 
                 file_get_contents($schema_path) . 
                 '</script>';
        }
    }
}
add_action('wp_head', 'add_json_ld_schema', 20);
?>
```

### Option 2: Add to Header Manually

Insert in `wp-content/themes/workcity/header.php` before `</head>`:

```html
<script type="application/ld+json">
{CONTENTS OF SCHEMA FILE}
</script>
```

### Option 3: Use Yoast SEO Plugin

1. Yoast SEO automatically generates some schema
2. Customize in: SEO → Search Appearance → Knowledge Graph
3. Upload/link these JSON-LD files for additional coverage

---

## Validation Tools

### Test Your Schema Markup

1. **Google Rich Results Test**
   - URL: https://search.google.com/test/rich-results
   - Copy JSON-LD content and validate

2. **Schema.org Validator**
   - URL: https://validator.schema.org/
   - Paste JSON and validate syntax

3. **Google Structured Data Test** (Legacy)
   - Helpful for debugging schema issues

4. **Yoast SEO Schema Checker**
   - Built into Yoast SEO plugin

---

## Schema Markup Benefits

### For Search Results
- ✅ Rich snippets with ratings and prices
- ✅ Business information display
- ✅ Contact details showing
- ✅ Service offerings visible

### For Knowledge Graph
- ✅ Increased Knowledge Panel visibility
- ✅ Better entity recognition
- ✅ Improved knowledge base integration
- ✅ Featured snippet eligibility

### For Local SEO
- ✅ Google Maps integration
- ✅ Local search ranking boost
- ✅ Service area optimization
- ✅ Business hours display

### For Voice Search
- ✅ Better semantic understanding
- ✅ Natural language query matching
- ✅ Featured results priority
- ✅ Skill-based answers

---

## Customization Guide

### Update Company Information

Before deployment, update these placeholders in each file:

| Placeholder | Replace With |
|---|---|
| `https://cleanandshine.com` | Your actual website URL |
| `+234-XXX-XXX-XXXX` | Actual phone number |
| `info@cleanandshine.com` | Actual email |
| `123 Business Avenue, Victoria Island` | Actual address |
| `https://cleanandshine.com/logo.png` | Logo URL |
| `https://cleanandshine.com/founder.jpg` | Founder photo URL |
| Social media URLs | Your actual profiles |

### Example Update:
```json
// Before:
"telephone": "+234-XXX-XXX-XXXX",

// After:
"telephone": "+234-703-123-4567",
```

---

## Implementation Checklist

- ✅ Update all URLs to production domain
- ✅ Add actual phone number and email
- ✅ Upload logo and images
- ✅ Link social media profiles
- ✅ Update founder information (if different)
- ✅ Validate schema with Google tool
- ✅ Test rich results display
- ✅ Monitor Google Search Console
- ✅ Track Knowledge Graph appearance
- ✅ Update reviews/ratings schema regularly

---

## Knowledge Panel Setup

To improve chances of getting a Knowledge Panel:

1. **Create Wikipedia page** (if eligible)
2. **Add verified social profiles**
   - Facebook, Instagram, Twitter, LinkedIn
   - Link to official website

3. **Build backlinks** from reputable sites
4. **Consistent schema markup** across web
5. **Complete Google My Business profile**
6. **Maintain active social presence**
7. **Generate media coverage** (press mentions)

---

## Monitoring & Updates

### Google Search Console
1. Go to: google.com/search-console
2. Add property for your website
3. Monitor "Rich results" report
4. Check "Coverage" for errors
5. Submit XML sitemap

### Update Frequency
- Update reviews/ratings monthly
- Update prices if changed
- Update social profiles as they grow
- Review schema validity quarterly
- Refresh opening hours seasonally

---

## Common Issues & Solutions

| Issue | Solution |
|---|---|
| Schema not showing in results | Validate syntax, check URLs, resubmit to GSC |
| Structured data errors | Check JSON syntax, use validator tools |
| Wrong information displaying | Update schema files, clear cache |
| Not appearing in rich results | Wait 2-4 weeks, submit to Search Console |
| Knowledge Panel not showing | Build more entity signals, get more citations |

---

## Files Structure

```
wp-content/
└── schema/
    ├── organization-schema.json
    ├── founder-schema.json
    ├── website-schema.json
    ├── organization-full-schema.json
    ├── services-schema.json
    ├── reviews-schema.json
    └── SCHEMA-DOCUMENTATION.md (this file)
```

---

## Related Files in Repository

- `README.md` - Main project documentation
- `.gitignore` - Git configuration
- `wp-content/themes/workcity/` - Custom theme files

---

## References

- [Schema.org Documentation](https://schema.org)
- [Google Structured Data Guide](https://developers.google.com/search/docs/appearance/structured-data)
- [JSON-LD Format](https://json-ld.org)
- [Local Business Schema](https://schema.org/LocalBusiness)
- [Organization Schema](https://schema.org/Organization)
- [Person Schema](https://schema.org/Person)

---

## Support

For questions about schema implementation:
1. Check Google Search Central documentation
2. Use Google Rich Results testing tool
3. Consult schema.org definitions
4. Review Yoast SEO schema guides

---

**Last Updated:** November 25, 2025  
**Version:** 1.0.0  
**Maintained By:** Clean & Shine Development Team
