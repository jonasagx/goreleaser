# Nightlies

!!! success "GoReleaser Pro"
    The nightly build feature is a [GoReleaser Pro feature](/pro/).

Whether if you need beta builds or a rolling-release system, the nightly builds feature will do it for you.

To enable it, you must use the `--nightly` flag in the [`goreleaser release` command](/cmd/goreleaser_release/).

You also have some customization options available:

```yaml
# .goreleaser.yml
nightly:
  # Allows you to change the version of the generated nightly release.
  #
  # Note that some pipes require this to be semantic version compliant (nfpm, for example).
  #
  # Default is `{{ incpatch .Version }}-{{ .ShortCommit }}-nightly`.
  name_template: '{{ incpatch .Version }}-devel'

  # Tag name to create if publish_release is enabled.
  #
  # Default is `nightly`
  tag_name: devel

  # Whether to publish a release or not.
  # Only works on GitHub.
  #
  # Default is `false`.
  publish_release: true

  # Whether to delete previous pre-releases for the same `tag_name` when releasing.
  # This allows you to keep a single pre-release.
  #
  # Default is `false`.
  keep_single_release: true
```

## How it works

When you run GoReleaser with `--nightly`, it will set the `Version` template variable to the evaluation of `nightly.name_template`.
This means that if you use `{{ .Version }}` on your name templates, you'll get the nightly version.

!!! tip
    Learn more about the [name template engine](/customization/templates/).

## What is skipped when using `--nightly`?

- Go mod proxying;
- GitHub/GitLab/Gitea releases (unless specified);
- Homebrew taps;
- Scoop manifests;
- Arch User Repositories;
- Krew Plugin Manifests;
- Milestone closing;
- All announcers;

Everything else is executed normally.
Just make sure to use the `Version` template variable instead of `Tag`.
You can also check if it is a nightly build inside a template with:

```
{{ if .IsNightly }}something{{ else }}something else{{ end }}
```

!!! info "Maybe you are looking for something else?"
    - If just want to build the binaries, and no packages at all, check the [`goreleaser build` command](/cmd/goreleaser_build/);
    - If you actually want to create a local "snapshot" build, check out the [snapshots documentation](/customization/snapshots/).
