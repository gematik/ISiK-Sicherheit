# Conformance: Scopes und Kontexte

Die Vorgaben von ISiK-Connect betreffen Systeme in der Rolle eines Ressourcenservers, Autorisierungsserver und App-Launcher. Diese Systeme MÜSSEN die auf dieser Seite beschriebenen Autorisierungsinformationen bei jedem Zugriffsversuch auf FHIR-Ressourcen verarbeiten können.

## Berechtigungen auf Ressourcentypen

Berechtigungen auf Ressourcentypen MÜSSEN sowohl in der _SMART Capabilities_ Datei als auch in den gegenüber dem ISiK-Ressourcenserver bestätigten _Scopes_ in der folgenden Syntax kodiert werden:

```(patient | user | system) \/ (_Ressourcetyp_ | \*) \. c?r?u?d?s? (\? (_param_\=_value_) (\& _param_\=_value_)* )?```

### Scope-Level
SMART-on-FHIR-Berechtigungen auf Ressourcen lassen sich in drei Kategorien einteilen, die alle durch ISiK-konforme Ressourcenserver unterstützt werden MÜSSEN:

* ["patient"-Level Scopes](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#patient-specific-scopes) geben an, welche verfügbaren Nutzerberechtigungen auf allen Ressourcen im gewählten _Patient Compartment_ an den Client delegiert werden bzw. werden sollen.  
* ["user"-Level Scopes](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#user-level-scopes) geben an, welche verfügbaren Nutzerberechtigungen auf allen Ressourcen durch den Benutzer an den Client delegiert werden bzw. werden sollen.
* ["system"-Level Scopes](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#system-level-scopes) geben an, welche verfügbaren Nutzerberechtigungen auf allen Ressourcen an einen (technischen) Client delegiert werden bzw. werden sollen, unabhängig davon welcher Benutzer hiermit interagiert.

Autorisierungen in einem _SMART on FHIR_ _Launch Kontext_, für den keine Compartment-Definition existiert (z. B. 'launch/location'), SOLLEN in einem _"user"_- oder _"system"-Level Scope_ erfolgen (z. B. 'user/Location.rs').

### Ressourcetyp und Operationen
Es MÜSSEN alle in ISiK Stufe 3 profilierten Ressourcentypen unterstützt werden. Sofern in ISiK Stufe 3 auf einem Ressourcentyp als zulässig definiert, MÜSSEN alle in FHIR definierten lesenden und modifizierenden Operationen unterstützt werden:

|Berechtigung|FHIR-Operation auf System-Ebene|FHIR-Operation auf Typ-Ebene|FHIR-Operation auf Instanz-Ebene|
|:-----------|:------------------------------|:---------------------------|--------------------------------|
|c           |                               |create                      |                                |
|r           |                               |                            |read, vread, history            |
|u           |                               |                            |update, patch                   |
|d           |                               |                            |delete                          |
|s           |search, history                |search, history             |                                |

Berechtigungen werden im _Scope_ in der dargestellten Reihenfolge ('cruds') angegeben (vgl. https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#clinical-scope-syntax). Bei einer falschen Reihenfolge MUSS der ISiK-Autorisierungsserver einen Fehler auslösen. 

Die Möglichkeit von [_Wildcard-Scopes_](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#wildcard-scopes) MUSS unterstützt werden.

### Filter
Alle in einem ISiK-Modul für einen Ressourcetyp unterstützten Suchparameter inkl. _Modifier_ und Kombinationsmöglichkeiten MÜSSEN als Teil eines _Scopes_ unterstützt werden.

### Beispiele

Die folgende Beispiele geben gültige _Scopes_ wieder:

* ```patient/Patient.rs```
* ```patient/Observation.cruds```
* ```patient/Observation.rs?category=http://terminology.hl7.org/CodeSystem/observation-category|laboratory```

## Launch-Context

Clients können über die zuvor beschriebenen Access-Scopes hinaus sogenannte Launch-Context-Scopes bei dem Autorisierungsserver anfordern. Launch Claims werden nicht als Teil des Access-Tokens, sondern als Teil des JSON-Dokumentes als Antwort auf die Anfrage eines Access-Tokens zurückgeliefert. Diese Claims enthalten entweder spezifische IDs oder URIs, die FHIR-Referenzen abbilden, um kontext-relevante Informationen vorab zu übermitteln, sodass der Client entsprechend einen "Einstiegspunkt" in das System erhält.

Launch-Scopes besitzen die Syntax ```launch/<resourcen-typ>```. Der "<resourcen-typ>" Teil des Scopes ist case-senstive und MUSS kleingeschrieben werden. Der Autorisierungsserver MUSS einen Patienten- (launch/patient) und Fallkontext (launch/encounter) herstellen können. Weitere Kontexte KÖNNEN durch den Autorisierungsserver etabliert werden - [vgl. SMART App Launch - 2.2.3.3.1 - fhirContext](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#fhir-context).

Die Kontextinformationen sind durch den Autorisierungsserver bei dem Ressourcen-Server anzufordern. Als Implementierungsunterstützung können die Informationen durch den "launch"-Parameter, welcher während eines EHR-Launchs ausgetauscht wurde, abgeleitet werden. Im Falle eines Standalone Launchs, können die Informationen manuell durch die Benutzer ausgewählt oder durch sonstige geeignete Maßnahmen bestimmt werden. Hierzu bestehen seitens der Spezifikation keine Vorgaben.

Die Vorgaben zur Kodierung der [Patient-, Encounter- und fhirContext- Claims](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#launch-context-arrives-with-your-access_token) MÜSSEN implementiert werden.

Bestätigungsrelevante Systeme in der Rolle eines ISiK-Ressourcenservers DÜRFEN im ["patient"-Level Scope](https://hl7.org/fhir/smart-app-launch/STU2.1/scopes-and-launch-context.html#patient-specific-scopes) (s.u.) KEINE Zugriffstoken (_Access Token_) akzeptieren, in denen kein Kontext als Bezugspunkt für die gewährten Zugriffsrechte angegeben ist bzw. per _Introspection_ ermittelt werden kann.

Es MUSS mindestens der Kontext "patient" unterstützt werden.

### Compartments

Bestätigungsrelevante Systeme in der Rolle eines ISiK-Ressourcenservers MÜSSEN bei der Durchsetzungen von Autorisierungen die Festlegungen zum [_Compartment Patient_](https://hl7.org/fhir/R4/compartmentdefinition-patient.html) unterstützen. Sie KÖNNEN weitere der von HL7 definierten _CompartmentDefinitionen_ unterstützen.

Bestätigungstelevante Systeme in der Rolle eines ISiK-Ressourcenserver DÜRFEN KEINE eigenen _CompartmentDefinitionen_ definieren, da eine Definition von _Compartments_ alleinig durch HL7 erfolgen darf. 

Die Unterstützung eines _Compartments_ umfasst, dass die Festlegungen in der _CompartmentDefinition_ die Gruppierung von über _Scopes_ angegebenen Berechtigungen zu der als Kontext angegebenen Ressource bestimmen. Im _"patient"-Level Scope_ (s.u.) bestimmt das [_Patient Compartment_]https://hl7.org/fhir/R4/compartmentdefinition-patient.html) die maximal zulässigen Berechtigungen eines Zugriffs auf die den angegebenen Kontext darstellende Ressource.

Beispiel: Der gegebene Kontext ist der Patient "123". Die über einen _Scope_ angegebene Autorisierung ist 'patient/Observation.r'. Der ISiK-Ressourcenserver darf nur Anfragen ausführen, die lesend auf _Observation_-Ressourcen zugreifen, die über 'Observation.subject' oder 'Observation.performer' dem Patienten "123" zugeordnet sind.
