import requests
from datetime import datetime
import os

def get_weather_forecast(zip_code):
    api_key = os.getenv('OPENWEATHER_API_KEY')
    if not api_key:
        raise ValueError("API key is not set in environment variables.")
    
    base_url = "http://api.openweathermap.org/data/2.5/forecast"
    query = f"?zip={zip_code},us&appid={api_key}&units=imperial"
    response = requests.get(base_url + query)
    
    if response.status_code != 200:
        print(f"Failed to fetch data: {response.status_code} - {response.text}")
        return

    forecast_data = response.json()

    # Organizing the data by date
    daily_forecast = {}
    for item in forecast_data['list']:
        date = datetime.fromtimestamp(item['dt']).strftime('%m/%d/%Y')
        if date not in daily_forecast:
            daily_forecast[date] = {
                'temp_min': item['main']['temp_min'],
                'temp_max': item['main']['temp_max'],
                'precipitation': 0
            }
        else:
            daily_forecast[date]['temp_min'] = min(daily_forecast[date]['temp_min'], item['main']['temp_min'])
            daily_forecast[date]['temp_max'] = max(daily_forecast[date]['temp_max'], item['main']['temp_max'])
        
        if 'rain' in item:
            daily_forecast[date]['precipitation'] += item['rain'].get('3h', 0)
        elif 'snow' in item:
            daily_forecast[date]['precipitation'] += item['snow'].get('3h', 0)

    # Printing the forecast
    print("# Date       Temperature       Precipitation")
    for date, values in daily_forecast.items():
        print(f"# {date} {values['temp_max']:.0f}/{values['temp_min']:.0f}                 {values['precipitation']:.2f}")

# Example usage
zip_code = input("Enter the zip code: ")
get_weather_forecast(zip_code)

