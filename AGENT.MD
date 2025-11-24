# AGENT.MD - ERPNext DIN 5008 Print Format Templates

## Project Overview

This project provides **professional, DIN 5008 compliant print format templates** for ERPNext (ERP system based on Frappe Framework). DIN 5008 is a German standard for business correspondence layout, ensuring professional document formatting for German business communications.

**Target ERPNext Version**: version-13 (adaptable to other versions with minor modifications)  
**License**: GNU General Public License v3.0  
**Primary Languages**: German and English (bilingual support)

---

## Core Purpose

The project enables ERPNext users to:
1. Generate professionally formatted business documents (invoices, quotations, sales orders, etc.)
2. Comply with German DIN 5008 business correspondence standards
3. Automatically sync print format changes from local development to ERPNext instances
4. Maintain consistent styling across all document types
5. Support multilingual document generation (German/English)

---

## Project Structure

```
template-erpnext/
├── config.ini                  # Configuration mapping print formats to doctypes
├── frappeclient.py            # Frappe REST API client wrapper
├── update.py                  # Automated sync script for print formats
├── requirements.txt           # Python dependencies
├── letter_head.jinja          # Footer template for letterhead
├── print_format/              # Jinja templates for each document type
│   ├── sales_invoice.jinja
│   ├── quotation.jinja
│   ├── sales_order.jinja
│   ├── delivery_note.jinja
│   ├── purchase_order.jinja
│   ├── request_for_quotation.jinja
│   ├── dunning.jinja
│   └── dunning_v14.jinja
├── print_style/               # SCSS/CSS styling
│   ├── print_style.scss       # Source SCSS file
│   ├── print_style.css        # Compiled CSS
│   └── print_style.css.map    # CSS source map
├── translations/              # Translation files
│   └── de.csv                 # German translations
└── docs/                      # Documentation images
    ├── letter_head.png
    └── print_format.png
```

---

## Key Components

### 1. **Print Format Templates** (`print_format/*.jinja`)

Jinja2 HTML templates for various ERPNext document types:
- **Sales Invoice** (`Ausgangsrechnung`) - invoices with tax calculations, timesheets
- **Quotation** (`Angebot`) - customer quotations
- **Sales Order** (`Auftragsbestätigung`) - order confirmations
- **Delivery Note** (`Lieferschein`) - shipping documents
- **Purchase Order** (`Bestellung`) - purchase orders
- **Request for Quotation** (`Angebotsanfrage`) - RFQs
- **Dunning** (`Mahnung`) - payment reminders

**Features**:
- DIN 5008 compliant layout (margins, fold marks, punch hole marks)
- Dynamic data binding via Jinja2 (`{{ doc.field_name }}`)
- Multilingual support via `_("Text")` translation function
- Conditional rendering (return invoices, amended invoices)
- Support for timesheets, taxes, discounts, payment terms
- Professional address formatting with sender/recipient sections

### 2. **Print Style** (`print_style/print_style.scss`)

SCSS stylesheet that defines:
- **DIN 5008 layout specifications**:
  - Fold marks at 87mm and 192mm
  - Punch hole mark at 105mm
  - Address window positioning (45mm height, 85mm width)
  - Margin specifications
- **Typography**: 10pt base font, 7pt footer
- **Component styling**: tables, headers, footers, contact blocks
- **Print-specific CSS**: page breaks, footer positioning

**Compilation**: Requires Sass to compile SCSS to CSS
```bash
sass --style=compressed print_style/print_style.scss print_style/print_style.css
```

### 3. **Letter Head Template** (`letter_head.jinja`)

Footer template containing:
- Company information table (3 columns: contact info, legal info, banking info)
- Dynamic field population from ERPNext Company doctype
- Customizable placeholders for:
  - Managing directors
  - Tax ID / Trade register number
  - Bank details (IBAN, BIC)
  - Court registry information
- Multilingual field labels

### 4. **Update Script** (`update.py`)

Python script for automated synchronization of print formats to ERPNext:

**Features**:
- Authenticates with ERPNext instance via username/password
- Reads configuration from `config.ini`
- Compiles SCSS to CSS automatically
- Syncs print formats (creates new or updates existing)
- Supports selective template updates (`--only-template` flag)
- Environment variable support via `.env` file

