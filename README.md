# trellis-facetwp-cache

[![GitHub tag](https://img.shields.io/github/tag/ItinerisLtd/trellis-facetwp-cache.svg)](https://github.com/ItinerisLtd/trellis-facetwp-cache/tags)
[![license](https://img.shields.io/github/license/ItinerisLtd/trellis-facetwp-cache.svg)](https://github.com/ItinerisLtd/trellis-facetwp-cache/blob/main/LICENSE)

Persist the [FacetWP Cache](https://facetwp.com/help-center/add-on-features-and-extras/caching/) `db.php` drop-in across Trellis deploys.

## The problem

The add-on needs a `db.php` drop-in in `WP_CONTENT_DIR`, so it can answer cached requests before WordPress loads.
It only creates that file when the plugin is activated, as a symlink into one specific release.
Every Trellis deploy builds a fresh release without it, and the old symlink dangles once that release is pruned.
Caching therefore stops after each deploy.

## What this role does

Before the share step of each deploy, if the new release contains the add-on:

1. Copies the add-on's `db.php` to `shared/facetwp-cache-db.php`, replacing it every deploy so plugin updates carry over.
2. Appends a `db.php` entry to `project_shared_children`, so Trellis symlinks the shared file into the release as `web/app/db.php`.

Sites without the add-on are left alone.

## Requirements

- Trellis with the `deploy_share_before` hook (`roles/deploy/tasks/share.yml`)
- Bedrock
- The FacetWP Cache add-on installed at `web/app/plugins/facetwp-cache`

The add-on's `db.php` finds `cache.php` through `WP_CONTENT_DIR`, not its own location, so the copy works from `shared/`.

Only one `db.php` can exist.
If another plugin provides one (for example Query Monitor in development), this role will replace it in deployed releases, so do not install such plugins on those environments.

## Installation

Add this role to `galaxy.yml`:

```yaml
# galaxy.yml
- src: itinerisltd.trellis_facetwp_cache
  version: x.x.x # Check for latest version!
```

Install it:

```bash
trellis galaxy install
```

## Usage

Add the role to the `deploy_share_before` hook:

```yaml
# group_vars/all/deploy-hooks.yml
# Learn more on https://roots.io/trellis/docs/deployments/#hooks
deploy_share_before:
  - "{{ playbook_dir }}/vendor/roles/itinerisltd.trellis_facetwp_cache/tasks/main.yml"
```

If the hook already exists, add the line to its list.

The role must run in `deploy_share_before`.
Trellis creates a missing shared file empty, so running later would leave an empty `db.php` that caches nothing.

## Limitations

The shared file is replaced before the new release goes live.
The live release briefly uses the new `db.php` with its older plugin code, and a failed deploy or rollback does not restore the previous file.
The file is a short loader, so this is harmless in practice.

If a site sets `project_shared_children` under its own `wordpress_sites` entry, Trellis ignores the global variable and the role cannot append to it.
The role fails with a message in that case.
Add the `db.php` entry to the site's own list instead:

```yaml
- path: "{{ project_public_path }}/app/db.php"
  src: facetwp-cache-db.php
  type: file
  mode: '0644'
```

## Testing

```bash
ansible-playbook -i 'localhost,' -c local tests/test.yml
```

## Author Information

[trellis-facetwp-cache](https://github.com/ItinerisLtd/trellis-facetwp-cache) is an [Itineris Limited](https://www.itineris.co.uk/) project created by [Lee Hanbury-Pickett](https://github.com/codepuncher).

Special thanks to [the Roots team](https://roots.io/about/) whose [Trellis](https://github.com/roots/trellis) makes this project possible.

## Feedback

Please submit an [issue](https://github.com/ItinerisLtd/trellis-facetwp-cache/issues/new) with anything you do or do not like, or fork the project and make suggestions.

## License

[trellis-facetwp-cache](https://github.com/ItinerisLtd/trellis-facetwp-cache) is released under the [MIT License](https://opensource.org/licenses/MIT).
