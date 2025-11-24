# ✅ Correct Usage Guide

## Quick Reference: What Works in ERPNext

### 📄 For Print Formats

**✅ USE THIS:**
```
sales_invoice_modern.jinja
```
- Everything in ONE file
- All CSS embedded
- Works perfectly in ERPNext
- Sync with update.py

**❌ DON'T USE:**
```
sales_invoice_modular.jinja
```
- Uses `{% include %}` statements
- ERPNext can't access external files
- Will show "Error on line X: filename.jinja"

---

### 📋 For Letter Head

**✅ USE THESE:**
```
header_modern_letterhead.jinja  (paste into Header HTML)
footer_modern_letterhead.jinja  (paste into Footer HTML)
```
- Copy content directly into ERPNext
- No includes needed
- Works in Letter Head context

**❌ DON'T USE:**
```
header_modern.jinja  (has letter_head variable - causes errors)
footer_modern.jinja  (has letter_head variable - causes errors)
```

---

## 🚀 Correct Setup Steps

### Step 1: Sync the Standalone Invoice

```bash
cd /home/benning/Desktop/template-erpnext
source env/bin/activate
python update.py --only-template sales_invoice_modern.jinja
```

### Step 2: Use in ERPNext

1. Open any **Sales Invoice**
2. Click **Print**
3. Select **"Modern Invoice"** format
4. ✅ Done!

---

## 📁 File Purpose Guide

| File | Purpose | Use In |
|------|---------|--------|
| `sales_invoice_modern.jinja` | ✅ **Production** | ERPNext Print Format |
| `sales_invoice_modular.jinja` | ℹ️ Reference only | Local dev/docs |
| `header_modern_letterhead.jinja` | ✅ **Production** | ERPNext Letter Head |
| `footer_modern_letterhead.jinja` | ✅ **Production** | ERPNext Letter Head |
| `header_modern.jinja` | ℹ️ Reference only | Documentation |
| `footer_modern.jinja` | ℹ️ Reference only | Documentation |
| `letter_head2.jinja` | ℹ️ Legacy | Old combined version |

---

## ⚠️ Common Errors & Solutions

### Error: "header_modern.jinja" on line X

**Problem**: Using modular version with includes  
**Solution**: Use `sales_invoice_modern.jinja` instead

### Error: Circular reference in Letter Head

**Problem**: Using `header_modern.jinja` in Letter Head  
**Solution**: Use `header_modern_letterhead.jinja` instead

### Error: Template not found

**Problem**: Trying to use includes in print format  
**Solution**: Use standalone version with embedded content

---

## 🎯 Best Practices

### ✅ DO:
- Use `sales_invoice_modern.jinja` for print formats
- Use `_letterhead` versions for Letter Head
- Sync via `update.py` script
- Test in ERPNext print preview

### ❌ DON'T:
- Use modular version in ERPNext
- Use `{% include %}` in print formats
- Copy modular files to ERPNext
- Mix letterhead and regular versions

---

## 📝 Summary

**For ERPNext Print Formats:**
```
✅ sales_invoice_modern.jinja (all-in-one)
```

**For ERPNext Letter Head:**
```
✅ header_modern_letterhead.jinja (paste content)
✅ footer_modern_letterhead.jinja (paste content)
```

**The modular versions are for:**
- Code organization reference
- Understanding template structure
- Future development ideas
- **NOT for direct ERPNext use**

---

## 🔧 If You Already Synced Modular Version

1. **Go to**: Print Format List in ERPNext
2. **Delete**: "Modern Invoice Modular" (if exists)
3. **Sync correct version**:
   ```bash
   python update.py --only-template sales_invoice_modern.jinja
   ```
4. **Use**: "Modern Invoice" print format

---

## ✨ Quick Commands

### Sync Modern Invoice (Correct)
```bash
cd /home/benning/Desktop/template-erpnext
source env/bin/activate
python update.py --only-template sales_invoice_modern.jinja
```

### Sync All Templates
```bash
python update.py
```

### Check Config
```bash
cat config.ini | grep "Modern Invoice"
```

Should show:
```
[Modern Invoice]
DocType = Sales Invoice
TemplateFile = print_format/sales_invoice_modern.jinja
```

---

**That's it! Use the standalone version and everything works perfectly.** ✅

