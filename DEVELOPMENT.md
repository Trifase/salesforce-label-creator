# Salesforce Apex Mass Label Tool - Technical & LLM Developer Guide

This document provides system-level architecture specifications, Apex class details, Tooling API REST endpoints, and security mechanisms for `apex-mass-label-tool`. It is designed for Large Language Models (LLMs) and Salesforce developers to safely extend, modify, or debug the tool.

---

## 1. System Architecture & Components

```
+-------------------------------------------------------------------+
|                        Lightning UI                               |
|            c:LabelCreator (Aura Component)                        |
+-------------------------------------------------------------------+
                                 |
                                 v
+-------------------------------------------------------------------+
|                    LabelCreatorController.cls                     |
|           (@AuraEnabled controller for progress tracking)          |
+-------------------------------------------------------------------+
                                 |
                                 v
+-------------------------------------------------------------------+
|                           LabelUtils.cls                          |
|    - getApiSessionId() (Session ID workaround via VF Page)       |
|    - createLabel() (Tooling API POST /ExternalString)             |
|    - createTranslation() (Tooling API POST /ExternalStringLoc...) |
|    - updateTranslation() (Tooling API PATCH /ExternalStringLoc...)|
+-------------------------------------------------------------------+
                                 |
                                 v
+-------------------------------------------------------------------+
|                  Salesforce Tooling REST API                      |
|                  /services/data/v56.0/tooling/                    |
+-------------------------------------------------------------------+
```

---

## 2. Session ID Authentication Workaround

### The Problem
In Lightning Experience (`c:LabelCreator`), invoking `UserInfo.getSessionId()` returns a restricted session token. Attempting to call Tooling API endpoints with this token results in an `HTTP 401 Unauthorized` response.

### The Solution
- `GetSessionId.page`: A minimal Visualforce page that outputs `{!$Api.Session_ID}` in a clean text format.
- `UtilsGetSessionId.cls`: Scrapes `Page.GetSessionId.getContent().toString()` inside Apex to extract a full API-capable session ID.
- `LabelUtils.getApiSessionId()`: Caches the API session ID in memory and falls back to `UserInfo.getSessionId()` if VF execution fails.

---

## 3. Tooling API REST Endpoints

### 3.1 Create Custom Label (`ExternalString`)
- **Endpoint**: `/services/data/v56.0/tooling/sobjects/ExternalString`
- **HTTP Method**: `POST`
- **Headers**:
  - `Authorization: Bearer <API_SESSION_ID>`
  - `Content-Type: application/json`
- **Request Body**:
```json
{
  "Name": "Account_Header_Title",
  "MasterLabel": "Account_Header_Title",
  "Value": "Account Overview",
  "Category": "Account",
  "Language": "en_US",
  "isProtected": "false"
}
```
- **Response**: `{"id": "101XXXXXXXXXXXXXXX", "success": true, "errors": []}`

### 3.2 Create Label Translation (`ExternalStringLocalization`)
- **Endpoint**: `/services/data/v56.0/tooling/sobjects/ExternalStringLocalization`
- **HTTP Method**: `POST`
- **Request Body**:
```json
{
  "ExternalStringId": "101XXXXXXXXXXXXXXX",
  "Language": "it",
  "Value": "Panoramica Account"
}
```

### 3.3 Update Label Translation (`ExternalStringLocalization`)
- **Endpoint**: `/services/data/v56.0/tooling/sobjects/ExternalStringLocalization/<LOCALIZATION_ID>`
- **HTTP Method**: `PATCH`
- **Request Body**: `{"Value": "Nuovo Testo Tradotto"}`

---

## 4. Apex Data Structures

### `LabelUtils.LabelWrapper`
```java
public class LabelWrapper {
    public String Name;            // Custom Label API Name (spaces replaced by '_')
    public String MasterLabel;     // Master Label Display Name
    public String Value;           // Master Language Text (en_US)
    public String Category;        // Label Category
    public String Language;        // Master Language Code (default: 'en_US')
    public String isProtected;     // 'true' or 'false'
    public String ItalianValue;    // Optional translation value for quick CSV imports
}
```

---

## 5. Developer & LLM Guidelines

1. **API Name Sanitization**: Custom Label `Name` and `MasterLabel` must not contain spaces or special characters. Always apply `.replaceAll('\\s+', '_')` or `.replace(' ', '_')`.
2. **Session ID Hygiene**: Never remove `GetSessionId.page` or `UtilsGetSessionId.cls`, as direct Lightning REST callouts will fail without Visualforce session elevation.
3. **API Versioning**: Tooling API requests are currently pinned to `v56.0`. If upgrading API version, verify sObject field compatibility.
