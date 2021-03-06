# Examples

### **Downloading minimal python library packages from a snapshot of `debian jessie`**.

`dpkg_src` and `dpkg_list` are repository rules, and therefore made to be used in the `WORKSPACE`.

First, set up the package source with `dpkg_src` rule.  This example uses a snapshot of debian jessie from July 1st 2017.  The rule outputs a `file:Packages.json` which contains a parsed and formatted `Packages.gz` for `dpkg_list` to consume.

```python
dpkg_src(
    name = "debian_jessie",
    arch = "amd64",
    distro = "jessie",
    sha256 = "8ff5e7a54d4e75bbbcd2f43ebc7cb4a082fbc5493bc9fb2dcdaaeacba6e76dee",
    snapshot = "20170701T034145Z",
    url = "http://snapshot.debian.org/archive",
)
```

You can also set up the package source using the full url for the `Packages.gz` file. The `package_prefix` is used to
prepend to the value of `Filename` in the `Packages.gz` file. In the following example, if the value of `Filename` is
`pool/jdk1.8/b/bazel/bazel_0.7.0_amd64.deb`, then the `.deb` artifact will later be downloaded from
`http://storage.googleapis.com/bazel-apt/pool/jdk1.8/b/bazel/bazel_0.7.0_amd64.deb`.

```python
dpkg_src(
    name = "bazel_apt",
    packages_gz_url = "http://storage.googleapis.com/bazel-apt/dists/stable/jdk1.8/binary-amd64/Packages.gz",
    package_prefix = "http://storage.googleapis.com/bazel-apt/",
    sha256 = "0fc4c6988ebf24705cfab0050cb5ad58e5b2aeb0e8cfb8921898a1809042416c",
)
```

You can now reference this `dpkg_src` rule when downloading packages in the `dpkg_list` rule.  The `dpkg_src` rule will output a `packages` map in `file:packages.bzl` for you to access the `.deb` artifacts. 

```python
dpkg_list(
    name = "package_bundle",
    packages = [
        "libpython2.7-minimal",
        "python2.7-minimal",
        "libpython2.7-stdlib",
        "zlib1g",
    ],
    sources = [
        "@debian_jessie//file:Packages.json",
    ],
)
```

Finally, in a `BUILD` file, you can access the `.deb` files for rules that might require them.  We reference the package map from the previous `dpkg_list` rule and access the packages.  

```python
load("@package_bundle//file:packages.bzl", "packages")

docker_build(
    name = "python27",
    base = "//base:base",
    debs = [
        packages["zlib1g"],
        packages["python2.7-minimal"],
        packages["libpython2.7-minimal"],
        packages["libpython2.7-stdlib"],
    ],
    entrypoint = [
        "/usr/bin/python2.7",
    ],
    symlinks = {
        "/usr/bin/python": "/usr/bin/python2.7",
    },
)
```

# Reference

## dpkg_src

```python
dpkg_src(name, url, arch, distro, snapshot, packages_gz_url, package_prefix, sha256, dpkg_parser)
```

A rule that downloads a `Packages.gz` snapshot file and parses it into a readable format for `dpkg_list`.
It supports snapshots from [http://snapshot.debian.org/](http://snapshot.debian.org/). (You can find out more about the format and sources available there.)
It also supports retrieving `Packages.gz` file from a given full url.

Either a set of {`url`, `arch`, `distro`, `snapshot`} or a set of {`packages_gz_url`, `package_prefix`} must be set.

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <p><code>name, required</code></p>
      </td>
    </tr>
    <tr>
      <td><code>url</code></td>
      <td>
        <p><code>the base url of the package repository</code></p>
        <p>The url that hosts snapshots of Packages.gz files.</p>
      </td>
    </tr>
    <tr>
      <td><code>arch</code></td>
      <td>
        <p><code>the target package architecture</code></p>
      </td>
    </tr>
    <tr>
      <td><code>distro</code></td>
      <td>
        <p><code>the name of the package distribution</code></p>
        <p>Examples: wheezy, jessie, jessie-backports, etc.</p>
      </td>
    </tr>
    <tr>
      <td><code>snapshot</code></td>
      <td>
        <p><code>the snapshot date of the Packages.gz</code></p>
        <p>Format: YYYYMMDDTHHMMSSZ.  You can query a list of possible dates for snapshot.debian.org at <a href=
        'http://snapshot.debian.org/archive/debian/?year=2009;month=10'>http://snapshot.debian.org/archive/debian/?year=2009;month=10</a>
      </td>
    </tr>
    <tr>
      <td><code>packages_gz_url</code></td>
      <td>
        <p><code>the full url for the Packages.gz file</code></p>
      </td>
    </tr>
    <tr>
      <td><code>package_prefix</code></td>
      <td>
        <p><code>the prefix to prepend to the value of Filename in the Packages.gz file</code></p>
      </td>
    </tr>
    <tr>
      <td><code>sha256</code></td>
      <td>
        <p><code>the sha256 of the Packages.gz file, required</code></p>
      </td>
    </tr>
    <tr>
      <td><code>dpkg_parser</code></td>
      <td>
        <p><code>A binary that translates a Packages.gz file into a format readable by dpkg_list, required</code></p>
      </td>
    </tr>
  </tbody>
</table>

## dpkg_list

```python
dpkg_list(name, packages, sources)
```

A rule that downloads `.deb` packages and makes them available in the WORKSPACE.

For a `dpkg_list` rule named `package_bundle`, packages can be used by loading `load("@package_bundle//file:packages.bzl", "packages")` into your `BUILD` file, then referencing the package with `packages['packagename']`

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <p><code>name, required</code></p>
      </td>
    </tr>
    <tr>
      <td><code>packages</code></td>
      <td>
        <p><code>a string array of packages to download, required</code></p>
        <p>The url that hosts snapshots of Packages.gz files.</p>
      </td>
    </tr>
    <tr>
      <td><code>sources</code></td>
      <td>
        <p><code>a list of outputs from dpkg_src, required</code></p>
        <p>A list of snapshot sources that will be checked when downloading the package.  If a package is present in multiple sources, the first source in the list will be chosen.</p>
      </td>
    </tr>
  </tbody>
</table>
