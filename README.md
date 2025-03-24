# Bypassing CAPTCHAs with Cypress

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to handle CAPTCHAs in Cypress, including effective bypass methods and what to do when CAPTCHAs persist, ensuring seamless browser automation.

- [What Is a CAPTCHA and Can It Be Automated?](#what-is-a-captcha-and-can-it-be-automated)
- [CAPTCHAs and Cypress: A Difficult Relationship](#captchas-and-cypress-a-difficult-relationship)
- [How to Handle CATPCHAs in Cypress](#how-to-handle-catpchas-in-cypress)
  - [Approach #1: Disable the CAPTCHAs](#approach-1-disable-the-captchas)
  - [Approach #2: Automate the CAPTCHA Interaction](#approach-2-automate-the-captcha-interaction)
  - [Approach #3: Integrate an Antibot Browser](#approach-3-integrate-an-antibot-browser)
- [The Cypress Solutions Do Not Work: What to Do Now?](#the-cypress-solutions-do-not-work-what-to-do-now)

## What Is a CAPTCHA and Can It Be Automated?

A CAPTCHA, short for **C**ompletely **A**utomated **P**ublic **T**uring test to tell **C**omputers and **H**umans **A**part, is a security measure designed to distinguish real users from automated bots. It presents challenges that are easy for humans to solve but difficult for machines. CAPTCHAs are commonly placed in key areas of a webpage to prevent bot activity.

### Common CAPTCHA Types

Popular CAPTCHA providers include **Google reCAPTCHA, hCaptcha,** and **BotDetect.** These CAPTCHAs typically fall into one or more of the following categories:

- **Text-based CAPTCHAs** – Users must type a sequence of distorted letters or numbers.  
- **Image-based CAPTCHAs** – Users are asked to identify specific objects in a grid of images.  
- **Audio-based CAPTCHAs** – Users must type the words they hear from an audio clip.  
- **Puzzle CAPTCHAs** – Users solve simple challenges, such as clicking a specific object or answering a question.  

![Puzzle CAPTCHA example](https://brightdata.com/wp-content/uploads/2024/07/Puzzle-CAPTCHA-example-1.png)

### How CAPTCHAs Are Used

CAPTCHAs are often integrated into critical user flows, such as form submissions, to prevent bots from completing actions automatically:  

![CAPTCHA in form submission process](https://brightdata.com/wp-content/uploads/2024/07/CAPTCHA-as-a-step-of-a-form-submission-process-example.png)

In these cases, the CAPTCHA is always displayed and cannot be bypassed using simple automation techniques. Some CAPTCHA-solving services use human operators or specialized AI models to solve these challenges in real time, but hardcoded CAPTCHAs are relatively rare due to their negative impact on user experience.

### CAPTCHAs in Advanced Anti-Bot Solutions

More commonly, CAPTCHAs are part of advanced anti-bot systems, such as Web Application Firewalls (WAFs) ([Learn More](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)):  

![Web Application Firewall example](https://brightdata.com/wp-content/uploads/2024/07/Example-of-a-Web-Application-Firewall-1024x488.png)

In such systems, CAPTCHAs are dynamically triggered when the website suspects the user might be a bot. This means they can sometimes be avoided by making the bot behave more like a real human—such as using a real browser and mimicking human interactions.

However, bypassing CAPTCHAs remains an ongoing challenge, as bot detection measures continuously evolve. Automated scripts must be updated frequently to counteract new anti-bot techniques.

## CAPTCHAs and Cypress: A Difficult Relationship

[Cypress](https://www.cypress.io/) is a front-end testing tool designed for the modern web. While it can be used for general browser automation tasks such as web scraping, its primary focus is [end-to-end (E2E) testing](https://www.techtarget.com/searchsoftwarequality/definition/End-to-end-testing). This means Cypress is best suited for interacting with websites and applications that you control.  

When Cypress is used to target external or third-party websites, problems can arise. The [official Cypress documentation](https://docs.cypress.io/guides/references/best-practices) advises against interacting with third-party sites whenever possible. One key reason for this is the risk of being detected as a bot and encountering a CAPTCHA.  

CAPTCHAs are specifically designed to block automated scripts, including those running within a Cypress test environment. When a CAPTCHA appears, it can disrupt Cypress automation workflows, preventing tests from completing successfully.  

However, while bypassing CAPTCHAs in Cypress is challenging, it is not impossible. The next sections will explore potential solutions and workarounds for handling CAPTCHAs in Cypress automation.  

## How to Handle CATPCHAs in Cypress

While CAPTCHAs are one of Cypress’s main challenges, it is not time to give up just yet. Let’s explore some potential approaches for implementing Cypress CAPTCHA bypassing logic.

### Approach #1: Disable the CAPTCHAs

Most CAPTCHA providers allow challenges to be disabled or bypassed in a testing environment. If you have control over the site where automation is needed, consider disabling the CAPTCHA entirely or replacing it with a simpler version.

For example, with reCAPTCHA v3, you can create a separate key for testing environments. For reCAPTCHA v2, you can use the following test keys:

* Site key: `6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI`
* Secret key: `6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe`

While using these keys, you will always get a reCAPTCHA “No CAPTCHA” widget as below:

![](https://developers.google.com/static/recaptcha/images/recaptcha_test.png)

This setup will display a special warning message to indicate that the CAPTCHA is not meant for production use. Automating a click on this message ensures that the anti-bot verification always passes. More details can be found in the [reCAPTCHA documentation](https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do).

Other CAPTCHA providers offer similar options for test environments.

### Approach #2: Automate the CAPTCHA Interaction

Some CAPTCHAs require only basic interactions, such as clicking a checkbox, as seen in the reCAPTCHA “No CAPTCHA” widget.  

![Simple clicking CAPTCHA example](https://brightdata.com/wp-content/uploads/2024/07/Simple-clicking-CAPTCHA-example.png)

While these challenges may appear simple, they often analyze mouse movements and other behavioral signals to verify if the user is human. However, not all CAPTCHAs are this advanced. Some are designed to block only basic bots and can be easier to bypass. In such cases, it may be possible to automate them using Cypress logic.  

Inspecting the CAPTCHA element from the example above reveals that it is embedded within an iframe.

![Inspecting the CAPTCHA element](https://brightdata.com/wp-content/uploads/2024/07/Inspecting-the-CAPTCHA-element-1024x510.png)

This is a common behavior for most CAPTCHA providers.

Since Cypress cannot automatically deal with cross-domain iframes, [set the `chromeWebSecurity` property to `false`](https://docs.cypress.io/guides/guides/web-security#Set-chromeWebSecurity-to-false) in the `cypress.json` file:

```json
{

"chromeWebSecurity": false

}
```

You can then select the CAPTCHA checkbox element and click it. For reCAPTCHA's “No CAPTCHA” widget, the automation code for doing that will be:

```js
cy.get('iframe[src*=recaptcha]')

.its('0.contentDocument')

.should(d => d.getElementById('recaptcha-token').click())
```

CAPTCHAs have become sophisticated enough to distinguish between clicks from a robot and a human, so this workaround will not work in most situations. What works today may not work tomorrow. For the most up-to-date approaches, check out the [GitHub gist](https://gist.github.com/Dema/dcc2b4f91b97ec9d56afc871e66fd3d7) where this approach comes from.

### Approach #3: Integrate an Antibot Browser

The previous two CAPTCHA bypass approaches rely on too many assumptions. A more efficient solution is to configure Cypress to control an anti-detect browser.

By default, Cypress provides access to one of the locally installed browsers from the [following list](https://docs.cypress.io/guides/guides/launching-browsers#Browsers):

* Chrome
* Chrome Beta
* Chrome Canary
* Chromium
* Edge
* Edge Beta
* Edge Canary
* Edge Dev
* Electron
* Firefox
* Firefox Developer Edition
* Firefox Nightly
* WebKit (Experimental)

It also supports any Chromium-based browsers. So, pick a Chromium-based browser from the list and install it.

Then, instruct Cypress to launch a script with the specified browser as below:

```bash
cypress open --browser <path_to_your_browser>
```

Here, `<path_to_your_browser>` is the absolute path to the folder that contains the binary of your anti-detect browser.

Similarly, you can configure the Cypress UI to show your anti-detect browser as a selectable option by adding the following code in `cypress.config.js`:

```js
import { defineConfig } from 'cypress'

export default defineConfig({

e2e: {

setupNodeEvents(on, config) {

const antidetectBrowser = {

name: '<ANTIDETECT_BROWSER_NAME>',

channel: 'stable',

family: 'chromium',

displayName: '<ANTIDETECT_BROWSER_DISPLAY_NAME>',

version,

path: '<path_to_your_browser>',

majorVersion,

}

return {

browsers: config.browsers.concat(antidetectBrowser),

}

},

},

})
```

> **Note**:
> 
> Instructing Cypress to run your automated code in an anti-detect browser will only reduce the chance of getting detected as a bot. If the anti-bot systems understand that you are running automated code, they may still enforce some CAPTCHAs to stop you.

## The Cypress Solutions Do Not Work: What to Do Now?

All three methods presented above have some major drawbacks:

* **Approach #1**: Requires you to have access to the code of the target site, this is not applicable to external online sites.
* **Approach #2**: Works only against very simple CAPTCHAs and is not reliable.
* **Approach #3**: Implies additional expenses, only helps avoid CAPTCHAs, not solve them.

While they are all worth trying, none of them allow you to bypass CAPTCHAs programmatically in your Cypress automation.

If you are looking for a real Cypress CAPTCHA bypasser, try Bright Data web scraping solutions. They have a [dedicated CAPTCHA-solving feature](https://brightdata.com/products/web-unlocker/captcha-solver) to automatically handle reCAPTCHA, hCaptcha, px\_captcha, SimpleCaptcha, GeeTest CAPTCHA, FunCaptcha, Cloudflare Turnstile, AWS WAF Captcha, KeyCAPTCHA, and many others.

Start with a free trial today.