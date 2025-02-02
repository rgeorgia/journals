---
title: 'Book 542.2'
date: 2025-02-01T16:45:34-05:00
draft: true
categories: ["sans"]
tags: ["infosec", "hacking", "sans"]
---

# Fuzzing, Scanning, Augthentication, and Session Testing

## Fuzzing

### Which inputs should be fuzzed?
All of them!
- Request headers
- POST parameters
- GET params
- PUT payloads
- **Any** to the client or server

Using generic word lists of common attack vectors
Specialized word lists for specific vulnerabilities
Credential lists

**Goal** is to fuzz as many inputs as possible. Focus on high-risk functions and places where sensitive data may live.

### Interception Proxy Fuzzing

For Burp, use "Intruder"
For Zap, use "Fuzz..."

**STEPS**
1. Choose a request with values to fuzz
2. Send the request to the fuzzing feature
3. Select fields to fuzz
4. Associate payloads with the chosen fields
5. Set desired options
6. Run the fuzzer
7. Review results


