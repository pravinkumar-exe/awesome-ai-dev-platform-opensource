# Placeholder Submission for Issue #116: Internal Infrastructure Metadata Exposure

This pull request is a placeholder for issue [#116](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/116) regarding the exposure of internal infrastructure metadata via the endpoint:


The endpoint is accessible to any authenticated user and leaks sensitive information such as:
- service name(e.g., "Aixblock Platform")
- Docker image names  
- Registry URLs  
- Environment types (e.g., `dev`)  
- Service versions

This information should be restricted to admin or internal backend services only.

---

## 🛠️ Proposed Fix

To address this issue securely:

- Apply strict **role-based access control (RBAC)** so only privileged (admin) users can access this endpoint.
- If a non-privileged user attempts to access the endpoint "https://app.aixblock.io/api/settings/installation-service/", return a `403 Forbidden` response.

### Example Response for Unauthorized Access:

```http
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You do not have permission to perform this action."
}
```
