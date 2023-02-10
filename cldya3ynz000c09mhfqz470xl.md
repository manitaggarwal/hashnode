# Use MongoDB to store Java Application Logs

## Introduction

MongoDB is a NoSQL database program and uses JSON-like documents with optional schemas. Which makes it exceptionally great for storing logs.

I wrote a simple spring boot microservice to consume logs using a REST API and a Kafka Listener and store them in MongoDB.

Using Java Generics, we can create a document to store any JSON information on the database.

## REST Controller

```java
    @PostMapping(value = "/logs", consumes = "application/json", produces = "text/plain")
    public void addLogs(@RequestBody LoggingRequest<?> request) {
        loggingService.saveLogs(request);
    }
```

## Request

```java
@Data // using lombok
public class LoggingRequest<T> {
    String applicationName;
    T logs;
}
```

## Document

```java
@Document("logfile") // using springframework.data.mongodb
@Data //using lombok
public class LogFile<T> {

    @Id
    private String id;

    private Date time;

    private T data;

    public LogFile(Date time, T data) {
        this.time = time;
        this.data = data;
    }
}
```

## Repository

```java
// using springframework.data.mongodb.repository.MongoRepository
public interface LoggingRepository extends MongoRepository<LogFile<?>, String> {
}
```

## Data in MongoDB

```json
{
  "time": {
    "$date": {
      "$numberLong": "1675923022932"
    }
  },
  "data": {
    "applicationName": "monitoring",
    "logs": {
      "type": "success",
      "message": "add.success"
    },
    "_class": "com.manitaggarwal.store.log.controller.request.LoggingRequest"
  },
  "_class": "com.manitaggarwal.store.log.document.LogFile"
}

{
  "time": {
    "$date": {
      "$numberLong": "1675915874047"
    }
  },
  "data": {
    "applicationName": "monitoring",
    "logs": {
      "type": "error",
      "trace": "customer.does.not.exists"
    },
    "_class": "com.manitaggarwal.store.log.controller.request.LoggingRequest"
  },
  "_class": "com.manitaggarwal.store.log.document.LogFile"
}
```

## Conclusion

Data is stored in MongoDB and can be queried using filters.

```json
{ "data.applicationName" : "monitoring", "data.logs.type" : "success" }
```

## Reference

[Github Repo Link](https://github.com/manitaggarwal/logging-service)