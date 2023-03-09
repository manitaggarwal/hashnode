---
title: "Rest Assured - Testing RESTful APIs"
datePublished: Thu Mar 09 2023 08:26:12 GMT+0000 (Coordinated Universal Time)
cuid: clf0uh7vf000709md09cr4ikp
slug: rest-assured
tags: java, testing, rest-api, springboot, rest-assured

---

In today's software development world, testing has become an integral part of the development process. Testing helps to ensure the quality and reliability of software. In this blog post, we will discuss how to use Rest Assured for testing RESTful APIs.

### Testing RESTful APIs with Rest Assured

Rest Assured is a Java-based library that simplifies the testing of RESTful APIs. It provides a simple and intuitive DSL (Domain-Specific Language) for writing tests for RESTful APIs. It also supports various HTTP methods such as GET, POST, PUT, DELETE, etc.

Let's look at an example of how to use Rest Assured to test a RESTful API. Consider the following endpoint:

```java
@GetMapping(value = "/student/{studentId}", produces = MediaType.APPLICATION_JSON_VALUE)
public Student getStudentByStudentId(@PathVariable String studentId) {
    return studentService.getStudent(studentId);
}
```

This endpoint retrieves a student with a given student ID. To test this endpoint, we can write a test case using Rest Assured as follows:

```java
/*
 * Given: Where a student exists in the system
 * When: Call is placed to retrieve the student using student id
 * Then: Details of the student are fetched correctly
 * */

@Test
public void getStudentWhenExists_ByStudentId() {

    // given - prerequisites
    String msisdn = getRandomNumber();
    String email = "harry@admin.com";
    String name = "Harry";
    Student response = (Student) JsonUtils.getObjectFromJson(
            APICall.addStudent(DefaultData.getAdminRequest(msisdn, email, name)).asString(),
            Student.class);
    Assert.assertEquals("add assert", msisdn, response.getMsisdn());

    String studentId = response.getStudentId();

    // when
    Student studentFromDatabase = (Student) JsonUtils.getObjectFromJson(APICall.getStudent(studentId).asString(),
            Student.class);

    // then
    Assert.assertEquals("get assert", studentFromDatabase.getStudentId(), studentId);

}
```

In the above test case, we are using the given to specify the parameters for the request. In this case, we are ensuring that the student whom we want to retrieve exists in the system.

Then we are using the when to send the GET request to the specified endpoint.

Finally, we are using the then to verify the response. We are verifying that the student retrieved is having the same ID as specified by us in the request.

### Conclusion

In this blog post, we discussed how to use Rest Assured. Official documentation for the rest assured can be found at [GettingStarted · rest-assured/rest-assured Wiki (](https://github.com/rest-assured/rest-assured/wiki/GettingStarted)[github.com](http://github.com)[)](https://github.com/rest-assured/rest-assured/wiki/GettingStarted).

Also, the complete code for this project can be found at [manitaggarwal/spring-boot-rest-assured: Implementing Rest Assured (](https://github.com/manitaggarwal/spring-boot-rest-assured)[github.com](http://github.com)[)](https://github.com/manitaggarwal/spring-boot-rest-assured)