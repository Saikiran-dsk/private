(SELECT 
    tc.TEST_RUN_ID, 
    COUNT(*) AS count_status,
    MAX(tc.WORK_FLOW_ID) AS max_work_flow_id
FROM 
    TEST_CONTRL tc
JOIN 
    TEST_RUN tr ON tr.TEST_RUN_ID = tc.TEST_RUN_ID
WHERE 
    tr.STATUS = 'completed'
GROUP BY 
    tc.TEST_RUN_ID
ORDER BY 
    count_status DESC
LIMIT 1)

UNION ALL

(SELECT 
    tc.TEST_RUN_ID, 
    COUNT(*) AS count_status,
    MAX(tc.WORK_FLOW_ID) AS max_work_flow_id
FROM 
    TEST_CONTRL tc
JOIN 
    TEST_RUN tr ON tr.TEST_RUN_ID = tc.TEST_RUN_ID
WHERE 
    tr.STATUS = 'in progress'
GROUP BY 
    tc.TEST_RUN_ID
ORDER BY 
    count_status DESC
LIMIT 1)
LIMIT 1;
