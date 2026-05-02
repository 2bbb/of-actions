# of-actions

Reusable GitHub Actions workflows for openFrameworks CI. Builds addons and apps across macOS, Linux, and Windows.

## Usage

### Addon

```yaml
jobs:
  build:
    uses: 2bbb/of-actions/.github/workflows/build-addon.yml@v1
    with:
      of_version: "0.12.1"
      addon_name: "ofxNozzle"
      test_app: "testApp"
```

### App

```yaml
jobs:
  build:
    uses: 2bbb/of-actions/.github/workflows/build-app.yml@v1
    with:
      of_version: "0.12.1"
      app_name: "myApp"
```

### Nightly

```yaml
with:
  of_version: "nightly"
```

## Inputs

### build-addon.yml

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `of_version` | ✅ | — | oF version (`"0.12.0"`, `"0.12.1"`, `"nightly"`) |
| `addon_name` | ✅ | — | Directory name under `addons/` |
| `test_app` | ✅ | — | Test app directory under `addons/{addon_name}/` |
| `submodules` | | `true` | Checkout submodules recursively |
| `preprocessor_defines` | | `""` | Windows preprocessor defines (semicolon-separated) |
| `cache_key_suffix` | | `"v1"` | Cache key suffix for busting |
| `run_test_app` | | `true` | Run the test app after building |

### build-app.yml

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `of_version` | ✅ | — | oF version (`"0.12.0"`, `"0.12.1"`, `"nightly"`) |
| `app_name` | ✅ | — | Directory name under `apps/myApps/` |
| `submodules` | | `true` | Checkout submodules recursively |
| `preprocessor_defines` | | `""` | Windows preprocessor defines (semicolon-separated) |
| `cache_key_suffix` | | `"v1"` | Cache key suffix for busting |
| `run_app` | | `true` | Run the app after building |

## How It Works

oF release asset naming is inconsistent across versions (macOS switched from .zip to .tar.gz in 0.12.1, Linux changed `linux64gcc6` to `linux64_gcc6`, Windows changed `vs2017` to `vs`). The workflows resolve download URLs dynamically via `gh release view` — no hardcoded URLs.

## Requirements

- Addon repos must have `addons.make` in the test app directory
- Addon repos must have `addon_config.mk` in the addon root
- App repos must have a `Makefile` with `OF_ROOT` and `addons.make`

## License

MIT
