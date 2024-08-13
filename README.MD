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

