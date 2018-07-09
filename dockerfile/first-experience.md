---
description: 一个简单的应用
---

# First Experience

```text
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD demo-1.0-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

目前j就用过这个，其他的之后也会用。

