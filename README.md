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

```js
angular.merge({}, JSON.parse('{"__proto__":{"innerHTML":"<img src=x onerror=alert(1337)>","polluted":"yes"}}'));
