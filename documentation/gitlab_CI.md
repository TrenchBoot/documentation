# GitlabCI configuration documentation

## Preparation

### GitHub

* Create account for CI bot
  - grant `Admin` access to this bot in the repositories you will want to use
    GitLabCI with

### GitLab

* Create account for CI bot

## Configuration

### GitLabCI for GitHub integration

The process is descriebd in the
[official documentation](https://docs.gitlab.com/ee/ci/ci_cd_for_external_repos/github_integration.html).

Following projects were integrated:

* `TrenchBoot` GitHub repositories:
  - [grub](https://github.com/trenchboot/grub)
  - [landing-zone](https://github.com/trenchboot/landing-zone)
  - [linux](https://github.com/trenchboot/linux)

* `3mdeb` GitHub repositories:
  - [grub](https://github.com/3mdeb/grub)
  - [landing-zone](https://github.com/3mdeb/landing-zone)
  - [linux](https://github.com/3mdeb/linux)
  - [meta-trenchboot](https://github.com/3mdeb/meta-trenchboot)
  - [nixos-trenchboot-configs](https://github.com/3mdeb/nixos-trenchboot-configs)

### CI/CD variables

Following variables are used on the
[trenchboot1 group level](https://gitlab.com/groups/trenchboot1/-/settings/ci_cd)
(they are used by both the repositories in `TrenchBoot` and `3mdeb` subgroups):
- `CACHIX_AUTHTOKEN` - authotken to push nix artifacts to the
  [3mdeb cachix](https://app.cachix.org/cache/3mdeb)
- `CACHIX_SIGNING_KEY` - signing key to sign the artifacts pushed to the
  [3mdeb cachix](https://app.cachix.org/cache/3mdeb)
- `RTE_IP_1000V` - local IP of the [RTE](https://3mdeb.com/products/open-source-hardware/rte/)
  connected to the `Asrock R1000V` board
- `RTE_IP_APU2` - local IP of the [RTE](https://3mdeb.com/products/open-source-hardware/rte/)
  connected to the `APU2` board
- `RTE_IP_SUPERMICRO` - local IP of the [RTE](https://3mdeb.com/products/open-source-hardware/rte/)
  connected to the `Supermicro m11sdv-8ct-ln4f` board
- `YOCTO_CACHE_PATH` - local path where the Yocto cache is stored in the local
  `HTTP` server
- `YOCTO_HTTP_CACHE_IP` - local IP address of the HTTP cache server
- `YOCTO_HTTP_CACHE_RSYNC_KEY`- private SSH key used to populate the Yocto
  downloads/sstate cache on the local server using `rsync`
- `YOCTO_HTTP_CACHE_USER` - username to the local HTTP cache server

Following variables are used on the
[trenchboot1/3mdeb subgroup level](https://gitlab.com/trenchboot1):
- `GITHUB_GROUP` - gitlab group prefix for `3mdeb` repositories
- `GITHUB_GROUP` - github group prefix for `3mdeb` repositories

Following variables are used on the
[trenchboot1/trenchboot subgroup level](https://gitlab.com/trenchboot1):
- `GITHUB_GROUP` - gitlab group prefix for `TrenchBoot` repositories
- `GITHUB_GROUP` - github group prefix for `TrenchBoot` repositories

Thanks to the group variables, we can have the same `.gitlab-ci.yml` content
for both groups, and adjust some variables (like prefixes) there.

### Setting up local runners

Follow the [gitlab-runner](gitlab_runner.md) documentation.
