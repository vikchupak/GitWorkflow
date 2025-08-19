- Checkout
  - Checkout code
  - Lint checks, formatting
- Build App - transform source code into something runnable
    - Install dependencies
    - Run tests (Node.js does not need to be built before tests)
    - Build App
      - compile classes, `dist/` folder
- Run tests (Java apps need to be build before tests)
  - Runs automated tests: unit tests, integration tests, end-to-end tests. **These tests are conducted automatically, without the need for manual interventions.**
- Build Artifact (Package) - produce distributable file.
  - Docker Image, Jar
- Push
  - Push Artifact to Repo
- Deploy
  - Deploy the image/artifact to an environment.

https://gitlab.com/devopsbootcamp8550504/08-jenkins/01-02-03-04-05-cicdwithjenkins/-/blob/pipeline-aws-deploy/Jenkinsfile?ref_type=heads
