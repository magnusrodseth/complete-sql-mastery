# Project: Video Rental Application

## Description

This is a project about designing the conceptual model, the logical model, the
physical model, and finally the database for a video rental application.

Requirements can be found below, and should be sufficient for modeling the data
in the database.

## Requirements

We’re going to build a desktop application called Vidly. This application will
be used at a video rental store. We need different levels of permissions for
different users.

The store manager should be able to add/update/delete the list of movies. They
will be in charge of setting the stock for each movie as well as the daily
rental rate. Cashiers should have a read-only view of the list of movies. They
should be able to manage the list of customers and the movies they rent.

At check out, a customer brings one or more movies. The cashier looks up a
customer by their phone number. If the customer is a first-time customer, the
cashier asks their full name, email and phone number, and then registers them in
the system. The cashier then scans the movies the customer has brought to check
out and records them in the system. Each movie has a 10 digit barcode printed on
the cover.

When the customer returns to the store, they’ll bring the movies they rented. If
a movie is lost, the customer should be charged 5 times the daily rental rate of
the movie. The cashier should mark the movie as lost and this will reduce the
stock. There is no need to keep track of the lost movies. All we need to know is
the the number of movies in stock and how much the customer was charged.

For other movies, the customer should be charged based on the number of days and
the daily rental rate.

We issue discount coupons from time to time. The customer can bring a coupon
when returning the movies.

It is possible that a customer returns the movies they’ve rented in multiple
visits.

We need to be able to track the:

- Top movies
- Top customers
- Revenue per day, month and year
