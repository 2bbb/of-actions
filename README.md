# of-actions

Reusable GitHub Actions workflows for openFrameworks CI. Builds addons and apps across macOS, Linux, and Windows with Debug/Release matrix support.

## Usage

### Addon

```yaml
jobs:
  build:
    uses: 2bbb/of-actions/.github/workflows/build-addon.yml@v2
    with:
      of_version: "0.12.1"
      addon_name: "ofxYourAddon"
      test_app: "testApp"
```

### With Debug + Release

```yaml
with:
  configs: '["Debug", "Release"]'
```

### With test assertions (ofxUnitTestsApp)

```yaml
with:
  test_mode: "test"
```

### App

```yaml
jobs:
  build:
    uses: 2bbb/of-actions/.github/workflows/build-app.yml@v2
    with:
      of_version: "0.12.1"
      app_name: "yourGreatApp"
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
| `test_mode` | | `"run"` | `build-only` / `run` / `test` (see below) |
| `configs` | | `'["Release"]'` | JSON array of build configs |

### build-app.yml

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `of_version` | ✅ | — | oF version |
| `app_name` | ✅ | — | Directory name under `apps/myApps/` |
| `submodules` | | `true` | Checkout submodules recursively |
| `preprocessor_defines` | | `""` | Windows preprocessor defines |
| `cache_key_suffix` | | `"v1"` | Cache key suffix |
| `test_mode` | | `"run"` | `build-only` / `run` / `test` |
| `configs` | | `'["Release"]'` | JSON array of build configs |

## test_mode

| Mode | Behavior |
|------|----------|
| `build-only` | Compile only, don't execute |
| `run` | Compile and execute, ignore exit code (`continue-on-error`) |
| `test` | Compile and execute, **fail CI on non-zero exit code** |

Use `test` when your test app exits with a meaningful code (e.g. ofxUnitTestsApp).

## How It Works

oF release asset naming is inconsistent across versions. The workflows resolve download URLs dynamically via `gh release view` — no hardcoded URLs.

Debug/Release is handled via `make {Config}` / `make Run{Config}` on macOS/Linux and `/p:configuration={Config}` on Windows.

## Writing Test Apps for CI

### ofAppNoWindow

Always use `ofAppNoWindow` for headless execution:

```cpp
#include "ofMain.h"
#include "ofAppNoWindow.h"
#include "ofApp.h"

int main() {
    ofInit();
    auto window = std::make_shared<ofAppNoWindow>();
    auto app = std::make_shared<ofApp>();
    ofRunApp(window, app);
    return ofRunMainLoop();
}
```

`ofAppNoWindow` does not loop — the app runs `setup()` and exits.

### ofxUnitTestsApp

For structured test assertions with proper exit codes, inherit from `ofxUnitTestsApp`:

```cpp
#include "ofMain.h"
#include "ofAppNoWindow.h"
#include "ofxUnitTests.h"
#include "ofxYourAddon.h"

class ofApp : public ofxUnitTestsApp {
    void run() override {
        ofxTest(your_function() == expected, "test description");
        ofxTestEq(actual, expected, "equality test");
    }
};

int main() {
    ofInit();
    auto window = std::make_shared<ofAppNoWindow>();
    auto app = std::make_shared<ofApp>();
    ofRunApp(window, app);
    return ofRunMainLoop();
}
```

Add `ofxUnitTests` to your `addons.make`:

```
ofxYourAddon
ofxUnitTests
```

The app exits with the number of failed tests (0 = all pass). Use `test_mode: "test"` in your CI to fail on test failures.

### Key points

- **`ofExit(code)`** terminates the app. Normal oF apps run an event loop and never exit — your test app must call `ofExit()` explicitly.
- **`ofAppNoWindow`** avoids needing a display server. On Linux, `xvfb-run` is used automatically.
- **`make RunDebug` / `make RunRelease`** are the oF make targets that build (if needed) then execute. The workflow uses these on macOS/Linux.

## Requirements

- Addon repos must have `addons.make` in the test app directory
- Addon repos must have `addon_config.mk` in the addon root
- App repos must have a `Makefile` with `OF_ROOT` and `addons.make`

## License

MIT
