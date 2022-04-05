# Schritt 2: App bittet um Autorisierung

Im nachfolgenden Schritt wird durch den Client an dem "authorize"-Endpunkt des Autorisierungsservers ein Autorisierungscode angefragt, welcher innerhalb eines Authorization Code Flows (vgl. [RFC6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1)) durch ein OAuth 2.0 Access Token ersetzt werden kann.

Abhängig davon ob der Client durch einen EHR Launch oder Standalone Launch gestartet wird, existieren unterschiedliche Schritte für die Anfrage eines Autorisierungscodes.

----

## EHR Launch

Das bestätigungsrelevante System MUSS in der Lage sein, den Client durch einen externen Kontextaufruf zu starten. Im Kontext der derzeitig eingeloggten Benutzer:in wird der Client gestartet, vgl. {{pagelink:SmartAppLaunch, text:EHR Launch / Standalone Launch}}. Der Aufruf des Clients MUSS alle in [SMART App Launch - 2.0.7 - Launch App: EHR Launch](https://hl7.org/fhir/smart-app-launch/STU2/app-launch.html#step-2-launch-ehr) dokumentierten Parameter enthalten.

Es sei darauf hingewiesen, dass jeder EHR Launch mit einem eindeutigen Launch Parameter zu assoziieren ist. Mit diesem beliebig gewählten Parameter (z.B. eine UUID) kann der Client das Access Token mit dem Kontext, aus dem der Client gestartet worden ist, verknüpfen. Der Kontext kann beispielsweise Informationen zum Patienten oder Kontakt/Fall enthalten, welcher dem Anwender zuvor präsentiert wurde. Dieser Kontext wird dem Client durch sogenannte [Launch Context Claims](https://hl7.org/fhir/smart-app-launch/STU2/scopes-and-launch-context.html#scopes-for-requesting-context-data) vermittelt. Diese Claims enthalten IDs der FHIR-Ressourcen welche die zuvor genannten Datenobjekte repräsentieren. Es ist notwendig innerhalb der SMART authorization sequence die angeforderten [Launch Context Claims](https://hl7.org/fhir/smart-app-launch/STU2/scopes-and-launch-context.html#scopes-for-requesting-context-data) an den Client zurückzugeben, vgl. Abschnitt {{pagelink:Schritt4AustauschAutorisierungscodesZugangstoken, text:Austausch des Autorisierungscodes für ein Zugangstoken}}. Der Client kann spezifische Kontextparameter anfragen. Sollten diese jedoch nicht verfügbar sein (z.B. der Client wurde ohne Patientenkontext aufgerufen), können die zurückgegebenen Launch Context Claims von den gewünschten Scopes abweichen.

----

## Standalone Launch Sequence

Aufgrund des fehlenden Kontexts zwischen Client und dem bestätigungsrelevanten System KANN der Client durch Angabe von gewünschten Smart Launch Scopes bestimmen, welche Details durch den Autorisierungsserver in der Access Token Response bereitgestellt werden MÜSSEN. Beispielsweise kann, äquivalent zum EHR Launch, der Patienten und/oder Kontakt/Fall Kontext angefordert werden. Genaue Details für die Syntax der Launch Context Scope finden sich im Kapitel {{pagelink:ScopesAndLaunchContext, text:SMART on FHIR Launch Context Scope Syntax}}.

----

## Parameter Autorisierungsanfrage

Der Aufruf des Clients MUSS alle in [SMART App Launch - 2.0.9 - Obtain authorization code](https://hl7.org/fhir/smart-app-launch/STU2/app-launch.html#step-4-authorization-code) dokumentierten Parameter enthalten. Insbesondere gilt dies für die Unterstützung von HTTP POST-basierten Autorisierungsanfragen und der Unterstützung von [Proof Key for Code Exchange by OAuth Public Clients](https://datatracker.ietf.org/doc/html/rfc7636).

Zu beachten ist, dass durch den SMART App Launch der "state"-Parameter abweichend von RFC6749 verpflichtend ist.

----

## TLS

Alle Autorisierungsendpunkte MÜSSEN per HTTPS (TLS-Verschlüsselung) erreichbar sein. Im Echtbetrieb MUSS die Kommunikation ausschließlich per HTTPS erfolgen. Vorgaben zur einzusetzenden TLS Version, siehe [Sicherheitsaspekte](https://simplifier.net/guide/ImplementierungsleitfadenISiK-Basismodul/UebergreifendeFestlegungenRest).

----

## Abruf SMART Configuration

Als Einstiegspunkt für einen Standalone Launch MUSS dem Client die Url des FHIR Endpunktes bekannt gegeben werden. Anschließend erfolgt eine Abfrage des ".well-known/smart-configuration" Dokumentes, welche durch den FHIR Endpunkt bereitgestellt werden MUSS. Vorgaben zum Format sind dem Abschnitt {{pagelink:Conformance, text:Smart Configuration}} zu entnehmen. Hierdurch erhält der Client dynamisch die Adresse des Autorisierungsservers inkl. "authorize" und "token" Endpunkt. Falls durch das bestätigungsrelevante System OAuth 2.0 Endpunkte optional über das CapabilityStatement des FHIR-Endpunktes zur Verfügung gestellt werden, MÜSSEN diese Inhalte identisch zu den Inhalten des ".well-known/smart-configuration" Dokumentes sein.

----

## Hinweise zu Identity Scopes

Um Informationen über die authentifizierten EndbenutzerIn zu erhalten, kann ein Client per OpenID Connect ein Identitätstoken zusammen mit einem Zugangstoken anfragen. Hierzu sind in Kombination die Scopes "openid" und "fhirUser" zu verwenden. Zu bestätigende Systeme MÜSSEN die Vorgaben nach [2.0.4 - Scopes for requesting identity data](https://hl7.org/fhir/smart-app-launch/STU2/scopes-and-launch-context.html#scopes-for-requesting-identity-data) umsetzen. Anstatt dem Claim "fhirUser" kann - aus Gründen der Rückwärtskompatibilität - auch der Claim "profile" verwendet werden. Die Unterstützung des "profile" claims ist optional. Die Vorgaben zur Umsetzung der OpenID Connect Core Spezifikation zur Erzeugung eines ID-Tokens sind bestätigungsrelevant.

----

## Hinweise zu Access Scopes

Innerhalb des Scope Parameters welcher als Teil der Autorisierungsanfrage versendet wird, kann der Client dem Server mitteilen, welche Scopes zur korrekten Ausführung notwendig sind, d.h. welche FHIR-Restful Interaktionen vom Client benötigt werden. Diese Scopes repräsentieren die Menge aller Scopes welche durch den Client gewünscht werden, jedoch nicht notwendigerweise durch den Server unterstüzt und/oder erlaubt werden. Es steht dem Autorisierungsserver frei diese Scopes einzuschränken, falls der Client für die Anforderung der Scopes nicht berechtigt ist. Weitere Details zur Syntax der Access Scopes siehe {{pagelink:ScopesAndLaunchContext, text:SMART on FHIR Access Scope Syntax}}.

Hieraus folgt, dass die angeforderten Scopes nur die Kategorien an Rechten repräsentieren, die an den anfragenden Client delegiert werden dürfen. Somit können generell bestimmte Rechte ausgeschlossen werden. Eine Anfrage an den FHIR-REST-API-Endpunkt kann jedoch trotz validem Token mit den spezifizierten Scopes abgelehnt werden, falls während der Evaluierung der Anfrage seitens des Servers festgestellt wird, dass die BenutzerIn die gewünschten Ressourcen nicht verarbeiten darf.

Der Autorisierungsserver MUSS eine Konfiguration der erlaubten Scopes pro Client unterstützen.

----

## Beispiel

POST /authorize HTTP/1.1<br>
Content-Type: application/x-www-form-urlencoded<br>
Host: server.example.com

```
response_type=code&
client_id=TestClientId&
redirect_uri=https://example.org/redirect_uri/fhir/client/exampleId&
scope=user/*.rs openid fhirUser&
state=df01f5f8-5bf2-45ea-ab7a-706361da0515&
aud=http://example.org/fhir/&
code_challenge=2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b&
code_challenge_method=S256
```
