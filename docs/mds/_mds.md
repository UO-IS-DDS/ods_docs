# (MDS) Modern Data Service

``` mermaid
stateDiagram-v2

  %% Declare Objects
  jenkins : Jenkins
  one_drive_folders : OneDrive Folders
  secure_one_drive_folders : OneDrive Folders
  cognos_security : Cognos Object Security
  ioex : IOEx
  state nighly_or_hourly <<fork>>
  state ad <<fork>>
  
  nightly : Nightly
  hourly : Hourly

  duckdb_main : Main DuckDB
  duckdb_secure : Secure DuckDB
  parquet_source_load : Parquet Source
  parquet_source_delta : Parquet Soure Delta
  parque_secure : Secure Parquet
  dbt_run : DBT
  metadata_publish : Metadata Publish
  
  %% Define Classes
  classDef DuckDB stroke:yellow
  classDef Application stroke:red
  classDef Consumption stroke:green


  %% Apply Classes
  class jenkins Application
  class nightly Application
  class hourly Application
  class dbt_run Application
  class metadata_publish Application
  class ioex Application
  class one_drive_folders Consumption
  class cognos_security Consumption
  class secure_one_drive_folders Consumption
  class OneDriveConsumer OneDrive
  class duckdb_main DuckDB
  class duckdb_secure DuckDB

  %% Define Flow
  jenkins --> ad
  ad --> one_drive_folders : OneDrive_AD_Security.py
  ad --> cognos_security : Cognos_AD_Security.py
  one_drive_folders --> nighly_or_hourly
  cognos_security --> nighly_or_hourly
  nighly_or_hourly --> nightly
  nighly_or_hourly --> hourly
  nightly --> parquet_source_load : Oracle2ParquetLoad.py
  parquet_source_load --> duckdb_main : Parquet2DuckDBLoad.py
  duckdb_main --> dbt_run : ODS_DBT_run.py
  hourly --> parquet_source_delta : Oracle2ParquetRefresh.py
  parquet_source_delta --> duckdb_main : Parquet2DuckDBRefresh.py
  dbt_run --> metadata_publish : ODS_metadata_publish.py
  dbt_run --> duckdb_secure : ODS_apply_duckdb_security.py
  duckdb_secure --> parque_secure : ODS_apply_parquet_security.py
  parque_secure --> ioex : Parquet2Oracle.py
  parque_secure --> secure_one_drive_folders : Parquet2OneDrive.py
  duckdb_secure --> secure_one_drive_folders : DuckDB2OneDrive.py

```