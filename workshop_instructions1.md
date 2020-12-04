# Universe Workshop Instructions: Automate Your Workflows with GitHub Actions and GitHub Packages

:bookmark: The following instructions will take on the assumption that you will be completing the steps on the GitHub UI. Feel free to use the terminal or any other GUIs you are comfortable with.

![UniversePresentation](https://user-images.githubusercontent.com/38323656/99338458-dcf99580-2849-11eb-9d17-a2e41376f721.png)

# Workflow 1: Steps to set up CI workflow

## Create a repository

1. Log into your GitHub account and create a new repository

    - Give it a **name**, e.g. `universe-automation`, and a **description**
    - It does not matter if you set the visiblity to **Private** or **Public**
    - Select `Add a README file`
    - Click `Create repository`

## Set up `npm` project files

:bulb: For the purpose of this exercise, we will be committing directly to your default branch (e.g. **main**) so you don't need to create a new branch beforehand

1. Add these files to the root of your repository; `package.json`, and `index.js`, using one of the following approaches:

    - **If you have [`node`](https://nodejs.org/en/) installed**, and you have cloned the repository onto your local machine, run the following commands:

      ```sh
      npm init -y
      npm install
      ```
      - Create an `index.js` file, see content description below
      - Update your `package.json` file, see content description below
      - Not sure if you have `node` installed? Run `node -v` in your terminal and see if it returns a version number or not.
      - To avoid committing your `node_modules` directory which is created after running `npm install`, add a `.gitignore` file to the root of your repository containing one line: `node_modules/`

    - **If you do not have [`node`](https://nodejs.org/en/) installed**, then you can create the project files manually and copy the content into each;
      - `package.json`: Contains your node project meta data

        <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        </br>
        :bulb: <b>Replace the placeholders OWNER with your GitHub handle or organization name (if you created the repository in an organization), and REPOSITORY_NAME.</b>
        </br>
        :warning: <b>Your package name cannot contain uppercase letters! If your values for OWNER and/or REPOSITORY contains uppercase letters, change them to lowercase before proceeding.</b>

        ```json
        {
          "name": "@<owner>/<repository-name>",
          "version": "1.0.0",
          "description": "Demo repository for GitHub Universe 2020",
          "main": "index.js",
          "scripts": {
            "test": "echo \"No tests specified, but that's alright for now!\" && exit 0"
          },
          "repository": {
            "type": "git",
            "url": "git+https://github.com/<owner>/<repository-name>.git"
          },
          "keywords": [],
          "author": "",
          "license": "ISC",
          "bugs": {
            "url": "https://github.com/<owner>/<repository-name>/issues"
          },
          "homepage": "https://github.com/<owner>/<repository-name>#readme",
          "dependencies": {}
        }
        ```
        </details>

      - `index.js`: Your source code
        <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        </br>

        ```js
        console.log("Hello, GitHub Universe!")
        ```
        </details>
    
    - Make sure your changes are committed to your default branch before proceeding

## Add a CI Workflow

1. Create the following file to the default branch:

    - `.github/workflows/ci.yml`
      - This will be the workflow file taking care of building and testing your source code
      - The file content will be empty for now

1. After the above steps are finished, you should have the following files in your repository;
    - `package.json`
    - `index.js`
    - `.github/workflows/ci.yml`
    - If you created the files using `npm` on your local machine, you should also have
      - `package-lock.json`
      - `.gitignore`

1. :tada: If everything looks fine so far, it's time to start creating our CI workflow!

1. Go to `.github/workflows/ci.yml` and enter edit mode by clicking the pencil :pencil: icon
        <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        </br>

      ```yaml
      #####################################
      #      Automate your workflow       #
      #   GitHub Universe Workshop 2020   #
      #####################################

      # This workflow will run CI on your codebase, label your PR, and comment on the result

      name: MYWORKFLOW

      on:
        pull_request: # the workflow will trigger on every pull request event

      jobs:
        build:
          runs-on: ubuntu-latest

          strategy:
            matrix:
              node-version: [12.x, 14.x] # matrix for building and testing your code across multiple node versions

          steps:
            - name: Checkout
              uses: actions/checkout@v2
            - name: Build node version ${{ matrix.node-version }}
              uses: actions/setup-node@v1
              with:
                  node-version: ${{ matrix.node-version }}
            - run: npm install
            - run: npm run build --if-present
            - run: npm test

        label:
          runs-on: ubuntu-latest

          needs: build #this ensures that we only trigger the label job if ci is successful

          steps:
            - name: Checkout
              uses: actions/checkout@v2
            - uses: actions/github-script@v3
              with:
                github-token: ${{ secrets.GITHUB_TOKEN }}
                script: |
                  github.issues.addLabels({
                    issue_number: context.issue.number,
                    owner: context.repo.owner,
                    repo: context.repo.repo,
                    labels: ['release']
                  })

        comment:
          runs-on: ubuntu-latest

          needs: [build, label]

          steps:
            - name: Checkout
              uses: actions/checkout@v2
            - name: Comment on the result
              uses: actions/github-script@v3
              with:
                github-token: ${{ secrets.GITHUB_TOKEN }}
                script: |
                  github.issues.createComment({
                    issue_number: context.issue.number,
                    owner: context.repo.owner,
                    repo: context.repo.repo,
                    body: `
                    Great job **@${context.payload.sender.login}**! Your CI passed, and the PR has been automatically labelled.

                    Once ready, we will merge this PR to trigger the next part of the automation :rocket:
                    `
                  })
      ```
      </details>

- :warning: `yaml` syntax relies on indentation, please make sure that this is not changed
- Update the name of your workflow
- Within the `comment` job, you can edit the body of the comment as you see fit
- Commit the changes to your default branch

- **Knowledge Check**

  - How many jobs are there in the workflow?
    <details><summary><b>Answer</b></summary>
    The workflow contains three jobs:

    - a build-job,
    - a label-job,
    - a comment-job
  </details>

  - What is the event that will trigger this workflow?
    <details><summary><b>Answer</b></summary>
    The workflow is triggered by any pull request events.
    </details>

  - Does this workflow use a build matrix?
    <details><summary><b>Answer</b></summary>
    Yes, this workflow will build and test across multiple node versions.
    </details>

:tada: Awesome, now our CI workflow should be complete!

## Test the CI Workflow

1. To test our CI workflow, we need to trigger it. And the event that triggers it is `pull_request`. So let's create a pull request to get this workflow running

1. Go to your `README.md` file and enter edit mode

1. Make some changes/add some text

1. Commit your changes to a new branch e.g. `update-readme` and click **Commit changes**

1. In the pull request, leave the title and the body to default and click **Create pull request**

1. In your PR, click on the **Checks** tab
    - You should now see your workflow kicking off, and executing all the steps defined in `.github/workflows/ci.yml`

1. After the workflow has completed, check that the following is true in your PR;
    - A label `release` has been added
    - A comment is added to the PR with the text corresponding to what you defined in the `comment` job inside `.github/workflows/ci.yml`

1. If your workflow fails, inspect the log output:
    - Which job failed?
    - Did you update your `package.json` file correctly?
    - Does the log indicate any syntax errors with your CI workflow file?

:warning: Do not merge your pull request just yet! There's more to do.

## [Click here to get started with Workflow 2](./workshop_instructions2.md)