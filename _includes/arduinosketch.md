Arduino sketch used:   

```c++
{% raw %}
#include <OneWire.h>
#include <DallasTemperature.h>

// Data wire is plugged into port 2 on the Arduino
#define ONE_WIRE_BUS 2

// Setup a oneWire instance to communicate with any OneWire devices (not just Maxim/Dallas temperature ICs)
OneWire oneWire(ONE_WIRE_BUS);

// Pass our oneWire reference to Dallas Temperature.
DallasTemperature sensors(&oneWire);

/*
   The setup function. We only start the sensors here
*/
void setup(void)
{
  // start serial port
  Serial.begin(9600);

  // Start up the library
  sensors.begin();
  sensors.setResolution(12);
}

/*
   Main function, get and show the temperature
*/
void loop(void)
{
  int numberOfDevices = sensors.getDeviceCount();
  if (numberOfDevices > 0) {
    sensors.requestTemperatures();
      }
      float t = sensors.getTempCByIndex(0);
      Serial.println(t);

  delay(30000);
}
{% endraw %}
```
