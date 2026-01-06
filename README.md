# Purpose

This repository provides a GitHub Action and Azure DevOps Template for uploading files to Flipdish's redundant CDN infrastructure. It uploads files to both Azure Blob Storage and AWS S3, with optional CDN cache purging for Azure Front Door, AWS CloudFront, and Cloudflare.

# GitHub Action: CDN Upload `flipdishbytes/flipdish-cdn-upload@v1.3`

To use this CDN upload action, add it to your pipeline workflow YAML file.

### How it works?

1. Validates source directory exists and counts files to upload
2. Uploads files to Azure Blob Storage container using Azure CLI
3. Uploads files to AWS S3 bucket using AWS CLI
4. Optionally purges CDN cache (AWS CloudFront, Cloudflare)

### How to use?

#### Basic Upload

```yaml
name: Upload to CDN

on:
  push:
    branches: [main]
    paths:
      - "dist/**"

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build assets
        run: npm run build

      - name: Upload to CDN
        uses: flipdishbytes/flipdish-cdn-upload@v1.3
        with:
          container-name: "your-app-name"
          source-directory: "./dist/assets"
          azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
          aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
```

#### With Container Cleaning

Delete orphaned files (files in container but not in source) after uploading:

```yaml
- name: Upload to CDN
  uses: flipdishbytes/flipdish-cdn-upload@v1.3
  with:
    container-name: "your-app-name"
    source-directory: "./dist/assets"
    clean-container: "true"
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_AWS_SECRET_ACCESS_KEY }}
```

#### With CDN Cache Purge

Purge CDN cache after upload to ensure new files are immediately available:

```yaml
- name: Upload to CDN
  uses: flipdishbytes/flipdish-cdn-upload@v1.3
  with:
    container-name: "your-app-name"
    source-directory: "./dist/assets"
    purge-cdn: "true"
    # Upload credentials
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
    # AWS CloudFront purge
    aws-cloudfront-distribution-id: ${{ vars.CDN_CLOUDFRONT_DISTRIBUTION_ID }}
    aws-cloudfront-invalidation-key-id: ${{ secrets.CDN_CLOUDFRONT_INVALIDATION_KEY_ID }}
    aws-cloudfront-invalidation-secret-key: ${{ secrets.CDN_CLOUDFRONT_INVALIDATION_SECRET_KEY }}
    # Cloudflare purge
    cloudflare-zone-id: ${{ vars.CDN_CLOUDFLARE_ZONE_ID }}
    cloudflare-api-token: ${{ secrets.CDN_CLOUDFLARE_API_TOKEN }}
```

#### With Custom Domain Cache Purge (Optional)

> **Note:** Most users do NOT need this. Only use `cloudflare-custom-domain` if you have a custom domain (e.g., `menu.yourrestaurant.com`) that points to your CDN container. If you only use the standard `cdn.flipdish.com` URLs, skip this parameter entirely.

When a custom domain is provided, two separate cache purge requests are made:
1. `cdn.flipdish.com/your-app-name/` (always purged)
2. `your-custom-domain.com/*` (entire custom domain, only if specified)

```yaml
- name: Upload to CDN
  uses: flipdishbytes/flipdish-cdn-upload@v1.3
  with:
    container-name: "your-app-name"
    source-directory: "./dist/assets"
    purge-cdn: "true"
    # Upload credentials
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
    # Cloudflare purge
    cloudflare-zone-id: ${{ vars.CDN_CLOUDFLARE_ZONE_ID }}
    cloudflare-api-token: ${{ secrets.CDN_CLOUDFLARE_API_TOKEN }}
    # Only add this if you have a custom domain pointing to your container!
    cloudflare-custom-domain: "menu.yourrestaurant.com"
```

#### Dry Run Mode

Test the upload without actually uploading files:

```yaml
- name: Upload to CDN (Dry Run)
  uses: flipdishbytes/flipdish-cdn-upload@v1.3
  with:
    container-name: "your-app-name"
    source-directory: "./dist/assets"
    dry-run: "true"
    azure-connection-string: ${{ secrets.CDN_AZURE_CONNECTION_STRING }}
    aws-access-key-id: ${{ secrets.CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY }}
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `container-name` | Yes | - | Name of the CDN container (e.g., fonts, icons, menus) |
| `source-directory` | Yes | - | Local directory path containing files to upload |
| `azure-connection-string` | Yes | - | Azure Storage connection string |
| `aws-access-key-id` | Yes | - | AWS Access Key ID for S3 upload |
| `aws-secret-access-key` | Yes | - | AWS Secret Access Key for S3 upload |
| `dry-run` | No | `false` | Perform a dry run without actually uploading |
| `clean-container` | No | `false` | Delete orphaned files in container after uploading |
| `purge-cdn` | No | `false` | Purge CDN cache after upload |
| `aws-cloudfront-*` | No | - | AWS CloudFront credentials for cache invalidation |
| `cloudflare-zone-id` | No | - | Cloudflare Zone ID for cache purge |
| `cloudflare-api-token` | No | - | Cloudflare API Token with cache purge permissions |
| `cloudflare-custom-domain` | No | - | **Optional** - Only needed if you have a custom domain pointing to your CDN container. When provided, purges the entire custom domain in addition to `cdn.flipdish.com/container/` |

### Outputs

| Output | Description |
|--------|-------------|
| `azure-files-uploaded` | Number of files uploaded to Azure |
| `aws-files-uploaded` | Number of files uploaded to AWS |
| `azure-url` | Direct Azure Blob Storage URL |
| `aws-url` | Direct AWS S3 URL |

# Azure DevOps Template: CDN Upload `- template: '.azure/upload-to-cdn.yml@CdnUpload'`

To use this CDN upload template, add it to your pipeline YAML file.

### How it works?

1. Validates source directory exists and counts files to upload
2. Uploads files to Azure Blob Storage container using Azure CLI
3. Uploads files to AWS S3 bucket using AWS CLI
4. Optionally purges CDN cache (AWS CloudFront, Cloudflare)

### How to use?

#### Basic Upload

```yaml
trigger:
  branches:
    include:
      - main

