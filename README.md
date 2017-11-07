# Anträge Auslesen API
API Definition zum Auslesen von Anträgen aus der Europace-Plattform aus Sicht eines Produktanbieters.

### Swagger Spezifikationen
Die API ist vollständig in Swagger definiert. Die Swagger Definitionen werden sowohl im JSON- als auch im YAML-Format zur Verfügung gestellt.

Aus diesen Dateien können mit Hilfe von [Swagger Codegen](https://github.com/swagger-api/swagger-codegen) Clients in verschiedenen Sprachen generiert werden.

### API Documentation

 - [RELEASE NOTES](https://github.com/hypoport/antraege-auslesen-api/releases)
 - [statische HTML Seite](http://htmlpreview.github.io?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/antraege-v2.3/Dokumentation/index.html)

Zur Unterstützung für das Mapping werden folgende Dateien bereit gestellt:
  - [CSV Datei](https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/antraege-v2.3/definitions.csv)
  - [Excel Datei](https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/antraege-v2.3/definitions.xls)

### Generierung des Clients
#### JAVA mit Retrofit

1. Die aktuelle Swagger-Codegen Version 2.2.2 downloaden
2. Client mit folgendem Kommando generieren:


```
java -jar swagger-codegen-cli-2.2.2.jar generate -i swagger.yaml -l java -c codegen-config-file.json -o europace-api-client
```

Example **codegen-config-file.json**:

```
{
  "artifactId": "europace-api-client",
  "groupId": "de.europace.api",
  "library": "retrofit2",
  "artifactVersion": "0.1",
  "dateLibrary": "java8"
}

```

### Authentifizierung

Die Authentifizierung läuft über den [OAuth2](https://oauth.net/2/) Flow vom Typ *ressource owner password credentials flow*.
https://tools.ietf.org/html/rfc6749#section-1.3.3


#### Credentials
Um die Credentials zu erhalten, erfagen Sie beim Helpdesk der Plattform die Zugangsdaten zur Auslesen API, bzw. bitten Ihren Auftraggeber dies zu tun.

#### Schritte 
1. Absenden eines POST Requests auf den [Login-Endpunkt](https://htmlpreview.github.io/?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/antraege-v2.3/Dokumentation/index.html#_oauth2) /login mit Username und Password. Der Username entspricht der PartnerId und das Password ist der API-Key. Auf dem Testsystem können diese Werte frei gewählt werden. Alternativ kann ein Login auch über einen GET Aufruf mit HTTP Basic Auth auf den Login-Endpunkt erfolgen.
2. Aus der JSON-Antwort das JWToken (access_token) entnehmen
3. Bei weiteren Requests muss dieses JWToken als Authorization Header mitgeschickt werden.

#### Test mit Mock-Daten
Für die Entwicklung neuer Clients können Sie mit einer Mock-Implementierung arbeiten. Diese ist unter https://baufismart.api.europace.de/mock erreichbar. So kann eine Liste von Anträgen zum Beispiel unter https://baufismart.api.europace.de/mock/antraege abgerufen werden.

Passende Access-Token können über den oben beschriebenen Authentifizierungs-Prozess unter https://api.europace.de/mock/login abgerufen werden.

### Quickstart

#### Liste aller meiner Anträge abrufen

```
GET https://baufismart.api.europace.de/v2/antraege
```

#### Einen konkreten Antrag abrufen

```
GET https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

### Mögliche PATCH Operationen eines Antrags

Details zu JSON Patch finden Sie unter [jsonpatch.com](http://jsonpatch.com/).
Es sind mehrere Operationen mit einem PATCH Aufruf möglich. 

Die URL zum Aufruf des Patch-Kommandos ist immer die gleiche:
```
PATCH https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

Im folgenden werden die Patch-Operationen als Body dargestellt.

##### Den Status eines Antrags neu setzen
```
[
	{ "op": "replace", "path": "/status", "value": 
		{
			"antragsteller": "BEANTRAGT",
			"produktAnbieter": "ABGELEHNT",
			"ablehnungsgrund": "Unterlagen fehlen"
		}
	}
]
```

Sollen nur Teile des Status aktualisiert werden, kann man auch ein Patch auf der Subressource aufrufen.
```
[
	{ "op": "replace", "path": "/status/produktAnbieter", "value": "ABGELEHNT" }
]
```

##### Die Antragsreferenz eines Antrags setzen:
```
[
	{ "op": "add", "path": "/antragsReferenz", "value": "<IHRE_VORGANGSNUMMER>" }
]
```

##### Die Antragsreferenz eines Antrags löschen:
```
[
	{ "op": "remove", "path": "/antragsReferenz" }
]
```

##### Das voraussichtliche Bearbeitungsdatum eines Antrags setzen:
```
[
	{ "op": "add", "path": "/voraussichtlicheBearbeitung", "value": "2017-11-12" }
]
```
##### Wechsel des Bearbeiters 

_noch nicht implementiert_