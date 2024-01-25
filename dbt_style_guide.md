# ODS DBT Style Guide

## SQL Style

- Refer to [DBTLab's SQL Style Guide](https://docs.getdbt.com/best-practices/how-we-style/2-how-we-style-our-sql)
- No NULLs (convert to value using COALESCE)
- Named SELECT columns should have explicit aliases (even if not renamed) and column-aligned one-space after the longest line in the SELECT clause, to serve as a visual index

## CTEs

For more information about why we use so many CTEs, read [this glossary entry](https://docs.getdbt.com/terms/cte).

- Where performance permits, CTEs should perform a single, logical unit of work.
- CTE names should be as verbose as needed to convey what they do.
- CTEs with confusing or noteable logic should be commented with SQL comments as you would with any complex functions and should be located above the CTE.
- CTEs duplicated across models should be pulled out and created as their own models.

## Jinja

Refer to [DBTLab's Jinja Style Guide](https://docs.getdbt.com/best-practices/how-we-style/4-how-we-style-our-jinja)

## Domains and Grains

"Domains own Grains"

Inspired by UO's current organization of Data Ownership, and concepts from [Data Mesh](https://www.datamesh-architecture.com), a `Domain` is a specific category of data that is critical to UO's operations, treating data in their domain as a `Product` they own/control/provision, from data creation, processing, to consumption.  Examples might be `General`, `Registrars`, `Admissions`, `Financial Aid`, `Business Affairs`, `HR`, `Security`.

`Grain` is a term referring to the named collection of unique attrubites, per record, in a dataset.  Example `Grain`s might be: `organization`, `person`, `student`, `student_term_level`, `employee`, `employee_pay_period`.  Each `Grain` is owned by one, and only one, `Domain`.  However, `Grains` often contain supporting data from `Grains` owned by different `Domain`s.  An example might be `student_term_level`, owned by the **Registrar** `Domain`, may include demographic/contact information owned by the **General** `Domain`. 
  
## Model Layers and Organization

- Only models in `staging` should select from [sources](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources).
- Models not in the `staging` folder should select from [refs](https://docs.getdbt.com/reference/dbt-jinja-functions/ref).
- Refer to [DBTLab's Staging, Intermediate, and Mart Model Notes](https://docs.getdbt.com/best-practices/how-we-style/0-how-we-style-our-dbt-projects)


- Staging &mdash; 1:1 models that clean and standardize ingested source data tables.

- Intermediate &mdash; enrichement (descriptions, transformations, calculations) and grain-pivoting for Mart models.

- Marts &mdash; Guaranteed-stable models of Dimensions, Facts, and component grains for consumption and exposures for other domains (think Ellucian ODS Reporting Views/Cognos Query Subjects).
  
- Mesh &mdash; Secured and exposed Mart models. 1:1 Views on Mart models implementing ODS Security based on DB user (and/or Consumption Service user), and exposed to all ODS Domains.

- Exposures &mdash; Are not models, but rather specifications for JOINing Mesh models (think Cognos Packages or PowerBI Datasets).  These specifications relate how multiple Mesh models are meant to be used together for specific purposes in consumption tools/methods. 
  
- Contracts &mdash; Guaranteed-stable models aligned with Exposures (think individual Reports, Integrations).

Example Directory Organization:

!!!!!! DB Schemas.... Source_SourceSchema_tablename
## Model File Naming and Coding

- Refer to [DBTLab's Model Style Guide](https://docs.getdbt.com/best-practices/how-we-style/1-how-we-style-our-dbt-models), and [this article](https://docs.getdbt.com/blog/stakeholder-friendly-model-names).

  **NOTE** &mdash; The use of double underscores for separation of concepts in filename patterns below (think of it as an oxford comma).

  - Staging
    - Sources YML &mdash; `_<source>__<domain>__sources.yml`
    - Models YML &mdash; `_<source>__<domain>__models.yml`
    - Models SQL &mdash; `stg_<source>__<source table name>.sql`

  - Intermediate
    - Models YML &mdash; `_int__<grain>__models.yml`
    - Models SQL &mdash; `int_<grain>s__<verb, past-tense>__<grain>.sql`

  - Mart
    - Models YML &mdash; `_mart__<domain>__models.yml`
    - Models SQL &mdash; `mart_<grain>.sql`

  - Mesh
    - Models YML &mdash; `_mesh__<domain>__models.yml`
    - Models SQL &mdash; `mesh_<grain>.sql`
  
  - Contracts (grouped in Exposure directories)
    - Exposure YML &mdash; `_exp__<domain>__models.yml`
    - Contract YML &mdash; `_con__<exposure>__models.yml`
    - Contract SQL &mdash; `<type/service>__<contract_name>.sql`

- Schema, table, and column names should be in `snake_case`.

- Limit the use of abbreviations that are related to domain knowledge. An onboarding employee will understand `current_order_status` better than `current_os`.

- Use names based on the _business_ rather than the source terminology.

- Each model should have a primary key to identify the unique row and should be named `<object/grain>_id`. For example, `account_id`. This makes it easier to know what `id` is referenced in downstream joined models.

- For `staging` models, columns should be ordered in categories, where identifiers are first and date/time fields are at the end.
- Date/time columns should be named according to these conventions:

  - Timestamps: `<event>_at`  
    Format: UTC  
    Example: `created_at`

  - Dates: `<event>_date`
    Format: Date  
    Example: `created_date`

- Booleans should be prefixed with `is_` or `has_`.
  Example: `is_active_customer` and `has_admin_access`

- Price/revenue fields should be in decimal currency (for example, `19.99` for $19.99; many app databases store prices as integers in cents). If a non-decimal currency is used, indicate this with suffixes. For example, `price_in_cents`.

- Consistency is key! Use the same field names (and descriptions) across models where possible. 

## Model Configurations

- Model configurations at the [folder level](https://docs.getdbt.com/reference/model-configs#configuring-directories-of-models-in-dbt_projectyml) are applied first.
- More specific configurations can be applied at the model level [using one of these methods](https://docs.getdbt.com/reference/model-configs#apply-configurations-to-one-model-only).
- Models within the `marts` and `contracts` folder should be materialized as `table` or `incremental`.

## Testing

- At a minimum, `unique` and `not_null` tests should be applied to the expected primary key of each model.
- Failed tests applied to `staging` models may be appropriate to convert into cleaning `CTE`s within the `staging` models and the `staging` models tagged with `<source>_bug` for tracking and troubleshooting source-system data quality issues.

Need to review folder/file and table desc, other conventions, and DB Schema.

Engagement involved row-criteria, and history for stage models

dbt run-operation generate_model_yaml --args '{"model_names": ["int_banner__entities__filtered_to__organizations"], "upstream_descriptions": true, "include_data_types": false}'