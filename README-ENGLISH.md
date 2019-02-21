# "Anträge Auslesen" API - English Documentation 🇺🇸🇬🇧

##### Current version: 2.9

API documentation for exporting mortage application data from the EUROPACE platform from the point-of-view of a lender/bank.

**Please note:** unlike the german documentation, this documentation is not guaranteed to be up to date. Please check before you implement anything. If you are unsure, write to devsupport@europace.de.


### Swagger spcification

This API is fully specified in Swagger. The specification is available as JSON ([swagger.json](swagger.json)) and as as YAML ([swagger.yaml](swagger.yaml)).


### Documentation

 - [RELEASE NOTES](https://github.com/hypoport/antraege-auslesen-api/releases)
 - [statische HTML Seite](http://htmlpreview.github.io?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/Dokumentation/index.html)

The following files are provided to help with mapping:
  - [CSV Datei](https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/definitions.csv)
  - [Excel Datei](https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/definitions.xls)


### Authentication



Authentication uses the [OAuth2](https://oauth.net/2/) flow of type: *ressource owner password credentials flow*.
https://tools.ietf.org/html/rfc6749#section-1.3.3

##### Credentials
To use the API you require a PARTNER_ID and API_KEY. If you do not have these credentials please contact devsupport@europace.de.


##### Steps
1. Make a POST Request to https://api.europace.de/login using PARTNER_ID and API_KEY (see example below).
2. Extract the JWToken (access_token) from the response.
3. All further requests to the API must include this JWToken as an Authorization header (see example below).

##### Example CURL request to get access access_token

```
curl -X POST \
  https://api.europace.de/login \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/x-www-form-urlencoded' \
  -d 'username=PARTNER_ID&password=API_KEY'
```

##### Example CURL request with bearer access_token

```
curl -X GET \
  https://baufismart.api.europace.de/v2/antraege \
  -H 'authorization: Bearer {{access_token}}' \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \

```

### Quickstart

##### List all mortage applications I have access to

```
GET https://baufismart.api.europace.de/v2/antraege
```

##### Access a specific mortage application

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

##### Die Antragsreferenz eines Antrags setzen

```
[
	{ "op": "add", "path": "/antragsReferenz", "value": "<IHRE_VORGANGSNUMMER>" }
]
```

##### Die Antragsreferenz eines Antrags löschen

```
[
	{ "op": "remove", "path": "/antragsReferenz" }
]
```

##### Das voraussichtliche Bearbeitungsdatum eines Antrags setzen

```
[
	{ "op": "add", "path": "/voraussichtlicheBearbeitung", "value": "2017-11-12" }
]
```
##### Wechsel des Bearbeiters

_noch nicht implementiert_

## Fragen und Anregungen
Bei Fragen und Anregungen entweder ein Issue in GitHub anlegen oder an [helpdesk@europace2.de](mailto:helpdesk@europace2.de) schreiben.
