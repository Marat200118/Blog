---
title: 'Aviation API Exploration'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jan 07 2025'
heroImage: '/blog-placeholder-4.jpg'
---

The core idea behind my app is to help users explore what they’re flying over—landmarks, cities, or natural features—all while offline. To do this, I needed to dive into the world of APIs to figure out how to get the data I need.

I started with APIs for flight data. OpenSky Network and AirLabs stood out because they provide detailed flight information like routes, schedules, and aircraft details. I also began looking at APIs like OpenStreetMap and GeoNames for mapping and landmark data. These tools are free, which is a big plus, but they also have their own quirks. Understanding how to make requests, parse the data, and display it in the app is a learning process, but it’s also super satisfying when it works.

One of the most important features of the app is offline functionality. Flights don’t have Wi-Fi (most of the time), so the app needs to pre-fetch data and store it locally. I’m currently testing SQLite for this, and it seems like a good option for lightweight, offline databases.

