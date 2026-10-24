# easyquery/.github

Shared GitHub settings for the [EasyQuery](https://korzh.com/easyquery) repositories.

## Reusable workflows

| Workflow | Used by |
|---|---|
| [`build-samples.yml`](.github/workflows/build-samples.yml) | every `eq-samples-*` repository: builds its root solution on Windows with MSBuild, then every client app (`.esproj` folders and `SpaRoot` folders) with `npm run build` |
| [`test-build-samples.yml`](.github/workflows/test-build-samples.yml) | this repository only: builds the published samples repositories with the `build-samples.yml` of the branch it runs from |

A samples repository calls it from its own `.github/workflows/build.yml`, **pinned to a
version tag**:

```yaml
jobs:
  build:
    uses: easyquery/.github/.github/workflows/build-samples.yml@v1
    with:
      solution: Samples.AspNetCore.sln
```

The check it reports is `build / build`. Branch protection on `main` and `beta` of the samples
repositories requires it, so renaming either job breaks their pull requests.

## Changing the build workflow

The samples repositories use `@v1`, never `@main`. A change on `main` here therefore reaches
none of them until the `v1` tag moves, and the tag only moves after the change has built every
published samples repository.

1. **Change `build-samples.yml` on a branch**, e.g. `ci/<what>`, and push the branch.
2. **Test it against all samples repositories:**

   ```bash
   gh workflow run test-build-samples.yml -R easyquery/.github --ref ci/<what>
   gh run watch -R easyquery/.github $(gh run list -R easyquery/.github --workflow test-build-samples.yml --limit 1 --json databaseId -q '.[0].databaseId')
   ```

   It runs the branch's `build-samples.yml` on the default branch of every repository in its
   matrix. All of them have to pass.
3. **Merge the branch into `main`.**
4. **Release it:**
   - **Compatible change** (same inputs, same job names): move the tag.

     ```bash
     git tag -f -a v1 -m "build-samples.yml v1: <what changed>" main
     git push -f origin v1
     ```

     Every samples repository uses it from its next build on.
   - **Breaking change** (an input removed or renamed, a job renamed, which would change the
     `build / build` check name): tag `v2` instead. Then move each samples repository to `@v2`
     through its own `dev → beta → main` pull requests. `v1` stays where it is for the
     repositories that haven't moved yet.

**When a samples repository is published**, add it to the matrix in `test-build-samples.yml`, so
later changes are tested against it too.
