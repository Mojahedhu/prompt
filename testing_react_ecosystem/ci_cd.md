To automate your testing and deployment, you can use GitHub Actions for the Continuous Integration (CI) side—running your tests and builds—while letting Vercel handle the Continuous Deployment (CD) side natively. Vercel is designed to connect directly to your GitHub repository and automatically deploy when code is merged, making a manual deployment script unnecessary for most workflows.

Here is the setup for a CI pipeline that runs your test suite and builds the application using `pnpm` before any code is allowed to deploy.

### 1. The GitHub Actions CI File

Create a new file in your project directory at exactly this path: `.github/workflows/ci.yml`.

```yaml
name: CI Pipeline

on:
  push:
    branches: ["main", "dev"]
  pull_request:
    branches: ["main", "dev"]

jobs:
  test-and-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 9 # Adjust to your specific pnpm version

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"

      - name: Install Dependencies
        run: pnpm install --frozen-lockfile

      - name: Run Unit & Integration Tests
        run: pnpm test

      - name: Build Application
        run: pnpm run build
```

---

### 2. Steps to Connect GitHub, CI, and Vercel

1. **Commit the Workflow:** Save the `ci.yml` file, commit it, and push it to your GitHub repository.
2. **Connect Vercel to GitHub:**

- Log into your Vercel dashboard.
- Click **Add New Project** and select **Import from Git**.
- Connect your GitHub account and select your repository.

3. **Configure Vercel Deployments:**

- In Vercel, leave the default build commands (Vercel automatically detects Next.js or standard React setups).
- Go to the project **Settings > Git > Ignored Build Step**.
- Vercel will now automatically create a "Preview Deployment" whenever you push to `dev` or open a Pull Request, and a "Production Deployment" when you merge into `main`.

4. **Protect Your Branches (Optional but Recommended):**

- In your GitHub Repository Settings, go to **Branches > Add branch ruleset**.
- Target the `main` and `dev` branches.
- Require status checks to pass before merging, selecting your `test-and-build` action. This ensures code cannot be merged and deployed to Vercel if your tests fail.

---

### 3. Git Branching Strategy Commands

Using a `main` (production), `dev` (staging/testing), and `feature` (active work) strategy keeps your development isolated and safe. Here are the commands to execute this workflow on your local machine.

**Setting up the base branches initially:**

```bash
# Ensure you are on main and it is up to date
git checkout main
git pull origin main

# Create and switch to the dev branch based on main
git checkout -b dev
git push -u origin dev

```

**Starting work on a new feature:**

```bash
# Always branch off from dev, not main
git checkout dev
git pull origin dev

# Create a new feature branch
git checkout -b feature/user-dashboard

```

**Saving and pushing your feature work:**

```bash
# Stage all changes
git add .

# Commit with a descriptive message
git commit -m "Add responsive layout for user dashboard"

# Push the feature branch to GitHub
git push -u origin feature/user-dashboard

```

**Merging the feature into dev:**
_Note: In an enterprise setting, you would open a Pull Request on GitHub to merge `feature/user-dashboard` into `dev`. If you want to merge it locally instead:_

```bash
# Switch back to the dev branch
git checkout dev
git pull origin dev

# Merge the feature branch into dev
git merge feature/user-dashboard

# Push the updated dev branch to GitHub (This triggers CI and Vercel preview)
git push origin dev

# Delete the local feature branch to keep your workspace clean
git branch -d feature/user-dashboard

```

**Deploying to Production:**
When the `dev` branch has accumulated enough tested features and is ready for the live environment, you merge `dev` into `main`.

```bash
git checkout main
git pull origin main
git merge dev
git push origin main # This triggers the Vercel Production deployment

```
