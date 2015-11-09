# API-standarder

Dette dokumentet inneholder standarder, og beste praksis for applikasjonsprogrammeringsgrensesnitt (API-er). 

...

### Design for vanlige bruksmåter

For API-er som skal syndikere data, vurder flere vanlige bruksmåter:

* **Bulk data.** Klienter vil ofte etablere sin egen kopi av API-datasettet i sin helhet.
For eksempel kan noen ville bygge en egen søkemotorpå toppen av datasettet,
ved hjelp av ulike parametere, og teknologi enn den "offisielle" API tillater.
Hvis API ikke kan lett fungere som en bulk dataleverandør, legg til en separat mekanisme for å anskaffe datasettet i bulk.
* **Å holde seg oppdatert.** Spesielt for store datasett, kan kunder ønsker å beholde datasettet
oppdatert uten å laste ned datasettet etter hver endring. Hvis dette er en ønsket bruksmåte, prioriter den i API-designet.
* **Kjøre "dyre" spørringer.** What would happen if a client wanted to automatically send text messages to thousands of people or light up the side of a skyscraper every time a new record appears? Consider whether the API's records will always be in a reliable unchanging order, and whether they tend to appear in clumps or in a steady stream. Generally speaking, consider the "entropy" an API client would experience.

### Versjonering

Aldri utgi et API uten versjonstall. Sett versjonsnummeret til API-et i URLen.

Versjoner skal være heltall med prefiksen 'v' foran, ikke desimaltall. For eksempel:
* Gode: v1, v2, v3
* Dårlig: v-1.1, v1.2, 1.3

Vedlikehold API-et minst en versjon tilbake.

### Bruk ditt eget API

Den beste måten for å forstå, implementere, og håndtere svakheter i API-et - er å bruke det i et produksjonssystem.

### Kontaktinformasjon

Benytt en opplagt mekanisme, slik at kundene enkelt kan rapportere problemer, og stille spørsmål om API-et.

Ved bruk av f.eks GitHub for API koden, bruk den tilhørende "issue trackeren" der.
I tillegg, legg ved en e-post adresse for direkte, ikke offentlig henvendelser.

### Varsel ved oppdateringer

Bruk en enkel mekanisme slik at klientene kan følge endringer for API-et.

