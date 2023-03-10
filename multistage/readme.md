# multistage docker files

### Add the following code in a file named HelloWorld.java

class HelloWorld {
   public static void main(String[] a) {
       System.out.println("Hello world!");
   }
}

### Then, create a Dockerfile with the following content in it,

```
FROM openjdk:11-jdk
COPY HelloWorld.java .
RUN javac HelloWorld.java
CMD java HelloWorld

```

Build the image with the following command,
```
docker build -t helloworld:huge .
```


Let’s modify our Dockerfile with the following content to show how multi-stage Docker build works.
```
FROM openjdk:11-jdk AS build
COPY HelloWorld.java .
RUN javac HelloWorld.java

FROM openjdk:11-jre AS run
COPY --from=build HelloWorld.class .
CMD java HelloWorld
```
Build the image with the following command,
```
docker build -t helloworld:small .
```

Now, let’s compare both images. Check the images created with the following command,
```
docker images
```
![image](https://user-images.githubusercontent.com/13297994/222604700-f1d3272b-2eef-4eaa-991d-aa602ff765e4.png)




Push the image to docker hub


docker pushxxxxxxxx/helloworld:small

![image](https://user-images.githubusercontent.com/13297994/222604545-de59f496-2dba-43ca-b438-1d8187bb7afd.png)

