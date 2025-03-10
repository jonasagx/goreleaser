# Release

GoReleaser can create a GitHub/GitLab/Gitea release with the current tag, upload all
the artifacts and generate the changelog based on the new commits since the
previous tag.

## GitHub

Let's see what can be customized in the `release` section for GitHub:

```yaml
# .goreleaser.yaml
release:
  # Repo in which the release will be created.
  # Default is extracted from the origin remote URL or empty if its private hosted.
  github:
    owner: user
    name: repo

  # IDs of the archives to use.
  # Defaults to all.
  ids:
    - foo
    - bar

  # If set to true, will not auto-publish the release.
  # Available only for GitHub and Gitea.
  # Default is false.
  draft: true

  # Whether to remove existing draft releases with the same name before creating a new one.
  # Only effective if `draft` is set to true.
  # Available only for GitHub.
  # Default is false.
  replace_existing_draft: true

  # Useful if you want to delay the creation of the tag in the remote.
  # You can create the tag locally, but not push it, and run GoReleaser.
  # It'll then set the `target_commitish` portion of the GitHub release to the value of this field.
  # Only works on GitHub.
  # Default is empty.
  target_commitish: '{{ .Commit }}'

  # If set, will create a release discussion in the category specified.
  #
  # Warning: do not use categories in the 'Announcement' format.
  #  Check https://github.com/goreleaser/goreleaser/issues/2304 for more info.
  #
  # Default is empty.
  discussion_category_name: General

  # If set to auto, will mark the release as not ready for production
  # in case there is an indicator for this in the tag e.g. v1.0.0-rc1
  # If set to true, will mark the release as not ready for production.
  # Default is false.
  prerelease: auto

  # What to do with the release notes in case there the release already exists.
  #
  # Valid options are:
  # - `keep-existing`: keep the existing notes
  # - `append`: append the current release notes to the existing notes
  # - `prepend`: prepend the current release notes to the existing notes
  # - `replace`: replace existing notes
  #
  # Default is `keep-existing`.
  mode: append

  # Header template for the release body.
  # Defaults to empty.
  header: |
    ## Some title ({{ .Date }})

    Welcome to this new release!

  # Footer template for the release body.
  # Defaults to empty.
  footer: |
    ## Thanks!

    Those were the changes on {{ .Tag }}!

  # You can change the name of the release.
  # Default is `{{.Tag}}` on OSS and `{{.PrefixedTag}}` on Pro.
  name_template: "{{.ProjectName}}-v{{.Version}} {{.Env.USER}}"

  # You can disable this pipe in order to not create the release on any SCM.
  # Keep in mind that this might also break things that depend on the release URL, for instance, homebrew taps.
  #
  # Defaults to false.
  disable: true

  # Set this to true if you want to disable just the artifact upload to the SCM.
  # If this is true, GoReleaser will still create the release with the changelog, but won't upload anything to it.
  #
  # Defaults to false.
  skip_upload: true

  # You can add extra pre-existing files to the release.
  # The filename on the release will be the last part of the path (base).
  # If another file with the same name exists, the last one found will be used.
  # These globs can also include templates.
  #
  # Defaults to empty.
  extra_files:
    - glob: ./path/to/file.txt
    - glob: ./glob/**/to/**/file/**/*
    - glob: ./glob/foo/to/bar/file/foobar/override_from_previous
    - glob: ./single_file.txt
      name_template: file.txt # note that this only works if glob matches 1 file only
```

!!! tip
    [Learn how to set up an API token, GitHub Enterprise, etc](/scm/github/).

## GitLab

Let's see what can be customized in the `release` section for GitLab.

```yaml
# .goreleaser.yaml
release:
  # Default is extracted from the origin remote URL or empty if its private hosted.
  # You can also use Gitlab's internal project id by setting it in the name
  #  field and leaving the owner field empty.
  gitlab:
    owner: user
    name: repo

  # IDs of the archives to use.
  # Defaults to all.
  ids:
    - foo
    - bar

  # You can change the name of the release.
  # Default is `{{.Tag}}` on OSS and `{{.PrefixedTag}}` on Pro.
  name_template: "{{.ProjectName}}-v{{.Version}} {{.Env.USER}}"

  # You can disable this pipe in order to not upload any artifacts.
  # Defaults to false.
  disable: true

  # What to do with the release notes in case there the release already exists.
  #
  # Valid options are:
  # - `keep-existing`: keep the existing notes
  # - `append`: append the current release notes to the existing notes
  # - `prepend`: prepend the current release notes to the existing notes
  # - `replace`: replace existing notes
  #
  # Default is `keep-existing`.
  mode: append

  # You can add extra pre-existing files to the release.
  # The filename on the release will be the last part of the path (base).
  # If another file with the same name exists, the last one found will be used.
  # These globs can also include templates.
  #
  # Defaults to empty.
  extra_files:
    - glob: ./path/to/file.txt
    - glob: ./glob/**/to/**/file/**/*
    - glob: ./glob/foo/to/bar/file/foobar/override_from_previous
    - glob: ./single_file.txt
      name_template: file.txt # note that this only works if glob matches 1 file only
```

