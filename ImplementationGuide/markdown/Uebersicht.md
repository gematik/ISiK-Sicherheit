# ISiK-Connect

Konnektivität und Sicherheit haben im Zusammenhang mit einem interoperablen Datenaustausch in und mit Krankenhäusern viele Facetten: Nutzer müssen authentifiziert sein, Berechtigungen müssen definiert und durchgesetzt werden, Daten müssen gegen Verfälschung geschützt und verfügbar sein, Datenänderungen müssen nachvollziehbar sein etc. Hierzu können Technologien wie z. B. [_SAML_](https://saml.xml.org/saml-specifications) oder [_UMA_](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html) genutzt werden. Diese Technologien basieren auf einem etablierten und erprobten Stack aus Standards wie [_ReST Representational State Transfer_](https://restfulapi.net/), [_OAuth2_](https://oauth.net/2/) oder , [_OpenID Connect_](https://openid.net/developers/specs/). Diese Standards sind domänenunabhängig spezifiziert und müssen daher für den Einsatz im deutschen Gesundheitswesen und ggf. auch für das Zusammenspiel mit dem HL7 FHIR-Standard profiliert werden. FHIR bietet hierzu unterstützende Ressourcen wie z. B. [_Consent_](https://hl7.org/fhir/R4/consent.html), [_AuditEvent_](https://hl7.org/fhir/R4/auditevent.html) oder [_CompartmentDefinition_](https://hl7.org/fhir/R4/compartmentdefinition.html) an, die eine Bindung zwischen FHIR und dedizierten Sicherheitsstandards herstellen können. Im Fall von ISiK sind zusätzlich im deutschen Gesundheitswesen bereits definierte Bausteine der Gesundheitstelematik wie z. B. sektorale Identitätsdienste oder Konnektoren/TI-Gateways zu berücksichtigen, die idealerweise für die Authentifizierung an Patienten- und Zuweiserportalen, dem Schutz von Gesundheitsdaten im Krankenhaus, einem Ende-zu-Ende gesicherten Datenaustausch mit Niedergelassenen und anderen potenziell ISiK-relevanten Themen genutzt werden sollen.


**Autorisierung von Zugriffen auf FHIR Ressourcen**

Ziel der vorliegenden Spezifikation ist es, per FHIR RESTful API formulierte Anfragen an einen Ressourcenserver unter Nutzung eines zuverlässigen und sicheren Autorisierungsprotokolls - und damit unter Berücksichtigung von vergebenen Berechtigungen und formulierten Sicherheitsmechanismen - zu beantworten. Hierzu werden Teile des von _SMART Health IT_ und _Boston Childrens Hospital_ auf Basis des OAuth2-Standards definierten, von HL7 als Teil des FHIR-Standards übernommenen [_'SMART on FHIR' API_](https://smarthealthit.org/smart-on-fhir-api/) profiliert, so dass diese im Kontext der bestehenden ISiK-Festlegung nutzbar sind. Das SMART-on-FHIR-API soll die Weiterentwicklung von _Electronic Health Records_ (EHR) zu Plattformen befördern, bei denen - analog zu mobilen Plattformen wie iOS oder Android - Anwendungen aus einem _Store_ in eine solche Plattform eingebracht und dort in einer sicheren Umgebung ausgeführt werden (_"SMART App Launch"_). Das Ziel von ISiK ist es an dieser Stelle die Einbindung dieser Drittsysteme standardisiert zu ermöglichen, sodass nicht für jedes bestätigungsrelevante System eine eigene Form der Autorisierung implementiert werden muss.

### SMART on FHIR 

Technische Systeme können in einer SMART-on-FHIR-konformen Umsetzung eines Autorisierungssystems eine oder mehrere der folgenden Rollen einnehmen:
* Ressourcenserver: IT-Systeme stellen in dieser Rolle geschützte Ressourcen über FHIR-RESTful-Endpunkte bereit oder nehmen geschützte Ressourcen über FHIR-RESTful-Endpunkte entgegen.
* Client: IT-Systeme in dieser Rolle fragen geschützte Ressourcen von Ressourcenservern ab oder senden geschützte Ressourcen an Ressourcenserver.  
* Autorisierungsserver: IT-Systeme in dieser Rolle stellen für authentifizierte Nutzer Sicherheitstoken aus, die einen Zugang zu auf Ressourcenservern verwalteten FHIR Ressourcen autorisieren.
* App-Launcher: IT-Systeme in dieser Rolle können einen Client aus ihrem eigenen Sicherheitskontext initialisieren oder in einen neu aufgesetzten Kontext stellen. Der Kontext kann z. B. der aktuell ausgewählte Patient oder Behandlungsfall sein.

Ein IT-System im Krankenhaus kann mehrere der genannten Rollen einnehmen. Beispielsweise kann ein KIS selbst Ressourcen bereitstellen ("Ressourcenserver"), Ressourcen von spezialisierten Subsystemen abrufen ("Client"), die Autorisierung von Nutzern durchführen ("Autorisierungsserver") und weitere Anwendungen aus dem aktuellen Nutzer- und/oder Patientenkontext heraus starten ("App-Launcher"). Durch eine solche Integration verschiedener Rollen kann ein KIS einen _Electronic Health Record_ im Sinne von _SMART on FHIR_ darstellen und damit als Plattform für wiederverwendbare, austauschbare und dynamisch startbare Anwendungen (_SMART Apps_) fungieren. 

### SMART App Launch

ISiK-Connect soll durch eine wie oben skizzierte Integration verschiedener Systemrollen eine Umsetzung des als Teil des SMART-on-FHIR-API definierten [HL7 Implementierungsleitfadens _Smart App Launch_](https://hl7.org/fhir/smart-app-launch/STU2.1/index.html) ermöglichen. Für ISiK-Connect werden jedoch keine normativen Festlegungen getroffen, die andere Profile auf dem _OAuth2_-Standard wie z. B. _IHE IUA_ grundsätzlich ausschließen. Dies soll es Krankenhäusern erlauben, für die Absicherung gegen externe Zugriffe (z. B. ausgehend von einem Patientenportal) und für die Autorisierung interner Zugriffe (z. B. zur Implementierung der OH KIS) unterschiedliche Standardprodukte einzusetzen, die jeweils auf die spezifischen Anforderungen zugeschnitten sind. 

Ziel des _Smart App Launch_ ist es, ein Zugriffstoken von einem OAuth2-kompatiblen Autorisierungsserver zu erhalten, mittels dessen ein Client eine FHIR-RESTful-Interaktion gegen einen Ressourcenserver durchführen kann. Dies erfolgt unter Berücksichtigung der vorab festgelegten Zugriffsrechte der Benutzer, die sich z. B. aus deren Rollen oder anderen Nutzerattributen ableiten können. Um ein Zugriffstoken zu erhalten und mit diesem als Autorisierungsnachweis auf einen Ressourcenserver zuzugreifen, sieht der _Smart App Launch_ die in der nachfolgenden Tabelle aufgeführten Schritte vor. Die Kreuze geben an, welche der oben beschriebenen logischen Bausteine an dem jeweiligen Schritt beteiligt sind.

| # | Schritt                                        | Ressourcenserver | Client| Autorisierungsserver | App-Launcher |
|:-:|:-----------------------------------------------|:--------------------:|:----------:|:--------------------:|:-----------------:|
| 1 | Aufsetzen Vertrauensbeziehungen                |     x                 |      X     |      X         |                   |
| 2 | Starten des Clients aus/in einen Launch-Kontext|                      |      X     |           (X)        |         X         |
| 3 | Abruf der SMART-Konfiguration                  |            X         |      X     |           |                   |
| 4a | Autorisierungsanfrage des Clients              |                      |      X     |            X         |                   |
| 4b | Prüfung Anfrage und Ausgabe Autorisierungscode |                      |      X     |            X         |                   |
| 5 | Austausch des Codes gegen ein Zugriffstoken    |                      |      X     |            X         |                   |
| 6 | Autorisierte ReST-Interaktion                  |            X         |      X     |           (X)        |                   |
|(7)| Neu-Ausstellen eines Zugriffstokens            |                      |      X     |            X         |                   |

Eine Übersicht des zusammenhängenden _SMART App Launch_ ist dem Abschnitt [SMART App Launch - 2.1.3 - SMART authorization & FHIR access: overview](https://hl7.org/fhir/smart-app-launch/STU2.1/app-launch.html#smart-authorization--fhir-access-overview) des SMART-on-FHIR-Standards zu entnehmen. Darauf aufbauende informative Hinweise zur Umsetzung des vollständigen _SMART App Launch_ im Krankenhaus sind auf der Seite ["SMART on FHIR"](ISiKundSMART.md) zusammengefasst-


## Vorgaben zur Autorisierung

Wie aus der Tabelle zu ersehen, betrifft das Gros der SMART-on-FHIR-Spezifikation das Zusammenspiel der Systeme in den Rollen des Clients und des Autorisierungsservers. Dieses wird zusätzlich stark geprägt durch den Kontext aus welchem bzw. in welchen der Client - d. h. die wiederverwendbare, austauschbare, modulare Anwendung - dynamisch gestartet wird:

- [SMART App Launch - EHR Launch](https://hl7.org/fhir/smart-app-launch/STU2.1/app-launch.html#step-2-launch-ehr):
Ein Client kann aus dem Kontext eines anderen Systems (das dann in der Rolle des App-Launchers agiert) direkt innerhalb einer bestehenden _User Session_ gestartet werden, beispielsweise indem der eingeloggte Benutzer den Client startet und dieser durch das System in einer neuen Browserinstanz oder einem eingebetteten _iframe_ geöffnet wird.

- [SMART App Launch - Standalone Launch](https://hl7.org/fhir/smart-app-launch/STU2.1/app-launch.html#launch-app-standalone-launch):
Clients, welche außerhalb eines anderen Systems gestartet werden (z.B. Mobile Apps welche Daten von einem ISiK-Ressourcenserver abfragen möchten), müssen ihren Kontext explizit über eine Nutzerinteraktion gegen eine als ISiK-App-Launcher fungierende Komponente aufbauen. Dieses kann beispielsweise ein Widget zur Auswahl des zu aktivierenden Patientenkontextes sein.

Beide Launch-Sequenzen bedingen eine weitreichende Integration von Autorisierungsserver und App-Laucher, die gemeinsam den Launch-Kontext des Clients bestimmen und verwalten. Dies Verknüpfung der beiden Komponenten ist der Spezifikation inhärent. Im Sinne der originalen SMART-on-FHIR-Spezifikation verhalten sich Autorisierungsserver und App-Launcher als eine Einheit. Entsprechend werde diese zusammen im Bestätigungsverfahren zertifiziert. Es sei darauf hingewiesen, dass dies explizit nur für diese Kombination gilt. Bestehende IAM-Subsysteme, für die Verwaltung von Benutzeridentitäten und Zugriffsrechte, sind hiervon nicht betroffen. Benutzeridentitäten können über OpenID Connect in den Autorisierungsfluss eingebunden werden. Die Entscheidung über die Zugriffsrechte entscheidetet ein Policy-Decision-Point in Kombination mit dem Ressourcenserver.

Diese in ISiK vorgenommene Profilierung des SMART-on-FHIR-API fokussiert auf die Absicherung der in anderen ISik-Modulen spezifizierten FHIR-Ressourcen wie z. B. _Patient_, _Encounter_ oder _Observation_. Hierbei macht die Spezifikation den Herstellern und Krankenhäusern hierbei aber weder Vorgaben zu konkreten Berechtigungsmodellen/-regeln oder dem Zuschnitt der zum Identitäts- und Berechtigungsmanagement (IAM) einzusetzenden Produkte.

