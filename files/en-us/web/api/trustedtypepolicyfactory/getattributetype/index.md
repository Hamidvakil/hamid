---
title: TrustedTypePolicyFactory.getAttributeType()
slug: Web/API/TrustedTypePolicyFactory/getAttributeType
tags:
  - API
  - Method
  - Reference
  - getAttributeType
  - TrustedTypePolicyFactory
browser-compat: api.TrustedTypePolicyFactory.getAttributeType
---
{{DefaultAPISidebar("Trusted Types API")}}

The **`getAttributeType()`** method of the {{domxref("TrustedTypePolicyFactory")}} interface allows web developers to check if a Trusted Type is required for an element, and if so which Trusted Type is used.

## Syntax

```js
getAttributeType(tagName, attribute)
getAttributeType(tagName, attribute, elementNS)
getAttributeType(tagName, attribute, elementNS, attrNS)
```

### Parameters

- `tagName`
  - : A {{domxref("DOMString","string")}} containing the name of an HTML tag.
- `attribute`
  - : A {{domxref("DOMString","string")}} containing an attribute.
- `elementNS`{{optional_inline}}
  - : A {{domxref("DOMString","string")}} containing a namespace, if empty defaults to the HTML namespace.
- `attrNS`{{optional_inline}}
  - : A {{domxref("DOMString","string")}} containing a namespace, if empty defaults to null.

### Return value

A {{domxref("DOMString","string")}} with one of:

- `"TrustedHTML"`
- `"TrustedScript"`
- `"TrustedScriptURL"`

Or, null.

## Examples

In this example, passing the {{htmlelement("script")}} element and {{htmlattrxref("src")}} attribute to `getAttributeType` returns "TrustedScriptURL".

```js
console.log(trustedTypes.getAttributeType('script', 'src')); // "TrustedScriptURL"
```

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}
