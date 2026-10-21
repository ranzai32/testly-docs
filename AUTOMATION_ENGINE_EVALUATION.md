# Automation Engine Evaluation

## Decision

Testly should use **Playwright as the primary recording and replay engine**. Recording should be implemented as a Testly-owned layer that produces the internal `.flow.yaml` model, while Playwright provides browser lifecycle, locator execution, waiting, isolation, screenshots, tracing, and report inputs. Chrome DevTools Protocol should be used only behind a narrow adapter for Chromium-specific diagnostics that Playwright does not expose adequately. Selenium should not be used in the MVP engine.

## Evaluation criteria

The engine must support two different workloads:

- Recording user intent as stable, readable actions rather than raw browser events
- Replaying those actions reliably in local, headed, headless, and CI environments

Scores use a five-point scale where 5 is the strongest fit for Testly.

| Criterion | Weight | Playwright | Selenium | CDP directly |
|---|---:|---:|---:|---:|
| Recording foundation | 20% | 5 | 3 | 3 |
| Replay reliability | 25% | 5 | 3 | 2 |
| Diagnostics and observability | 15% | 5 | 4 | 5 |
| Cross-browser path | 15% | 4 | 5 | 1 |
| Implementation effort | 15% | 5 | 3 | 1 |
| Maintenance risk | 10% | 4 | 4 | 1 |
| Weighted result | 100% | **4.75** | **3.55** | **2.20** |

## Recording comparison

| Capability | Playwright | Selenium | CDP directly |
|---|---|---|---|
| Existing recorder | Built-in code generator records interactions and generates resilient locators | Selenium IDE records and replays browser actions | No product-level recorder; all event capture and interpretation must be built |
| Locator generation | Prioritizes role, text, test ID, and other user-facing locators | Records Selenium commands with targets based on element context | Exposes DOM primitives but provides no stable locator policy |
| Browser session control | First-class contexts, pages, popups, frames, and isolated profiles | Mature WebDriver sessions; recorder integration is a separate concern | Requires target discovery, session routing, lifecycle management, and reconnection logic |
| Network and console capture | High-level events with Chromium-specific escape hatches | WebDriver BiDi is the cross-browser direction, but feature coverage is still evolving | Deepest Chromium visibility through Network, Runtime, Log, and related domains |
| Fit for `.flow.yaml` | Recorder output can be mapped into Testly domain steps | Selenium IDE commands require a translation layer | Entire semantic model and locator generator must be created from scratch |

Playwright offers the strongest starting point, but Testly should not persist generated Playwright source code as its primary model. The recorder should normalize browser activity into domain actions such as `navigate`, `click`, `fill`, `select`, and `assert`, then serialize those actions to `.flow.yaml`.

## Replay comparison

| Capability | Playwright | Selenium | CDP directly |
|---|---|---|---|
| Waiting behavior | Automatically checks visibility, stability, event reception, enabled state, and editability before relevant actions | Explicit and implicit waits are available, but the engine must define and apply a consistent waiting policy | All readiness and retry behavior must be implemented by Testly |
| Browser coverage | Chromium, Firefox, WebKit, Chrome, and Edge | Broadest support through browser-specific WebDriver implementations, including Safari | Chromium and other Blink-based targets only |
| CI operation | Integrated headless execution, isolation, parallelization, screenshots, traces, and reports | Mature Grid ecosystem and broad remote execution support | Requires custom process, protocol, artifact, timeout, and failure management |
| Failure diagnostics | Screenshots, video, trace data, console, requests, responses, and page events | Screenshots and logs are available; richer event access depends increasingly on WebDriver BiDi | Maximum raw detail, but Testly must correlate every event with each test step |
| Test stability | Locator API and actionability checks reduce timing-related failures | Reliable when waits and locator rules are designed carefully | Low-level commands provide no test-oriented stability guarantees |

Playwright's automatic actionability checks are especially important for a no-code product because users cannot be expected to repair timing logic manually. The engine checks whether a target is actionable before interacting with it and retries supported assertions until their conditions are met.

## Option analysis

### Playwright

**Strengths**

- Provides a maintained test generator that proves the recording and locator-generation model.
- Combines browser control, auto-waiting, assertions, isolation, tracing, screenshots, and network events in one API.
- Runs on Chromium, Firefox, and WebKit, with Chrome and Edge channels also supported.
- Uses the same Node.js and TypeScript ecosystem planned for Playwright export.
- Minimizes the amount of browser-specific infrastructure Testly must own.

**Limitations**

