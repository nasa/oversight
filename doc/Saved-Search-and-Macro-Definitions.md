# Saved Search and Macro Definitions

This document describes the format and syntax of saved searches generated by Oversight.


## Sample 

given a saved input name of `syslog`, we may get a saved search definition similar to:

```
`syslog_source` 
| `eval_last_inventoried` 
| `syslog_enrichment_expression` 
| `set_id(src,ip)` 
| `sort_dedup(ip)` 
| `set_key(ip)` 
| `syslog_source_filter`
| `set_not_expired` 
| table `syslog_fields` 
| `outputlookup(syslog_lookup)` 
| `syslog_inventory_filter`
```
and the alert action configured for the saved search is equivalent to:
```
| sendalert update_inventory param.source_name=syslog
```

Note that leading and trailing `|` are inserted into the search between the macro names as needed.
Note that the macros names that begin with `syslog` are specific to this saved search.  
Macro names which do not being with `syslog` are shared by all generated saved search definitions.

| macro name | purpose | input parameter | example input parameter value| condition |
| --- | --- | --- | --- | --- |
| `` `syslog_source` `` | Initial search expression(s) which retreive the desired events | `source_expression` | index=main sourcetype=syslog | always |
| `` `syslog_enrichment_expression` `` | additional search expressions(s) to enrich or normalize the events | `enrichment_expression` | ` eventstats max(_time) as most_recent_time \| eval status=if(most_recent_time >= relative_time(now(), "-24h@h"), "good", "bad")` | if `enrichment_expresion` is specified |
| `` `set_id(src,ip` `` | creates a field alias for the first field, named the second field and uses the 2nd field as the lookup record key. | `id_field_rename` | `id_field = src, id_field_rename=ip` | if `id_field_rename` is specified |
| `` `sort_dedup(ip)` `` | performs `dedup` on the key field | uses `id_field_rename` if specified, otherwise uses `id_field` | `id_field_rename=ip` | if `id_field_rename` is specified | 
| `` `set_key(ip)` `` | creates a field named `_key` which is used as record key for the lookup table |  uses `id_field_rename` if specified, otherwise uses `id_field` , also strips illegal character | `id_field_rename=ip` | always |
| `` `syslog_source_filter` `` | filters events written to the lookup for this input source | `source_filter` | `search ip=1* AND ip!=172.17.0.* AND ip!=192.168.168.*` | if `source_filter` is specified |
| `` `set_not_expired` `` | records must be set to `expired=false` so they are not filtered by the lookups | -- | -- | always |
| table `` `syslog_fields` `` | ensures consistent set of fields and no extreanous fields are stored in lookup | derived from `` `id_field_rename` `` or `` `id_field` ``, `` `source_fields` `` and `` `enrichment_fields` ``| `source_fields=ip, hostname; enrichment_fields=customer_name, SLA` | always |
| `` `outputlookup(syslog_lookup)` `` | ensures that `outputlookup` specifies `the key_field` parameter | -- | -- | always |
| `` `syslog_inventory_filter` `` | filters events which are sent to the `update_inventory` alert action (in addition to any source filter from earlier in the pipeline) | `inventory_filter` and `inventory_source` | `search ip!=10.0.0.0` | if `inventory_filter` is specified and `inventory_source` is checked |
| `sendalert update_inventory param.source_name=syslog` | aggregates summary info into the aggregation lookup | `inventory_source` checked | -- | if `inventory_source` is checked |



Copyright © 2020 United States Government as represented by the Administrator of the National Aeronautics and Space Administration.  All Other Rights Reserved.