# Mobile report / variant_recoder

Look up a URI of a variant in TogoVar by a HGVS expression or a dbSNP rsID by using [Ensembl Variant Recoder](https://rest.ensembl.org/documentation/info/variant_recoder). An example of a GET request to the Variant Recoder is [https://rest.ensembl.org/variant_recoder/human/ALDH2:p.Glu504Lys?content-type=application/json](https://rest.ensembl.org/variant_recoder/human/ALDH2:p.Glu504Lys?content-type=application/json).

## Parameters

* `variant_id` Variant ID
  * default:
  * dbSNP rsID example: 
    * [rs671](/sparqlist/api/variant_recoder?variant_id=rs671)
  * HGVSg (GRCh38) example: 
    * [NC_000012.12:g.111803962G>A](/sparqlist/api/variant_recoder?variant_id=NC_000012.12:g.111803962G>A)
    * [Chr12(GRCh38):g.111803962G>A](/sparqlist/api/variant_recoder?variant_id=Chr12(GRCh38):g.111803962G>A)
  * HGVSg (GRCh37) example: 
    * [NC_000012.11:g.112241766G>A](/sparqlist/api/variant_recoder?variant_id=NC_000012.11:g.112241766G>A)
    * [Chr12(GRCh37):g.112241766G>A](/sparqlist/api/variant_recoder?variant_id=Chr12(GRCh37):g.112241766G>A)
  * HGVSp example
    * [ALDH2:p.Glu504Lys](/sparqlist/api/variant_recoder?variant_id=ALDH2:p.Glu504Lys)
    * [ENSP00000261733.2:p.Glu504Lys](/sparqlist/api/variant_recoder?variant_id=ENSP00000261733.2:p.Glu504Lys)
    * [ENSP00000403349.3:p.Glu457Lys](/sparqlist/api/variant_recoder?variant_id=ENSP00000403349.3:p.Glu457Lys)
    * [NP_000681.2:p.Glu504Lys](/sparqlist/api/variant_recoder?variant_id=NP_000681.2:p.Glu504Lys)
    * [NP_001191818.1:p.Glu457Lys](/sparqlist/api/variant_recoder?variant_id=NP_001191818.1:p.Glu457Lys)
  * HGVSc example: 
    * [ENST00000261733.7:c.1510G>A](/sparqlist/api/variant_recoder?variant_id=ENST00000261733.7:c.1510G>A)
    * [ENST00000416293.7:c.1369G>A](/sparqlist/api/variant_recoder?variant_id=ENST00000416293.7:c.1369G>A)
    * [ENST00000548536.1:c.*1386G>A](/sparqlist/api/variant_recoder?variant_id=ENST00000548536.1:c.*1386G>A)
    * [ENST00000549106.1:c.*89G>A](/sparqlist/api/variant_recoder?variant_id=ENST00000549106.1:c.*89G>A)
    * [NM_000690.4:c.1510G>A](/sparqlist/api/variant_recoder?variant_id=NM_000690.4:c.1510G>A)
    * [NM_001204889.2:c.1369G>A](/sparqlist/api/variant_recoder?variant_id=NM_001204889.2:c.1369G>A)
    * [NM_000690:c.1510G>A](/sparqlist/api/variant_recoder?variant_id=NM_000690:c.1510G>A)
  * Convert error example:
    * [NC_000012.12:g.111803961G>A](/sparqlist/api/variant_recoder?variant_id=NC_000012.12:g.111803961G>A)

## `is_grch37` 
```javascript
async ({variant_id}) => {
  if(variant_id.includes("GRCh37")){
    return true;
  }

  const regex = /NC_0+\d+\.(\d+):g\.\d+[A-Z]>[A-Z]/;
  const match = variant_id.match(regex);
  if(match && match[1] == "11"){
    return true;
  }

  return false;
}
```

## `result` 
```javascript
async ({variant_id, is_grch37}) => {
  try {
    const variant_recoder_fqdn = is_grch37 ? "grch37.rest.ensembl.org" : "rest.ensembl.org";
    const variant_recoder_url = 
      "https://" + variant_recoder_fqdn + "/variant_recoder/human/" + variant_id + "?content-type=application/json";
    const response = await fetch(variant_recoder_url);

    if (response.ok) { 
      const data = await response.json();
      const hgvsg = data[0].A.hgvsg[0];
      const regex = /NC_0+(\d+)\.\d+:g\.(\d+)([A-Z])>([A-Z])/;
      const match = hgvsg.match(regex);
      chr = match[1];
      pos = match[2];
      ref = match[3];
      alt = match[4];

      const ref_genome = is_grch37 ? "GRCh37" : "GRCh38";
      const togovar_uri = 
        "http://identifiers.org/hco/" + chr + "/" + ref_genome + "#" + pos + "-" + ref + "-" + alt;

      const result_json = {
        is_recoded: true,
        togovar_uri: togovar_uri,
        is_grch37: is_grch37
      };

      if (data.hasOwnProperty("warnings")){
        result_json.warnings = data[0].warnings[0];
      }

      return JSON.stringify(result_json);
    } else {
      const data = await response.json();
      const error = data.hasOwnProperty("error") ? data.error : null;
      const result_json = {
        is_recoded: false,
        error: error
      };

      return JSON.stringify(result_json);
    }
  } catch (error) {
    const result_json = {
      is_recoded: false,
      error: error
    };

    return JSON.stringify(result_json);
  }
}
```
