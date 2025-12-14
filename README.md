🔑 API Configuration
Go to OpenWeatherMap and create a free account
Navigate to the API Keys section
Generate a new API key (it may take 10-15 minutes to activate)
Replace the placeholder API key in the code:
# In weather_app.py
API_KEY = "your-openweathermap-api-key-here"

🧩Code Structure & Explanation
Full Code Breakdown
The code is organized into 4 main sections for readability and maintainability:

1. Configuration (Hardcoded Constants)
API_KEY = "991a821ffc89b7ed83008db6c24a7494"  # Replace with your key
BASE_URL = "http://api.openweathermap.org/data/2.5/weather"
ONE_CALL_URL = "http://api.openweathermap.org/data/2.5/onecall"
ICON_BASE_URL = "http://openweathermap.org/img/wn/"
API_KEY: Authentication token for OpenWeatherMap API
BASE_URL: Endpoint to fetch basic weather data (used to get latitude/longitude for a city)
ONE_CALL_URL: Endpoint for detailed weather data (current, hourly, daily forecast)
ICON_BASE_URL: Base URL for OpenWeatherMap's weather icons

2. Helper Functions (Reusable Logic)
get_wind_direction(deg)
Converts wind degrees (0-360) to human-readable directions (N, NE, E, etc.):
def get_wind_direction(deg):
    directions = ["N", "NE", "E", "SE", "S", "SW", "W", "NW"]
    idx = round(deg / 45) % 8
    return directions[idx]
Takes wind degrees (e.g., 90 = East, 180 = South)
Returns a 2-letter direction code (e.g., "NW" for 315°)
convert_timestamp(timestamp, timezone_offset)
Converts Unix timestamps to local time (HH:MM format):
def convert_timestamp(timestamp, timezone_offset):
    local_time = datetime.fromtimestamp(timestamp + timezone_offset)
    return local_time.strftime("%H:%M")
Adjusts for the city's timezone offset (from API response)
Used for hourly forecast time labels
get_date_from_timestamp(timestamp, timezone_offset)
Converts Unix timestamps to human-readable dates (e.g., "Mon, 15 Jul"):
def get_date_from_timestamp(timestamp, timezone_offset):
    local_date = datetime.fromtimestamp(timestamp + timezone_offset)
    return local_date.strftime("%a, %d %b")
Used for 7-day forecast date labels

3. Main App Class (WeatherApp)
The core of the application—handles UI creation and data processing.
__init__(self, root)
Initializes the main window (title, size, centering)
Defines color scheme (light blue background, white cards)
Calls create_ui() to build the interface
center_window(self, width, height)
Centers the app window on the user's screen:
def center_window(self, width, height):
    screen_width = self.root.winfo_screenwidth()
    screen_height = self.root.winfo_screenheight()
    x = (screen_width - width) // 2
    y = (screen_height - height) // 2
    self.root.geometry(f"{width}x{height}+{x}+{y}")
create_ui(self)
Builds the entire user interface in modular sections:
Title Bar: App name ("Global Weather Tracker")
Search Bar: City input field + "Get Weather" button
Tab System: Two tabs ("Current Weather" and "7-Day Forecast")
Current Weather Section:
Left panel: Weather icon + current temperature + location
Right panel: Metrics (humidity, wind, UV index, etc.)
Hourly forecast (4pm-6pm)
7-Day Forecast Section: Card-based layout for daily forecasts
get_weather(self)
The main data-fetching logic:
Validates city input (shows warning if empty)
Step 1: Fetch basic data (latitude/longitude) using BASE_URL
Step 2: Fetch detailed data (current/hourly/daily) using ONE_CALL_URL
Updates the UI with fresh data (calls update_current(), update_hourly(), update_7day())
Error handling: Catches API errors, network issues, or invalid city names
update_current(self, data, basic_data, tz_offset)
Updates the "Current Weather" tab with real-time data:
Extracts current weather data (temp, feels-like, humidity, etc.)
Updates all label text (e.g., self.temp_label.config(text=f"{round(current['temp'])} °C"))
Loads and displays the weather icon via load_icon()
update_hourly(self, hourly_data, tz_offset)
Filters and displays hourly forecast for 4pm, 5pm, 6pm:
Clears old hourly data
Iterates through 24 hours of hourly data
Creates a card for each target hour (16:00, 17:00, 18:00)
Displays time, icon, temperature, and feels-like temp
update_7day(self, daily_data, tz_offset)
Builds the 7-day forecast:
Clears old daily data
Iterates through the next 7 days of forecast data
Creates a card for each day with:
Date (e.g., "Tue, 16 Jul")
Weather icon
Min/max temperature
Humidity and wind speed
load_icon(self, icon_code, label, size)
Loads and displays weather icons from OpenWeatherMap:
def load_icon(self, icon_code, label, size):
    try:
        url = f"{ICON_BASE_URL}{icon_code}@2x.png"
        res = requests.get(url)
        img = Image.open(io.BytesIO(res.content)).resize(size, Image.Resampling.LANCZOS)
        photo = ImageTk.PhotoImage(img)
        label.config(image=photo)
        label.image = photo  # Keep reference to avoid garbage collection
    except:
        label.config(text="🌤️", font=("Arial", 30))  # Fallback emoji
Fetches the icon image from OpenWeatherMap's CDN
Resizes the image to fit the UI
Uses a fallback emoji if the icon fails to load

4. App Execution
if __name__ == "__main__":
    # Install Pillow if missing
    try:
        from PIL import Image, ImageTk
    except ImportError:
        import subprocess, sys
        subprocess.check_call([sys.executable, "-m", "pip", "install", "pillow"])
        from PIL import Image, ImageTk
    root = tk.Tk()
    app = WeatherApp(root)
    root.mainloop()
Auto-installs pillow if missing (for new users)
Initializes the Tkinter root window
Starts the app's main loop (keeps the window open)

🎯 Usage
Enter a City: Type any city name (e.g., "London", "New York", "Tokyo") in the search bar
Get Weather: Click the "Get Weather" button
View Current Weather: Check real-time metrics and hourly forecast (4pm-6pm)
View 7-Day Forecast: Switch to the "7-Day Forecast" tab to see daily predictions
Error Handling: If the city is invalid or the API fails, a friendly error message will appear

🚨 Troubleshooting
Common Issues
API Key Errors:
Ensure your API key is active (wait 10-15 minutes after creation)
Verify the key is correctly pasted (no extra spaces)
Check OpenWeatherMap's API usage limits (free tier: 60 calls/minute)
Icon Loading Issues:
Ensure internet connectivity
If icons fail to load, the app uses a fallback emoji (🌤️)
City Not Found:
Use full city names (e.g., "Los Angeles" instead of "LA")
Add country code for ambiguous cities (e.g., "Paris, FR" vs "Paris, TX")
ModuleNotFoundError:
Run pip install requests pillow to install missing libraries
