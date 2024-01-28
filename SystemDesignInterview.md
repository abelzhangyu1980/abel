# Proximity Service
a proximity service is used to discover nearby places i.e restaurants, hotels, theaters,museums..etc. input to the service wil be user's coordination.

## understand the problem
- search radius
- maximal radius allowed
- allow user to change radius
- business maintenance (add/update/delete...)
- NFRs: low latency, data privacy, high availability and scalability

## High level design
- search api
    /v1/search/nearby parameters: latitude, longitude, radius
- business api
    get /v1/businesses/1, post /v1/businesses/ put /v1/businesses/v1 delete /v1/businesses/1

## Data model
table design goes here

address, city, name, latitude, longtitude

### Geo index table
simple way: select  xxx from businss where latitude between ( %latitude% - radius and %latitude% + radius) and longtitude between ()