**Usage**:
```bash
python update.py                          # Sync all formats
python update.py --only-template sales_invoice.jinja  # Sync specific template
```

**Configuration** (`.env` file):
```
BASE_URL=https://your-erpnext.com
USER=your_username
PASSWORD=your_password
```

### 5. **Frappe Client** (`frappeclient.py`)

Python wrapper for Frappe REST API:
- Session-based authentication
- CRUD operations for doctypes
- Print format management (get_doc, update, insert)
- PDF/HTML generation from print formats
- Error handling with FrappeException

**Key Methods**:
- `get_doc(doctype, name)` - Retrieve document
- `update(doc)` - Update existing document
- `insert(doc)` - Create new document
- `get_list(doctype, filters)` - Query documents

### 6. **Configuration** (`config.ini`)

INI-format configuration mapping print format names to:
- `DocType` - ERPNext document type
- `TemplateFile` - Path to Jinja template
- `IsStandard` - Whether format is standard (0/1)

Example:
```ini
[Ausgangsrechnung]
DocType = Sales Invoice
TemplateFile = print_format/sales_invoice.jinja
```

### 7. **Translations** (`translations/de.csv`)

CSV file mapping English terms to German translations:
- Field labels (Phone → Telefon, Date → Datum)
- Document types (Invoice → Rechnung)
- Table headers (Description → Beschreibung)
- System messages

---

## Technical Details

### Dependencies (`requirements.txt`)
- **requests** - HTTP library for API calls
- **pylint** - Code linting
- **typer** - CLI argument parsing
- **python-dotenv** - Environment variable management

### Python Version
Developed for Python 3.10+ (uses modern type hints like `str | None`)

### Jinja2 Template Variables

Available in print format templates:
- `doc` - Main document object (e.g., Sales Invoice)
- `letter_head` - Company letterhead HTML
- `footer` - Footer content from Letter Head
- `print_settings` - Print configuration
- `frappe` - Frappe utilities (get_doc, utils.formatdate, db, lang)
- `_()` - Translation function

**Common Patterns**:
```jinja
{{ doc.name }}                              # Document ID
{{ doc.get_formatted('grand_total') }}     # Formatted currency
{{ frappe.utils.formatdate(doc.date, "long") }}  # Formatted date
{% set company = frappe.get_doc("Company", doc.company) %}  # Related doc
{{ _("Invoice") }}                         # Translated text
```

### DIN 5008 Specifications

**Fold Marks** (Faltmarken):
- Upper fold mark: 87mm from top
- Lower fold mark: 192mm from top

**Punch Hole Mark** (Lochmarke):
- 105mm from top (for binder holes)

**Address Window**:
- Position: 20mm from left, starts after 45mm header
- Size: 85mm width, 45mm height
- Sender line: Small font (50%), 17.7mm height
- Recipient address: Below sender line

**Margins**:
- Left: 25mm
- Right: 20mm
- Top: 10mm
- Bottom: 45mm (for footer)

---

## Workflow & Usage

### Initial Setup

1. **Install dependencies**:
   ```bash
   python3.10 -m venv env
   source env/bin/activate  # Linux/Mac
   pip install -r requirements.txt
   ```

2. **Install Sass** (for SCSS compilation):
   ```bash
   npm install -g sass
   # or
   brew install sass/sass/sass
   ```

3. **Configure environment**:
   Create `.env` file with ERPNext credentials:
   ```
   BASE_URL=https://your-erpnext-instance.com
   USER=administrator
   PASSWORD=your_password
   ```

4. **Configure print formats**:
   Edit `config.ini` to match your ERPNext setup

### Development Workflow

1. **Edit templates**: Modify `.jinja` files in `print_format/`
2. **Edit styles**: Modify `print_style/print_style.scss`
3. **Sync to ERPNext**:
   ```bash
   python update.py
   ```
4. **Preview**: Check print preview in ERPNext

### ERPNext Configuration

After syncing templates:

1. **Create Letter Head**:
   - Navigate to: Setup > Printing > Letter Head
   - Copy content from `letter_head.jinja` to "Footer HTML"
   - Add header image/HTML
   - Customize company-specific information (bank details, directors, etc.)

2. **Set Company Defaults**:
   - Open Company doctype
   - Set "Default Letter Head"
   - Add company address (appears as sender)

3. **Configure Address Templates**:
   Create templates for different countries (Germany vs. others)

