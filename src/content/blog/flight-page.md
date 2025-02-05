---
title: 'Flight page'
description: 'The flight page of the app'
pubDate: 'Jan 23 2025'
heroImage: '/flight-page-placeholder.jpg'
---

### Flight page
The flight page in my application is the page, where user can follow the flight in real-time. If the user has an upcoming flight, the app should show the estimated flight path. If the user is already on the flight, the app should show the real-time flight path. Depending on the user's connection, the app should show the estimated or real-time flight path.

At this time the flight page is not completed yet, but the core functionality is already implemented. 12 hours before the flight the app shows an upcoming flight in the flight page. 

```typescript
// tab2.page.ts

export class Tab2Page implements OnInit {
  flight: Flight | null = null; 
  flightPath: { latitude: number; longitude: number, timestamp: number }[] = [];
  status: string = '';
  currentAproximatePosition: { latitude: number; longitude: number } | null = null;
  message: string = ''; 
  private intervalId: any;

  flightDetails: any = null; 
  previousFlightDuration: number | null = null; 

  constructor(private storageService: StorageService, private navCtrl: NavController) {}

  async ngOnInit() {
    const flights = await this.storageService.getAllFlights();
    console.log('Fetched flights from storage:', flights);
    console.log('Flight Path:', this.flightPath);

    const now = new Date();
    const upcomingFlight = flights.find((flight: Flight) => {
      const scheduledArrival = new Date(flight.flightDetails.scheduled_in);
      const timeToDeparture = (scheduledArrival.getTime() - now.getTime()) / (1000 * 60 * 60);
      return timeToDeparture > 0 && timeToDeparture <= 12;
    });

    if (upcomingFlight) {
      this.flight = upcomingFlight;
      this.flightDetails = upcomingFlight.flightDetails;
      console.log('Upcoming flight:', this.flight);

      if (this.flight.previousFlight?.flightPath) {
        console.log('Previous Flight Path:', this.flight.previousFlight.flightPath);
        this.flightPath = this.formatFlightPath(this.flight.previousFlight.flightPath);

        const firstPoint = this.flightPath[0];
        const lastPoint = this.flightPath[this.flightPath.length - 1];
        this.previousFlightDuration = lastPoint.timestamp - firstPoint.timestamp;

        console.log('Previous Flight Duration (seconds):', this.previousFlightDuration);

        this.startLiveUpdates();
      } else {
        console.warn('No previous flight path available:', this.flight.previousFlight);
      }
    }
  }

  ngOnDestroy() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }

  goBackToMain() {
    this.navCtrl.navigateBack('/tabs/tab1');
  }

  updateFlightStatus() {
    if (!this.flight || !this.flightPath.length || !this.previousFlightDuration) return;

    const now = Date.now() / 1000; 
    const scheduledDeparture = Math.floor(
      new Date(this.flight.flightDetails.scheduled_out).getTime() / 1000
    );

    const elapsedTime = now - scheduledDeparture;

    if (elapsedTime < 0) {
      this.status = 'Waiting';
    } else if (elapsedTime >= 0 && elapsedTime < 30 * 60) { // First 30 minutes
      this.status = 'Boarding';
    } else if (elapsedTime >= 30 * 60 && elapsedTime < this.previousFlightDuration * 0.2) {
      this.status = 'Climbing';
    } else if (
      elapsedTime >= this.previousFlightDuration * 0.2 &&
      elapsedTime < this.previousFlightDuration * 0.8
    ) {
      this.status = 'Cruising';
    } else if (
      elapsedTime >= this.previousFlightDuration * 0.8 &&
      elapsedTime < this.previousFlightDuration
    ) {
      this.status = 'Descending';
    } else {
      this.status = 'Landed';
    }
  }

  updatePlanePosition() {
    if (!this.flight || !this.flightPath.length) return;

    const now = Math.floor(Date.now() / 1000);
    const scheduledDeparture = Math.floor(
      new Date(this.flight.flightDetails.scheduled_out).getTime() / 1000
    );

    const elapsedTime = now - scheduledDeparture;

    this.currentAproximatePosition = this.calculateCurrentPosition(elapsedTime);
    console.log('Current Approximate Position:', this.currentAproximatePosition);
  }

  private calculateCurrentPosition(elapsedTime: number): { latitude: number; longitude: number } | null {
    if (!this.flightPath || this.flightPath.length < 2 || elapsedTime < 0) return null;

    const totalDuration = this.previousFlightDuration!;
    if (elapsedTime > totalDuration) {
      // Plane has landed; return the last point in the flight path
      const lastPoint = this.flightPath[this.flightPath.length - 1];
      return { latitude: lastPoint.latitude, longitude: lastPoint.longitude };
    }

    const totalPathPoints = this.flightPath.length;
    const relativeTime = elapsedTime / totalDuration;

    const mappedIndex = Math.floor(relativeTime * (totalPathPoints - 1));
    const nextIndex = Math.min(mappedIndex + 1, totalPathPoints - 1);

    const start = this.flightPath[mappedIndex];
    const end = this.flightPath[nextIndex];

    if (!start || !end) return null;

    const segmentDuration = end.timestamp - start.timestamp;
    const segmentElapsedTime = elapsedTime - start.timestamp;

    if (segmentDuration <= 0 || segmentElapsedTime < 0) {
      return { latitude: start.latitude, longitude: start.longitude };
    }

    const segmentProgress = Math.min(1, Math.max(0, segmentElapsedTime / segmentDuration));

    return {
      latitude: start.latitude + segmentProgress * (end.latitude - start.latitude),
      longitude: start.longitude + segmentProgress * (end.longitude - start.longitude),
    };
  }



  startLiveUpdates() {
    this.intervalId = setInterval(() => {
      this.updateFlightStatus();
      this.updatePlanePosition();
    }, 1000);
  }

  formatFlightPath(flightPath: any[]): { timestamp: number; latitude: number; longitude: number }[] {
    const formatted = flightPath.map((point) => ({
      timestamp: point[0],
      latitude: point[1],
      longitude: point[2],
    }));
    console.log('Formatted Flight Path:', formatted);
    return formatted;
  }

}

```

