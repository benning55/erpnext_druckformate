# Letter Head Error Fix

## The Problem

When using `header_modern.jinja` and `footer_modern.jinja` in **ERPNext Letter Head**, you'll see errors because these files contain a `letter_head` variable check that creates a circular reference.

**Errors appear on lines checking**: `{% if letter_head and letter_head.startswith('<img') %}`

---

## The Solution

Use the Letter Head-specific versions instead:

### For Letter Head Configuration

✅ Use: **`header_modern_letterhead.jinja`**  
✅ Use: **`footer_modern_letterhead.jinja`**

These versions:
- Remove the `letter_head` variable reference
- Have simplified margins (no negative values)
- Work perfectly in Letter Head context

### For Print Format Templates

✅ Use: **`header_modern.jinja`**  
✅ Use: **`footer_modern.jinja`**

These versions:
- Support dynamic letter head loading
- Have print format-specific margins
- Used with `{% include %}` in templates

---

## File Usage Guide

| File | Use In | Purpose |
|------|--------|---------|
| `header_modern_letterhead.jinja` | **Letter Head** | Header for Letter Head config |
| `footer_modern_letterhead.jinja` | **Letter Head** | Footer for Letter Head config |
| `header_modern.jinja` | **Print Format** | Include in templates |
| `footer_modern.jinja` | **Print Format** | Include in templates |

---

## How to Fix Your Letter Head

### Step 1: Open Letter Head in ERPNext

1. Go to: **Setup → Printing → Letter Head**
2. Open your Letter Head (e.g., "Header2")

### Step 2: Update Header HTML

**Delete current content** and use `header_modern_letterhead.jinja`:

```jinja
{% set company = frappe.get_doc("Company", doc.company) %}
{% set company_address_name = frappe.db.get_value("Dynamic Link", filters={"link_doctype": "Company", "link_name": doc.company, "parenttype": "Address"}, fieldname="parent") %}
{% set cmp_addr = frappe.get_doc("Address", company_address_name) if company_address_name else None %}

<div class="modern-header">
    <div class="header-content">
        <div class="header-left">
            <h1 class="invoice-title">INVOICE</h1>
            <p class="company-tagline">{{ company.company_name }}</p>
        </div>
        <div class="header-right">
            <div class="company-logo">
                <div class="logo-placeholder">
                    <svg width="60" height="60" viewBox="0 0 60 60" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <rect width="60" height="60" rx="8" fill="white" opacity="0.3"/>
                        <path d="M30 15L45 25V40L30 50L15 40V25L30 15Z" fill="white"/>
                    </svg>
                </div>
            </div>
            <div class="company-info-header">
                <p><strong>{{ company.company_name }}</strong></p>
                {% if cmp_addr %}
                <p>{{ cmp_addr.address_line1 }}</p>
                <p>{{ cmp_addr.city }}, {% if cmp_addr.state %}{{ cmp_addr.state }},{% endif %} {{ cmp_addr.pincode }}</p>
                {% endif %}
                <p>{{ _("Tax ID") }}: {{ company.tax_id or "N/A" }}</p>
                <p>{{ _("Phone") }}: {{ company.phone_no or "N/A" }}</p>
                <p>{{ _("Email") }}: {{ company.email or "N/A" }}</p>
            </div>
        </div>
    </div>
</div>

<style>
.modern-header {
    background: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%);
    color: white;
    padding: 40px 50px;
    margin: 0;
}
.header-content { display: flex; justify-content: space-between; align-items: flex-start; }
.header-left { flex: 1; }
.invoice-title { font-size: 48px; font-weight: 700; margin: 0 0 5px 0; letter-spacing: 2px; }
.company-tagline { font-size: 16px; margin: 0; opacity: 0.9; }
.header-right { display: flex; align-items: flex-start; gap: 20px; text-align: right; }
.company-logo { width: 80px; height: 80px; background: white; border-radius: 12px; display: flex; align-items: center; justify-content: center; padding: 10px; }
.company-logo img { max-width: 100%; max-height: 100%; object-fit: contain; }
.logo-placeholder { display: flex; align-items: center; justify-content: center; }
.company-info-header { font-size: 11px; line-height: 1.6; }
.company-info-header p { margin: 3px 0; }
@media print {
    .modern-header { page-break-inside: avoid; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
}
</style>
```

### Step 3: Update Footer HTML

Use `footer_modern_letterhead.jinja` content in **Footer HTML** field.

### Step 4: Save

Click **Save** - errors should be gone! ✅

---

## Quick Copy-Paste Instructions

### For ERPNext Letter Head:

1. **Copy content from**: `header_modern_letterhead.jinja`
2. **Paste into**: Letter Head → Header HTML
3. **Copy content from**: `footer_modern_letterhead.jinja`
4. **Paste into**: Letter Head → Footer HTML
5. **Save**

---

## Key Differences

### header_modern_letterhead.jinja vs header_modern.jinja

| Aspect | letterhead version | regular version |
|--------|-------------------|-----------------|
| `letter_head` check | ❌ Removed | ✅ Included |
| Margins | `margin: 0` | `margin: -10mm -25mm...` |
| Use case | Letter Head config | Print Format includes |
| Conditional logo | No | Yes |

### Why Two Versions?

**Letter Head Context**:
- Used globally across all prints
- `letter_head` variable would be circular
- Needs neutral margins

**Print Format Context**:
- Used in specific templates via include
- Can access `letter_head` variable
- Needs specific print margins

---

## Troubleshooting

### Still seeing errors?

1. **Clear cache**: Settings → Clear Cache
2. **Reload page**: Hard refresh (Ctrl+Shift+R)
3. **Check file**: Ensure you copied the `_letterhead` version
4. **Verify syntax**: Check all `{% %}` tags are closed

### Logo not showing?

Replace the SVG section with your image:
```html
<img src="/files/your-logo.png" alt="Logo" style="max-width: 100%; max-height: 100%;">
```

### Colors not appearing?

Ensure CSS is included within the same HTML field.

---

## Summary

**Problem**: Original header/footer files have `letter_head` variable causing errors  
**Solution**: Use `_letterhead` versions for Letter Head configuration  
**Result**: Clean, error-free Letter Head ✅

---

**Files Created**:
- ✅ `header_modern_letterhead.jinja` (for Letter Head)
- ✅ `footer_modern_letterhead.jinja` (for Letter Head)
- ℹ️ `header_modern.jinja` (for Print Formats - keep as is)
- ℹ️ `footer_modern.jinja` (for Print Formats - keep as is)

