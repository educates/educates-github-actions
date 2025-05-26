# Publish Workshop

This GitHub action supports the following for an Educates workshop:

* Creating an OCI image artefact containing workshop content files and pushing
  it to the GitHub container registry.
* Creating a release against the GitHub repository and attach as assets
  Kubernetes resource files for deploying the workshop to Educates.

## Support for Multiple Workshops

This GitHub action can publish either a single workshop (default) or multiple workshops in one run. The behavior is controlled by the `multiple-workshops` input parameter.

- **Single Workshop (default):**
  - The action publishes a single workshop, with the location of the workshop definition specified by the `workshop-resource-file` input (default: `resources/workshop.yaml`).
- **Multiple Workshops:**
  - Set `multiple-workshops: true` and specify the `path` input as the directory containing multiple workshop subdirectories (e.g., `workshops`).
  - Each subdirectory under the specified path should contain a workshop definition and its resources and 
  the location of each workshop definition will be specified by the `workshop-resource-file` input (default: `resources/workshop.yaml`) following the `path`.

---

## GitHub Workflow Examples

### Single Workshop (default)

```
name: Publish Workshop

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
        uses: educates/educates-github-actions/publish-workshop@v6
        with:
          token: ${{secrets.GITHUB_TOKEN}}
```

### Multiple Workshops

```
name: Publish Workshops

on:
  push:
    tags:
      - "[0-9]+.[0-9]+"

jobs:
  publish-workshops:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Publish Workshop
        uses: educates/educates-github-actions/publish-workshop@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          multiple-workshops: true
          path: workshops
```

#### Example Folder Structure for Multiple Workshops

```
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

---

## Workshop Definition

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

---

## Action Configuration

Configuration parameters which can be set in the `with` clause for this
GitHub action are as follows:

| Name                            | Required | Type     | Description                        |
|---------------------------------|----------|----------|------------------------------------|
| `path`                          | False    | String   | Relative directory path under `$GITHUB_WORKSPACE` to workshop files or to the directory containing multiple workshops. Defaults to "." for single workshop, set to e.g. `workshops` for multiple workshops. |
| `token`                         | True     | String   | GitHub access token. Must be set to `${{secrets.GITHUB_TOKEN}}` or appropriate personal access token variable reference. |
| `trainingportal-resource-file`  | False    | String   | Relative path under workshop directory to the `TrainingPortal` resource file. Defaults to "resources/trainingportal.yaml". |
| `workshop-resource-file`        | False    | String   | Relative path under workshop directory to the `Workshop` resource file. Defaults to "resources/workshop.yaml". |
| `multiple-workshops`            | False    | Boolean  | Set to `true` to enable publishing multiple workshops from the directory specified by `path`. |
