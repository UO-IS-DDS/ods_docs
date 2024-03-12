# (MDS) Modern Data Service

## Manifest

``` mermaid
graph LR

  extract --> Oracle
  subgraph targets [ ]
    Cognos --- Onedrive
  end
  secure --> targets
  load --> targets
  subgraph connect
    Oracle
    targets
  end
  subgraph domain
    Security --- Common
    Common --- Registrar
  end 
  Oracle --> domain
  targets --> domain
  subgraph model
    source_tables[Source Tables] --- DBT[DBT Models]
  end
  domain --> model
  subgraph export_type[Export Type]
    Parquet 
    DuckDB
    Excel
  end
  subgraph option
    export_type
    table_key
  end
  model --> option

```  

`manifest.csv` is located at CLI application root.  Its structure is validated(1) 
upon CLI statement execution.
{ .annotate }

  1.  `validate.py` also checks valid connection credentials and coherence between AD and DBT security for `connect`

??? note annotate "`manifest.csv` provides flexibility in usage when combined with telescopic CLI commands `E|S|L <connect> <domain> [model] [option]`"
    | connect | domain  | model | option |
    |---|---|---|---|
    | prod | madsd-8008 | saturn.saraatt | saraatt_surrogate_id |
    | prod | madsd-8008 | saturn.saradap | saradap_surrogate_id |
    | onedrive | dds-7734 | general.gwodupj | none |
    | cognos | mds-registrar | dim_student | none |
    | cognos | mds-registrar | fct_student_term_level | none |
    | onedrive | mds-registrar | dim_student | none |
    | onedrive | mds-registrar | fct_student_term_level | none |
    | onedrive | mds-registrar | exp_student_detail | none |
    | onedrive | mds-registrar | student_data_report | excel |
    | local | lampman | latin_list | excel |
    | prod | mds-finaid | faismgr.rtvyicd | none |
    | prod | mds-finaid | faismgr.rurvers | none |
    | prod | mds-finaid | faismgr.rbracmp | rbracmp_surrogate_id |
    | prod | mds-finaid | faismgr.rbrapbc | rbrapbc_surrogate_id |
    | prod | mds-common | general.glbextr | glbextr_surrogate_id |
    | prod | mds-common | general.goremal | goremal_surrogate_id |
    | prod | mds-hr | payroll.pwvretc | none |
    | prod | mds-hr | payroll.perdtot | perdtot_surrogate_id |
    | prod | mds-registrar | saturn.saraatt | saraatt_surrogate_id |
    | prod | mds-registrar | saturn.saraatt | saraatt_surrogate_id |
    | prod | mds-registrar | saturn.saraatt | saraatt_surrogate_id |
    | prod | mds-registrar | saturn.saradap | saradap_surrogate_id |

## Python CLI File Structure
``` mermaid
graph LR

main.py --> extract.py
main.py --> validate.py
validate.py ~~~ extract.py
validate.py ~~~ load.py
extract.py --> oracle.py
main.py --> secure.py
main.py --> load.py
subgraph s1 [ ]
  cognos.py --- onedrive.py
end
secure.py --> s1
load.py --> s1
```
