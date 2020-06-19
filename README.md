# ASGI RateLimit

Limit user access frequency. Base on ASGI.

## Install

```
# Only install
pip install asgi-ratelimit

# Use redis
pip install asgi-ratelimit[redis]

# Use jwt
pip install asgi-ratelimit[jwt]

# Install all
pip install asgi-ratelimit[full]
```

## Usage

The following example will limit users under the `"default"` group to access `/second_limit` at most once per second and `/minute_limit` at most once per minute. And the users in the `"admin"` group have no restrictions.

```python
from typing import Tuple

from ratelimit import RateLimitMiddleware, Rule
from ratelimit.backends.redis import RedisBackend


async def AUTH_FUNCTION(scope) -> Tuple[str, str]:
    """
    Resolve the user's unique identifier and the user's group from ASGI SCOPE.

    If there is no group information, it should return "default".
    """
    return USER_UNIQUE_ID, GROUP_NAME


rate_limit = RateLimitMiddleware(
    ASGI_APP,
    AUTH_FUNCTION,
    RedisBackend(),
    {
        "/second_limit": [Rule(second=1), Rule(group="admin")],
        "/minute_limit": [Rule(minute=1), Rule(group="admin")],
    },
)

# Or in starlette/fastapi/index.py
app.add_middleware(
    RateLimitMiddleware,
    authenticate=AUTH_FUNCTION,
    backend=RedisBackend(),
    config={
        "/second_limit": [Rule(second=1), Rule(group="admin")],
        "/minute_limit": [Rule(minute=1), Rule(group="admin")],
    },
)
```

### Built-in auth functions

#### Client IP

```python
from ratelimit.auths.ip import client_ip
```

Obtain user IP through `scope["client"]` or `X-Real-IP`.

#### Json Web Token

```python
from ratelimit.auths.jwt import create_jwt_auth

jwt_auth = create_jwt_auth("KEY", "HS256")
```

Get `user` and `group` from JWT that in `Authorization` header.