resources:
  repositories:
    - repository: CdnUpload
      endpoint: "flipdishbytes"
      type: github
      name: flipdishbytes/flipdish-cdn-upload
      ref: main
      trigger: none

stages:
  - stage: "deploy"
    displayName: "Deploy to CDN"
    jobs:
      - job: BuildAndUpload
        displayName: "Build and Upload"
        pool:
          vmImage: "ubuntu-latest"
        steps:
          - checkout: self

          - script: npm run build
            displayName: "Build assets"

          - template: ".azure/upload-to-cdn.yml@CdnUpload"
            parameters:
              containerName: "your-app-name"
              sourceDirectory: "$(Build.SourcesDirectory)/dist/assets"
              azureConnectionString: $(CDN_AZURE_CONNECTION_STRING)
              aws-access-key-id: $(CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID)
              aws-secret-access-key: $(CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY)
```

#### With Container Cleaning

```yaml
- template: ".azure/upload-to-cdn.yml@CdnUpload"
  parameters:
    containerName: "your-app-name"
    sourceDirectory: "$(Build.SourcesDirectory)/dist/assets"
    cleanContainer: true
    azureConnectionString: $(CDN_AZURE_CONNECTION_STRING)
    aws-access-key-id: $(CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID)
    aws-secret-access-key: $(CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY)
```

#### With CDN Cache Purge

```yaml
- template: ".azure/upload-to-cdn.yml@CdnUpload"
  parameters:
    containerName: "your-app-name"
    sourceDirectory: "$(Build.SourcesDirectory)/dist/assets"
    purgeCdn: true
    # Upload credentials
    azureConnectionString: $(CDN_AZURE_CONNECTION_STRING)
    aws-access-key-id: $(CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID)
    aws-secret-access-key: $(CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY)
    # AWS CloudFront purge
    awsCloudfrontDistributionId: $(CDN_CLOUDFRONT_DISTRIBUTION_ID)
    awsCloudfrontInvalidationKeyId: $(CDN_CLOUDFRONT_INVALIDATION_KEY_ID)
    awsCloudfrontInvalidationSecretKey: $(CDN_CLOUDFRONT_INVALIDATION_SECRET_KEY)
    # Cloudflare purge
    cloudflareZoneId: $(CDN_CLOUDFLARE_ZONE_ID)
    cloudflareApiToken: $(CDN_CLOUDFLARE_API_TOKEN)
```

#### With Custom Domain Cache Purge (Optional)

> **Note:** Most users do NOT need this. Only use `cloudflareCustomDomain` if you have a custom domain (e.g., `menu.yourrestaurant.com`) that points to your CDN container. If you only use the standard `cdn.flipdish.com` URLs, skip this parameter entirely.

When a custom domain is provided, two separate cache purge requests are made:
1. `cdn.flipdish.com/your-app-name/` (always purged)
2. `your-custom-domain.com/*` (entire custom domain, only if specified)

```yaml
- template: ".azure/upload-to-cdn.yml@CdnUpload"
  parameters:
    containerName: "your-app-name"
    sourceDirectory: "$(Build.SourcesDirectory)/dist/assets"
    purgeCdn: true
    # Upload credentials
    azureConnectionString: $(CDN_AZURE_CONNECTION_STRING)
    aws-access-key-id: $(CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID)
    aws-secret-access-key: $(CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY)
    # Cloudflare purge
    cloudflareZoneId: $(CDN_CLOUDFLARE_ZONE_ID)
    cloudflareApiToken: $(CDN_CLOUDFLARE_API_TOKEN)
    # Only add this if you have a custom domain pointing to your container!
    cloudflareCustomDomain: "menu.yourrestaurant.com"
```

#### Dry Run Mode

```yaml
- template: ".azure/upload-to-cdn.yml@CdnUpload"
  parameters:
    containerName: "your-app-name"
    sourceDirectory: "$(Build.SourcesDirectory)/dist/assets"
    dryRun: true
    azureConnectionString: $(CDN_AZURE_CONNECTION_STRING)
    aws-access-key-id: $(CDN_YOUR_APP_NAME_AWS_ACCESS_KEY_ID)
    aws-secret-access-key: $(CDN_YOUR_APP_NAME_AWS_SECRET_ACCESS_KEY)
```

### Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `containerName` | Yes | - | Name of the CDN container (e.g., fonts, icons, menus) |
| `sourceDirectory` | Yes | - | Local directory path containing files to upload |
| `azureConnectionString` | Yes | - | Azure Storage connection string |
| `awsAccessKeyId` | Yes | - | AWS Access Key ID for S3 upload |
| `awsSecretAccessKey` | Yes | - | AWS Secret Access Key for S3 upload |
| `dryRun` | No | `false` | Perform a dry run without actually uploading |
| `cleanContainer` | No | `false` | Delete orphaned files in container after uploading |
| `purgeCdn` | No | `false` | Purge CDN cache after upload |
| `awsCloudfront*` | No | - | AWS CloudFront credentials for cache invalidation |
| `cloudflareZoneId` | No | - | Cloudflare Zone ID for cache purge |
| `cloudflareApiToken` | No | - | Cloudflare API Token with cache purge permissions |
| `cloudflareCustomDomain` | No | - | **Optional** - Only needed if you have a custom domain pointing to your CDN container. When provided, purges the entire custom domain in addition to `cdn.flipdish.com/container/` |
