 Placeholder Submission for Issue #137: Unauthorized Modification of S3 Storage Configuration

This pull request corresponds to the vulnerability described in [Issue #137](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/137).

The issue identifies a serious access control vulnerability in the endpoint:
PUT /api/storages/s3-server/{id}


Any authenticated user can modify sensitive S3 storage configurations belonging to other users simply by altering the `{id}` parameter. Modifiable fields include:

- title,description,prefix,region_name,presign_ttl,,aws_session_token,bucket.

This results in:
- Insecure Direct Object Reference (IDOR)
- Broken Access Control
- Potential for service disruption or resource hijacking

** Proposed Fix **
To address this securely:

- Apply strict access control checks to ensure only the owner of the S3 storage or an authorized admin can update the configuration.
- Reject unauthorized modification attempts with a '403 Forbidden' response.

### Example Response for Unauthorized Access:

```http
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You do not have permission to modify this storage configuration."
}


