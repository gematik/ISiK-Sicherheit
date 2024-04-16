# ISiK Connect und SMART on FHIR

ISiK-Connect setzt auf dem HL7-Standard _SMART on FHIR_ auf, der u. a. das Zusammenspiel der Standards FHIR und OAuth2 für die Weitergabe von Zugriffskontexten zum sicheren Zugriff auf FHIR-Ressourcen beschreibt. Wie in der [Motivation](Motivation.md) beschrieben, umfassen die Vorgaben von ISiK-Connect jedoch nur eine Untermenge des SMART-on-FHIR-API. 

Ergänzend zu den normativen Abschnitten [ISiK-Connect: Autorisierung](ISiKAutorisierung.md) und [ISiK-Konformität](Conformance.md) wird in diesem und den untergeordneten Abschnittem skizziert, wie eine vollständige Umsetzung des SMART-on-FHIR-Standards im Kontext von ISiK aussehen kann.

## _SMART App Launch_
Die auf den nachgeordneten Seiten beschriebene Implementierungsleitfaden dient zur Erläuterung des Ablaufs eines _Smart App Launch_ (siehe [Übersicht](Uebersicht.md)). Ziel des _Smart App Launch_ ist es, ein Zugangstoken von einem OAuth2-kompatiblen Autorisierungsserver zu erhalten, mittels dessen eine FHIR RESTful API-Interaktion durchgeführt werden kann. Dies erfolgt unter Berücksichtigung der Zugriffsrechte der Benutzer in dem den _SMART App Launch_ auslösenden System (KIS, Portal), das im Folgenden analog zur SMART-on-FHIR-Spezifikation als 'EHR' (_Electronic Health Record_) bezeichnet wird. 

Um ein Zugangstoken für den Zugriff auf einen Ressourcenserver zu erhalten, sind folgende sechs Schritte notwendig, die auf den Unterseiten zu dieser Seite jeweils im Detail beschrieben sind:

* Registrierung eines _SMART Clients_ mit dem Autorisierungsserver
* Der _SMART Client_ fragt den Autorisierungsserver um Autorisierung an
* Der Autorisierungsserver evaluiert die Autorisierungsanfrage und initiiert ggf, die Authentifizierung des (menschlichen) Nutzers
* Austausch des vom EHR an den Client ausgegebenen Autorisierungscodes gegen ein Zugangstoken.
* Ausführung einer durch das Zugangstoken abgesicherten FHIR Restful Interaktion am Ressourcenserver
* Ausstellung eines "Refresh"-Zugangstoken.

Eine Übersicht des zusammenhängenden _SMART App Launch_ ist dem Abschnitt [SMART App Launch - 2.0.3 - SMART authorization & FHIR access: overview](https://hl7.org/fhir/smart-app-launch/STU2.1/app-launch.html#smart-authorization--fhir-access-overview) der SMART-on-FHIR-Spezifikation zu entnehmen.

