import RPi.GPIO as GPIO
import time
import requests

pir_pin = 17  # Puedes cambiar el número del pin según tu configuración
led_pin = 18  # Puedes cambiar el número del pin según tu configuración
thingspeak_api_key = "9H93BA1GS0EM8V2L"
thingspeak_url = f"https://api.thingspeak.com/update?api_key={thingspeak_api_key}&field1="

GPIO.setmode(GPIO.BCM)
GPIO.setup(pir_pin, GPIO.IN)
GPIO.setup(led_pin, GPIO.OUT)

tiempo_led_encendido = 0
tiempo_led_apagado = 0
inicio_24_horas = time.time()

try:
    while True:
        if GPIO.input(pir_pin):
            GPIO.output(led_pin, GPIO.HIGH)
            tiempo_led_encendido += 1
        else:
            GPIO.output(led_pin, GPIO.LOW)
            tiempo_led_apagado += 1

        time.sleep(1)

        tiempo_transcurrido = time.time() - inicio_24_horas
        if tiempo_transcurrido >= 24 * 60 * 60:
            porcentaje_encendido = (tiempo_led_encendido / (tiempo_led_encendido + tiempo_led_apagado)) * 100
            print(f"Porcentaje de tiempo del LED encendido en las últimas 24 horas: {porcentaje_encendido:.2f}%")

            try:
                requests.get(f"{thingspeak_url}{porcentaje_encendido}")
            except Exception as e:
                print(f"Error al enviar datos a ThingSpeak: {e}")

            tiempo_led_encendido = 0
            tiempo_led_apagado = 0
            inicio_24_horas = time.time()

except KeyboardInterrupt:
    pass

finally:
    GPIO.cleanup(
