# Open Connectivity for Tourism (OCTo)

This project includes an independently-developed API specification for facilitating communication between partners, SLA guidelines, as well as templates and validation tools for implementing the API.

To run the documentation locally, clone the repository and run `npm start`. For a watch mode using nodemon, run `npm run start:w`

## API Structure

The API is divided into two main areas: Core and Capability.

### Core API

This represents the minimum required implementation for any partner using this API. The functionality defined in the Core API has been determined to be common enough across all products and bookings that implementation is mandatory.

### Capability API

To provide better support for features that may not be required by all products or bookings, the Core API is built to support capabilities. These capabilities can be specific to a particular Reseller or Booking platform, but there will also be adopted standards for capabilities that are very common (e.g. Pricing, Booking Questions, etc.) to provide a standard interface for the most common additional features that are not covered in the Core API.

## API Templates

To be developed.

## API Tools

To be developed.
