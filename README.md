# Oracle APEX: Run STORED_PROCEDURE via DBMS_SCHEDULER with Spinner & Status Polling (APEX 22.2)

This project demonstrates how to run an **Oracle STORED_PROCEDURE using a DBMS_SCHEDULER job from Oracle APEX 22.2**, disable a button while the job is running, show a spinner, periodically check the scheduler job status using AJAX, and finally re-enable the button with a success message after completion.

---

## 🚀 Features

- Run a STORED_PROCEDURE via `DBMS_SCHEDULER` from an APEX page
- Disable action button while the scheduler job is running
- Show spinner during background processing
- Poll scheduler job status using AJAX (time interval check)
- Automatically stop polling after job completion
- Enable button and show success message
- Optional page submit or region refresh

---

## 🧱 Architecture Overview

1. User clicks **Attendance Process** button
2. Oracle APEX creates a scheduler job
3. Job name is stored in a hidden page item
4. JavaScript polls job status every 5 seconds
5. When the job finishes:
   - Polling stops
   - Spinner is removed
   - Button is enabled
   - Success message is shown
   - (Optional) Page submit or region refresh

---

## 📋 Prerequisites

- Oracle APEX **22.2**
- Required privileges:
  - `DBMS_SCHEDULER`
  - `USER_SCHEDULER_RUNNING_JOBS`
- Example stored procedure:
  ```sql
  ATTENDANCE_DAILY_PROCESS

  
 ## 🧩 Page Items

Create the following Items and one hidden items:
| Item Name        | Type   |
| ---------------- | ------ |
| `P437_JOB_NAME`  | Hidden |
| `P437_FROM_DATE` | Date   |
| `P437_TO_DATE`   | Date   |
| `P437_EMP_ID`    | Text   |


## create a Button Name and give it a static id 
Button Name: ATTENDANCE_PROCESS
Static ID (Required):btn_attendance_process


## 🧠 Create a Dynamic Action on click 'ATTENDANCE_PROCESS' button 

### True Action: Exicute server-side code: 

```pl/sql code

DECLARE
    L_JOBNAME VARCHAR2(50);
BEGIN
    L_JOBNAME := 'ATT_JOB_' || TO_CHAR(SYSDATE,'YYYYMMDDHH24MISS');

    DBMS_SCHEDULER.CREATE_JOB (
        job_name            => L_JOBNAME,
        job_type            => 'STORED_PROCEDURE',
        job_action          => 'ATTENDANCE_DAILY_PROCESS',
        number_of_arguments => 3,
        enabled             => FALSE
    );

    DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE(L_JOBNAME, 1, :P437_FROM_DATE);
    DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE(L_JOBNAME, 2, :P437_TO_DATE);
    DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE(L_JOBNAME, 3, :P437_EMP_ID);

    DBMS_SCHEDULER.ENABLE(L_JOBNAME);

    :P437_JOB_NAME := L_JOBNAME;
END;

 ```

### Create another True Action for diasable button and start spinner: 

``` Exicute javascript Code:

// Disable button
apex.item('btn_attendance_process').disable();

// Show spinner and store handle
attSpinner = apex.util.showSpinner();

```

## 🌐 Create on demand AJAX Callback (Check Job Status)

- Application Process Name: CHECK_ATT_JOB_STATUS
- Type: AJAX Callback

```Exicute code

DECLARE
    l_count NUMBER;
BEGIN
    -- Check if job is still running
    SELECT COUNT(*)
      INTO l_count
      FROM USER_SCHEDULER_JOBS
     WHERE JOB_NAME = :P437_JOB_NAME;

    IF l_count > 0 THEN
        apex_json.open_object;
        apex_json.write('status', 'RUNNING');
        apex_json.close_object;
    ELSE
        apex_json.open_object;
        apex_json.write('status', 'COMPLETED');
        apex_json.close_object;
    END IF;
END;
```


## 🧠 Global JavaScript Variables

Add to Page → JavaScript → Function and Global Variable Declaration

```javascript

var attSpinner = null;
var attJobInterval = null;

```

### Create Another True Action to Check Scheduler Job, enable button and stop spinner

- JavaScript Polling Logic

``` javascript code

attJobInterval = setInterval(function () {

    apex.server.process(
        'CHECK_ATT_JOB_STATUS',
        {},
        {
            dataType: 'json',
            success: function (pData) {

                if (pData.status === 'COMPLETED') {

                    // Stop polling
                    clearInterval(attJobInterval);

                    // Enable button (correct way)
                    $('#btn_attendance_process').prop('disabled', false);

                    // Show success message
                    apex.message.showPageSuccess(
                        'Attendance process completed successfully.'
                    );

                    // Submit page AFTER delay
                    setTimeout(function () {
                        apex.submit({
                            request: 'JOB_COMPLETED',
                            showWait: false
                        });
                    }, 1000);
                }
            }
        }
    );

}, 5000);


```


 # Thank you
 ## Sanjay Sikder

 You can connect with me on [LinkedIn](https://www.linkedin.com/in/sanjay-sikder/)!
