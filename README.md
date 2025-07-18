import pandas as pd
import requests
import matplotlib.pyplot as plt
from datetime import datetime  # Make sure to import datetime

# API key and city
api_key = "c7c1f60c1b544e7cae495916252206"
city = "Sydney"

# Current weather API
current_url = f"http://api.weatherapi.com/v1/current.json?key={api_key}&q={city}"

# 5-day forecast API
forecast_url = f"http://api.weatherapi.com/v1/forecast.json?key={api_key}&q={city}&days=5"

# Fetch current weather data for current indicators: temperature, humidity, pressure
current_response = requests.get(current_url)
if current_response.status_code == 200:
    current_data = current_response.json()
    current_temp = current_data["current"]["temp_c"]
    current_humidity = current_data["current"]["humidity"]
    current_pressure = current_data["current"]["pressure_mb"]

    # Getting the current time
    current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    # Display current weather indicators
    print(f"Current Temperature: {current_temp} °C")
    print(f"Current Humidity: {current_humidity} %")
    print(f"Current Pressure: {current_pressure} hPa")
    print(f"Current Time: {current_time}")

    # Indicating conditions based on current values
    def weather_conditions(temp, humidity, pressure):
        if temp > 30:
            temp_indicator = "🔥"  # Hot
        elif temp < 10:
            temp_indicator = "❄️"  # Cold
        else:
            temp_indicator = "🌤️"  # Mild

        if humidity > 70:
            humidity_indicator = "💧"  # High Humidity
        elif humidity < 30:
            humidity_indicator = "🌵"  # Low Humidity
        else:
            humidity_indicator = "🌈"  # Comfortable Humidity

        if pressure > 1013:
            pressure_indicator = "⬆️"  # High Pressure
        elif pressure < 1010:
            pressure_indicator = "⬇️"  # Low Pressure
        else:
            pressure_indicator = "⚖️"  # Normal Pressure

        return temp_indicator, humidity_indicator, pressure_indicator

    temp_indicator, humidity_indicator, pressure_indicator = weather_conditions(
        current_temp, current_humidity, current_pressure
    )

    print(f"Temperature Indicator: {temp_indicator}")
    print(f"Humidity Indicator: {humidity_indicator}")
    print(f"Pressure Indicator: {pressure_indicator}")

else:
    print("Something went wrong! ")
    print("Status Code:", current_response.status_code)
    print("Message:", current_response.text)

# Fetch forecast weather data
response = requests.get(forecast_url)
if response.status_code == 200:
    data = response.json()

    # Lists to store forecast data
    dates = []
    temps = []
    humidity = []
    pressure = []

    for day in data["forecast"]["forecastday"]:
        dates.append(day["date"])
        temps.append(day["day"]["avgtemp_c"])
        humidity.append(day["day"]["avghumidity"])
        # Getting pressure at noon hour (12 PM)
        pressure.append(day["hour"][12]["pressure_mb"])

    # Bar Chart for 5 Days
    plt.figure(figsize=(12, 6))
    x = range(len(dates))

    plt.bar(x, temps, width=0.25, label="Temperature (°C)", color="skyblue")
    plt.bar(
        [i + 0.25 for i in x],
        humidity,
        width=0.25,
        label="Humidity (%)",
        color="orange",
    )
    plt.bar(
        [i + 0.50 for i in x],
        pressure,
        width=0.25,
        label="Pressure (hPa)",
        color="green",
    )

    plt.xticks([i + 0.25 for i in x], dates)
    plt.title(f"5-Day Weather Forecast for {city}")
    plt.xlabel("Date")
    plt.ylabel("Values")
    plt.legend()
    plt.tight_layout()
    plt.show()

    # Pie Chart for today's weather (Day 1)
    today = data["forecast"]["forecastday"][0]
    temp = today["day"]["avgtemp_c"]
    hum = today["day"]["avghumidity"]
    press = today["hour"][12]["pressure_mb"]

    values = [temp, hum, press]
    labels = ["Temp (°C)", "Humidity (%)", "Pressure (hPa)"]

    plt.figure(figsize=(6, 6))
    plt.pie(
        values,
        labels=labels,
        autopct="%1.1f%%",
        startangle=90,
        colors=["skyblue", "orange", "green"],
    )
    plt.title(f"Today's Weather Breakdown in {city}")
    plt.axis("equal")
    plt.show()

else:
    print("Something went wrong with fetching forecast data! ")
    print("Status Code:", response.status_code)
    print("Message:", response.text)
