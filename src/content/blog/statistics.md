---
title: 'Statistics page'
description: 'Statistics page'
pubDate: 'Feb 05 2025'
heroImage: '/statistics-placeholder.jpg'
---

## Building the Statistics Page: Tracking Your Flight Achievements

Once I had the core functionality of the app in place—tracking flights and displaying real-time flight paths—I wanted to add a page to give users a summary of their flying history. The goal of the statistics page was simple: provide users with meaningful data about their flight journeys. This includes the total distance they've traveled, how many hours they've spent in the air, and how many flights they've completed.

Of course, I couldn’t stop there—I also added achievements and a chart to make the experience more interactive.

### Designing the Statistics Page Layout

I started by sketching out how I wanted the statistics page to look. The page would include:

**Total Flights Completed**

**Total Distance Traveled**

**Total Flight Time**

**Achievements**

**Monthly Flight Hours Chart**

Since I was working in Ionic, I decided to use a mix of built-in UI components like cards and lists, alongside custom components like a line chart and progress bars.

```html
<ion-content>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Flight Statistics</ion-card-title>
    </ion-card-header>
    <ion-card-content>
      <div class="stat-item">
        <ion-label>Total Flights:</ion-label>
        <ion-note>{{ totalFlights }}</ion-note>
      </div>

      <div class="stat-item">
        <ion-label>Total Distance Traveled:</ion-label>
        <ion-note>{{ totalDistance | number: '1.0-2' }} km</ion-note>
      </div>

      <div class="stat-item">
        <ion-label>Total Flight Time:</ion-label>
        <ion-note>{{ formatFlightTime(totalFlightTime) }}</ion-note>
      </div>
    </ion-card-content>
  </ion-card>

  <ion-card>
    <ion-card-header>
      <ion-card-title>Achievements</ion-card-title>
    </ion-card-header>
    <ion-card-content>
      <ul>
        <li *ngFor="let achievement of achievements">{{ achievement }}</li>
      </ul>
    </ion-card-content>
  </ion-card>

  <ion-card>
    <ion-card-header>
      <ion-card-title>Monthly Flight Hours</ion-card-title>
    </ion-card-header>
    <ion-card-content>
      <app-line-chart [data]="monthlyFlightHours"></app-line-chart>
    </ion-card-content>
  </ion-card>
</ion-content>

```

The layout is clean and broken down into sections for statistics, achievements, and a chart that shows flight hours by month. I used the line chart as a way to visualize the user's flying patterns over time.

### Fetching and Calculating the Data

Once the UI was in place, the next step was to fetch the user's saved flights from Ionic storage and calculate the necessary statistics. I created a method in the StatisticsPage class to handle this:

```typescript
async loadFlights() {
  this.flights = await this.storageService.getAllFlights(this.profileId);
  console.log('Fetched flights:', this.flights);
}
```

Once I had the flights, I filtered out any flights that were already completed (i.e., flights where the departure time was in the past).

```typescript
const completedFlights = this.flights.filter(
  (flight) => flight.actualFlight && new Date(flight.flightDetails.scheduled_out) < new Date()
);
```

From there, I looped through these completed flights to calculate:

**Total flight time:**

I measured the time between the first and last timestamps in the flight path or using the firstSeen and lastSeen properties provided by the API.

```typescript
if (flight.actualFlight.lastSeen && flight.actualFlight.firstSeen) {
  const flightTime = (flight.actualFlight.lastSeen - flight.actualFlight.firstSeen) / 3600;
  this.totalFlightTime += flightTime;
}
```

**Total distance traveled:**

I calculated the distance between each point in the flight path using the Haversine formula.

```typescript
const flightDistance = this.calculateFlightDistance(flight.actualFlight.flightPath);
this.totalDistance += flightDistance;

calculateFlightDistance(flightPath: any[]): number {
  let distance = 0;

  for (let i = 0; i < flightPath.length - 1; i++) {
    const point1 = flightPath[i];
    const point2 = flightPath[i + 1];

    distance += this.haversine(point1[1], point1[2], point2[1], point2[2]);
  }

  return distance;
}
```

### Challenges and Improvements
I faced a few challenges while implementing the statistics page. One major issue was handling incomplete or corrupted flight path data. I had to write a helper function to filter out invalid points and convert them into a usable format:

```typescript
convertFlightPathPoint(point: any): { latitude: number; longitude: number } {
  if (Array.isArray(point) && point.length >= 3) {
    return { latitude: point[1], longitude: point[2] };
  }
  return point;
}

```