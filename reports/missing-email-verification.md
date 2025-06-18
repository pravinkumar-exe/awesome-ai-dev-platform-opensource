This pull request serves as a placeholder for the vulnerability reported in Issue #56.

The issue highlights a critical authentication flaw:
Users can register using any syntactically valid email address without verifying ownership. Immediately after signup, they receive session tokens and gain access to protected endpoints — even if they do not control the provided email.

**Proposed Fix**
To address this securely:
- Enforce email ownership verification before granting access to authenticated endpoints or issuing session tokens with full privileges.
- The app already has a secure token mechanism for password resets(e.g.,https://app.aixblock.io/user/resetpassword/?uidb64=...&token=...).
- A similar flow should be used to verify new user email addresses.

After signup:

- Inform users: “Your account has been created, but you must verify your email to continue. A verification link has been sent to your email address.”
- Block access to protected APIs until verification is complete.

```
HTTP/2 401 Unauthorized
{
  "status_code": 401,
  "detail": "Please verify your email address before accessing this feature. A verification email has been sent."
}
```
