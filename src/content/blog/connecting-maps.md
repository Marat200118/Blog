---
title: 'Connecting Leaflet maps and flight page'
description: 'Connecting Leaflet maps and flight page'
pubDate: 'Jan 21 2025'
heroImage: '/ionic-maps-placeholder.jpg'
---

### Connecting Leaflet maps and flight page

So after I almost finished the logic for the fetching and accessing the data from the API, I decided to add the maps to the flight page. I wanted to show the flight path on the map. I decided to use the Leaflet maps because it is a very popular library for maps. My goal here was more to experiment with visualising the flight path on the map than to make it look perfect. Also, I will reseach more about the Leaflet maps in a offline mode.

So as I worked with leaflet maps before, I knew that I need to install the leaflet library and setup the map. To use it properly and make it reusable I decided to create a new component for the map. In the components folder I created a component called 'actual-flight-path-map', because for the estimated flight path I will create another component. Probably because it will be a bit different, with animations and mmore states.


```typescript
// actual-flight-path-map.component.ts

import { Component, Input, OnInit } from '@angular/core';
import * as L from 'leaflet';

@Component({
  selector: 'app-actual-flight-path-map',
  standalone: true,
  template: `<div id="map"></div>`,
  styleUrls: ['./actual-flight-path-map.component.scss'],
})
export class ActualFlightPathMapComponent implements OnInit {
  @Input() flightPath: { latitude: number; longitude: number }[] = [];

  private map: L.Map | undefined;

  ngOnInit(): void {
    console.log('Processed flight path:', this.flightPath);
    if (!this.flightPath || this.flightPath.length === 0) {
      console.error('No flight path provided.');
      return;
    }

    const isValidPath = this.flightPath.every(
      (point) =>
        typeof point.latitude === 'number' && typeof point.longitude === 'number'
    );

    if (!isValidPath) {
      console.error('Invalid flight path data:', this.flightPath);
      return;
    }

    this.initializeMap();
    this.plotFlightPath();
  }


  private initializeMap(): void {
    this.map = L.map('map').setView([
      this.flightPath[0].latitude,
      this.flightPath[0].longitude,
    ], 10);

    console.log('Map:', this.map);

    // this.map = L.map('map').setView([0, 0], 2);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      minZoom: 5,
      maxZoom: 18,
    }).addTo(this.map);

    setTimeout(() => {
      this.map?.invalidateSize();
    }, 100);

  }

  private plotFlightPath(): void {
    if (!this.map) return;

    const polylineCoordinates = this.flightPath.map((point) => [
      point.latitude,
      point.longitude,
    ] as [number, number]);

    const flightPathPolyline = L.polyline(polylineCoordinates, {
      color: 'blue',
      weight: 4,
    });

    flightPathPolyline.addTo(this.map);
    console.log('Flight path polyline:', flightPathPolyline);

    this.map.fitBounds(flightPathPolyline.getBounds());
  }

  ngOnDestroy(): void {
    if (this.map) {
      this.map.remove();
    }
  }
}

```

And included this in the details page, where I pass the actualFlight path to the component.

```html
  <div *ngIf="flight.actualFlight?.flightPath">
    <app-actual-flight-path-map [flightPath]="formatFlightPath(flight.actualFlight.flightPath)"></app-actual-flight-path-map>
  </div>
```

After some hours debugging and fixing the issues, I finally got the map working. I was able to see the flight path on the map. It was a great feeling to see the flight path on the map working. I will continue to work on the map and make it more interactive and user-friendly. I will also research more about the Leaflet maps and see how I can improve the map in my app.

![Connecting maps](/maps-setup-1.png)

So after I did it to the actual flights, I did the same for the estimated flight path. For now I reused the same component, but I will create a new one for the estimated flight path. I will also add some animations and more states to the estimated flight path map.

```html
 <div *ngIf="flight.previousFlight?.flightPath">
    <app-actual-flight-path-map [flightPath]="formatFlightPath(flight.previousFlight.flightPath)"></app-actual-flight-path-map>
  </div>
```

