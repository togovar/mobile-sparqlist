# Mobile report / ClinVar

Report clinical significance in ClinVar for a variant that is specified by a Variant ID listed in the Parameters section.

## Parameters

* `variant_id` Variant ID
  * default:
  * dbSNP rsID example: 
    * [rs671](/sparqlist/api/mobile_clinvar?variant_id=rs671)
  * HGVSg (GRCh38) example: 
    * [NC_000012.12:g.111803962G>A](/sparqlist/api/mobile_clinvar?variant_id=NC_000012.12:g.111803962G>A)
    * [Chr12(GRCh38):g.111803962G>A](/sparqlist/api/mobile_clinvar?variant_id=Chr12(GRCh38):g.111803962G>A)
  * HGVSg (GRCh37) example: 
    * [NC_000012.11:g.112241766G>A](/sparqlist/api/mobile_clinvar?variant_id=NC_000012.11:g.112241766G>A)
    * [Chr12(GRCh37):g.112241766G>A](/sparqlist/api/mobile_clinvar?variant_id=Chr12(GRCh37):g.112241766G>A)
  * HGVSp example
    * [ALDH2:p.Glu504Lys](/sparqlist/api/mobile_clinvar?variant_id=ALDH2:p.Glu504Lys)
    * [ENSP00000261733.2:p.Glu504Lys](/sparqlist/api/mobile_clinvar?variant_id=ENSP00000261733.2:p.Glu504Lys)
    * [ENSP00000403349.3:p.Glu457Lys](/sparqlist/api/mobile_clinvar?variant_id=ENSP00000403349.3:p.Glu457Lys)
    * [NP_000681.2:p.Glu504Lys](/sparqlist/api/mobile_clinvar?variant_id=NP_000681.2:p.Glu504Lys)
    * [NP_001191818.1:p.Glu457Lys](/sparqlist/api/mobile_clinvar?variant_id=NP_001191818.1:p.Glu457Lys)
  * HGVSc example: 
    * [ENST00000261733.7:c.1510G>A](/sparqlist/api/mobile_clinvar?variant_id=ENST00000261733.7:c.1510G>A)
    * [ENST00000416293.7:c.1369G>A](/sparqlist/api/mobile_clinvar?variant_id=ENST00000416293.7:c.1369G>A)
    * [ENST00000548536.1:c.*1386G>A](/sparqlist/api/mobile_clinvar?variant_id=ENST00000548536.1:c.*1386G>A)
    * [ENST00000549106.1:c.*89G>A](/sparqlist/api/mobile_clinvar?variant_id=ENST00000549106.1:c.*89G>A)
    * [NM_000690.4:c.1510G>A](/sparqlist/api/mobile_clinvar?variant_id=NM_000690.4:c.1510G>A)
    * [NM_001204889.2:c.1369G>A](/sparqlist/api/mobile_clinvar?variant_id=NM_001204889.2:c.1369G>A)
    * [NM_000690:c.1510G>A](/sparqlist/api/mobile_clinvar?variant_id=NM_000690:c.1510G>A)
  * Example of variant recoder error
    * [NC_000012.12:g.111803961G>A](/sparqlist/api/mobile_clinvar?variant_id=NC_000012.12:g.111803961G>A)
    * [Null variant ID](/sparqlist/api/mobile_clinvar?variant_id=)
  * Example of successful variant recoder but no ClinVar record found
    * [rs672](/sparqlist/api/mobile_clinvar?variant_id=rs672)

## Response

### Description of attributes
* Attribute: is_recoded
  * Value: true or false
  * Description Is variant recoder successfully completed?
* Attribute: is_grch37
  * Value: true or false
  * Description: Is genomic position given by GRCh37 coordinates?
* Attribute: error
  * Value: JSON
  * Description: Error message from the variant recoder
* Attribute: results
  * Value: JSON
  * Description: clinical significance in ClinVar

