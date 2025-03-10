# Archives

The binaries built will be archived together with the `README` and `LICENSE` files into a
`tar.gz` file. In the `archives` section you can customize the archive name,
additional files, and format.

Here is a commented `archives` section with all fields specified:

```yaml
# .goreleaser.yaml
archives:
  -
    # ID of this archive.
    # Defaults to `default`.
    id: my-archive

    # Builds reference which build instances should be archived in this archive.
    # Default is empty, which includes all builds.
    builds:
    - default

    # Archive format. Valid options are `tar.gz`, `tar.xz`, `tar`, `gz`, `zip` and `binary`.
    # If format is `binary`, no archives are created and the binaries are instead
    # uploaded directly.
    # Default is `tar.gz`.
    format: zip

    # This will create an archive without any binaries, only the files are there.
    # The name template must not contain any references to `Os`, `Arch` and etc, since the archive will be meta.
    # Default is false.
    meta: true

    # Archive name template.
    # Defaults:
    # - if format is `tar.gz`, `tar.xz`, `gz` or `zip`:
    #   - `{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}{{ with .Arm }}v{{ . }}{{ end }}{{ with .Mips }}_{{ . }}{{ end }}{{ if not (eq .Amd64 "v1") }}{{ .Amd64 }}{{ end }}`
    # - if format is `binary`:
    #   - `{{ .Binary }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}{{ with .Arm }}v{{ . }}{{ end }}{{ with .Mips }}_{{ . }}{{ end }}{{ if not (eq .Amd64 "v1") }}{{ .Amd64 }}{{ end }}`
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"

    # Replacements for GOOS and GOARCH in the archive name.
    # Keys should be valid GOOSs or GOARCHs.
    # Values are the respective replacements.
    # Default is empty.
    replacements:
      amd64: 64-bit
      386: 32-bit
      darwin: macOS
      linux: Tux

    # Set this to true if you want all files in the archive to be in a single directory.
    # If set to true and you extract the archive 'goreleaser_Linux_arm64.tar.gz',
    # you'll get a folder 'goreleaser_Linux_arm64'.
    # If set to false, all files are extracted separately.
    # You can also set it to a custom folder name (templating is supported).
    # Default is false.
    wrap_in_directory: true

    # If set to true, will strip the parent directories away from binary files.
    #
    # This might be useful if you have your binary be built with a subdir for some reason, but do no want that subdir inside the archive.
    #
    # Default is false.
    strip_parent_binary_folder: true

    # Can be used to change the archive formats for specific GOOSs.
    # Most common use case is to archive as zip on Windows.
    # Default is empty.
    format_overrides:
      - goos: windows
        format: zip

    # Additional files/template/globs you want to add to the archive.
    # Defaults are any files matching `LICENSE*`, `README*`, `CHANGELOG*`,
    #  `license*`, `readme*` and `changelog*`.
    files:
      - LICENSE.txt
      - README_{{.Os}}.md
      - CHANGELOG.md
      - docs/*
      - design/*.png
      - templates/**/*
      # a more complete example, check the globbing deep dive below
      - src: '*.md'
        dst: docs
        # Strip parent folders when adding files to the archive.
        # Default: false
        strip_parent: true
        # File info.
        # Not all fields are supported by all formats available formats.
        # Defaults to the file info of the actual file if not provided.
        info:
          owner: root
          group: root
          mode: 0644
          # format is `time.RFC3339Nano`
          mtime: 2008-01-02T15:04:05Z

    # Before and after hooks for each archive.
    # Skipped if archive format is binary.
    # This feature is available in [GoReleaser Pro](/pro) only.
    hooks:
      before:
      - make clean # simple string
      - cmd: go generate ./... # specify cmd
      - cmd: go mod tidy
        output: true # always prints command output
        dir: ./submodule # specify command working directory
      - cmd: touch {{ .Env.FILE_TO_TOUCH }}
        env:
        - 'FILE_TO_TOUCH=something-{{ .ProjectName }}' # specify hook level environment variables

      after:
      - make clean
      - cmd: cat *.yaml
        dir: ./submodule
      - cmd: touch {{ .Env.RELEASE_DONE }}
        env:
        - 'RELEASE_DONE=something-{{ .ProjectName }}' # specify hook level environment variables

    # Disables the binary count check.
    # Default: false
    allow_different_binary_count: true
```

!!! success "GoReleaser Pro"
    Archive hooks is a [GoReleaser Pro feature](/pro/).

!!! tip
    Learn more about the [name template engine](/customization/templates/).

!!! tip
    You can add entire folders, its subfolders and files by using the glob notation,
    for example: `myfolder/**/*`.

!!! warning
    The `files` and `wrap_in_directory` options are ignored if `format` is `binary`.

!!! warning
    The `name_template` option will not reflect the filenames under the `dist` folder if `format` is `binary`.
    The template will be applied only where the binaries are uploaded (e.g. GitHub releases).

## Deep diving into the globbing options

We'll walk through what happens in each case using some examples.

```yaml
# ...
files:

# Adds `README.md` at the root of the archive:
- README.md

# Adds all `md` files to the root of the archive:
- '*.md'

# Adds all `md` files to the root of the archive:
- src: '*.md'

# Adds all `md` files in the current folder to a `docs` folder in the archive:
- src: '*.md'
  dst: docs

# Recursively adds all `go` files to a `source` folder in the archive.
# in this case, `cmd/myapp/main.go` will be added as `source/cmd/myapp/main.go`
- src: '**/*.go'
  dst: source

# Recursively adds all `go` files to a `source` folder in the archive, stripping their parent folder.
# In this case, `cmd/myapp/main.go` will be added as `source/main.go`:
- src: '**/*.go'
  dst: source
  strip_parent: true
# ...
```

## Packaging only the binaries

Since GoReleaser will always add the `README` and `LICENSE` files to the
archive if the file list is empty, you'll need to provide a filled `files`
on the archive section.

A working hack is to use something like this:

```yaml
# .goreleaser.yaml
archives:
- files:
  - none*
```

This would add all files matching the glob `none*`, provide that you don't
have any files matching that glob, only the binary will be added to the
archive.

For more information, check [#602](https://github.com/goreleaser/goreleaser/issues/602)

## A note about Gzip

Gzip is a compression-only format, therefore, it couldn't have more than one
file inside.

Presumably, you'll want that file to be the binary, so, your archive section
will probably look like this:

```yaml
# .goreleaser.yaml
archives:
- format: gz
  files:
  - none*
```

This should create `.gz` files with the binaries only, which should be
extracted with something like `gzip -d file.gz`.

!!! warning
    You won't be able to package multiple builds in a single archive either.
    The alternative is to declare multiple archives filtering by build ID.

## Disable archiving

You can do that by setting `format` to `binary`:

```yaml
# .goreleaser.yaml
archives:
- format: binary
```

Make sure to check the rest of the documentation above, as doing this has some
implications.