The page has a live-flight-map component, which is responsible for showing the flight path. And animation of the airplane on the map.

```typescript
import { Component, Input, OnInit, OnChanges, SimpleChanges, OnDestroy } from '@angular/core';
import * as L from 'leaflet';

@Component({
  selector: 'app-live-flight-path-map',
  standalone: true,
  template: `<div id="live-map" style="height: 100%; width: 100%;"></div>`,
  styleUrls: ['./live-flight-map.component.scss'],
})
export class LiveFlightPathMapComponent implements OnInit, OnChanges, OnDestroy {
  @Input() flightPath: { latitude: number; longitude: number; timestamp: number }[] = [];
  @Input() planePosition: { latitude: number; longitude: number } | null = null;
  @Input() scheduledDeparture: number | null = null;
  @Input() previousFlightDuration: number | null = null;

  private map: L.Map | undefined;
  private planeMarker: L.Marker | undefined;

  ngOnInit(): void {
    if (!this.flightPath || this.flightPath.length === 0) {
      console.error('No flight path provided.');
      return;
    }

    this.initializeMap();
    this.plotFlightPath();
  }

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['flightPath']) {
      console.log('Updated Flight Path:', this.flightPath);
    }
    if (changes['planePosition'] && this.planePosition) {
      this.updatePlaneMarker();
    }
  }

  private initializeMap(): void {
    this.map = L.map('live-map').setView(
      [this.flightPath[0].latitude, this.flightPath[0].longitude],
      10
    );

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
    this.map.fitBounds(flightPathPolyline.getBounds());
  }

  private updatePlaneMarker(): void {
    if (!this.map || !this.planePosition) return;

    const bearing = this.calculateBearing();

    const planeIcon = L.divIcon({
      html: `<div style="transform: rotate(${bearing - 90}deg);">
              <img src="assets/airplane-outline.svg" style="width: 32px; height: 32px;">
            </div>`,
      className: '',
      iconSize: [32, 32],
      iconAnchor: [16, 16],
    });

    if (!this.planeMarker) {
      this.planeMarker = L.marker([this.planePosition.latitude, this.planePosition.longitude], {
        icon: planeIcon,
      }).addTo(this.map);
    } else {
      this.planeMarker.setLatLng([this.planePosition.latitude, this.planePosition.longitude]);
      this.planeMarker.setIcon(planeIcon);
    }

    this.map.setView([this.planePosition.latitude, this.planePosition.longitude], 10, { animate: true });
  }

   private calculateBearing(): number {
    if (!this.planePosition || this.flightPath.length < 2) return 0;

    const nextIndex = this.flightPath.findIndex(
      (point) =>
        point.latitude === this.planePosition?.latitude &&
        point.longitude === this.planePosition?.longitude
    );

    if (nextIndex === -1 || nextIndex >= this.flightPath.length - 1) return 0;

    const current = this.planePosition;
    const next = this.flightPath[nextIndex + 1];

    const lat1 = (current.latitude * Math.PI) / 180;
    const lon1 = (current.longitude * Math.PI) / 180;
    const lat2 = (next.latitude * Math.PI) / 180;
    const lon2 = (next.longitude * Math.PI) / 180;

    const y = Math.sin(lon2 - lon1) * Math.cos(lat2);
    const x =
      Math.cos(lat1) * Math.sin(lat2) -
      Math.sin(lat1) * Math.cos(lat2) * Math.cos(lon2 - lon1);
    const bearing = (Math.atan2(y, x) * 180) / Math.PI;

    return (bearing + 360) % 360; 
  }

  ngOnDestroy(): void {
    if (this.map) {
      this.map.remove();
    }
  }
}

```

So as we can see, the app maps the previous flight path and to the current time shows the approximate position of the airplane. The app also shows the status of the flight, like "Boarding", "Climbing", "Cruising", "Descending", "Landed". 

The app is not completed yet, but the core functionality is already implemented. I will continue to work on the flight page and make it more interactive and user-friendly. I will also research more about the Leaflet maps and see how I can improve the map in my app.


![Airplane on map](/airplane-map.png)
![Airplane on map](/airplane-map2.png)

### Testing the flight page

So after implementing the flight page, I decided to test the page. I added a new flight ant tracked. I opened the app and navigated to the flight page. I was able to see the upcoming flight and the previous flight path. I was also able to see the airplane on the map moving according to the time. It was a great feeling to see the flight page working. I will continue to test the flight page and make sure it works as expected.

![Testing the flight page](/console-map.png)