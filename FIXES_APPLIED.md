# Automated Security Fixes

14 fixes applied:

1. **Dockerfile** - Add a non-root user for build operations where possible
2. **Dockerfile** - Remove openssh-server unless absolutely required for the build process
3. **Dockerfile** - Remove nginx from the builder stage
4. **Dockerfile** - Remove sudo from the package list
5. **Dockerfile** - Remove explicit USER root and rely on proper ordering of commands before USER switch
6. **Dockerfile** - Add explicit restrictive permissions for gpg/source directory
7. **Dockerfile.tilt** - Add a non-root user and switch to it using the USER directive before the ENTRYPOINT
8. **Dockerfile.tilt** - Bind the debugger to localhost only (--listen=127.0.0.1:2345) to prevent remote access
9. **Dockerfile.tilt** - Add cleanup commands to remove apt cache and lists after installation
10. **Dockerfile.tilt** - Set restrictive permissions (700) on sensitive directories
11. **Dockerfile.dev** - Use a specific image tag or SHA256 digest for reproducible and secure builds. Use ARG to allow version specification.
12. **Dockerfile.dev** - Minimize root usage by setting proper file permissions before COPY and using --chown flag, or ensure the base image has proper permissions set.
13. **Dockerfile.dev** - Add a checksum verification step or use multi-stage builds with verified sources.
14. **Dockerfile.ui.tilt** - Add a USER directive to run the container as the built-in 'node' user (non-root) and ensure proper file ownership
