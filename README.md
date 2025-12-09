# Phase 4 Combined Integration Test: Mixed Poetry and UV

## Test ID
T-P4-005

## Category
Combined Integration - Migration Scenario / Mixed Package Managers

## Priority
P1

## Description
This fixture simulates a project in migration from Poetry to UV, or a project that supports both package managers. It contains both Poetry and UV configuration sections and lock files, testing how the dependency tree builder handles ambiguous or mixed configurations.

## Features Combined
1. **Poetry Configuration** - [tool.poetry] with Poetry-style dependencies
2. **UV Configuration** - [tool.uv] with UV-specific settings
3. **Dual Lock Files** - Both poetry.lock and uv.lock present
4. **Dependency Conflicts** - Some packages defined in both formats
5. **Environment Variables** - Support for ignoring Poetry when UV is preferred
6. **Migration Patterns** - Common patterns during Poetry→UV migration

## Real-World Inspiration
Based on patterns from:
- Projects migrating from Poetry to UV
- Projects maintaining compatibility with both tools
- Enterprise codebases during transition periods
- Teams evaluating UV alongside Poetry

## Project Structure
```
mixed-poetry-uv/
├── pyproject.toml (contains both [tool.poetry] and [tool.uv])
├── poetry.lock (Poetry lock file)
├── uv.lock (UV lock file)
└── README.md
```

## Expected Behavior

### Scenario 1: Both Lock Files Present
The dependency tree builder should:
1. Detect both poetry.lock and uv.lock
2. Prefer uv.lock by default (UV is the target tool)
3. Provide option to force Poetry lock file usage
4. Warn about potential conflicts

### Scenario 2: Environment Variable Control
```bash
# Prefer UV (default)
dependency-tree-builder scan .

# Force Poetry lock file (if needed for comparison)
PREFER_POETRY=true dependency-tree-builder scan .

# Ignore Poetry completely (UV-only mode)
IGNORE_POETRY=true dependency-tree-builder scan .
```

### Scenario 3: Dependency Conflicts
When same package is defined differently:
- Poetry: `requests = "^2.31.0"`
- UV: `requests>=2.31.0,<3.0.0`

Tool should:
- Use UV version if uv.lock present
- Flag discrepancies between Poetry and UV definitions
- Report which package manager was used for each dependency

## Test Objectives
1. Verify detection of both Poetry and UV configurations
2. Verify lock file priority (uv.lock preferred)
3. Verify environment variable support for package manager selection
4. Verify handling of dependency definition conflicts
5. Verify migration scenario support
6. Verify clear error messages when configurations conflict

## Success Criteria
- [ ] Both [tool.poetry] and [tool.uv] sections detected
- [ ] Both poetry.lock and uv.lock detected
- [ ] uv.lock used by default when both present
- [ ] Environment variable to force Poetry lock works
- [ ] Environment variable to ignore Poetry works
- [ ] Dependency conflicts reported clearly
- [ ] Tool doesn't crash on mixed configuration
- [ ] Clear indication of which package manager was used

## Configuration Variants

### Variant A: Both Lock Files (Default)
- poetry.lock: 45 dependencies
- uv.lock: 43 dependencies (UV resolves more efficiently)
- Expect: uv.lock used, difference noted

### Variant B: UV Lock Only
- poetry.lock: missing
- uv.lock: present
- Expect: uv.lock used, Poetry config ignored

### Variant C: Poetry Lock Only
- poetry.lock: present
- uv.lock: missing
- Expect: poetry.lock used with warning

### Variant D: No Lock Files
- Both lock files missing
- Expect: Error or warning, cannot build tree without lock

## Migration Patterns Tested

### Pattern 1: Gradual Migration
```toml
[tool.poetry]
# Legacy dependencies still defined
dependencies = {...}

[tool.uv]
# New UV-specific configuration
dev-dependencies = [...]
```

### Pattern 2: Parallel Support
```toml
[project]
# PEP 621 dependencies (UV-compatible)
dependencies = [...]

[tool.poetry]
# Poetry-specific metadata for compatibility
```

### Pattern 3: UV-Only with Poetry History
```toml
[project]
# Full UV configuration

# [tool.poetry]
# Commented out Poetry config for reference
```

## UV Version Compatibility
- Minimum: UV 0.4.0+ (for handling mixed configs)
- Recommended: UV 0.7.0+

## Files in this Fixture
- `pyproject.toml` - Mixed Poetry and UV configuration
- `poetry.lock` - Poetry lock file
- `uv.lock` - UV lock file
- `README.md` - This file

## Usage
```bash
# Default behavior (prefer UV)
dependency-tree-builder scan . > output_uv.json

# Force Poetry lock file
PREFER_POETRY=true dependency-tree-builder scan . > output_poetry.json

# Compare outputs
diff output_uv.json output_poetry.json

# Ignore Poetry completely
IGNORE_POETRY=true dependency-tree-builder scan . > output_uv_only.json
```

## Mend Integration Notes
This fixture tests Mend's ability to:
- Detect project's primary package manager
- Handle migration scenarios
- Scan projects during transition period
- Configure which package manager to use
- Report which lock file was used

**Mend Configuration for Mixed Projects:**
```properties
# Prefer UV
python.resolveDependencies=true
python.packageManager=uv

# Or prefer Poetry
python.packageManager=poetry
python.poetryPath=poetry
```

## Related Tests
- T-P1-005: Lock File Format Compatibility
- T-P1-006: pyproject.toml Parsing
- T-P2-001: Complex pyproject.toml

## Edge Cases

### Edge Case 1: Version Mismatch
Poetry resolves to requests 2.31.0, UV resolves to requests 2.32.0
- Report: Dependency version differs between Poetry and UV

### Edge Case 2: Different Dependency Lists
Poetry includes package X, UV doesn't (or vice versa)
- Report: Dependency found in Poetry but not UV (or vice versa)

### Edge Case 3: Workspace with Mixed Managers
Some workspace members use Poetry, others use UV
- Detect: Mixed package managers in workspace
- Recommend: Standardize on one tool

## Test Variations
1. **With Both Lock Files** - Test default UV preference
2. **With Only poetry.lock** - Test Poetry fallback
3. **With Only uv.lock** - Test UV-only mode
4. **With No Lock Files** - Test error handling
5. **With Conflicting Versions** - Test conflict reporting
6. **With Workspace** - Test mixed managers in monorepo