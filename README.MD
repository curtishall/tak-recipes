### Tips and Tricks for massaging GPS data into TAK Server using node red


##### Custom icon sets:

WinTAK, iTAK and ATAK can support icon sets instead of sending a CoT type.  Here is an example of XML being sent to TAK Server with a Fire Truck icon:



```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<event version="2.0" uid="6276" type="a-u-G" how="h-g-i-g-o" time="2024-08-11T21:09:06.000+00:00" start="2024-08-11T21:09:06.000+00:00" icon="f7f71666-8b28-4b57-9fbb-e38e61d33b79/Service/firebrigade.png" stale="2024-08-13T18:12:06.712Z">
  <point lat="38.713837" lon="-91.714066" hae="200.4" ce="9999999.0" le="9999999.0"/>
  <detail>
    <contact callsign="BRUSH 1106"/>
    <usericon iconsetpath="6d781afb-89a6-4c07-b2b9-a89748b6a38f/Service/firebrigade.png"/>
    <remarks>No Address</remarks>
    <uid>
      <Droid>6276</Droid>
    </uid>
  </detail>
</event>   
```
Node RED function:

```javascript

const thirtySeconds = 30 * 1000;
const fiveMinutes = 60 * 5 * 1000;
const tenMinutes = 60 * 10 * 1000;
const sixtyMinutes = 60 * 60 * 1000;
const now = new Date(Date.now()).toISOString();
const stale = new Date(Date.now() + fiveMinutes).toISOString();

const traccarResponse = msg.payload;
const devices = flow.get('devices') || {}; // Get stored devices

if (!Array.isArray(traccarResponse) || traccarResponse.length === 0) {
    node.error("Payload is not an array or is empty");
    return null;
}

const positions = traccarResponse.map((positionJson) => {
    const latitude = positionJson["latitude"];
    const longitude = positionJson["longitude"];
    const altitude = positionJson["altitude"] || 0; // Default to 0 if undefined
    const deviceId = positionJson["deviceId"] || "Unknown Device"; // Default if undefined

    if (latitude === undefined || longitude === undefined) {
        node.warn(`Missing latitude/longitude for device ID ${deviceId}`);
        return null;
    }

    const deviceName = devices[deviceId] || deviceId.toString(); // Get device name or fallback to ID

    const callStartTime = positionJson["deviceTime"] || now;
    const lastUpdate = positionJson["fixTime"] || now;
    const remarks = positionJson["address"] || "No Address";

    return {
        "payload": {
            "event": {
                "_attributes": {
                    "version": "2.0",
                    "uid": deviceId.toString(),
                    "type": "a-u-G",
                    "how": "h-g-i-g-o",
                    "time": lastUpdate,
                    "start": callStartTime,
                    "icon": "f7f71666-8b28-4b57-9fbb-e38e61d33b79/Service/firebrigade.png",
                    "stale": stale
                },
                "point": {
                    "_attributes": {
                        "lat": latitude,
                        "lon": longitude,
                        "hae": altitude,
                        "ce": "9999999.0",
                        "le": "9999999.0"
                    }
                },
                "detail": {
                    "contact": {
                        "_attributes": {
                            "callsign": deviceName
                        }
                    },
                  "usericon": {
                          "_attributes": {
                            "iconsetpath": "6d781afb-89a6-4c07-b2b9-a89748b6a38f/Service/firebrigade.png"
                                        }
                             },    
                    "remarks": remarks,
                    "uid": {
                        "Droid": deviceId.toString()
                    }
                }
            }
        }
    };
}).filter(event => event !== null); // Filter out any null events due to missing data

if (positions.length > 0) {
    positions.forEach((takEvent) => {
        node.send(takEvent);
    });
} else {
    node.warn("No valid TAK events generated");
}

return null;
```
