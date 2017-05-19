# Introduction

## The Topics

Welcome! The goal here is to help you \(the fabulous gumshoe-level Elm developer\) get a feel for how to perform some tasks in Elm that are essential to the creation of many modern real-world Web applications. Those skills are, specifically, performing HTTP requests, handling the JSON that comes back from the requests in a sensible way, and routing to different pages in your app based on the URL in the location bar. All three of those topics will be covered in this document in an introductory fashion.

When first learning Elm I had a very difficult time wrapping my head around how to make a simple HTTP request, and how to handle JSON. Reading documentation didn't seem to help me much, looking at code helped a little bit, but poking around existing code, and making changes until things broke and worked again was what really helped me get started. During this workshop I'm assuming you learn like I did, and I'll give you some code by example, but then I'll ask you to poke around on your own a bit, so that you can get the hang of it without me breathing over your shoulder.

## The Project: Hipstore

I wanted to teach these skills with a watered-down and silly version of what might be a real-world application. In this case, we'll be building an extremely simple online product list and shopping cart for "Hipstore" a fantastical and fabricated company that rents out [hipsters](https://en.wikipedia.org/wiki/Hipster_%28contemporary_subculture%29) for a day to enliven social events and brighten photoshoots. Also available for rent are fancy and fashionable hipster accessories that are compatible with any of the hipsters available on the site.

And also, the currency is entirely taco-based 🌮.

A live example of the finished project can be seen here: [http://app.hipstore.sploding.rocks/](http://app.hipstore.sploding.rocks/)