- Playwright browser versions are coupled to Playwright releases and require managed browser installation.
- Its Firefox and WebKit support use Playwright-managed builds rather than arbitrary installed Firefox or Safari binaries.
- The built-in code generator is a user tool, not a stable domain API for Testly's file format.
- Some deep Chromium diagnostics may still require a CDP session.

### Selenium

**Strengths**

- Uses the W3C WebDriver standard and supports a broad set of browsers and language bindings.
- Has a mature remote execution ecosystem through Selenium Grid.
- Selenium IDE demonstrates standalone recording and playback.
- WebDriver BiDi provides a standards-based path for bidirectional network, logging, and script events.

**Limitations**

- Recording and runtime automation are split between Selenium IDE and WebDriver, increasing integration work.
- Testly would need to build a consistent waiting, tracing, locator, and artifact layer around WebDriver.
- The advantages of multiple language bindings do not materially help a desktop product with one internal engine.
- WebDriver BiDi is the correct long-term direction, but Selenium documentation describes the transition from classic WebDriver and CDP as ongoing.

### CDP directly

**Strengths**

- Provides the deepest access to Chromium DOM, input, network, runtime, performance, and debugging events.
- Avoids abstraction limitations when implementing specialized Chrome diagnostics.
- Makes it possible to attach to targets through a direct WebSocket connection.

**Limitations**

- CDP is a debugging protocol rather than a test automation framework.
- The tip-of-tree protocol changes frequently and does not guarantee backward compatibility.
- Testly would own browser discovery, protocol negotiation, target sessions, input synthesis, locator semantics, waits, retries, isolation, downloads, dialogs, frames, and artifacts.
- Direct CDP creates a hard Chromium dependency and conflicts with the planned cross-browser path.

## Proposed architecture

| Component | Responsibility | Technology choice |
|---|---|---|
| Recorder | Capture user intent and convert it into domain actions | Testly-owned recorder on Playwright pages and browser contexts |
| Locator builder | Generate and rank role, label, text, test ID, and CSS fallback locators | Testly-owned policy using Playwright locator semantics |
| Flow serializer | Validate and persist domain actions | Testly-owned `.flow.yaml` domain layer |
| Replay engine | Execute domain actions and assertions | Playwright |
| Diagnostics collector | Associate screenshots, console output, and network activity with steps | Playwright events and tracing |
| Chromium diagnostics adapter | Access missing low-level Chromium events when justified | Optional CDP adapter isolated behind an interface |
| Exporter | Convert domain actions into editable test source | Playwright TypeScript exporter |

The `.flow.yaml` schema must remain independent of Playwright API names. This preserves the option to add another replay adapter later without changing stored tests.

## MVP implementation boundaries

The MVP should:

- Record and replay Chrome-compatible Chromium sessions through Playwright.
- Use a separate automation profile rather than the user's default Chrome profile.
- Support navigation, click, fill, select, check, keyboard, and explicit assertion steps.
- Capture a screenshot, console messages, and relevant network activity for each failed step.
- Keep credentials outside flow files and mask matching values in artifacts.
- Pin the Playwright version and its browser binaries in development and CI.

The MVP should not:

- Build a general-purpose CDP client.
- Depend on Selenium IDE project files or commands.
- Expose raw Playwright locators as the only locator representation in `.flow.yaml`.
- Promise Firefox or WebKit recording before Chrome recording is stable.
- Mix Playwright and Selenium within a single replay run.

## Re-evaluation triggers

Reconsider the decision if Testly requires any of the following:

- Running against browsers or devices that Playwright cannot control
- Integrating with an existing Selenium Grid mandated by target customers
- Executing tests in languages other than the internal Node.js runtime
- Supporting standards-only browser automation as a contractual requirement
- Using a Chromium feature unavailable through Playwright and too substantial for a narrow CDP adapter

## Sources

- [Playwright test generator](https://playwright.dev/docs/codegen)
- [Playwright auto-waiting and actionability](https://playwright.dev/docs/actionability)
- [Playwright browser support](https://playwright.dev/docs/browsers)
- [Selenium WebDriver](https://www.selenium.dev/documentation/webdriver/)
- [Selenium WebDriver BiDi](https://www.selenium.dev/documentation/webdriver/bidi/)
- [Selenium IDE](https://www.selenium.dev/documentation/ide/)
- [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)
- [CDP DOM domain](https://chromedevtools.github.io/devtools-protocol/tot/DOM/)
- [CDP Input domain](https://chromedevtools.github.io/devtools-protocol/tot/Input/)
- [CDP Network domain](https://chromedevtools.github.io/devtools-protocol/tot/Network/)
