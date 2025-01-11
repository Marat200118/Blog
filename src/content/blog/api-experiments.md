---
title: 'First experiments with APIs'
description: 'First experiments with APIs'
pubDate: 'Jan 11 2025'
heroImage: '/blog-placeholder-api-exp.png'
---

### First steps with FlightAware AeroAPI

So after defining the API stack, and the logic of the app, I started to experiment with the APIs. First task for me was to make a form for the user to add the flight. We need to get the flight number and the departure date from the user, and then send a request to the FlightAware AeroAPI to get the flight data. Firstly, I tried to connect the API directly in the Ionic app, but I faced the CORS issue. So I decided to create a simple Node.js server that will make the request to the API and then send the data to the app.

To test the API, I used the flight number "FR8448" and the departure date "2025-01-11". Which is flight from Riga to Stocholm. As soon as API can receive the flight number in numeric format, I had to remove the "FR" prefix from the flight number.

Here is the code of the server:

```javascript
const express = require("express");
const axios = require("axios");
const cors = require("cors");

const app = express();
const PORT = 3000;
app.use(cors());

app.get("/api/schedules", async (req, res) => {
  const { dateStart, dateEnd, flightNumber } = req.query;

  if (!dateStart || !dateEnd || !flightNumber) {
    return res.status(400).json({ error: "All parameters are required." });
  }

  const apiKey = "wHf94IBGL2dxGFS13wlB5sbGS34bBfT3";
  const numericFlightNumber = flightNumber.replace(/^\D+/g, "");
  const url = `https://aeroapi.flightaware.com/aeroapi/schedules/${dateStart}/${dateEnd}?flight_number=${numericFlightNumber}&airline=FR`;

  console.log("Making API Call to:", url);

  try {
    const response = await axios.get(url, {
      headers: { "x-apikey": apiKey },
    });

    console.log("Response from FlightAware API (Full):", response.data);

    const filteredFlights = response.data.scheduled.filter(
      (flight) =>
        flight.ident_iata?.includes(flightNumber) ||
        flight.actual_ident_iata?.includes(flightNumber)
    );

    console.log("Filtered Flights:", filteredFlights);
    res.json({ scheduled: filteredFlights });
  } catch (error) {
    console.error("Error Details:", error.response?.data || error.message);
    res.status(500).send("Error fetching flight data");
  }
});

app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});

