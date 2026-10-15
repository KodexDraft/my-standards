## Active Plugins

- The `my-standards` plugin is active. Its conventions apply to all code in this project.
- Use the `api-endpoints` skill (`my-standards:api-endpoints`) when adding new routes.

## Commits

Never run `git commit` or `git push` by hand. When asked to commit or push, use one of the `my-standards` commands:

- `my-standards:commit-main`: commit directly on `main` and push.
- `my-standards:commit-branch`: commit on a branch (never `main`) and push, no PR.
- `my-standards:commit-push-pr`: commit, push the branch and open a pull request.

If it is unclear which one is wanted, ask.
