# Linux packages (via nFPM)

GoReleaser can be wired to [nfpm](https://github.com/goreleaser/nfpm) to
generate and publish `.deb`, `.rpm` and `.apk` packages.

Available options:

```yaml
# .goreleaser.yaml
nfpms:
  # note that this is an array of nfpm configs
  -
    # ID of the nfpm config, must be unique.
    # Defaults to "default".
    id: foo

    # Name of the package.
    # Defaults to `ProjectName`.
    package_name: foo

    # You can change the file name of the package.
    #
    # Default:`{{ .PackageName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}{{ with .Arm }}v{{ . }}{{ end }}{{ with .Mips }}_{{ . }}{{ end }}{{ if not (eq .Amd64 "v1") }}{{ .Amd64 }}{{ end }}`
    file_name_template: "{{ .ConventionalFileName }}"

    # Build IDs for the builds you want to create NFPM packages for.
    # Defaults empty, which means no filtering.
    builds:
      - foo
      - bar

    # Replacements for GOOS and GOARCH in the package name.
    # Keys should be valid GOOSs or GOARCHs.
    # Values are the respective replacements.
    # Default is empty.
    replacements:
      amd64: 64-bit
      386: 32-bit
      darwin: macOS
      linux: Tux

    # Your app's vendor.
    # Default is empty.
    vendor: Drum Roll Inc.

    # Template to your app's homepage.
    # Default is empty.
    homepage: https://example.com/

    # Your app's maintainer (probably you).
    # Default is empty.
    maintainer: Drummer <drum-roll@example.com>

    # Template to your app's description.
    # Default is empty.
    description: |-
      Drum rolls installer package.
      Software to create fast and easy drum rolls.

    # Your app's license.
    # Default is empty.
    license: Apache 2.0

    # Formats to be generated.
    formats:
      - apk
      - deb
      - rpm
      - termux.deb

    # Packages your package depends on. (overridable)
    dependencies:
      - git
      - zsh

    # Packages it provides. (overridable)
    provides:
      - bar

    # Packages your package recommends installing. (overridable)
    recommends:
      - bzr
      - gtk

    # Packages your package suggests installing. (overridable)
    suggests:
      - cvs
      - ksh

    # Packages that conflict with your package. (overridable)
    conflicts:
      - svn
      - bash

    # Packages it replaces. (overridable)
    replaces:
      - fish

    # Template to the path that the binaries should be installed.
    # Defaults to `/usr/bin`.
    bindir: /usr/bin

    # Version Epoch.
    # Default is extracted from `version` if it is semver compatible.
    epoch: 2

    # Version Prerelease.
    # Default is extracted from `version` if it is semver compatible.
    prerelease: beta1

    # Version Metadata (previously deb.metadata).
    # Default is extracted from `version` if it is semver compatible.
    # Setting metadata might interfere with version comparisons depending on the packager.
    version_metadata: git

    # Version Release.
    release: 1

    # Section.
    section: default

    # Priority.
    priority: extra

    # Makes a meta package - an empty package that contains only supporting files and dependencies.
    # When set to `true`, the `builds` option is ignored.
    # Defaults to false.
    meta: true

    # Changelog YAML file, see: https://github.com/goreleaser/chglog
    #
    # You can use goreleaser/chglog to create the changelog for your project,
    # pass that changelog yaml file to GoReleaser,
    # and it should in turn setup it accordingly for the given available
    # formats (deb and rpm at the moment).
    #
    # Experimental.
    changelog: ./foo.yml

    # Contents to add to the package.
    # GoReleaser will automatically add the binaries.
    contents:
      # Basic file that applies to all packagers
      - src: path/to/local/foo
        dst: /usr/local/bin/foo

      # Simple config file
      - src: path/to/local/foo.conf
        dst: /etc/foo.conf
        type: config

      # Simple symlink.
      # Corresponds to `ln -s /sbin/foo /usr/local/bin/foo`
      - src: /sbin/foo
        dst: /usr/local/bin/foo
        type: "symlink"

      # Corresponds to `%config(noreplace)` if the packager is rpm, otherwise it is just a config file
      - src: path/to/local/bar.conf
        dst: /etc/bar.conf
        type: "config|noreplace"

      # The src and dst attributes also supports name templates
      - src: path/{{ .Os }}-{{ .Arch }}/bar.conf
        dst: /etc/foo/bar-{{ .ProjectName }}.conf

      # These files are not actually present in the package, but the file names
      # are added to the package header. From the RPM directives documentation:
      #
      # "There are times when a file should be owned by the package but not
      # installed - log files and state files are good examples of cases you might
      # desire this to happen."
      #
      # "The way to achieve this, is to use the %ghost directive. By adding this
      # directive to the line containing a file, RPM will know about the ghosted
      # file, but will not add it to the package."
      #
      # For non rpm packages ghost files are ignored at this time.
      - dst: /etc/casper.conf
        type: ghost
      - dst: /var/log/boo.log
        type: ghost

      # You can use the packager field to add files that are unique to a specific packager
      - src: path/to/rpm/file.conf
        dst: /etc/file.conf
        type: "config|noreplace"
        packager: rpm
      - src: path/to/deb/file.conf
        dst: /etc/file.conf
        type: "config|noreplace"
        packager: deb
      - src: path/to/apk/file.conf
        dst: /etc/file.conf
        type: "config|noreplace"
        packager: apk

      # Sometimes it is important to be able to set the mtime, mode, owner, or group for a file
      # that differs from what is on the local build system at build time.
      - src: path/to/foo
        dst: /usr/local/foo
        file_info:
          mode: 0644
          mtime: 2008-01-02T15:04:05Z
          owner: notRoot
          group: notRoot

      # Using the type 'dir', empty directories can be created. When building RPMs, however, this
      # type has another important purpose: Claiming ownership of that folder. This is important
      # because when upgrading or removing an RPM package, only the directories for which it has
      # claimed ownership are removed. However, you should not claim ownership of a folder that
      # is created by the distro or a dependency of your package.
      # A directory in the build environment can optionally be provided in the 'src' field in
      # order copy mtime and mode from that directory without having to specify it manually.
      - dst: /some/dir
        type: dir
        file_info:
          mode: 0700

    # Scripts to execute during the installation of the package.
    # Keys are the possible targets during the installation process
    # Values are the paths to the scripts which will be executed
    scripts:
      preinstall: "scripts/preinstall.sh"
      postinstall: "scripts/postinstall.sh"
      preremove: "scripts/preremove.sh"
      postremove: "scripts/postremove.sh"

    # Some attributes can be overridden per package format.
    overrides:
      deb:
        conflicts:
          - subversion
        dependencies:
          - git
        suggests:
          - gitk
        recommends:
          - tig
        replaces:
          - bash
        provides:
          - bash
      rpm:
        replacements:
          amd64: x86_64
        file_name_template: "{{ .ProjectName }}-{{ .Version }}-{{ .Arch }}"
        files:
          "tmp/man.gz": "/usr/share/man/man8/app.8.gz"
        config_files:
          "tmp/app_generated.conf": "/etc/app-rpm.conf"
        scripts:
          preinstall: "scripts/preinstall-rpm.sh"

    # Custom configuration applied only to the RPM packager.
    rpm:
      # RPM specific scripts.
      scripts:
        # The pretrans script runs before all RPM package transactions / stages.
        pretrans: ./scripts/pretrans.sh
        # The posttrans script runs after all RPM package transactions / stages.
        posttrans: ./scripts/posttrans.sh

      # The package summary.
      # Defaults to the first line of the description.
      summary: Explicit Summary for Sample Package

      # The package group. This option is deprecated by most distros
      # but required by old distros like CentOS 5 / EL 5 and earlier.
      group: Unspecified

      # Compression algorithm.
      compression: lzma

      # These config files will not be replaced by new versions if they were
      # changed by the user. Corresponds to %config(noreplace).
      config_noreplace_files:
        path/to/local/bar.con: /etc/bar.conf

      # These files are not actually present in the package, but the file names
      # are added to the package header. From the RPM directives documentation:
      #
      # "There are times when a file should be owned by the package but not
      # installed - log files and state files are good examples of cases you might
      # desire this to happen."
      #
      # "The way to achieve this, is to use the %ghost directive. By adding this
      # directive to the line containing a file, RPM will know about the ghosted
      # file, but will not add it to the package."
      ghost_files:
        - /etc/casper.conf
        - /var/log/boo.log

      # The package is signed if a key_file is set
      signature:
        # Template to the PGP secret key file path (can also be ASCII-armored).
        # The passphrase is taken from the environment variable
        # `$NFPM_ID_RPM_PASSPHRASE` with a fallback to `$NFPM_ID_PASSPHRASE`,
        # where ID is the id of the current nfpm config.
        # The id will be transformed to uppercase.
        # E.g. If your nfpm id is 'default' then the rpm-specific passphrase
        # should be set as `$NFPM_DEFAULT_RPM_PASSPHRASE`
        key_file: '{{ .Env.GPG_KEY_PATH }}'

    # Custom configuration applied only to the Deb packager.
    deb:
      # Lintian overrides
      lintian_overrides:
        - statically-linked-binary
        - changelog-file-missing-in-native-package

      # Custom deb special files.
      scripts:
        # Deb rules script.
        rules: foo.sh
        # Deb templates file, when using debconf.
        templates: templates

      # Custom deb triggers
      triggers:
        # register interest on a trigger activated by another package
        # (also available: interest_await, interest_noawait)
        interest:
          - some-trigger-name
        # activate a trigger for another package
        # (also available: activate_await, activate_noawait)
        activate:
          - another-trigger-name

      # Packages which would break if this package would be installed.
      # The installation of this package is blocked if `some-package`
      # is already installed.
      breaks:
        - some-package

      # The package is signed if a key_file is set
      signature:
        # Template to the PGP secret key file path (can also be ASCII-armored).
        # The passphrase is taken from the environment variable
        # `$NFPM_ID_DEB_PASSPHRASE` with a fallback to `$NFPM_ID_PASSPHRASE`,
        # where ID is the id of the current nfpm config.
        # The id will be transformed to uppercase.
        # E.g. If your nfpm id is 'default' then the deb-specific passphrase
        # should be set as `$NFPM_DEFAULT_DEB_PASSPHRASE`
        key_file: '{{ .Env.GPG_KEY_PATH }}'

        # The type describes the signers role, possible values are "origin",
        # "maint" and "archive". If unset, the type defaults to "origin".
        type: origin

    apk:
      # APK specific scripts.
      scripts:
        # The preupgrade script runs before APK upgrade.
        preupgrade: ./scripts/preupgrade.sh
        # The postupgrade script runs after APK.
        postupgrade: ./scripts/postupgrade.sh

      # The package is signed if a key_file is set
      signature:
        # Template to the PGP secret key file path (can also be ASCII-armored).
        # The passphrase is taken from the environment variable
        # `$NFPM_ID_APK_PASSPHRASE` with a fallback to `$NFPM_ID_PASSPHRASE`,
        # where ID is the id of the current nfpm config.
        # The id will be transformed to uppercase.
        # E.g. If your nfpm id is 'default' then the apk-specific passphrase
        # should be set as `$NFPM_DEFAULT_APK_PASSPHRASE`
        key_file: '{{ .Env.GPG_KEY_PATH }}'


        # The name of the signing key. When verifying a package, the signature
        # is matched to the public key store in /etc/apk/keys/<key_name>.rsa.pub.
        # If unset, it defaults to the maintainer email address.
        key_name: origin
```

!!! tip
    Learn more about the [name template engine](/customization/templates/).

!!! info
    Fields marked with "overridable" can be overriden for any format.

## A note about Termux

Termux is the same format as `deb`, the differences are:
- it uses a different `bindir` (prefixed with `/data/data/com.termux/files/`)
- it uses slightly different architecture names than Debian

