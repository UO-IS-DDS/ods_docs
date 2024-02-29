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

??? info "dds_late_ods_etl__start.groovy"
    ``` groovy linenums="1" hl_lines="38-53"
    pipeline {
        agent {
            node {
                label 'devsrc01'
            }
        }
        options {
            disableConcurrentBuilds()
        }
        triggers {
            cron('H 8 * * 1-5') // Triggers M-F 8am
        }
        
        environment {
            VERSION = '0.1.0'
            DATABASE_CREDENTIALS_ID = 'idrworx-prod'
            DB_HOST = 'ioepcl.uoregon.edu'
            DB_PORT = '1560'
            DB_SERVICE_NAME = 'ioe_prod.uoregon.edu'
        }
    
        stages {
            stage('Check for late ETL') {
                steps {
                    timeout(time: 10, unit: 'MINUTES') {
                        withCredentials([usernamePassword(credentialsId: "${DATABASE_CREDENTIALS_ID}", 
                                                          passwordVariable: 'PASSWORD', 
                                                          usernameVariable: 'USERNAME')]) {
                            script {
                                try {
                                    def etl_domains = sh (
                                        script: '''
                                            $ORACLE_HOME/bin/sqlplus -S ${USERNAME}/${PASSWORD}@//${DB_HOST}:${DB_PORT}/${DB_SERVICE_NAME} <<EOF
                                            set heading off
                                            set feedback off
                                            set pagesize 0
                                            set linesize 1000
                                            SELECT DISTINCT DECODE(mdblogh_process, 
                                                                   'REFRESH_STUDENT_UO'    , 'Student <br>',
                                                                   'REFRESH_FINANCE_UO'    , 'Finance <br>',
                                                                   'REFRESH_VAL_HIE_GEN_UO', 'General <br>',
                                                                   'REFRESH_AR_UO'         , 'Accounts Receivable <br>',
                                                                   'REFRESH_HR_UO'         , 'Human Resources <br>',
                                                                   'REFRESH_FINAID_UO'     , 'Financial Aid <br>') domain
                                            FROM IA_ADMIN.IA_MDBLOGH_UO
                                            WHERE TRUNC(mdblogh_run_date) = TRUNC(SYSDATE)
                                              AND mdblogh_error_ind = 'N'
                                              AND mdblogh_process IN ('REFRESH_STUDENT_UO',
                                                                      'REFRESH_FINANCE_UO',
                                                                      'REFRESH_VAL_HIE_GEN_UO',
                                                                      'REFRESH_AR_UO',
                                                                      'REFRESH_HR_UO',
                                                                      'REFRESH_FINAID_UO');
                                            exit;
    EOF
                                        ''',
                                        returnStdout: true
                                    ).trim()
    
                                    if (etl_domains) {
                                        env.ETL_PROCESS_LATE = true
                                    } else {
                                        env.ETL_PROCESS_LATE = false
                                    }
                                    
                                    env.ETL_DOMAINS = etl_domains // used in user email notification
    
                                } catch (Exception e) {
                                      echo "An error occurred: ${e.getMessage()}"
                                      currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
            }
            stage('Notify and enable monitoring') {
                when {
                    expression {
                        env.ETL_PROCESS_LATE == "true"
                    }
                }
                parallel {
                    stage('Send user email'){
                        steps {
                            timeout(time: 10, unit: 'MINUTES') {
                                script {
                                    try {
                                        echo "Sending email"
                                        emailext to: 'is-status@uoregon.edu',
                                                 subject: 'LATE - IDR Daily Refresh',
                                                 body: "<p>Good morning IDR Users!<br><br>" +
                                                       "Unfortunately the data refresh for the following area(s) are running late this morning:<br><br>" +
                                                       "${etl_domains}" + "<br>" +
                                                       "This could cause some reports to show incomplete results. We apologize for this inconvenienceand are working to fix the issue. We will send an update when the refresh is finished for allareas.<br><br>" +
                                                       "If you have any questions or concerns, please reach out to us via the <a href='https://        serviceuoregon.edu/TDClient/Requests/ServiceDet?ID=18948'>UO Service Portal</a>.<br><br>" +
                                                       "Thank you for your patience,<br><br>" +
                                                       "Your Friendly Neighborhood IDR Team<p>",
                                                 from: 'noreply-is.idr@uoregon.edu',
                                                 mimeType: 'text/html'
                                    } catch (Exception e) {
                                            echo "Error sending email: ${e.getMessage()}"
                                            currentBuild.result = 'FAILURE'
                                    }
                                }
                            }
                        }
                    }
                    stage('Enable monitoring job'){
                        steps {
                            timeout(time: 10, unit: 'MINUTES') {
                                script {
                                    try {
                                        echo "enable job"
                                        def check_job = Jenkins.instance.getItemByFullName("IDR/IDR Late Daily Refresh Monitor")
                                        check_job.setDisabled(false)
                                    } catch (Exception e) {
                                            echo "Error enabling monitoring job: ${e.getMessage()}"
                                            currentBuild.result = 'FAILURE'
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
        post {
            failure {
                echo "Failure - sending email to DDS"
                emailext to:"lampman@uoregon.edu",
                         subject: "FAILED - Daily ETL Check and User Notification",
                         body: "See attached log",
                         attachLog: true
            }
        }
    }
    ```