4. **Set Default Print Formats**:
   - Use "Customize Form" for each DocType
   - Set "Default Print Format" to the synced format name

### VSCode Integration

For automatic sync on file changes:
1. Install "File Watcher" extension
2. Configure to run `python update.py --only-template {changed_file}` on save
3. SCSS changes trigger full sync (all templates use same CSS)

---

## Customization Guide

### Adding New Print Format

1. Create new Jinja template in `print_format/` (e.g., `my_format.jinja`)
2. Add configuration to `config.ini`:
   ```ini
   [My Format Name]
   DocType = My DocType
   TemplateFile = print_format/my_format.jinja
   ```
3. Run `python update.py` to sync

### Modifying Styles

1. Edit `print_style/print_style.scss`
2. **Important**: Scope all styles within `.print-format` to avoid affecting ERPNext UI
3. Run sync script (automatically compiles SCSS)

### Adding Translations

1. Add entries to `translations/de.csv`:
   ```csv
   English Term,Deutsche Übersetzung,
   ```
2. Use in templates: `{{ _("English Term") }}`

### Multilingual Content

For longer text blocks:
```jinja
{% if frappe.lang == "de" %}
    <p>Deutscher Text</p>
{% else %}
    <p>English text</p>
{% endif %}
```

---

## Common Patterns & Examples

### Accessing Related Documents
```jinja
{% set company = frappe.get_doc("Company", doc.company) %}
{{ company.website }}
```

### Conditional Rendering
```jinja
{% if doc.is_return %}
    <p>{{ _("Correction Invoice") }}</p>
{% elif doc.amended_from %}
    <p>{{ _("Cancellation Invoice") }}</p>
{% else %}
    <p>{{ _("Invoice") }}</p>
{% endif %}
```

### Looping Through Items
```jinja
{% for item in doc.items %}
    <tr>
        <td>{{ loop.index }}</td>
        <td>{{ item.item_name }}</td>
        <td>{{ item.get_formatted("amount") }}</td>
    </tr>
{% endfor %}
```

### Formatting Values
```jinja
{{ doc.get_formatted('grand_total') }}           # Currency
{{ frappe.utils.formatdate(doc.date, "long") }}  # Long date
{{ frappe.utils.formatdate(doc.date, "dd.mm.yy") }}  # Short date
```

### Tax Calculations
```jinja
{% for tax_row in doc.taxes %}
    {% if tax_row.tax_amount or print_settings.print_taxes_with_zero_amount %}
        <tr>
            <td>{{ tax_row.description }}</td>
            <td>{{ tax_row.get_formatted("tax_amount") }}</td>
        </tr>
    {% endif %}
{% endfor %}
```

---

## Key Features & Capabilities

### Document Features
- ✅ Multi-page support with page numbering
- ✅ Letterhead header and footer
- ✅ DIN 5008 compliant layout
- ✅ Sender/recipient address formatting
- ✅ Contact person details
- ✅ Tax calculations (including reverse charge)
- ✅ Discount handling
- ✅ Payment terms and due dates
- ✅ Timesheet integration (for service invoices)
- ✅ Advance payment tracking
- ✅ Terms and conditions section
- ✅ Incoterms support

### Technical Features
- ✅ Automated deployment via Python script
- ✅ SCSS compilation with Sass
- ✅ Environment-based configuration
- ✅ Selective template updates
- ✅ REST API integration
- ✅ Error handling and validation
- ✅ VSCode integration support

---

## Troubleshooting

### Windows Sass Compilation Issues
If Sass compilation fails on Windows, modify `update.py`:
```python
def get_css(input_path: Path) -> str:
    return run(
        ["sass", "--style=compressed", input_path], 
        check=True, 
        capture_output=True,
        shell=True  # Add this line
    ).stdout.decode()
```

### Authentication Errors
- Verify `.env` credentials
- Check ERPNext user has API access permissions
- Ensure ERPNext instance is accessible from network

### Template Sync Issues
- Verify `config.ini` format
- Check DocType names match exactly
- Ensure template files exist at specified paths

### Style Not Applying
- Ensure SCSS compiles without errors
- Check all styles are scoped within `.print-format`
- Verify CSS is included in print format configuration

---

## API Reference

### FrappeClient Main Methods

