# Configuration Guide for OpenClaw-CapSolver Integration

This document provides detailed instructions for setting up the CapSolver extension within your OpenClaw environment, along with troubleshooting tips and best practices.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Choosing the Right Browser](#choosing-the-right-browser)
- [Step-by-Step Setup](#step-by-step-setup)
  - [1. Obtain the CapSolver Chrome Extension](#1-obtain-the-capsolver-chrome-extension)
  - [2. Configure Your CapSolver API Key](#2-configure-your-capsolver-api-key)
  - [3. Update OpenClaw Browser Settings](#3-update-openclaw-browser-settings)
  - [4. Restart the OpenClaw Gateway](#4-restart-the-openclaw-gateway)
  - [5. Confirm Successful Loading](#5-confirm-successful-loading)
- [Full Configuration Template](#full-configuration-template)
- [Troubleshooting](#troubleshooting)
  - [Extension Fails to Load](#extension-fails-to-load)
  - [CAPTCHA Verification Fails](#captcha-verification-fails)
  - [Initial Browser Action Timeout](#initial-browser-action-timeout)
  - [Chrome Crashes After Browser Change](#chrome-crashes-after-browser-change)
- [Best Practices](#best-practices)

## Prerequisites

Before proceeding with the integration, ensure you have the following:

1.  **OpenClaw**: Your OpenClaw instance should be installed and its gateway service actively running.
2.  **CapSolver Account**: A registered CapSolver account with an active API key. If you don't have one, you can [register for CapSolver here](https://dev.to/capsolver).
3.  **Compatible Browser**: A version of Chromium or Chrome for Testing. Please note the critical information below regarding browser compatibility.

## Choosing the Right Browser

It is crucial to use a compatible browser version. **Google Chrome 137+ (released mid-2025)** and later versions have silently removed support for the `--load-extension` command-line flag in their standard branded builds. This means that Chrome extensions **cannot be loaded** in automated sessions using these versions.

This restriction also applies to Microsoft Edge. You **must** use one of the following alternatives:

| Browser Choice | Supports Extension Loading | Recommendation |
|----------------|----------------------------|----------------|
| Google Chrome 137+ | No | Not Recommended |
| Microsoft Edge | No | Not Recommended |
| **Chrome for Testing** | **Yes** | **Recommended** |
| **Chromium (Standalone)** | **Yes** | **Recommended** |
| Playwright Bundled Chromium | **Yes** | **Recommended** |

### How to Install Chrome for Testing:

```bash
# Option 1: Via Playwright (recommended for ease of management)
npx playwright install chromium

# The binary will typically be located at a path similar to:
# ~/.cache/ms-playwright/chromium-XXXX/chrome-linux64/chrome  (for Linux systems)
# ~/Library/Caches/ms-playwright/chromium-XXXX/chrome-mac/Chromium.app/Contents/MacOS/Chromium  (for macOS systems)
```

```bash
# Option 2: Direct Download from Google Chrome Labs
# Visit: https://googlechromelabs.github.io/chrome-for-testing/
# Download the appropriate version that matches your operating system.
```

After installation, make sure to note the full path to the browser binary, as you will need it for the OpenClaw configuration.

## Step-by-Step Setup

### 1. Obtain the CapSolver Chrome Extension

Download the CapSolver Chrome extension and extract its contents to a designated directory, for example, `~/.openclaw/capsolver-extension/`:

1.  Navigate to the [CapSolver browser extension releases page on GitHub](https://github.com/capsolver/capsolver-browser-extension/releases).
2.  Download the latest `CapSolver.Browser.Extension-chrome-vX.X.X.zip` file.
3.  Extract the downloaded zip file using the following shell commands:

    ```bash
    mkdir -p ~/.openclaw/capsolver-extension
    unzip CapSolver.Browser.Extension-chrome-v*.zip -d ~/.openclaw/capsolver-extension/
    ```

4.  Verify that the extraction was successful by checking for the `manifest.json` file:

    ```bash
    ls ~/.openclaw/capsolver-extension/manifest.json
    ```
    You should see `manifest.json` listed, confirming the extension is in the correct location.

### 2. Configure Your CapSolver API Key

Open the extension's configuration file, typically located at `~/.openclaw/capsolver-extension/assets/config.js`, and replace the placeholder `apiKey` value with your actual CapSolver API key:

```js
export const defaultConfig = {
  apiKey: 'CAP-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',  // ← Insert your API key here
  useCapsolver: true,
  // ... rest of config
};
```

Your API key can be found on your [CapSolver dashboard](https://dev.to/capsolver).

### 3. Update OpenClaw Browser Settings

Edit your OpenClaw configuration file, `~/.openclaw/openclaw.json`, to enable the browser and specify the path to the CapSolver extension:

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/path/to/chrome-for-testing/chrome",
    "extensions": [
      "~/.openclaw/capsolver-extension"
    ],
    "noSandbox": true,
    "defaultProfile": "openclaw"
  }
}
```

**Important**: You must replace `/path/to/chrome-for-testing/chrome` with the actual, full path to your Chrome for Testing or Chromium binary. For example:

-   **Linux (Playwright installation)**: `~/.cache/ms-playwright/chromium-1200/chrome-linux64/chrome`
-   **macOS (Playwright installation)**: `~/Library/Caches/ms-playwright/chromium-1200/chrome-mac/Chromium.app/Contents/MacOS/Chromium`

> **Note**: Setting `"noSandbox": true` is often required when running OpenClaw in server environments, Docker containers, or CI/CD systems where the Chrome sandbox might not initialize correctly due to kernel capability restrictions. On desktop systems with proper sandboxing, this option can typically be omitted.

### 4. Restart the OpenClaw Gateway

After making changes to the configuration, restart your OpenClaw gateway to apply them:

```bash
# If you are managing OpenClaw with PM2:
pm2 restart opencrawl --update-env

# If you are running OpenClaw directly:
openclaw gateway restart
```

### 5. Confirm Successful Loading

Verify that the CapSolver extension has loaded correctly by checking the gateway logs:

```bash
pm2 logs opencrawl --lines 20 --nostream
```

Look for log entries similar to these, indicating successful extension loading:

```
[browser/chrome] Loading 1 extension(s)
[browser/chrome] Spawning Chrome: /path/to/chrome-for-testing (args: 15)
```

For a more advanced check, you can verify the extension's active service worker via the Chrome DevTools Protocol (CDP):

```bash
curl -s http://127.0.0.1:8091/json/list
```

A successful setup will display a `service_worker` entry with a `chrome-extension://` URL, confirming that CapSolver is loaded and operational:

```json
{
  "title": "Service Worker chrome-extension://cnopfoopenkdblckmekkipihdnambjhf/background.js",
  "type": "service_worker",
  "url": "chrome-extension://cnopfoopenkdblckmekkipihdnambjhf/background.js"
}
```

## Full Configuration Template

Here is a complete example of the `~/.openclaw/openclaw.json` configuration file with the CapSolver extension integrated:

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/path/to/chrome-for-testing/chrome",
    "extensions": [
      "~/.openclaw/capsolver-extension"
    ],
    "noSandbox": true,
    "defaultProfile": "openclaw"
  }
}
```

### Configuration Options Explained

| Option | Description |
|--------|-------------|
| `browser.executablePath` | Specifies the absolute path to your Chrome for Testing or Chromium binary. This is a required setting. |
| `browser.extensions` | An array of directory paths where Chrome extensions are located. Each path should point to an unzipped extension folder. |
| `browser.noSandbox` | A boolean flag. Set to `true` when running OpenClaw on servers or within Docker containers, as the Chrome sandbox often requires specific kernel capabilities not available in these environments. |
| `browser.defaultProfile` | The name of the browser profile to use. The default value is `"openclaw"`. |

Note that the CapSolver API key is configured directly within the extension's `assets/config.js` file, as detailed in Step 2.

## Troubleshooting

This section addresses common issues you might encounter during setup or operation.

### Extension Fails to Load

**Symptom**: Your gateway logs indicate `Loading 1 extension(s)`, but no `chrome-extension://` targets appear when checking the Chrome DevTools Protocol (CDP).

**Cause**: You are likely using a branded version of Google Chrome 137+ or Microsoft Edge, which silently ignores the `--load-extension` flag.

**Fix**: Switch to Chrome for Testing or a standalone Chromium build. Update the `browser.executablePath` in your OpenClaw configuration to point to the correct binary.

**How to Verify Your Chrome Version**:

```bash
/path/to/your/chrome --version
# Example for Chrome for Testing: "Chromium 143.0.7499.4"
# Example for Branded Chrome: "Google Chrome 143.0.7499.109"
```

### CAPTCHA Verification Fails (Form Submission Issues)

**Possible Causes and Solutions:**
1.  **Insufficient Wait Time**: The AI agent might be submitting the form before CapSolver has completed the CAPTCHA. **Solution**: Increase the wait time in your instructions to at least 60 seconds.
2.  **Invalid API Key**: Your CapSolver API key might be incorrect or expired. **Solution**: Double-check your API key on your [CapSolver dashboard](https://dev.to/capsolver).
3.  **Insufficient Balance**: Your CapSolver account may have run out of credits. **Solution**: Top up your CapSolver account balance.
4.  **Extension Not Loaded**: The CapSolver extension might not be loading correctly. **Solution**: Refer to the "Extension Fails to Load" section above.

### Initial Browser Action Timeout

**Symptom**: The very first browser action after an OpenClaw gateway restart times out, but subsequent actions work without issues.

**Cause**: This can occur due to the combination of a Chromium 
cold start and Playwright CDP connection retries, potentially exceeding the default 20-second tool timeout on the initial attempt.

**Fix**: This is a known behavior. Simply retry the command. The second attempt should succeed as the browser will have fully initialized.

### Chrome Crashes After Switching Browsers

**Symptom**: After changing the `executablePath` from a branded Chrome version to Chrome for Testing or Chromium, the browser crashes with disk cache errors.

**Cause**: The user data directory (browser profile) was created by a different Chrome version and is incompatible with the newly specified browser binary.

**Fix**: Delete the old browser profile directory and restart the OpenClaw gateway:

```bash
rm -rf ~/.openclaw/browser/openclaw/user-data
# Then restart the gateway as described in Step 4
```

## Best Practices

Adhering to these best practices will help ensure a smooth and reliable operation of your OpenClaw-CapSolver integration:

### 1. Always Use Generous Wait Times

While CAPTCHA resolution typically takes 5-20 seconds, factors like network latency, challenge complexity, or internal retries can extend this duration. A **30-60 second** wait time is recommended as a sweet spot to ensure the CapSolver extension has ample time to complete its task before OpenClaw proceeds with form submission. It is always safer to wait a bit longer than to submit prematurely.

### 2. Keep Your Messages Natural

When instructing your AI agent, use natural and conversational language. Avoid explicitly mentioning "CAPTCHA" or "CapSolver" as the AI is not designed to understand these terms in the context of extension control. For example:

-   **Instead of**: `"Navigate to URL, wait for captcha solver, then submit"`
-   **Use**: `"Go to URL, wait about a minute, then submit the form"`

Natural phrasing works more effectively with the AI and helps prevent potential refusals or misunderstandings.

### 3. Monitor Your CapSolver Balance

Each CAPTCHA solve consumes credits from your CapSolver account. Regularly check your balance on the [CapSolver dashboard](https://dev.to/capsolver) to avoid service interruptions due to insufficient funds.

### 4. Use `noSandbox` on Servers

If you are running OpenClaw on a remote server, within a Docker container, or in a continuous integration (CI) environment, it is almost always necessary to set `"noSandbox": true` in your browser configuration. The Chrome sandbox requires specific kernel capabilities that are typically not available in these constrained server environments.

### 5. Set `DISPLAY` for Headless Servers

Chrome extensions, even when running in a headless browser, often require a display context. On servers without a physical monitor, you can use Xvfb (X Virtual Framebuffer) to create a virtual display:

```bash
# Install Xvfb (if not already installed)
sudo apt-get install xvfb

# Start a virtual display in the background
Xvfb :99 -screen 0 1280x720x24 &

# Set the DISPLAY environment variable for OpenClaw
export DISPLAY=:99
```

This ensures that the browser environment, including extensions, has the necessary display context to function correctly.
