# Samodejna obdelava zapisnikov sestankov s strankami

Rešitev praktične naloge za študentsko delo na področju AI in avtomatizacije poslovnih procesov.

Svetovalec naloži transkript sestanka. AI izlušči povzetek, zahteve stranke, ključna dejstva, naslednje korake z odgovornimi osebami in roke. Koda vsak podatek preveri s citatom iz transkripta, svetovalec ga pregleda in potrdi. Potrjeni podatki se zapišejo v poslovni sistem, AI pa pripravi osnutek follow-up sporočila.

```
Transkript → AI izlušči podatke → Samodejne kontrole → Pregled in potrditev → Zapis v sistem → Osnutek follow-upa
```

Celotna rešitev (odgovori na 6 vprašanj iz naloge): [`dokumenti/resitev_naloge.pdf`](dokumenti/resitev_naloge.pdf)

**Video prikaz:** https://www.loom.com/share/97061bf735794e00bfe2f44cd3b45f4c

## Vsebina

| Mapa | Kaj je v njej |
|---|---|
| `workflow/povzetki_sestankov.json` | n8n workflow za uvoz |
| `workflow/code_nodes/` | Koda vseh Code node-ov kot ločene datoteke (enaka kot v workflowu) |
| `prompt/` | Prompt za ekstrakcijo in JSON shema izhoda |
| `validacija/` | Samodejne kontrole (`validate.js`) s testi in pravim izhodom Gemini (`fixtures/`) |
| `testni_podatki/` | Testni transkript z 9 pastmi, pričakovan izhod, seznam zaposlenih, označen transkript (.docx) |
| `dokumenti/` | Rešitev naloge (PDF) |

## Code node-i

| Datoteka | Node | Naloga |
|---|---|---|
| `1_preberi_transkript.js` | Preberi transkript | Prebere .txt (UTF-8 ali Windows-1250), izračuna datum sestanka, pripravi prompt |
| `3_samodejno_preverjanje.js` | 3. Samodejno preverjanje | Validacija AI izhoda in HTML strani za pregled |
| `5_uporabi_odlocitev.js` | 5. Uporabi odločitev | Uporabi odločitev iz pregleda, ponovno validira, pripravi zapis in vhod za follow-up |
| `7_koncni_pregled_osnutka.js` | Končni pregled osnutka | Preveri, da osnutek ne omenja internih tem, in pripravi stran z rezultatom |

Node-a 3 in 5 vsebujeta funkcije iz `validacija/validate.js`, ker n8n Code node ne more uvažati lokalnih datotek.

## Zagon

1. V n8n ustvari Data table `crm_notes` s stolpci `client_company`, `meeting_date`, `summary`, `requirements`, `key_facts`, `open_questions`, `tasks`, `approved_by` (vsi *string*).
2. Uvozi `workflow/povzetki_sestankov.json`.
3. V node-ih **Gemini** izberi credential in model, v node-u **6. Zapiši v poslovni sistem** tabelo `crm_notes`.
4. Objavi workflow in odpri `http://localhost:5678/form/sestanek`.
5. Naloži `testni_podatki/transkript_sestanek.txt`.

## Testi validacije

```bash
node validacija/test_validate.js
```

| Primer | Opis | Pričakovano |
|---|---|---|
| GOLD | Pravilen izhod | 0 napak, 2 opozorili (enako ime »Luka«, naloga brez lastnika) |
| BAD | Tipične napake AI | Blokirano – izmišljen citat, izmišljen rok, napačen datum, napačna oseba … |
| HEDGED | Negotov podatek, označen kot potrjen | Opozorilo |
| HUMAN | Ročni popravek pri pregledu | Sprejet brez citata, označen kot ročni |
| GEMINI_RUN_4 | Pravi izhod Gemini | 0 napak, 2 pravilni opozorili |

## Opombe

- **Seznam zaposlenih** je za demo zapisan v kodi. V produkciji bi prišel iz kadrovskega sistema ali CRM prek API.
- **Poslovni sistem** v prototipu simulira n8n Data table. V produkciji ta korak kliče API sistema.
- **API ključi** niso v repozitoriju. Credentials so shranjeni v n8n.
