---
title: BlobEvent()
slug: Web/API/BlobEvent/BlobEvent
tags:
  - API
  - BlobEvent
  - Constructor
  - DOM
  - DOM Reference
  - Experimental
  - Media Stream Encoding
  - Reference
browser-compat: api.BlobEvent.BlobEvent
---
{{APIRef("Media Capture and Streams")}}{{SeeCompatTable}}

The **`BlobEvent()`** constructor returns a newly created
{{domxref("BlobEvent")}} object with an associated {{domxref("Blob")}}.

## Syntax

```js
new BlobEvent(options)
new BlobEvent(options, timecode)
```

### Parameters

- `options`
  - : An object containing the properties allowed as `options` in {{domxref("Event.Event", "Event()")}} and the following property:
    - `data`
      - : The {{domxref("Blob")}} associated with the event.
- `timecode` {{optional_inline}}
  - : A {{domxref("DOMHighResTimeStamp")}} to be used in initializing the blob event.

## Specifications

{{Specifications}}

## Browser compatibility

{{Compat}}

## See also

- The {{domxref("BlobEvent")}} interface it belongs to.
