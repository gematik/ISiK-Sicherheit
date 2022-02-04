# Schritt 4: Austausch des Autorisierungscodes für ein Zugangstoken

Sowohl für Public als auch Confidential Clients erfolgt durch den SMART App Launch ein Authorization Code Flow. Dieser Ablauf wird verwendet um die in [Implicit Grant - OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-09#section-2.1.2) dokumentierten Sicherheitsrisiken zu minimieren. Im folgenden Schritt wird somit mittels einer vom Client initiierten Anfrage der Authorization Code durch ein Access Token ausgetauscht.

----

## Authentifizierung der Clients

Confidential Clients müssen sich gegenüber dem "Token"-Endpunkt des Autorisierungsservers authentifizieren. Hierfür MÜSSEN folgende Möglichkeiten unterstützt werden:

### JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants (Empfehlung)

Die präferierte Variante für die Authentifizierung des Clients erfolgt per [RFC 7523 - JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants](https://datatracker.ietf.org/doc/html/rfc7523). Hierzu MÜSSEN folgende Schritte untersützt werden:

1. Der Client generiert ein Private/Public Key Pair. Hierzu kann entweder [RSA (vgl. RFC8017 - PKCS #1: RSA Cryptography Specifications Version 2.2)](https://datatracker.ietf.org/doc/html/rfc8017) oder [ECDSA (vgl. RFC6979 - Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA))](https://datatracker.ietf.org/doc/html/rfc6979) als Algorithmus verwendet werden. Der öffentliche Schlüsselteil wird dem Autorisierungsserver als [JSON Web Key](https://datatracker.ietf.org/doc/html/rfc7517) eingebettet in einem [JSON Web Key Set](https://datatracker.ietf.org/doc/html/rfc7517#section-5) übermittelt. 

Folgende Anforderungen werden an den JSON Web Key gestellt:

- Als Signaturalgorithmus MUSS durch den Autorisierungsserver mindestens RS256, ES256, RS384, sowie ES384 akzeptiert werden, [siehe RFC7518 - "alg" (Algorithm) Header Parameter Values for JWS](https://datatracker.ietf.org/doc/html/rfc7518#section-3.1). Weitere Signaturalgorithmen KÖNNEN unterstützt werden.

- Der JSON Web Key MUSS mindestens folgende Parameter enthalten:
    - ["kty" (Key Type) Parameter](https://datatracker.ietf.org/doc/html/rfc7517#section-4.1)
    - ["kid" (Key ID) Parameter](https://datatracker.ietf.org/doc/html/rfc7517#section-4.5)
    - Für einen öffentlichen RSA Web Key: "n" (Modulus) Parameter, "e" (Exponent) Parameter (Siehe [RFC7518 - 6.3 Parameters for RSA Keys](https://datatracker.ietf.org/doc/html/rfc7518#section-6.3))
    - Für einen öffentlichen Elliptic Curve Web Key: "crv" (Curve) Parameter, "x" (X Coordinate) Parameter, "y" (Y Coordinate) Parameter (Siehe [RFC7518 - 6.2 Parameters for RSA Keys](https://datatracker.ietf.org/doc/html/rfc7518#section-6.2))

2. Ein Austausch des JSON Web Key MUSS durch eine der nachfolgenden Optionen implementiert werden. Option 1) wird aufgrund der in [SMART App Launch - 5 - Client Authentication: Asymmetric (public key)](https://hl7.org/fhir/smart-app-launch/STU2/client-confidential-asymmetric.html#client-authentication-asymmetric-public-key) aufgeführten Vorteile empfohlen.

(1) Austausch einer TLS-abgesicherten URL über die das oben genannte JSON Web Key Set abgerufen werden kann. Der Autorisierungsserver SOLLTE prüfen, dass diese URL mit dem ["jku" Parameter](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1.2) der Signatur des für die Authentifizierung des Clients verwendeten JSON Web Token übereinstimmt.

(2) Das JSON Web Key Set KANN dem Autorisierungsserver direkt mitgeteilt werden. In diesem Fall SOLLTE das JSON Web Key Set mindestens zwei Schlüssel enthalten, sodass eine unterbrechungsfreie Schlüsselrotation durchgeführt werden kann. Nachteile dieser Option sind in [SMART App Launch - 5.0.4 - Registering a SMART Backend Service (communicating public keys)](https://hl7.org/fhir/smart-app-launch/STU2/client-confidential-asymmetric.html#registering-a-client-communicting-public-keys) zusammengefasst.

Die verwendeten JSON Web Keys SOLLTEN regelmäßig gewechselt werden, um einem Schlüsselmissbrauch vorzubeugen.

3. Der Client erzeugt ein JSON Web Token entsprechend der in [SMART App Launch - 5.0.5 - Authenticating to the Token endpoint](https://hl7.org/fhir/smart-app-launch/STU2/client-confidential-asymmetric.html#authenticating-to-the-token-endpoint) definierten Vorgaben und verwendet dies als "client_assertion".

### HTTP Basic authentication:

Der Client tauscht während der Registrierung (siehe {{pagelink:Schritt1RegistierungClient, text:Schritt 1 - Registrierung einer SMART App mit dem bestätigungsrelevanten System}}) ein Client Secret mit dem bestätigungsrelevanten System aus. Eine Authentifizierung des Clients erfolgt per [RFC7617 - The 'Basic' HTTP Authentication Scheme](https://datatracker.ietf.org/doc/html/rfc7617). Als "username" MUSS die Client Id verwendet werden. Das Passwort MUSS das vorher ausgetauschte Client Secret verwendet werden.

----

## Austausch des Autorisierungscodes für ein Zugangstoken

In Abschnitt [SMART App Launch - 2.0.10 - Obtain access token](https://hl7.org/fhir/smart-app-launch/STU2/app-launch.html#obtain-access-token) werden alle notwendigen Parameter definiert durch die der Client mittels einer HTTP POST Anfrage (application/x-www-form-urlencoded kodiert) am token-Endpunkt des Autorisierungsservers ein Zugangstoken erhalten kann. Zu beachten ist, dass neben den in der Kernspezifikation gekennzeichneten Pflichtparametern, die Parameter "id_token" und "refresh_token" unterstützt werden MÜSSEN. Ein id_token MUSS ausgestellt werden, wenn der Client einen "openid fhirUser" Scope anfragt. Durch den Client angefragte Launch Context Claims MÜSSEN zurückgegeben werden. Eine Ausnahme ergibt sich durch den Fall, dass der Kontext im bestätigungsrelevanten System nicht vorliegt (z.B. es besteht kein Fall/Patientenkontext). 

Alle verpflichtenden Implementierungsdetails aus [SMART App Launch - 2.0.10 - Obtain access token](https://hl7.org/fhir/smart-app-launch/STU2/app-launch.html#obtain-access-token) MÜSSEN durch den Autorisierungsserver unterstützt werden.

Es sei explizit darauf hingewiesen, dass sowohl die SMART App Launch Spezifikation, als auch der vorliegende Implementierungsleitfaden keine Vorgaben bezüglich der Struktur oder des Inhalts des Zugangstokens enthalten. Die Verwendung eines Referenztokens wird empfohlen um ein Token Revocation Mechanismus effizient implementieren zu können. Siehe {{pagelink:Schritt6RefreshToken, text:Schritt 6: Refresh Token & Revocation}}.

----

## Beispiel