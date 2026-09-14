# React CI/CD Demo Project

## Project Description

This repository is a small React application built with Vite and configured to demonstrate CI/CD practices in a real-world frontend workflow. The app is a simple landing page that displays a GitHub Actions message and course-related text, as seen in `src/App.jsx`.

The project uses:

- React 18 for the UI
- Vite as the build tool and dev server
- Vitest + Testing Library for automated tests
- ESLint for static code quality checks
- GitHub Actions for CI/CD automation
- SonarQube for code quality analysis
- GitHub Pages deployment configuration

The key project metadata is defined in `package.json`:

- `npm run dev` starts the Vite development server
- `npm run build` creates the production build in the `dist` folder
- `npm run test` runs the test suite
- `npm run lint` checks for linting issues

The application also includes a `vite.config.js` configuration with test settings using `jsdom` and a `base` path set to `/react-cicd-actions/`, which is consistent with deployment to GitHub Pages.

## Installation

Before you begin, make sure you have Node.js 20 or later installed and a package manager such as npm available.

1. Clone the repository:

```bash
git clone <repository-url>
cd starter-code-cicd
```

2. Install project dependencies:

```bash
npm install
```

3. Start the application in development mode:

```bash
npm run dev
```

4. Build the app for production:

```bash
npm run build
```

5. Run the automated tests:

```bash
npm test
```

6. Run ESLint checks:

```bash
npm run lint
```

7. Preview the production build locally:

```bash
npm run preview
```

## GitHub Actions Usage

This project includes multiple workflow files under `.github/workflows/` that automate testing, build validation, and deployment.

### 1. Deploy React Pages at GitHub

File: `.github/workflows/DeployReactPages.yml`

This workflow is the main production deployment pipeline for the project. It is designed to ensure that the React app is tested, built, and then deployed to GitHub Pages.

It runs when:

- code is pushed to `master`
- code is pushed to `feature/*`
- the workflow is manually triggered with `workflow_dispatch`

The workflow also ignores changes to `README.md`, `docs/**`, and `.github/**` so that documentation-only updates do not trigger deployment.

The pipeline has three jobs:

1. `test`
   - Checks out the repository using `actions/checkout@v7`
   - Sets up Node.js 20 using `actions/setup-node`
   - Uses `actions/cache` to save and restore `~/.npm` based on the lock file
   - Runs `npm ci` to install dependencies from the lock file
   - Executes `npm test` to validate the app before building

2. `build`
   - Depends on the `test` job
   - Re-checks out the repository
   - Installs dependencies again
   - Runs `npm run build` to create the production bundle
   - Uploads the `dist` folder using `actions/upload-pages-artifact@v3`, which prepares the site for GitHub Pages deployment

3. `deploy`
   - Depends on the `build` job
   - Grants `pages: write` and `id-token: write` permissions so GitHub can publish the site securely
   - Uses the GitHub Pages environment
   - Runs `actions/deploy-pages@v4` to publish the static site

This workflow is the clearest example of a full CI/CD flow in this repository: test → build → deploy.

### 2. Deploy Distribution

File: `.github/workflows/deployDist.yml`

This workflow demonstrates a simpler deployment pipeline for the built distribution artifact. It follows almost the same pattern as the previous workflow but is more focused on artifact packaging and distribution.

The workflow triggers on:

- pushes to `master`
- pushes to `feature/*`
- manual dispatch

Jobs inside the workflow:

1. `test`
   - Checks out the code
   - Installs Node.js 20
   - Caches npm modules
   - Runs `npm ci`
   - Runs `npm test`

2. `build`
   - Depends on `test`
   - Runs `npm run build`
   - Uploads the `dist` folder as the artifact `dist-artifacts` with `actions/upload-artifact@v7`

3. `deploy`
   - Depends on `build`
   - Downloads the artifact using `actions/download-artifact@v7`
   - Runs a placeholder deploy command: `echo "Deploy successful..."`

This workflow is useful for learning how GitHub Actions can separate build output from deployment logic and how build artifacts can be passed between jobs.

### 3. Greeting Workflow

File: `.github/workflows/greetings.yml`

This is the simplest workflow in the project and is mainly used for demonstration purposes. It is triggered only by manual dispatch using `workflow_dispatch`.

The job is named `greet` and runs on `ubuntu-latest`. It executes:

```bash
echo "Hello, world!"
```

This file helps beginners understand the basic structure of a GitHub Actions workflow:

- `name`
- `on`
- `jobs`
- `runs-on`
- `steps`
- `run`

### 4. SonarQube Scan Workflow

File: `.github/workflows/sonarscan.yml`

This workflow integrates code quality analysis into the CI process. It runs when:

- a push is made to `master`
- a pull request is opened, synchronized, or reopened

The workflow contains a single job named `sonarqube`:

- uses `actions/checkout@v4` with `fetch-depth: 0` to get full repository history for more accurate analysis
- executes `SonarSource/sonarqube-scan-action` to scan the codebase
- passes in the required environment variables:
  - `SONAR_TOKEN`
  - `GITHUB_TOKEN`

This workflow is specifically designed to enforce code quality checks and catch maintainability, security, and bug issues before merging code.

### Workflow Summary

Across the repository, the GitHub Actions workflows teach the following patterns:

- automated testing before deployment
- build validation with Node.js and npm
- caching dependencies for speed
- artifact upload and download between jobs
- deployment to GitHub Pages
- manual triggers for simple tasks
- SonarQube integration for quality gates

These workflows are useful examples for learning how real-world frontend CI/CD systems are assembled and automated.

## SonarQube Usage

This project is configured for SonarQube / SonarCloud analysis through both the workflow file and the `sonar-project.properties` file.

### Sonar configuration

File: `sonar-project.properties`

```properties
sonar.projectKey=riteshcchaudhari-ui_react-cicd-actions
sonar.organization=riteshcchaudhari-ui
```

This config identifies the project in SonarCloud and links the repository to the correct project key and organization.

### Sonar workflow

File: `.github/workflows/sonarscan.yml`

This workflow:

- runs on pushes to `master`
- runs on pull requests opened or updated
- checks out the repository with full git history using `fetch-depth: 0`
- runs the SonarQube scan using `SonarSource/sonarqube-scan-action`
- passes in:
  - `SONAR_TOKEN` from GitHub secrets
  - `GITHUB_TOKEN` from GitHub secrets

This gives the project a code quality gate and helps detect bugs, vulnerabilities, security hotspots, and maintainability issues before the code is merged or deployed.

## What You Can Learn from This Project

This repository is a practical learning project for modern frontend CI/CD and automated quality checks. A developer can learn the following from it:

- React application basics with Vite
- Component rendering and simple UI composition in `src/App.jsx`
- Testing with React Testing Library and Vitest
- DOM-based assertions and behavior validation
- ESLint configuration for code quality enforcement
- GitHub Actions workflow design for CI/CD
- Build, test, and deployment stages as separate jobs
- GitHub Pages deployment using artifact upload and deploy actions
- SonarQube integration for code quality analysis
- CI/CD best practices such as caching dependencies, separating jobs, and using build artifacts
- Deployment automation patterns used in real production pipelines

This project is especially useful for developers who want to understand how a frontend app moves from local development to automated validation and deployment in GitHub-based workflows.
