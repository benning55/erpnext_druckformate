# ERPNext Modern Purchase Order Template - Session Context

**Date Created**: November 23, 2025  
**Project**: TMONIT Purchase Order Print Format for ERPNext  
**Goal**: Create a modern, DIN 5008-compliant purchase order template with header/footer on every page in PDF output

---

## PROJECT OVERVIEW

### What We Built
A modern Purchase Order print format template for ERPNext (version 13) that mimics the professional design of a D-Sensors invoice:

- **Design Reference**: D-Sensors invoice (Rechnung RE2505231.pdf)
- **Company**: TMONIT
- **Color Scheme**: Professional blue (#003d7a)
- **Standard**: DIN 5008 compliant (German business correspondence)

### File Structure
```
template-erpnext/
├── print_format/
│   └── purchase_order_modern.jinja  ← Main template file (556 lines)
├── config.ini                       ← Configuration mapping
├── update.py                        ← Sync script to ERPNext
├── requirements.txt                 ← Python dependencies
└── SESSION.md                       ← This file
```

---

## DESIGN SPECIFICATIONS

### Layout Structure

1. **Blue Header Bar** (Background: #003d7a)
   - Left: TMONIT logo (from `/files/TMONITLogo.png`)
   - Right: Company address
   - Padding: 15mm 20mm

2. **Gray Info Bar** (Background: #f5f5f5)
   - Company name and short address
   - Padding: 8mm 20mm

3. **Content Area** (White background)
   - Left column: Order address & shipping address
   - Right column: Purchase Order title & document details
   - Yellow delivery note box with orange left border
   - Items table with blue header (#003d7a)
   - Totals section (right-aligned)

4. **Blue Footer Bar** (Background: #003d7a)
   - Three columns:
     - Column 1: Contact Information
     - Column 2: Legal Information
     - Column 3: Bank Details
   - Page numbering at top

### Design Features
- **Typography**: Arial, 10pt base, professional spacing
- **Colors**: 
  - Primary blue: `#003d7a`
  - Gray bar: `#f5f5f5`
  - Yellow note: `#fff3cd`
  - Border: `#ffc107`
- **DIN 5008 Compliant**: Proper margins, fold marks (currently not visible)

---

## CURRENT STATUS

### ✅ What Works (in HTML preview)
- Beautiful modern design matching D-Sensors reference
- Professional blue color scheme
- Clean layout with proper spacing
- All sections render correctly
- Responsive table design
- Proper typography

### ❌ Current Problem
**PDF generation breaks the layout:**

1. **Header/Footer positioning**: When using `position: fixed` for print media:
   - Footer appears in the middle of the page
   - Content overlaps with header/footer
   - Layout completely breaks in PDF

2. **Current workaround**: Removed fixed positioning
   - Layout now works correctly in PDF
   - BUT: Header and footer only appear on first page
   - Multi-page documents don't have header/footer on subsequent pages

### 🎯 Main Goal (UNSOLVED)
**Make PDF output exactly match HTML preview with header and footer on EVERY page**

---

## TECHNICAL DETAILS

### Template Type
- **Format**: Jinja2 HTML template
- **Framework**: Frappe/ERPNext version 13
- **PDF Engine**: wkhtmltopdf (ERPNext's default PDF generator)

### Key Jinja Variables Available
```jinja
{{ doc }}                    - Purchase Order document
{{ company }}                - Company object
{{ frappe }}                 - Frappe utilities
{{ _("Text") }}              - Translation function
{{ doc.get_formatted() }}    - Formatted values
```

### Current Hardcoded Values (Mock Data)
Since ERPNext database fields were causing errors, all company info is hardcoded:

**Header & Info Bar:**
- Address: Hauptstraße 123, 10115 Berlin, Deutschland

**Footer - Contact Info:**
- Address: Hauptstraße 123 · 10115 Berlin
- Phone: +49 30 12345678
- Email: info@tmonit.com
- Website: www.tmonit.com

**Footer - Legal Info:**
- Tax ID: DE123456789
- Registration No.: HRB 12345
- Court: Amtsgericht Berlin

**Footer - Bank Details:**
- Bank: Deutsche Bank
- IBAN: DE89 1234 5678 9012 3456 78
- BIC: DEUTDEFF

**Note**: These need to be updated with real TMONIT information.

---

## ATTEMPTED SOLUTIONS

### Attempt 1: CSS Fixed Positioning ❌
```css
@media print {
    .document-header {
        position: fixed;
        top: 0;
    }
    .document-footer {
        position: fixed;
        bottom: 0;
    }
    .document-content {
        margin-top: 50mm;
        padding-bottom: 65mm;
    }
}
@page {
    margin: 50mm 0 60mm 0;
}
```
**Result**: Complete layout breakage in PDF, footer in middle of page

### Attempt 2: Natural Document Flow ✅ (Current)
```css
@media print {
    /* No fixed positioning */
    /* Natural flow: header → content → footer */
}
@page {
    margin: 0;
}
```
**Result**: Layout works perfectly BUT header/footer only on page 1

---

## KNOWN ISSUES & CONSTRAINTS

### wkhtmltopdf Limitations
1. **Limited CSS support**: 
   - `position: fixed` doesn't work reliably
   - `@page` margin rules have limited support
   - Page counters (`<span class="page">`) don't always work

2. **Header/Footer Challenge**:
   - ERPNext uses wkhtmltopdf for PDF generation
   - wkhtmltopdf has special options for repeating headers/footers
   - These are typically set at PDF generation level, not in CSS

3. **Print-color-adjust**:
   - Need `-webkit-print-color-adjust: exact` for colors
   - Need `print-color-adjust: exact` for standards compliance

### ERPNext Specific
- Custom print formats are stored in database as HTML/CSS
- Synced via `update.py` script
- Logo must be uploaded to ERPNext Files: `/files/TMONITLogo.png`
- Print settings affect rendering (e.g., `print_settings.print_uom_after_quantity`)

---

## POSSIBLE SOLUTIONS TO EXPLORE

### Option 1: wkhtmltopdf Header/Footer Arguments
ERPNext might support passing wkhtmltopdf arguments for repeating headers/footers:
- `--header-html` flag
- `--footer-html` flag
- `--margin-top`, `--margin-bottom`

**Investigation needed**: How to pass these through ERPNext's print system

### Option 2: ERPNext Letter Head System
Use ERPNext's built-in letter head functionality:
- Header in "Letter Head" doctype (header HTML field)
- Footer in "Letter Head" doctype (footer HTML field)
- Print format contains only the content section

**Trade-off**: May require restructuring entire template

### Option 3: CSS Paged Media Module
Try advanced CSS for print:
```css
@page {
    @top-center {
        content: element(header);
    }
    @bottom-center {
        content: element(footer);
    }
}
.document-header {
    position: running(header);
}
.document-footer {
    position: running(footer);
}
```
**Risk**: May not be supported by wkhtmltopdf

### Option 4: JavaScript-Based PDF Generation
Replace wkhtmltopdf with a different PDF engine:
- Puppeteer (Chrome headless)
- WeasyPrint
- Prince XML

**Trade-off**: Requires ERPNext customization/app development

### Option 5: Table-Based Layout with thead/tfoot
Wrap entire document in a table, use `<thead>` and `<tfoot>`:
```html
<table>
    <thead><tr><th>Header content</th></tr></thead>
    <tfoot><tr><td>Footer content</td></tr></tfoot>
    <tbody><tr><td>Main content</td></tr></tbody>
</table>
```
**Note**: Tables can repeat headers/footers across pages in some PDF engines

---

## HOW TO DEPLOY CHANGES

### 1. Edit Template
```bash
cd /home/benning/Desktop/template-erpnext
nano print_format/purchase_order_modern.jinja
```

### 2. Sync to ERPNext
```bash
# Activate virtual environment
source env/bin/activate  # or: ./env/bin/python

# Sync specific template
./env/bin/python update.py --only-template print_format/purchase_order_modern.jinja

# Or sync all templates
./env/bin/python update.py
```

### 3. Test in ERPNext
1. Open any Purchase Order
2. Click "Print"
3. Select "Modern Purchase Order" format
4. View HTML preview (works)
5. Click "Download PDF" (header/footer issue)

---

## CONFIGURATION

### config.ini Entry
```ini
[Modern Purchase Order]
DocType = Purchase Order
TemplateFile = print_format/purchase_order_modern.jinja
```

### Requirements (requirements.txt)
```
requests
pylint
typer
python-dotenv
```

### Environment (.env - not in repo)
```
BASE_URL=https://your-erpnext.com
USER=your_username
PASSWORD=your_password
```

---

## KEY CODE SECTIONS

### Header Structure (Lines ~395-407)
```html
<div class="document-header">
    <div class="header-left">
        <img src="/files/TMONITLogo.png" alt="TMONIT Logo" class="header-logo">
    </div>
    <div class="header-right">
        <strong>TMONIT</strong>
        Hauptstraße 123<br>
        10115 Berlin<br>
        Deutschland
    </div>
</div>
```

### Footer Structure (Lines ~520-556)
```html
<div class="document-footer">
    <div class="page-number">- Seite 1 von 1 -</div>
    <div class="footer-grid">
        <!-- 3 columns: Contact, Legal, Bank -->
    </div>
</div>
```

### Print Media Query (Lines ~305-350)
```css
@media print {
    /* Color preservation */
    .document-header {
        background-color: #003d7a !important;
        -webkit-print-color-adjust: exact;
        print-color-adjust: exact;
    }
    /* ... more rules ... */
}
```

---

## NEXT STEPS FOR AI ASSISTANT

### Immediate Tasks
1. **Research wkhtmltopdf integration** with ERPNext:
   - How does ERPNext call wkhtmltopdf?
   - Can we pass custom arguments?
   - Location of PDF generation code in ERPNext

2. **Test table-based layout** approach:
   - Create test version with `<table>` structure
   - Use `<thead>` for header, `<tfoot>` for footer
   - Test if it repeats across pages

3. **Investigate Letter Head system**:
   - Examine ERPNext Letter Head doctype
   - Test header/footer repetition
   - See if it works with our design

### Questions to Answer
- Where in ERPNext codebase is PDF generation handled?
- Can we override wkhtmltopdf arguments via Print Format settings?
- Does ERPNext support custom PDF rendering engines?
- Are there any ERPNext apps/extensions that solve this?

### Testing Checklist
- [ ] Header appears on all pages in PDF
- [ ] Footer appears on all pages in PDF
- [ ] Content doesn't overlap with header/footer
- [ ] Colors are preserved in PDF
- [ ] Page numbering works correctly
- [ ] Multi-page documents render properly
- [ ] Layout matches HTML preview exactly

---

## REFERENCE MATERIALS

### D-Sensors Invoice Design (Reference PDF)
Key elements we matched:
- Blue header with logo and company info
- Gray info bar below header
- Two-column content layout (address left, details right)
- Yellow delivery note box
- Professional table design
- Three-column footer

### ERPNext Documentation
- Print Format: https://docs.erpnext.com/docs/v13/user/manual/en/setting-up/print/print-format
- Jinja: https://frappeframework.com/docs/v13/user/en/api/jinja
- Letter Head: https://docs.erpnext.com/docs/v13/user/manual/en/setting-up/print/letter-head

### Related Files in Project
- `AGENT.MD` - Original project documentation
- `Rechnung RE2505231.pdf` - D-Sensors reference invoice
- `sales_invoice_modern.jinja` - Similar modern invoice template
- `header_modern.jinja` - Modern header template
- `footer_modern.jinja` - Modern footer template

---

## CONTACT & CONTEXT

**Company**: TMONIT  
**User**: benning  
**System**: Linux 6.8.0-87-generic  
**ERPNext Version**: version-13  
**Python**: 3.10  

**Critical Success Criteria**: 
PDF output must be identical to HTML preview with header and footer on every page. This is non-negotiable for professional business documents.

---

## SUMMARY FOR AI ASSISTANT

**TL;DR**: We built a beautiful modern Purchase Order template for ERPNext that looks perfect in HTML preview but breaks in PDF. The main issue is getting the blue header and footer to repeat on every page in PDF output while maintaining the exact layout we see in HTML. Fixed positioning doesn't work with wkhtmltopdf. Need to find a reliable solution that makes PDF = HTML exactly.

**Current Status**: Template is 95% complete and looks perfect in browser, but PDF generation doesn't repeat header/footer on multiple pages.

**Your Mission**: Find a way to make wkhtmltopdf (or an alternative) generate PDFs with repeating headers/footers that match the HTML preview exactly.

Good luck! 🚀

