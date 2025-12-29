# Multi-Language CLI Support (i18n) Implementation Plan

**Issue**: #93 – Multi-Language CLI Support  
**Status**: Design Phase  
**Target Languages**: English (en), Spanish (es), Hindi (hi), Japanese (ja), Arabic (ar), Portuguese (pt), French (fr)

---

## 1. Architecture Overview

This proposal introduces **python-i18n** as the core i18n framework, providing a lightweight, flexible solution for message catalogs and language management without heavy dependencies.

### Key Principles

- **Minimal Core Impact**: Localization layer isolated from business logic
- **Zero Configuration Required**: Works out-of-the-box with fallback to English
- **Language-Agnostic Design**: Supports any language without code changes
- **User Control**: Language selection via CLI, config files, and environment variables
- **Extensible**: Easy to add new languages, regions, and translation variations

---

## 2. Directory Structure

```
cortex/
├── i18n/
│   ├── __init__.py                 # Core i18n manager module
│   ├── translator.py               # Main Translator class
│   ├── language_manager.py         # Language detection and switching
│   ├── fallback_handler.py         # Fallback logic for missing translations
│   ├── pluralization.py            # Pluralization rules per language
│   └── formatters.py               # Text formatting (RTL, currency, dates)
│
└── translations/
    ├── README.md                   # Translation contributor guide
    ├── __init__.py
    ├── en.json                     # English (source language)
    ├── es.json                     # Spanish
    ├── hi.json                     # Hindi
    ├── ja.json                     # Japanese
    ├── ar.json                     # Arabic (RTL)
    ├── pt.json                     # Portuguese
    └── fr.json                     # French
```

---

## 3. Translation Structure & Examples

### 3.1 JSON Catalog Format

Each translation file uses a hierarchical JSON structure with namespaces for organization:

```json
{
  "common": {
    "yes": "Yes",
    "no": "No",
    "continue": "Continue",
    "cancel": "Cancel",
    "error": "Error",
    "success": "Success",
    "warning": "Warning",
    "confirm": "Are you sure?"
  },
  
  "cli": {
    "help": "Display this help message",
    "version": "Show version information",
    "verbose": "Enable verbose output",
    "quiet": "Suppress non-essential output"
  },
  
  "install": {
    "prompt": "What would you like to install?",
    "checking_deps": "Checking dependencies for {package}",
    "resolving": "Resolving package dependencies...",
    "downloading": "Downloading {package_count, plural, one {# package} other {# packages}}",
    "installing": "Installing {packages}...",
    "success": "{package} installed successfully",
    "failed": "Installation of {package} failed: {error}",
    "dry_run": "[DRY RUN] Would install {packages}",
    "already_installed": "{package} is already installed (version {version})"
  },
  
  "config": {
    "language_set": "Language set to {language}",
    "language_not_found": "Language '{language}' not found. Using English.",
    "current_language": "Current language: {language}",
    "available_languages": "Available languages: {languages}"
  },
  
  "errors": {
    "network": "Network error: {details}",
    "permission": "Permission denied: {details}",
    "invalid_package": "Package '{package}' not found",
    "disk_space": "Insufficient disk space ({needed}GB needed, {available}GB available)",
    "api_key_missing": "API key not configured. Run 'cortex wizard' to set it up.",
    "timeout": "Operation timed out after {seconds} seconds"
  },
  
  "prompts": {
    "confirm_install": "Install {packages}? (y/n)",
    "select_version": "Select version for {package}:",
    "enter_api_key": "Enter your {provider} API key:"
  },
  
  "status": {
    "checking": "Checking system...",
    "detected_os": "Detected OS: {os} {version}",
    "detected_arch": "Architecture: {arch}",
    "hardware_info": "CPU cores: {cores}, RAM: {ram}GB"
  }
}
```

### 3.2 Language-Specific Example: Spanish (es.json)

