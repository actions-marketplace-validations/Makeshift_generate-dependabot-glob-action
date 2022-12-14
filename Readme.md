# Generate Dependabot Glob Action

This action creates a `dependabot.yml` file from a user-provided template by replacing instances of directory globs with an array of objects matching that glob, with all the other keys copied.

For example, the following template:

```yaml
  - package-ecosystem: 'docker'
    directory: '/test/docker/*/Dockerfile*'
    schedule:
      interval: 'daily'
```

Will result in:

```yaml
  - package-ecosystem: 'docker'
    directory: '/test/docker/container_1/'
    schedule:
      interval: 'daily'
  - package-ecosystem: 'docker'
    directory: '/test/docker/container_2/'
    schedule:
      interval: 'daily'
  - package-ecosystem: 'docker'
    directory: '/test/docker/weird_dockerfile/'
    schedule:
      interval: 'daily'
```

Note that the basename of any matching directory is used as the value.

This action uses the [glob](https://www.npmjs.com/package/glob) node module. Refer to its documentation for more information on the glob syntax.

The default configuration for `glob` is as follows:

```js
const globOpts = {
  root: process.cwd(),
  mark: true,
  matchBase: true,
  nomount: true,
  follow: core.getInput('follow-symbolic-links') === 'true' || true
}
```

If these options are not sufficient, please open an issue and let me know.

## Quickstart

### Create a `.github/dependabot.template.yml` file

This is just a normal `dependabot.yml` file, but with globs/wildcards in the `directory` field.
Note that comments will not be transferred to the generated file.

  ```yaml
version: 2

updates:
  - package-ecosystem: 'github-actions'
    # No globs
    directory: '/'
    schedule:
      interval: 'daily'

  - package-ecosystem: 'docker'
    # Simple globs
    directory: '/test/docker/*/Dockerfile*'
    schedule:
      interval: 'weekly'

  - package-ecosystem: 'npm'
    # Simple glob + extglob
    directory: '/test/npm/*/{package-lock.json,yarn.lock}'
    ignore:
      - dependency-name: '*'
    schedule:
      interval: 'daily'

  - package-ecosystem: 'terraform'
    # Searches the entire tree, but only matches files with the given name
    # This actually outputs without a leading slash, but dependabot doesn't seem to care
    # Note the . is escaped, node-glob doesn't search hidden files by default
    directory: '\.terraform.lock.hcl'
    commit-message:
      prefix: 'terraform'
    schedule:
      interval: 'weekly'

  ```

### Create a `.github/workflows/generate_dependabot.yml` file

The action does not create a PR or otherwise commit the generated file, so we can use another action like peter-evans/create-pull-request to do that.

```yaml
name: Generate dependabot.yml

on:
  push:
  repository_dispatch:
  workflow_dispatch:

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      
      - uses: actions/checkout@v3
        
      - name: Generate dependabot.yml
        uses: Makeshift/generate-dependabot-glob-action@v1

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v4
```

Done. Now, whenever you push to the repository, or manually trigger the workflow, a PR will be created with the generated `dependabot.yml` file matching your wildcards if they've changed.

<!-- action-docs-inputs -->
## Inputs

| parameter | description | required | default |
| --- | --- | --- | --- |
| template-file | Location of the file to use as template | `false` | .github/dependabot.template.yml |
| follow-symbolic-links | Indicates whether to follow symbolic links | `false` | true |
<!-- action-docs-inputs -->

<!-- action-docs-outputs -->

<!-- action-docs-outputs -->
