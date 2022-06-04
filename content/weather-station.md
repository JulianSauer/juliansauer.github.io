---
title: "Weather Station"
weight: 1
header_menu: true
---
------------------------------------------------------------------------------------------------------
In 2021 I passed my AWS Certified Cloud Practitioner exam.
To get some hands-on expierence I've created a simply website which displays weather data from local readings as well as a forecast.
It can be found [here](https://wetter.julian-sauer.com/#/en/).

# Hardware setup

I've bought [this Outdoor Weather Station](https://www.tinkerforge.com/en/shop/outdoor-weather-station-ws-6147.html) from Tinkerforge as well as their [Outdoor Weather Bricklet](https://www.tinkerforge.com/en/shop/outdoor-weather-bricklet.html) and a [HAT Zero Brick](https://www.tinkerforge.com/en/shop/bricks/hat-zero-brick.html) to read the sensor data using a Raspberry Pi Zero which I had lying around.
The data consists of wind speed and direction, rain, temperature and humidity.
Finding a good spot for the station was actually a bigger problem than I had initially thought because there's always a tree, a house or a garage blocking the wind.
That's what I've come up with:
{{< figure src="/images/weather-station/outdoor-weather-station.jpg" >}}

I've setup the Raspberry Pi in a closet so that nobody trips on the power cable.
Here's a photo of my first test connecting everything.
The Raspberry Pi 2 in this picture was later replaced by a Raspberry Pi Zero W.
{{< figure src="/images/weather-station/raspberry-pi.jpg" >}}

------------------------------------------------------------------------------------------------------

# Development
Now for the software part: It might be a bit overengineered but the goal was to try out a few services of the AWS platform.
I've setup everything using Terraform.
Here's a quick diagram of the architecture:
{{< figure src="/images/weather-station/weather-station.png" >}}
The weather station and the Raspberry Pi can be found on the right side.
A cron job reads the sensor data every 15 minutes and sends it to a topic of the Simple Notification Service.
This triggers a Lambda function which persists the data in a DynamoDB table.

Another cron job checks the battery of the weather station on a daily basis.
If it's low a message will be send to a different topic which triggers an e-mail notification.
Besides the weather station's data there's also an hourly and a daily forecast displayed on the website.
I am using the API of [Tomorrow.io](https://www.tomorrow.io/weather-api/) because it has a free tier.
The website could load the forecast from Tomorrow.io directly however that might exceed the free limit.
That's the one reason why I trigger a Lambda function using EventBridge to send the forecast data regularly to the same topic as the sensor data.
But besides caching the current forecast this also stores a forecast history.
In the future I'd like to add the possibility to compare forecasts with the actual sensor readings showing how reliable the forecasts are.
I've also thought about integrating additional forecast providers and see which one works best.

The website is hosted on S3 as a static website and acquires it's data using a serverless API.
An API Gateway routes the requests to different Lambda functions which search the database.
{{< figure src="/images/weather-station/weather-station-website.png" >}}

The tech stack isn't very exciting besides the cloud services.
I've mostly used Golang with the AWS Go SDK as well as the Go library from Tinkerforge for connecting the Pi with the weather station.
The code base can be found on Github [(see below)]({{< ref "#source-code" >}}) where different pipelines compile the code.
The resulting artifacts are getting uploaded to S3 and deployed to Lambda or - in case of the Raspberry Pi - will be downloaded manually by me and thrown onto the Pi using SSH.
Having to update the service on the Pi manually isn't perfect but I couldn't find a proper Github Action for that problem.
Instead I tried solving most concerns within AWS which worked so far.
I've had to update the service on the Raspberry Pi only once.

The website was build using VueJS with [Chart.js](https://www.chartjs.org/).
I've used Typescript although VueJS seems to prefer Javascript...at least that was my impression when looking at their documentation.
For mocking the API during local development I've used a simple Python script.
The deployment is again done automatically using Github Actions.

As a conclusion I can say that I've definitly learned a lot.
I'm mostly a backend developer in my day-to-day work so creating a website from scratch was challenging.
It was also the first time that I've setup infrastructure using Terraform.
That took me a while but when I moved the entire cloud infrastructure to a different account I was very glad because that was a breeze.
I was able to recreate everyhting in the new account and then tearing down the services in the old one with just a few terminal commands.
Now for the actual services of AWS: I think that I do have a much better understanding of what some of them can or cannot do.
For example using the Secrets Manager instead of the Parameter Store for storing secrets like my Tomorrow.io API seemed intuitive to me but also unnecessary.
The Parameter Store would've been sufficient - a mistake that costs me a few cents per month til this day.
But I was generally impressed by the free tiers within AWS.
Especially the Simple Storage Service is now my goto solution for anything data related or for hosting websites.

### Source code {#source-code}
Here are the links to the source code:
- [Weather Station Sensor](https://github.com/JulianSauer/Weather-Station-Sensor): Pushes the sensor data into the cloud.
- [Weather Station Data Serializer](https://github.com/JulianSauer/Weather-Station-Data-Serializer): Persists sensor and forecast data in the database.
- [Weather Station API](https://github.com/JulianSauer/Weather-Station-API): Can be used to query the persisted data. It also contains a function that caches the forecasts.
- [Weather Station UI](https://github.com/JulianSauer/Weather-Station-UI): Contains the website for the visual representation.
- [Weather Station Infrastructure](https://github.com/JulianSauer/Weather-Station-Infrastructure): Represents and manages the infrastructure in AWS.
