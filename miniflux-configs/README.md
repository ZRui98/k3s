# Miniflux configs

miniflux rss

## prereqs

set up with some secrets before hand
```
kubectl create secret -n miniflux generic miniflux-credentials \
   --from-file=admin-password.secret \
   --from-file=database-url.secret \
   --from-file=postgres-password.secret
```