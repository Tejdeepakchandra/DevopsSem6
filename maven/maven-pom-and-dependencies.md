# Maven POM & Dependency Management

## Project Object Model (POM)

The `pom.xml` is the fundamental unit of work in Maven. It contains:
- Project metadata (groupId, artifactId, version)
- Dependencies
- Build plugins and configuration
- Profiles for environment-specific builds

### Basic POM Structure:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project Coordinates (GAV) -->
    <groupId>com.example</groupId>        <!-- Organization/group -->
    <artifactId>my-app</artifactId>       <!-- Project name -->
    <version>1.0-SNAPSHOT</version>       <!-- Version -->
    <packaging>jar</packaging>            <!-- Output type: jar, war, pom -->
    
    <name>My Application</name>
    <description>A sample Maven project</description>
</project>
```

---

## Parent POM

A **Parent POM** allows multiple projects (modules) to inherit common configuration.

```xml
<!-- Parent POM (pom.xml in root) -->
<project>
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>    <!-- Must be 'pom' for parent -->
    
    <modules>
        <module>module-a</module>
        <module>module-b</module>
    </modules>
</project>

<!-- Child POM (module-a/pom.xml) -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0</version>
    </parent>
    
    <artifactId>module-a</artifactId>
</project>
```

Benefits of Parent POM:
- Centralized dependency versions
- Shared plugin configuration
- Consistent build behavior across modules

---

## Dependency Scope

Maven dependencies have **scopes** that control when they are available:

| Scope       | Compile | Test | Runtime | Packaged | Example                    |
|-------------|---------|------|---------|----------|----------------------------|
| `compile`   | ✅       | ✅    | ✅       | ✅        | Spring Framework           |
| `provided`  | ✅       | ✅    | ❌       | ❌        | Servlet API (server has it) |
| `runtime`   | ❌       | ✅    | ✅       | ✅        | JDBC drivers               |
| `test`      | ❌       | ✅    | ❌       | ❌        | JUnit, Mockito             |
| `system`    | ✅       | ✅    | ❌       | ❌        | Local JARs (avoid this)    |

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>       <!-- Only available during testing -->
</dependency>
```

---

## Transitive Dependencies

When your project depends on library A, and library A depends on library B, then **B is a transitive dependency** of your project.

```
Your Project → Spring Web → Spring Core → Commons Logging
                                          (transitive dep)
```

### Version Conflicts & Resolution

When two dependencies require different versions of the same library:

```
Your Project
├── Library A → commons-io:2.6
└── Library B → commons-io:2.11
```

**Maven's resolution strategy**: **Nearest definition wins** (shortest path in dependency tree)

If both are at the same depth, the **first declaration** in pom.xml wins.

### Viewing the Dependency Tree:
```bash
# See full dependency tree
mvn dependency:tree

# Analyze for conflicts
mvn dependency:analyze
```

---

## Using dependencyManagement

`dependencyManagement` in the parent POM **declares versions** without adding them as actual dependencies. Child modules can then reference dependencies without specifying versions.

```xml
<!-- Parent POM -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.15.0</version>    <!-- Version controlled centrally -->
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Child POM - no version needed -->
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <!-- Version inherited from parent's dependencyManagement -->
    </dependency>
</dependencies>
```

### Excluding Transitive Dependencies:
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.3.20</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## Key Takeaways

1. The **POM** is the heart of every Maven project – it defines everything
2. **Parent POM** centralizes configuration for multi-module projects
3. **Dependency scopes** control when libraries are available (compile, test, runtime)
4. **Transitive dependencies** are automatically resolved but can cause version conflicts
5. Use **dependencyManagement** to control versions across modules
6. Use `mvn dependency:tree` to debug dependency issues
