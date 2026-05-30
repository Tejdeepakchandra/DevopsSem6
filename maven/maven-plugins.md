# Maven Plugins & Execution

## What are Maven Plugins?

Maven is essentially a **plugin execution framework**. All the real work (compiling, testing, packaging) is done by plugins. Each plugin provides one or more **goals** that can be bound to lifecycle phases.

---

## Compiler Plugin

The **maven-compiler-plugin** compiles Java source code.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>       <!-- Java source version -->
                <target>17</target>       <!-- Java target bytecode version -->
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Key Points:**
- Bound to the `compile` and `test-compile` phases by default
- Controls which Java version to compile for
- Can enable/disable annotation processing

---

## Surefire Plugin (Unit Testing)

The **maven-surefire-plugin** runs unit tests during the `test` phase.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <includes>
            <include>**/*Test.java</include>
            <include>**/*Tests.java</include>
        </includes>
        <excludes>
            <exclude>**/IntegrationTest.java</exclude>
        </excludes>
        <!-- Skip tests (not recommended for CI) -->
        <!-- <skipTests>true</skipTests> -->
    </configuration>
</plugin>
```

**Key Points:**
- Auto-detects JUnit 4, JUnit 5, and TestNG
- Test reports generated in `target/surefire-reports/`
- Use `-DskipTests` to skip test execution
- Use `-Dmaven.test.skip=true` to skip compilation and execution of tests

---

## Shade Plugin (Uber JAR / Fat JAR)

The **maven-shade-plugin** creates an **uber JAR** – a single JAR containing your code and ALL dependencies.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.5.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.App</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

**Key Points:**
- Creates a self-contained JAR that can run with `java -jar`
- Useful for microservices and containerized applications
- Handles resource merging (e.g., META-INF/services)
- Can relocate/rename packages to avoid classpath conflicts

---

## Maven Wrapper (mvnw)

The **Maven Wrapper** ensures everyone on a team uses the **same Maven version** without requiring a global Maven installation.

### Setting Up:
```bash
# Generate wrapper files
mvn wrapper:wrapper -Dmaven=3.9.4
```

### Generated Files:
```
project/
├── mvnw              # Unix/Mac script
├── mvnw.cmd          # Windows script
└── .mvn/
    └── wrapper/
        ├── maven-wrapper.jar
        └── maven-wrapper.properties
```

### Usage:
```bash
# Instead of 'mvn', use './mvnw'
./mvnw clean install        # Unix/Mac
mvnw.cmd clean install      # Windows
```

**Why use it?**
- No need to install Maven globally
- Guarantees consistent Maven version across the team
- Essential for CI/CD pipelines – the build server doesn't need Maven pre-installed
- Commit wrapper files to version control

---

## Maven and Docker Integration

### dockerfile-maven-plugin

The **dockerfile-maven-plugin** integrates Docker image building into the Maven lifecycle.

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <executions>
        <execution>
            <id>default</id>
            <goals>
                <goal>build</goal>
                <goal>push</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>myregistry/my-app</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

### Sample Dockerfile for a Maven-based Application:

```dockerfile
# Multi-stage build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Benefits of Multi-Stage Builds:
- Build environment stays separate from runtime
- Smaller final image (only JRE, not full JDK + Maven)
- Build artifacts don't bloat the production image

### Pushing to Registries:
```bash
# Build and push in one command
mvn dockerfile:build dockerfile:push

# Or as part of the lifecycle
mvn deploy   # If plugin is bound to deploy phase
```

---

## Key Takeaways

1. **Compiler plugin** – controls Java version compatibility
2. **Surefire plugin** – runs unit tests, generates reports
3. **Shade plugin** – creates fat/uber JARs for standalone execution
4. **Maven Wrapper** – locks Maven version, no global install needed
5. **dockerfile-maven-plugin** – integrates Docker builds into Maven lifecycle
6. **Multi-stage Dockerfiles** – keep production images lean
