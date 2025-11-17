# flipdish-cdn-upload

This repo implements a GitHub action task to upload files to Flipdish's redundant CDN infrastructure.

## How to use

Add the CDN upload action to your workflow:

#### Basic Upload

```yaml
name: Upload to CDN

on:
  push:
    branches: [main]
    paths:
      - 'dist/**'

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build assets
        run: npm run build

      - name: Upload to CDN
        uses: flipdishbytes/flipdish-redundant-cdn@v1.0
        with:
          container-name: 'your-app-name'
          source-directory: './dist/assets'
          azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
          aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
```

#### With Container Cleaning

Delete all existing files before uploading:

```yaml
- name: Upload to CDN
  uses: flipdishbytes/flipdish-redundant-cdn@v1.0
  with:
    container-name: 'your-app-name'
    source-directory: './dist/assets'
    clean-container: 'true'  # Deletes all existing files before upload
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
```

#### With CDN Cache Purge

Purge CDN cache after upload to ensure new files are immediately available:

```yaml
- name: Upload to CDN
  uses: flipdishbytes/flipdish-redundant-cdn@v1.0
  with:
    container-name: 'your-app-name'
    source-directory: './dist/assets'
    purge-cdn: 'true'
    # Azure Front Door purge
    azure-frontdoor-client-id: ${{ vars.CDN_FRONTDOOR_PURGE_CLIENT_ID }}
    azure-frontdoor-client-secret: ${{ secrets.CDN_FRONTDOOR_PURGE_CLIENT_SECRET }}
    azure-frontdoor-tenant-id: ${{ vars.CDN_FRONTDOOR_PURGE_TENANT_ID }}
    azure-frontdoor-subscription-id: ${{ vars.ARM_SUBSCRIPTION_ID }}
    azure-frontdoor-profile-name: ${{ vars.CDN_FRONTDOOR_PROFILE_NAME }}
    azure-frontdoor-endpoint-name: ${{ vars.CDN_FRONTDOOR_ENDPOINT_NAME }}
    azure-resource-group: ${{ vars.CDN_FRONTDOOR_RESOURCE_GROUP }}
    # AWS CloudFront purge
    aws-cloudfront-distribution-id: ${{ vars.CDN_CLOUDFRONT_DISTRIBUTION_ID }}
    aws-cloudfront-invalidation-key-id: ${{ secrets.CDN_CLOUDFRONT_INVALIDATION_KEY_ID }}
    aws-cloudfront-invalidation-secret-key: ${{ secrets.CDN_CLOUDFRONT_INVALIDATION_SECRET_KEY }}
    # Upload credentials
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
```

**Action Outputs:**
- `azure-files-uploaded` - Number of files uploaded to Azure
- `aws-files-uploaded` - Number of files uploaded to AWS
- `azure-url` - Direct Azure Blob Storage URL
- `aws-url` - Direct AWS S3 URL
