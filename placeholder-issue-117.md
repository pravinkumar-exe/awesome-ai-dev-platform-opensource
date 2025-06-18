This pull request corresponds to the vulnerability described in Issue #117.

The issue outlines a critical access control vulnerability in the following endpoint: 

[PUT https://app.aixblock.io/api/settings/installation-service/{id}].
Any authenticated non-admin user can update internal configuration data for backend services such as:
PostgreSQL,Minio Storage,Aixblock Platform by manipulating the {id} parameter. These configurations are sensitive and directly impact the platform’s backend behavior. Modifying them should only be permitted for admin-level users.

This flaw is a combination of:
- Insecure Direct Object Reference (IDOR) — Users can modify any resource by changing the ID in the URL.
- Broken Access Control — No role-based restriction is enforced on this critical endpoint.

**Proposed Fix**

To mitigate this issue securely:
- Enforce role-based access control (RBAC) to ensure only admin users can perform PUT operations on this endpoint.
- Validate the user's role server-side before processing the request.
- If a non-admin user attempts access, return a 403 Forbidden response.

Example Response for Unauthorized Access:
```
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You do not have permission to perform this action."
}
```
