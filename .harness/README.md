Build & Test Neuraxle on Harness CI
=======================================
This is a fork of Neuraxle project. This project is used for testing Harness CI's capabilities by the CME team at Harness. Please follow [these instructions](https://github.com/harness-community/Neuraxle/blob/trunk/.harness/README.md) on how to clone this repo and see the results yourself.


- [Harness Fast CI Blog Announcement](https://harness.io/blog/announcing-speed-enhancements-and-hosted-builds-for-harness-ci)
- [Get Started with Harness CI](https://harness.io/products/continuous-integration)

## Setting up this pipeline on Harness CI Hosted Builds

1. Create a [GitHub Account](https://github.com) or use an existing one

2. Fork [this repository](https://github.com/harness-community/Neuraxle/) into your GitHub account. 

3. If you are new to Harness CI, signup for [Harness CI](https://app.harness.io/auth/#/signup)
  * Select the `Continuous Integration` module and choose the `Starter pipeline` wizard to create your first pipeline using the forked repo from #2.
  * Go to the newly created pipeline and hit the `Triggers`tab. If everything went well, you should see two triggers auto-created. A `Pull Request`trigger and a `Push`trigger. For this exercise, we only need `Pull Request`trigger to be enabled. So, please disable or delete the `Push`trigger.

4. If you are an existing Harness CI user, create a new pipeline to use the cloud option for infrastructure and setup the PR trigger.

5. Add the pipeline.yaml stages in the YAML editor:

```
pipeline:
  name: test-neuraxle-public-release
  identifier: testneuraxlepublicrelease
  projectIdentifier: default
  orgIdentifier: default
  tags: {}
  properties:
    ci:
      codebase:
        connectorRef: scm-connector
        repoName: neuraxle
        build: <+input>
  stages:
    - stage:
        identifier: Test_Python_Package
        type: CI
        name: Test Python Package
        spec:
          cloneCodebase: true
          execution:
            steps:
              - step:
                  identifier: Install_dependencies
                  type: Run
                  name: Build and Test
                  spec:
                    connectorRef: account.harnessImage
                    image: python:3.6
                    shell: Sh
                    command: |-
                      python -m pip install --upgrade pip
                      pip install -r requirements.txt
                      python setup.py install
                      pip install flake8
                      # stop the build if there are Python syntax errors or undefined names
                      flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
                      # exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
                      flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
                      python setup.py test
                  when:
                    stageStatus: All
                  failureStrategies: []
          platform:
            os: Linux
            arch: Amd64
          runtime:
            type: Cloud
            spec: {}
        strategy:
          matrix:
            python:
              - 3.6
              - 3.7
              - 3.8
            maxConcurrency: 3
```
> Make sure you modify the connectors with the connectors you create. 

6. Create a Pull Request in a new branch by updating the project. (e.g. add a comment or new line). This should invoke a build in Harness CI

7. Merge the PR after the pipeline execution is successfull.

8. Enable GitHub Actions : The repository forked in Step 2 already has a GitHub Actions workflow file added. You can choose to enable this workflow from the Actions tab on GitHub.

9. Create any other Pull Request with a few source or test file changes.

10. This PR will trigger the Harness CI pipeline (as well as GitHub Actions workflow if enabled in Step-9). Here we are implementing testpythonpackage.yml workflow from Github Actions.
