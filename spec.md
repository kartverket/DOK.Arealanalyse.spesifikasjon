# Teknisk dokumentasjon - OGC Processes API for DOK Arealanalyse
[OGC API Processes](https://ogcapi.ogc.org/processes/) er valgt brukt som rammene til DOK analysen. 

## Landingsside
GET /

Respons:
```json
{
"links": [
{
"rel": "self",
"type": "application/json",
"title": "This document as JSON",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443?f=json"
},
{
"rel": "alternate",
"type": "application/ld+json",
"title": "This document as RDF (JSON-LD)",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443?f=jsonld"
},
{
"rel": "alternate",
"type": "text/html",
"title": "This document as HTML",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443?f=html",
"hreflang": "en-US"
},
{
"rel": "service-desc",
"type": "application/vnd.oai.openapi+json;version=3.0",
"title": "The OpenAPI definition as JSON",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/openapi"
},
{
"rel": "service-doc",
"type": "text/html",
"title": "The OpenAPI definition as HTML",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/openapi?f=html",
"hreflang": "en-US"
},
{
"rel": "conformance",
"type": "application/json",
"title": "Conformance",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/conformance"
},
{
"rel": "data",
"type": "application/json",
"title": "Collections",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/collections"
},
{
"rel": "http://www.opengis.net/def/rel/ogc/1.0/processes",
"type": "application/json",
"title": "Processes",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/processes"
},
{
"rel": "http://www.opengis.net/def/rel/ogc/1.0/job-list",
"type": "application/json",
"title": "Jobs",
"href": "https://dok-arealanalyse-api.azurewebsites.net:443/jobs"
}
],
"title": "OGC API-tjenester - DOK-analyse",
"description": "OGC API er en ny generasjon standarder fra Open Geospatial Consortium. På denne serveren tilbys testimplementasjoner av Arealanalyse"
}
```

* processID = dok-analyse
* Kjør analyse: POST /processes/{processID}/execution
    * requestBody = [no.geonorge.dokanalyse.analysisinput.v0.1.schema.json](schema/no.geonorge.dokanalyse.analysisinput.v0.1.schema.json)
    * [Eksempel forespørsel](schema/sampledata/request.json)
    * Hvis modus er synkron kommer resultatet som respons
        * respons = [no.geonorge.dokanalyse.analysisresponse.v0.1.schema.json](schema/no.geonorge.dokanalyse.analysisresponse.v0.1.schema.json)
        * [Eksempel resultat](schema/sampledata/result1.json)
    * Hvis asynkron modus vil respons bli jobid
* Hvis asynkron med jobid så må en sjekke status og vente på at jobben er ferdig
    * Status: GET /jobs/{jobId}
    * Resultat: GET /jobs/{jobId}/results
        * respons = [no.geonorge.dokanalyse.analysisresponse.v0.1.schema.json](schema/no.geonorge.dokanalyse.analysisresponse.v0.1.schema.json)
        * [Eksempel resultat](schema/sampledata/result1.json)

## Beskrivelse av datamodell for input og output

![Datamodell for forespørsel og respons!](Arealanalyse.png)

### AnalysisInput

| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| InputGeometry      | GM_Surface       |  1 | område | Området en ønsker å analysere mot. Kan feks være en eiendom eller planområde.
| requestedBuffer   | Integer        | 0..1 | ønskeBuffer | Antall meter som legges på InputGeometry som buffer i analysen.  Kan utelates og avgjøres av analysen hva som er fornuftig buffer.
| context | String  | 0..1  | kontekst  | hint om hva analysen skal brukes til. Feks planinitiativ, reguleringsplan, kommuneplan, byggesak, ros analyse, konsekvensutredning slik at relevante/egnede datasett blir brukt og de riktige analyser/generaliseringer blir brukt. Kontekst kan også brukes for å hente ut riktig veiledningstekster fra geolett registeret(i geolett 1 ble det lagt inn byggesak som kontekst for veiledninger).
| theme | String  | 0..1  | tema  | dok tema kan angis for å begrense analysen til aktuelle tema.
| includeGuidance | Boolean  | 0..1  | inkluderVeiledning  | angir om veiledningstekster skal inkluderes i resultat om det finnes i geolett. Kan være avhengig av å styres med context for å få riktige tekster.
| includeQualitymeasurement | Boolean  | 0..1  | inkluderKvalitetsinformasjon  | angir om kvalitetsinformasjon skal taes med i resultatet der det er mulig, slik som dekningskart, egnethet, nøyaktighet, etc.
| includeFilterChosenDOK | Boolean  | 0..1  | inkluderFilterValgtDOK  | angir filter med  bare datasett som kommune har valgt inn i Det Offentlige Kartgrunnlag (DOK).
| includeFacts | Boolean  | 0..1  | inkluderFakta  | angir om fakta informasjon skal inkluderes i resultatet.

### AnalysisOutput

| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| resultList      | [Result](#result)       |  0..* | resultatListe | strukturert resultat på analysen.
| report        | Binary    | 0..1 | rapport | Rapporten levert som pdf (tilsvarende funksjonalitet som før for å dokumentere resultatet).
| inputGeometry        | GM_Surface    | 0..1 | område | valgt område for analyse.
| inputGeometryArea | Integer    | 0..1  | områdeareal   | beregner arealet i kvm på valgte område for analyse.
| factSheetRasterResult      | Binary    | 0..1  | kartutsnitt   | wms url for å hente rasterbilde for området. 
| factSheetCartography       | Binary    | 0..1  | tegnforklaring   | wms url for å hente tegneregler for resultatet.
| factList      | [FactPart](#factpart)       |  0..* | oppsummeringListe | oppsummering av deler til faktaark.
| municipalityNumber      | String    | 0..1  | kommunenummer   | Kommunenummer som kan brukes til kommunens valgte DOK og i fakta visning.
| municipalityName     | String    | 0..1  | kommunenavn   |  Kommunenavn som kan brukes til kommunens valgte DOK og i fakta visning.

#### Result

| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| runAlgorithm      | String       |  1..* |  algoritmeKjørt | beskriver hvilken analyse som er kjørt og hvordan denne er satt sammen.
| buffer            | Integer   | 0..1  | buffer        | Buffer i antall meter som er brukt rundt området.
| runOnInputGeometry        | GM_Surface    | 0..1 | område | område analysen er kjørt mot.
| runOnDataset      | [Dataset](#dataset)   | 1 | kjørtPåDatasett   | Beskrivelse av datasett analysen er kjørt mot.
| inputGeometryArea | Integer    | 0..1  | områdeareal   | beregner arealet i kvm på valgte område for analyse.
| hitArea | Integer    | 0..1  | treffareal   | beregner arealet i kvm på datasett som treffer innenfor analyseområdet.
| resultStatus      | DatasetResult | 1 | statusResultat    | resultat av analysen om det er treff eller ikke treff. (HIT, NO-HIT, HIT-RED, HIT-YELLOW, NO-HIT-YELLOW, NO-HIT-GREEN)
| possibleActions   | String    | 0..*  | muligeTiltak  | liste over mulige tiltak. Kan være hentet fra Geolett register.
| rasterResult      | Binary    | 0..1  | kartutsnitt   | rasterbilde av data/resultat. Vurdere WMS referanse isteden for base64 string?
| cartography       | Binary    | 0..1  | tegnforklaring   | viser tegneregler for resultatet. Vurdere WMS referanse isteden for base64 string?
| guidanceText      | String    | 0..1  | veiledningsTekst   | Kan være hentet fra Geolett register.
| guidanceUri       | [Uri](#Uri)    | 0..*  | veiledningslenke   | Veiledningsreferanser. Kan være hentet fra Geolett register.
| description       | String    | 0..1  | forklarendeTekst   | beskrivelse av resultat. Kan være hentet fra Geolett register.
| distanceToObject  | Integer   | 0..1  | avstandTilObjekt  | nærmeste avstand fra område til objekt. Mest relevant om det ikke er treff innenfor området som forespørres. Anbefaling 83 i teknologisk rammeverk.
| theme             | String    | 0..1  | tema   | DOK tema for datasettet.
| qualityMeasurement | [QualityMeasurement](#qualitymeasurement) | 0..* | kvalitetsmåling | liste over relevante kvaliteter slik som fullstendighet (dekningskart), egnethet (fra DOK), etc
| qualityWarning   | String   | 0..*   | kvalitetsadvarsel   | liste over kvalitetsvarsler fra kvalitetsindikatorer som kommer over angitt grense i sin konfigurasjon.
| data              | Any        | 0..1  | data   | mulighet for å returnere lister med data med mer informasjon om treffene i analysen.

#### Dataset

| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| datasetid      | String   |  1 |  datasettid | datasett id fra geonorge kartkatalog.
| title          | String   | 1  | tittel        | tittel på datasett.
| description          | String   | 0..1  | beskrivelse        | beskrivelse av datasett.
| owner          | String   | 1  | eier        | informasjon om den som eier og forvalter datasett.
| updated          | String   | 1  | datasettOppdatert        | data eller metadata oppdatert? begge?
| datasetDescriptionUri          | String   | 0..1  | datasettbeskrivelseLenke        | lenke til ytterligere beskrivelse av datasett.

#### QualityMeasurement

| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| qualityDimensionId      | String   |  1 |  kvalitetsdimensjonId | kvalitetsdimensjoner slik som fullstendighet_dekningskart, egnethet_reguleringsplan, etc.
| qualityDimensionName     | String   |  1 |  kvalitetsdimensjonNavn | kvalitetsdimensjoner i klart språk.
| value          | String   | 1  | verdi        | verdi for kvalitetsdimensjon.
| comment          | String   | 0..1  | kommentar        | kommentar til kvalitetsdimensjon.

#### Uri
| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| href      | String   |  1 |  lenke | 
| title          | String   | 0..1  | tittel        | 

#### Factpart
| Navn      | Type | Multiplisitet | Alias | Beskrivelse |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| runAlgorithm      | String       |  1..* |  algoritmeKjørt | beskriver hvilken analyse som er kjørt og hvordan denne er satt sammen.
| buffer            | Integer   | 0..1  | buffer        | Buffer i antall meter som er brukt rundt området.
| runOnInputGeometry        | GM_Surface    | 0..1 | område | område analysen er kjørt mot.
| runOnDataset      | [Dataset](#dataset)   | 1 | kjørtPåDatasett   | Beskrivelse av datasett analysen er kjørt mot.
| data              | Any        | 0..1  | data   | mulighet for å returnere lister med data som gir innsikt i fakta om analyseområdet.
| dataTemplate              | String        | 0..1  | datamal   | mulighet for å levere en mal for presentasjon av data.
