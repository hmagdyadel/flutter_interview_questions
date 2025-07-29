# Fintech Mobile Security - Interview Questions

## Table of Contents
1. [Mobile Security Threats](#mobile-security-threats)
2. [Data Encryption & Storage](#data-encryption--storage)
3. [Advanced Payment Security](#advanced-payment-security)
4. [Network Security (SSL/TLS)](#network-security-ssltls)
5. [Cryptographic Concepts](#cryptographic-concepts)
6. [Memory & PIN Security](#memory--pin-security)
7. [EMV Standards](#emv-standards)
8. [Mobile Wallets](#mobile-wallets)
9. [Device Security](#device-security)
10. [Compliance & Best Practices](#compliance--best-practices)

---

## Mobile Security Threats

### Q1: What are the top 5 security threats in fintech mobile applications?
**Answer:**
1. **Insecure Data Storage** - Storing sensitive data in plain text in SharedPreferences or SQLite
2. **Unsecured Communication** - Using HTTP instead of HTTPS, lack of certificate pinning
3. **Reverse Engineering** - Unobfuscated APKs exposing business logic and API keys
4. **Code Injection & Runtime Hooking** - Tools like Frida modifying app behavior
5. **Screen Overlay Attacks** - Malicious overlays stealing credentials and PINs

### Q2: How would you protect against screen overlay attacks in a payment app?
**Answer:**
- Use custom in-app PIN pads instead of system keyboard
- Implement `FLAG_SECURE` to prevent screenshots
- Detect overlay attacks using `SYSTEM_ALERT_WINDOW` permission checks
- Randomize PIN pad layout for each session
- Monitor for suspicious accessibility services

### Q3: What is runtime hooking and how do you protect against it?
**Answer:**
Runtime hooking uses tools like Frida or Xposed to modify app behavior in real-time. Protection methods:
- Use Native Development Kit (NDK) for critical security functions
- Implement anti-debugging checks
- Code obfuscation and packing
- Runtime Application Self-Protection (RASP)
- Regular integrity checks during app execution

---

## Data Encryption & Storage

### Q4: How do you securely store sensitive data like authentication tokens in Android?
**Answer:**
```kotlin
// Use EncryptedSharedPreferences with Android Keystore
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
```

### Q5: What's the difference between Android Keystore and regular encryption?
**Answer:**
- **Android Keystore**: Hardware-backed security, keys never leave secure hardware, tamper-resistant
- **Regular Encryption**: Software-based, keys stored in app memory/storage, vulnerable to extraction
- **Benefits of Keystore**: Keys generated in secure hardware, biometric authentication, automatic key rotation

### Q6: When would you use SQLCipher vs EncryptedSharedPreferences?
**Answer:**
- **EncryptedSharedPreferences**: Small key-value data (tokens, settings, user preferences)
- **SQLCipher**: Large structured data, complex queries, relational data, transaction history
- **Performance**: SharedPreferences faster for simple data, SQLCipher better for complex operations

---

## Advanced Payment Security

### Q7: What is DUKPT and why is it important in payment processing?
**Answer:**
**DUKPT (Derived Unique Key Per Transaction)**:
- Each transaction uses a unique encryption key
- Prevents key reuse attacks
- If one key is compromised, only that transaction is affected
- Essential for EMV, contactless, and mobile payments
- Used in POS terminals and payment SDKs

### Q8: Explain the difference between tokenization and encryption in payment systems.
**Answer:**
| Aspect | Tokenization | Encryption |
|--------|-------------|------------|
| **Process** | Replace data with meaningless token | Convert data using mathematical algorithm |
| **Reversibility** | Requires secure token vault | Reversible with correct key |
| **Example** | `4111111111111111` → `TOK_8ds7f2gh` | `4111111111111111` → `Af8hd832u83hf...` |
| **Use Case** | Long-term storage, recurring payments | Temporary protection, data in transit |
| **Security** | No mathematical relationship | Mathematical relationship exists |

**When to use**: Tokenization for stored card references (Google Pay), Encryption for temporary data protection.

---

## Network Security (SSL/TLS)

### Q9: What is certificate pinning and why is it crucial for fintech apps?
**Answer:**
Certificate pinning ensures the app only trusts specific certificates, preventing Man-in-the-Middle (MITM) attacks.

**Implementation with OkHttp:**
```kotlin
val certificatePinner = CertificatePinner.Builder()
    .add("api.bank.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()
```

**Why crucial**: Prevents attackers from using fake certificates to intercept financial transactions.

### Q10: How do you implement secure API communication in a fintech app?
**Answer:**
1. **Use TLS 1.3** for all communications
2. **Certificate Pinning** to prevent MITM attacks
3. **Request/Response Signing** using HMAC or digital signatures
4. **API Key Security** - never hardcode, use secure storage
5. **Timeout Configuration** to prevent hanging connections
6. **Retry Logic** with exponential backoff
7. **Rate Limiting** on client side

---

## Cryptographic Concepts

### Q11: Explain the difference between RSA and AES encryption and when to use each.
**Answer:**

**RSA (Asymmetric)**:
- **Key Pair**: Public and private keys
- **Speed**: Slower, suitable for small data
- **Use Cases**: Key exchange, digital signatures, authentication
- **Example**: Encrypting AES keys, EMV transactions

**AES (Symmetric)**:
- **Key**: Same key for encryption/decryption
- **Speed**: Fast, suitable for large data
- **Use Cases**: Data encryption, bulk data protection
- **Example**: Encrypting card numbers, tokens, files

**Best Practice**: Use RSA to exchange AES keys, then use AES for actual data encryption.

### Q12: What is a cryptographic nonce and why is it important?
**Answer:**
**Nonce (Number Used Once)**:
- Unique value used only once for each encryption operation
- Prevents replay attacks and ensures encryption uniqueness
- **Critical for AES-GCM**: Reusing nonce completely breaks security
- **Required for ECDSA**: Different parameters needed for each signature

**Implementation:**
```kotlin
// Generate unique nonce for each encryption
val nonce = ByteArray(12)
SecureRandom().nextBytes(nonce)
```

### Q13: What is HMAC and how is it used in API security?
**Answer:**
**HMAC (Hash-based Message Authentication Code)**:
- Combines secret key with hashing algorithm (SHA-256)
- Provides authentication and integrity (not encryption)
- Verifies message hasn't been tampered with

**API Usage:**
```kotlin
// Sign API request
val signature = generateHMAC(requestData + timestamp, secretKey)
// Send: data + timestamp + signature
// Server verifies signature to ensure authenticity
```

**Benefits**: Prevents message tampering, ensures request authenticity, non-repudiation.

---

## Memory & PIN Security

### Q14: How do you securely handle sensitive data in memory?
**Answer:**
**Problems with Strings:**
- Immutable in Java/Kotlin
- Remain in heap memory indefinitely
- Visible in memory dumps

**Secure Approach:**
```kotlin
// Wrong - String persists in memory
val pin = "1234"

// Correct - ByteArray can be cleared
val pinBytes = "1234".toByteArray()
// Use the data
pinBytes.fill(0) // Clear immediately after use
```

**Best Practices:**
- Use `ByteArray` or `CharArray` for sensitive data
- Clear data immediately after use
- Never log sensitive information
- Use `FLAG_SECURE` to prevent screenshots

### Q15: How would you implement a secure PIN entry screen?
**Answer:**
**Security Measures:**
1. **Custom PIN Pad**: Don't use system keyboard
2. **Randomized Layout**: Change button positions each session
3. **Screen Protection**: Use `FLAG_SECURE` to prevent capture
4. **No Screenshots**: Disable screen recording during PIN entry
5. **Overlay Detection**: Check for malicious overlays
6. **Immediate Clearing**: Clear PIN from memory after use

**Implementation:**
```kotlin
class SecurePinView : View {
    private var pinDigits = CharArray(4)
    
    override fun onDetachedFromWindow() {
        pinDigits.fill('*') // Clear PIN when view destroyed
        super.onDetachedFromWindow()
    }
}
```

---

## EMV Standards

### Q16: What is EMV and why is it important for mobile payments?
**Answer:**
**EMV (Europay, Mastercard, Visa)**:
- Global standard for smart card payments
- Ensures secure, standardized, tamper-resistant transactions
- Used in chip cards, POS terminals, NFC payments

**Mobile Integration:**
- Android apps with external card readers
- NFC contactless payments
- Host Card Emulation (HCE)
- Integration with POS device SDKs

**Importance**: Industry standard compliance, secure card-present transactions, fraud reduction.

### Q17: Describe the EMV transaction flow and its key steps.
**Answer:**
**10-Step EMV Transaction Flow:**
1. **Application Selection**: Choose payment app from card
2. **Initiate Processing**: Start session with card
3. **Read Application Data**: Get PAN, expiry, issuer keys
4. **Offline Data Authentication**: Verify card authenticity (SDA/DDA)
5. **Processing Restrictions**: Check card validity
6. **Cardholder Verification**: PIN, signature, or no CVM
7. **Terminal Risk Management**: Decide offline vs online
8. **Terminal Action Analysis**: Determine if authorization needed
9. **Online Processing**: Send request to bank if required
10. **Transaction Completion**: Finalize and update logs

**Developer Importance**: Understanding helps with debugging, analyzing transaction logs, and handling TLV data.

---

## Mobile Wallets

### Q18: Explain the difference between Secure Element (SE) and Host Card Emulation (HCE).
**Answer:**

**Secure Element (SE) - Apple Pay:**
- **Hardware**: Dedicated security chip in device
- **Storage**: Cards stored in secure hardware
- **Security**: Maximum security, tamper-resistant
- **Limitation**: Requires specific hardware support

**Host Card Emulation (HCE) - Google Pay:**
- **Software**: Emulation handled by OS
- **Storage**: Dynamic tokens, no hardware storage
- **Security**: OS-level protection, less secure on compromised devices
- **Advantage**: Faster development, no hardware dependency

### Q19: How does tokenization work in mobile wallets like Google Pay?
**Answer:**
**Process:**
1. **Token Generation**: Real card number replaced with unique token
2. **Cryptogram Creation**: Encrypted signature proves authenticity
3. **Transaction**: Token + cryptogram sent to merchant
4. **Verification**: Bank verifies token and signature
5. **Response**: Instant approval/decline

**Security Benefits:**
- No real card data transmitted
- Tokens are one-time use or limited use
- Device authentication required
- Additional biometric/PIN verification

---

## Device Security

### Q20: How do you implement root detection in a fintech app?
**Answer:**
**Detection Methods:**
1. **File System Checks**: Look for `/system/xbin/su`, root apps
2. **Permission Checks**: Unusual system permissions
3. **RootBeer Library**: Pre-built detection library
4. **Play Integrity API**: Google's device attestation
5. **Runtime Checks**: Test for Xposed, Magisk modules

**Implementation:**
```kotlin
// Multiple detection layers
fun isDeviceRooted(): Boolean {
    return checkSuBinary() || 
           checkRootApps() || 
           checkDangerousProps() ||
           checkTestKeys()
}
```

**Advanced Protection**: Code obfuscation, multiple check points, server-side verification.

### Q21: What is the Play Integrity API and when should you use it?
**Answer:**
**Play Integrity API** (replaces SafetyNet):
- Google's device and app integrity verification service
- Checks device security, app authenticity, and user account

**What it verifies:**
- **Device Integrity**: Not rooted, bootloader locked, genuine device
- **App Integrity**: Official app from Google Play Store
- **Account Integrity**: Legitimate Google account

**When to use:**
- Before user authentication
- Before processing payments
- Before storing sensitive data
- For fraud detection systems

**Implementation:**
```kotlin
val integrityManager = IntegrityManagerFactory.create(context)
val integrityTokenResponse = integrityManager.requestIntegrityToken(
    IntegrityTokenRequest.builder()
        .setNonce(nonce)
        .build()
)
```

### Q22: How do you protect against app tampering and repackaging?
**Answer:**
**App Signature Verification:**
1. **Runtime Checks**: Verify app signature matches original
2. **Certificate Pinning**: Ensure app certificate is authentic
3. **Code Obfuscation**: Make reverse engineering difficult
4. **Anti-Debugging**: Detect debugging attempts
5. **Integrity Checks**: Regular self-verification during runtime

**Implementation:**
```kotlin
fun verifyAppSignature(): Boolean {
    val expectedSignature = "your_expected_signature"
    val currentSignature = getCurrentAppSignature()
    return expectedSignature == currentSignature
}

// Check on app startup and periodically
if (!verifyAppSignature()) {
    // Block app functionality or report to server
}
```

---

## Compliance & Best Practices

### Q23: What are the key PCI DSS requirements for mobile applications?
**Answer:**
**Key Requirements:**
- **Requirement 3.2.1**: No storage of sensitive data after authorization
- **Requirement 3.4**: Protect stored cardholder data with encryption
- **Requirement 4**: Encrypt cardholder data transmission over networks
- **Requirement 8**: Assign unique user IDs and strong authentication
- **Requirement 11**: Regular security testing

**Mobile-Specific Implementations:**
- Use tokenization instead of storing card data
- Implement proper key management with Android Keystore
- Secure network communications with TLS
- Regular penetration testing and vulnerability assessments

### Q24: How do you implement defense in depth for a fintech mobile app?
**Answer:**
**Multiple Security Layers:**

1. **Network Layer**: TLS 1.3, certificate pinning, API authentication
2. **Application Layer**: Code obfuscation, anti-tampering, root detection
3. **Data Layer**: Encryption at rest, secure key management, tokenization
4. **Device Layer**: Device attestation, biometric authentication, secure storage
5. **User Layer**: Strong authentication, user education, fraud monitoring

**Example Implementation:**
```kotlin
// Layer 1: Network Security
implementTLS13WithCertPinning()

// Layer 2: App Security  
verifyAppIntegrity()
detectRootAndTampering()

// Layer 3: Data Security
encryptSensitiveData()
useAndroidKeystore()

// Layer 4: Device Security
performDeviceAttestation()
enableBiometricAuth()

// Layer 5: User Security
implementStrongAuth()
monitorForFraud()
```

### Q25: What are the best practices for secure key management in mobile fintech apps?
**Answer:**
**Key Management Best Practices:**

1. **Use Android Keystore**: Hardware-backed key storage
2. **Key Rotation**: Regular key updates and rotation
3. **Separate Keys**: Different keys for different purposes
4. **Key Derivation**: Derive keys from master key when needed
5. **No Hardcoding**: Never embed keys in source code
6. **Secure Generation**: Use secure random number generators
7. **Key Attestation**: Verify keys are hardware-backed

**Implementation:**
```kotlin
// Generate key in Android Keystore
val keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore")
val keyGenParameterSpec = KeyGenParameterSpec.Builder(
    "SecureKey",
    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
)
.setBlockModes(KeyProperties.BLOCK_MODE_GCM)
.setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
.setUserAuthenticationRequired(true)
.build()

keyGenerator.init(keyGenParameterSpec)
keyGenerator.generateKey()
```

---

## Scenario-Based Questions

### Q26: A user reports their payment was processed twice. How would you investigate this from a security perspective?
**Answer:**
**Investigation Steps:**
1. **Check Transaction Logs**: Look for duplicate transaction IDs
2. **Verify Nonce Usage**: Ensure unique nonce was used for each request
3. **API Replay Analysis**: Check if request was replayed maliciously
4. **Device Integrity**: Verify transaction came from legitimate device
5. **Network Analysis**: Check for MITM attacks or network issues
6. **User Behavior**: Analyze if user accidentally triggered duplicate payment

**Prevention Measures:**
- Implement idempotency keys
- Use unique transaction nonces
- Add request signing with timestamps
- Implement rate limiting

### Q27: How would you handle a scenario where your app detects it's running on a rooted device?
**Answer:**
**Response Strategy:**
1. **Risk Assessment**: Evaluate the level of root access
2. **Graceful Degradation**: Limit functionality rather than complete block
3. **User Communication**: Explain security risks clearly
4. **Alternative Options**: Offer secure alternatives (web app, in-branch)
5. **Continuous Monitoring**: Regular re-checks during app usage

**Implementation Options:**
```kotlin
when (detectRootLevel()) {
    ROOT_LEVEL_HIGH -> {
        // Block critical functions
        blockPayments()
        showSecurityWarning()
    }
    ROOT_LEVEL_MEDIUM -> {
        // Increased security measures
        requireBiometricAuth()
        enableAdditionalLogging()
    }
    ROOT_LEVEL_LOW -> {
        // Monitor and warn
        showWarningMessage()
        increaseMonitoring()
    }
}
```

### Q28: Your fintech app needs to integrate with a third-party payment processor. What security considerations would you implement?
**Answer:**
**Security Implementation:**
1. **API Security**: Mutual TLS, request signing, rate limiting
2. **Data Minimization**: Only send necessary data to third party
3. **Tokenization**: Use tokens instead of raw card data
4. **Audit Logging**: Log all interactions for compliance
5. **Error Handling**: Secure error messages, no data leakage
6. **Timeout Management**: Proper timeout and retry logic
7. **Certificate Validation**: Verify third-party certificates
8. **Compliance Check**: Ensure third party meets PCI DSS requirements

**Code Example:**
```kotlin
class SecurePaymentIntegration {
    fun processPayment(tokenizedData: TokenizedCardData) {
        val signedRequest = signRequest(tokenizedData)
        val response = secureApiCall(signedRequest)
        auditLog.log("Payment processed", response.transactionId)
    }
    
    private fun signRequest(data: Any): SignedRequest {
        val signature = generateHMAC(data, secretKey)
        return SignedRequest(data, signature, timestamp)
    }
}
```

---

## Technical Implementation Questions

### Q29: Write code to securely store and retrieve a user's authentication token.
**Answer:**
```kotlin
class SecureTokenManager(private val context: Context) {
    
    private val masterKey by lazy {
        MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build()
    }
    
    private val encryptedPrefs by lazy {
        EncryptedSharedPreferences.create(
            context,
            "secure_tokens",
            masterKey,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }
    
    fun storeToken(token: String) {
        encryptedPrefs.edit()
            .putString("auth_token", token)
            .apply()
    }
    
    fun getToken(): String? {
        return encryptedPrefs.getString("auth_token", null)
    }
    
    fun clearToken() {
        encryptedPrefs.edit()
            .remove("auth_token")
            .apply()
    }
}
```

### Q30: How would you implement a secure random number generator for cryptographic operations?
**Answer:**
```kotlin
class SecureRandomGenerator {
    
    private val secureRandom = SecureRandom()
    
    fun generateNonce(length: Int = 12): ByteArray {
        val nonce = ByteArray(length)
        secureRandom.nextBytes(nonce)
        return nonce
    }
    
    fun generateSalt(length: Int = 16): ByteArray {
        val salt = ByteArray(length)
        secureRandom.nextBytes(salt)
        return salt
    }
    
    fun generateSessionId(): String {
        val bytes = ByteArray(32)
        secureRandom.nextBytes(bytes)
        return Base64.encodeToString(bytes, Base64.URL_SAFE or Base64.NO_WRAP)
    }
    
    // For PIN generation
    fun generateSecurePin(length: Int = 4): String {
        val digits = StringBuilder()
        repeat(length) {
            digits.append(secureRandom.nextInt(10))
        }
        return digits.toString()
    }
}
```

---

## Summary

These interview questions cover the critical aspects of fintech mobile security:

**Core Areas Covered:**
- Mobile security threats and mitigation
- Cryptographic implementations (AES, RSA, HMAC)
- Payment security (EMV, tokenization, DUKPT)
- Memory and data protection
- Device integrity and attestation
- Network security (SSL/TLS, certificate pinning)
- Compliance requirements (PCI DSS)
- Best practices for secure development

**Key Takeaways for Interviews:**
1. **Understand the "why"** behind each security measure
2. **Know practical implementations** with code examples
3. **Explain trade-offs** between security and usability
4. **Demonstrate compliance knowledge** (PCI DSS, EMV)
5. **Show awareness of evolving threats** and countermeasures

**Preparation Tips:**
- Practice explaining complex concepts simply
- Prepare real-world scenarios and solutions
- Understand the business impact of security decisions
- Stay updated with latest security threats and solutions
- Know the regulatory landscape for fintech applications