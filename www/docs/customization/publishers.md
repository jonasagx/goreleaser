# Custom Publishers

GoReleaser supports publishing artifacts by executing a custom publisher.

## How it works

You can declare multiple `publishers` instances. Each publisher will be
executed for each (filtered) artifact. For example, there will be a total of
6 executions for 2 publishers with 3 artifacts.

Publishers run sequentially in the order they're defined
and executions are parallelized between all artifacts.
In other words the publisher is expected to be safe to run
in multiple instances in parallel.

If you have only one `publishers` instance, the configuration is as easy as adding
the command to your `.goreleaser.yaml` file:

```yaml
publishers:
  - name: my-publisher
    cmd: custom-publisher -version={{ .Version }} {{ abs .ArtifactPath }}
```

### Environment

Commands, which are executed as custom publishers only inherit a subset of
the system environment variables (unlike existing hooks) as a precaution to
avoid leaking sensitive data accidentally and provide better control of the
environment for each individual process where variable names may overlap
unintentionally.

Environment variables that are kept:

- `HOME`
- `USER`
- `USERPROFILE`
- `TMPDIR`
- `TMP`
- `TEMP`
- `PATH`


You can however use `.Env.NAME` templating syntax, which enables
more explicit inheritance.

```yaml
- cmd: custom-publisher
  env:
    - SECRET_TOKEN={{ .Env.SECRET_TOKEN }}
```

The publisher explicit environment variables take precedence over the
inherited set of variables as well.

### Variables

Command (`cmd`), workdir (`dir`) and environment variables (`env`) support templating

```yaml
publishers:
  - name: production
    cmd: |
      custom-publisher \
      -product={{ .ProjectName }} \
      -version={{ .Version }} \
      {{ .ArtifactName }}
    dir: "{{ dir .ArtifactPath }}"
    env:
      - TOKEN={{ .Env.CUSTOM_PUBLISHER_TOKEN }}
```

so the above example will execute `custom-publisher -product=goreleaser -version=1.0.0 goreleaser_1.0.0_linux_amd64.zip` in `/path/to/dist` with `TOKEN=token`, assuming that GoReleaser is executed with `CUSTOM_PUBLISHER_TOKEN=token`.

Supported variables:

- `Version`
- `Tag`
- `ProjectName`
- `ArtifactName`
- `ArtifactPath`
- `Os`
- `Arch`
- `Arm`

## Customization

Of course, you can customize a lot of things:

```yaml
# .goreleaser.yaml
publishers:
  -
    # Unique name of your publisher. Used for identification
    name: "custom"

    # IDs of the artifacts you want to publish
    ids:
     - foo
     - bar

    # Publish checksums (defaults to false)
    checksum: true

    # Publish signatures (defaults to false)
    signature: true

    # Working directory in which to execute the command
    dir: "/utils"

    # Command to be executed
    cmd: custom-publisher -product={{ .ProjectName }} -version={{ .Version }} {{ .ArtifactPath }}

    # Environment variables
    env:
      - API_TOKEN=secret-token

    # You can publish extra pre-existing files.
    # The filename published will be the last part of the path (base).
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

These settings should allow you to push your artifacts to any number of endpoints,
which may require non-trivial authentication or has otherwise complex requirements.

!!! tip
    Learn more about the [name template engine](/customization/templates/).
