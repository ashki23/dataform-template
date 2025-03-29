# Dataform template
A template for BQ Dataform pipeline.

## The template repository structure
The following shows the repository structure:

```yml
.
├── README.md
├── definitions
│   ├── intermediate
│   ├── outputs
│   └── sources
└── workflow_settings.yaml
```

### The workflow settings file
The `workflow_settings.yaml` file, introduced in Dataform core 3.0, stores Dataform workflow settings in the YAML format. The following code shows a sample `workflow_settings.yaml` file:

```yml
defaultProject: my-gcp-project-id
defaultDataset: dataform
defaultLocation: US
defaultAssertionDataset: dataform_assertions
```

In the preceding code sample, the key-value pairs are described as follows:

- `defaultProject`: Your BigQuery Google Cloud project ID.
- `defaultDataset`: The BigQuery dataset in which Dataform creates assets, called dataform by default.
- `defaultLocation`: Your default BigQuery dataset region. In this location, Dataform processes your code and stores executed data. This processing region has to match the location of your BigQuery datasets, but it does not need to match the Dataform repository region.
- `defaultAssertionDataset`: The BigQuery dataset in which Dataform creates views with assertion results, called dataform_assertions by default.

### The definitions directory
The following structure of subdirectories in the `definitions` directory reflects the key stages of a SQL workflow:

- `sources`: Data source declarations and basic transformation of source data, for example, filtering.
- `intermediate`: Tables and actions that read from sources and transform data before you use the transformed data to define outputs tables. Tables typically not exposed to additional processes or tools, such as business intelligence (BI) tools, after Dataform executes them to BigQuery.
- `outputs`: Definitions of tables consumed by processes or tools, such as BI, after Dataform executes them in BigQuery.
- `extra`: Files outside of the main pipeline of your SQL workflow, for example, files that contain workflow data prepared for additional use, like machine learning. An optional and custom subdirectory.

The following example shows a subdirectory structure inside the `definitions` directory with filenames that conform to the recommended naming strategy:

```yml
definitions/
    sources/
        google_analytics.sqlx
        google_analytics_filtered.sqlx
    intermediate/
        stg_analytics_concept.sqlx
    outputs/
        customers.sqlx
        sales/
            sales.sqlx
            sales_revenue.sqlx
        ads/
            campaigns.sqlx
            ads_revenue.sqlx
```

## SQLX files
To use Dataform core, we can write SQLX files. Each SQLX file contains a query that defines a database relation that Dataform creates and updates inside BigQuery.

### SQLX file config block
A SQLX file consists of a config block and a body. All config properties, and the config block itself, are optional. Given this, any plain SQL file is a valid SQLX file that Dataform executes as-is. We can use the cofing block for:

- [Data source declarations](https://cloud.google.com/dataform/docs/declare-source), for example:

  ```sql
  config {
      type: "declaration",
      database: "bigquery-public-data",
      schema: "samples",
      name: "shakespeare",
  }
  ```

  where:

  - `database`: the project ID of the project which contains the data source.
  - `schema`: the BigQuery dataset in which the data source exists.
  - `name`: the name of the table or view that you want to use as the data source. You can later use that name to reference the data source in Dataform.

- [Create tables](https://cloud.google.com/dataform/docs/create-tables), for example:

  ```sql
  config {
      type: "table"
  }
  ```
 
  where:

  - `type`: specifies the table type, which can be one of the following: `table`, `view`, or `incremental`.

### SQLX file body
The SQLX body contains the SQL query used to create a table or perform other SQL operations in BigQuery. In the body we can use the `ref` function that lets us reference tables defined in the Dataform project instead of hard coding the schema and table names of the data table. For example:

```sql
config { type: "table" }

SELECT
  order_date AS date,
  order_id AS order_id,
  order_status AS order_status,
  SUM(item_count) AS item_count,
  SUM(amount) AS revenue

FROM ${ref("store_clean")}

GROUP BY 1, 2, 3
```

## References
- [Dataform template repository](https://github.com/ashki23/dataform-template)
- [Create and execute a SQL workflow in Dataform](https://cloud.google.com/dataform/docs/quickstart-create-workflow)
- [Introduction to SQL workflows](https://cloud.google.com/dataform/docs/sql-workflows)
- [Configure Dataform workflow settings](https://cloud.google.com/dataform/docs/manage-repository#configure-workflow-settings)
- [Recommended structure of the definitions directory](https://cloud.google.com/dataform/docs/structure-repositories)
- [SQLX file config block](https://cloud.google.com/dataform/docs/overview#sqlx_file_config_block)
- [SQLX file body](https://cloud.google.com/dataform/docs/overview#sqlx_file_body)
