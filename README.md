# Anträge API

As a loan provider get all the data of your applications for highly effective validations and approvals.

![loanProvider](https://img.shields.io/badge/-loanProvider-lightblue)
![mortgageLoan](https://img.shields.io/badge/-mortgageLoan-lightblue)

[![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://docs.api.europace.de/common/authentifizierung/authorization-api/)
[![YAML](https://img.shields.io/badge/OAS-YAML-lightblue)](https://raw.githubusercontent.com/europace/baufismart-antraege-api/master/swagger.yaml)
[![YAML](https://img.shields.io/badge/OAS-HTML-lightblue)](https://europace.github.io/baufismart-antraege-api/docs/index.html)

[![Github](https://img.shields.io/badge/-Github-black?logo=github)]((https://github.com/https://github.com/europace/baufismart-antraege-api))

[![GitHub release](https://img.shields.io/github/v/release/europace/baufismart-antraege-api)](https://github.com/europace/baufismart-antraege-api/releases)

[![Pattern](https://img.shields.io/badge/Pattern-Tolerant%20Reader-yellowgreen)](https://martinfowler.com/bliki/TolerantReader.html)

## Usecases
- get informed about new applications
- get the data of applications for approval
- inform about state of application
- send messag to advisor
- inform about own reference (e.g. account-number)
- manage loan officer on application
## Requirements
- authenticated as loan provider

## Quick Start
To test our APIs and your use cases as quickly as possible, we have created a [Postman Collection](https://github.com/europace/api-quickstart) for you.

### Authentication
Please use [![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://docs.api.europace.de/common/authentifizierung/authorization-api/) to get access to the APIs. The OAuth2 client requires the following scopes:

| Scope                               | API Use case                                                         |
|-------------------------------------|----------------------------------------------------------------------|
| `baufinanzierung:echtgeschaeft`     | to use api in production mode                                        |
| `baufinanzierung:antrag:lesen`      | to get application information                                       |
| `baufinanzierung:antrag:schreiben`  | to update application data (eg state, loan office and own reference) |

### First try
To start tests begin with [Get a list of applications](#get-a-list-of-applications)

## Segments of Antragsnummer
Europace is a multi-lender plattform. For this case antragsnummer is segmented by / for different solutions and applications

Example: `ABC12F/2/1`

Syntax: `[case-id]/[iteratorSolution]/[iteratorApplication]`

| segment | description |
|---------|-------------|
|case-id | case-id of advisor |
|iteratorSolution | a case can contain one or more solutions. Solutions can be loosely coupled and differently approved |
|iteratorApplication | one solution can contain one or more applications, wich have to decide by one loan department, all applications are bonded by solution, each application can contain one or more loans |

Example constellation
In the case ABC12F europace market-engine created a financial solution for the customer with the following three loans:
1. DSL-Bank - Annuitätendarlehen
2. KfW-Bank - Förderdarlehen
3. Hanseatic-Bank - Nachrangdarlehen

Loan 1 and 2 are descided by DSL-Bank, because KfW-Bank products are handled by the mortage bank (background process). Loan 3 is a consumer loan wich is optimizing the mortage conditions. This loan will be decided by Hanseatic themselfs and only needed, if DSL-Bank will approve.

If the advisor applies this offer, it will create 1 solution with 2 applications: \
`ABC12F/1/1` for DSL-Bank \
`ABC12F/1/2` for Hanseatic

More applies of different offers will create more solutions: \
`ABC12F/2/1` \
... \
Furthermore maybe: \
`ABC12F/3/1` \
`ABC12F/3/2` \
`ABC12F/3/3`

## Get all the data you need for approval
### Get a list of applications
As loan provider, you'll get a list of all your applications, sorted by lastChanged, paged and filterable.

Requirement:
- the caller is loan provider or part of it
- the caller is `Kreditsachbearbeiter`
- the caller has scope `baufinanzierung:antrag:lesen`

example-request:
``` http
GET /v2/antraege/ HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
```
You'll receive a list of links to applications.

example-response:
``` json
{
    "_links": {
        "next": {
            "href": "https://baufismart.api.europace.de/v2/antraege?datenKontext=ECHT_GESCHAEFT&sort=absteigend&page=1&limit=10"
        },
        "self": {
            "href": "https://baufismart.api.europace.de/v2/antraege?datenKontext=ECHT_GESCHAEFT&sort=absteigend&page=0&limit=10"
        }
    },
    "antraege": [
        {
            "_links": {
                "self": {
                    "href": "https://baufismart.api.europace.de/v2/antraege/N19ABC/1/%7Bteilantrag%7D"
                }
            },
            "antragsNummer": "N19ABC/1",
            "datenKontext": "ECHT_GESCHAEFT",
            "zeitpunktDerAnnahme": "2022-02-09T08:41:33.572+01:00",
            "letzteAenderung": "2022-02-09T08:41:38.038+01:00",
            "entscheidungsreifeVomVertriebSignalisiert": false
        },
        {
            "_links": {
                "self": {
                    "href": "https://baufismart.api.europace.de/v2/antraege/AB2U4Z/2/%7Bteilantrag%7D"
                }
            },
            "antragsNummer": "AB2U4Z/2",
            "datenKontext": "ECHT_GESCHAEFT",
            "zeitpunktDerAnnahme": "2022-02-09T08:41:12.407+01:00",
            "letzteAenderung": "2022-02-09T08:41:22.09+01:00",
            "entscheidungsreifeVomVertriebSignalisiert": true
        }
    ]
}
```
> Pls note:
> the list is paged.

You can filter the results by using the following parameters:
- datenkontext (using test- or production-mode)
- antragsReferenz (filer by your own reference)
- aenderungSeit (lastChangeUntil for getting all changes after the last call)
- and many more - see documentation

#### Get all applications changed since
Example request:
```http
GET /v2/antraege/?aenderungSeit=2022-02-10 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
```

Example resonse: \
see above

### Get data of an application
As loan Provider, you'll get all data of an application for validation and decision.

Requirement:
- the caller is loan provider or part of it
- the caller is `Kreditsachbearbeiter`
- the caller has scope `baufinanzierung:antrag:lesen`

example-request:
``` http
GET /v2/antraege/ABC12F/1/1 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
```

example-response:
You will receive a list of antraege with all the data.
``` json
{
  "antragsNummer": "ABC12F/1/1",
  "datenKontext": "ECHT_GESCHAEFT",
  "zeitpunktDerAnnahme": "2022-02-09T08:41:33.572+01:00",
  "produktAnbieter": { ...
  },
  "beratung": { ...
  },
  "status": { ...
  },
  "dokumente": [  ...
  ],
  "prolongation": false,
  "letzteAenderung": "2022-02-09T08:41:38.038+01:00",
  "entscheidungsreifeVomVertriebSignalisiert": false,
  "angebot": { ...
  },
  "vermittler": { ...
  },
  "ansprechpartner": { ...
  },
  "kreditSachbearbeiter": { ...
  },
  "zugrundeliegendeDaten": { ...
  },
  "_links": {
      "self": {
          "href": "https://baufismart.api.europace.de/v2/antraege/ABC12F/1/1"
      }
  }
}
```

### Get proofs
To receive all the shared proofs for an application please use the [Unterlagen Push API](https://docs.api.europace.de/baufinanzierung/unterlagen/unterlagen-push-api/)

> Please note: The property `dokumente` is deprecated.

## Drive the process and inform your customer
### Set planned approving date
As loan provider, update the planned approving date for well informed advisors and easy conversations with sales.

Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
PATCH /v2/antraege/ABC12F/1/1 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 66

[
	{ "op": "add", "path": "/voraussichtlicheBearbeitung", "value": "2022-02-10" }
]
```

example response:
``` http
204 no content
```

### Set loan officer
As loan provider, update the loan officer for well informed advisors and easy conversations with sales.

If you set the loan officer, she/he can edit the application and the contact information will be automatically shared with the advisor in BaufiSmart.
Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
PATCH /v2/antraege/ABC12F/1/1 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 66

[
	{ "op": "replace", "path": "/ansprechpartner/partnerId", "value": "XYZ55" }
]
```

example response:
``` http
204 no content
```

### Set loan officer contact details
As loan provider, update the the loan officer contact details for well informed advisors and easy conversations with sales.

If you don't use different europace users for loan officers identity, it is helpful to set contact details for the advisors as information.

> This operation is not necessary, if your loan officers has an identity in europace (see [set loan officer](#set-loan-officer)) and it is set on the application.

Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
PATCH /v2/antraege/ABC12F/1/1 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 66

{ "op": "replace", "path": "/ansprechpartner/externerPartner", 	"value": 
          {
             "kreditBetriebPartnerId":"MYID03",
             "name":"Frau Angela Anaconda", 
             "telefonnummer":"0170 7717789"
          }
}
```
`kreditBetriebPartnerId` is the partnerId of the loan office (can be an organization).

example response:
``` http
204 no content
```

### Set loan provider reference
As loan provider, update your own reference for well informed advisors and easy conversations with sales.

The loan providers reference can be used in communication between advisor and loan officer. Typically, an account-number is used here, when available.

Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
PATCH /v2/antraege/ABC12F/1/1 HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 66

[
	{ "op": "add", "path": "/antragsReferenz", "value": "whoop" }
]
```

example response:
``` http
204 no content
```

### Set state and send message
As loan provider, you can set a state with message for an application to inform the advisor about your descision.

If you set a state for an application the advisor will be automatically informed by your decision and message. You can use it, to send the advisor a message if you choose the right state and use the property `kommentar`.

Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
POST /v2/antraege/ABC12F/1/1/status HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 165

[
  {
     "produktAnbieter": "UNTERSCHRIEBEN",
     "antragsteller": "WIDERRUFEN",
     "kommentar": "{{comment}}"
  }
]
```

The values for the state of produktAnbieter can be:

* `NICHT_BEARBEITET`
* `UNTERSCHRIEBEN`
* `ABGELEHNT`
* `ZURUECKGESTELLT`

The values for the state of antragsteller can be:

* `BEANTRAGT`
* `UNTERSCHRIEBEN`
* `NICHT_ANGENOMMEN`
* `WIDERRUFEN`

The values of Ablehnungsgrund can be:

* `FINANZIELLE_SITUATION`
* `NEGATIV_MERKMAL`
* `WERTERMITTLUNG`
* `KRITERIEN`
* `UNTERLAGEN_UNVOLLSTAENDIG`
* `GEGENANGEBOT`
* `KEINE_ANGABE`

example response:
``` http
201 created
```

### Send message
As loan provider, you can send a message to the advisor to communicate without a denied-state.

> Important! Please prefer [set state](#set-state-and-send-message) for communication with advisor, because states are more helpful for process reporting. This method should be your last choice.

Requirement:
- the caller is loan officer for the application
- the caller has scope `baufinanzierung:antrag:schreiben`

example request:
``` http
POST /v2/antraege/ABC12F/1/1/nachricht HTTP/1.1
Host: baufismart.api.europace.de
Content-Type: application/json
Authorization: Bearer {{access-token}}
Content-Length: 165

{ 
    "text": "your message to advisor"
}
```

example response:
``` http
201 created
```

## How to make a counteroffer
As loan provider, you can resend an adjusted offer, to create a working solution for the customer.

A counteroffer is an adjusted offer based on the application set by the loan provider.

If a counteroffer is created, the original offer will be declined and marked as 'ABGELEHNT' and the new offer is automatically approved and marked as `UNTERSCHRIEBEN ` from the loan provider.

A single loan or a combination of loans can be offered as a counteroffer. The `vertriebsProvisionGesamtAbsolut` (commission amount) has to be exclude the europace-fee. The europace-fee will be recalculated based on your new loan details and is contained in the the new application.

Example request:

``` json
POST /v2/antraege/ABC12F/1/1/gegenangebot HTTP/1.1
Host: baufismart.api.europace.de
Authorization: Bearer {{access-token}}
Content-Type: application/json
Content-Length: 1324

{
  "kennung": "Kombinatorisches Gegenangebot zum existierenden Angebot",
  "darlehen": [
    {
      "darlehensTyp": "ANNUITAETEN_DARLEHEN",
      "darlehensBetrag": 100000.00,
      "sollZins": 1.17,
      "effektivZins": 1.2,
      "rateMonatlich": 264.17,
      "vertriebsProvisionGesamtAbsolut": 1010.00,
      "zinsBindungInJahren": 10,
      "sonderTilgungJaehrlich": 10.0,
      "tilgungsBeginnAm": "2020-11-30",
      "produktDetails": "Angepasstes Finanzierung (weitere Beschreibung möglich)",
      "anfaenglicheTilgung": 2.0,
      "bereitstellungsZinsFreieZeitInMonaten": 3,
      "bereitstellungsZins": 3.0,
      "laufzeitKalkulatorischInJahren": 39
    },
    {
      "darlehensTyp": "KFW_DARLEHEN",
      "darlehensBetrag": 100000.00,
      "sollZins": 0.84,
      "effektivZins": 0.87,
      "rateMonatlich": 383.52,
      "anfaenglicheTilgung": 3.762,
      "vertriebsProvisionGesamtAbsolut": 1072.00,
      "laufzeitKalkulatorischInJahren": 25,
      "zinsBindungInJahren": 10,
      "bereitstellungsZinsFreieZeitInMonaten": 12,
      "produktDetails": "KfW Darlehen zusätzlich zum angepassten Finanzierung",
      "kfwProgramm": "PROGRAMM_153",
      "kfwEnergieEffizienzStandard": "STANDARD_40",
      "tilgungsFreieAnlaufJahre": 1,
      "rateMonatlichInDerTilgungsfreienAnlaufzeit": 77.00
    }
  ]
}
```

Example response:
``` json
201 created
{
    "antragsNummer": "ABC12F/2/1/",
    "_links": "https://baufismart.api.europace.de/v2/antraege/ABC12F/2/1/"
}
```

## FAQ
### How do I know that the application has changed?

On the application (list and individual retrieval), the date of the last change to the application is output in 'letzteAenderung'.

For the query of all requests there are two parameters for limiting the search results:

* `aenderungSeit`
* `aenderungBis`

If one of the parameter set, a date (`YYYY-MM-DD hh:mi:ss`) is expected.

### What is a relevant change to the application?

What events lead to the setting of the `lastChange` field?
* changing state of application
* changing loan office on application
* new proof is shared

### Where can i get more details about the API?
You can use our reference HTML and read through the latest release notes:

- [Complete API reference](https://europace.github.io/baufismart-antraege-api/docs/index.html)
- [API release notes](https://github.com/europace/baufismart-antraege-api/releases/)

## Terms of use
The APIs are provided under the following [Terms of Use](https://docs.api.europace.de/nutzungsbedingungen).

## Support
If you have any questions or problems, you can contact devsupport@europace2.de.
