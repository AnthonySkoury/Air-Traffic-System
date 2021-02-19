# SDR-Aircraft-Traffic-Application
An air traffic system that uses an ADS-B receiver to obtain data.

## Disclaimer
Please refer to here for more info on [ADS-B](https://www.faa.gov/nextgen/programs/adsb/faq/). Note that this system uses just receiving capabilities of software defined radios. It is illegal to transmit with a software defined radio without certain licenses, especially pertaining aircraft. Please refer [here](https://www.fcc.gov/wireless/bureau-divisions/mobility-division/amateur-radio-service) to learn about licenses that allow transmitting with radios.

## Note
There are two methods of installation, with script (beta) and manual (tested). For more information on using each component or for troubleshooting, refer the manual installation sections and the README for each component.

## Hardware Tested
### Host Device
* Raspberry Pi 3 Model B+
* Raspberry Pi 4 8GB (Plan to test)
### Software Defined Radio
* [RTL-SDR RTL2832U](https://www.amazon.com/RTL-SDR-Blog-RTL2832U-Software-Defined/dp/B0129EBDS2/ref=sr_1_4?dchild=1&keywords=rtl+sdr&qid=1613098817&sr=8-4)
* HackRF (Plan to test)
* PlutoSDR (Plan to test)

# General Usage Setup
## Prerequisites
* RTL SDR
* Raspberry Pi with Raspbian
  * To set up Raspberry Pi: 
    * Install [Raspberry Pi Imager](https://www.raspberrypi.org/software/) 
    * Follow this guide to flash Raspberry Pi OS onto your SD card: https://www.raspberrypi.org/documentation/installation/installing-images/
    * Once the image is installed run the following commands
    ```bash
    sudo apt-get update
    sudo apt-get install build-essential git -y
    ```

## Installation using script
First [clone the repository](https://help.github.com/en/articles/cloning-a-repository) via Git using the following command and change into the directory
```bash
git clone https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application.git
cd SDR-Aircraft-Traffic-Application
```

Please use the files install.sh and librtlsdr.pc
In the same directory with install.sh or in the root directory of the cloned repository (SDR-Aircraft-Traffic-Application) run the following command:
```bash
sudo echo y | sudo ./install.sh |& sudo tee -a installation.log
```
Is issues persist with Dump1090 portion of install, refer to fixes in the section below.
**After installation, dependencies for the virtual environment for the backend must be synchronized by the user with these steps:**
* Ensure you are in the root directory for the repo, SDR-Aircraft-Traffic-Application then run:
```bash
pipenv shell
```
* Optional step if error occurs. If directory issues exist when running pip install pipenv, modify ~/.bashrc with the following command
```bash
echo 'export PATH="${HOME}/.local/bin:$PATH"' >> ~/.bashrc
```

Install the requirements from the Pipfile:
```bash
pipenv sync
```
When using the backend you must remain in the pipenv shell so that the virtual environment dependencies are registered. Change directories into the backend to access the Django manager manage.py

```bash
cd backend/
```

Create the database:

```bash
python manage.py migrate
```
### Starting each component
To get an API key for the frontend (required for Google maps portion of frontend) refer to the following instructions [Obtaining a Google Maps API Key](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application#starting-the-frontend). To run the decoder and backend refer to these [instructions](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application#running-the-decoder-and-backend). To start the frontend component refer to these [instructions](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application#starting-the-frontend). For more information, refer to  the README of each component (in the directory of each).

## Manual Installation

First [clone the repository](https://help.github.com/en/articles/cloning-a-repository) via Git using the following command in your terminal
```bash
git clone https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application.git
```

### Setting Up Docker
* If you do not have Docker on your machine please follow Docker (for manual installation follow these steps)
  * Docker can be set up on the Raspberry Pi or on another machine.
    * [To install on Raspberry Pi](https://docs.docker.com/engine/install/debian/)
      * The simplest way is to use the convenience script as follows:
        ```bash
        curl -fsSL https://get.docker.com -o get-docker.sh
        sudo sh get-docker.sh
        ```
    * [To install on Mac/Windows/Linux PC](https://docs.docker.com/get-docker/)

### Setting up RTL-SDR and Dump1090
* Install rtl-sdr lib using `sudo apt-get install rtl-sdr librtlsdr-dev`
* cd into dump1090 directory `cd SDR-Aircraft-Traffic-Application/decoder/RTL-SDR/dump1090/`
* `make`
* If issues persist with Makefile follow the steps [here](https://github.com/antirez/dump1090/issues/142), in summary:
  * Run `pkg-config --libs librtlsdr --debug` to find the path of `librtlsdr.pc` on Raspbian it is most likely `/usr/lib/arm-linux-gnueabihf/pkgconfig/librtlsdr.pc` and on Linux it is most likely `/usr/lib/x86_64-linux-gnu/pkgconfig/librtlsdr.pc`
  * Change the file to to following:
  
    ```bash
    prefix=/usr
    exec_prefix=${prefix}
    libdir=${exec_prefix}/lib
    includedir=${prefix}/include
    ```
* Run `./dump1090 --interactive --net` inside the dump1090 directory to start retrieving ADS-B data

### Setting up the database and backend

**Setting up and running the Database**
* To work with the Django backend, first a PostgreSQL database is required. The recommended approach would be to use a PostgreSQL Docker Container.

To run a PostgreSQL Container in Docker, the image needs to be installed.

[PostgreSQL Image](https://hub.docker.com/_/postgres)

The following command can be used to set up the Docker Container. Note that the password can be changed but the name and port should stay consistent as they are the expected ones in the Django database settings. Django will use those to connect to the database.

```bash
docker run --name aircraft_db -e POSTGRES_USER=aircraft_db -e POSTGRES_DB=aircraft_db -e POSTGRES_PASSWORD=raspberry -d -p 5432:5432 postgres
```

**Setting up the backend**
* cd into the SDR-Aircraft-Traffic-Application directory
`cd SDR-Aircraft-Traffic-Application/`
 
Get set up with the virtual environment for dependencies:
```bash
echo 'export PATH="${HOME}/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
pip3 install --user --default-timeout=1000 pipenv
pipenv shell
```
* if directory issues exist when running pip install pipenv, modify ~/.bashrc with the line
* export PATH="/home/pi/.local/bin:$PATH"
* at the end of the file


Install the requirements from the Pipfile:

```bash
sudo apt-get install libpq-dev python-dev
pipenv sync
```
Change directories into the backend to access the Django manager manage.py

```bash
cd SDR-Aircraft-Traffic-Application/backend/
```

Create the database:

```bash
python manage.py migrate
```

Run the development server:

```bash
python manage.py runserver
```

Backend located at **127.0.0.1:8000** or http://localhost:8000 .
* For more information, please read the backend README [here](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application/blob/main/backend/README.md)

### Setting up the frontend
**Installing Dependencies**

npm, Node.js, and the create-react-app is required to run the web app

Install Node.js and npm: https://www.npmjs.com/get-npm

Run `npx create-react-app my-app` to install React

`cd SDR-Aircraft-Traffic-Application/frontend`

To install the required dependencies, run `npm install`

If needed, install the Google Maps API with `npm install google-map-react`

**Obtaining a Google Maps API Key**

Go to the [Google Cloud Platform Console](https://console.cloud.google.com/apis/credentials?authuser=0&_ga=2.126357261.1224200343.1612121726-26711352.1612121726) and create a new project

You may have to create a new account and link your credit card to access the free tier. Google provides $200 of monthly credit and it only costs @2 per 1000 requests.

In the credentials tab, click on the "Create Credentials" button and select "API Key"

In the `frontend/src/map/Map.js` file, add the API key to the value of key in `bootstrapURLKeys={{ key: '' }}`.

For more information on getting a Google Maps API key, please refer to the Google docs [here](https://developers.google.com/maps/documentation/javascript/get-api-key)

* For more information, please read the frontend README [here](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application/blob/main/frontend/README.md)

### Running the decoder and backend
Start the virtual environment

`cd SDR-Aircraft-Traffic-Application`

`pipenv shell`

Run the development server:

Change directory to SDR-Aircraft-Traffic-Application/backend
```bash
python manage.py runserver
```

Change directory to SDR-Aircraft-Traffic-Application/decoder/RTL-SDR/dump1090

Run `./dump1090 --interactive --net` to start dump1090

In another window, change directory to SDR-Aircraft-Traffic-Application/decoder

Run `python3 data_acquisition.py` to start the decoder (uses a python3 library)

ADS-B data should be recorded in the Django database

You can view this in at 127.0.0.1:8000 or http://localhost:8000 in your browser

In order to allow the backend to listen on other devices in your network:
`python manage.py runserver 0.0.0.0:8000`
You can view the database by accessing the address of your http://{$RASPBERRYPI_IP_ADDRESS}:8000/

Similarly, the line below in AircraftData.js in /frontend/src/aircraft_data/ needs to be changed to the IP address of your Raspberry Pi
`const res = await fetch("http://{$RASPBERRYPI_IP_ADDRESS}:8000/api/aircraftdata/")`

If neither of these links work to view the pages from another device, it is possible there is a firewall issue in your network. However, the links are still accessible on the Raspberry Pi itself, with the option of using localhost instead of the IP address on your Raspberry Pi's browser.

### Starting the frontend
In a new window change directory to SDR-Aircraft-Traffic-Application/frontend

To start the web app, use `npm start` and it should be located on localhost:3000

* if issues arise, check the docker
* in the backend directory, run "sudo docker ps"
* if need be, run "sudo docker restart aircraft_db" in the backend directory

## System Design

### System Architecture
Note that the database server in this system is in a Docker Container by default, though the server can be hosted any way as long as it's a PostgreSQL database with the default ports.
![System Diagram](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application/blob/main/images/system_diagram_high_level.png?raw=true)

### Database Design
The following is the database design which would be the details of the PostgreSQL database in the database server. These are the variables and attributes stored as well as their relationship. The data records have a foreign key referencing an aircraft in the aircraft table.
![Database Diagram](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application/blob/main/images/db_diagram.png?raw=true)


## Development

### Tools Used
* [Git](https://git-scm.com/) for version control
* [Django](https://www.djangoproject.com/) for backend
* [React](https://reactjs.org/) for frontend
* [Docker](https://www.docker.com/) for database host
* [PostgreSQL](https://www.postgresql.org/) for database used

Knowledge and skills needed for developing

* Programming Python, Javascript, HTML, CSS to some extent
* General understanding of REST API framework
* General Understanding of how SDRs work

## Contributing

Please contact us or Peter Burke if you are interested in taking this project further.

## Versioning

We use [Git](https://git-scm.com/) for versioning.

## Authors

* **Anthony Skoury** - *Computer Engineer* - [My Website](https://anthonyskoury.github.io/)
* **Randall Cheng** - *Electrical Engineer, Computer Engineer*
* **Alan Wong**

## License
This project is licensed under the APGL_v3 License - see the [LICENSE.md](https://github.com/AnthonySkoury/SDR-Aircraft-Traffic-Application/blob/main/LICENSE) file for details


## Acknowledgments

* Professor Peter Burke for advising us
* Authors for all the Open-Source Libraries used in this program
* Anyone who develops this project further
