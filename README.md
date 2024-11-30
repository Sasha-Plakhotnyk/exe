import requests
from bs4 import BeautifulSoup
import sqlite3
import logging
from datetime import datetime

# Налаштування логування
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Клас DateWeather
class DateWeather:
    def __init__(self, date, temperature, precipitation, wind_speed, wind_direction):
        self.date = date
        self.temperature = temperature
        self.precipitation = precipitation
        self.wind_speed = wind_speed
        self.wind_direction = wind_direction

    def __repr__(self):
        return (f"DateWeather(date={self.date}, temperature={self.temperature}, "
                f"precipitation={self.precipitation}, wind_speed={self.wind_speed}, "
                f"wind_direction={self.wind_direction})")

# Функція для отримання даних з вебсайту
def fetch_weather_data(url, days=7):
    logging.info("Fetching weather data from the website...")
    response = requests.get(url)
    if response.status_code != 200:
        logging.error("Failed to fetch data from the website.")
        raise Exception(f"HTTP Error: {response.status_code}")
    soup = BeautifulSoup(response.text, 'html.parser')
    weather_data = []

    # Парсинг даних
    # Цей блок потрібно адаптувати під структуру конкретного сайту!
    for i in range(days):
        date = (datetime.now().date() + timedelta(days=i)).isoformat()
        temperature = f"{15 + i}°C"  # Замінити на реальні дані
        precipitation = "Так" if i % 2 == 0 else "Ні"  # Замінити на реальні дані
        wind_speed = f"{5 + i} м/с"  # Замінити на реальні дані
        wind_direction = "Північ" if i % 2 == 0 else "Південь"  # Замінити на реальні дані

        weather_data.append({
            "date": date,
            "temperature": temperature,
            "precipitation": precipitation,
            "wind_speed": wind_speed,
            "wind_direction": wind_direction,
        })
    logging.info("Weather data fetched successfully.")
    return weather_data

# Функція для створення бази даних і таблиці
def create_database():
    logging.info("Creating SQLite3 database...")
    connection = sqlite3.connect('weather.db')
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS weather (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            date TEXT NOT NULL,
            temperature TEXT NOT NULL,
            precipitation TEXT NOT NULL,
            wind_speed TEXT NOT NULL,
            wind_direction TEXT NOT NULL
        )
    ''')
    connection.commit()
    connection.close()
    logging.info("Database created successfully.")

# Функція для запису даних у базу
def insert_weather_data(data):
    logging.info("Inserting data into SQLite3 database...")
    connection = sqlite3.connect('weather.db')
    cursor = connection.cursor()
    for entry in data:
        cursor.execute('''
            INSERT INTO weather (date, temperature, precipitation, wind_speed, wind_direction)
            VALUES (?, ?, ?, ?, ?)
        ''', (entry['date'], entry['temperature'], entry['precipitation'], entry['wind_speed'], entry['wind_direction']))
    connection.commit()
    connection.close()
    logging.info("Data inserted successfully.")

# Функція для вибірки даних з бази
def fetch_data_from_database(condition, value):
    logging.info(f"Fetching data from database with condition: {condition} = {value}...")
    connection = sqlite3.connect('weather.db')
    cursor = connection.cursor()
    query = f"SELECT * FROM weather WHERE {condition} = ?"
    cursor.execute(query, (value,))
    rows = cursor.fetchall()
    connection.close()
    return [DateWeather(*row[1:]) for row in rows]

# Основний скрипт
if __name__ == "__main__":
    try:
        # 1. Отримання даних з вебсайту
        url = "https://example.com/weather"  # Замінити на реальний сайт
        weather_data = fetch_weather_data(url)

        # 2. Створення бази даних і таблиці
        create_database()

        # 3. Запис даних у базу
        insert_weather_data(weather_data)

        # 4. Вибірка даних
        selected_data = fetch_data_from_database("date", "2024-12-01")
        print("Selected Data:", selected_data)

    except Exception as e:
        logging.error(f"An error occurred: {e}")
