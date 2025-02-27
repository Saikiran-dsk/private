https://coderbyte.com/sl-candidate?inviteKey=M34m7yLi5f

https://meet.google.com/vjo-xinp-aws?hs=224
















{
  value: any,              // Actual cell value
  data: any,               // Full row data
  node: RowNode,           // Row details
  colDef: ColDef,          // Column definition
  column: Column,          // Column object
  rowIndex: number,        // Row index
  api: GridApi,            // Grid API instance
  columnApi: ColumnApi,    // Column API instance
  context: any,            // Context object (if provided)
  refreshCell: Function    // Function to refresh the cell
}




**FSS - SysOps Work Scenarios**

This document outlines the different process flows in the FSS SysOps system, including the **Approved Process, Rejected Process, Cancel Process, and File Validation Failed Process**. Each process follows a structured workflow to ensure smooth execution.

---

## **1. Approved Process**

### **Step 1: Open Modal**
- The user clicks on the highlighted link on the screen.
- A modal window opens for further processing.

### **Step 2: Select Cycle Date and Upload File**
- The user selects the cycle date from the calendar input.
- The user uploads a file through the **Upload** button.

### **Step 3: File Validation & Processing**
- The uploaded file is validated in the backend.
- If valid, it proceeds to the next steps.

### **Step 4: View Uploaded Links**
- The user sees a grid with three clickable links:
  1. **Uploaded File** – Shows the original file.
  2. **Updated File** – Displays the modified file.
  3. **Secondary Approval** – Redirects to the secondary approval screen.

### **Step 5: Verification of Files**
- Clicking **Uploaded File** opens a modal with the original file contents.
- Clicking **Updated File** opens a modal with the updated file contents.
- Clicking **Secondary Approval** redirects to the approval screen.

### **Step 6: Secondary Approval Process**
- If the approver approves the file, it is reflected in the **NSS Test Table**.

### **Step 7: Submit to Fed**
- The file is submitted to Fed, awaiting response.

### **Step 8: View Fed Response**
- Upon receiving a response, a new link appears.
- Clicking the link opens a modal with the **Fed request and response details**.

### **Step 9: Final Report Generation**
- Once the response is received, a **Test Report** link appears.
- Clicking the link displays complete details of the workflow.

---

## **2. Rejected Process**

### **Step 1 to Step 5: Same as Approved Process**
- The process follows the same steps up to the **Secondary Approval** stage.

### **Step 6: Secondary Approval Rejection**
- If the secondary approval is **rejected**, it is recorded in the **NSS Test Table**.
- The workflow is updated with **rejection comments**.

### **Step 7: Disable Further Steps**
- Since the request is rejected, the next steps are disabled.
- The process moves directly to the **Test Report** step.

### **Step 8: View Rejection Details**
- Clicking the **Test Report** link displays all details, including rejection comments and workflow status.

---

## **3. Cancel Process**

### **Step 1 to Step 5: Same as Approved Process**
- The process follows the same steps up to the **Secondary Approval** stage.

### **Step 6: Workflow Cancellation**
- If the user **cancels** the workflow, it cancels:
  - The entire process.
  - The secondary approval step.

### **Step 7: Disable Further Steps**
- Once canceled, all further steps are disabled.
- The process moves to the **Test Report** step.

### **Step 8: View Cancellation Details**
- Clicking the **Test Report** link displays all details, including cancellation status.

---

## **4. File Validation Failed Process**

### **Step 1 to Step 3: Same as Approved Process**
- The user selects a cycle date and uploads a file.
- The backend validates the file.

### **Step 4: Validation Failure Handling**
- If the file is **invalid**, the system:
  - Enables the **Upload File** step again.
  - Disables all next steps.
  - Directly enables the **Test Report** step.

### **Step 5: View Original File**
- Clicking **Uploaded File** allows the user to verify the original file.

### **Step 6: View Validation Failure Details**
- Clicking the **Test Report** link displays details about the validation failure and workflow status.

---

## **Conclusion**
Each of the four processes follows a structured workflow to ensure proper handling of approvals, rejections, cancellations, and file validation failures. Users can track each step through the provided UI links and modals, ensuring transparency and efficiency in the system.





import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

import java.util.List;

@Configuration
public class CorsConfig {
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(List.of("*")); // Replace with frontend URL
        configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(List.of("Content-Type", "Authorization"));
        configuration.setExposedHeaders(List.of("Custom-Response-Header"));

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}


 @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ProblemDetail handleMaxSizeException(MaxUploadSizeExceededException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.PAYLOAD_TOO_LARGE, "File size exceeds the allowed limit!");
    }












