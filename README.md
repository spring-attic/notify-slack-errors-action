# This repository is no longer actively maintained by VMware, Inc.


# notify-slack-errors-action

This action posts a summary build error message to Slack and enables you to report on the status of all jobs in the build. If no jobs failed, the action will not post a message to Slack. **Please note that this action is meant to be used in tandem with [track-build-errors-action](https://github.com/spring-projects/track-build-errors-action).**

On build error, this action will send a Slack notification detailing which jobs in the build failed, who the commit author was, and will link back to the original build:

![Screen Shot 2020-09-03 at 5 55 56 PM](https://user-images.githubusercontent.com/15992415/92188562-c9758b80-ee11-11ea-8b6a-15bb83af0d9e.png)

## Strategy
The [track-build-errors-action](https://github.com/spring-projects/track-build-errors-action) exports a job-specific errors file at the end of each failed job. This action combines all of the failed job messages and sends a single notification to Slack, which includes a link to the original build.

In order for this action to work, you must first download the errors folder in order to make the previously created error files available within the scope of the notify-slack job. This is shown in the example below.

Make sure to use this action at the end of your build process, and to require all other jobs to finish executing first by using the `needs:` key. Additionally, make sure to include `if: always()` to ensure that you always get notifications – if you don't include this, you will **not** receive notifications for earlier failed jobs. This is also shown in the example below.

If you'd like an example that demonstrates the usage of this action and the [track-build-errors-action](https://github.com/spring-projects/track-build-errors-action), see this [sample project](https://github.com/elliedori/sample-action-usage-project).

## Inputs

### `branch-name`
**Required** The name of the branch associated with this build. Defaults to "".

### `commit-owner`
**Required** The owner of the commit associated with this build. Defaults to "".

### `commit-sha`
**Required** The commit SHA associated with this build. Defaults to "".

### `repo-name`
**Required** The name of the repository associated with this build. Defaults to "".

### `run-id`
**Required** The unique run ID associated with this build. Defaults to "".

### `slack-webhook-url`
**Required** Your Slack webhook URL. Defaults to "". For more on how to set up a Slack webhook, see [this help page](https://api.slack.com/messaging/webhooks).


## Example usage

```
jobs:
  initiate_error_tracking: ...
  job_1: ...
  job_2: ...
  job_3: ...
  notify_result:
    name: Notify Slack on failure
    needs: [initiate_error_tracking, job_1, job_2, job_3]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Download errors folder
        uses: actions/download-artifact@v2
        with:
          name: errors
      - name: Send Slack message
        uses: spring-projects/notify-slack-errors-action@v0.01
        with:
          slack-webhook-url: ${{ secrets.SLACK_WEBHOOK_URL }}
          branch-name: ${{ github.ref }}
          commit-sha: ${{ github.sha }}
          commit-owner: ${{ github.actor }}
          repo-name: ${{ github.repository }}
          run-id: ${{ github.run_id }}
```
