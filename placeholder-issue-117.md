# Code Fix for Issue #117: IDOR and Broken Access Control on Service Configuration

This pull request addresses [Issue #117](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/117), which describes a critical access control vulnerability in the following endpoint:
PUT /api/settings/installation-service/{id}.

## Vulnerability Summary

Any authenticated non-admin user can exploit this endpoint to:

- Modify sensitive backend service configurations (PostgreSQL, Minio, Aixblock Platform)
- Change fields like `name`, `description`, `image`, `registry`, and `environment`
- Target any service by manipulating the `{id}` parameter

This is caused by:

- **IDOR** – Users can update arbitrary objects by changing the `id`
- **Broken Access Control** – No server-side role validation

These actions result in real-time updates, confirmed via a `200 OK` response and reflected in subsequent `GET` requests.


## ✅ Code-Based Fix

To mitigate this:

- We enforce **role-based access control (RBAC)** so only admin users can update services.
- Non-admin users receive a `403 Forbidden` response.
- The `{id}` is securely validated before update.
- Only valid fields are updated to prevent data tampering.

```python
from fastapi import APIRouter, Depends, HTTPException, Path, Body, status
from app.dependencies import get_current_user
from app.models import User, InstallationServiceConfig

router = APIRouter()

@router.put("/api/settings/installation-service/{id}", tags=["Admin Settings"])
async def update_installation_service_config(
    id: int = Path(..., description="ID of the service to update"),
    update_data: dict = Body(...),
    current_user: User = Depends(get_current_user)
):
    # Restrict access to admin users only
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You do not have permission to perform this action."
        )

    # Fetch the service record securely
    service = await InstallationServiceConfig.get(id=id)
    if not service:
        raise HTTPException(status_code=404, detail="Service not found.")

    # Update only valid fields
    for key, value in update_data.items():
        if hasattr(service, key):
            setattr(service, key, value)

    await service.save()
    return service
}
```

##Example Response for Unauthorized Access
```
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You do not have permission to perform this action."
}
```
