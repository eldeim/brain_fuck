# Introduction to Mobile Application Security

## The Prenetation Testing Process

1. Pre-engagement
   1. Define sensive data
      1. user creds, user info, etc
      2. PII
      3. Data protected by laws and regulations
   2. Indentify security related items
      1. Hashing
      2. Encyption
      3. Encoding
      4. Token storage (API,session,etc...)
      5. Random number generators
2. Recon
   1. Purpose of the app
   2. Developer of the app
   3. Industry, how the app works
3. App mapping - Scanning
   1. App architecture
      1. How it manafes user sessions
      2. How the app communicates
   2. Threat modeling
4. Exploitation - CI/CD
   1. Not all vulnerabilities are relevant or exploitable
   2. Look at:
      1. Damage potential
      2. Discoverability
      3. Reproducibility
      4. Explotability
      5. Data or user impacted
5. Reporting
   1. Executive summary
   2. Definition of the scope
   3. Methods used
   4. Findings
   5. Recommendatios

***

## Common Mobile Application Vulnerabilities

### Insecure Data Store

* Storing sensitive data like credentials or session tokens in plaintext
* Common storage issues:
  * Using SharedPreferences on unencrypted SQLite databaes
* Impact. Data theft if the device is compromised

### Insecure Communication

* Transmitting sensitive data over HTTP instead if HTTPS
* Failing to validate SSL/TLS cetfs
* Impact: Data interception via man-in-the-middle attacks

### Weak Authentication and Authorization

* Hadcoded credentials in the app
* Poor implementation of user authentication mechanisms
* Missing access control for backend APIs
* Impact: Unathorized access and privilege escalation

### Excesive Permissions

* Apps requesting unnecesary permissions, such as access to location or contacts
* Impact: Exposure of sensitive data that the app doesnt need to function

### Insecure APIs

* APIs with insufficient input validation or authetication
* Overexposed endpoints accessible to unauthorized users
* Impact: Data leaks, unaythorized transactions, or system compromise

Reverse Engineering

* Lack of obfuscation makes it easier for attackers to decompile and analyze apps
* Extracting API keys or modifying app logic
* Impact: Misuse of backend services or bypassing security measure

Outdated Components

* Using third-party libraries with known vulnerabilities
* Impact: Exploitation of unpatched vulnerabilities in libraries or frameworks
