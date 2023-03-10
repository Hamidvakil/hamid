---
title: InstallEvent()
slug: Web/API/InstallEvent/InstallEvent
tags:
  - API
  - Constructor
  - Experimental
  - InstallEvent
  - Reference
  - Service Workers
  - ServiceWorker
browser-compat: api.InstallEvent.InstallEvent
---
{{non-standard_header}}{{deprecated_header}}

The **`InstallEvent()`** constructor creates a new {{domxref("InstallEvent")}} object.

## Syntax

```js
new InstallEvent(type)
new InstallEvent(type, init)
```

### Parameters

- `type`
  - : The type of the event.
- `init` {{optional_inline}}

  - : An options object containing any custom settings that you want to apply to the event object. Available options are as follows:

    - `activeWorker`
      - : The {{domxref("ServiceWorker")}} that is currently actively controlling the page.

## Browser compatibility

{{Compat}}

## See also

- {{jsxref("Promise")}}
- [Fetch API](/en-US/docs/Web/API/Fetch_API)
