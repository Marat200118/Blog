---
title: 'Connecting Ionic storage'
description: 'Connecting Ionic storage'
pubDate: 'Jan 18 2025'
heroImage: '/ionic-storage-placeholder.jpg'
---

### Connecting Ionic storage

After I did the fetching and displaying the data from the API using the local storage, I decided to use the Ionic storage. The reason for this is that the local storage is not the best solution for storing the data. The Ionic storage is a better solution because it is a key-value store that is more secure and faster than the local storage. Also, how I discovered later, in the iphone simulator the local storage is not working properly.

I read the documentation and I found out that the Ionic storage is very easy to use. I just needed to install the package and import it in the main.ts file. Then I could use it in the service file.

I faced lots of problems with the Ionic storage. The first one was that I faced lots of errors with importing it properly. I needed to define drivers in the main.ts file. The second problem was that I needed to use the async and await functions to make it work properly. The third problem was, with pushing the data to the storage.

I made this storage.service.ts file:

```javascript

//storage.service.ts
import { Injectable } from '@angular/core';
import { Storage } from '@ionic/storage-angular';
import { Drivers } from '@ionic/storage';
import * as CordovaSQLiteDriver from 'localforage-cordovasqlitedriver';
import { Flight } from '../models/flight.model';

@Injectable({
  providedIn: 'root',
})
export class StorageService {
  private _storage: Storage | null = null;

  constructor(private storage: Storage) {
    this.init();
  }

  async init() {
    try {
      await this.storage.defineDriver(CordovaSQLiteDriver);
      const storage = await this.storage.create();
      console.log('Storage initialized with driver:', storage.driver);

      this._storage = storage;
    } catch (error) {
      console.error('Failed to initialize storage:', error);
    }
  }

  private async ensureInitialized(): Promise<void> {
    if (!this._storage) {
      await this.init();
    }
  }

  async addFlight(flight: Flight): Promise<void> {
    await this.ensureInitialized();
    const flights = (await this.getAllFlights()) || [];
    flights.push(flight);
    await this._storage?.set('flights', flights);
  }

  async getAllFlights(): Promise<Flight[]> {
    return (await this.get('flights')) || [];
  }

  async getFlightById(flightId: string): Promise<Flight | undefined> {
    const flights = await this.getAllFlights();
    return flights.find((flight) => flight.flightId === flightId);
  }

  async updateFlight(flight: Flight): Promise<void> {
    const flights = await this.getAllFlights();
    const index = flights.findIndex((f) => f.flightId === flight.flightId);
    if (index !== -1) {
      flights[index] = flight;
      await this.set('flights', flights);
    }
  }

  async getPreviousFlight(previousFlightId: string): Promise<Flight | undefined> {
    return await this.getFlightById(previousFlightId);
  }

  async clear(): Promise<void> {
    await this._storage?.clear();
  }

  public async set(key: string, value: any): Promise<void> {
    await this._storage?.set(key, value);
  }

  public async get(key: string): Promise<any> {
    return await this._storage?.get(key);
  }
}

```

The functions I use in here, I took from the examples in the documentation and other example files. Also, I needed to remake the tab1.page.ts file, because I needed to use the storage service in there. I needed to change the functions in there to use the storage service.

```javascript
this.http.get(url, { headers, params }).subscribe(
    async (response: any) => {
      console.log("API Response:", response);

      if (response.scheduled && response.scheduled.length > 0) {
        const flight = response.scheduled[0];

        const existingFlight = await this.storageService.getFlightById(flight.fa_flight_id);
        if (existingFlight) {
          console.log('Flight already exists in storage:', existingFlight);
          alert('This flight is already saved.');
        } else {
          const flightRecord: Flight = {
            flightId: flight.fa_flight_id,
            flightDetails: flight,
            // previousFlightId: null,
            // actualFlightPathId: null,
          };


          await this.storageService.addFlight(flightRecord);
          this.flights.unshift(flightRecord);
          await this.loadFlightsFromStorage();
          this.modal.dismiss();
          console.log("Flight added to storage:", flightRecord);
        }
      } else {
        alert("No scheduled flights found for the provided details.");
      }
    },
    (error) => {
      console.error("Error fetching flight data:", error);
      alert("Failed to fetch flight data. Please try again.");
    }
  );
```

I use the FLIGHT model to store the data in the storage.

The main purpose of this, is that I don't want to make 3 different tables or records in the storage (Flights, ActualFlightPaths, PreviousFlightPaths). So I neede to remae my whole saving thing, to achieve the following structure:

```json
{
  flightId: "CLH2287-1737102489-airline-680p",
  flightDetails: {
    ident: 'DLH2287',
    ident_icao: 'DLH2287',
    ident_iata: 'LH2287',
    actual_ident: 'CLH2287',
    actual_ident_icao: 'CLH2287',
    ...
  },
  actualFlight: {
    icao24: '48ad0f',
    firstSeen: 1736702530,
    estDepartureAirport: 'EVRA',
    estArrivalAirport: 'EPWA',
    flightPath: [ /* Flight path data */ ]
  },
  previousFlight: {
    icao24: '48ad0b',
    firstSeen: 1736683190,
    estDepartureAirport: 'EVRA',
    estArrivalAirport: 'EPWA',
    flightPath: [ /* Flight path data */ ]
  }
}
```

So the logic is that I create a flight, when user adds a flight. 8 hours before the flight I fetch previous flight, with the flight path, so it can be used, if the user is offline. And 12 hours after the flight, I fetch the actual flight path, so the user can see the flight path he actually went. Why 12 hours? Because the Opensky Network Api are updating the todays flights, at night, so I need to wait for the next day to get the actual flight path.

After some tries, I managed to make it work, but I still need to improve the loading of the data from the storage,and saving instances of the flights, so data is easier to access.

![Ionic storage](/ionic-storage.png)