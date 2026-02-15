# Test Plan

> This document describes the testing strategy for this project. It serves as the single source of truth for testing decisions and rationale.

## Overview

**Project:** ap-cull-light
**Primary functionality:** Cull astrophotography light frames based on HFR and RMS thresholds

## Testing Philosophy

This project follows the [ap-base Testing Standards](https://github.com/jewzaam/ap-base/blob/main/standards/testing.md).

Key testing principles for this project:

- All external dependencies (ap_common, filesystem) are mocked to ensure unit test isolation
- Tests verify both success paths and error conditions for file rejection logic
- Threshold evaluation logic is tested with numeric, non-numeric, and missing values

## Test Categories

### Unit Tests

Tests for isolated functions with mocked dependencies.

| Module | Function | Test Coverage | Notes |
|--------|----------|---------------|-------|
| `cull_lights.py` | `reject_image()` | Normal move, dryrun, debug logging, file not under source, OSError, generic exception, path traversal safety | ap_common.move_file is mocked |
| `cull_lights.py` | `cull_lights()` | HFR/RMS threshold evaluation, auto-accept percent, dryrun, skip pattern, both thresholds, non-numeric values, missing filename, user rejection, multiple directory groups, no rejections, empty data, debug logging | ap_common.get_filtered_metadata is mocked |
| `cull_lights.py` | `main()` | CLI argument parsing for all options, validation errors (no thresholds, invalid percent, invalid regex, missing source, non-directory source, same source/reject dir), quiet flag | Uses sys.argv patching |

### Integration Tests

Tests for multiple components working together.

| Workflow | Components | Test Coverage | Notes |
|----------|------------|---------------|-------|
| Quiet mode | `cull_lights()` + `get_filtered_metadata` | printStatus parameter passing | Verifies quiet flag propagates correctly |

## Untested Areas

| Area | Reason Not Tested |
|------|-------------------|
| Actual file I/O (moving files) | ap_common.move_file is mocked; tested in ap-common |
| FITS/XISF header parsing | ap_common.get_filtered_metadata is mocked; tested in ap-common |
| Interactive user prompts (real stdin) | Mocked via `builtins.input`; interactive testing not feasible in CI |

## Bug Fix Testing Protocol

All bug fixes to existing functionality **must** follow TDD:

1. Write a failing test that exposes the bug
2. Verify the test fails before implementing the fix
3. Implement the fix
4. Verify the test passes
5. Verify reverting the fix causes the test to fail again
6. Commit test and fix together with issue reference

### Regression Tests

| Issue | Test | Description |
|-------|------|-------------|

## Coverage Goals

**Target:** 80%+ line coverage

**Philosophy:** Coverage measures completeness, not quality. A test that executes code without meaningful assertions provides no value. Focus on:

- Testing behavior, not implementation details
- Covering edge cases and error conditions
- Ensuring assertions verify expected outcomes

## Running Tests

```bash
# Run all tests
make test

# Run with coverage
make coverage

# Run specific test
pytest tests/test_cull_lights.py::TestClass::test_function
```

## Test Data

Test data is:
- Generated programmatically in fixtures where possible
- Created using `tmp_path` pytest fixture for filesystem isolation
- All metadata dictionaries are constructed inline in test methods

**No Git LFS** - all test data must be small (< 100KB) or generated.

## Maintenance

When modifying this project:

1. **Adding features**: Add tests for new functionality after implementation
2. **Fixing bugs**: Follow TDD protocol above (test first, then fix)
3. **Refactoring**: Existing tests should pass without modification (behavior unchanged)
4. **Removing features**: Remove associated tests

## Changelog

| Date | Change | Rationale |
|------|--------|-----------|
| 2026-02-15 | Initial test plan | Project creation |

---
