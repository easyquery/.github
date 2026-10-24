# easyquery/.github

Shared GitHub settings for the [EasyQuery](https://korzh.com/easyquery) repositories.

## Reusable workflows

| Workflow | Used by |
|---|---|
| [`build-samples.yml`](.github/workflows/build-samples.yml) | every `eq-samples-*` repository: builds its root solution on Windows with MSBuild |

A samples repository calls it from its own `.github/workflows/build.yml`:

```yaml
jobs:
  build:
    uses: easyquery/.github/.github/workflows/build-samples.yml@main
    with:
      solution: Samples.AspNetCore.sln
```

The check it reports is `build / build`. Branch protection on `main` and `beta` of the samples
repositories requires it, so renaming either job breaks their pull requests.
