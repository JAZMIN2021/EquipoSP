



![Simulacion] (https://github.com/JAZMIN2021/EquipoSP/assets/79472215/dcf77df2-e970-4445-b546-e21a8df1d5d9)


# Codigo 

```python
# Módulos necesarios.
from machine import Pin, UART, I2C
from ssd1306 import SSD1306_I2C
import utime, time
from gps import nmea_strings

# Conexión a Oled I2C.
i2c=I2C(0, sda=Pin(0), scl=Pin(1), freq=400000)
oled = SSD1306_I2C(128, 64, i2c)


# Estado del satélite.
FIX_STATUS = False

# Variables para guardar los parámetros de las coordenadas.
latitude = ""
longitude = ""
satellites = ""
gpsTime = ""

# Coordenadas del GPS.
def getPositionData(nmea_string):
    global FIX_STATUS, latitude, longitude, satellites, gpsTime

    parts = nmea_string.decode().split(',')

    if parts[0] == '$GPGGA' and len(parts) == 15:
        if parts[1] and parts[2] and parts[3] and parts[4] and parts[5] and parts[6] and parts[7]:
            print(nmea_string)

            latitude = convertToDegree(parts[2])

            if parts[3] == 'S':
                latitude = -latitude
            longitude = convertToDegree(parts[4])

            if parts[5] == 'W':
                longitude = -longitude
            satellites = parts[7]
            gpsTime = parts[1][0:2] + ":" + parts[1][2:4] + ":" + parts[1][4:6]
            FIX_STATUS = True



# Convertir la longitud y latitud.
def convertToDegree(raw_degrees):
    raw_as_float = float(raw_degrees)
    first_digits = int(raw_as_float / 100)
    next_two_digits = raw_as_float - float(first_digits * 100)

    converted = float(first_digits + next_two_digits / 60.0)
    return converted



# Bucle externo para procesar cada cadena NMEA.
for nmea_string in nmea_strings:
    getPositionData(nmea_string)

    # Si los datos son encontrados, se imprimen en el Oled display.
    if FIX_STATUS:
        print("Fix......")
        oled.fill(0)
        oled.text("Latitud: " + str(latitude), 0, 0)
        oled.text("Longitud: " + str(longitude), 0, 10)
        oled.text("NoSatelite: " + str(satellites), 0, 20)
        oled.text("Tiempo: " + str(gpsTime), 0, 30)
        oled.show()
        print(latitude)
        print(longitude)
        print(satellites)
        print(gpsTime)

# Si no se encuentra ningún fix en las cadenas NMEA.
if not FIX_STATUS:
    print("No GPS fix found.")
    oled.fill(0)
    oled.text("No GPS fix found", 0, 0)
    oled.show()
# Codigo 2.1.2