??? info "dds_late_ods_etl__finish.groovy"
    ``` groovy linenums="1"
    pipeline {
        agent {
            node {
                label 'devsrc01'
            }
        }
        options {
            disableConcurrentBuilds()
        }
        triggers {
            //cron('*/5 * * * *') // Triggers every 5 min, when enabled
            cron('*/1 * * * *')
        }
    
        environment {
            VERSION = '0.1.0'
            DATABASE_CREDENTIALS_ID = 'idrworx-prod'
            DB_HOST = 'ioepcl.uoregon.edu'
            DB_PORT = '1560'
            DB_SERVICE_NAME = 'ioe_prod.uoregon.edu'
        }
    
        stages {
            stage('Check for complete ETL') {
                steps {
                    timeout(time: 10, unit: 'MINUTES') {
                        withCredentials([usernamePassword(credentialsId: "${DATABASE_CREDENTIALS_ID}", passwordVariable: 'PASSWORD', usernameVariable:'USERNAME')]) {
                            script {
                                try {
                                    def etl_domains = sh (
                                        script: '''
                                            $ORACLE_HOME/bin/sqlplus -S ${USERNAME}/${PASSWORD}@//${DB_HOST}:${DB_PORT}/${DB_SERVICE_NAME} <<EOF
                                            set heading off
                                            set feedback off
                                            set pagesize 0
                                            set linesize 1000
                                            SELECT 'X'
                                            FROM DUAL 
                                            WHERE NOT EXISTS (SELECT 'X'
                                                              FROM IA_ADMIN.IA_MDBLOGH_UO
                                                              WHERE TRUNC(mdblogh_run_date) = TRUNC(SYSDATE)
                                                                AND mdblogh_error_ind <> 'N'
                                                                AND mdblogh_process IN ('REFRESH_STUDENT_UO',
                                                                                        'REFRESH_FINANCE_UO',
                                                                                        'REFRESH_VAL_HIE_GEN_UO',
                                                                                        'REFRESH_AR_UO',
                                                                                        'REFRESH_HR_UO',
                                                                                        'REFRESH_FINAID_UO'));
                                            exit;
    EOF
                                        ''',
                                        returnStdout: true
                                    ).trim()
                                    
                                    if (etl_domains) {
                                        env.ETL_PROCESS_COMPLETE = true
                                    } else {
                                        env.ETL_PROCESS_COMPLETE = false
                                    }
                                } catch (Exception e) {
                                      echo "An error occurred: ${e.getMessage()}"
                                      currentBuild.result = 'FAILURE'
                                }
                            }
                        }
                    }
                }
            }
            stage('Notify and disable monitoring') {
                when {
                    expression {
                        env.ETL_PROCESS_COMPLETE == "true"
                    }
                }
                parallel {
                    stage('Send user email'){
                        steps {
                            timeout(time: 10, unit: 'MINUTES') {
                                script {
                                    try {
                                        echo "Sending email"
                                        emailext to: 'is.idr@uoregon.edu',
                                                 subject: 'COMPLETE - IDR Daily Refresh',
                                                 body: "<p>IDR Users!<br><br>" +
                                                       "The data refreshes are now complete for all areas. We recommend re-running any reports that requiredata as of yesterday.<br><br>" +
                                                       "If you have any questions or concerns, please reach out to us via the <a href='https://service.uoregonedu/TDClient/Requests/ServiceDet?ID=18948'>UO Service Portal</a>.<br><br>" +
                                                       "Thank you for your patience,<br><br>" +
                                                       "Your Friendly Neighborhood IDR Team</p>",
                                                 from: 'noreply-is.idr@uoregon.edu',
                                                 mimeType: 'text/html'
                                    } catch (Exception e) {
                                            echo "Error sending email: ${e.getMessage()}"
                                            currentBuild.result = 'FAILURE'
                                    }
                                }
                            }
                        }
                    }
                    stage('Disable monitoring job'){
                        steps {
                            timeout(time: 10, unit: 'MINUTES') {
                                script {
                                    try {
                                        echo "disable job"
                                        def check_job = Jenkins.instance.getItemByFullName("IDR/IDR Late Daily Refresh Monitor")
                                        check_job.setDisabled(true)
                                    } catch (Exception e) {
                                            echo "Error disabling monitoring job: ${e.getMessage()}"
                                            currentBuild.result = 'FAILURE'
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
        post {
            failure {
                echo "Failure - sending email to DDS"
                emailext to:"lampman@uoregon.edu",
                         subject: "FAILED - Daily ETL Monitor and DDS Notification",
                         body: "See attached log",
                         attachLog: true
            }
        }
    }
    ```

