# SD-JWT Implementation in Kotlin

This is a Kotlin implementation of the [Selective Disclosure for JWTs](https://github.com/oauthstuff/draft-selective-disclosure-jwt)
spec using the [Nimbus JOSE + JWT](https://connect2id.com/products/nimbus-jose-jwt) 
library.

Up to date with draft version: **02** 

## Checking Out the Implementation

In the [SdJwtKtTest.kt](src/test/kotlin/org/sd_jwt/SdJwtKtTest.kt) file 
there are examples that show how the library can be used
on the issuance, wallet and verifier side.

### Running the Examples

If you have Docker installed you can simply run:

1. ``docker build -t sd-jwt .``
2. ``docker run -it --rm sd-jwt``

## Import into Gradle Project

*build.gradle*
```groovy
plugins {
    /* ... */
    id 'org.jetbrains.kotlin.plugin.serialization' version '1.7.10'
}

dependencies {
    /* ... */
    implementation 'org.sd-jwt:sd-jwt-kotlin:0.0.0'

    // https://mvnrepository.com/artifact/com.nimbusds/nimbus-jose-jwt
    implementation("com.nimbusds:nimbus-jose-jwt:9.24.3")
    // For ED25519 key pairs
    implementation("com.google.crypto.tink:tink:1.7.0")

    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.4.0")
}
```

## Library Usage

### Initialization

First you need to define your credential as a 
[kotlinx serializable](https://github.com/Kotlin/kotlinx.serialization) 
data class.

```kotlin
@Serializable
data class SimpleTestCredential(
    @SerialName("given_name") val givenName: String? = null,
    @SerialName("family_name") val familyName: String? = null,
    val email: String? = null,
    val b: Boolean? = null,
    val age: Int? = null
)
```

Then you need a few variables to get started.

```kotlin
val verifier = "http://verifier.example.com"
val issuer = "http://issuer.example.com"

val issuerKeyJson = """{"kty":"OKP","d":"Pp1foKt6rJAvx0igrBEfOgrT0dgMVQDHmgJZbm2h518","crv":"Ed25519","kid":"IssuerKey","x":"1NYF4EFS2Ov9hqt35fVt2J-dktLV29hs8UFjxbOXnho"}"""
val issuerKey = OctetKeyPair.parse(issuerKeyJson)

val trustedIssuers = mutableMapOf<String, String>(issuer to issuerKey.toPublicJWK().toJSONString())
```

### Issuer Creating the Credential

```kotlin
val claims = SimpleTestCredential("Alice", "Wonderland", "alice@example.com", false, 21)
val credential = createCredential(claims, null, issuer, issuerKey)
```

### Wallet Creating the Presentation

```kotlin
val releaseClaims = SimpleTestCredential(givenName = "",  email = "", age = 0) // Non-null claims will be revealed
val presentation = createPresentation(credential, releaseClaims, verifier, "12345", null)
```

### Verifier Parsing and Verifying the Credential

```kotlin
val verifiedSimpleTestCredential = verifyPresentation<SimpleTestCredential>(presentation, trustedIssuers, "12345", verifier)
```
