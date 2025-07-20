# Changelog

## [0.8.2] - 2025-07-20

### Documentation

- Update README.md with latest doctest example

## [0.8.1] - 2025-07-20

### Documentation

- Re-enable doctest running
- Add notice regarding independence from the `reqwest` project

### Miscellaneous Tasks

- Update to edition 2021

### Ci

- Update release.sh to incorporate `cargo-semver-checks`

### Refact

- Update serialization calls to use `cookie_store::serde`

## [0.8.0] - 2024-05-31

### Miscellaneous Tasks

- Update `cookie_store = "^0.21"`

## [0.7.0] - 2024-03-23

### Miscellaneous Tasks

- Update reqwest to v0.12.0

### Ci

- Remove `--topo-order` argument to `git-cliff`

## [0.6.0] - 2023-06-17

### Documentation

- Add CHANGELOG.md
- Update doctest to use `CookieStore::new()` in example

### Ci

- Update release.sh for git-cliff arg changes
- Change `reqwest` dependency specification to `^0.11` instead of specific version
- Setup for `git cliff` usage

### Refact

- Update to `cookie_store 0.20`

