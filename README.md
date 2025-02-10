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
