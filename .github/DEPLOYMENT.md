# GitHub Actions Deployment Setup

This document explains how to set up the GitHub Actions workflow for deploying the Firebase Extension.

## Required GitHub Secrets

To use the deployment workflow, you need to configure the following secrets in your GitHub repository settings (Settings > Secrets and variables > Actions):

### 1. `GOOGLE_CLOUD_SERVICE_ACCOUNT_KEY`
A JSON key for a Google Cloud service account with the necessary permissions to deploy Firebase Extensions.

**Security Note:** For enhanced security in production environments, consider using [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) instead of storing long-lived service account keys. This eliminates the need to store credentials in GitHub.

**How to create:**
1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Navigate to IAM & Admin > Service Accounts
3. Create a new service account or select an existing one
4. Grant the following roles:
   - `Firebase Extensions Publisher`
   - `Cloud Functions Developer`
   - `Service Account User`
5. Create and download a JSON key
6. Copy the entire JSON content and add it as a secret in GitHub

### 2. `FIREBASE_TOKEN` (Alternative to Service Account)
A Firebase CI token for authentication. This can be used instead of the service account key.

**How to create:**
```bash
firebase login:ci
```

Copy the token and add it as a secret in GitHub.

### 3. `FIREBASE_PROJECT_ID`
Your Firebase project ID (e.g., `my-project-12345`)

**How to find:**
- Check your Firebase Console URL: `https://console.firebase.google.com/project/[PROJECT_ID]`
- Or run: `firebase projects:list`

### 4. `FIREBASE_PUBLISHER_ID`
Your Firebase Extensions publisher ID (e.g., `your-publisher-id`)

**How to find:**
- Go to [Firebase Extensions Publisher Hub](https://console.firebase.google.com/project/_/extensions/publisher)
- Your publisher ID is shown in the dashboard

### 5. `FIREBASE_EXTENSION_ID`
The ID of your extension (e.g., `auth-send-message-to-slack`)

This should match the `name` field in your `extension.yaml` file.

## Workflow Triggers

The deployment workflow can be triggered in two ways:

### 1. Automatic Deployment on Version Tags
Push a version tag to trigger automatic deployment:

```bash
git tag v0.0.2
git push origin v0.0.2
```

The workflow will extract the version number from the tag and deploy the extension.

### 2. Manual Deployment
Trigger the workflow manually from the GitHub Actions tab:

1. Go to Actions > Deploy Firebase Extension
2. Click "Run workflow"
3. Optionally specify an extension ref
4. Click "Run workflow"

## Deployment Process

The workflow performs the following steps:

1. **Checkout code** - Gets the latest code from the repository
2. **Setup Node.js** - Installs Node.js 20 with npm caching
3. **Install dependencies** - Runs `npm ci` in the functions directory
4. **Lint code** - Runs ESLint to check code quality
5. **Build extension** - Compiles TypeScript to JavaScript
6. **Authenticate** - Authenticates with Google Cloud using the service account
7. **Setup Cloud SDK** - Installs and configures gcloud CLI
8. **Install Firebase CLI** - Installs the Firebase command-line tools
9. **Deploy extension** - Uploads the extension to Firebase Extensions

## Testing the Workflow

Before deploying to production, you can test the workflow:

1. Ensure all secrets are properly configured
2. Make a test commit to a feature branch
3. Manually trigger the workflow to verify it works
4. Check the Actions tab for any errors

## Troubleshooting

### Authentication Errors
- Verify that your service account has the correct permissions
- Ensure the JSON key is properly formatted (valid JSON)
- Check that the Firebase project ID is correct

### Build Errors
- Ensure all dependencies are listed in `package.json`
- Check that TypeScript compiles locally: `cd functions && npm run build`
- Verify Node.js version compatibility (should be 20)

### Deployment Errors
- Verify that your publisher ID and extension ID are correct
- Ensure the extension is properly registered in Firebase Extensions Publisher Hub
- Check that the `extension.yaml` file is valid

## Local Testing

Before pushing, you can test the extension locally:

```bash
# Install dependencies
cd functions
npm install

# Lint code
npm run lint

# Build extension
npm run build

# Test with Firebase emulator
npm test
```

## Additional Resources

- [Firebase Extensions Documentation](https://firebase.google.com/docs/extensions)
- [Firebase CLI Reference](https://firebase.google.com/docs/cli)
- [Firebase Extensions Publisher Guide](https://firebase.google.com/products/extensions/publisher)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