```

After the first try, I got the response:

```json
{
  scheduled: [
    {
      ident: 'VOZ8448',
      ident_icao: 'VOZ8448',
      ident_iata: 'VA8448',
      actual_ident: 'UAL1785',
      actual_ident_icao: 'UAL1785',
      actual_ident_iata: 'UA1785',
      aircraft_type: 'A319',
      scheduled_in: '2025-01-10T03:56:00Z',
      scheduled_out: '2025-01-10T01:34:00Z',
      origin: 'KSLC',
      origin_icao: 'KSLC',
      origin_iata: 'SLC',
      origin_lid: 'SLC',
      destination: 'KSFO',
      destination_icao: 'KSFO',
      destination_iata: 'SFO',
      destination_lid: 'SFO',
      fa_flight_id: 'UAL1785-1736230512-fa-1580p',
      meal_service: 'Business: Snack or brunch, Food for sale / Economy: Food for sale',
      seats_cabin_business: 12,
      seats_cabin_coach: 114,
      seats_cabin_first: 0
    },
    {
      ident: 'AVA8448',
      ident_icao: 'AVA8448',
      ident_iata: 'AV8448',
      actual_ident: 'AVA8448',
      actual_ident_icao: 'AVA8448',
      actual_ident_iata: 'AV8448',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T04:33:00Z',
      scheduled_out: '2025-01-10T03:40:00Z',
      origin: 'SKCL',
      origin_icao: 'SKCL',
      origin_iata: 'CLO',
      origin_lid: null,
      destination: 'SKRG',
      destination_icao: 'SKRG',
      destination_iata: 'MDE',
      destination_lid: null,
      fa_flight_id: 'AVA8448-1736240230-airline-692p',
      meal_service: 'Economy: Food for sale',
      seats_cabin_business: 0,
      seats_cabin_coach: 180,
      seats_cabin_first: 0
    },
    {
      ident: 'QTR8448',
      ident_icao: 'QTR8448',
      ident_iata: 'QR8448',
      actual_ident: 'TAM3278',
      actual_ident_icao: 'TAM3278',
      actual_ident_iata: 'JJ3278',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T12:05:00Z',
      scheduled_out: '2025-01-10T10:30:00Z',
      origin: 'SBGR',
      origin_icao: 'SBGR',
      origin_iata: 'GRU',
      origin_lid: null,
      destination: 'SBCH',
      destination_icao: 'SBCH',
      destination_iata: 'XAP',
      destination_lid: null,
      fa_flight_id: 'TAM3278-1736321397-airline-4188p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 8,
      seats_cabin_coach: 168,
      seats_cabin_first: 0
    },
    {
      ident: 'LAN8448',
      ident_icao: 'LAN8448',
      ident_iata: 'LA8448',
      actual_ident: 'DAL1571',
      actual_ident_icao: 'DAL1571',
      actual_ident_iata: 'DL1571',
      aircraft_type: 'B712',
      scheduled_in: '2025-01-10T18:42:00Z',
      scheduled_out: '2025-01-10T17:04:00Z',
      origin: 'KSTL',
      origin_icao: 'KSTL',
      origin_iata: 'STL',
      origin_lid: 'STL',
      destination: 'KATL',
      destination_icao: 'KATL',
      destination_iata: 'ATL',
      destination_lid: 'ATL',
      fa_flight_id: 'DAL1571-1736313435-fa-694p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 12,
      seats_cabin_coach: 98,
      seats_cabin_first: 0
    },
    {
      ident: 'THY8448',
      ident_icao: 'THY8448',
      ident_iata: 'TK8448',
      actual_ident: 'AHY8042',
      actual_ident_icao: 'AHY8042',
      actual_ident_iata: 'J28042',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T12:45:00Z',
      scheduled_out: '2025-01-10T09:45:00Z',
      origin: 'LTBJ',
      origin_icao: 'LTBJ',
      origin_iata: 'ADB',
      origin_lid: null,
      destination: 'UBBB',
      destination_icao: 'UBBB',
      destination_iata: 'GYD',
      destination_lid: null,
      fa_flight_id: 'AHY8042-1736315901-airline-1121p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 12,
      seats_cabin_coach: 144,
      seats_cabin_first: 0
    },
    {
      ident: 'UAL8448',
      ident_icao: 'UAL8448',
      ident_iata: 'UA8448',
      actual_ident: 'PVL8343',
      actual_ident_icao: 'PVL8343',
      actual_ident_iata: 'PB8343',
      aircraft_type: 'DH8D',
      scheduled_in: '2025-01-10T22:30:00Z',
      scheduled_out: '2025-01-10T21:10:00Z',
      origin: 'CYVR',
      origin_icao: 'CYVR',
      origin_iata: 'YVR',
      origin_lid: null,
      destination: 'CYXS',
      destination_icao: 'CYXS',
      destination_iata: 'YXS',
      destination_lid: null,
      fa_flight_id: null,
      meal_service: 'Economy: No meal',
      seats_cabin_business: 0,
      seats_cabin_coach: 78,
      seats_cabin_first: 0
    },
    {
      ident: 'AAL8448',
      ident_icao: 'AAL8448',
      ident_iata: 'AA8448',
      actual_ident: 'JAL320',
      actual_ident_icao: 'JAL320',
      actual_ident_iata: 'JL320',
      aircraft_type: 'A359',
      scheduled_in: '2025-01-10T08:30:00Z',
      scheduled_out: '2025-01-10T06:55:00Z',
      origin: 'RJFF',
      origin_icao: 'RJFF',
      origin_iata: 'FUK',
      origin_lid: null,
      destination: 'RJTT',
      destination_icao: 'RJTT',
      destination_iata: 'HND',
      destination_lid: null,
      fa_flight_id: 'JAL320-1736319793-schedule-1444p',
      meal_service: 'First: Meal / Business: Meal / Economy: Meal',
      seats_cabin_business: 94,
      seats_cabin_coach: 263,
      seats_cabin_first: 12
    },
    {
      ident: 'VIR8448',
      ident_icao: 'VIR8448',
      ident_iata: 'VS8448',
      actual_ident: 'IGO2057',
      actual_ident_icao: 'IGO2057',
      actual_ident_iata: '6E2057',
      aircraft_type: 'A21N',
      scheduled_in: '2025-01-10T14:40:00Z',
      scheduled_out: '2025-01-10T12:30:00Z',
      origin: 'VIDP',
      origin_icao: 'VIDP',
      origin_iata: 'DEL',
      origin_lid: null,
      destination: 'VECC',
      destination_icao: 'VECC',
      destination_iata: 'CCU',
      destination_lid: null,
      fa_flight_id: 'IGO2057-1736320642-airline-1097p',
      meal_service: 'Economy: Food for sale',
      seats_cabin_business: 0,
      seats_cabin_coach: 232,
      seats_cabin_first: 0
    },
    {
      ident: 'SIA8448',
      ident_icao: 'SIA8448',
      ident_iata: 'SQ8448',
      actual_ident: 'TGW120',
      actual_ident_icao: 'TGW120',
      actual_ident_iata: 'TR120',
      aircraft_type: 'B788',
      scheduled_in: '2025-01-10T15:15:00Z',
      scheduled_out: '2025-01-10T10:20:00Z',
      origin: 'WSSS',
      origin_icao: 'WSSS',
      origin_iata: 'SIN',
      origin_lid: null,
      destination: 'ZHHH',
      destination_icao: 'ZHHH',
      destination_iata: 'WUH',
      destination_lid: null,
      fa_flight_id: 'TGW120-1736325626-airline-879p',
      meal_service: 'Economy: No meal',
      seats_cabin_business: 0,
      seats_cabin_coach: 329,
      seats_cabin_first: 0
    },
    {
      ident: 'CCA7922',
      ident_icao: 'CCA7922',
      ident_iata: 'CA7922',
      actual_ident: 'CSZ8448',
      actual_ident_icao: 'CSZ8448',
      actual_ident_iata: 'ZH8448',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T12:35:00Z',
      scheduled_out: '2025-01-10T11:20:00Z',
      origin: 'ZYTX',
      origin_icao: 'ZYTX',
      origin_iata: 'SHE',
      origin_lid: null,
      destination: 'ZSYT',
      destination_icao: 'ZSYT',
      destination_iata: 'YNT',
      destination_lid: null,
      fa_flight_id: 'CSZ8448-1736321771-airline-1533p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 8,
      seats_cabin_coach: 150,
      seats_cabin_first: 0
    },
    {
      ident: 'CSZ8448',
      ident_icao: 'CSZ8448',
      ident_iata: 'ZH8448',
      actual_ident: null,
      actual_ident_icao: null,
      actual_ident_iata: null,
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T12:35:00Z',
      scheduled_out: '2025-01-10T11:20:00Z',
      origin: 'ZYTX',
      origin_icao: 'ZYTX',
      origin_iata: 'SHE',
      origin_lid: null,
      destination: 'ZSYT',
      destination_icao: 'ZSYT',
      destination_iata: 'YNT',
      destination_lid: null,
      fa_flight_id: 'CSZ8448-1736321771-airline-1533p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 8,
      seats_cabin_coach: 150,
      seats_cabin_first: 0
    },
    {
      ident: 'CDG9448',
      ident_icao: 'CDG9448',
      ident_iata: 'SC9448',
      actual_ident: 'CSZ8448',
      actual_ident_icao: 'CSZ8448',
      actual_ident_iata: 'ZH8448',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T12:35:00Z',
      scheduled_out: '2025-01-10T11:20:00Z',
      origin: 'ZYTX',
      origin_icao: 'ZYTX',
      origin_iata: 'SHE',
      origin_lid: null,
      destination: 'ZSYT',
      destination_icao: 'ZSYT',
      destination_iata: 'YNT',
      destination_lid: null,
      fa_flight_id: 'CSZ8448-1736321771-airline-1533p',
      meal_service: 'Business: No meal / Economy: No meal',
      seats_cabin_business: 8,
      seats_cabin_coach: 150,
      seats_cabin_first: 0
    },
    {
      ident: 'CAL8448',
      ident_icao: 'CAL8448',
      ident_iata: 'CI8448',
      actual_ident: 'JAL168',
      actual_ident_icao: 'JAL168',
      actual_ident_iata: 'JL168',
      aircraft_type: 'E190',
      scheduled_in: '2025-01-10T13:00:00Z',
      scheduled_out: '2025-01-10T11:45:00Z',
      origin: 'RJSK',
      origin_icao: 'RJSK',
      origin_iata: 'AXT',
      origin_lid: null,
      destination: 'RJTT',
      destination_icao: 'RJTT',
      destination_iata: 'HND',
      destination_lid: null,
      fa_flight_id: 'JAL168-1736318285-airline-593p',
      meal_service: 'Business: Meal / Economy: Meal',
      seats_cabin_business: 15,
      seats_cabin_coach: 80,
      seats_cabin_first: 0
    },
    {
      ident: 'TAP8448',
      ident_icao: 'TAP8448',
      ident_iata: 'TP8448',
      actual_ident: 'SWR1228',
      actual_ident_icao: 'SWR1228',
      actual_ident_iata: 'LX1228',
      aircraft_type: 'BCS3',
      scheduled_in: '2025-01-10T22:25:00Z',
      scheduled_out: '2025-01-10T20:15:00Z',
      origin: 'LSZH',
      origin_icao: 'LSZH',
      origin_iata: 'ZRH',
      origin_lid: null,
      destination: 'ESGG',
      destination_icao: 'ESGG',
      destination_iata: 'GOT',
      destination_lid: null,
      fa_flight_id: 'SWR1228-1736319504-airline-627p',
      meal_service: 'Business: No meal / Economy: Food for sale',
      seats_cabin_business: 1,
      seats_cabin_coach: 144,
      seats_cabin_first: 0
    },
    {
      ident: 'CDG9448',
      ident_icao: 'CDG9448',
      ident_iata: 'SC9448',
      actual_ident: 'CSZ8448',
      actual_ident_icao: 'CSZ8448',
      actual_ident_iata: 'ZH8448',
      aircraft_type: 'A320',
      scheduled_in: '2025-01-10T16:40:00Z',
      scheduled_out: '2025-01-10T13:25:00Z',
      origin: 'ZSYT',
      origin_icao: 'ZSYT',
      origin_iata: 'YNT',
      origin_lid: null,
      destination: 'ZUTF',
      destination_icao: 'ZUTF',
      destination_iata: 'TFU',
      destination_lid: null,
      fa_flight_id: 'CSZ8448-1736321771-airline-1534p',
      meal_service: 'Business: Dinner / Economy: Dinner',
      seats_cabin_business: 8,
      seats_cabin_coach: 150,
      seats_cabin_first: 0
    }
  ],
  links: {
    next: '/schedules/2025-01-10/2025-01-11?flight_number=8448&cursor=cc99f36ae341026'
  },
  num_pages: 1
}
```

And also it added to my home screen all the flights with the number "8448".

![Full API response](/full-response.png)

How we can see from the response, we got the flight data for the flight number "8448". But we need to filter the flights by the flight number, because the API can return the flights with the same number but different airlines. So I filtered the flights by the flight number and the airline code. After that, I got the flight data for the flight number "FR8448".

So I added couple of adjustments, because flight usually matches the ident_iata or actual_ident_iata with the flight number.

```javascript