!!! tip
    [Learn how to set up an API token, self-hosted GitLab, etc](/scm/gitlab/).

!!! tip
    If you use GitLab subgroups, you need to specify it in the `owner` field, e.g. `mygroup/mysubgroup`.

!!! warning
    Only GitLab `v12.9+` is supported for releases.

## Gitea

You can also configure the `release` section to upload to a [Gitea](https://gitea.io) instance:

```yaml
# .goreleaser.yaml
release:
  # Default is empty.
  gitea:
    owner: user
    name: repo

  # IDs of the artifacts to use.
  # Defaults to all.
  ids:
    - foo
    - bar

  # You can change the name of the release.
  # Default is `{{.Tag}}` on OSS and `{{.PrefixedTag}}` on Pro.
  name_template: "{{.ProjectName}}-v{{.Version}} {{.Env.USER}}"

  # You can disable this pipe in order to not upload any artifacts.
  # Defaults to false.
  disable: true

  # What to do with the release notes in case there the release already exists.
  #
  # Valid options are:
  # - `keep-existing`: keep the existing notes
  # - `append`: append the current release notes to the existing notes
  # - `prepend`: prepend the current release notes to the existing notes
  # - `replace`: replace existing notes
  #
  # Default is `keep-existing`.
  mode: append

  # You can add extra pre-existing files to the release.
  # The filename on the release will be the last part of the path (base).
  # If another file with the same name exists, the last one found will be used.
  # These globs can also include templates.
  #
  # Defaults to empty.
  extra_files:
    - glob: ./path/to/file.txt
    - glob: ./glob/**/to/**/file/**/*
    - glob: ./glob/foo/to/bar/file/foobar/override_from_previous
    - glob: ./single_file.txt
      name_template: file.txt # note that this only works if glob matches 1 file only
```

To enable uploading `tar.gz` and `checksums.txt` files you need to add the following to your Gitea config in `app.ini`:

```ini
[attachment]
ALLOWED_TYPES = application/gzip|application/x-gzip|application/x-gtar|application/x-tgz|application/x-compressed-tar|text/plain
```

!!! tip
    [Learn how to set up an API token](/scm/gitea/).

!!! tip
    Learn more about the [name template engine](/customization/templates/).

!!! warning
    Gitea versions earlier than 1.9.2 do not support uploading `checksums.txt`
    files because of a [bug](https://github.com/go-gitea/gitea/issues/7882)
    so you will have to enable all file types with `*/*`.

!!! warning
    `draft` and `prerelease` are only supported by GitHub and Gitea.

### Define Previous Tag

GoReleaser uses `git describe` to get the previous tag used for generating the Changelog.
You can set a different build tag using the environment variable `GORELEASER_PREVIOUS_TAG`.
This is useful in scenarios where two tags point to the same commit.

## Custom release notes

You can specify a file containing your custom release notes, and
pass it with the `--release-notes=FILE` flag.
GoReleaser will then skip its own release notes generation,
using the contents of your file instead.
You can use Markdown to format the contents of your file.

On Unix systems you can also generate the release notes in-line by using
[process substitution](https://en.wikipedia.org/wiki/Process_substitution).
To list all commits since the last tag, but skip ones starting with `Merge` or
`docs`, you could run this command:

```sh
goreleaser --release-notes <(some_changelog_generator)
```

Some changelog generators you can use:

- [buchanae/github-release-notes](https://github.com/buchanae/github-release-notes)
- [miniscruff/changie](https://github.com/miniscruff/changie)

!!! info
    If you create the release before running GoReleaser, and the
    said release has some text in its body, GoReleaser will not override it with
    its release notes.
