# GMIT Distributed Systems
## Lab: Building a REST API in Java with the Dropwizard Microservice Framework

## Overview
In this lab we'll build an implementation of the Artist API we designed in the [OpenAPI lab](https://github.com/john-french/distributed-systems-labs/tree/master/openAPI). The full OpenAPI defintion for the Artist API is [here](https://app.swaggerhub.com/apis/john-french/ArtistAPI/1.0.0). This definition will be our guide for what we need to implement.

We'll use the Dropwizard Java microservice framework to build this API. In thi slab we'll get a Dropwizard server up and running with one of our artists endpoints. This README won't reproduce the Dropwizard documentation, refer to the Dropwizard [Getting Started Guide](https://www.dropwizard.io/en/stable/getting-started.html) for further explanations of what we're doing.

(I mean this. Read the docs! Being a developer means figuring out how to use things by reading the documentation. Get used to it.)

## Lab Procedure
### Project Setup
The project so far only has a pom file. Take a look at it. It contains:
- a dependency for dropwizard, the framework we're using, which will pull in most of the other dependencies we'll need (Jersey, Jetty etc)
- maven build plugins for building executable jars


### Create a Configuration
Every Dropwizard app has to have a configuration, and a class for that configuration to be serialised to. So we need to create a configuration class. We're not going to use any user-defined configuration parameters in our simple API, so this class will be empty.

- Create a new class `ArtistApiConfig` in `ie.gmit.ds`.

```
package ie.gmit.ds;

import io.dropwizard.Configuration;

public class ArtistApiConfig extends Configuration {
}
```
We'll also create a configuration file. Create a new file `artistApiConfig.yaml` in the root folder of the project. We'll use it to configure Dropwizard to start our application on port 9000:
```
server:
  applicationConnectors:
    - type: http
      port: 9000
```

### Add an Artist resource class
Our API will manage artists, so we'll need a class to represent an `Artist`. In our OpenAPI definition we specified that an `Artist` would have the following fields:
- `artistName`: string
- `artistId`: integer
- `artistGenre`: string
- `albumsRecorded`: integer

Create a simple POJO (Plain Old Java Object) called `Artist` in with these 4 fields. The class should have:
- a no-argument constructor `public Artist() {}`
- a constructor that takes all 4 fields as arguments
```
public Artist(int artistId, String artistName, String artistGenre, int albumsRecorded) {
```
- getters for all 4 fields. Each getter should be annotated with `@JsonProperty`. This will allow the class to be serialised/deserialised by Jackson.
- don't create setters. We want our class to be immutable (can't be changed after it's created).

### Add an Artist Resource class
Now that we've created an `Artist` class, we can create the class which map incoming HTTP requests to methods using JAX-RS/Jersey annotations.

Create a new class called `ArtistApiResource`. Annotate the class with JAX-RS annotations that indicate that this class will:
- handle requests to the `/artists` base path
- return JSON responses
```
@Path("/artists")
@Produces(MediaType.APPLICATION_JSON)
public class ArtistApiResource {
```
Our artist API will manage a collection of artist data. Normally this would be in a database. For now, lets just create a map in our resource class, and put an example artist in it.
```
private HashMap<Integer, Artist> artistsMap = new HashMap<>();

public ArtistApiResource() {
    Artist testArtist = new Artist(1, "The GZA", "HipHop", 2);
    artistsMap.put(testArtist.getArtistId(), testArtist);
}
```
Now we can define some API operations. We'll start with `GET /artists`, the request what will return all artists. We'lll use the JAX_RS annotation `@GET` to map this method to a `GET` request:
```
@GET
public List<Artist> getArtists() {
    // artistsMap.values() returns a Collection<Artist>
    // We can convert this collection to an ArrayList by passing it into an
    // ArrayList<Artist> constructor
    return new ArrayList<Artist>(artistsMap.values());
}
```
This method just return the contents of the `artistsMap` as a list.


### Add an Application class
Now that we have the building blocks of our API set up, lets wire them up into a dropwizard application. We'll need to create an application class that extends `io.dropwizard.Application`:
```
package ie.gmit.ds;

import io.dropwizard.Application;
import io.dropwizard.setup.Environment;

public class ArtistApiApplication extends Application<ArtistApiConfig> {

    public static void main(String[] args) throws Exception {
        new ArtistApiApplication().run(args);
    }

    public void run(ArtistApiConfig artistApiConfig, Environment environment) throws Exception {

        final ArtistApiResource resource = new ArtistApiResource();

        environment.jersey().register(resource);
    }
}
```
The `run` method (called by the main method), registers our `ArtistApiResource` class, thereby making it available to the application to handle REST requests.

### Build and run
We can now build an executable jar. In a terminal, run `mvn package`. Once the build has finished, you should see a fat jar (>10MB) called `restAPI-dropwizard-lab-1.0-SNAPSHOT.jar` in the `target/` folder. Executing this with the parameters `server` and the path to our config file will start up our server:
```
java -jar target/restAPI-dropwizard-lab-1.0-SNAPSHOT.jar server src/main/resources/artistApiConfig.yaml
```

### Try it out
Going to the URL `<HOST/PORT URL>`/artists should now return our artist list (containing just one artist) in JSON format.

### Add more endpoints
Try to add handling for more endpoints in our API definition, e.g.:
- `GET /artists/{artistId}`
- `POST /artists`