```json
{
  "common": {
    "yes": "Sí",
    "no": "No",
    "continue": "Continuar",
    "cancel": "Cancelar",
    "error": "Error",
    "success": "Éxito",
    "warning": "Advertencia",
    "confirm": "¿Estás seguro?"
  },
  
  "install": {
    "prompt": "¿Qué te gustaría instalar?",
    "checking_deps": "Verificando dependencias para {package}",
    "downloading": "Descargando {package_count, plural, one {# paquete} other {# paquetes}}",
    "installing": "Instalando {packages}...",
    "success": "{package} instalado exitosamente",
    "failed": "La instalación de {package} falló: {error}",
    "already_installed": "{package} ya está instalado (versión {version})"
  },
  
  "errors": {
    "network": "Error de red: {details}",
    "permission": "Permiso denegado: {details}",
    "invalid_package": "Paquete '{package}' no encontrado"
  }
}
```

### 3.3 Language-Specific Example: Hindi (hi.json)

```json
{
  "common": {
    "yes": "हाँ",
    "no": "नहीं",
    "continue": "जारी रखें",
    "cancel": "रद्द करें",
    "error": "त्रुटि",
    "success": "सफल",
    "warning": "चेतावनी",
    "confirm": "क्या आप सुनिश्चित हैं?"
  },
  
  "install": {
    "prompt": "आप क्या इंस्टॉल करना चाहते हैं?",
    "checking_deps": "{package} के लिए निर्भरताएं जांच रहे हैं",
    "downloading": "{package_count, plural, one {# पैकेज} other {# पैकेज}} डाउनलोड कर रहे हैं",
    "installing": "{packages} स्थापित कर रहे हैं...",
    "success": "{package} सफलतापूर्वक स्थापित हुआ",
    "failed": "{package} की स्थापना विफल रही: {error}"
  },
  
  "errors": {
    "network": "नेटवर्क त्रुटि: {details}",
    "permission": "अनुमति अस्वीकृत: {details}",
    "invalid_package": "पैकेज '{package}' नहीं मिला"
  }
}
```

### 3.4 Language-Specific Example: Japanese (ja.json)

```json
{
  "common": {
    "yes": "はい",
    "no": "いいえ",
    "continue": "続行",
    "cancel": "キャンセル",
    "error": "エラー",
    "success": "成功",
    "warning": "警告",
    "confirm": "よろしいですか?"
  },
  
  "install": {
    "prompt": "何をインストールしたいですか?",
    "checking_deps": "{package} の依存関係を確認中...",
    "downloading": "{package_count, plural, one {# パッケージ} other {# パッケージ}}をダウンロード中",
    "installing": "{packages} をインストール中...",
    "success": "{package} が正常にインストールされました",
    "failed": "{package} のインストールに失敗しました: {error}"
  }
}
```

### 3.5 Language-Specific Example: Arabic (ar.json - RTL)

```json
{
  "common": {
    "yes": "نعم",
    "no": "لا",
    "continue": "متابعة",
    "cancel": "إلغاء",
    "error": "خطأ",
    "success": "نجح",
    "warning": "تحذير",
    "confirm": "هل أنت متأكد?"
  },
  
  "install": {
    "prompt": "ماذا تود تثبيته؟",
    "checking_deps": "جاري التحقق من التبعيات لـ {package}",
    "downloading": "جاري التحميل {package_count, plural, one {# حزمة} other {# حزم}}",
    "installing": "جاري التثبيت {packages}...",
    "success": "تم تثبيت {package} بنجاح",
    "failed": "فشل تثبيت {package}: {error}"
  }
}
```

---

## 4. Core i18n Components

### 4.1 Main Translator Module (`i18n/translator.py`)

