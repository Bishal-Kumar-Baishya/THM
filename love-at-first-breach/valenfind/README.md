# ValenFind - CTF

**Target:** ValenFind Dating Application  
**Difficulty:** Medium  
**Status:** Complete ✅

## Summary
Critical Local File Inclusion vulnerability allows unauthenticated attackers to read arbitrary files and compromise the entire application.

## Key Findings
- **Critical LFI** via path traversal in `/api/fetch_layout`
- **Hardcoded API credentials** exposed in source code
- **Full database access** with user credentials and PII

## Files
- `REPORT.md` - Professional penetration test report
- `WALKTHROUGH.md` - Step-by-step exploitation guide

## Quick Links
[Full Report](./REPORT.md) | [Walkthrough](./WALKTHROUGH.md)