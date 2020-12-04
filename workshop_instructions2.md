# Workflow 2: Steps to set up release and publish package workflow

## Add a workflow that publishes an NPM package and creates a draft release

1. We are going to be using the [`release-drafter` Action](https://github.com/marketplace/actions/release-drafter).
    - Add a workflow file at this directory `.github/workflows/release.yml`:

        <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        </br>

      ```yaml
      #####################################
      #      Automate your workflow       #
      #   GitHub Universe Workshop 2020   #
      #####################################

      # This workflow will create a NPM package and a new release

      name: Publish and release

      on:
        push:
          # branches to consider in the event; optional, defaults to all
          branches:
            - main

      jobs:

        update_release_draft:
          runs-on: ubuntu-latest

          steps:
            # Drafts your next Release notes as Pull Requests are merged into "master"
            - uses: release-drafter/release-drafter@v5
              with:
                config-name: release-drafter-config.yml
              env:
                GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

        publish-gpr:
          runs-on: ubuntu-latest

          needs: update_release_draft

          steps:
            - uses: actions/checkout@v2

            - uses: actions/setup-node@v1
              with:
                node-version: 12
                registry-url: https://npm.pkg.github.com/

            - run: npm publish
              env:
                NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      ```
    </details>

    - Notice how this workflow calls for a special config file called `release-drafter-config.yml`. This is a config file that allows you to add more customization. So let's go ahead and add a config file for this action at `.github/release-drafter-config.yml`:
        <details>
        <summary><b>Click here to view file contents to copy:</b></summary>
        </br>

      ```yaml
      name-template: 'v$RESOLVED_VERSION 🌈'
      tag-template: 'v$RESOLVED_VERSION'
      categories:
        - title: 'Changes included in this release'
          labels:
            - 'release'
      change-template: '- $TITLE @$AUTHOR (#$NUMBER)'
      change-title-escapes: '\<*_&' # You can add # and @ to disable mentions, and add ` to disable code blocks.
      version-resolver:
        major:
          labels:
            - 'major'
        minor:
          labels:
            - 'minor'
        patch:
          labels:
            - 'patch'
        default: patch
      template: |
        # 🚀 You did it!

        $CHANGES
      ```
      </details>

    - You can read more about the configuration options for this Action [here](https://github.com/marketplace/actions/release-drafter#configuration-options).
    - This action will create a release with release notes when the pull request is merged into the `main` branch.

## Watch your actions come to life!

1. Merge the pull request

1. Click on the `Code` tab and click on `Packages` on the right sidebar to find your package

1. Click on `Releases` on the right sidebar to find your new draft Release


## Additional Exercise

Customize your `release-drafter` Action such that it is able to categorize release notes based on labels that were added to pull requests. For example, if the label `bugfix` is added to a pull request, then when the pull request is merged, the Action would add the pull request to the release notes and show it as a Bugfix. Here's an example:

<img src="https://github.com/release-drafter/release-drafter/blob/master/design/screenshot-2.png" align-=center width=400>

   - You can find out how to add more customizations from the configuration options for this Action [here](https://github.com/marketplace/actions/release-drafter#configuration-options).
        <details>
        <summary><b>Click here to see the example</b></summary>
        </br>

        ```yaml
        name-template: 'v$RESOLVED_VERSION 🌈'
        tag-template: 'v$RESOLVED_VERSION'
        categories:
          - title: '🚢 Features'
            labels:
              - 'feature'
              - 'enhancement'
          - title: '🐛 Bug Fixes'
            labels:
              - 'fix'
              - 'bugfix'
              - 'bug'
          - title: '🧰 Maintenance'
            label: 'chore'
        change-template: '- $TITLE @$AUTHOR (#$NUMBER)'
        change-title-escapes: '\<*_&' # You can add # and @ to disable mentions, and add ` to disable code blocks.
        version-resolver:
          major:
            labels:
              - 'major'
          minor:
            labels:
              - 'minor'
          patch:
            labels:
              - 'patch'
          default: patch
        template: |
          ## Changes

          $CHANGES
        ```

        <b>The config file above allows you to add more labels like 'feature' and 'bug'.</b>
  </details>
