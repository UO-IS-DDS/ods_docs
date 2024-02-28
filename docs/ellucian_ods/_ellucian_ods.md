# Ellucian ODS
``` mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'primaryColor': '#1d2129',
      'primaryTextColor': '#e2e4e9',
      'primaryBorderColor': '#ff6e42',
      'lineColor': '#F8B229'
    }
  }
}%%
gantt
    title ODS Replacement Roadmap
    dateFormat YYYY-MM-DD
    axisFormat %b-%Y
        POC      :a1, 2024-02-25, 45d
        Proof    :a3, after a1, 300d
        Refactor :a4, after a3, 540d
        PowerBI  :a5, after a4, 365d
```
=== "POC"

    * Sed sagittis eleifend rutrum
    * Donec vitae suscipit est
    * Nulla tempor lobortis orci

=== "Proof"

    1. Sed sagittis eleifend rutrum
    2. Donec vitae suscipit est
    3. Nulla tempor lobortis orci
   
=== "Refactor"

=== "PowerBI"

    1. Sed sagittis eleifend rutrum
    2. Donec vitae suscipit est
    3. Nulla tempor lobortis orci

=== "C"

    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"

    ``` mermaid
    %%{
        init: {
            'theme': 'forest',
        }
    }%%
    xychart-beta
        title "Sales Revenue"
        x-axis [jan, feb, mar, apr, may, jun, jul, aug,     sep, oct, nov, dec]
        y-axis "Revenue (in $)" 4000 --> 11000
        bar [5000, 6000, 7500, 8200, 9500, 10500, 11000,     10200, 9200, 8500, 7000, 6000]
        line [5000, 6000, 7500, 8200, 9500, 10500, 11000,     10200, 9200, 8500, 7000, 6000]
    ```


Daily ETL ***LATE*** Workflow
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

    [IA_Admin](https://duckweb.uoregon.edu/odsedw/twbkwbis.P_WWWLogin){:target="_blank"} is the primitive [orchestration tool](https://www.fivetran.com/blog/data-orchestration-explained-no-diy){:target="_blank"} technology chosen by Ellucian for their ODS product.

    Nightly ETL jobs are [data transformations](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview){:target="_blank"} developed by Ellucian in the geriatric [ODI - Oracle Data Integrator](https://en.wikipedia.org/wiki/Oracle_Data_Integrator){:target="_blank"}.  These jobs are clustered into Map Groups, aligned with business areas: Student, General, Finance, Financial Aid, Human Resorces, and Accounts Receivable.

    All ETL Jobs display their status in IA_Admin Control Reports, which browser-render data from to the IA_ADMIN.IA_MDBLOGH_UO database view in IOEP. Below, `REFRESH_STUDENT_UO` has yet to finish refreshing.

    | MDBLOGH_RUN_DATE | MDBLOGH_PROCESS | MDBLOGH_ERROR_IND |
    |------------------|-----------------|-------------------|
    | 2/21/2024 | REFRESH_STUDENT_UO |  |
    | 2/21/2024 | REFRESH_FINANCE_UO | N |
    | 2/20/2024 | REFRESH_FINANCE_UO | N |

=== "Step 2"

    empty

=== "Step 3"

    empty

=== "Step 4"

    empty

=== "Step 5"

    empty

