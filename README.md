# arduinohttpgetandpostrequest
arduino with lan module http get and post request
#include <SPI.h>
#include <Ethernet.h>

// Replace the MAC address below by the MAC address printed on a sticker on the Arduino Shield 2
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };

EthernetClient client;

char HOST_NAME[] = "192.168.1.19"; // Change to your Django server's IP address
int HTTP_PORT = 8000;
String HTTP_METHOD = "GET";
String PATH_NAME = "/esp32/insert_temp/";
String temperature = "29.1"; // Replace with your temperature reading

void setup() {
  Serial.begin(9600);

  // Initialize the Ethernet shield using DHCP:
  if (Ethernet.begin(mac) == 0) {
    Serial.println("Failed to obtain an IP address using DHCP");
    while (true);
  }

  // Wait for the Ethernet connection to be ready
  delay(1000);
}

void loop() {
  // Connect to the web server on port 80:
  if (client.connect(HOST_NAME, HTTP_PORT)) {
    Serial.println("Connected to server");

    // Make a HTTP request:
    // Send HTTP header
    client.print(HTTP_METHOD + " " + PATH_NAME + "?temperature=" + temperature + " HTTP/1.1\r\n");
    client.print("Host: ");
    client.println(HOST_NAME);
    client.println("Connection: close\r\n");

    // Read response from the server
    while (client.connected()) {
      if (client.available()) {
        char c = client.read();
        Serial.print(c);
      }
    }

    // The server disconnected, stop the client:
    client.stop();
    Serial.println("\nDisconnected");
  } else {
    // If not connected:
    Serial.println("Connection failed");
  }

  // Wait for a minute before sending the next temperature reading
  delay(60000);
}

django/views.py
# temperature_monitoring/views.py

from django.http import HttpResponse
from .models import TemperatureReading

def insert_temperature(request):
    if 'temperature' in request.GET:
        temperature = request.GET['temperature']
        TemperatureReading.objects.create(temperature=float(temperature))
        return HttpResponse("Temperature recorded successfully")
    else:
        return HttpResponse("Temperature is not set")

# temperature_monitoring/urls.py

from django.urls import path
from .views import insert_temperature

urlpatterns = [
path('insert_temp/', insert_temperature, name='insert_temperature'),
]

# temperature_monitoring/models.py

from django.db import models

class TemperatureReading(models.Model):
    temperature = models.FloatField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"Temperature: {self.temperature}°C at {self.created_at}"

# temperature_monitoring/serializers.py

from rest_framework import serializers
from .models import TemperatureReading

class TemperatureReadingSerializer(serializers.ModelSerializer):
    class Meta:
        model = TemperatureReading
        fields = ['id', 'temperature', 'created_at']

