# official-product-data-sources

A catalog of official product registries (recalls, certifications) that AI agents can query directly, with test requests we have actually run.

| Registry | What you can check | API? | Key needed? | Verified |
|---|---|---|---|---|
| U.S. CPSC Recalls (SaferProducts.gov REST) | U.S. consumer product recalls: title, description, hazards, remedies, recall URL, product UPCs (rarely filled, see below) | Yes, REST (JSON) | No | 2026-10-18 |
| ENERGY STAR Certified Dehumidifiers (data.energystar.gov, dataset `mgiu-hu4z`) | Certified models, with brand, `model_number` and `upc` as separate fields | Yes, Socrata (JSON) | No | 2026-10-18 |
| ENERGY STAR Certified Products UPC Codes (data.energystar.gov, dataset `8edu-y555`) | UPC to certified model, across product categories | Yes, Socrata (JSON) | No | 2026-10-18 |
| ENERGY STAR dataset catalog (Socrata Discovery API) | List of all ENERGY STAR dataset IDs | Yes, REST (JSON) | No | 2026-10-18 |
| openFDA Food Enforcement Reports (api.fda.gov) | FDA food recall enforcement reports: product description, reason for recall, classification, status, recalling firm | Yes, REST (JSON) | No (optional key raises limits) | 2026-10-18 |
| NHTSA Recalls by vehicle (api.nhtsa.gov) | U.S. vehicle safety recalls for a given make, model and model year: campaign number, component, summary, remedy | Yes, REST (JSON) | No | 2026-10-18 |

"Verified" means we sent the test request below on that date and got HTTP 200 with records. It does not mean the data itself is complete or correct.

## Test requests

Each of these returned HTTP 200 without any key on 2026-10-18.

```bash
# CPSC recalls whose title matches a word
curl "https://www.saferproducts.gov/RestWebServices/Recall?format=json&RecallTitle=heater"

# ENERGY STAR certified dehumidifiers
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?\$limit=2"
# exact match by model number (certification record, not a recall)
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?model_number=AERD501P"

# ENERGY STAR certified products UPC codes
curl "https://data.energystar.gov/resource/8edu-y555.json?\$limit=2"
# exact match by UPC (certification record, not a recall)
curl "https://data.energystar.gov/resource/8edu-y555.json?upc=196388555765"

# All ENERGY STAR dataset IDs
curl "https://api.us.socrata.com/api/catalog/v1?domains=data.energystar.gov"

# CPSC recalls whose title matches "space heater"
curl "https://www.saferproducts.gov/RestWebServices/Recall?format=json&RecallTitle=space%20heater"

# ENERGY STAR dehumidifiers: total rows, and rows with a UPC
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?\$select=count(*)"
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?\$select=count(*)&\$where=upc%20IS%20NOT%20NULL"

# openFDA food enforcement reports (one record)
curl "https://api.fda.gov/food/enforcement.json?limit=1"

# NHTSA recalls for one vehicle (replace the placeholders)
curl "https://api.nhtsa.gov/recalls/recallsByVehicle?make=<make>&model=<model>&modelYear=<year>"
```

## A finding: CPSC recalls rarely carry UPCs

In our sample of 3 categories (dehumidifier, heater, charger), Oct 2026, only 3 of 249 CPSC recall records had the `ProductUPCs` field filled (about 1%, all in the heater results). The sample is the `RecallTitle` queries for those three words, which returned 18, 175 and 56 records. Model numbers usually appear only in the free-text `Description`.

A separate `RecallTitle=space heater` query on 2026-10-18 returned 19 recall records (22 product entries). None had `ProductUPCs` filled, and the structured `Products[].Model` field was empty in all 22 entries.

So matching a product to a CPSC recall usually means extracting and matching model numbers from text, while ENERGY STAR can be queried by exact model number or UPC, for certified products only.

## Other numbers we measured

- **ENERGY STAR dehumidifiers (`mgiu-hu4z`):** 565 rows in total, of which 288 have the `upc` field filled (about 51%), counted with the two `count(*)` queries above on 2026-10-18. So exact UPC lookup misses about half of the certified dehumidifier models.
- **openFDA food enforcement:** the test request reported 29,485 total records (`meta.results.total`, dataset `last_updated` 2026-10-07).
- **NHTSA:** a test query for one 2018 car model returned 6 recall records.

## Usage notes from the openFDA docs

- **Limits** ([openFDA authentication docs](https://open.fda.gov/apis/authentication/)): with no API key, 240 requests per minute and 1,000 requests per day, per IP address. With a key, 240 per minute and 120,000 per day, per key.
- **Not for public alerts** ([food enforcement docs](https://open.fda.gov/apis/food/enforcement/)): the docs say this data "should not be used as a method to collect data to issue alerts to the public, nor should it be used to track the lifecycle of a recall", and that "FDA does not update the status of a recall after the recall has been classified according to its level of hazard."

## What builders say

The team behind [Sift](https://devpost.com/software/sift-2rdezv), a project submitted to the USAII Global AI Hackathon 2026 (a hackathon self-description, not industry research), wrote:

> "Recall databases are noisy. The official FDA and CPSC databases match on keywords, so searching a common brand word surfaces recalls for completely unrelated products in totally different industries."

They added a verification step and drop matches they cannot confirm.

## Not yet verified

- **openFDA device and drug recall endpoints**: not tested yet (only food enforcement above was tested)
- **USDA FSIS recall API** (fsis.usda.gov): returned HTTP 403 on 2026-10-18
- **EU EPREL** (energy label database): returned HTTP 403 on 2026-10-18
- **Rate limits** for the CPSC, ENERGY STAR and NHTSA APIs: no rate-limit headers were returned. Unknown.

Not affiliated with CPSC, ENERGY STAR, FDA, NHTSA, USDA or any agency.

## Goal

Let agents check facts against official registries instead of product pages. Each answer should include the official source URL and the time of the query, say "not found" when nothing matches, and never judge product quality.

## Status

Prototype planned. No code yet; this repo is a catalog only.

## License

MIT
