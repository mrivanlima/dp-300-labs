# Lab 05 — Dynamic Data Masking

## What I configured
Applied data masks to a Customers table to hide
sensitive data from low-privilege users.

## T-SQL used

-- Create table with sensitive data
CREATE TABLE Customers (
    CustomerID  INT PRIMARY KEY,
    FirstName   VARCHAR(50),
    LastName    VARCHAR(50),
    Email       VARCHAR(100),
    CreditCard  VARCHAR(20),
    SSN         VARCHAR(11)
);

-- Apply masks
ALTER TABLE Customers ALTER COLUMN Email
ADD MASKED WITH (FUNCTION = 'email()');

ALTER TABLE Customers ALTER COLUMN CreditCard
ADD MASKED WITH (FUNCTION = 'partial(0,"XXXX-XXXX-XXXX-",4)');

ALTER TABLE Customers ALTER COLUMN SSN
ADD MASKED WITH (FUNCTION = 'partial(0,"XXX-XX-",4)');

-- Create low-privilege user
CREATE USER CallCenterAgent WITHOUT LOGIN;
GRANT SELECT ON Customers TO CallCenterAgent;

-- Test masking as low-privilege user
EXECUTE AS USER = 'CallCenterAgent';
SELECT * FROM Customers;
REVERT;

## Mask types
| Mask | Function | Result |
|------|----------|--------|
| Email | email() | mXXX@XXXX.com |
| Partial | partial(0,"XXXX-",4) | XXXX-1234 |
| Default | default() | XXXX |
| Random | random(1,100) | 42 |

## Key concepts learned
- DDM hides data from low-privilege users only
- Admins and DBAs always see real data
- Data is NOT encrypted — just masked on output
- No application changes required
- EXECUTE AS USER = impersonate another user for testing

## Security feature comparison
| Feature | Encrypts? | Hides from DBA? | App changes? |
|---------|-----------|-----------------|--------------|
| TDE | Yes | No | No |
| Always Encrypted | Yes | Yes | Yes |
| Dynamic Data Masking | No | No | No |

## Exam tips
- "No app changes + hide partial data" = DDM
- "Hide from DBAs" = Always Encrypted
- "Encrypt at rest" = TDE
- DDM does NOT protect against privileged users
