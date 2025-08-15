## Release 1.0.0 — Switch from Bitnami Redis Chart

> **⚠️ Breaking Change** — This affects you if you're using Redis.

### Secrets

With the new Redis chart, the password is now set in a Redis configuration block inside a Secret.

**Example:**

```yaml
redis.conf: |
  requirepass secret1234
  maxmemory 50mb
```
The chart supports an external secret for this secret as well.

### Service name change:

The new redis service is named 
```
my-app-redis
```

Previously it was named:
```
my-app-redis-master
```

Update the application configuration to match the new service name.


