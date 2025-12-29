# Cortex Linux i18n Quick Reference

**Fast guide to implementing and using i18n in Cortex Linux**

---

## For Users: Switching Languages

### Method 1: CLI Argument (Highest Priority)
```bash
cortex --language es install nginx
cortex -L ja status
cortex -L ar config language
```

### Method 2: Environment Variable
```bash
export CORTEX_LANGUAGE=hi
cortex install python3
```

### Method 3: Config File
```bash
# Edit ~/.cortex/preferences.yaml
language: es

# Then just use cortex normally
cortex install nginx  # Will use Spanish
```

### Method 4: System Locale
Cortex auto-detects from your system language settings.

### Priority Order
```
CLI arg (-L es) > CORTEX_LANGUAGE env > Config file > System locale > English
```

---

## For Developers: Using Translations in Code

### Basic Setup
```python
from cortex.i18n import get_translator, Translator

# Method 1: Simple one-liner
translator = get_translator('es')

# Method 2: Create new instance
translator = Translator('ja')
```

### Getting Messages
```python
# Simple message
msg = translator.get('common.yes')  # Returns: 'Sí' (for Spanish)

# With variables
msg = translator.get(
    'install.success',
    package='nginx'
)  # Returns: 'nginx instalado exitosamente'

# Pluralization
msg = translator.get_plural(
    'install.downloading',
    count=5,
    package_count=5
)  # Returns: 'Descargando 5 paquetes'
```

### Language-Specific Features
```python
# Check if RTL
if translator.is_rtl():
    # Handle Arabic, Hebrew, etc.
    pass

# Switch language
translator.set_language('ar')
```

### Available Methods
```python
translator.get(key, **kwargs)              # Get translated message
translator.get_plural(key, count, **kwargs) # Get plural form
translator.is_rtl()                         # Check if RTL language
translator.set_language(code)               # Switch language
```

---

## For Translators: Adding Languages

### 5-Step Process

#### Step 1: Create Translation File
```bash
cp cortex/translations/en.json cortex/translations/de.json
```

#### Step 2: Translate All Values
Edit `de.json` - translate the **values only**, keep keys unchanged:

```json
{
  "common": {
    "yes": "Ja",
    "no": "Nein",
    "error": "Fehler"
  },
  "install": {
    "success": "{package} wurde erfolgreich installiert"
  }
}
```

**Keep these unchanged**:
- JSON structure
- Key names
- Variable placeholders like `{package}`
- Pluralization syntax

#### Step 3: Register Language
Edit `cortex/i18n/language_manager.py`:

```python
SUPPORTED_LANGUAGES: Dict[str, str] = {
    'en': 'English',
    'es': 'Español',
    'de': 'Deutsch',  # ← Add this line
    # ... rest ...
}
```

#### Step 4: Test
```bash
cortex --language de install nginx --dry-run
cortex -L de search python
```

#### Step 5: Submit PR
- Title: `[i18n] Add German Translation`
- Include translation file in PR
- Mention contributors

**That's it!** No other code changes needed.

---

## Translation File Format

### Structure
```json
{
  "namespace": {
    "key": "Translated message"
  }
}
```

### Variables
```json
{
  "install": {
    "success": "{package} installed"  ← Keep {package} unchanged
  }
}
```

Usage:
```python
translator.get('install.success', package='nginx')
# Returns: "nginx installed"
```

### Pluralization
```json
{
  "install": {
    "count": "Downloading {num, plural, one {# file} other {# files}}"
  }
}
```

Usage:
```python
translator.get_plural('install.count', count=5)
# Returns: "Downloading 5 files"
```

**Keep format unchanged**: `{variable, plural, one {...} other {...}}`

---

## Validation

### Check a Translation File
```bash
# Validate all translations
python3 scripts/validate_translations.py

# Strict mode (warnings are errors)
python3 scripts/validate_translations.py --strict
```

### What it Checks
- ✅ Valid JSON syntax
- ✅ All required keys present
- ✅ No extra keys
- ✅ Variable placeholders match
- ✅ Pluralization syntax correct

---

## Common Tasks

### Add a New Translation Key

1. Add to English file `cortex/translations/en.json`:
```json
{
  "install": {
    "checking_cache": "Checking package cache..."
  }
}
```

2. Add to all other translation files:
```json
{
  "install": {
    "checking_cache": "Verificando caché de paquetes..." // Spanish example
  }
}
```

3. Use in code:
```python
translator.get('install.checking_cache')
```

### Change a Translation

Edit the appropriate translation file and update the value (keep key):

```json
// Before
"success": "Package installed"

// After
"success": "Successfully installed package"
```

