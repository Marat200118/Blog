---
title: 'Api stack for the project'
description: 'What I choosed as my stack for the project'
pubDate: 'Jan 10 2025'
heroImage: '/blog-placeholder-api-stack.png'
---

### So, what API stack I choosed?

How I wrote in previous post about the APIs researched, it is difficult to find a good source of aviation data. But after a few days of researching, I finally found the perfect stack for my app. Here is what I choosed:

1. **OpenSky Network** - I understood that I really can use such a powerful source of data for free.

The thing is that OpenSky Network can find the flight and an airplane by the ICAO24 code. This code is unique aircraft identifier. But as a user you don't know this code, so I needed to find another API that can provide flight schedule by the flight number and then I can get the ICAO24 code and get the real-time or historical data from OpenSky Network.

2. **FlightAware AeroAPI** - This API provides a lot of information about flights, including scheduled departure and arrival times, aircraft type, and more. I found that it is a good fit for my app, because it provides the flight schedules and can communicate with OpenSky Network, so I can access also the historical data of the flight and then with OpenSky Network I can get the flight path.

