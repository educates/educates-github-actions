Publish Multiple Workshops
==========================

This GitHub action supports the following for a collection of Educates workshops:

* For each workshop:
  * Creating an OCI image artefact containing workshop content files and pushing
    it to the GitHub container registry.
* Creating a release against the GitHub repository and attach as assets
  Kubernetes resource files for deploying the collection of workshop to Educates.

Note that this GitHub action can publish multilpe workshops and will publish, 
for each workshop, an OCI image artefact containing the workshop files. 

The GitHub action requires that it be triggered in response to a Git tag being
applied to the GitHub repository.

GitHub Workflow
---------------

The name of the GitHub action is:

```
educates/educates-github-actions/publish-multiple-workshops
```

To have a collection of Educates workshops published upon the repository being tagged as
version `X.Y` use:

```
name: Publish Collection of Workshops

on:
  push:
    tags:
      - "[0-9]+.[0-9]+"

jobs:
  publish-workshop:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Create release
        uses: educates/educates-github-actions/publish-multiple-workshops@v7
        with:
          token: ${{secrets.GITHUB_TOKEN}}
```

Note that version `v7` and later of this GitHub action produces an exported workshop
definition which requires Educates 2.6.0 or later.

Workshop Definition
-------------------

This GitHub action makes use of the `educates publish-workshop` command. As
such, the workshop definition must include a `spec.publish` section which
defines where the image is to be published:

```
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: {name}
spec:
  publish:
    image: $(image_repository)/{name}-files:$(workshop_version)
```

The workshop definition may optionally specify what files should be included in
the OCI image artefact for the workshop.

```
apiVersion: training.educates.dev/v1beta1
kind: Workshop
metadata:
  name: {name}
spec:
  publish:
    image: $(image_repository)/{name}-files:$(workshop_version)
    files:
    - directory:
        path: .
      includePaths:
      - /workshop/**
      - /templates/**
      - /README.md
```

See the Educates documentation for more information.

Collection of workshops directory structure
-------------------------------------------
This action assumes that a collection of workshops will have the `recommended` folder structure,
and every workshop in the collection `must` have the same workshop structure. 

This is an example of the recommended structure:

```
resources
├── trainingportal.yaml
workshops
├── lab-conda-environment
│   ├── README.md
│   ├── resources
│   │   └── workshop.yaml
│   └── workshop
│       └── content
├── lab-cookie-consent
│   ├── README.md
│   ├── resources
│   │   └── workshop.yaml
│   └── workshop
│       ├── content
│       └── setup.d
├── lab-docker-runtime
│   ├── README.md
│   ├── resources
│   │   └── workshop.yaml
│   └── workshop
│       └── content
└── lab-examiner-scripts
    ├── README.md
    ├── resources
    │   └── workshop.yaml
    └── workshop
        ├── content
        └── examiner
```

Action Configuration
--------------------

Configuration parameters which can be set in the `with` clause for the this
GitHub action are as follows:

| Name                            | Required | Type     | Description                        |
|---------------------------------|----------|----------|------------------------------------|
| `path`                          | False    | String   | Relative directory path under `$GITHUB_WORKSPACE` to the collection of workshops. Defaults to "`workshops`". |
| `token`                         | True     | String   | GitHub access token. Must be set to `${{secrets.GITHUB_TOKEN}}` or appropriate personal access token variable reference. |
| `trainingportal-resource-file`  | False    | String   | Relative path under `$GITHUB_WORKSPACE` to the `TrainingPortal` resource file. Defaults to "`resources/trainingportal.yaml`". |
| `workshop-resource-file`        | False    | String   | Relative path under workshop directory to the `Workshop` resource file. Defaults to "`resources/workshop.yaml`". Every workshop must have same directory structure. |
