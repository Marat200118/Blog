---
title: 'How I made a UI for the app.'
description: 'How I made a UI for the app.'
pubDate: 'Feb 01 2025'
heroImage: '/interface-placeholder.jpg'
---

## How I designed the UI for my flight tracking app

When I started building my app, I knew that the UI would play a big role in making the user experience enjoyable and intuitive. I wanted users to feel connected to their journey in the air, whether they were simply tracking an upcoming flight or actively flying and following their path. The biggest challenge? Making everything look clean and functional without overwhelming the user with too much information.

So, let's break down how I approached each part of the UI.

### The Flight Page

This page is the heart of the app. It shows real-time information like the flight path, current airplane location, and status updates such as "Boarding" or "Cruising." I wanted this page to feel dynamic, updating live without needing constant user interaction.

Here's a glimpse of what it shows:

**The flight's route with the origin and destination airport codes.**

**Departure and arrival times.**

**The current status of the flight (e.g., "Waiting," "Climbing," etc.).**

An interactive map with the airplane's position plotted on the route.

**Challenges:**

At first, it was tricky to calculate and display the airplane's approximate position in real-time. I had to fetch the previous flight's path from storage and use timestamps to determine where the plane should be on that path based on the current time.

Also, I experimented with different ways to show flight status transitions, such as "Climbing" and "Descending." This part involved breaking down the flight duration into phases and updating the UI every second.

### Creating the Real-Time Flight Path Map

I used Leaflet.js for the map. It's lightweight and works well for interactive web and mobile applications. I plotted the flight path as a polyline and added an airplane icon that rotates based on its current direction.

Here’s what happens in the component:

**The map initializes with the flight's starting coordinates.**

**The flight path is drawn using the path points stored in the app.**

**The airplane icon updates its position and bearing based on the flight's progress.**

At first, I had issues with keeping the map responsive. On smaller devices, the airplane icon didn’t update smoothly. I added animations and adjusted the icon size to improve the experience.


### Visual Consistency

I took time to make sure the design elements, like colors and spacing, felt consistent throughout the app. I used IonChip components to label key flight information, giving the UI a polished look. Additionally, IonCards were used for sections like flight summaries and statistics, providing clear separation between different types of content.

To make this I created a designs in Figma and then implemented them in the app. I also used the Ionic documentation to find the best components for each part of the UI.

![Figma design](/iPhone16Plus-8.jpg)
![Figma design](/iPhone16Plus-9.jpg)
![Figma design](/iPhone16Plus-16.jpg)
![Figma design](/sign-in.jpg)