```python
"""Core translator module for Cortex Linux i18n support"""

from typing import Any, Dict, Optional
import json
from pathlib import Path


class Translator:
    """
    Main translator class providing message translation and formatting.
    
    Features:
    - Lazy loading of translation catalogs
    - Nested key access (e.g., 'install.success')
    - Variable interpolation with {key} syntax
    - Pluralization support
    - RTL language detection
    - Graceful fallback to English
    """
    
    def __init__(self, language: str = "en"):
        """
        Initialize translator.
        
        Args:
            language: Language code (e.g., 'en', 'es', 'hi', 'ja', 'ar')
        """
        self.language = language
        self._catalogs: Dict[str, Dict[str, Any]] = {}
        self._rtl_languages = {"ar", "he", "ur", "yi"}  # Right-to-left
        
    def get(self, key: str, **kwargs) -> str:
        """
        Get translated message.
        
        Args:
            key: Dot-separated key path (e.g., 'install.success')
            **kwargs: Variables for interpolation
            
        Returns:
            Translated and formatted message
            
        Example:
            translator.get('install.success', package='nginx')
            # Returns: "nginx installed successfully"
        """
        pass
    
    def get_plural(self, key: str, count: int, **kwargs) -> str:
        """
        Get pluralized translation.
        
        Args:
            key: Translation key with plural form
            count: Number for pluralization decision
            **kwargs: Additional format variables
            
        Returns:
            Correctly pluralized message
            
        Example:
            translator.get_plural('install.downloading', 5, package_count=5)
        """
        pass
    
    def is_rtl(self) -> bool:
        """Check if current language is right-to-left"""
        return self.language in self._rtl_languages
    
    def set_language(self, language: str) -> bool:
        """
        Switch to different language.
        
        Args:
            language: Language code
            
        Returns:
            True if language loaded successfully, False otherwise
        """
        pass
    
    def _load_catalog(self, language: str) -> Dict[str, Any]:
        """Load translation catalog for language from JSON file"""
        pass
    
    def _interpolate(self, text: str, **kwargs) -> str:
        """Replace {key} with values from kwargs"""
        pass
```

### 4.2 Language Manager (`i18n/language_manager.py`)

```python
"""Language detection and management"""

from enum import Enum
from typing import Optional
import locale
import os


class LanguageManager:
    """
    Detects and manages language preferences.
    
    Priority order:
    1. CLI argument (--language)
    2. Environment variable (CORTEX_LANGUAGE)
    3. Config file preference
    4. System locale
    5. Fallback to English
    """
    
    SUPPORTED_LANGUAGES = {
        'en': 'English',
        'es': 'Español',
        'hi': 'हिन्दी',
        'ja': '日本語',
        'ar': 'العربية',
        'pt': 'Português',
        'fr': 'Français',
    }
    
    def __init__(self, prefs_manager=None):
        """
        Initialize language manager.
        
        Args:
            prefs_manager: PreferencesManager instance for config access
        """
        self.prefs_manager = prefs_manager
    
    def detect_language(self, cli_arg: Optional[str] = None) -> str:
        """
        Detect language with priority fallback chain.
        
        Priority:
        1. CLI argument
        2. CORTEX_LANGUAGE env var
        3. Preferences file
        4. System locale
        5. English fallback
        
        Args:
            cli_arg: Language code from CLI argument
            
        Returns:
            Validated language code
        """
        pass
    
    def get_system_language(self) -> str:
        """Extract language from system locale settings"""
        pass
    
    def is_supported(self, language: str) -> bool:
        """Check if language is supported"""
        return language in self.SUPPORTED_LANGUAGES
    
    def get_available_languages(self) -> Dict[str, str]:
        """Return dict of all supported languages"""
        return self.SUPPORTED_LANGUAGES.copy()
```

### 4.3 Fallback Handler (`i18n/fallback_handler.py`)

```python
"""Graceful fallback handling for missing translations"""

from typing import Optional


class FallbackHandler:
    """
    Manages fallback behavior when translations are missing.
    
    Strategy:
    1. Return translated message if available in target language
    2. Fall back to English translation if available
    3. Generate placeholder message with key name
    4. Log warning for missing translations
    """
    
    def __init__(self, logger=None):
        """
        Initialize fallback handler.
        
        Args:
            logger: Logger instance for warnings
        """
        self.logger = logger
        self.missing_keys = set()  # Track missing translations
    
    def handle_missing(self, key: str, language: str) -> str:
        """
        Handle missing translation gracefully.
        
        Args:
            key: Translation key that was not found
            language: Target language
            
        Returns:
            Fallback message (English translation, placeholder, or error message)
        """
        pass
    
    def get_missing_translations(self) -> set:
        """Return set of all missing translation keys encountered"""
        return self.missing_keys.copy()
    
    def export_missing_for_translation(self) -> str:
        """
        Export missing translations as CSV for translator team.
        
        Returns:
            CSV content with keys and placeholders
        """
        pass
```

