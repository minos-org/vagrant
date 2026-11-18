About
-----

Recipes to use the official minos images for the [vagrant](https://www.vagrantup.com/) platform:

The boxes from 22.04 on are built by [packers](https://github.com/minos-org/packers) (`minos.pkr.hcl`)
and published as assets of its releases, each `.box` beside its `.sha256`.
Vagrant finds a box and its versions through the box's catalog, which this repository serves on
GitHub Pages at `https://minos.io/vagrant/<box>.json`.

Usage
-----

    $ cd core-24.04
    $ vagrant up && vagrant ssh

A box alone, through its catalog:

    $ vagrant box add https://minos.io/vagrant/core-24.04.json
    $ vagrant init minos/core-24.04 && vagrant up

The same machine without Vagrant: unpack the box into its own directory and import its OVF (or open
`box.ovf` with *File → Import Appliance* in VirtualBox):

    $ mkdir core-24.04 && tar xf minos-core-24.04.box -C core-24.04
    $ VBoxManage import core-24.04/box.ovf

Catalog
-------

One JSON file per box, in Vagrant's box metadata format
(https://developer.hashicorp.com/vagrant/docs/boxes/format): the box's name, and one entry per
version with one provider per box file. Each release of packers is a version, newest first; packers
keeps only its newest release, so a catalog lists one.

    {
      "name": "minos/core-24.04",
      "description": "minos core on Ubuntu 24.04 LTS",
      "versions": [{
        "version": "2026.11.17.1630",
        "providers": [{
          "name": "virtualbox",
          "url": "https://github.com/minos-org/packers/releases/download/2026.11.17/minos-core-24.04.box",
          "checksum_type": "sha256",
          "checksum": "<sha256 of the .box: the digest GitHub computed for it, or its line in minos-core-24.04.sha256>",
          "architecture": "amd64",
          "default_architecture": true
        }]
      }]
    }

- `version` is when the release was published, in UTC (`YYYY.MM.DD.HHMM`). The release itself, and the
  box's `url`, are named by the day alone (`YYYY.MM.DD`), and a rebuild the same day takes the day's
  release over: were the version the tag, the rebuilt box would carry the version of the one it replaces,
  and `vagrant box outdated` would never offer it. Vagrant compares versions as RubyGems does, segment by
  segment as numbers, so `2026.11.17.1630` comes after `2026.11.17.0915` and after the `2026.11.17.4` of
  the releases named `YYYY.MM.DD.N` before (where `2026.11.17-1` would be a prerelease that comes before
  it).
- `architecture` and `default_architecture` are read by Vagrant 2.4 and ignored by older ones.
- Vagrant tells a catalog from a box by the `Content-Type` of the answer (`application/json`). A release
  asset arrives as `application/octet-stream` and a raw file of the repository as `text/plain`, so
  Vagrant would take either for a box: the catalogs are served by GitHub Pages, which answers `.json`
  with `application/json`. The boxes themselves can stay release assets.

`catalog/` holds the catalogs; GitHub Pages serves it as the root of `minos.io/vagrant/`. They are
written, never edited: every hour `.github/workflows/catalog.yml` rewrites them from packers' releases
(`.github/catalog`, which reads its public API), commits the ones that changed, and deploys `catalog/`
whenever `minos.io/vagrant/` serves something else. A box with no release yet has a catalog with no
versions.

Dependencies
------------

- [Vagrant](https://www.vagrantup.com/)
- [Virtualbox](https://www.virtualbox.org)
