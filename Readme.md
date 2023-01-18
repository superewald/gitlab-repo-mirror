Example repository for mirroring from github to gitlab using webhooks and pipeline triggers.

# How-To

## prepare gitlab project

1. create a [project scoped access token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html) with `write_repository, api` flags
1. create a [project scoped ci variable](https://docs.gitlab.com/ee/ci/variables/index.html#for-a-project) named `ACCESS_TOKEN` and store the access token within
1. create a [pipeline trigger token](https://docs.gitlab.com/ee/ci/triggers/#create-a-trigger-token)
1. add the following job to your `.gitlab-ci.yml`:
    ```yaml
    mirror_repository:
        image: alpine
        variables:
            SRC_ORIGIN: https://github.com/path-to-your-repository
            DEST_ORIGIN: https://oauth2:$ACCESS_TOKEN@gitlab.com/path-to-your-repository
        stage: mirror
        only:
            - triggers
        before_script:
            - apk add git
            - git config user.name ci-bot
            - git config user.email "ci-bot@example.xyz"
        script:
            - git clone $SRC_ORIGIN gh-mirror
            - cd gh-mirror
            - git remote remove origin
            - git remote add origin $DEST_ORIGIN
            - git push --prune --all
            - git push --prune --tags
    ```
## prepare github repository

1. create a webhook for push event and insert the pipeline trigger url as payload url