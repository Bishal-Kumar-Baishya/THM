# Hidden Deep Into my Heart - CTF

**Target:** Cupid's Vault
**Difficulty:** Easy 
**Status:** Complete ✅

## Summary
A Flask web application vulnerable to credential exposure through publicly accessible configuration files. Sensitive admin credentials were discovered in robots.txt comments, allowing unauthorized administrative access. The vulnerability demonstrates the critical importance of secure credential management and treating configuration files as security-sensitive assets.

## Key Findings
- Sensitive information (credentials) exposed in robots.txt comments
- Default admin username with weak deployment practices
- Hidden paths provide false sense of security without proper authentication

## Files
- `REPORT.md` - Professional penetration test report
- `WALKTHROUGH.md` - Step-by-step exploitation guide

## Quick Links
[Full Report](./REPORT.md) | [Walkthrough](./WALKTHROUGH.md)