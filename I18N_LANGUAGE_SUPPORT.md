# Cortex Linux i18n - Complete Language Support Guide

## 🌍 Supported Languages (12 Total)

### Original Languages (5)
| Code | Language | Native Name | RTL | Status |
|------|----------|------------|-----|--------|
| en | English | English | ✗ | ✓ Complete |
| es | Spanish | Español | ✗ | ✓ Complete |
| ja | Japanese | 日本語 | ✗ | ✓ Complete |
| ar | Arabic | العربية | ✓ | ✓ Complete |
| hi | Hindi | हिन्दी | ✗ | ✓ Complete |

### Newly Added Languages (7)
| Code | Language | Native Name | RTL | Status |
|------|----------|------------|-----|--------|
| pt | Portuguese | Português | ✗ | ✓ Complete |
| fr | French | Français | ✗ | ✓ Complete |
| de | German | Deutsch | ✗ | ✓ Complete |
| it | Italian | Italiano | ✗ | ✓ Complete |
| ru | Russian | Русский | ✗ | ✓ Complete |
| zh | Chinese (Simplified) | 中文 | ✗ | ✓ Complete |
| ko | Korean | 한국어 | ✗ | ✓ Complete |

## Usage Examples

### Python Code
```python
from cortex.i18n import get_translator, LanguageManager

translator = get_translator()
lang_manager = LanguageManager()

# Switch languages
translator.set_language("de")  # German
msg = translator.get("install.prompt")
# Returns: "Was möchten Sie installieren?"

# Get all available languages
langs = lang_manager.get_available_languages()
# Returns: {'en': 'English', 'es': 'Español', ..., 'ko': '한국어'}

# Detect system language
detected = lang_manager.detect_language()
```

### Terminal
```bash
# Test German
python3 << 'EOF'
from cortex.i18n import get_translator
t = get_translator()
t.set_language("de")
print(t.get("common.yes"))  # Ja
print(t.get("errors.invalid_package", package="test"))