### 4.4 Pluralization Rules (`i18n/pluralization.py`)

```python
"""Language-specific pluralization rules"""

from typing import Callable, Dict


class PluralRules:
    """
    Define pluralization rules for different languages.
    
    Different languages have different pluralization patterns:
    - English: one vs. other (1 package, 5 packages)
    - Spanish: one vs. other
    - French: one vs. other (but culturally different from English)
    - Russian: one, few, many (1, 2-4, 5+)
    - Polish: one, few, many
    - Arabic: zero, one, two, few, many, other
    - Japanese: No plural distinction
    """
    
    RULES: Dict[str, Callable] = {
        'en': lambda n: 'one' if n == 1 else 'other',
        'es': lambda n: 'one' if n == 1 else 'other',
        'fr': lambda n: 'one' if n == 1 else 'other',
        'ja': lambda n: 'other',  # Japanese doesn't distinguish
        'ar': lambda n: arabic_plural_rule(n),
        'hi': lambda n: 'one' if n == 1 else 'other',
        'pt': lambda n: 'one' if n == 1 else 'other',
    }
    
    @classmethod
    def get_plural_form(cls, language: str, count: int) -> str:
        """
        Get plural form for language and count.
        
        Args:
            language: Language code
            count: Number for pluralization
            
        Returns:
            Plural form key ('one', 'few', 'other', etc.)
        """
        rule = cls.RULES.get(language, cls.RULES['en'])
        return rule(count)


def arabic_plural_rule(n: int) -> str:
    """Arabic has 6 plural forms (CLDR standard)"""
    if n == 0:
        return 'zero'
    elif n == 1:
        return 'one'
    elif n == 2:
        return 'two'
    elif n % 100 in (3, 4, 5, 6, 7, 8, 9, 10):
        return 'few'
    elif n % 100 in (11, 12, 13, 14, 15, 16, 17, 18, 19):
        return 'many'
    else:
        return 'other'
```

### 4.5 Text Formatters (`i18n/formatters.py`)

```python
"""Language and script-aware text formatting"""


class TextFormatter:
    """
    Handles language-specific text formatting.
    
    Capabilities:
    - RTL (Right-to-Left) text handling
    - Date/time formatting per locale
    - Number and currency formatting
    - Text direction metadata
    """
    
    def __init__(self, language: str):
        self.language = language
        self.rtl_languages = {'ar', 'he', 'ur', 'yi'}
    
    def format_text(self, text: str) -> str:
        """
        Apply language-specific formatting.
        
        For RTL languages, may add unicode directional markers.
        
        Args:
            text: Text to format
            
        Returns:
            Formatted text with optional directional markers
        """
        if self.language in self.rtl_languages:
            return f"\u202b{text}\u202c"  # Add RTL marks
        return text
    
    def format_date(self, date) -> str:
        """Format date according to locale"""
        pass
    
    def format_number(self, number: float, decimals: int = 2) -> str:
        """Format number with locale-specific separators"""
        pass
    
    def format_list(self, items: list) -> str:
        """
        Format list with locale-specific separators.
        
        Example:
        - English: 'item1, item2, and item3'
        - Spanish: 'elemento1, elemento2 y elemento3'
        """
        pass
```

---

## 5. Integration Points

### 5.1 CLI Integration (`cli.py`)

The CLI will be updated to support:

```python
# In CortexCLI class
def __init__(self, verbose: bool = False, language: str = None):
    self.verbose = verbose
    self.lang_manager = LanguageManager(prefs_manager=self.prefs_manager)
    self.language = language or self.lang_manager.detect_language()
    self.translator = Translator(language=self.language)

def translate(self, key: str, **kwargs) -> str:
    """Convenience method for translations"""
    return self.translator.get(key, **kwargs)
```

