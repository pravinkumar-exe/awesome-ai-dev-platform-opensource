This pull request corresponds to the vulnerability described in Issue #161[https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/161] – Unauthorized Access to Avatar Images & IDOR via User Profile API.

The application suffers from two related access control issues:

**Public Avatar Access**

User avatars are stored in the /data/avatars/ directory with predictable filenames. These files are accessible without authentication, allowing anyone with the filename (even after the avatar is replaced) to view them directly.

**IDOR via /api/user/{id}**

This endpoint returns limited profile data (ID, name, email, avatar filename) for any user ID. It requires authentication but allows cross-user access, enabling enumeration of avatar filenames. When combined with public access to avatar URLs, this creates an IDOR and privacy leak.

**Proposed Code Fix**

**1. Restrict Access to Avatar Files**

Serve avatar images via an authenticated view that checks ownership or admin privileges

```
# views.py
from django.contrib.auth.decorators import login_required
from django.http import FileResponse, HttpResponseForbidden
from django.shortcuts import get_object_or_404
from myapp.models import User

@login_required
def get_avatar(request, user_id):
    target_user = get_object_or_404(User, id=user_id)
    
    if request.user != target_user and not request.user.is_staff:
        return HttpResponseForbidden("Unauthorized access.")
    
    avatar_path = target_user.profile.avatar.path
    return FileResponse(open(avatar_path, 'rb'), content_type="image/jpeg")
```

**2. Remove Avatar Field from Cross-User /api/user/{id}**

Only return the avatar field when the authenticated user is requesting their own data:

```
# serializers.py
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'first_name', 'last_name', 'username']  # No avatar by default

    def to_representation(self, instance):
        data = super().to_representation(instance)
        request = self.context.get('request')
        
        if request.user == instance or request.user.is_staff:
            data['avatar'] = instance.profile.avatar.url
        
        return data
```

These changes ensure that:

  - Avatar files cannot be accessed directly without authentication.

  - /api/user/{id} no longer leaks avatar filenames unless authorized.

  - The app enforces proper boundaries between users to protect profile images.
