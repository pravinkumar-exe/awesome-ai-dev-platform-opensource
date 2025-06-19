# Placeholder Submission for Issue #116: Internal Infrastructure Metadata Exposure

This pull request is a placeholder for issue [#116](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/116) regarding the exposure of internal infrastructure metadata via the endpoint:


The endpoint is accessible to any authenticated user and leaks sensitive information such as:
- service name(e.g., "Aixblock Platform")
- Docker image names  
- Registry URLs  
- Environment types (e.g., `dev`)  
- Service versions

This information should be restricted to admin or internal backend services only.


 ##Proposed Code-Based Fix (FastAPI + RBAC Example)
Assuming the backend is written in Python (FastAPI or similar) and uses a user object with role or is_admin flags:

```
from fastapi import APIRouter, Depends, HTTPException, status
from typing import List
from app.dependencies import get_current_user  # assumes auth dependency exists
from app.models import User  # assumed user model

router = APIRouter()

@router.get("/api/settings/installation-service/", tags=["Admin Settings"])
async def get_installation_service_settings(current_user: User = Depends(get_current_user)):
    # ✅ Check if user is admin
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="You do not have permission to perform this action."
        )
    
    # Fetch internal service metadata only for admin users
    return {
        "service_name": "Aixblock Platform",
        "docker_images": ["aixblock/platform:v1.2", "aixblock/worker:v1.1"],
        "registry_url": "registry.aixblock.io",
        "environment": "dev",
        "service_version": "1.2.0"
    }
```
Explanation:

- get_current_user: assumed auth dependency that fetches the authenticated user.

- current_user.is_admin: check to ensure only admin users can access this.

- Returns 403 Forbidden if unauthorized.

**Alternative Role-Based Model (If Using Roles)**
If your system uses role strings like "admin" / "user":

```
if current_user.role != "admin":
    raise HTTPException(
        status_code=status.HTTP_403_FORBIDDEN,
        detail="You do not have permission to perform this action."
    )
```
