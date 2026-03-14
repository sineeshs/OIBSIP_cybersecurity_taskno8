# PCAP Analysis: Cleartext Credential Exposure & Authentication Failure

## 📌 Overview
This document details the network traffic analysis of an unencrypted HTTP session where a user interacts with a deliberately vulnerable web application. The packet capture (PCAP) demonstrates reconnaissance, session establishment, and a cleartext authentication attempt. This analysis is useful for understanding HTTP application flow and designing custom detection rules for SIEM environments (such as Wazuh) where standard HTTP status code alerts fall short.

## 🎯 Target Information
* **Destination Host:** `testasp.vulnweb.com`.
* **Application Context:** A test and demonstration site for the Acunetix Web Vulnerability Scanner. The site explicitly states it is vulnerable to SQL injections, directory traversal, and other attacks.
* **Server Infrastructure:** Microsoft-IIS/8.5 powered by ASP.NET.

## 🕵️‍♂️ Threat Actor / Client Profile
* **Source OS/Browser:** 64-bit Linux utilizing Mozilla Firefox 128.0 (`Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0`).

## 🔍 Detailed Traffic Flow Analysis

### 1. Initial Reconnaissance & Fingerprinting
* **Action:** The client initiates the session with an `HTTP GET /` request to the root directory.
* **Server Response:** The server responds with `HTTP/1.1 200 OK`.
* **Analysis:** The response headers leak the backend infrastructure (`Server: Microsoft-IIS/8.5`, `X-Powered-By: ASP.NET`), providing an attacker with exact versioning for targeted exploit research.

### 2. Session Establishment & Vulnerability
* **Action:** Alongside the `200 OK` response, the server sets a session tracking cookie.
* **Cookie Value:** `ASPSESSIONIDCSADDQDD=NIPOLKCCFJNCNNCCABGHNNBD`.
* **Analysis:** Because the transmission occurs over unencrypted HTTP (Port 80), this session ID is completely exposed. A network eavesdropper could capture this value and hijack the active session without ever needing the user's credentials.

### 3. Authentication Attempt (Cleartext POST)
* **Action:** After navigating to the login portal (`GET /Login.asp`), the user submits credentials via an `HTTP POST` request to `/Login.asp?RetURL=%2FDefault%2Easp%3F`.
* **Payload Submitted:** The form data is transmitted as `application/x-www-form-urlencoded`.
* **Exposed Credentials:** The packet payload clearly shows the submitted variables: `tfUName=admin&tfUPass=11111`. 

### 4. Server Verdict (Failed Login)
* **Action:** The server processes the POST request and returns an `HTTP/1.1 200 OK`.
* **Analysis:** Despite the authentication failing, the server does *not* return a 401 Unauthorized or 403 Forbidden status code. Instead, it returns a standard 200 OK and embeds the failure message directly within the HTML body: `<b>Invalid login!</b>`.

---
