# Visuo server

This is the source code of a simplified version of the server developed for the [VISUO project](http://vintage.winklerbros.net/clouds.html) between the [Nanyang Technological University](http://www.ntu.edu.sg), the [Advanced Digital Sciences Center](http://adsc.illinois.edu/) and the [National Metrology Centre](https://www.a-star.edu.sg/nmc/) (Singapore). It collects and stores data from sky imagers, weather stations and radiosondes, and provides a web interface for visualizing and downloading the data behind a login mechanism. It is developed in python using the [django web framework](https://www.djangoproject.com/). Visualizations are coded in javascript using [d3.js](https://d3js.org/) and the [Google Maps JavaScript Api](https://developers.google.com/maps/documentation/javascript/).

A sample deployment with two days of data (28th and 29th of October 2015) is provided here (username: `demo`, password: `demodemo`): http://visuo.pythonanywhere.com/

![alt text](https://github.com/FSavoy/visuo-server/raw/master/common-static/img/screenshot.png "Interface screenshot")

## Web interface

The interface is composed of three pages:
- **Data visualization**: plots graphs of weather and radiosonde data on a daily basis and interactively plots the sky pictures captured throughout that day on a map.
- **Sky pictures**: filters and lists images captured by the sky imagers.
- **Data downloads**: filters and used to download weather or radiosonde data in excel or matlab formats.

The django admin console (`[base url]/admin`) is also available for the admin user to manage users and the database.

## Organization of the source code

The source code is organized in several django apps:
- **accounts**: extends the djando admin login system to provides login pages for the web application
- **api**: REST apis for uploading data (sky pictures and weather measurements)
- **data**: contains the models for creating the database for all devices and measurements
- **downloads**: generates the download page
- **picture_browser**: generates the sky images browsing page
- **visualization**: generates the interactive data visualization page

The required packages are listed under `requirements.txt`.

## Installation

The source code should be deployed as a standard django web app (including initializing the database, creating the superuser, serving static and media files, ...). Please refer to the [official documentation](https://docs.djangoproject.com/en/1.11/howto/deployment/). Choose database and media files storage strategies which scale with the amount of data that the app should store.

The following steps are also required:
- We use [django-bower](https://django-bower.readthedocs.io/en/latest/) to manage javascript packages. Run `./manage.py bower install` to download the dependencies.
- [Get a Google Maps Api key](https://developers.google.com/maps/documentation/javascript/get-api-key) and include it in the `visuo-open-source/settings.py` file.
- We use [Alligator](http://alligator.readthedocs.io/en/latest/index.html) to deal with offline tasks. Choose a queue backend and configure it according to http://alligator.readthedocs.io/en/latest/installing.html.

The superuser might use the django admin console at (`[base url]/admin`) to create multiple accounts for several users.

## Adding devices

Devices can upload their latest measurements to the server in real time over the internet using a REST api. Devices should be declared to the server via the admin console. First, create a new user under the *Authentication and authorization* section. Provide a username and a password for the device user account. Then add a token for the user account that you just created in the *Auth Token* section. It will be used to authenticate the device. Then add a new *Measuring device* instance under the *Data* section. Choose the device type (Whole Sky Imager, Weather Station or Radiosonde), provide a name and the location, and select the username you just created.

### Sky imagers

Images captured by sky imagers can be imported in two different ways:
- A batch upload function can be executed on the server itself by calling `.\manage.py import_images --folder [the local folder where the images to be imported are stored] --device [name of the sky imager as created above]`.
- A REST api at `[base url]/api/skypicture/` can be used to upload an image via a POST. The request needs to be authenticated, either using the token or the user account of the respective sky imager. A sample script which runs on a single board computer in our custom-made whole sky imagers is provided at `scripts/send_sky_imager.py`. It controls the camera, captures images at regular intervals using the *gphoto* library and sends them to the server using the REST api.

### Weather stations

We use [Davis Vantage Pro 2](http://www.davisnet.com/solution/vantage-pro2/) weather stations. They upload their data in similar ways:
- A script importing a (csv based) archive file generated by *WeatherLink* can be called with `.\manage.py import_weather --file [local path to the archive file] --device [name of the weather station as created above]`. The code (at `data\management\commands\import_weather.py`) could be modified to reflect different storage schemes.
- A REST api at `[base url]/api/weathermeasurement/` can be invoked to upload a single measurement via a POST. The request also needs to be authenticated by the respective weather station user account. A sample script which runs on the weather station PC is provided at `scripts\read_vantage_pro_2.py`. It reads the latest measurements from *WeatherLink* (.wlk binary files) in a continuous loop and sends them to the server.

### Radiosondes

Importing radiosonde measurements follows a different approach. We use the [database maintained by the University of Wyoming](http://weather.uwyo.edu/upperair/sounding.html) to gather the data. This project only supports a single radiosonde launch location, which should be informed by writing the station number into the `visuo-open-source/settings.py` file. Executing `.\manage.py download_radiosonde --start-date [YYYY-mm-dd] --end-date [YYYY-mm-dd]` downloads the data between the provided dates. Calling the same script without arguments (`.\manage.py download_radiosonde`) downloads the latest data available. Use a *cron* job on the server to invoke this script twice a day in order to automatically download the measurements for all new radiosonde launches.

## License

This software is released under a 3-Clause BSD License. It is provided “as is” without any warranty. Users are required to obtain the licenses for the required dependencies.
