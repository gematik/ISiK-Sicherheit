# Release Notes

Im Rahmen der ISiK-Veröffentlichungen wird das [Semantic Versioning](https://semver.org/lang/de/) verwendet.

Die erste Ziffer X bezeichnet ein Major-Release und regelt die Gültigkeit von Releases. Die dritte Ziffer Y (Release x.0.y) bezeichnet eine technische Korrektur und versioniert kleinere Änderungen (Packages) während eines Jahres, z. B. 1.0.1.

Version: 3.0.0-rc4

Datum: 29.05.2023

* Die über die Interaktion mit dem Ressourcenserver hinausgehenden Teile von ISiK-Sicherheit aus ISiK Stufe 2 wurden als informative Erläuterungen wieder hinzugefügt. Hersteller oder Krankenhäuser, die SMART on FHIR vollständig umsetzen wollen, erhalten so Hinweise, wie dieses kongruet zu den sonstigen ISiK-Vorgaben möglich ist.
* Die Hinweise zu einem "break-the-glass" Szenario wurden gestrichen, da hier die regulativen Rahmenbedingungen in Deutschland deutlich von den Annahmen in den referenzierten, auf das amerikanische Gesundheitssystem fokussierenden Quellen abweichen.
* Hinweis auf den Aufrufparameter ISS_IDP hinzugefügt, der für die Unterstützung der sektoralen IdP relevant ist.

Version: 3.0.0-rc3

Datum: 25.04.2023

* ReleaseNotes added


Version: 3.0.0-rc2

Datum: 24.04.2023

Wesentliche Änderungen gegenüber der Version 2.0.1 sind in folgendem Pull-Request erfolgt:
* Jca_smart_light by @jcaumann in https://github.com/gematik/spec-ISiK-Sicherheit/pull/46

Die Änderungen sind folgende:
*	Für ISIK Stufe 2 waren insbesondere EHR/KIS als bestätigungsrelevante Systeme definiert; in ISiK Stufe 3 sind nur noch Ressourcenserver für ISiK Security bestätigungsrelevant. Die Motivationen für diese stärkere Fokussierung sind die dadurch konsistentere Fortschreibung von ISiK-Basis sowie die fehlenden Rückmeldungen aus dem ISiK-2-Kommentierungsverfahren zu dem ISiK-2 zugrunde liegenden Plattformkonzept von SMART on FHIR. 
* ISiK Security in ISiK Stufe 3 stellt einen Bezug zwischen SMART Kontexten und FHIR CompartmentDefinition her. Ressourcenserver müssen die Zugehörigkeit einer angefragten Ressource zu einem Kontext durch Abgleich mit der passenden CompartmentDefinition sicherstellen. 
*	Die verpflichtenden Vorgaben zu den Inhalten der SMART Configuration wurden in ISiK Security in ISiK Stufe 3 auf die Elemente reduziert, die unmittelbar aus dem OAuth2-Standard abgeleitete Informationen betreffen und für einen interoperablen Austausch von Informationen zu den an den Client delegierten Berechtigungen erforderlich sind. Allein für einen SMART App Launch relevante Elemente sind zunächst lediglich optional.
*	Für die Kodierung der Scopes wurde in ISiK Stufe 2 die Unterstützung der SMART on FHIR Syntax in den Versionen 1 und 2 verpflichtend vorgegeben. In Stufe 3 ist nur noch die v2-Syntax verpflichtend. Hiermit sollen Deployments vereinfacht werden, bei denen die Durchsetzung der (delegierten)     Berechtigungen über ein vorgelagertes API-Gateway bzw.- einen Reverse Proxy erfolgt.


----