<p>
  <a href="https://github.com/tbayaa/cypress-slack-video-upload-action/actions"><img alt="typescript-action status" src="https://github.com/actions/typescript-action/workflows/build-test/badge.svg"></a>
</p>

# Upload Cypress videos and screenshots directly to Slack thread

This GitHub action enables you to take the screenshots and videos generated by Cypress.
### This is a fixed fork of `trymbill/cypress-slack-video-upload-action`

## Inputs

### `action`

**Required** One of actions: start, finish, upload, where:
`start` - starts thread with `in progress` status
`finish` - updates thread messages with run status
`upload` - uploads assets

### `token`

**Required** Slack app token. See [Internal app tokens](https://slack.com/intl/en-ru/help/articles/215770388-Create-and-regenerate-API-tokens#internal-app-tokens)

- Create an app
- Under **Bot Token Scopes**, add `files:write` and `chat:write` permissions
- Install the app into your workspace
- Invite the bot to whatever channels you want to send the videos and screenshots to `/invite <botname>`
- Grab the `Bot User OAuth Token` from the `OAuth & Permissions` page
- Add that token as a secret to your GitHub repo's `Actions Secrets` found under `Settings -> Secrets` (in the examples below we call it `SLACK_TOKEN`)

### `channel`

**Required** Slack channel name to upload to

### `screenshots`

**Optional** The folder where Cypress stores screenshots on the build machine.

Default: `cypress/screenshots`

(this relative path resolves to `/home/runner/work/<REPO_NAME>/<REPO_NAME>/cypress/screenshots`)

If your project uses Cypress from the project root folder, the default value will work for you.
But if your project uses Cypress in a subfolder (like most monorepos), you'll need to provide the relative path to that folder
(i.e. `e2e/cypress/screenshots`).
(Don't include a trailing slash on your path!)

### `videos`

**Optional** The folder where Cypress stores videos on the build machine.

Default: `cypress/videos`

(this relative path resolves to `/home/runner/work/<REPO_NAME>/<REPO_NAME>/cypress/videos`)

If your project uses Cypress from the project root folder, the default value will work for you.
But if your project uses Cypress in a sub-folder (like most mono-repos), you'll need to provide the relative path to that folder
(i.e. `e2e/cypress/videos`).
(Don't include a trailing slash on your path!)

### `message-text`

**Required** Slack message text.

### `thread-id`

**Optional/Required** Thread ID to continue in (finish, upload)

### `status`

**Optional** Status of run

### `author`

**Required** Slack username of the author

## Examples

```yml
...

jobs:
  - name: 'Start thread in slack'
    uses: tbayaa/cypress-slack-video-upload-action@v1
    id: start
    with:
      token: ${{ secrets.SLACK_TOKEN }}
      channel: ${{ github.event.client_payload.channel }}
      message-text: "Starting Smoke Tests at ${{ github.event.client_payload.cypress_server }}"
      author: ${{ github.event.client_payload.slack_user }}
      action: start

  - name: Cypress run
    uses: cypress-io/github-action@v5

  - name: 'Update thread in slack'
    uses: tbayaa/cypress-slack-video-upload-action@v1
    if: always()
    with:
      token: ${{ secrets.SLACK_TOKEN }}
      channel: ${{ github.event.client_payload.channel }}
      message-text: "Finished Smoke Tests at ${{ github.event.client_payload.cypress_server }}"
      author: ${{ github.event.client_payload.slack_user }}
      action: finish
      thread-id: ${{ steps.start.outputs.thread-id }}
      status: ${{ job.status }}

  - name: 'Upload screenshots and videos to Slack'
    uses: tbayaa/cypress-slack-video-upload-action@v1
    if: failure()
    with:
      token: ${{ secrets.SLACK_TOKEN }}
      channel: ${{ github.event.client_payload.channel }}
      action: upload
      thread-id: ${{ steps.start.outputs.thread-id }}

```
