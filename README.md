import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

public class WeatherApp {

    private static final String API_URL = "https://samples.openweathermap.org/data/2.5/forecast/hourly?q=London,uk&appid=b6907d289e10d714a6e88b30761fae22";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        String weatherData = fetchWeatherData();
        if (weatherData == null) {
            System.out.println("Failed to fetch weather data.");
            scanner.close();
            return;
        }

        while (true) {
            System.out.println("1. Get weather");
            System.out.println("2. Get Wind Speed");
            System.out.println("3. Get Pressure");
            System.out.println("0. Exit");

            System.out.print("Enter your choice: ");
            int choice = scanner.nextInt();
            scanner.nextLine(); // Consume newline left-over from previous nextInt()

            switch (choice) {
                case 1:
                    System.out.print("Enter the date (yyyy-MM-dd HH:mm:ss): ");
                    String date1 = scanner.nextLine();
                    double temperature = getTemperatureByDate(weatherData, date1);
                    if (temperature != Double.MIN_VALUE) {
                        System.out.printf("Temperature on %s: %.2f K\n", date1, temperature);
                    } else {
                        System.out.println("No data available for the given date.");
                    }
                    break;
                case 2:
                    System.out.print("Enter the date (yyyy-MM-dd HH:mm:ss): ");
                    String date2 = scanner.nextLine();
                    double windSpeed = getWindSpeedByDate(weatherData, date2);
                    if (windSpeed != Double.MIN_VALUE) {
                        System.out.printf("Wind Speed on %s: %.2f m/s\n", date2, windSpeed);
                    } else {
                        System.out.println("No data available for the given date.");
                    }
                    break;
                case 3:
                    System.out.print("Enter the date (yyyy-MM-dd HH:mm:ss): ");
                    String date3 = scanner.nextLine();
                    double pressure = getPressureByDate(weatherData, date3);
                    if (pressure != Double.MIN_VALUE) {
                        System.out.printf("Pressure on %s: %.2f hPa\n", date3, pressure);
                    } else {
                        System.out.println("No data available for the given date.");
                    }
                    break;
                case 0:
                    System.out.println("Exiting the program.");
                    scanner.close();
                    return;
                default:
                    System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private static String fetchWeatherData() {
        try {
            URL url = new URL(API_URL);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");

            if (connection.getResponseCode() == 200) {
                InputStream inputStream = connection.getInputStream();
                Scanner scanner = new Scanner(inputStream, StandardCharsets.UTF_8);
                StringBuilder response = new StringBuilder();
                while (scanner.hasNextLine()) {
                    response.append(scanner.nextLine());
                }
                scanner.close();
                return response.toString();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private static double getTemperatureByDate(String weatherData, String date) {
        String[] dates = weatherData.split("\"dt_txt\":\"");
        for (String forecast : dates) {
            int endIndex = forecast.indexOf("\",");
            String forecastDate = forecast.substring(0, endIndex);
            if (forecastDate.contains(date)) {
                int tempIndex = forecast.indexOf("\"temp\":") + "\"temp\":".length();
                int commaIndex = forecast.indexOf(",", tempIndex);
                return Double.parseDouble(forecast.substring(tempIndex, commaIndex));
            }
        }
        System.out.println(Double.MIN_VALUE);
        return Double.MIN_VALUE;
    }

    private static double getWindSpeedByDate(String weatherData, String date) {
        String[] dates = weatherData.split("\"dt_txt\":\"");
        for (String forecast : dates) {
            int endIndex = forecast.indexOf("\",");
            String forecastDate = forecast.substring(0, endIndex);
            if (forecastDate.contains(date)) {
                int windIndex = forecast.indexOf("\"speed\":") + "\"speed\":".length();
                int commaIndex = forecast.indexOf(",", windIndex);
                return Double.parseDouble(forecast.substring(windIndex, commaIndex));
            }
        }
        return Double.MIN_VALUE;
    }

    private static double getPressureByDate(String weatherData, String date) {
        String[] dates = weatherData.split("\"dt_txt\":\"");
        for (String forecast : dates) {
            int endIndex = forecast.indexOf("\",");
            String forecastDate = forecast.substring(0, endIndex);
            if (forecastDate.contains(date)) {
                int pressureIndex = forecast.indexOf("\"pressure\":") + "\"pressure\":".length();
                int commaIndex = forecast.indexOf(",", pressureIndex);
                return Double.parseDouble(forecast.substring(pressureIndex, commaIndex));
            }
        }
        return Double.MIN_VALUE;
    }
}
