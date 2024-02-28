## Monitoring and Notifications

``` mermaid
sequenceDiagram
  autonumber
  IA_Admin->>Jenkins: 8am M-F
  Note right of Jenkins: Any Map Groups not refreshed?
  Jenkins->>Users: Auto Email
  loop Every 5 min
      Jenkins->>Jenkins: Check for complete
  end
  Note right of Jenkins: All Map Groups refreshed!
  Jenkins->>DDS: Auto Email
  DDS->>Users: Manual Email Forward
```

=== "Step 1"

    [IA_Admin](https://duckweb.uoregon.edu/odsedw/twbkwbis.P_WWWLogin){:target="_blank"} is the [orchestration tool](https://www.fivetran.com/blog/data-orchestration-explained-no-diy){:target="_blank"} technology chosen by Ellucian for their ODS product.

    Nightly ETL jobs are [data transformations](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview){:target="_blank"} developed by Ellucian in [ODI - Oracle Data Integrator](https://en.wikipedia.org/wiki/Oracle_Data_Integrator){:target="_blank"}.  These jobs are clustered into Map Groups, aligned with business areas: Student, General, Finance, Financial Aid, Human Resorces, and Accounts Receivable.

    All ETL Jobs display their status in IA_Admin Control Reports, which browser-render data from to the IA_ADMIN.IA_MDBLOGH_UO database view in IOEP. Below, `REFRESH_STUDENT_UO` has yet to finish refreshing.

    | MDBLOGH_RUN_DATE | MDBLOGH_PROCESS | MDBLOGH_ERROR_IND |
    |------------------|-----------------|-------------------|
    | 2/21/2024 | REFRESH_STUDENT_UO |  |
    | 2/21/2024 | REFRESH_FINANCE_UO | N |
    | 2/20/2024 | REFRESH_FINANCE_UO | N |

=== "Step 2"

    ``` mermaid
    sequenceDiagram
      autonumber
      IA_Admin->>Jenkins: 8am M-F
      Note right of Jenkins: Any Map Groups not refreshed?
      Jenkins->>Users: Auto Email
      loop Every 5 min
          Jenkins->>Jenkins: Check for complete
      end
      Note right of Jenkins: All Map Groups refreshed!
      Jenkins->>DDS: Auto Email
      DDS->>Users: Manual Email Forward
    ```

=== "Step 3"

    ``` mermaid
    sequenceDiagram
      autonumber
      IA_Admin->>Jenkins: 8am M-F
      Note right of Jenkins: Any Map Groups not refreshed?
      Jenkins->>Users: Auto Email
      loop Every 5 min
          Jenkins->>Jenkins: Check for complete
      end
      Note right of Jenkins: All Map Groups refreshed!
      Jenkins->>DDS: Auto Email
      DDS->>Users: Manual Email Forward
    ```

=== "Step 4"

    

=== "Step 5"

    empty