=== "Step 1"

    [IA_Admin](https://duckweb.uoregon.edu/odsedw/twbkwbis.P_WWWLogin){:target="_blank"} is the [orchestration tool](https://www.fivetran.com/blog/data-orchestration-explained-no-diy){:target="_blank"} technology chosen by Ellucian for their ODS product.

    Nightly ETL jobs are [data transformations](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview){:target="_blank"} developed by Ellucian in [ODI - Oracle Data Integrator](https://en.wikipedia.org/wiki/Oracle_Data_Integrator){:target="_blank"}.  These jobs are clustered into `Map Groups`, aligned with business areas: Student, General, Finance, Financial Aid, Human Resorces, and Accounts Receivable.

    All ETL Jobs display their status in IA_Admin Control Reports, which browser-render data from to the IA_ADMIN.IA_MDBLOGH_UO database view in IOEP. Below, `REFRESH_STUDENT_UO` has yet to finish refreshing.

    | MDBLOGH_RUN_DATE | MDBLOGH_PROCESS | MDBLOGH_ERROR_IND |
    |------------------|-----------------|-------------------|
    | 2/21/2024 | REFRESH_STUDENT_UO |  |
    | 2/21/2024 | REFRESH_FINANCE_UO | N |
    | 2/20/2024 | REFRESH_FINANCE_UO | N |

    `Map Groups` are scheduled to run at 1am and complete by 7am.  When these fail to complete by 8am, users are notified.

    To faciliate automation of this montoring and notification, a Jenkins Job [IDR Late Daily Refresh Email](https://esm.uoregon.edu:8080/view/IDR/job/IDR/job/IDR%20Late%20Daily%20Refresh%20Email/){:target="_blank"} is scheduled to run Monday - Friday at 8am. This job checks for any `Map Groups` that have yet to complete.


=== "Step 2"

    If all `Map Groups` have completed by 8am, the job concludes silently.  
    
    However, if **ANY** `Map Groups` have yet to complete, a notifcation email is generated to notify IDR Users (UO Staff with Cognos accounts) with the specific `Map Groups` still running.

    ??? abstract "Example Email to IDR Users"
        **To: idr-status@uoregon.edu**<br>
        **From: noreply-is.idr@uoregon.edu**<br>
        **Subject: LATE - IDR Daily Refresh**

        Good morning IDR Users!
        
        Unfortunately the data refresh for the following area(s) are running late this morning:
        
        Financial Aid<br>
        Student<br>
        Human Resources<br>
        
        
        
        This could cause some reports to show incomplete results. We apologize for this inconvenience and are working to fix the issue. We will send an update when the refresh is finished for all areas.
        
        If you have any questions or concerns, please reach out to us via the [UO Service Portal](https://service.uoregon.edu/TDClient/Requests/ServiceDet?ID=18948){:target="_blank"}.
        
        Thank you for your patience,
        
        Your Friendly Neighborhood IDR Team
    

=== "Step 3"

    In addition to sending a 'LATE - IDR Daily Refresh' email to users, another Jenkins Job [IDR Late Daily Refresh Monitor](https://esm.uoregon.edu:8080/view/IDR/job/IDR/job/IDR%20Late%20Daily%20Refresh%20Monitor/){:target="_blank"} is executed.  

    This job checks for completion of **ALL** `Map Groups`, is scheduled to run every 5 minutes, and is default-disabled except when enabled by the previous Jenkins Job [IDR Late Daily Refresh Email](https://esm.uoregon.edu:8080/view/IDR/job/IDR/job/IDR%20Late%20Daily%20Refresh%20Email/){:target="_blank"}.

    When this job confirms **ALL** `Map Groups` have completed, it **self-disables** (stops running every 5 minutes) and sends a notification email to IDR Staff.


=== "Step 4"

    In addition to **self-disabling**, the Jenkins Job [IDR Late Daily Refresh Monitor](https://esm.uoregon.edu:8080/view/IDR/job/IDR/job/IDR%20Late%20Daily%20Refresh%20Monitor/){:target="_blank"}, it sends a notifcation email to IDR Staff, alerting them to the completion of the late Daily Refreshing.

    ??? abstract "Example Email to IDR Users"
        **To: is.idr@uoregon.edu**<br>
        **From: noreply-is.idr@uoregon.edu**<br>
        **Subject: COMPLETE - IDR Daily Refresh**

        IDR Users!

        The data refreshes are now complete for all areas. We recommend re-running any reports that require data as of yesterday.
        
        If you have any questions or concerns, please reach out to us via the [UO Service Portal](https://service.uoregon.edu/TDClient/Requests/ServiceDet?ID=18948){:target="_blank"}.
        
        Thank you for your patience,
        
        Your Friendly Neighborhood IDR Team


=== "Step 5"

    Once IDR Staff confirm completion of Daily Refreshes, they manually forward the email notification of-completion to IDR Users.

This is process-automation was instituted on 01-MAR-2024.  Previously, this involved close-coordination with IDR Staff and DBAs during off-hours.

In addition to this notification are manual steps taken by IDR Staff to update the status-report on the landing-page of [Cognos](https://cognos.uoregon.edu/ibmcognos/bi/?perspective=welcome){:target="_blank"} to reflect Daily Refreshes are complete. 