```python
client = FrappeClient(url, username, password)

# Document operations
doc = client.get_doc("DocType", "name")
client.update(doc_dict)
client.insert(doc_dict)
client.delete("DocType", "name")

# Queries
results = client.get_list("DocType", fields=["name", "field"], filters={"status": "Open"})

# Print operations
pdf = client.get_pdf("Sales Invoice", "INV-001", print_format="Custom")
html = client.get_html("Sales Invoice", "INV-001")
```

### update.py CLI

```bash
python update.py                          # Sync all formats
python update.py --base-url URL           # Override URL
python update.py --username USER          # Override username
python update.py --password PASS          # Override password
python update.py --config-file path.ini   # Use alternate config
python update.py --only-template file.jinja  # Sync single template
```

---

## Resources & References

### Frappe Framework Documentation
- [Frappe Jinja API](https://frappeframework.com/docs/v13/user/en/api/jinja)
- [safe_exec.py](https://github.com/frappe/frappe/blob/version-13/frappe/utils/safe_exec.py) - Available functions
- [printview.py](https://github.com/frappe/frappe/blob/4c6b58da2699189e6992707254f7a95f5c7df64a/frappe/www/printview.py#L148-L157) - Print context
- [standard.css](https://github.com/frappe/frappe/blob/8b7c976f680b2aac1b33cf45720beaf653ccdad0/frappe/templates/styles/standard.css) - Default styles
- [standard.html](https://github.com/frappe/frappe/blob/8b7c976f680b2aac1b33cf45720beaf653ccdad0/frappe/templates/print_formats/standard.html) - Default template

### External Standards
- **DIN 5008**: German standard for business correspondence
- **Jinja2**: Template engine documentation
- **Sass**: CSS preprocessor documentation

---

## Development Notes for AI Agents

### When Working with This Project

1. **Always scope CSS**: Any CSS modifications must be within `.print-format` class
2. **Test multilingual**: Verify both German and English outputs
3. **Preserve DIN 5008 specs**: Don't modify fold marks, margins without good reason
4. **Use get_formatted()**: For currency and dates to respect locale
5. **Handle edge cases**: Return invoices, amended invoices, reverse charge
6. **Consider print preview**: Changes may look different in PDF vs. browser
7. **Backup before sync**: `update.py` immediately pushes to ERPNext
8. **Version compatibility**: Code targets ERPNext v13, other versions may differ

### Code Style
- Python 3.10+ type hints used (`str | None`)
- Jinja2 templates use 4-space indentation
- SCSS uses nested structure
- German variable names in config.ini (document names)

### Testing Strategy
1. Modify template locally
2. Run `python update.py --only-template filename.jinja`
3. Open ERPNext print preview
4. Test with various document states (draft, submitted, amended)
5. Verify German and English translations
6. Check PDF output matches browser preview

---

## File Format Details

### config.ini Format
```ini
[Print Format Display Name]
DocType = ERPNext DocType Name
TemplateFile = relative/path/to/template.jinja
IsStandard = 0  # or 1
```

### Translation CSV Format
```csv
English,Deutsch,
Term,Übersetzung,
```

### Environment File Format
```
BASE_URL=https://erpnext.example.com
USER=username
PASSWORD=password
```

---

## Project Metadata

- **Project Type**: ERPNext Print Format Templates
- **Domain**: ERP, Business Document Generation
- **Region**: Germany (DIN 5008 compliant)
- **Framework**: Frappe Framework, ERPNext
- **Languages**: Python, Jinja2, SCSS, HTML
- **Standards**: DIN 5008 German Business Correspondence
- **License**: GPL-3.0

---

## Summary for AI Agents

This is a **professional ERPNext print format template system** for generating DIN 5008 compliant business documents. Key capabilities:

1. **Template Management**: Jinja2 templates for 7+ document types
2. **Automated Deployment**: Python script syncs templates to ERPNext via REST API
3. **Professional Styling**: SCSS-based styling following German DIN 5008 standards
4. **Multilingual**: German/English support with translation system
5. **ERPNext Integration**: Full integration with Frappe/ERPNext data model

When assisting with this project:
- Maintain DIN 5008 compliance
- Test both languages
- Scope all CSS properly
- Use Frappe API conventions
- Follow existing code patterns
- Preserve document layout integrity

The project is production-ready and actively used for German business document generation in ERPNext environments.

