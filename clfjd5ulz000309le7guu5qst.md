---
title: "gRPC Implementation in Spring Boot"
datePublished: Wed Mar 22 2023 07:29:06 GMT+0000 (Coordinated Universal Time)
cuid: clfjd5ulz000309le7guu5qst
slug: grpc-implementation-in-spring-boot
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679470086438/9299dba3-46a4-41eb-a63a-7c8e1f84c90d.png
tags: java, springboot, grpc

---

gRPC is a high-performance open-source framework that provides an efficient and easy-to-use mechanism for building distributed systems. gRPC allows for seamless communication between services and is particularly well-suited for modern cloud-native applications. In this post, we'll explore the benefits of gRPC and how to use it in Java.

### Benefits of gRPC:

* **Language Agnostic:** One of the most significant benefits of gRPC is that it's language agnostic. gRPC provides libraries for multiple programming languages, including Java, Python, C++, and many others. This feature enables services written in different languages to communicate seamlessly.
    
* **High Performance:** gRPC is built on top of HTTP/2, which makes it incredibly fast and efficient. HTTP/2 provides multiple benefits like server push, request multiplexing, and header compression, which makes gRPC a high-performance framework.
    
* **Easy to Use:** gRPC has a straightforward API, which makes it easy to use. The framework provides features like request/response streaming, bidirectional streaming, and server-side streaming. These features make it simple to build complex distributed systems.
    
* **Interoperability:** gRPC allows for seamless communication between different systems, which enables services to work together even if they are written in different languages. gRPC uses a well-defined protocol buffer format to ensure interoperability.
    

### Using gRPC in Java:

* Define the service using protocol buffers: Protocol buffers are used to define the service contract, data models, and messages. Protocol buffers are used to generate the server and client code.
    
    Example: proto file for a unary service.
    
    ```plaintext
    syntax = "proto3";
    
    package greeting;
    
    option java_package = "com.manitaggarwal.grpcclient";
    option java_multiple_files = true;
    
    message GreetingRequest {
      string firstName = 1;
    }
    
    message GreetingResponse {
      string result = 1;
    }
    
    service GreetingService {
      rpc greet(GreetingRequest) returns (GreetingResponse);
    }
    ```
    
* Generate the server and client code: Once you have defined the service using protocol buffers, you need to generate the server and client code. The code generation is done using the protoc compiler, which generates the code in the language of your choice.
    
    Dependencies for gRPC in maven.
    
    ```xml
    <dependency>
    	<groupId>io.grpc</groupId>
    	<artifactId>grpc-netty-shaded</artifactId>
    	<version>1.53.0</version>
    	<scope>runtime</scope>
    </dependency>
    <dependency>
    	<groupId>io.grpc</groupId>
    	<artifactId>grpc-protobuf</artifactId>
    	<version>1.53.0</version>
    </dependency>
    <dependency>
    	<groupId>io.grpc</groupId>
    	<artifactId>grpc-stub</artifactId>
    	<version>1.53.0</version>
    </dependency>
    <dependency> <!-- necessary for Java 9+ -->
    	<groupId>org.apache.tomcat</groupId>
    	<artifactId>annotations-api</artifactId>
    	<version>6.0.53</version>
    	<scope>provided</scope>
    </dependency>
    ```
    
    Plugin requirements
    
    ```xml
    <build>
    	<extensions>
    		<extension>
    			<groupId>kr.motd.maven</groupId>
    			<artifactId>os-maven-plugin</artifactId>
    			<version>1.7.1</version>
    		</extension>
    	</extensions>
    	<plugins>
    		<plugin>
    			<groupId>org.xolstice.maven.plugins</groupId>
    			<artifactId>protobuf-maven-plugin</artifactId>
    			<version>0.6.1</version>
    			<configuration>
    				<protocArtifact>com.google.protobuf:protoc:3.21.7:exe:${os.detected.classifier}</protocArtifact>
    				<pluginId>grpc-java</pluginId>
    				<pluginArtifact>io.grpc:protoc-gen-grpc-java:1.52.1:exe:${os.detected.classifier}</pluginArtifact>
    			</configuration>
    			<executions>
    				<execution>
    					<goals>
    						<goal>compile</goal>
    						<goal>compile-custom</goal>
    					</goals>
    				</execution>
    			</executions>
    		</plugin>
    	</plugins>
    </build>
    ```
    
* Implement the server: Once you have the server code, you can implement the server logic. The server logic is where you implement the business logic of your service.
    
    ```java
    public class GreetingServiceImpl extends GreetingServiceGrpc.GreetingServiceImplBase {
    
        @Override
        public void greet(GreetingRequest request,
                          StreamObserver<GreetingResponse> responseObserver) {
            System.out.println("request.getFirstName() = " + request.getFirstName());
            responseObserver.onNext(GreetingResponse.newBuilder()
                    .setResult("Hello " + request.getFirstName())
                    .build());
            responseObserver.onCompleted();
        }
    }
    ```
    
* Implement the client: Once you have the client code, you can use it to make requests to the server. The client code provides a simple API that makes it easy to communicate with the server.
    
    ```java
    @SpringBootApplication
    public class GrpcClientApplication {
    
        private final static String CHANNEL_NAME = "localhost";
        private final static int SERVER_PORT = 50051;
    
        private final static List<String> list = List.of("Dan", "Pan");
    
        public static void main(String[] args) throws InterruptedException {
            SpringApplication.run(GrpcClientApplication.class, args);
            ManagedChannel channel = ManagedChannelBuilder
                    .forAddress(CHANNEL_NAME, SERVER_PORT)
                    .usePlaintext()
                    .build();
    
            GreetingServiceGrpc.GreetingServiceBlockingStub stub =
                    newBlockingStub(channel);
    
            for (String name : list) {
                GreetingResponse response =
                        stub.greet(GreetingRequest.newBuilder()
                                .setFirstName(name)
                                .build());
    
                System.out.println(response.getResult());
            }
    
            channel.shutdown();
        }
    ```
    

### Conclusion:

gRPC is a powerful framework that provides a high-performance and easy-to-use mechanism for building distributed systems. Its language-agnostic nature, high performance, easy-to-use API, and interoperability make it an excellent choice for building modern cloud-native applications. Using gRPC in Java is simple, and it provides a lot of benefits that make it an excellent choice for building complex distributed systems.

Code for the above can be found [here on the GitHub.](https://github.com/manitaggarwal/grpc-unary-example)