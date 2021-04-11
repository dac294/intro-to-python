# Checkpoint 4: Submitting Data from Web Forms

Let's create a web form with a text input element to allow the user to specify their given zip code. In the templates directory, add a new HTML file called "weather_form.html" and place inside the following contents:

```html
{% extends "layout.html" %}

{% block content %}

    <h2>Weather Form</h2>

    <p>Request an hourly forecast for your zip code...</p>

    <form action="/weather/forecast" method="POST">

        <label>Country Code:</label>
        <input type="text" name="country_code" placeholder="US" value="US">
        <br>

        <label>Zip Code:</label>
        <input type="text" name="zip_code" placeholder="20057" value="20057">
        <br>

        <button>Submit</button>
    </form>

{% endblock %}
```

Here we're saying when the user submits the form, we'll send their form inputs via POST request to a route called "/weather/forecast". So let's update our weather routes to handle the POST request:

```py

# web_app/routes/weather_routes.py

from flask import Blueprint, request, jsonify, render_template

from app.weather_service import get_hourly_forecasts

weather_routes = Blueprint("weather_routes", __name__)

@weather_routes.route("/weather/form")
def weather_form():
    print("WEATHER FORM...")
    return render_template("weather_form.html")

@weather_routes.route("/weather/forecast", methods=["GET", "POST"])
@weather_routes.route("/weather/forecast.json", methods=["GET", "POST"])
def weather_forecast():
    print("WEATHER FORECAST...")

    if request.method == "GET":
        print("URL PARAMS:", dict(request.args))
        request_data = request.args
    elif request.method == "POST": # the form will send a POST
        print("FORM DATA:", dict(request.form))
        request_data = request.form

    country_code = request_data.get("country_code") or "US"
    zip_code = request_data.get("zip_code") or "20057"

    results = get_hourly_forecasts(country_code=country_code, zip_code=zip_code)
    print(results.keys())

    if ".json" in request.url:
        return jsonify(results)
    else:
        return render_template("weather_forecast.html", country_code=country_code, zip_code=zip_code, results=results)

```

After we get a weather forecast for the given zip code, we'll send the results to a new page called "weather_forecast.html", so let's create that page now in the "templates" directory, and place inside the following contents:

```html
{% extends "layout.html" %}

{% block content %}

    <h2>Weather Forecast for {{ results["city_name"] }}</h2>

    <p>Zip Code: {{ zip_code }}</p>

    <!-- TODO: consider using a table instead of a list -->
    <!-- TODO: consider adding images using the provided icon urls -->
    <ul>
    {% for hourly in results["hourly_forecasts"] %}
        <li>{{ hourly["timestamp"] }} | {{ hourly["temp"] }} | {{ hourly["conditions"].upper() }}</li>
    {% endfor %}
    </ul>

{% endblock %}
```

Here, we are using the Jinja template language to loop through our forecasts and display each.

Restart the server and visit http://localhost:5000/weather/form to test the newly-integrated weather forecasting functionality.