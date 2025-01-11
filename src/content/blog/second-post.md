---
title: 'Aviation API Exploration'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jan 09 2025'
heroImage: '/blog-placeholder-api.png'
---

The core idea behind my app is to help users explore what they’re flying over—landmarks, cities, or natural features—all while offline. To do this, I needed to dive into the world of APIs to figure out how to get the data I need.

So when I started researching APIs, I found that it is quite challenging to find a good source of aviation data. The most popular and powerful APIs are usually paid, and the free ones are often limited in terms of the data they provide. Here are some examples of APIs I explored:

### AviationStack
AviationStack is a powerful API that provides real-time flight data, including flight status, departure and arrival times, and more. It’s easy to use and has a lot of features that make it a great choice for developers who need accurate and up-to-date information about flights. But the downside is that it’s a paid service, and the free tier has some limitations. So for free, you can make only 100 requests per month, which is not enough for my app.

Also, AviationStack doesn’t provide detailed information about the flight path and flight scheduled (in free tier), which is a key feature of my app. So I had to keep looking for other options.

![AviationStack placeholder](/aviationstack.png)


### Aviation Edge
Aviation Edge is another API that provides a lot of information about flights, including flight status, departure and arrival times, aircraft type, and more. There is no free tier, but I wanted to start exploring how the data can work in my app for free.

![AviationEdge placeholder](/aviationedge.png)

### Airlabs 
Airlabs seemed really interesting for me, because for free you can get 1000 requests per month, which is a lot for a free tier. I registered, but the thing is that they accep every registration manually, so I'm still waiting for the approval. I waited for a week, but still no response, so I decided to move on.

![AirLabs placeholder](/airlabs.png)

### OpenSky Network
OpenSky Network is a non-profit organization that provides real-time flight data, including flight status, departure and arrival times, and more. The API is free to use, but it has some limitations. It really has lots of data, and after researching more, I found, that their online flight data and historical flight routes can really help me to build an app. So I decided to go with OpenSky Network, but the only thing, is that they don't provide the flight schedules, and I need to find another API for that.

![OpenSky placeholder](/opensky.png)

### FlightAware AeroAPI
For my application I needed a good source of flight schedules. So when the user adds a flight via an application, the system can make a request to fetch the detailed information about the scheduled flight. I found that FlightAware AeroAPI is a good fit for this purpose. It provides a lot of information about flights, including scheduled departure and arrival times, aircraft type, and more. The API is easy to use and has a lot of documentation. It has a system pay-as-you-go, so you pay only for the requests you make with 5$ free requests. I registered and started exploring the API.

![Aero placeholder](/aero.png)