Vanlige måter å håndtere dette på, er ved bruk av e-post lister eller en [dedikert utvikler blog](https://developer.github.com/changes/) med en RSS-strøm.

### API endepunkt

Et "endepunkt" er en kombinasjon av to ting:
* Et verb (f.eks `GET` eller `POST`)
* URL stien (f.eks `/artikler`)

Informasjon kan bli sendt til et endepunkt på følgende måter:

* URL-en's søkestreng (f.eks `?year=2015`)
* HTTP headers (f.eks `X-Api-Key: nøkkel`)

Når vi snakker om "RESTful" i disse dager, mener vi egentlig å lage enkle, intuitive endepunkter som
representerer unike funksjoner i API-et.

Generelt:

* **Unngå singel-endepunkt API-er**. Ikke putt flere operasjoner i samme endepunkt med samme HTTP-verb.
* **Prioriter enkelhet**. Det bør være enkelt å gjette hva et endepunkt gjør ved å se på URLen og HTTP-verbet, uten å måtte se på spørrestrengen.
* Endepunkt URL-er bør annonsere ressurser, og **unngå verb**

Noen eksempler på disse prinsippene i aksjon:

* [FBOpen API documentation](https://18f.github.io/fbopen/)
* [OpenFDA example query](http://open.fda.gov/api/reference/#example-query)
* [Sunlight Congress API methods](https://sunlightlabs.github.io/congress/#using-the-api)

### Bruk flertall substantiver

Ikke bland sammen entall og flertall substantiver.

Hold det enkelt ved å bare bruke flertall substantiver for alle ressurser.

### HTTP-verb

HTTP-verb, eller metoder, skal bli brukt i samsvar med definisjonene under [HTTP/1.1 standarden](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). 

Her er et eksempel på et HTTP-verb kart for opprette, lese, oppdatere, og slette operasjoner i en bestemt kontekst:

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

### Bruk JSON

[JSON](https://en.wikipedia.org/wiki/JSON) er en utmerket, bredt støttet transport format, egnet for de fleste API-er.

Bruk av JSON, og bare JSON, er i praktisk standard for API-er. Det reduserer kompleksitet for både API-leverandør, og forbruker.

Generelle JSON retningslinjer:

* Responser bør være **et JSON objekt** (ikke en array). Ved bruk av array for å returnere resultater, begrenses muligheten til å inkludere metadata om resultatene, og API-ets evne til å legge til flere toppnøkler i fremtiden.
* **Ikke bruk uforutsigbare nøkler**. Parsing av en JSON respons der nøkelene er uforutsigbare (f.eks utledet fra data) er vanskelig, og skaper friksjon for klienter.
* **Bruk [`camelCase`](https://en.wikipedia.org/wiki/CamelCase) for nøkler**. Ulike språk benytter ulike praksiser. JSON bruker `camelCase`, ikke [`snake_case`](https://en.wikipedia.org/wiki/Snake_case).

### Bruk et konsekvent datoformat

Bruk [ISO 8601](https://xkcd.com/1179/) i UTC for både forespørsler og svar.

For datoer ser det slik ut `2015-02-27`. For fullstendige tidspunkt `2015-02-27T10:00:00Z`.

Dette datoformatet brukes over hele nettet, og setter hvert felt i en konsistent rekkefølge.

### API Nøkler

Disse standardene tar ikke stilling til om ikke å bruke API-nøklene.

Men _hvis_ nøklene brukes til å administrere, og godkjenne API-tilgang, bør API tillate noen form for uautorisert adgang, uten nøkler.

Dette gjør det mulig for nykommere å eksperimentere med API i demo miljøer og med enkle `curl` /` wget`  forespørsler.

Vurder om dette er produktets mål å tillate et visst nivå av normal bruk i produksjon av API-et, uten håndheving avansert registrering av klienter.

### Feilhåndtering

Håndter alle feil (inkludert uoppfangede unntak), og returner det i en datastruktur i samme format som resten av API-et.

Feilmeldinger må ha med en HTTP status kode, en melding til utviklere, melding for sluttbrukeren (når det er hensiktsmessig), en intern feilkode (som henviser til en spesifisert intern ID), og en link hvor utvikleren kan finne mer info. For eksempel:

```json
{
  "status": 400,
  "developerMessage": "Verbose, plain language description of the problem. Provide developers
   suggestions about how to solve their problems here",
  "userMessage": "This is a message that can be passed along to end-users, if needed.",
  "errorCode": "444444",
  "moreInfo": "http://www.example.gov/developer/path/to/help/for/444444,
   http://drupal.org/node/444444",
}
```

Eksempler på HTTP-svarkoder

* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error
* Full liste over [HTTP-statuskoder](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

HTTP-svar med feilinformasjon bør bruke en `4XX` statuskode for å indikere en svikt på klientsiden (for eksempel ugyldig autorisasjon eller en ugyldig parameter), og en` 5XX` statuskode for å angi en svikt på serversiden (for eksempel et uoppfanget unntak).

### Sidenummerering

Hvis sidenummerering er nødvendig for å navigere datasett, bruk den metoden som er mest fornuftig for API-ets data.

Vanlige parametere:

* `page` og` per_page`. Er intuitivt for mange bruksmåter. Linker til "side 2" vil kanskje ikke alltid inneholde de samme dataene.
* `offset` og `limit`. Denne standarden kommer fra SQL-database verden, og er et godt alternativ når du trenger stabile permalinks til resultat-settet.
* `since` and `limit`. Få alt "siden" noen ID eller tidsstempel. Nyttig når det er en prioritert oppgave å la kundene effektivt være synkronisert med data. Vanligvis krever det at resultatsettes rekkefølge er stabil.

Metadata:

Inkluder nok metadata så klientene kan kalkulere hvor mye data det er, og hvordan man eventuelt skal hente det neste datasettet med resultater.

Et eksempel på hvordan dette kan være implimentert:

```json
{
  "results": [ ... actual results ... ],
  "pagination": {
    "count": 2340,
    "page": 4,
    "per_page": 20
  }
}
```

### Bruk HTTPS

Nye API-er må bruke og kreve [HTTPS kryptering](https://en.wikipedia.org/wiki/HTTP_Secure) (med TLS/SSL). HTTPS gir:

* **Sikkerhet**. Innholdet i forespørselen krypteres over Internett.
* **Autentisitet**. En sterkere garanti for at en klient kommuniserer med det virkelige API-et.
* **Personvern**. Forbedret personvern for applikasjoner og brukere som benytter API-et. HTTP-headers og søkestrengparametre (blant annet) vil bli kryptert.
* **Kompatibilitet**. Bredere klient kompatibilitet. For at CORS forespørsler til API-et skal virke på HTTPS nettsteder - ikke bli blokkert som "mixed content" - må disse forespørslene må være over HTTPS.

HTTPS bør konfigureres ved hjelp av moderne beste praksis, herunder koder som det støtter [forward secrecy](http://en.wikipedia.org/wiki/Forward_secrecy), og [HTTP Strict Transport Security](http://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security). **Dette er ikke uttømmende**: Bruk verktøy som [SSL Labs](ssllabs.com/ssltest/analyze.html) til å vurdere API-ets HTTPS konfigurasjon.

For et eksisterende API som går over vanlig HTTP, er første skritt å legge til HTTPS-støtte, og oppdatere dokumentasjonen til å erklære det standard, bruk det i eksempler osv.

Deretter vurder muligheten til å deaktivere eller omdirigere HTTP forespørsler. Se [GSA/api.data.gov#34](https://github.com/GSA/api.data.gov/issues/34) for en diskusjon angående noen av utfordringene ved overgangen fra HTTP->HTTPS.

### Komprimering

For bedre ytelse anbefaler vi å bruke gzip-komprimering av API-svarene.

#### Server Name Indication

Hvis du kan, bruk [Server Name Indication](http://en.wikipedia.org/wiki/Server_Name_Indication) (SNI) for å tjene HTTPS forespørsler.

SNI er en utvidelse til TLS, [først foreslått i 2003](http://tools.ietf.org/html/rfc3546), som tillater SSL-sertifikater for flere domener å bli servert fra en enkelt IP-adresse.

Ved hjelp av en IP-adresse til å betjene flere HTTPS-aktiverte domener kan gi betydelig lavere kostnader og kompleksitet i server-hosting og administrasjon.

Dette gjelder særlig når IPv4-adresser blir mer sjeldne og kostbare. SNI er en god idé, og det er bred støtte.

Men noen klienter og nettverk fortsatt ikke skikkelig støtte for SNI. For øyeblikket:

* Internet Explorer 8 og under på Windows XP
* Android 2.3 (Gingerbread) og under.
* Alle versjoner av Python 2.x (En versjon av Python 2.x med SNI [er planlagt](http://legacy.python.org/dev/peps/pep-0466/)).
* Noen enterprise nettverksmiljøer er konfigurert på en tilpasset måte som deaktiverer eller forstyrrer SNI støtte.

### Bruk UTF-8

Bruk bare [UTF-8](http://utf8everywhere.org).

Forvent tegn med "smarte anførselstegn" i API-resultatet, selv om de ikke er forventet.

Et API skal fortelle klientene at det forventes UTF-8 ved å inkludere en karaktersett notasjon i `Content-Type` header for svaret.

Et API som returnerer JSON må bruke:

```
Content-Type: application/json; charset=utf-8
```

### CORS

For at klienter skal kunne bruke et API fra innsiden i nettlesere, må API-et [aktivere CORS](http://enable-cors.org).

For den enkleste og mest vanlige bruksmåten, der hele API-et bør være tilgjengelig fra innsiden av nettleseren, aktivering av CORS er så enkel som å inkludere denne HTTP header i alle svarene:

```
Access-Control-Allow-Origin: *
```
Det er støttet av [alle moderne nettlesere](http://enable-cors.org/client.html), og vil bare virke i JavaScript klienter som f.eks [jQuery](https://jquery.com).

For mer avansert konfigurasjon, se på [W3C spesifikasjonene](http://www.w3.org/TR/cors/) eller [Mozilla's veiledning](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

**Hva med JSONP?**

JSONP er [ikke sikkert og gir dårlig ytelse](https://gist.github.com/tmcw/6244497). Hvis IE8 eller IE9 må være supportert, bruk Microsoft's [XDomainRequest](http://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx?Redirected=true) objektet istedet for JSONP. Det er [biblioteker](https://github.com/mapbox/corslite) som kan hjelpe til med dette.


### Dokumentasjon

Et API er bare så god som sin dokumentasjon.

Dokumentasjon skal være enkelt å finne, og offentlig tilgjengelig.

Eksemplene bør inneholde fulle forespørsler, og svar.

## Inspirasjon

* [18F US federal government](https://github.com/18F/api-standards)
* [White House](https://github.com/WhiteHouse/api-standards)
* [Vinay Sahni](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)

## Lisens

Dette prosjektet er lisensiert som [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).

Alle bidrag til dette prosjektet vil bli lansert under CC0.
Ved å sende en pull request, godtar du å overholde
med denne fraskrivelse av opphavsrett.
