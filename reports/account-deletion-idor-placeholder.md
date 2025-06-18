# Placeholder Submission for Issue #58: IDOR on Account Deletion

This pull request serves as a placeholder for the vulnerability reported at: [AIxBlock-2023#58](https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/58)

The issue describes an Insecure Direct Object Reference (IDOR) vulnerability in the account deletion endpoint: DELETE https://app.aixblock.io/api/users/{user_id}

Any authenticated user can delete another user's account by changing the `user_id` in the request. The server does not verify whether the requestor actually owns the account, and it deletes the target account permanently without any confirmation or access control validation.

** Proposed Fix **

To mitigate this vulnerability:

- Implement strict authorization checks so that only the **account owner** can initiate a deletion request for their own account.
- If a user attempts to delete another user's account, return a `403 Forbidden` error with a clear message.

### Example Unauthorized Response:

```http
HTTP/2 403 Forbidden
{
  "status_code": 403,
  "detail": "You are not allowed to delete this account."
}

