# IFC-verktøy – 4D-planlegger og IFC-leser

To nettleserverktøy for å lese IFC-modeller, ta ut mengder og koble modellen til fremdriftsplanen (4D).

| Side | Hva den gjør |
|---|---|
| `index.html` | Startside med lenker til verktøyene |
| `4d.html` | **4D-planlegger**: IFC-filer + plan fra MS Project, kobling av aktiviteter til elementer, avspilling av byggeperioden, mengder per aktivitet, lagring til 4D-fil |
| `ifc-leser.html` | **IFC-leser**: 3D-visning, filtre på etasje og elementtype, egenskaper og mengdeuttak til Excel |

Alt kjører i brukerens nettleser. Filer som åpnes, lastes **ikke** opp. Serveren leverer bare de statiske filene i dette repoet.

## Publisere med GitHub Pages

1. Last opp innholdet i denne mappen til roten av et repo, med `index.html` øverst.
2. Gå til **Settings › Pages** i repoet.
3. Under *Build and deployment* velger du **Deploy from a branch**, branch `main` og mappe `/ (root)`. Trykk **Save**.
4. Etter et minutt eller to ligger siden på `https://<bruker-eller-org>.github.io/<repo>/`.

Fila `.nojekyll` sørger for at GitHub serverer alle filer som de er.

> **Synlighet:** Med en vanlig GitHub-konto er Pages-sider offentlige på internett, også fra private repoer. Selve verktøyet inneholder ingen prosjektdata, så det er uproblematisk. **Ikke** legg prosjektfiler (IFC, planer, 4D-filer) i `samples/` i et offentlig repo. Skal tilgangen begrenses til Veidekke, kreves GitHub Enterprise med privat Pages, eller hosting på intern webserver eller i Azure Static Web Apps. Filene er de samme.

## Kjøre lokalt

Sidene må serveres over http, ikke åpnes direkte som fil (`file://`), fordi nettleseren laster WebAssembly og moduler fra mappen:

```
python -m http.server 8000
```

Åpne så `http://localhost:8000`.

## Eksempeldata (valgfritt)

Knappen **Eksempel** vises bare hvis disse filene finnes:

- `samples/eksempel.4d.json`: en lagret 4D-fil med IFC-filene innebygd (4D-planleggeren)
- `samples/eksempel.ifc`: en IFC-fil (IFC-leseren)

Bruk dette bare på intern hosting eller i private repoer med privat Pages.

## Bruk av 4D-planleggeren

1. **Legg til IFC**: én eller flere modeller (ARK, RIB, VVS). De legges i samme koordinatsystem.
2. **Importer fremdriftsplan**:
   - MS Project: *Fil › Lagre som › XML-format (\*.xml)*. Dette anbefales, fordi hierarki og Unik ID blir med.
   - Excel (.xlsx) eller CSV med kolonnene *Navn*, *Start* og *Slutt*, gjerne også *Unik ID* og *Disposisjonsnivå*.
   - `.mpp` kan ikke leses direkte. Det er MS Projects lukkede format.
3. **Koble**: Klikk en aktivitet og velg elementer i modellen. Du kan klikke dem (Ctrl for flere), eller filtrere og bruke «Velg synlige». Trykk så **Koble valgte**. **Foreslå** og **Autokoble** leser etasje og elementtype ut fra aktivitetsnavnet, for eksempel «Vegger U2».
4. **Se koblinger**: Klikk en aktivitet, så blir de koblede elementene blå. Klikk et element, så blir aktivitetene det hører til markert i rødt.
5. **Spill av**: ▶ eller mellomromstasten. Pågår vises i oransje. Ikke startet kan skjules eller vises som skygge.
6. **Lagre**: **Lagre** (Ctrl+S) og **Lagre som …** skriver en `.4d.json`-fil på din PC. I Chrome og Edge lagres det direkte til samme fil. I andre nettlesere lastes fila ned. **Åpne 4D-fil** fortsetter der du slapp.

### Om 4D-fila

- JSON med fremdriftsplan, aktivitetstyper (bygges, rives, midlertidig), innstillinger og koblinger.
- Koblingene lagres på elementenes **IFC GlobalId**, som følger elementet mellom eksporter fra Archicad. Elementer som slettes og tegnes på nytt får ny GlobalId og må kobles på nytt.
- IFC-filene kan bakes inn (gzip-komprimert). Da åpnes alt med én fil. Hvis de ikke bakes inn, må de samme IFC-filene legges til etter åpning.
- Importeres en oppdatert plan, beholdes koblingene. Aktivitetene matches på navn, deretter på Unik ID.

## Mappestruktur

```
index.html          startside
4d.html             4D-planlegger
ifc-leser.html      IFC-leser
vendor/             tredjepartsbiblioteker (se THIRD_PARTY.md)
samples/            valgfrie eksempelfiler (tom i repoet)
.nojekyll           GitHub Pages: server filene uendret
```

## Oppdatere biblioteker

Bibliotekene ligger låst i `vendor/`. Ved oppgradering byttes filene ut med samme navn:

- `vendor/three/three.module.js` og `vendor/three/addons/controls/OrbitControls.js` fra npm-pakken `three`
- `vendor/web-ifc/web-ifc-api.js` og `web-ifc.wasm` fra npm-pakken `web-ifc` (versjonene må høre sammen)
- `vendor/xlsx/xlsx.full.min.js` fra npm-pakken `xlsx`

Test import av IFC, plan og lagring etter hver oppgradering.
