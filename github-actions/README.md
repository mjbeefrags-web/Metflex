# CI configuration

GitHub Actions conventionally lives in `.github/workflows/`. No application files, dependency lockfiles, test commands, registry, or deployment target exist, so an executable build/deploy workflow cannot be authored without inventing commands. Once the Builder supplies reproducible lint, type-check, test, and image-build commands, place workflows under `.github/workflows/`; gate deployment on those checks and use protected environment credentials via OIDC where supported.
