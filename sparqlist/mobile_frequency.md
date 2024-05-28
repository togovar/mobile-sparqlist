# Mobile report / frequency

Report variant frequencies by populations in TogoVar specified by a Variant ID listed in the Parameters section.

## Parameters

* `variant_id` Variant ID
  * default:
  * dbSNP rsID Example: 
    * [rs671](/sparqlist/api/mobile_frequency?variant_id=rs671)
  * HGVSg (GRCh38) Example: 
    * [NC_000012.12:g.111803962G>A](/sparqlist/api/mobile_frequency?variant_id=NC_000012.12:g.111803962G>A)
    * [Chr12(GRCh38):g.111803962G>A](/sparqlist/api/mobile_frequency?variant_id=Chr12(GRCh38):g.111803962G>A)
  * HGVSg (GRCh37) Example: 
    * [NC_000012.11:g.112241766G>A](/sparqlist/api/mobile_frequency?variant_id=NC_000012.11:g.112241766G>A)
    * [Chr12(GRCh37):g.112241766G>A](/sparqlist/api/mobile_frequency?variant_id=Chr12(GRCh37):g.112241766G>A)
  * HGVSp Example
    * [ALDH2:p.Glu504Lys](/sparqlist/api/mobile_frequency?variant_id=ALDH2:p.Glu504Lys)
    * [ENSP00000261733.2:p.Glu504Lys](/sparqlist/api/mobile_frequency?variant_id=ENSP00000261733.2:p.Glu504Lys)
    * [ENSP00000403349.3:p.Glu457Lys](/sparqlist/api/mobile_frequency?variant_id=ENSP00000403349.3:p.Glu457Lys)
    * [NP_000681.2:p.Glu504Lys](/sparqlist/api/mobile_frequency?variant_id=NP_000681.2:p.Glu504Lys)
    * [NP_001191818.1:p.Glu457Lys](/sparqlist/api/mobile_frequency?variant_id=NP_001191818.1:p.Glu457Lys)
  * HGVSc Example: 
    * [ENST00000261733.7:c.1510G>A](/sparqlist/api/mobile_frequency?variant_id=ENST00000261733.7:c.1510G>A)
    * [ENST00000416293.7:c.1369G>A](/sparqlist/api/mobile_frequency?variant_id=ENST00000416293.7:c.1369G>A)
    * [ENST00000548536.1:c.*1386G>A](/sparqlist/api/mobile_frequency?variant_id=ENST00000548536.1:c.*1386G>A)
    * [ENST00000549106.1:c.*89G>A](/sparqlist/api/mobile_frequency?variant_id=ENST00000549106.1:c.*89G>A)
    * [NM_000690.4:c.1510G>A](/sparqlist/api/mobile_frequency?variant_id=NM_000690.4:c.1510G>A)
    * [NM_001204889.2:c.1369G>A](/sparqlist/api/mobile_frequency?variant_id=NM_001204889.2:c.1369G>A)
    * [NM_000690:c.1510G>A](/sparqlist/api/mobile_frequency?variant_id=NM_000690:c.1510G>A)
  * Example of variant recoder error:
    * [NC_000012.12:g.111803961G>A](/sparqlist/api/mobile_frequency?variant_id=NC_000012.12:g.111803961G>A)
    * [Null variant ID](/sparqlist/api/mobile_clinvar?variant_id=)
  * Example of successful variant recoder but no frequency record found:
    * [rs672](/sparqlist/api/mobile_frequency?variant_id=rs672)

## Response

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
  * Description: Allele frequencies in TogoVar

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

## `frequency`

```sparql
PREFIX dct:  <http://purl.org/dc/terms/>
PREFIX gvo:  <http://genome-variation.org/resource#>
PREFIX rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?source (GROUP_CONCAT(DISTINCT ?_filter ; separator = ", ") AS ?filter) ?quality ?info_label ?info_value
WHERE {
{{#if variant_recoder.is_recoded}}
  VALUES ?variant { <{{variant_recoder.togovar_uri}}> }

  GRAPH <http://togovar.org/variant> {
    ?variant dct:identifier ?tgv_id .
  }

  VALUES ?g {
    <http://togovar.org/variant/frequency/gem_j_wga>
    <http://togovar.org/variant/frequency/hgvd>
    <http://togovar.org/variant/frequency/jga_ngs>
    <http://togovar.org/variant/frequency/jga_snp>
    <http://togovar.org/variant/frequency/tommo>
    <http://togovar.org/variant/frequency/gnomad_genomes>
    <http://togovar.org/variant/frequency/gnomad_exomes>
  }

  GRAPH ?g {
    ?variant gvo:info [
      rdfs:label ?info_label ;
      rdf:value ?info_value
    ] .

    OPTIONAL { ?variant gvo:filter ?_filter . }
    OPTIONAL { ?variant gvo:qual ?quality . }

    BIND(REPLACE(STR(?g), ".*/", "") AS ?source)
  }
{{/if}}
}
ORDER BY ?source
```

## `results`
```javascript
({ variant_recoder, frequency }) =>  {
  if (frequency.results.bindings.length == 0 ||
     (Object.keys(frequency.results.bindings[0]).length == 1 && frequency.results.bindings[0].filter.value == "")){
    variant_recoder.results = [];
  } else {
    variant_recoder.results = frequency.results.bindings.map((d) => ({
      source: d.source ? d.source.value : null,
      filter: d.filter ? d.filter.value : null,
      quality: d.quality ? Number(d.quality.value) : null,
      info_label: d.info_label ? d.info_label.value : null,
      info_value: d.info_value ? Number(d.info_value.value) : null
    }));
  }

  return variant_recoder;
};
```