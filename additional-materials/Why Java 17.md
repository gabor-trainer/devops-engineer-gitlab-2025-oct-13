# Why Java 17?

Here are the specific, professional reasons for standardizing on Java 17 for this training:

1.  **Parity with the CI/CD Build Environment:**
    Our Day 2 CI/CD labs will execute the Maven build inside a Docker container. The standard, modern Maven Docker images bundle a specific version of the JDK. The image we will specify in our `.gitlab-ci.yml` is `maven:3.9-eclipse-temurin-17`, which, as its tag implies, comes with **OpenJDK 17**. By having students install Java 17 locally, we ensure that the code they compile and test on their machine is being processed by the **exact same major version of the Java compiler and runtime** that the GitLab Runner will use. This eliminates a whole class of subtle bugs that can arise from version discrepancies.

2.  **It is a Long-Term Support (LTS) Release:**
    In an enterprise context, we do not simply choose the "latest" version of a language. We choose stable, predictable versions with long-term support. Java 17 is a designated **LTS release**. This means it receives security updates and bug fixes for many years (until at least 2029). Non-LTS versions (like 18, 19, 20) have a very short support window and are not suitable for production systems. By choosing an LTS version, we are teaching students to think about software lifecycle and stability, a critical consideration for any professional architect or developer.

3.  **Toolchain and Ecosystem Compatibility:**
    Java 17 is a modern, mature LTS release that is fully supported by the entire toolchain we are using:
    *   **Maven:** All modern versions of Maven run perfectly with Java 17.
    *   **VS Code:** The Java extensions for VS Code have excellent support for Java 17.
    *   **Docker:** Official, stable Docker images are readily available for Java 17.
    This ensures a smooth, well-integrated experience without compatibility issues.

4.  **Training Consistency and Risk Reduction:**
    By mandating a single, specific version for every student, we create a uniform baseline for the entire class. This dramatically reduces the risk of environment-specific problems during the labs. It prevents us from wasting valuable training time debugging issues like "my code works locally on Java 21 but fails in the CI pipeline running on Java 17."

