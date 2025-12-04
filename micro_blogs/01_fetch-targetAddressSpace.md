**fetch() support update:**  
Modern browsers now fully support `fetch()` with `targetAddressSpace: 'local'`, allowing developers to request exemptions from mixed-content checks once the user grants LNA permission. This makes it possible to reach HTTP-only private devices without mixed-content blocking.

This capability dates back to the PNA era and remains a core part of private-network access today.

**Example: Fetching from a local device**
```javascript
// Example: Accessing a local HTTP device after LNA permission is granted
fetch("http://192.168.1.10/status", {
  targetAddressSpace: "local"
})
  .then(res => res.json())
  .then(data => console.log("Device status:", data))
  .catch(err => console.error("Fetch failed:", err));
```
Reference: https://www.mail-archive.com/blink-dev@chromium.org/msg13745.html
