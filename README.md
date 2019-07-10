# Art in the Age of Machine Learning
### *Predictive Analytics for Restaurants*

**Project Scope**

The restaurant industry operates with notoriously thin operating margins, typically dropping around 5% of total revenue to their bottom lines. For every $1.00 in sales, approximately $0.30 - $0.35 will likely go to cost of goods (e.g. raw ingredients, dry goods, alcohol) while an additional $0.30 - $0.35 will go to labor, leaving roughly $0.30 - $0.40 to cover overhead (rent, insurance, utilities, equipment rentals, waste management, accounting, marketing) and provide a return on capital, to the extent there is any excess after covering all the costs.

An accurate sales forecast provides several avenues of cost saving + revenue expansion to restaurants:

* **Labor Optimization** | Minimize labor hours on slow nights while maintaining appropriate staffing levels on busier nights.
* **Minimize Food Waste** | More precise daily prep levels to help limit food waste.
* **Lean Inventory** | Inventory is a drag on cash flow and high levels of inventory of fresh goods can lead to spoilage and food waste.
* **Marketing Initiatives** | Maximize marketing dollars by targeting slow nights and weeks.

Using real sales data from a two-star restaurant located in Brooklyn, NY, this is a pilot project exploring what an end-to-end restaurant sales forecasting tool utilizing machine learning would entail.

**Data Sources**
* **Restaurant Sales Data** | Restaurant sales and guest counts ("covers") were downloaded directly from the restuarant's Point of Sale ("POS") on a check-by-check basis from 1/1/2017 through 06/30/2019, then aggregated into nightly totals covering the Dinner period only.

* **Weather Data** | Weather data was accessed via the DarkSky API. The source code for the API call can be found [here](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/weather.py "weather.py"). The latitude and longitude for the restuarant, required to access the weather data, is accessed via the Yelp API - the source code for this API call is found [here](https://github.com/maks-p/restaurant_sales_forecasting/blob/master/restaurant_info.py "restaurant_info.py").

**Restaurant Background**
The subject is an award-winning, two-start restaurant located in Brooklyn, NY. The restaurant has a patio that adds a substantial amount of seats. The restaurant is performing extremely well, earning $5.8 MM in net sales in 2018 (dinner service only):

![sales](https://user-images.githubusercontent.com/42282874/60993726-05e84d00-a31d-11e9-8924-89b91ea62c25.png)
![covers](https://user-images.githubusercontent.com/42282874/60993725-05e84d00-a31d-11e9-8b7f-eff9ede75f09.png)

