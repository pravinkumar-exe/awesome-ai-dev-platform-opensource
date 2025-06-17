This pull request corresponds to the vulnerability described in [Issue #117](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/117).

The issue outlines an access control vulnerability in the endpoint:

    PUT /https://app.aixblock.io/api/settings/installation-service/{id}

Any authenticated non-admin user can update internal configuration data for backend services like PostgreSQL, Minio Storage, and the Aixblock Platform by changing the `{id}` parameter. These modifications are critical and should be restricted to admin users only.

The flaw demonstrates:
- Insecure Direct Object Reference (IDOR)
- Broken Access Control

This file is submitted as a placeholder, as required by the disclosure process. It originates from a separate branch for traceability. Looking forward to feedback from the maintainers.