const filteredFlights = response.data.scheduled.filter(
      (flight) =>
        flight.ident_iata?.includes(flightNumber) ||
        flight.actual_ident_iata?.includes(flightNumber)
    );

```

After I got the filtered flights, I sent the data to the app and displayed it on the screen.

And here is the result:

![Filtered Flights](/filtered-flights-api.png)

And also finally I got the flight data for the flight number "FR8448" to my home screen:

![Flight data](/filtered-flight-home.png)


### Saving the flight data to the local storage

After I got the flight data, I needed to save it to the local storage, so I can use it later in the app. I used the Ionic Storage to save the flight data to the local storage. This is how it works now:

```javascript
this.http.get(this.apiBaseUrl, { headers, params }).subscribe(
      async (response: any) => {
        console.log('API Response (Full):', response);

        if (response.scheduled && response.scheduled.length > 0) {
          response.scheduled.forEach(async (flight: any) => {
            const newFlight = {
              id: Date.now(),
              ...flight,
            };

            console.log('Adding Flight to Home Screen:', newFlight);

            this.flights.push(newFlight);

            const storedFlights = JSON.parse(localStorage.getItem('flights') || '[]');
            storedFlights.push(newFlight);
            localStorage.setItem('flights', JSON.stringify(storedFlights));
          });

          this.modal.dismiss(null, 'confirm');
        } else {
          alert('No scheduled flights found for the provided details.');
        }
      },
      (error) => {
        console.error('Error fetching flight data:', error);
        alert('Failed to fetch flight data. Please try again.');
      }
    );
```