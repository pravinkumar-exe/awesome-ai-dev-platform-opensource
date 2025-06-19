 This pull request corresponds to the vulnerability described in Issue #155(https://github.com/AIxBlock-2023/awesome-ai-dev-platform-opensource/issues/155) – SSRF in S3 Endpoint Validation.

 Bug Type: Server-Side Request Forgery (SSRF), Stack Trace Disclosure
 Severity: Critical

# Summary:
 The S3 storage configuration feature allows authenticated users to submit arbitrary s3_endpoint URLs
 along with any valid bucket name. The backend then makes a direct connection to the provided endpoint,
 resulting in an SSRF vulnerability. This also leaks signed AWS headers (SigV4), and returns detailed
 stack traces including exact file paths and internal code structure upon failure.

-  A valid-looking bucket name must be supplied to trigger the server-side request.

# Fix:
  - Block internal IP ranges (e.g., 127.0.0.1, 169.254.x.x, 10.x.x.x)
  - Allow only valid AWS S3 domains
  - Suppress sensitive stack trace data in error responses

**Proposed Code Fix**
```python
import re
import socket
import logging
from urllib.parse import urlparse
from rest_framework.exceptions import ValidationError

logger = logging.getLogger(__name__)

def is_safe_url(url):
    """
    Validates the s3_endpoint to:
    - Block internal/private IPs (e.g., 127.0.0.1, 169.254.169.254, etc.)
    - Allow only valid AWS S3 endpoints
    """
    try:
        parsed = urlparse(url)
        hostname = parsed.hostname
        if not hostname:
            return False

        addr_info = socket.getaddrinfo(hostname, None)
        for family, _, _, _, sockaddr in addr_info:
            ip = sockaddr[0]
            if ip.startswith(('127.', '169.254.', '10.', '192.168.', '172.')):
                return False

        # Allow only S3 endpoints
        allowed_pattern = r"^s3[.-]?(dualstack\.)?[a-z0-9-]+\.amazonaws\.com$"
        if not re.match(allowed_pattern, hostname):
            return False

        return True
    except Exception as e:
        logger.debug("is_safe_url validation failed: %s", e)
        return False


# In models.py where validate_connection() is defined

def validate_connection(self):
    if not self.bucket or not re.match(r"^[a-zA-Z0-9.\-_]{1,255}$", self.bucket):
        raise ValidationError("Invalid or empty bucket name.")

    if not is_safe_url(self.s3_endpoint):
        raise ValidationError("Invalid or unsafe S3 endpoint URL.")

    try:
        bucket_region = self.s3_client.get_bucket_location(
            Bucket=self.bucket.rstrip('\n')
        )["LocationConstraint"]
    except Exception as e:
        logger.debug("S3 connection validation error: %s", e)
        raise ValidationError("Could not connect to the specified S3 bucket. Ensure the endpoint and bucket are correct.")
```
