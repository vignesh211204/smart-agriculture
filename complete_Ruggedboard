import os
import time
import json
import serial
import threading
import paho.mqtt.client as mqtt
import mraa

# ================= GPIO Configuration using MRAA ===================
RELAY_GPIO_NUMBER = 37  # MRAA number
relay = mraa.Gpio(RELAY_GPIO_NUMBER)
relay.dir(mraa.DIR_OUT)

def set_relay_state(state):
    """Control relay using MRAA GPIO."""
    try:
        relay.write(1 if state else 0)
        print("[GPIO] Relay {}".format("ON" if state else "OFF"))
    except Exception as e:
        print("[GPIO] Relay Write Error: {}".format(e))

# ================= UART Configuration ===================
SERIAL_PORT = "/dev/ttyS3"
BAUD_RATE = 115200

try:
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print("[UART] Connected to {} at {} baud.".format(SERIAL_PORT, BAUD_RATE))
    ser.flush()
except serial.SerialException as e:
    print("[UART] Connection Error: {}".format(e))
    exit(1)

# ================= MQTT Configuration ===================
ACCESS_TOKEN = 'H59jGw9ZK1tATFVshl6a'
BROKER = "demo.thingsboard.io"
PORT = 1883
PUBLISH_TOPIC = "v1/devices/me/telemetry"
SUBSCRIBE_TOPIC = "v1/devices/me/rpc/request/+"

def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("[MQTT] Connected to ThingsBoard.")
        client.subscribe(SUBSCRIBE_TOPIC)
    else:
        print("[MQTT] Connection failed: {}".format(rc))

def on_disconnect(client, userdata, rc):
    print("[MQTT] Disconnected.")

def on_message(client, userdata, msg):
    try:
        message = msg.payload.decode()
        print("[MQTT] RPC Received: {} => {}".format(msg.topic, message))
        payload = json.loads(message)
        method = payload.get("method", "")
        params = payload.get("params", "")

        print("[DEBUG] Method:", method)
        print("[DEBUG] Params:", params)

        if method == "setValue":
            print("[DEBUG] setValue method triggered")

            request_id = msg.topic.split("/")[-1]
            response_topic = "v1/devices/me/rpc/response/{}".format(request_id)
            client.publish(response_topic, json.dumps({"success": True}), qos=1)
            print("[MQTT] Sent RPC response")

            relay_state = None
            if isinstance(params, dict) and "relay" in params:
                relay_state = 1 if str(params["relay"]).lower() in ["1", "true", "on"] else 0
                print("[DEBUG] Relay param (dict) detected:", relay_state)
            elif isinstance(params, bool):
                relay_state = 1 if params else 0
                print("[DEBUG] Relay param (bool) detected:", relay_state)

            if relay_state is not None:
                set_relay_state(relay_state)
            else:
                print("[DEBUG] Unrecognized RPC param format:", params)

            command = json.dumps({"setValue": params}) + "\n"
            ser.write(command.encode())
            print("[UART] Sent to STM32: {}".format(command.strip()))

    except json.JSONDecodeError:
        print("[MQTT] Invalid JSON")

# MQTT client setup
mqtt_client = mqtt.Client()
mqtt_client.username_pw_set(ACCESS_TOKEN)
mqtt_client.on_connect = on_connect
mqtt_client.on_disconnect = on_disconnect
mqtt_client.on_message = on_message

try:
    mqtt_client.connect(BROKER, PORT, keepalive=60)
except Exception as e:
    print("[MQTT] Connection Error: {}".format(e))
    exit(1)

# ================= UART Reading + MQTT Publishing ===================
def uart_read_loop():
    print("[UART] Listening for sensor data...")
    while True:
        try:
            if ser.in_waiting > 0:
                raw = ser.readline().decode('utf-8', errors='ignore').strip()
                if not raw:
                    continue

                print("[UART] Received: {}".format(raw))

                if ":" in raw:
                    key, value = raw.split(":", 1)
                    key = key.strip()
                    value = value.strip()

                    # Normalize STM32 key names to MQTT keys
                    if "ADC Value(Soil Sensor)" in key:
                        key = "Soil Moisture ADC"
                    elif "ADC2 Value(DHT22)" in key:
                        key = "ADC2 Value(DHT22)"
                    elif "Temperature" in key:
                        key = "Temperature"
                    elif "Humidity" in key:
                        key = "Humidity"

                    # Value formatting
                    try:
                        if "C" in value or "%" in value:
                            formatted_value = value
                        elif "." in value:
                            formatted_value = float(value)
                        else:
                            formatted_value = int(value)
                    except ValueError:
                        formatted_value = value

                    data = {key: formatted_value}
                    mqtt_client.publish(PUBLISH_TOPIC, json.dumps(data), qos=1)
                    print("[MQTT] Published: {}".format(data))

            time.sleep(0.1)

        except Exception as e:
            print("[UART ERROR] {}".format(e))
            time.sleep(1)

# Start UART reading in background
uart_thread = threading.Thread(target=uart_read_loop)
uart_thread.daemon = True
uart_thread.start()

# Start MQTT client loop
try:
    mqtt_client.loop_forever()
except KeyboardInterrupt:
    print("\n[EXIT] Shutting down...")
finally:
    set_relay_state(False)
    mqtt_client.disconnect()
    if ser.is_open:
        ser.close()
    print("[CLEANUP] Done.")