So I tested it adding a new flight from Brussels to Berlin with Flight number SN 2587 Departing on Tuesday January 21 at 16:20, I found previous flight between same cities and saved it as a previousFlight under flight object. And I was able to see the estimated flight path on the map.

![Connecting maps](/maps-setup.png)

### Passing upcoming flight to the flight page

Because I want to use the flight page to provide the user with the live information about his flight path, I decided to pass the upcoming flight to the flight page. So it appears 12 hours before the flight and the user can see the flight path on the map as estimated. If the user will be in airplane mode, We will use the estimated flight path and mapped time to show the approximate location of the flight.

```typescript
//tab2.page.ts

export class Tab2Page implements OnInit {
  flight: Flight | null = null; 
  flightPath: any = null;
  message: string = ''; 

  constructor(private storageService: StorageService, private navCtrl: NavController) {}

  async ngOnInit() {
    const flights = await this.storageService.getAllFlights();
    console.log('Fetched flights from storage:', flights);

    const now = new Date();
    const upcomingFlight = flights.find((flight: Flight) => {
      const scheduledDeparture = new Date(flight.flightDetails.scheduled_out);
      const timeToDeparture = (scheduledDeparture.getTime() - now.getTime()) / (1000 * 60 * 60);
      return timeToDeparture > 0 && timeToDeparture <= 12; 
    });

    if (upcomingFlight) {
      this.flight = upcomingFlight;
      console.log('Upcoming flight:', this.flight);

      if (!this.flight.previousFlight) {
        console.warn('No previous flight data available.');
      }
    } else {
      this.message = 'No upcoming flights within the next 12 hours.';
      console.log(this.message);
    }
  }

  goBackToMain() {
    this.navCtrl.navigateBack('/tabs/tab1');
  }

  formatFlightPath(flightPath: any[]): { latitude: number; longitude: number }[] {
  return flightPath
      .map((point) => ({
        latitude: point[1],
        longitude: point[2], 
      }))
      .filter(
        (point) =>
          typeof point.latitude === 'number' &&
          typeof point.longitude === 'number'
      );
  }
}

```

So with the storage Service we are getting all the flights from the storage and we are checking if there is any upcoming flight in the next 12 hours. If there is, we are passing it to the flight page. If there is no upcoming flight, we are showing the message that there is no upcoming flight in the next 12 hours.



```html
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>Flight</ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-header collapse="condense">
    <ion-toolbar>
      <ion-title size="large">Flight</ion-title>
    </ion-toolbar>
  </ion-header>

  <div *ngIf="flight; else noFlight">
    <ion-card>
      <ion-card-header>
        <ion-card-title>{{ flight.flightDetails.ident }}</ion-card-title>
      </ion-card-header>
      <ion-card-content>
        <p><strong>Origin:</strong> {{ flight.flightDetails.origin }}</p>
        <p><strong>Destination:</strong> {{ flight.flightDetails.destination }}</p>
        <p><strong>Departure:</strong> {{ flight.flightDetails.scheduled_out | date: 'yyyy-MM-dd HH:mm' }}</p>
        <p><strong>Arrival:</strong> {{ flight.flightDetails.scheduled_in | date: 'yyyy-MM-dd HH:mm' }}</p>
      </ion-card-content>
    </ion-card>

    <div *ngIf="flight.previousFlight; else noPath">
      <ion-card>
        <ion-card-header>
          <ion-card-title>Estimated flight path</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <app-actual-flight-path-map *ngIf="flight.previousFlight?.flightPath" [flightPath]="formatFlightPath(flight.previousFlight.flightPath!)">
          </app-actual-flight-path-map>
        </ion-card-content>
      </ion-card>
    </div>

    <ng-template #noPath>
      <div class="no-path-message">
        <p>Your estimated flight path is not known yet.</p>
      </div>
    </ng-template>
  </div>

  <ng-template #noFlight>
    <div class="no-flight-message">
      <p>{{ message }}</p>
    </div>
  </ng-template>

  <ion-button expand="full" (click)="goBackToMain()">Go Back</ion-button>
</ion-content>
```

![Connecting page](/maps-setup-flight-page.png)