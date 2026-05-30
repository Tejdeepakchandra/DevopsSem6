# Maven Build Lifecycle

## Why Build Tools Exist

In software development, building a project involves multiple steps:
- Compiling source code
- Running tests
- Packaging binaries
- Deploying artifacts

Doing these manually is error-prone and time-consuming. **Build tools** automate this entire process, ensuring consistency and reproducibility across different environments and team members.

### Problems Solved by Automated Builds
1. **Reproducibility** – Same build process everywhere (dev, CI, production)
2. **Dependency Management** – Auto-download and resolve library versions
3. **Standardization** – Uniform project structure and conventions
4. **Speed** – Incremental builds, caching, parallel execution
5. **Integration** – Seamless CI/CD pipeline integration

---

## Maven Build Lifecycle Phases

Maven defines a **default build lifecycle** with ordered phases. When you run a phase, all preceding phases are executed first.

### The Default Lifecycle (in order):

| Phase       | Description                                                        |
|-------------|-------------------------------------------------------------------|
| `validate`  | Validates the project is correct and all required info is available |
| `compile`   | Compiles the source code (`src/main/java`)                        |
| `test`      | Runs unit tests using a testing framework (JUnit, TestNG)         |
| `package`   | Packages compiled code into distributable format (JAR, WAR)       |
| `verify`    | Runs integration tests and checks to verify package quality       |
| `install`   | Installs the package into the local Maven repository (`~/.m2`)    |
| `deploy`    | Copies the package to a remote repository for sharing             |

### How It Works:

```bash
# Running 'package' will automatically execute: validate → compile → test → package
mvn package

# Running 'install' will execute: validate → compile → test → package → verify → install
mvn install

# Clean lifecycle (separate) - removes target/ directory
mvn clean

# Most common combo: clean + install
mvn clean install
```

### Other Lifecycles:
- **Clean Lifecycle**: `pre-clean` → `clean` → `post-clean`
- **Site Lifecycle**: `pre-site` → `site` → `post-site` → `site-deploy`

---

## Maven Directory Structure

Maven enforces a **standard directory layout** (convention over configuration):

```
my-project/
├── pom.xml                          # Project Object Model
├── src/
│   ├── main/
│   │   ├── java/                    # Application source code
│   │   │   └── com/example/App.java
│   │   └── resources/               # Configuration files, properties
│   │       └── application.properties
│   └── test/
│       ├── java/                    # Test source code
│       │   └── com/example/AppTest.java
│       └── resources/               # Test-specific resources
└── target/                          # Build output (generated)
    ├── classes/                     # Compiled .class files
    ├── test-classes/                # Compiled test classes
    └── my-project-1.0.jar           # Packaged artifact
```

This standard structure means any developer familiar with Maven can immediately navigate the project.

---

## Key Takeaways

1. Maven lifecycle phases execute **in order** – running a later phase triggers all earlier ones
2. The `clean` lifecycle is separate from the default lifecycle
3. `mvn clean install` is the most commonly used command in development
4. Convention over configuration reduces boilerplate setup
5. Understanding lifecycle phases is essential for CI/CD pipeline design
