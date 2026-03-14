# PCAP Analysis: Cleartext Credential Exposure & Authentication Failure

## 📌 Overview
This document details the network traffic analysis of an unencrypted HTTP session where a user interacts with a deliberately vulnerable web application. The packet capture (PCAP) demonstrates reconnaissance, session establishment, and a cleartext authentication attempt. This analysis is useful for understanding HTTP application flow and designing custom detection rules for SIEM environments (such as Wazuh) where standard HTTP status code alerts fall short.

## 🎯 Target Information
* [cite_start]**Destination Host:** `testasp.vulnweb.com` [cite: 1]
* [cite_start]**Application Context:** A test and demonstration site for the Acunetix Web Vulnerability Scanner[cite: 2]. [cite_start]The site explicitly states it is vulnerable to SQL injections, directory traversal, and other attacks[cite: 5].
* [cite_start]**Server Infrastructure:** Microsoft-IIS/8.5 powered by ASP.NET[cite: 2].

## 🕵️‍♂️ Threat Actor / Client Profile
* [cite_start]**Source OS/Browser:** 64-bit Linux utilizing Mozilla Firefox 128.0 (`Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0`)[cite: 1, 11].

## 🔍 Detailed Traffic Flow Analysis

### 1. Initial Reconnaissance & Fingerprinting
* [cite_start]**Action:** The client initiates the session with an `HTTP GET /` request to the root directory[cite: 1].
* [cite_start]**Server Response:** The server responds with `HTTP/1.1 200 OK`[cite: 2].
* [cite_start]**Analysis:** The response headers leak the backend infrastructure (`Server: Microsoft-IIS/8.5`, `X-Powered-By: ASP.NET`)[cite: 2], providing an attacker with exact versioning for targeted exploit research.

### 2. Session Establishment & Vulnerability
* [cite_start]**Action:** Alongside the `200 OK` response, the server sets a session tracking cookie[cite: 2].
* [cite_start]**Cookie Value:** `ASPSESSIONIDCSADDQDD=NIPOLKCCFJNCNNCCABGHNNBD`[cite: 2].
* [cite_start]**Analysis:** Because the transmission occurs over unencrypted HTTP (Port 80), this session ID is completely exposed[cite: 1, 2]. A network eavesdropper could capture this value and hijack the active session without ever needing the user's credentials.

### 3. Authentication Attempt (Cleartext POST)
* [cite_start]**Action:** After navigating to the login portal (`GET /Login.asp`) [cite: 10][cite_start], the user submits credentials via an `HTTP POST` request to `/Login.asp?RetURL=%2FDefault%2Easp%3F`[cite: 18].
* [cite_start]**Payload Submitted:** The form data is transmitted as `application/x-www-form-urlencoded`[cite: 18].
* [cite_start]**Exposed Credentials:** The packet payload clearly shows the submitted variables: `tfUName=admin&tfUPass=11111`[cite: 18]. 

### 4. Server Verdict (Failed Login)
* [cite_start]**Action:** The server processes the POST request and returns an `HTTP/1.1 200 OK`[cite: 19].
* **Analysis:** Despite the authentication failing, the server does *not* return a 401 Unauthorized or 403 Forbidden status code. [cite_start]Instead, it returns a standard 200 OK [cite: 19] [cite_start]and embeds the failure message directly within the HTML body: `<b>Invalid login!</b>`[cite: 23].

---
