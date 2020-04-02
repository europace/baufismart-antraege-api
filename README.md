# Anträge API

##### Aktuelle Version: 2.19

[Aktuelles RELEASE](https://github.com/hypoport/antraege-auslesen-api/releases/)

API Definition zum Auslesen von Anträgen aus der Europace-Plattform aus Sicht eines Produktanbieters. Außerdem erlaubt die API das Setzen des Antragsstatus und des Kreditsachbearbeiters.

### Swagger Spezifikationen
Die API ist vollständig in Swagger definiert. Die Swagger Definitionen werden sowohl im JSON- ([swagger.json](swagger.json)) als auch im YAML-Format ([swagger.yaml](https://github.com/europace/baufismart-antraege-api/blob/master/swagger.yaml)) zur Verfügung gestellt.

Diese Spezifikationen können auch zur Generierung von Clients für diese API verwendet
werden. Dazu empfehlen wir das Tool [Swagger Codegen](https://github.com/swagger-api/swagger-codegen)

### Dokumentation

 - [RELEASE NOTES](https://github.com/hypoport/antraege-auslesen-api/releases)
 - [statische HTML Seite](http://htmlpreview.github.io?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/Dokumentation/index.html)
 - [API DOCS](https://antraege-auslesen-api-5.api-docs.io/2.14/v2/einen-antrag-aktualisieren)

### Generierung des Clients
##### JAVA mit Retrofit

1. Die aktuelle Swagger-Codegen Version, mindestens 2.2.2, downloaden
2. Client mit folgendem Kommando generieren:

Example:

```
java -jar swagger-codegen-cli-2.2.2.jar generate -i swagger.yaml -l java -c codegen-config-file.json -o europace-api-client
```

Für Retrofit2

```
java -jar swagger-codegen-cli-2.2.2.jar generate -i swagger.yaml -l java --library retrofit2 -c codegen-config-file.json -o europace-api-client
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

##### Credentials
Um die Credentials zu erhalten, erfragen Sie beim Helpdesk der Plattform die Zugangsdaten zur Auslesen API, bzw. bitten Ihren Auftraggeber dies zu tun.

##### Schritte
1. Absenden eines POST Requests auf den [Login-Endpunkt](https://htmlpreview.github.io/?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/Dokumentation/index.html#_oauth2) mit Username und Password. Der Username entspricht der PartnerId und das Password ist der API-Key. Alternativ kann ein Login auch über einen GET Aufruf mit HTTP Basic Auth auf den Login-Endpunkt erfolgen.
2. Aus der JSON-Antwort das JWToken (access_token) entnehmen
3. Bei weiteren Requests muss dieses JWToken als Authorization Header mitgeschickt werden.

##### Beispiel Implementierung für die Authentifizierung mit einem Java Client und Retrofit

Den API-Client nicht mit dem Default-Konstruktor, sondern dem Credentials-Kontruktor erzeugen. Z.B:

```
 api = new ApiClient("oauth2","partnerID", "api-key").createService(AntraegeApi.class);
```

# Häufige Fragen

## Wie bekomme ich eine Liste aller meiner Anträge?

```
GET https://baufismart.api.europace.de/v2/antraege
```
Diese Liste kommt seitenweise.

## Wie rufe ich einen konkreten Antrag ab?

```
GET https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

## Wie erkenne ich, dass sich der Antrag geändert hat?

Am Antrag (Liste und Einzelabruf) wird im Feld `letzteAenderung` das Datum der letzten Änderung des Antrags ausgegeben.

Für die Abfrage aller Anträge gibt es zwei Parameter zur Einschränkung der Suchergebnisse:

* `aenderungSeit`
* `aenderungBis`

Für beide Parameter wird ein Datum erwartet, wenn gesetzt.

### Was ist eine relevante Änderung am Antrag?

Welche Ereignisse führen zum Setzen des `letzteAenderung` Feldes?

* Statusänderung des Antrags 
* Änderung des Kreditsachbearbeiters
* Freigabe eines Dokumentes für den Kreditbetrieb

## Welche PATCH Operationen sind möglich?

Die folgenden Felder können an einem Antrag verändert werden:

* Antragsstatus inklusive Ablehnungsgrund
* Eigene Referenz setzen und löschen
* Kreditsachbearbeiters / Ansprechpartner setzen und ändern


Details zu JSON Patch finden Sie unter [jsonpatch.com](http://jsonpatch.com/).
Es sind mehrere Operationen mit einem PATCH Aufruf möglich.

Die URL zum Aufruf des Patch-Kommandos ist immer die gleiche:
```
PATCH https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

Im folgenden werden die Patch-Operationen als Body dargestellt.

### Wie setze ich den Status eines Antrags neu?

```
[
	{ "op": "replace", "path": "/status", "value":
		{
			"antragsteller": "BEANTRAGT",
			"produktAnbieter": "ABGELEHNT",
			"ablehnungsgrund": "FINANZIELLE_SITUATION",
			"kommentar": "Kommentar zum Statuswechsel"
		}
	}
]
```
Der Ablehnungsgrund kann lauten:

* FINANZIELLE_SITUATION
* NEGATIV_MERKMAL
* WERTERMITTLUNG
* KRITERIEN
* UNTERLAGEN_UNVOLLSTAENDIG
* GEGENANGEBOT
* KEINE_ANGABE

Sollen nur Teile des Status aktualisiert werden, kann man auch ein Patch auf der Subressource aufrufen.
```
[
	{ "op": "replace", "path": "/status/produktAnbieter", "value": "ABGELEHNT" }
]
```

### Kann ich meine eigene Referenz am Antrag hinterlegen?
Ja!

```
[
	{ "op": "add", "path": "/antragsReferenz", "value": "<DEINE_REFERENZ>" }
]
```

### Wie kann ich die Antragsreferenz eines Antrags wieder löschen?

```
[
	{ "op": "remove", "path": "/antragsReferenz" }
]
```

### Wie kann ich die voraussichtliche Bearbeitungsdatum eines Antrags setzen?

```
[
	{ "op": "add", "path": "/voraussichtlicheBearbeitung", "value": "2017-11-12" }
]
```
### Wechsel des Kreditsachbearbeiters / Ansprechpartner mit seiner Europace PartnerId.

```
[
	{ "op": "replace", "path": "/ansprechpartner/partnerId", "value": "AB123" }
]
```

## Fragen und Anregungen
Bei Fragen und Anregungen entweder ein Issue in GitHub anlegen oder an [helpdesk@europace2.de](mailto:helpdesk@europace2.de) schreiben.
