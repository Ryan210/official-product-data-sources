# official-product-data-sources

A catalog of official product registries (recalls, certifications) that AI agents can query directly, with test requests we have actually run.

| Registry | What you can check | API? | Key needed? | Verified |
|---|---|---|---|---|
| U.S. CPSC Recalls (SaferProducts.gov REST) | U.S. consumer product recalls: title, description, hazards, remedies, recall URL, product UPCs (rarely filled, see below) | Yes, REST (JSON) | No | 2026-10-18 |
| ENERGY STAR Certified Dehumidifiers (data.energystar.gov, dataset `mgiu-hu4z`) | Certified models, with brand, `model_number` and `upc` as separate fields | Yes, Socrata (JSON) | No | 2026-10-18 |
| ENERGY STAR Certified Products UPC Codes (data.energystar.gov, dataset `8edu-y555`) | UPC to certified model, across product categories | Yes, Socrata (JSON) | No | 2026-10-18 |
| ENERGY STAR dataset catalog (Socrata Discovery API) | List of all ENERGY STAR dataset IDs | Yes, REST (JSON) | No | 2026-10-18 |

"Verified" means we sent the test request below on that date and got HTTP 200 with records. It does not mean the data itself is complete or correct.

## Test requests

Each of these returned HTTP 200 without any key on 2026-10-18.

```bash
# CPSC recalls whose title matches a word
curl "https://www.saferproducts.gov/RestWebServices/Recall?format=json&RecallTitle=heater"

# ENERGY STAR certified dehumidifiers
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?\$limit=2"
# exact match by model number
curl "https://data.energystar.gov/resource/mgiu-hu4z.json?model_number=AERD501P"

# ENERGY STAR certified products UPC codes
curl "https://data.energystar.gov/resource/8edu-y555.json?\$limit=2"
# exact match by UPC
curl "https://data.energystar.gov/resource/8edu-y555.json?upc=196388555765"

# All ENERGY STAR dataset IDs
curl "https://api.us.socrata.com/api/catalog/v1?domains=data.energystar.gov"
```

## A finding: CPSC recalls rarely carry UPCs

In our sample of 3 categories (dehumidifier, heater, charger), Oct 2026, only 3 of 249 CPSC recall records had the `ProductUPCs` field filled (about 1%, all in the heater results). The sample is the `RecallTitle` queries for those three words, which returned 18, 175 and 56 records. Model numbers usually appear only in the free-text `Description`.

So matching a product to a CPSC recall usually means extracting and matching model numbers from text, while ENERGY STAR can be matched exactly by model number or UPC.

## What builders say

The team behind [Sift](https://devpost.com/software/sift-2rdezv), a project submitted to the USAII Global AI Hackathon 2026 (a hackathon self-description, not industry research), wrote:

> "Recall databases are noisy. The official FDA and CPSC databases match on keywords, so searching a common brand word surfaces recalls for completely unrelated products in totally different industries."

They added a verification step and drop matches they cannot confirm.

## Not yet verified

- **openFDA** (food enforcement reports, device recalls, api.fda.gov)
- **EU EPREL** (energy label database)
- **Rate limits** for the CPSC and ENERGY STAR APIs: no rate-limit headers were returned. Unknown.

## Goal

Let agents check facts against official registries instead of product pages. Each answer should include the official source URL and the time of the query, say "not found" when nothing matches, and never judge product quality.

## Status

Prototype in progress. No code yet; this repo is a catalog only.

## License

MIT
