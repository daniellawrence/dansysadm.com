Title:  continous delivery of test environment using docker
Date: 2012-02-01 10:20
Category: blog
Tags: docker

"Is anyone using systest?", "Anyone care if I deploy my branch to systest?", "Can I redeploy systest?"

A 1 command process to deploy your application to an environment is excellent,


How we make this all happen?
----------------------------

We got a nice item across the line in the same CI environment, pushing all test execution into a docker container.

The first thing the ci will do for a new test run is to create a brand new container just for this branch.

```
docker build -t project-name:branch-name .
```

This has everything need to run the test suite on our target platform; nodejs(, npm), python(, tox) & nginx

Then we can run the container with a few environment varibles

```
docker run -i -t -d -e BRANCH=branch-name -e COMMIT=gitref project-name:branch-name /path/to/ci-script.bash
TEST_RC=${?}
```

The _ci-script.bash_ is a script that is going to take care of running `tox` and reporting back the results.

If the test run was successful we start the magic...

As the target is a system testing environment, I don't realy care if its contains the test environment for now.
