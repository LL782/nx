# Configuring CI Using Jenkins and Nx

Below is an example of an Jenkins setup, building and testing only what is affected.

```groovy
pipeline {
    agent none
    environment {
        NX_BRANCH = env.BRANCH_NAME.replace('PR-', '')
    }
    stages {
        stage('Pipeline') {
            parallel {
                stage('Main') {
                    when {
                        branch 'main'
                    }
                    agent any
                    steps {
                        sh "npx nx-cloud start-ci-run --distribute-on='5 linux-medium-js' --stop-agents-after='build'" // this line enables distribution
                        sh "npm ci"
                        sh "npx nx-cloud record -- nx format:check"
                        sh "npx nx affected --base=HEAD~1 -t lint test build --parallel=3"
                    }
                }
                stage('PR') {
                    when {
                        not { branch 'main' }
                    }
                    agent any
                    steps {
                        sh "npx nx-cloud start-ci-run --distribute-on='5 linux-medium-js' --stop-agents-after='build'" // this line enables distribution
                        sh "npm ci"
                        sh "npx nx-cloud record -- nx format:check"
                        sh "npx nx affected --base origin/${env.CHANGE_TARGET} -t lint test build --parallel=3"
                    }
                }
            }
        }
    }
}
```

### Get the Commit of the Last Successful Build

Unlike `GitHub Actions` and `CircleCI`, you don't have the metadata to help you track the last successful run on `main`. In the example below, the base is set to `HEAD~1` (for push) or branching point (for pull requests), but a more robust solution would be to tag an SHA in the main job once it succeeds and then use this tag as a base. See the [nx-tag-successful-ci-run](https://github.com/nrwl/nx-tag-successful-ci-run) and [nx-set-shas](https://github.com/nrwl/nx-set-shas) (version 1 implements tagging mechanism) repositories for more information.

We also have to set `NX_BRANCH` explicitly.
