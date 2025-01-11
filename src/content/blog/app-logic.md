---
title: 'App logic and user flow'
description: 'The full logic of the app and user flow'
pubDate: 'Jan 8 2025'
heroImage: '/Blog-placeholder-logic.jpg'
---

### So, how the app should work?
After few nights constantly thinking about the app and exploring the APIs, I finally came up with the full logic of the app. Here is how it should work:

1. **User opens the app** - The user opens the app and is able to see the main screen with already added flights or an empty screen if there are no flights added yet. In the main screen user also can add the flight by clicking the "Add flight" button.

2. **User adds a flight** - When the user clicks the "Add flight" button, the app opens the modal with form, where user should provide the **Flight number** and **Departure date**. After the user fills the form and clicks the "Add" button, the app sends a request to the Flight schedule API to get the flight data. If the flight is found, the app adds the flight to the main screen. If the flight is not found, the app shows the error message.

### What's next?

When the users have an upcoming flights, this is where the most interesting part of the app starts.

3. **At the day of the flight** - Right before the flight, the app should send a request to the OpenSky Network API. The app should find the previous flight of the same airline, and same deraprture and arrival airports. **(Because most likely the route will stay the more or less the same)**. The app should store the flight path to the local storage.

4. **During the flight** - During the flight  the app constantly checks for the connection (airplane mode). If user is offline, we are using the stored flight path to show the user the approximate current location of the airplane. If user is online, the app should send a request to the OpenSky Network API to get the real-time data of the flight.

5. **After the flight** - After the flight, the app should show the user the full flight path, and the information about the flight. The app should also show the user the previous flight path, so the user can compare the routes. Also, providing a statistics in the separate screen, like how many flights user had, how many kilometers he flew, etc.