### Add Support for New Language

1. Copy English: `cp cortex/translations/en.json cortex/translations/[code].json`
2. Translate all strings
3. Add to `SUPPORTED_LANGUAGES` in `language_manager.py`
4. Test: `cortex -L [code] install nginx --dry-run`
5. Submit PR

---

## Troubleshooting

### Translation Not Showing
```python
# Check if key exists
translator.get('namespace.key')
# If returns '[namespace.key]', key doesn't exist

# Check language support
from cortex.i18n import LanguageManager
LanguageManager.is_supported('es')  # True/False
```

### Placeholder Not Replaced
```python
# ✅ Correct
translator.get('msg', package='nginx')

# ❌ Wrong - variable name doesn't match
translator.get('msg', pkg='nginx')  # {package} won't be replaced
```

### RTL Display Issues
```python
translator = Translator('ar')
if translator.is_rtl():
    # System handles display - just use normally
    msg = translator.get('common.yes')
```

### Missing Key Fallback
```python
# Returns placeholder with warning
translator.get('missing.key')  # Returns: '[missing.key]'

# Check missing keys
from cortex.i18n import get_fallback_handler
handler = get_fallback_handler()
print(handler.get_missing_translations())
```

---

## Supported Languages

| Code | Language | Status |
|------|----------|--------|
| en | English | ✓ Complete |
| es | Español | ✓ Complete |
| hi | हिन्दी | ✓ Complete |
| ja | 日本語 | ✓ Complete |
| ar | العربية | ✓ Complete |
| pt | Português | ⏳ Needed |
| fr | Français | ⏳ Needed |

---

## API Reference

### `Translator` Class

```python
from cortex.i18n import Translator

translator = Translator(language='es')

# Methods
translator.get(key: str, **kwargs) -> str
translator.get_plural(key: str, count: int, **kwargs) -> str
translator.is_rtl() -> bool
translator.set_language(language: str) -> bool
```

### `LanguageManager` Class

```python
from cortex.i18n import LanguageManager

manager = LanguageManager(prefs_manager=None)

# Methods
manager.detect_language(cli_arg: str = None) -> str
manager.is_supported(language: str) -> bool
manager.get_available_languages() -> Dict[str, str]
manager.get_language_name(language: str) -> str
manager.get_system_language() -> Optional[str]
```

### `PluralRules` Class

```python
from cortex.i18n import PluralRules

# Methods
PluralRules.get_plural_form(language: str, count: int) -> str
PluralRules.supports_language(language: str) -> bool
```

### `FallbackHandler` Class

```python
from cortex.i18n import get_fallback_handler

handler = get_fallback_handler()

# Methods
handler.handle_missing(key: str, language: str) -> str
handler.get_missing_translations() -> Set[str]
handler.export_missing_for_translation() -> str
handler.report_summary() -> str
```

---

## Performance

- **Translation lookup**: O(1) dictionary access
- **Language switch**: <100ms (JSON file loading)
- **Memory per language**: ~2MB
- **No network calls**: All local files

---

## Security

✅ JSON files only - no code execution risk  
✅ UTF-8 encoded for all scripts  
✅ No user-supplied keys  
✅ Graceful fallback for invalid input

---

## Related Documentation

- **Full Design**: See [I18N_IMPLEMENTATION_PLAN.md](../I18N_IMPLEMENTATION_PLAN.md)
- **PR Description**: See [PR_DESCRIPTION.md](../PR_DESCRIPTION.md)
- **Translator Guide**: See [cortex/translations/README.md](../cortex/translations/README.md)
- **Code Examples**: See `cortex/i18n/translator.py` docstrings

---

## Quick Links

- 🐛 **Report Issues**: [GitHub Issues](https://github.com/cortexlinux/cortex/issues)
- 💬 **Ask Questions**: [Discord](https://discord.gg/uCqHvxjU83)
- 📧 **Contact**: translations@cortexlinux.com
- 📚 **Docs**: https://docs.cortexlinux.com/i18n

---

## Examples by Language

### Spanish
```bash
export CORTEX_LANGUAGE=es
cortex install nginx
```

### Hindi
```bash
cortex -L hi install python3
cortex -L hi search git
```

### Japanese
```bash
CORTEX_LANGUAGE=ja cortex status
cortex --language ja config language
```

### Arabic
```bash
cortex -L ar wizard
cortex -L ar install nginx --dry-run
```

---

## Contributing

See **[cortex/translations/README.md](../cortex/translations/README.md)** for:
- Translation guidelines
- Common mistakes
- Language-specific tips
- Submission process