import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class WorkflowServiceTest {

    @Mock
    private WorkFlowRunDao workFlowRunDao; // Mock DAO dependency

    @InjectMocks
    private WorkflowService workflowService; // Class under test

    private static final int WORKFLOW_RUN_ID = 1;
    private static final int TEST_RUN_ID = 100;

    @BeforeEach
    void setUp() {
        workflowService = new WorkflowService(workFlowRunDao);
    }

    /** 
     * Test successful execution of create method.
     */
    @Test
    void testCreate_Success() {
        doNothing().when(workFlowRunDao).create(any(WorkflowRun.class));

        assertDoesNotThrow(() -> workflowService.create(WORKFLOW_RUN_ID, TEST_RUN_ID));

        verify(workFlowRunDao, times(1)).create(any(WorkflowRun.class));
    }

    /** 
     * Test scenario where DAO throws an exception.
     */
    @Test
    void testCreate_DAOException() {
        doThrow(new DAOAccessException("Database Error")).when(workFlowRunDao).create(any(WorkflowRun.class));

        Exception exception = assertThrows(DAOAccessException.class, 
            () -> workflowService.create(WORKFLOW_RUN_ID, TEST_RUN_ID));

        assertEquals("Failed to persist workflowRun", exception.getMessage());
        verify(workFlowRunDao, times(1)).create(any(WorkflowRun.class));
    }

    /** 
     * Test unexpected exception handling.
     */
    @Test
    void testCreate_RuntimeException() {
        doThrow(new RuntimeException("Unexpected Error")).when(workFlowRunDao).create(any(WorkflowRun.class));

        Exception exception = assertThrows(RuntimeException.class, 
            () -> workflowService.create(WORKFLOW_RUN_ID, TEST_RUN_ID));

        assertEquals("createAcknowledgement Step creating failed", exception.getMessage());
        verify(workFlowRunDao, times(1)).create(any(WorkflowRun.class));
    }
}














import lombok.extern.slf4j.Slf4j;
import java.util.Optional;

@Slf4j
public void create(int workFlowRunId, int testRunId) throws DAOAccessException, RuntimeException, ResourceNotFoundException {
    long startTime = System.currentTimeMillis(); // Capture start time for performance measurement
    log.info("createAcknowledgement Step Started creating");

    try {
        // Initialize a new WorkflowRun instance
        WorkflowRun workflowRun = new WorkflowRun();
        
        // Set values for the WorkflowRun object
        workflowRun.setTestRunId((long) testRunId);
        workflowRun.setTaskId((long) Constants.RECEIVED_ACKNOWLEDGE_FROM_FED);
        workflowRun.setTaskStatusDetails(Constants.FED_RESPONSE_WAITING_MESSAGE);
        workflowRun.setStatus(Constants.IN_PROGRESS);
        workflowRun.setWorkflowRunId(workFlowRunId);
        
        // Use Optional to avoid NullPointerException before persisting the object
        Optional.ofNullable(workflowRun).ifPresent(workfLowRunDao::create);
        
        long endTime = System.currentTimeMillis(); // Capture end time
        log.info("createAcknowledgement Step creation - Time taken in ms: {}", endTime - startTime);
    } catch (Exception e) {
        // Log error with detailed message and stack trace
        log.error("createAcknowledgement Step creation failed: {}", e.getMessage(), e);
        throw new RuntimeException("createAcknowledgement Step creating failed", e);
    }
}





SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    IS_NULLABLE, 
    CHARACTER_MAXIMUM_LENGTH, 
    NUMERIC_PRECISION, 
    NUMERIC_SCALE
FROM 
    INFORMATION_SCHEMA.COLUMNS
WHERE 
    TABLE_NAME = 'MY_TABLE';

SELECT 
    tc.CONSTRAINT_TYPE, 
    kcu.COLUMN_NAME, 
    tc.CONSTRAINT_NAME
FROM 
    INFORMATION_SCHEMA.TABLE_CONSTRAINTS tc
JOIN 
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE kcu
    ON tc.CONSTRAINT_NAME = kcu.CONSTRAINT_NAME
WHERE 
    tc.TABLE_NAME = 'MY_TABLE';



ALTER TABLE MY_TABLE DROP CONSTRAINT PK_MY_TABLE;

ALTER TABLE MY_TABLE DROP CONSTRAINT PK_MY_TABLE;
ALTER TABLE MY_TABLE ALTER COLUMN ID SET NULL;


SELECT 
    CONSTRAINT_NAME 
FROM 
    INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE 
    TABLE_NAME = 'MY_TABLE' 
    AND CONSTRAINT_TYPE = 'PRIMARY KEY';











-- Step 1: Get the maximum TEST_RUN_ID based on STATUS and the associated WORK_FLOW_ID
SELECT 
    tr.TEST_RUN_ID, 
    tr.STATUS,
    MAX(tc.WORK_FLOW_ID) AS max_work_flow_id,
    MAX(tr.TEST_RUN_ID) AS max_test_run_id
FROM 
    TEST_CONTRL tc
JOIN 
    TEST_RUN tr ON tr.TEST_RUN_ID = tc.TEST_RUN_ID
WHERE 
    tr.STATUS IN ('completed', 'in progress')  -- You can add more statuses as needed
GROUP BY 
    tr.TEST_RUN_ID, tr.STATUS
ORDER BY 
    tr.TEST_RUN_ID DESC, tr.STATUS DESC
LIMIT 1;

-- Step 2: Get the last record (if needed)
SELECT 
    tr.TEST_RUN_ID,
    tr.TEST_NM,
    tr.STATUS,
    tr.USR_ID,
    tc.WORK_FLOW_ID
FROM 
    TEST_RUN tr
JOIN 
    TEST_CONTRL tc ON tr.TEST_RUN_ID = tc.TEST_RUN_ID
ORDER BY 
    tr.TEST_RUN_ID DESC  -- Or use a datetime column if you have one for ordering
LIMIT 1;



SELECT 
    tr.TEST_RUN_ID,
    tr.STATUS,
    tr.TEST_NM,
    tr.USR_ID,
    tc.WORK_FLOW_ID
FROM 
    TEST_RUN tr
JOIN 
    TEST_CONTRL tc ON tr.TEST_RUN_ID = tc.TEST_RUN_ID
WHERE 
    tr.TEST_RUN_ID = (SELECT MAX(TEST_RUN_ID) FROM TEST_RUN);
