---
title: 'FlightAware AeroAPI + OpenSky Network'
description: 'First experiments with APIs'
pubDate: 'Jan 13 2025'
heroImage: '/connection-placeholder.jpg'
---

### FlightAware AeroAPI connection with OpenSky Network
So to make the app work, I needed to connect two APIs: FlightAware AeroAPI and OpenSky Network. The first one provides the flight schedules, and the second one provides the real-time and historical flight data.

The complex thing here was, that OpenSky Network can find the flight and an airplane by the ICAO24 code. This code is unique aircraft identifier. But as a user you don't know this code, and FlightAware AeroAPI can't provide this code. So I needed to find a way to connect these two APIs.

### How I tried to connect them?
To make an experiment, I made a sample test flight, which I added manually:

```javascript
const testFlight = {
      id: Date.now(),
      ident: 'LOT784',
      ident_icao: 'LOT784',
      ident_iata: 'LO784',
      actual_ident: null,
      actual_ident_icao: null,
      actual_ident_iata: null,
      aircraft_type: 'E75L',
      scheduled_in: '2025-01-12T13:50:00Z',
      scheduled_out: '2025-01-12T12:00:00Z',
      origin: 'EVRA',
      origin_icao: 'EVRA',
      origin_iata: 'RIX',
      origin_lid: null,
      destination: 'EPWA',
      destination_icao: 'EPWA',
      destination_iata: 'WAW',
      destination_lid: null,
      fa_flight_id: 'UAL784-1736491986-airline-250p',
      meal_service: 'Business: Snack or brunch / Economy: Food for sale',
      seats_cabin_business: 3,
      seats_cabin_coach: 76,
      seats_cabin_first: 0,
      isTest: true,
    };
```

When I press this test flight, the app is sending  a request to the OpenSky Network API to get more information about the flight, which was in past based on the historical data.

So I made a service file, where I do this request:

```javascript
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { catchError, switchMap } from 'rxjs/operators';
import { Observable, throwError } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class FlightService {
  private proxyBaseUrl = 'http://localhost:3000/api';

  constructor(private http: HttpClient) {}

  private toUnixTimestamp(isoDate: string): number {
    return Math.floor(new Date(isoDate).getTime() / 1000);
  }

  private getIcao24(callsign: string): Observable<any> {
    const url = `${this.proxyBaseUrl}/opensky/states?callsign=${callsign}`;

    return this.http.get(url).pipe(
      catchError((error) => {
        console.error('Error fetching ICAO24:', error);
        return throwError(() => new Error('Failed to fetch ICAO24 from OpenSky API.'));
      })
    );
  }

  getDeparturesWithArrival(
    departureAirport: string,
    startTime: string,
    endTime: string,
    arrivalAirport?: string,
    callsign?: string

  ): Observable<any> {

    const begin = this.toUnixTimestamp(startTime);
    const end = this.toUnixTimestamp(endTime);

    let url = `${this.proxyBaseUrl}/opensky/departures?airport=${departureAirport}&begin=${begin}&end=${end}`;
    
    if (arrivalAirport) {
      url += `&arrivalAirport=${arrivalAirport}`;
    }
    if (callsign) {
      url += `&callsign=${callsign}`;
    }

    return this.http.get(url).pipe(
      catchError((error) => {
        console.error('Error fetching departures:', error);
        return throwError(() => new Error('Failed to fetch departures from OpenSky API.'));
      })
    );
  }


  getFlightPath(callsign: string, startTime: string, endTime: string): Observable<any> {
    const begin = this.toUnixTimestamp(startTime);
    const end = this.toUnixTimestamp(endTime);

    const url = `${this.proxyBaseUrl}/opensky/flights?callsign=${callsign.trim()}&begin=${begin}&end=${end}`;

    return this.http.get(url).pipe(
      catchError((error) => {
        console.error('Error fetching flight path:', error);
        return throwError(() => new Error('Failed to fetch flight path from OpenSky API.'));
      })
    );
  }
}
```

And this is how the server looks like:

```javascript
app.get("/api/opensky/departures", async (req, res) => {
  const { airport, begin, end, arrivalAirport, callsign } = req.query;

  if (!airport || !begin || !end) {
    return res.status(400).json({
      error: "Parameters 'airport', 'begin', and 'end' are required.",
    });
  }

  const url = `https://opensky-network.org/api/flights/departure?airport=${airport}&begin=${begin}&end=${end}`;
  console.log(
    `Fetching departure flights from OpenSky API for airport: ${airport}`
  );
  console.log("URL:", url);

  try {
    const response = await axios.get(url);
    const flights = response.data;

    console.log(`Found ${flights.length} flights departing from ${airport}.`);

    let filteredFlights = arrivalAirport
      ? flights.filter((flight) => flight.estArrivalAirport === arrivalAirport)
      : flights;

    if (callsign) {
      const normalizedCallsign = callsign.toUpperCase();
      filteredFlights = filteredFlights.filter((flight) => {
        const flightCallsign = flight.callsign?.trim();
        return (
          flightCallsign && flightCallsign.includes(normalizedCallsign)
        );
      });
    }

    return res.json(filteredFlights);
  } catch (error) {
    console.error(
      "Error fetching departures:",
      error.response?.data || error.message
    );
    res.status(500).json({
      error: "Failed to fetch departure flights from OpenSky API.",
    });
  }
});
```

How you can see the server is fetching the departure flights from the OpenSky Network API, and then filtering them by the arrival airport and callsign.

Also, the flight number is LOT784, but the callsign is "LOT4KK", and we need to filter the flights by the callsign, because OpenSky Network API can't find the flight by the flight number. So we are taking the "LOT" part and also checking the time.

This is what I got, before the filtering:

![Flight response](/response-from-opensky.png)

And this is what I got after the filtering, and added to the details page:

![Flight response](/opensky-added-home.png)

The time of the flights are the same, arrival and destination airports also, so I can be sure that this is the flight I need.