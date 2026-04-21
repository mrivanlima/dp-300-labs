# Lab 06 — Row-Level Security

## What I configured
Applied RLS to Customers table so each sales rep
only sees their own customers. Combined with DDM
from Lab 05 for layered security.

## T-SQL used

-- Security filter function
CREATE FUNCTION dbo.fn_SecurityFilter(@SalesRep VARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_result
WHERE @SalesRep = USER_NAME();

-- Security policy
CREATE SECURITY POLICY SalesRepFilter
ADD FILTER PREDICATE dbo.fn_SecurityFilter(SalesRep)
ON dbo.Customers
WITH (STATE = ON);

-- Test as different users
EXECUTE AS USER = 'RepB';
SELECT * FROM Customers;
REVERT;

## Results
| User | Rows visible | Data masked? |
|------|-------------|--------------|
| Admin | All rows | No |
| RepA | Only RepA customers | Yes (DDM) |
| RepB | Only RepB customers | Yes (DDM) |

## Key concepts learned
- RLS filters ROWS, DDM filters COLUMNS
- Filter function uses USER_NAME() to identify caller
- Security policy applies the filter to the table
- RLS + DDM can work together simultaneously
- Application needs NO changes — filtering is invisible
- Admin always sees all rows regardless of RLS

## DDM vs RLS vs Always Encrypted
| Feature | Hides | From whom |
|---------|-------|-----------|
| DDM | Columns (partial) | Low-privilege users |
| RLS | Rows | Unauthorized users |
| Always Encrypted | Columns (fully) | Everyone including DBAs |

## Exam tips
- "Each user sees only their own data" = Row-Level Security
- "Hide partial column data" = Dynamic Data Masking
- "Hide from DBAs" = Always Encrypted
- RLS uses FILTER PREDICATE in a SECURITY POLICY