### Example of successful response
```
{
  "is_recoded": true,
  "togovar_uri": "http://identifiers.org/hco/12/GRCh38#111803962-G-A",
  "is_grch37": false,
  "results": [
    {
      "title": "NM_000690.4(ALDH2):c.1510G>A (p.Glu504Lys)",
      "review_status": "reviewed by expert panel",
      "interpretation": "Pathogenic",
      "last_evaluated": "2010-02-01",
      "condition": "AMED syndrome, digenic",
      "medgen": "C5436906",
      "clinvar": "http://ncbi.nlm.nih.gov/clinvar/variation/18390",
      "vcv": "VCV000018390"
    }
  ]
}
```
### Example of error
```
{
  "is_recoded": false,
  "error": "variant ID is a required parameter for this endpoint",
  "results": []
}
```

## `variant_recoder`
```javascript
async ({variant_id}) => {
  const URL = "/sparqlist/api/variant_recoder?variant_id=" + variant_id;
  try {
    const response = await fetch(URL);
    const response_json = await response.json();
    return response_json;
} catch (error){
    console.log(error);
  }
}
```

## Endpoint

{{#if variant_recoder.is_grch37}}
https://grch37.togovar.org/sparql
{{else}}
https://grch38.togovar.org/sparql
{{/if}}

## `clinvar`

```sparql
DEFINE sql:select-option "order"

PREFIX cvo:  <http://purl.jp/bio/10/clinvar/>
PREFIX dct:  <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?title ?review_status ?interpretation ?last_evaluated ?condition ?medgen ?clinvar ?vcv
WHERE {
{{#if variant_recoder.is_recoded}}
  VALUES ?variant { <{{variant_recoder.togovar_uri}}> }

  GRAPH <http://togovar.org/variant> {
    ?variant dct:identifier ?tgv_id .
  }

  GRAPH <http://togovar.org/variant/annotation/clinvar> {
    ?variant dct:identifier ?variation_id .

    BIND(IRI(CONCAT("http://ncbi.nlm.nih.gov/clinvar/variation/", ?variation_id)) AS ?clinvar)
  }

  GRAPH <http://togovar.org/clinvar> {
    ?clinvar a cvo:VariationArchiveType ;
      rdfs:label ?title ;
      cvo:accession ?vcv ;
      cvo:interpreted_record/cvo:review_status ?review_status ;
      cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession ?_rcv .

    ?_rcv cvo:interpretation ?interpretation ;
      cvo:date_last_evaluated ?last_evaluated ;
      cvo:interpreted_condition_list/cvo:interpreted_condition ?_interpreted_condition .

    ?_interpreted_condition rdfs:label ?condition ;
      dct:source ?db ;
      dct:identifier ?medgen .
    FILTER(?db IN ("MedGen"))
  }
{{/if}}
}
ORDER BY ?title ?review_status ?interpretation DESC(?last_evaluated) ?condition
```

## `results`
```javascript
({ variant_recoder, clinvar }) =>  {
  if (clinvar.results.bindings.length == 0 || Object.keys(clinvar.results.bindings[0]).length == 0){
    variant_recoder.results = [];
  } else {
    variant_recoder.results = clinvar.results.bindings.map((d) => ({
      title: d.title.value,
      review_status: d.review_status.value,
      review_status_stars: (function() {
        switch(d.review_status.value) {
          case "no assertion provided":
            return 0;
          case "no assertion criteria provided":
            return 0;
          case "no assertion for the individual variant":
            return 0;
          case "criteria provided, single submitter":
            return 1;
          case "criteria provided, conflicting interpretations":
            return 1;
          case "criteria provided, multiple submitters, no conflicts":
            return 2;
          case "reviewed by expert panel":
            return 3;
          case "practice guideline":
            return 4;
          default:
        }
      })(),
      interpretation: d.interpretation.value,
      last_evaluated: d.last_evaluated.value,
      condition: d.condition.value,
      medgen: d.medgen.value,
      clinvar: d.clinvar.value
    }));
  }

  return variant_recoder;
};
```