### 5.2 User Preferences Integration

```python
# In UserPreferences dataclass
@dataclass
class UserPreferences:
    # ... existing fields ...
    language: str = "en"  # Already exists!
    
# In PreferencesManager
def set_language(self, language_code: str) -> bool:
    """Set language preference and validate"""
    if language_code not in LanguageManager.SUPPORTED_LANGUAGES:
        return False
    self.preferences.language = language_code
    self.save()
    return True
```

### 5.3 New CLI Commands

```bash
# Set language
cortex config language es
cortex config language hi

# Show current language
cortex config language

# Show available languages
cortex config languages

# List missing translations (for developers)
cortex dev translations-missing
```

---

## 6. Language Selection Mechanisms

### 6.1 Method 1: Environment Variable

```bash
export CORTEX_LANGUAGE=es
cortex install nginx

export CORTEX_LANGUAGE=hi
cortex status
```

### 6.2 Method 2: Configuration File

Users can edit `~/.cortex/preferences.yaml`:

```yaml
language: es
verbosity: normal
theme: default
```

### 6.3 Method 3: CLI Argument

```bash
cortex --language ja install python3
cortex -L ar status
```

### 6.4 Method 4: Interactive Wizard

During first run or `cortex wizard`:

```
Welcome to Cortex Linux!

Select your language:
1) English (en)
2) Español (es)
3) हिन्दी (hi)
4) 日本語 (ja)
5) العربية (ar)
6) Português (pt)
7) Français (fr)

Enter number [1]:
```

### 6.5 Priority Order

```
1. CLI argument (--language, -L)
2. Environment variable (CORTEX_LANGUAGE)
3. Config file (~/.cortex/preferences.yaml)
4. System locale (from locale.getdefaultlocale())
5. Fallback: English (en)
```

---

## 7. Fallback Behavior

### Missing Translation Handling

When a translation key is missing:

```
language: hi (Hindi)
key: install.checking_deps
```

**Fallback Chain**:

1. **Try Hindi**: `cortex/translations/hi.json` → if found, return it
2. **Try English**: `cortex/translations/en.json` → if found, return it
3. **Placeholder**: Return `[install.checking_deps]` with warning
4. **Log**: Write warning to debug log: `Missing translation: install.checking_deps (hi)`

### Example

```python
# Attempt to translate with missing key
translator = Translator('ja')
result = translator.get('install.unknown_key', package='nginx')
# Returns: "[install.unknown_key]" if not in ja.json or en.json
# Logs: "WARNING: Missing translation: install.unknown_key (ja)"
```

### Missing Language Handling

If a language code is requested that doesn't exist:

```python
translator = Translator('zz')  # Invalid language code
# Falls back to English
# Logs: "WARNING: Language 'zz' not found, using English"
```

---

## 8. Adding New Languages

### Step 1: Create Translation File

Create `cortex/translations/[lang_code].json` based on English template:

```bash
cp cortex/translations/en.json cortex/translations/de.json
```

### Step 2: Translate Keys

Edit the new file and translate all values (keep keys unchanged):

```json
{
  "common": {
    "yes": "Ja",
    "no": "Nein",
    "error": "Fehler"
  }
}
```

### Step 3: Register Language

Update `i18n/language_manager.py`:

```python
SUPPORTED_LANGUAGES = {
    'en': 'English',
    'es': 'Español',
    'de': 'Deutsch',  # Add new language
    # ... rest ...
}
```

### Step 4: Update Pluralization (if needed)

In `i18n/pluralization.py`, add rules if language has unique pluralization:

```python
RULES: Dict[str, Callable] = {
    # ... existing ...
    'de': lambda n: 'one' if n == 1 else 'other',
}
```

### Step 5: Test

```bash
cortex --language de install nginx --dry-run
```

**No code changes required!** New languages are discovered and loaded automatically.

---

