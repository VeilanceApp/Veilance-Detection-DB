# Veilance Detection Database

The **Veilance Detection Database** is an open-source collection of browser behavior detections used by [Veilance](https://veilance.org/) to identify privacy-invasive, fingerprinting, tracking, telemetry, permission, storage, and browser API activity performed by websites.

The database provides human-readable detection rules that translate low-level browser activity into understandable findings.

Instead of simply showing that a website called a specific JavaScript API, Veilance can use these detections to explain **what the behavior may mean**.

For example, a website reading screen characteristics once may be normal. A website repeatedly querying screen properties, font availability, WebGL information, client hints, and network characteristics during the same visit may indicate browser fingerprinting.

The detection database provides the rules used to make that distinction.

---

## What This Repository Is

This repository contains individual JSON detection rules used by Veilance.

Each file describes:

* What behavior should be detected
* Which Veilance indicator or indicators should trigger it
* How frequently the behavior must occur
* The category of the behavior
* The severity of the finding
* A human-readable explanation of what happened

Each detection exists as its own JSON file so detections can be individually reviewed, updated, added, removed, or distributed.

Example:

```json
{
  "id": "repeated-font-probing",
  "name": "Repeated font probing",
  "category": "Fingerprinting",
  "description": "A page used font-probing signals at least ten times.",
  "severity": "medium",
  "match": {
    "indicatorId": "font-probing",
    "minCount": 10
  }
}
```

This rule tells Veilance that if the `font-probing` indicator occurs at least ten times during an observation, the activity should be surfaced as **Repeated font probing**.

---

## Why This Exists

Modern websites interact with a large number of browser APIs.

Many of those APIs have legitimate purposes, but they can also be used for:

* Browser fingerprinting
* Cross-session identification
* Advertising telemetry
* Behavioral analytics
* Device profiling
* Permission enumeration
* Storage-based tracking
* Network profiling
* Hardware capability discovery
* Automated-browser detection
* Persistent telemetry

The problem is that most users have no practical way to see this activity.

Browser developer tools may expose individual API calls or network requests, but interpreting hundreds or thousands of events requires significant technical knowledge.

Veilance collects browser-visible behavior and converts it into structured indicators.

The Detection Database sits on top of those indicators and answers a more useful question:

> **What is the website actually doing?**

---

## How It Works

Veilance observes supported browser APIs and website activity locally inside the browser.

Those observations are converted into normalized **indicators**.

An indicator might represent activity such as:

```text
font-probing
canvas-readback
webgl
battery
network-information
cookie-access
browser-storage
beacon
permission-query
```

Detection rules then evaluate those indicators.

A basic detection may require one indicator:

```json
{
  "match": {
    "indicatorId": "battery"
  }
}
```

A frequency-based detection can require an indicator to appear multiple times:

```json
{
  "match": {
    "indicatorId": "font-probing",
    "minCount": 10
  }
}
```

More advanced detections can correlate several different signals.

For example:

```json
{
  "match": {
    "mode": "all",
    "signals": [
      {
        "indicatorId": "navigator-characteristics",
        "minCount": 2
      },
      {
        "indicatorId": "network-information"
      },
      {
        "indicatorId": "battery"
      }
    ]
  }
}
```

This allows Veilance to detect **behavioral patterns**, rather than treating every browser API call independently.

---

## Indicators vs. Detections

There is an important distinction between an **indicator** and a **detection**.

### Indicator

An indicator represents something Veilance directly observed.

Examples:

```text
Website accessed Battery Status API
Website queried navigator.webdriver
Website created a WebRTC data channel
Website read canvas pixel data
```

Indicators are the raw behavioral building blocks.

### Detection

A detection interprets one or more indicators and assigns meaning to the observed behavior.

Examples:

```text
Repeated font probing
Automation flag probing
Device fingerprint assembled
Persistent telemetry pattern
Permission enumeration
```

In simplified form:

```text
Browser activity
      ↓
Veilance instrumentation
      ↓
Indicators
      ↓
Detection rules
      ↓
Human-readable findings
```

---

## Detection Format

A typical detection contains the following fields:

```json
{
  "id": "automation-flag-probing",
  "name": "Automation flag probing",
  "category": "Fingerprinting",
  "description": "A page read navigator.webdriver, which can be used to distinguish automated browser sessions from normal browsing.",
  "severity": "medium",
  "match": {
    "indicatorId": "navigator-characteristics",
    "api": "Navigator",
    "action": "read-webdriver"
  }
}
```

### `id`

A unique machine-readable identifier.

```json
"id": "automation-flag-probing"
```

IDs should generally:

* Use lowercase characters
* Use hyphens between words
* Clearly describe the detection
* Remain stable once published

### `name`

A short human-readable detection name.

```json
"name": "Automation flag probing"
```

### `category`

The general behavior class.

Examples may include:

```text
Fingerprinting
Tracking
Storage
Permissions
Telemetry
Network
Privacy
Device Access
```

### `description`

Explains what was observed and why it may be relevant.

Descriptions should remain factual and should not automatically describe behavior as malicious.

A browser API can have both legitimate and privacy-relevant uses.

### `severity`

Represents the significance of the behavior.

Typical values:

```text
low
medium
high
critical
```

Severity should reflect the behavior being detected, not automatically imply malicious intent.

### `match`

Defines the conditions required for the rule to trigger.

A match may evaluate:

* Indicator ID
* API
* Action
* Minimum occurrence count
* Multiple correlated signals
* Other supported indicator attributes

---

## Detection Philosophy

The database is designed around a few principles.

### Behavior Over Reputation

A detection should primarily describe what a website **did**, not whether the website has a good or bad reputation.

For example:

```text
Repeated canvas readback
```

is preferable to:

```text
Malicious fingerprinting website
```

unless there is enough evidence to support the stronger conclusion.

### Observable Evidence

Detections should be based on activity that Veilance can actually observe.

Rules should not attempt to infer unsupported behavior.

### Explainability

A user should be able to understand why a detection fired.

Whenever possible, a finding should map back to specific browser activity.

### Low False-Positive Bias

Many browser APIs have legitimate uses.

Detections should therefore distinguish between ordinary API usage and behavior that becomes interesting because of:

* Frequency
* Combination
* Sequence
* Scope
* Persistence

### Open Review

Detection logic should be inspectable.

Users should be able to see exactly why Veilance classified a behavior a certain way.

---

## Example: Fingerprinting Detection

A website may collect several characteristics of a browser:

```text
Screen dimensions
Timezone
Language
Installed fonts
Canvas output
WebGL renderer
Network characteristics
Hardware concurrency
Device memory
```

Individually, many of these signals are normal.

When combined, however, they can form a relatively unique browser fingerprint.

A Veilance detection can identify that combination:

```json
{
  "id": "multi-signal-device-fingerprinting",
  "name": "Multi-signal device fingerprinting",
  "category": "Fingerprinting",
  "description": "A page combined multiple device and browser characteristics that may contribute to a browser fingerprint.",
  "severity": "medium",
  "match": {
    "mode": "all",
    "signals": [
      {
        "indicatorId": "navigator-characteristics"
      },
      {
        "indicatorId": "screen-characteristics"
      },
      {
        "indicatorId": "canvas-readback"
      }
    ]
  }
}
```

This approach lets Veilance detect patterns without assuming that every individual browser API call is suspicious.

---

## Repository Structure

Each detection should be stored as an individual JSON file.

Example:

```text
detections/
├── automation-flag-probing.json
├── battery-status-access.json
├── drm-key-system-probe.json
├── high-entropy-client-hints.json
├── permission-enumeration.json
├── repeated-font-probing.json
├── repeated-network-information-probing.json
└── persistent-telemetry-pattern.json
```

Keeping detections separated makes it easier to:

* Review changes
* Track detection history
* Remove problematic rules
* Test individual rules
* Accept community contributions
* Distribute database updates

---

## What This Database Is Not

The Veilance Detection Database is **not a malware blacklist**.

A detection firing does not automatically mean:

```text
The website is malicious.
```

It means:

```text
Veilance observed behavior matching this detection.
```

Many browser capabilities used for fingerprinting or tracking also have legitimate applications.

The database is intended to provide **visibility and context**, allowing users and researchers to make their own assessment.

---

## Open Source

The Veilance Detection Database is open source.

The detection logic used to classify browser behavior can therefore be:

* Inspected
* Audited
* Forked
* Improved
* Tested
* Discussed
* Contributed to

There should not be a hidden classification system deciding what websites are doing without allowing users to inspect the rules.

If Veilance reports that a behavior occurred, the rule responsible for that finding should be available for review.

---

## Community Contributions

Contributions are welcome.

Useful contributions include:

* New detection rules
* Improved descriptions
* False-positive reductions
* Better severity classifications
* New fingerprinting techniques
* New browser APIs
* Tracking behavior research
* Privacy Sandbox observations
* Detection test cases

Before submitting a new detection, confirm that the required indicator is actually produced by Veilance.

A detection cannot identify behavior that the extension does not currently instrument.

If a new browser behavior requires additional instrumentation, changes may also be required in the main Veilance repository.

---

## Adding a Detection

Create a new `.json` file containing one detection.

Example:

```json
{
  "id": "example-detection",
  "name": "Example detection",
  "category": "Fingerprinting",
  "description": "Description of the browser behavior being detected.",
  "severity": "medium",
  "match": {
    "indicatorId": "example-indicator"
  }
}
```

The filename should normally match the detection ID:

```text
example-detection.json
```

Before submitting the rule:

1. Confirm the indicator exists.
2. Confirm the rule parses correctly.
3. Test the rule against expected activity.
4. Consider legitimate uses of the API.
5. Avoid unnecessarily high severity ratings.
6. Keep the description factual.
7. Avoid duplicating an existing detection.

---

## Research Use

The database can also be useful outside of the Veilance browser UI.

Researchers can use the rules to study:

* Fingerprinting techniques
* Website telemetry behavior
* Browser API adoption
* Tracking patterns
* Changes in web privacy practices
* Emerging browser identification techniques

Because the rules are structured JSON, they can also be processed programmatically for analysis, testing, or dataset generation.

---

## Relationship to Veilance

The Detection Database is one component of the broader Veilance project.

Veilance is designed to make browser behavior visible by observing what websites request and interact with during a visit.

The basic architecture is:

```text
Website
   ↓
Browser APIs
   ↓
Veilance instrumentation
   ↓
Sanitized indicators
   ↓
Detection Database
   ↓
Findings
   ↓
User
```

The extension performs the observation.

The detection database provides the interpretation.

Separating the two allows detection logic to improve independently from the extension itself.

---

## Privacy

The Detection Database itself contains detection definitions only.

It does not contain:

* Browsing histories
* User identities
* Cookies
* Authentication credentials
* Browser profiles
* Personal telemetry

The rules simply define how supported Veilance indicators should be interpreted.

How Veilance collects, processes, stores, or uploads telemetry is defined separately by the main Veilance project and its privacy policy.

---

## Goals

The long-term goal of the Veilance Detection Database is to build an open, community-reviewable knowledge base of observable browser behaviors.

As websites and browsers evolve, new mechanisms for identification, telemetry, tracking, storage, and device discovery will continue to appear.

This repository provides a place where those behaviors can be documented as practical, machine-readable detections.

The goal is simple:

> **Make it easier to understand what websites are asking your browser to do.**

---

## License

This repository is open source.

See the repository's `LICENSE` file for the exact license terms.

---

## Related Projects

* **Veilance** — Browser privacy and behavior visibility extension
* **Veilance Detection Database** — Open detection rules used to interpret observed browser activity

Project website:

https://veilance.org/

