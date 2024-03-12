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
  subgraph source
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

## Python CLI File Structure
``` mermaid
graph LR

main.py --> extract.py
extract.py --> oracle.py
main.py --> secure.py
main.py --> load.py
subgraph s1 [ ]
  cognos.py --- onedrive.py
end
secure.py --> s1
load.py --> s1
```
