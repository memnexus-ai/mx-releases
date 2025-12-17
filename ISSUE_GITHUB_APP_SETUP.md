# Create GitHub App for cross-repo OpenAPI spec sync (replace PAT)

## Summary
Replace the Personal Access Token (GATEWAY_SYNC_TOKEN) with a GitHub App installation token to allow the mx-core-api workflow to checkout and push to memnexus-ai/mx-api-gateway (syncing contracts/openapi.yaml). This improves security, uses short-lived tokens, and follows GitHub best practices.

## Why
- No user PATs stored long-term; tokens are short-lived and revoked automatically
- Least-privilege, auditable permissions per repository
- Official GitHub approach (actions/create-github-app-token)

## Steps

1) Create a GitHub App (organization: memnexus-ai)
- Name: e.g., "MemNexus CI Bot"
- Webhooks: none required
- Repository permissions:
  - Contents: Read and write
  - Metadata: Read (default)
- Subscribe to events: none required

2) Install the App on these repositories
- memnexus-ai/mx-core-api
- memnexus-ai/mx-api-gateway

3) Generate and store credentials in mx-core-api
- In the App settings, generate a Private key (PEM)
- In mx-core-api repo settings → Actions:
  - Secrets: PRIVATE_KEY = <paste the App private key PEM>
  - Variables: APP_ID = <numeric App ID of the GitHub App>
  (Optionally store at org level if multiple repos will consume these.)

4) Update the workflow to use the App token (replacing PAT usage)
Edit .github/workflows/deploy-dev.yml and add the token step before checking out mx-api-gateway, then use that token for checkout:

```yaml
- uses: actions/create-github-app-token@v2
  id: app-token
  with:
    app-id: ${{ vars.APP_ID }}
    private-key: ${{ secrets.PRIVATE_KEY }}
    owner: memnexus-ai
    repositories: mx-api-gateway
    permission-contents: write

- name: Checkout API Gateway repository
  uses: actions/checkout@v4
  with:
    repository: memnexus-ai/mx-api-gateway
    token: ${{ steps.app-token.outputs.token }}
    persist-credentials: false
    path: mx-api-gateway
```

5) Verify
- Re-run the "Sync OpenAPI Spec to API Gateway" job (or the full workflow)
- Expect a commit to mx-api-gateway/contracts/openapi.yaml when there are changes

6) Cleanup
- Remove the legacy secret GATEWAY_SYNC_TOKEN from mx-core-api repository secrets
- Update internal documentation to reference the GitHub App approach

## Notes
- Installation tokens expire in ~1 hour and are revoked at job end by default
- Ensure the App remains installed on both repositories
- To rotate the private key, generate a new key in the App and update the PRIVATE_KEY secret
- Reference: https://github.com/actions/create-github-app-token

## Acceptance Criteria
- GitHub App created with Contents: Read/Write and Metadata: Read
- App installed on memnexus-ai/mx-core-api and memnexus-ai/mx-api-gateway
- APP_ID variable and PRIVATE_KEY secret configured in mx-core-api
- deploy-dev.yml updated to use actions/create-github-app-token and checkout with that token
- Workflow runs green and pushes changes to mx-api-gateway/contracts/openapi.yaml when a diff exists
- GATEWAY_SYNC_TOKEN removed from repo secrets