## 9. Edge Cases & Special Handling

### 9.1 Pluralization Examples

```python
# English
translator.get_plural('download.count', 1)   # "Downloading 1 package"
translator.get_plural('download.count', 5)   # "Downloading 5 packages"

# Spanish (same as English)
translator.get_plural('download.count', 1)   # "Descargando 1 paquete"
translator.get_plural('download.count', 5)   # "Descargando 5 paquetes"

# Arabic (6 plural forms)
translator.get_plural('download.count', 0)   # "جاري التحميل 0 حزم" (zero form)
translator.get_plural('download.count', 1)   # "جاري التحميل حزمة واحدة" (one form)
translator.get_plural('download.count', 2)   # "جاري التحميل حزمتين" (two form)
translator.get_plural('download.count', 11)  # "جاري التحميل 11 حزمة" (many form)
```

### 9.2 RTL Language Handling

```python
# Arabic text needs directional markers in terminals
translator = Translator('ar')
if translator.is_rtl():
    # Add RTL marks: U+202B (RLE) and U+202C (PDF)
    text = formatter.format_text(text)
```

### 9.3 Variable Interpolation with Special Characters

```python
# Support escaped braces for literal output
translator.get('path.template', pattern='{{*}}')
# Returns: "Pattern: {{*}}" (braces not interpolated)
```

### 9.4 Dynamic Pluralization in Keys

```json
{
  "download": {
    "count": "Downloading {package_count, plural, one {# package} other {# packages}}"
  }
}
```

### 9.5 Context-Specific Translations

For ambiguous words that have different meanings in different contexts:

```python
# Using namespacing
translator.get('verb.install', package='nginx')     # "Install" (action)
translator.get('noun.install', version='2.1')       # "Installation" (noun)
```

### 9.6 Date & Number Formatting

```python
# Locale-aware formatting
formatter = TextFormatter('de')
formatter.format_date(datetime.now())      # "25.12.2023"
formatter.format_number(1234.56)           # "1.234,56"

formatter = TextFormatter('en')
formatter.format_date(datetime.now())      # "12/25/2023"
formatter.format_number(1234.56)           # "1,234.56"
```

### 9.7 Missing Environment Variable Handling

```python
# If CORTEX_LANGUAGE env var has invalid value
os.environ['CORTEX_LANGUAGE'] = 'invalid'
lang_manager.detect_language()  # Falls back to system locale, then English
```

---

## 10. Testing Strategy

### 10.1 Unit Tests

```python
# tests/test_i18n/test_translator.py
def test_basic_translation():
    translator = Translator('es')
    assert translator.get('common.yes') == 'Sí'

def test_interpolation():
    translator = Translator('en')
    result = translator.get('install.success', package='nginx')
    assert result == 'nginx installed successfully'

def test_fallback_to_english():
    translator = Translator('ja')
    # If key missing in ja.json, should get English
    result = translator.get('some.missing.key')
    # Should not crash, should have fallback

def test_pluralization():
    translator = Translator('en')
    one = translator.get_plural('install.packages', 1)
    many = translator.get_plural('install.packages', 5)
    assert 'package' in one.lower()
    assert 'packages' in many.lower()

def test_rtl_detection():
    ar_translator = Translator('ar')
    assert ar_translator.is_rtl() is True
    
    en_translator = Translator('en')
    assert en_translator.is_rtl() is False

def test_language_switching():
    translator = Translator('en')
    assert translator.language == 'en'
    translator.set_language('es')
    assert translator.language == 'es'
```

### 10.2 Integration Tests

```python
# tests/test_i18n/test_cli_integration.py
def test_cli_with_language_arg():
    cli = CortexCLI(language='es')
    output = cli.translate('common.yes')
    assert output == 'Sí'

def test_language_detection_priority():
    # Test that CLI arg > env var > config > system locale
    pass

def test_first_run_wizard_language():
    # Test language selection in wizard
    pass
```

### 10.3 Translation Coverage

