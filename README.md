# Supply Chain Case Study
- Analyzing the data set that was provided and to get solutions for the problem statements.

## Problem Statements and their solutions

Q1. Get the number of orders by the Type of Transaction. Please exclude orders shipped from Sangli and Srinagar. Also, exclude the SUSPECTED_FRAUD cases based on the Order Status. Sort the result in the descending order based on the number of orders.

```sql
SELECT NAME, BranchName, CBALANCE
FROM AccountMaster AS A
JOIN BranchMaster AS B
ON A.BRID = B.BRID
WHERE CBALANCE IN (
    SELECT CBALANCE
    FROM (
        SELECT CBALANCE, ROW_NUMBER() OVER(ORDER BY CBALANCE DESC) AS RNK
        FROM AccountMaster
    ) AS T
    WHERE RNK = 2
    UNION
    SELECT MIN(CBALANCE) FROM AccountMaster
);
