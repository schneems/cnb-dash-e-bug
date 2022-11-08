# CNB dash-e bug

Args with `-e` are somehow eaten from default process types. In this case, we create a simple bash script that echos the args that are passed to it:

```bash
#!/usr/bin/env bash

echo "Printing args"
printf '%s\n' "$*"
```

Here's the default process with args:

```
$ pack inspect sample | grep Processes -A 5
Processes:
  TYPE                        SHELL        COMMAND                                              ARGS                                 WORK DIR
  print-args (default)        bash         /layers/sample-counter/sys-info/print-args.sh        before-dash-e -e after-dash-e        /workspace
```

When we run this, you would expect to see `-e` in the output, but it is not present:

## Expected

```
$ docker run -it --rm sample
Printing args
before-dash-e -e after-dash-e
```

## Actual

```
$ docker run -it --rm sample
Printing args
before-dash-e after-dash-e
```

Note that the `-e` is missing

## Reproduce

Clone the code:

```
git clone https://github.com/schneems/cnb-dash-e-bug
cd cnb-dash-e-bug
```

Build the app:

```
$ pack build sample --buildpack ./ --path ./
22: Pulling from heroku/builder
Digest: sha256:e7916849c5cddcad0877666e672ddd37bcfdaf5064dba6967032b780291e66f5
Status: Image is up to date for heroku/builder:22
22-cnb: Pulling from heroku/heroku
Digest: sha256:48752e45ae12ea577f519b734a9578c7d0dab08359edc2c252253bb67a1c6564
Status: Image is up to date for heroku/heroku:22-cnb
Restoring data for SBOM from previous image
===> DETECTING
sample-counter 0.1.0
===> RESTORING
Restoring metadata for "sample-counter:sys-info" from app image
===> BUILDING
---> Hello processes buildpack
---> Adding sys-info process
---> Done
===> EXPORTING
Reusing layer 'sample-counter:sys-info'
Reusing layer 'launch.sbom'
Adding 1/1 app layer(s)
Reusing layer 'launcher'
Reusing layer 'config'
Reusing layer 'process-types'
Adding label 'io.buildpacks.lifecycle.metadata'
Adding label 'io.buildpacks.build.metadata'
Adding label 'io.buildpacks.project.metadata'
Setting default process type 'print-args'
Saving sample...
*** Images (6992ee87a261):
      sample
Successfully built image sample
```

Execute the default process:

```
$ docker run -it --rm sample
Printing args
before-dash-e  after-dash-e
```
