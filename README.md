# Salesforce Apex Mass Label & Translation Tool ⚡🏷️

> A URL-addressable Salesforce component and Apex utility to mass create and translate Custom Labels using the Salesforce Tooling API.

`apex-mass-label-tool` provides a streamlined solution for Salesforce administrators and developers to bulk create Custom Labels (`ExternalString`) and their corresponding translations (`ExternalStringLocalization`) directly from Lightning Experience, bypassing the manual standard Salesforce UI limits.

---

## ✨ Features

- **Bulk Custom Label Creation**: Mass create Custom Labels from formatted CSV text or structured inputs.
- **Automated Localization & Translations**: Seamlessly create and update translations (e.g. Italian, Spanish, German, French) alongside or after label creation.
- **Visualforce Session ID Workaround**: Automatically retrieves an API-capable session ID from Visualforce (`GetSessionId.page`) to overcome Lightning restricted session ID limitations (`401 Unauthorized` on Tooling API calls).
- **URL Addressable Lightning Component**: Launchable directly via URL:
  `https://<your-org-domain>/lightning/cmp/c__LabelCreator`
- **Real-Time UI Progress**: Processes labels sequentially with immediate success/failure feedback per label.

---

## 🚀 Installation & Setup

1. **Deploy Components to Salesforce**:
   Deploy the Apex classes, Visualforce page, and Aura component bundle to your target Salesforce org:
   - `LabelCreatorController.cls`
   - `LabelUtils.cls`
   - `UtilsGetSessionId.cls`
   - `GetSessionId.page`
   - `LabelCreator` (Aura Bundle)

2. **Access the Component**:
   Navigate to the component via URL in your Lightning org:
   ```
   https://<your-org-domain>/lightning/cmp/c__LabelCreator
   ```

---

## 🛠️ Usage Example

### CSV Format for Mass Import
Paste CSV formatted text into the component input:

```csv
Label_Name;Category_Name;English Master Text;Italian Translation Text
Account_Header_Title;Account;Account Overview;Panoramica Account
Contact_Email_Error;Validation;Please enter a valid email address;Inserisci un indirizzo email valido
```

---

## 🛠️ Tech Stack & API Integration

- **Salesforce Apex**: API v56.0
- **Salesforce Tooling API**: Endpoint `/services/data/v56.0/tooling/sobjects/`
  - `ExternalString`: Custom Label sObject
  - `ExternalStringLocalization`: Custom Label Translation sObject
- **Aura Component Framework**: URL Addressable interface (`c:LabelCreator`)

---

## 🤖 AI / LLM Developer Documentation

For technical specifications, Tooling API REST endpoints, session ID retrieval mechanisms, and Apex class relationship diagrams, please read:

👉 **[DEVELOPMENT.md](DEVELOPMENT.md)**