```python
# tests/test_i18n/test_coverage.py
def test_all_languages_have_base_keys():
    """Ensure all required keys exist in every language"""
    base_keys = load_translation_keys('en')
    for lang_code in LanguageManager.SUPPORTED_LANGUAGES:
        lang_keys = load_translation_keys(lang_code)
        missing = base_keys - lang_keys
        assert not missing, f"Missing keys in {lang_code}: {missing}"

def test_no_extra_keys_in_translations():
    """Ensure no language has extra keys not in English"""
    en_keys = load_translation_keys('en')
    for lang_code in LanguageManager.SUPPORTED_LANGUAGES:
        lang_keys = load_translation_keys(lang_code)
        extra = lang_keys - en_keys
        assert not extra, f"Extra keys in {lang_code}: {extra}"
```

---

## 11. Configuration and Setup

### 11.1 Dependencies to Add

```
# pyproject.toml
dependencies = [
    # ... existing ...
    "python-i18n>=0.3.9",
]
```

### 11.2 Environment Setup

```bash
# Install package with i18n support
pip install -e ".[i18n]"

# Or install all development dependencies
pip install -r requirements-dev.txt
```

### 11.3 First-Run Setup

On first run, if no language is configured:

```
Welcome to Cortex Linux 0.1.0!

Select your language:
1) English (en)
2) Español (es)
3) हिन्दी (hi)
4) 日本語 (ja)
5) العربية (ar)
6) Português (pt)
7) Français (fr)

Your choice [1 (English)]:
```

---

## 12. Translation Contributor Guide

### For Contributors

1. **Fork the repository**
2. **Choose a language** from the supported list or request a new one
3. **Copy `cortex/translations/en.json`** as a starting point
4. **Translate all keys** maintaining JSON structure
5. **Test locally**:
   ```bash
   cortex --language [code] install nginx --dry-run
   ```
6. **Submit PR** with translation file and updated `SUPPORTED_LANGUAGES`

### Key Guidelines

- ✅ Keep JSON structure identical to English
- ✅ Translate values only, never change keys
- ✅ Keep variable placeholders (`{key}`) unchanged
- ✅ Maintain punctuation and formatting
- ✅ Test with special characters and long strings
- ✅ Follow grammar and conventions of target language
- ❌ Don't add extra keys
- ❌ Don't change the structure

---

## 13. Rollout Plan

### Phase 1: Foundation (Week 1-2)
- ✅ Create i18n module structure
- ✅ Implement `Translator` class
- ✅ Create English translation catalog
- ✅ Add basic CLI integration

### Phase 2: Languages (Week 3-4)
- Add Spanish, Hindi, Japanese translations
- Implement language manager and detection
- Add user preference integration

### Phase 3: Advanced Features (Week 5-6)
- Pluralization support
- RTL language handling
- Date/number formatting
- Fallback mechanisms

### Phase 4: Testing & Polish (Week 7-8)
- Comprehensive unit tests
- Integration testing
- Documentation
- Contributor guide

### Phase 5: Release
- Version bump to 0.2.0
- Update README with i18n docs
- Announce new feature
- Invite community translations

---

## 14. Success Metrics

- [ ] CLI fully functional in 7 languages
- [ ] >95% translation coverage for all languages
- [ ] Zero runtime errors for missing translations
- [ ] Language switching within 100ms
- [ ] >90% test coverage for i18n module
- [ ] Documentation complete for contributors
- [ ] Community contributions for new languages

---

## 15. References & Resources

- **python-i18n**: https://pypi.org/project/python-i18n/
- **CLDR Pluralization Rules**: http://cldr.unicode.org/index/cldr-spec/plural-rules
- **RFC 5646 Language Tags**: https://tools.ietf.org/html/rfc5646
- **Unicode Bidirectional Text**: https://unicode.org/reports/tr9/
- **Gettext Format**: https://www.gnu.org/software/gettext/manual/

---

## Questions & Discussion

- What other languages should be supported initially?
- Should we use JSON or YAML for translation catalogs?
- How do we handle community translations?
- Should we support region-specific variants (e.g., en-US vs en-GB)?

