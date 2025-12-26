# Issue: User Preferences Commands Removal

**Created:** 26 December 2024  
**Status:** Open  
**Priority:** Medium  
**Type:** Feature Removal / Refactoring

## Summary

Remove the `check-pref` and `edit-pref` CLI commands from Cortex while preserving the underlying `user_preferences.py` module for internal use by other components (e.g., snapshot manager, coordinator).

## Background

The user preferences system was implemented to allow users to configure Cortex behavior. However, the CLI commands (`check-pref` and `edit-pref`) are being removed from the user-facing interface while the underlying preference management system remains for internal use.

## Current Implementation

### Commands to Remove

1. **`check-pref` command** (`cortex/cli.py`)
   - Location: Lines 1064-1069
   - Function: `check_pref(self, key: str | None = None)` (Lines 636-655)
   - Purpose: View current preferences or specific preference key

2. **`edit-pref` command** (`cortex/cli.py`)
   - Location: Lines 1071-1077
   - Function: `edit_pref(self, action, key, value)` (Lines 659-704)
   - Purpose: Modify preferences (set, add, delete, list, validate)

### Dependencies to Preserve

The following components **must remain** as they are used internally:

- `cortex/user_preferences.py` - Core preferences module
- `PreferencesManager` class - Used by:
  - `cortex/snapshot_manager.py` (line 88)
  - Other internal components
- Test file: `tests/test_user_preferences.py`
- Documentation: `docs/USER_PREFERENCES_IMPLEMENTATION.md`

## Changes Required

### 1. Remove CLI Commands

**File:** `cortex/cli.py`

Remove:
- Import statements (lines 16-19):
  ```python
  from cortex.user_preferences import (
      PreferencesManager,
      format_preference_value,
      print_all_preferences,
  )
  ```
  
- Instance variable initialization (line 33):
  ```python
  self.prefs_manager = None  # Lazy initialization
  ```

- Methods:
  - `_get_prefs_manager(self)` (lines 630-633)
  - `check_pref(self, key)` (lines 636-655)
  - `edit_pref(self, action, key, value)` (lines 659-704)

- Argument parser setup (lines 1063-1077):
  ```python
  # Preferences commands
  check_pref_parser = subparsers.add_parser(...)
  edit_pref_parser = subparsers.add_parser(...)
  ```

- Command handlers in main() (lines ~1190-1193):
  ```python
  elif args.command == "check-pref":
      return cli.check_pref(key=args.key)
  elif args.command == "edit-pref":
      return cli.edit_pref(action=args.action, key=args.key, value=args.value)
  ```

### 2. Update Help Text

**File:** `cortex/cli.py`

Remove references to user preference commands from:
- Main help text in `show_rich_help()` function
- Any examples mentioning `check-pref` or `edit-pref`

### 3. Update Documentation

**Files to Update:**
- `docs/COMMANDS.md` - Remove preference command documentation
- `README.md` - Remove any examples using preference commands
- `CHANGELOG.md` - Add entry noting command removal

## Verification Steps

1. **Test that CLI still works:**
   ```bash
   cortex --help
   cortex install nginx --dry-run
   cortex history
   cortex stack --list
   ```

2. **Verify internal usage still works:**
   ```bash
   # Snapshot manager should still function
   cortex snapshot create "test"
   cortex snapshot list
   ```

3. **Run tests:**
   ```bash
   pytest tests/test_user_preferences.py -v
   pytest tests/ -v
   ```

4. **Check for broken imports:**
   ```bash
   python -c "from cortex.user_preferences import PreferencesManager; print('OK')"
   ```

## Files Modified

- [ ] `cortex/cli.py` - Remove CLI commands and handlers
- [ ] `docs/COMMANDS.md` - Update command documentation
- [ ] `CHANGELOG.md` - Document the change

## Files Preserved

- [x] `cortex/user_preferences.py` - Keep module
- [x] `tests/test_user_preferences.py` - Keep tests
- [x] `docs/USER_PREFERENCES_IMPLEMENTATION.md` - Keep implementation docs

## Rollback Plan

If this change causes issues, the commands can be restored by:
1. Reverting changes to `cortex/cli.py`
2. Re-adding the command parser definitions
3. Re-adding the command handlers in `main()`

## Related Issues

- User preferences are still accessible programmatically
- Internal components can continue using `PreferencesManager`
- May want to document that preferences are for internal use only

## Testing

All existing tests should pass:
```bash
pytest tests/test_user_preferences.py -v
pytest tests/ -k "not test_concurrent" -v
```

## Notes

- This is a **breaking change** for users who were using `check-pref` or `edit-pref` commands
- Consider adding a deprecation notice in the next release before removal
- The wizard command (`cortex wizard`) may still interact with preferences internally

## Next Steps

1. Review and approve this issue
2. Implement the changes
3. Update documentation
4. Test thoroughly
5. Update CHANGELOG.md
6. Merge to main branch
