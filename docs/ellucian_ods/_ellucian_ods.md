# Ellucian ODS

``` mermaid
sequenceDiagram
  autonumber
  IA_ADMIN->>Jenkins: 8am M-F
  Note right of Jenkins: Any Map Groups not refreshed?
  Jenkins->>Users: Auto Email
  loop Every 5 min
      Jenkins->>Jenkins: Check for complete
  end
  Note right of Jenkins: All Map Groups refreshed!
  Jenkins->>DDS: Auto Email
  DDS->>Users: Manual Email Forward
```