---
title: How Data Flows through Dash.js
date: "2022-09-19T11:00:00.284Z"
---

For now this is the last part planned, and we will briefly look at encrypted streams.
 [stream](./../../streams/sintelTrailerStream.mpd).
It needs a browser that uses widevine DRM and where CORS is disabled as it is configured for testing only.
The below sample was modified from the Dash.js samples and our stream files should sit in the same directory.

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Widevine DRM instantiation example</title>


    });
</script>
<script src="/highlighter.js"></script>
</body>
</html>
```

Lets look at the init segment.

![initEnc](./initEnc.png)

