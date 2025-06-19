# Code Fix for Issue #137: Unauthorized Modification of S3 Storage Configuration

This pull request corresponds to the vulnerability described in [Issue #137](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/137).

The issue identifies a serious access control vulnerability in the endpoint:
PUT /api/storages/s3-server/{id}


Any authenticated user can modify sensitive S3 storage configurations belonging to other users simply by altering the `{id}` parameter. Modifiable fields include:

- title,description,prefix,region_name,presign_ttl,,aws_session_token,bucket.

This results in:
- Insecure Direct Object Reference (IDOR)
- Broken Access Control
- Potential for service disruption or resource hijacking

** Proposed Code-Based Fix **

To fix this vulnerability securely, we enforce the following rule:

> Only the **owner** of the storage configuration or an **admin** user is allowed to perform updates.

All other users will receive a `403 Forbidden` response.

```python
from fastapi import APIRouter, Depends, HTTPException, Path, Body, status
from app.dependencies import get_current_user
from app.models import User, S3StorageConfig  # assumed models

router = APIRouter()

@router.put("/api/storages/s3-server/{id}", tags=["Storage Settings"])
async def update_s3_storage_config(
    id: int = Path(..., description="ID of the S3 storage config to update"),
    update_data: dict = Body(...),
    current_user: User = Depends(get_current_user)
):
    # Fetch the storage config
    config = await S3StorageConfig.get(id=id)
    if not config:
        raise HTTPException(status_code=404, detail="Storage configuration not found.")

    # Allow only owner or admin to modify
    if config.owner_id != current_user.id and not current_user.is_admin:
        raise HTTPException(
            status_code=403,
            detail="You do not have permission to modify this storage configuration."
        )

    #  Update only valid fields
    for key, value in update_data.items():
        if hasattr(config, key):
            setattr(config, key, value)

    await config.save()
    return config

```
### Example Response for Unauthorized Access:

```http
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You do not have permission to modify this storage configuration."
}

```
