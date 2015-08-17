---
layout: post
title: Build a location based API with Spring Data MongoDB and GeoJSON
author: Driss Amri
date: 2015-08-18
tags:
 - Java
 - Spring
 - Spring Data MongoDB
 - Spring Boot
 - MongoDB
 - GeoJSON
---
Recently I was intrigued about a project discussion in which a high volume REST API was needed to store and retrieve location based information. A performant solution was need that could work with big datasets, atleast containing a few million entries.

I haven't done much work in the Geospatial area but I thought this could be an interesting use case to get my hands dirty to learn something new.
The requested API needed to be able to store location points for a subject (person). On the other hand these locations needed to be retrieved based on their proximity to a certain point (latitude and longitude).

After stumbling upon MongoDB's support for [GeoJSON](http://geojson.org/), a format for encoding a variety of geographic data structures, it looked like a perfect match for this use case. High volume and performance shouldn't be a problem for MongoDB. A [GeoJSON Point](http://docs.mongodb.org/v3.0/reference/geojson/) (latitude, longtitude) was already supported in MongoDB since v2.4, a lot of other features and improvements have been added since then in the current v3.0. Sounds impressive, no?

I decided to start a quick prototype has contained two methods:

* `GET` `/locations` that has four query parameters. `lat`: latitude, `long`: longtitude, `d`: distance from the location and `s`: subject.
* `POST` `/locations` that has one query parameter. `s`: subject and a `body` that contains a list of locations for this specific subject.

The technologies for this prototype are:

 * Spring Boot
 * Spring Data MongoDB
 * Gradle

## Generate a basic project
Head over to [Spring Initializer](https://start.spring.io/), or if you are using IntelliJ you can just choose to create a new Spring Boot application, and choose Spring Web, Spring Data MongoDB and Gradle. You should end up with a `build.gradle` something along the lines of:


_build.gradle_
{% highlight groovy %}
buildscript {
  ext {
    springBootVersion = '1.3.0.M3'
  }
  repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/milestone" }
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    classpath 'com.sourcemuse.gradle.plugin:gradle-mongo-plugin:0.8.0'
  }
}

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'spring-boot'
apply plugin: 'mongo'

jar {
  baseName = 'locations'
  version = '0.0.1-SNAPSHOT'
}

springBootVersion = '1.3.0.M3'

repositories {
  mavenCentral()
  maven { url "https://repo.spring.io/milestone" }
}

dependencies {
  compile("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  compile("org.springframework.boot:spring-boot-starter-data-mongodb")
  compile("org.springframework.boot:spring-boot-starter-web")
  testCompile("org.springframework.boot:spring-boot-starter-test")
}

task wrapper(type: Wrapper) {
  gradleVersion = '2.3'
}
{% endhighlight %}

Since the Gradle Wrapper is recommended by the [documentation](https://docs.gradle.org/current/userguide/gradle_wrapper.html) I also used it.

## Add Spring Data MongoDB repository
Spring Data JPA is basicially a standard in most of my projects, so I was happy to see that there is also a [Spring Data MongoDB](http://projects.spring.io/spring-data-mongodb/) project and it already also has support for [Geospatial queries](http://docs.spring.io/spring-data/data-mongo/docs/current/reference/html/#mongo.geospatial)!

First let's create a simple `LocationEntity` which will be stored in MongoDB. For the sake of brevity I left out the getters/setters/equals/hashcode, you can find the full source on [GitHub](https://github.com/drissamri/blog-examples/blob/master/spring-data-mongodb-geospatial/src/main/java/be/drissamri/locations/repository/domain/LocationEntity.java):

_LocationEntity.java_
{% highlight java %}
@Document(collection = "locations")
public class LocationEntity {
  private String id;
  private String subject;
  private GeoJsonPoint location;

  public LocationEntity(final String subject, final GeoJsonPoint location) {
    this.subject = subject;
    this.location = location;
  }
}
{% endhighlight %}

Next we need a Spring Repository that will actually query for all locations for a certain subject and a proximity near a location. This is where all the heavy lifting (magic?) has to happen.

_LocationRepository.java_
{% highlight java %}
public interface LocationRepository extends MongoRepository<LocationEntity, String> {

  List<LocationEntity> findBySubjectAndLocationNear(String sid, Point p, Distance d);

}
{% endhighlight %}

That's all you need, Spring Data will take care of everything based on the method name. That's awesome, right? You pass a subject, a certain Point (longitude and latitude) and how far in distance it should look from that certain point in miles or kilometers. Spring magic at its best.

## Implementing REST endpoints
As we defined before, we'll need to two endpoints to retrieve the locations based on the repository that we just created and also create new LocationEntity entries. Let's start with retrieving the locations:

_LocationResource.java_
{% highlight java %}
@RestController
public class LocationResource {
  @Autowired
  private LocationRepository repository;

  @RequestMapping(method = RequestMethod.GET)
  public final List<LocationEntity> getLocations(
    @RequestParam("lat") String latitude,
    @RequestParam("long") String longitude,
    @RequestParam("d") double distance,
    @RequestParam(value = "s", required = false) String subjects) {

    return this.repository.findBySubjectAndLocationNear(subjects,
      new Point(Double.valueOf(longitude), Double.valueOf(latitude)),
      new Distance(distance, Metrics.KILOMETERS));
  }
}
{% endhighlight %}

Spring gives you a nice `RestController` annotation that will make sure all the methods in the class are annotated with `ResponseBody`. The only this this method do is have a few query parameters and use them to call our `LocationRepository`. We have to define the metric we want to use for the `distance` parameter, in our case we'll use kilometers.

Next, we want to add locations for a certain subject

_LocationResource.java_
{% highlight java %}
  @RequestMapping(method = RequestMethod.POST)
  @ResponseStatus(HttpStatus.CREATED)
  public final void addLocations(
    @RequestParam("s") String sid,
    @RequestBody List<LocationEntry> entries) {

    List<LocationEntity> entities = new ArrayList<>();
    for (LocationEntry location : entries) {
      final GeoJsonPoint locationPoint = new GeoJsonPoint(
        Double.valueOf(location.getLongitude()),
        Double.valueOf(location.getLatitude()));

      entities.add(new LocationEntity(sid, locationPoint));
    }

    this.repository.save(entities);
  }
{% endhighlight %}

The `LocationEntry` POJO is an object that only contains latitude and longitude, since that's all we need for a location that will be added. Based on this POJO we created a new `GeoJsonPoint` to store it in MongoDB.

With these few steps I was able to create a Geospatial Location based API prototype within a few hours, mostly spent on doing research on MongoDB and Spring Data MongoDB capabilities. I'm happy that I now know MongoDB is capable of quite a bit of Geospatial things, and that it is quite easy to use as well.

Hope you learned something new as well! You can find the source code for this example on [GitHub](https://github.com/drissamri/blog-examples/tree/master/spring-data-mongodb-geospatial).
