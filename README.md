# Anträge API

[![GitHub release (latest by date)](https://img.shields.io/github/v/release/europace/baufismart-antraege-api?color=%23c6e6f0&label=Release&logo=github&style=flat-square)](https://github.com/europace/baufismart-antraege-api/releases/latest/)

Die API erlaubt den Zugriff auf Anträge für die Kreditentscheidung als Produktanbieter.

[API Referenz](http://htmlpreview.github.io?https://raw.githubusercontent.com/hypoport/antraege-auslesen-api/master/Dokumentation/index.html)  ⋅
[Release Notes](https://github.com/europace/baufismart-antraege-api/releases/)

## Authentifizierung

> #### Hinweis
>
> Bis Juli 2021 ist die Authentifizierung noch über API-Key möglich. Danach wird die Anträge-API nur noch über OAuth2 ansprechbar sein.
> Wir empfehlen neuen Partnern daher, sich mittels OAuth2 zu verbinden. Bei Fragen wende dich bitte an [devsupport@europace2.de](mailto:devsupport@europace2.de).

Zur Authentifizierung nutzt Europace [OAuth2](https://oauth.net/2/), um Zugang zu den APIs zu ermöglichen. Die Beschreibung unserer Authorization-API findest du [hier](https://github.com/europace/authorization-api).

Der OAuth2-Client benötigt folgende Scopes, um die API verwenden zu können:

| Scope                             | API Use case |
|-----------------------------------|---------------------------------|
| `baufinanzierung:antrag:lesen`       | als Produktanbieter, Daten von Anträgen abrufen. |
| `baufinanzierung:antrag:schreiben`   | als Produktanbieter, Daten eines Antrags ändern (z.B. Antragsstatus, Kreditsachbearbeiter).|

## Häufige Fragen

### Wie bekomme ich eine Liste aller meiner Anträge?

```
GET https://baufismart.api.europace.de/v2/antraege
```

Diese Liste kommt seitenweise.

### Wie rufe ich einen konkreten Antrag ab? 

Als Produktanbieter

```
GET https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

Als Vertrieb

```
GET https://baufismart.api.europace.de/v2/vorgaenge/AB1234/1/1/antraege
```

### Wie erkenne ich, dass sich der Antrag geändert hat?

Am Antrag (Liste und Einzelabruf) wird im Feld `letzteAenderung` das Datum der letzten Änderung des Antrags ausgegeben.

Für die Abfrage aller Anträge gibt es zwei Parameter zur Einschränkung der Suchergebnisse:

* `aenderungSeit`
* `aenderungBis`

Für beide Parameter wird ein Datum erwartet, wenn gesetzt.

#### Was ist eine relevante Änderung am Antrag?

Welche Ereignisse führen zum Setzen des `letzteAenderung` Feldes?

* Statusänderung des Antrags
* Änderung des Kreditsachbearbeiters
* Freigabe eines Dokumentes für den Kreditbetrieb

### Welche PATCH Operationen sind möglich?

Die folgenden Felder können an einem Antrag verändert werden:

* Antragsstatus inklusive Ablehnungsgrund
* Eigene Referenz setzen und löschen
* Kreditsachbearbeiters / Ansprechpartner setzen und ändern

Details zu JSON Patch findest du unter [jsonpatch.com](http://jsonpatch.com/).
Es sind mehrere Operationen mit einem `PATCH Aufruf möglich.

Die URL zum Aufruf des `PATCH`-Kommandos lautet:

```
PATCH https://baufismart.api.europace.de/v2/antraege/AB1234/1/1
```

Im Folgenden werden die Patch-Operationen als Body dargestellt.

#### Wie setze ich den Status eines Antrags neu?

```json
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

* `FINANZIELLE_SITUATION`
* `NEGATIV_MERKMAL`
* `WERTERMITTLUNG`
* `KRITERIEN`
* `UNTERLAGEN_UNVOLLSTAENDIG`
* `GEGENANGEBOT`
* `KEINE_ANGABE`

Sollen nur Teile des Status aktualisiert werden, kann man auch ein Patch auf der Subressource aufrufen.

```json
[
	{ "op": "replace", "path": "/status/produktAnbieter", "value": "ABGELEHNT" }
]
```

#### Kann ich meine eigene Referenz am Antrag hinterlegen?
Ja!

```json
[
	{ "op": "add", "path": "/antragsReferenz", "value": "<DEINE_REFERENZ>" }
]
```

#### Wie kann ich die Antragsreferenz eines Antrags wieder löschen?

```json
[
	{ "op": "remove", "path": "/antragsReferenz" }
]
```

#### Wie kann ich die voraussichtliche Bearbeitungsdatum eines Antrags setzen?

Es besteht die Möglichkeit den voraussichtlichen Zeitpunkt der Antragsbearbeitung mitzuteilen.

```json
[
	{ "op": "add", "path": "/voraussichtlicheBearbeitung", "value": "2017-11-12" }
]
```
#### Wechsel des Kreditsachbearbeiters / Ansprechpartner mit seiner Europace PartnerId.

```json
[
	{ "op": "replace", "path": "/ansprechpartner/partnerId", "value": "AB123" }
]
```

## Fragen und Anregungen
Bei Fragen und Anregungen entweder ein Issue in GitHub anlegen oder an [devsupport@europace2.de](mailto:devsupport@europace2.de) schreiben.

## Nutzungsbedingungen
Mit der Verwendung dieser API stimme ich den [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zu.
