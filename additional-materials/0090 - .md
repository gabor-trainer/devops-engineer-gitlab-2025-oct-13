### **Introduction to YAML**

#### **1. The Need for Data Serialization: A Tale of Four Formats**

In software development and operations, we constantly need to express structured, hierarchical data in a text format that is both **human-readable** and **machine-parsable**. This is the core purpose of data serialization languages. They serve as a bridge between the complex data structures our programs use (like objects, lists, and dictionaries) and the simple text files we use for configuration and data exchange.

Over the years, several formats have emerged, each with its own strengths and weaknesses.

| Format   | Key Characteristic                                                                                                                                                                  | Best Suited For                                                                              | Example                   |
| :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------- | :------------------------ |
| **XML**  | **Verbose & Tag-based.** Uses explicit opening and closing tags. Very powerful but often difficult for humans to read.                                                              | Legacy enterprise systems, SOAP APIs, complex document structures (e.g., Maven's `pom.xml`). | `<user><id>1</id></user>` |
| **JSON** | **Ubiquitous & Strict.** Uses braces, brackets, and quotes. The native format for web APIs, but its strict syntax (no comments, trailing commas) makes it clumsy for configuration. | Data interchange, REST APIs, web applications.                                               | `{"user": {"id": 1}}`     |
| **YAML** | **Human-Readable & Minimalist.** Uses indentation to denote structure, minimizing syntax characters. The standard for modern configuration files.                                   | **Configuration files** for DevOps tools, CI/CD pipelines, and cloud-native applications.    | `user: {id: 1}`           |
| **TOML** | **Explicit & Unambiguous.** Designed to be an obvious, minimal configuration file format. Uses sections like INI files.                                                             | Simple application configuration where ambiguity is a primary concern.                       | `[user]\nid = 1`          |

While all can represent the same data, **YAML has won the configuration management space** due to its focus on human readability.

#### **2. YAML in the Wild: Common Use Cases**

Learning YAML is not optional in modern DevOps; it is a prerequisite. You will find it everywhere:

*   **GitLab CI/CD:** `.gitlab-ci.yml` files define your entire automation pipeline.
*   **Docker Compose:** `docker-compose.yml` files define and run multi-container applications.
*   **Kubernetes:** All resource definitions (Pods, Deployments, Services) are expressed in YAML.
*   **Ansible:** Playbooks, inventories, and variable files are all written in YAML.
*   **GitHub Actions:** `.github/workflows/*.yml` files define CI/CD workflows.
*   **CloudFormation & Helm:** Used for defining cloud infrastructure and application packages.

---

### **The Fundamentals: Basic YAML Syntax**

YAML's power comes from its simplicity, which is almost entirely based on structure.

#### **Key-Value Pairs**
The most basic unit in YAML is a key-value pair, also known as a **mapping**.
**Rule:** A key is followed by a colon and a space (`: `).

```yaml
# A simple key-value pair
name: Gabor Szabo
```

#### **Indentation: The Golden Rule of YAML**
YAML uses indentation to define structure. This is the most critical and most common point of error.

> **The Golden Rule:** Indentation **must** be done with **spaces**. **Tabs are forbidden** and will cause your file to be invalid. The number of spaces is not fixed, but it must be **consistent** within the same block. Two spaces is the most common and recommended convention.

Indentation creates nesting. The following shows that `email` and `is_admin` "belong" to `user`.

```yaml
user:
  email: student.name@company.com
  is_admin: false
```

#### **Lists (Sequences)**
To represent a list of items, you use a dash followed by a space (`- `).

```yaml
# A simple list of strings
roles:
  - developer
  - administrator

# A list of numbers
project_ids:
  - 101
  - 205
  - 310
```

#### **Combining Mappings and Sequences**
The real power of YAML comes from combining these structures. The most common pattern in DevOps is a **list of dictionaries**.

```yaml
# A list of users, where each user is an object
users:
  - name: Gabor Szabo
    email: gabor.szabo@company.com
    role: trainer
  - name: Student Name
    email: student.name@company.com
    role: developer
```

#### **Data Types and Multi-line Strings**
YAML supports standard data types.
*   **Strings:** Can be unquoted, but should be quoted (`'` or `"`) if they contain special characters or could be mistaken for another type (e.g., `"true"`, `"1.0"`).
*   **Numbers:** `42`, `3.14`
*   **Booleans:** `true`, `false`

For long blocks of text, YAML has excellent support for multi-line strings.
```yaml
# The '|' preserves newlines.
description: |
  This is a multi-line description.
  Each line will be preserved exactly as written.

# The '>' folds newlines into spaces, creating a single long line.
# It is useful for long paragraphs.
summary: >
  This is a single long sentence that has been wrapped
  in the YAML file for readability but will be parsed
  as one continuous line of text.
```

#### **Comments**
Lines beginning with a hash (`#`) are comments and are ignored by the parser. Use them liberally to document your configuration.

---

### **Advanced YAML Features**

These are powerful features for creating maintainable, DRY (Don't Repeat Yourself) configurations.

#### **Anchors (`&`) and Aliases (`*`)**
This is YAML's native mechanism for reusability. An **anchor (`&`)** assigns a name to a node (a value, list, or object). An **alias (`*`)** then references that node, duplicating its content.

This is the principle behind GitLab CI's `extends` keyword.

```yaml
# Define a reusable block of script commands with an anchor named 'default_script'
.default_job_definition: &default_script
  - echo "Setup complete."
  - run-dependencies.sh

job1:
  script:
    - *default_script # Use an alias to insert the commands
    - echo "Running job 1..."

job2:
  script:
    - *default_script # Reuse the same commands here
    - echo "Running job 2..."
```

#### **Merge Keys (`<<`)**
A merge key allows you to merge an entire mapping (anchored object) into another mapping. This is extremely useful for sharing default properties.

```yaml
# Define a set of default properties for a job
.job_defaults: &job_defaults
  stage: test
  image: alpine:latest

test_job_1:
  <<: *job_defaults # Merge in the defaults
  script:
    - echo "This job runs in the test stage with the alpine image."

test_job_2:
  <<: *job_defaults # Merge in the same defaults
  script:
    - echo "So does this one."
  image: ubuntu:latest # But we can still override a specific key
```

#### **Multi-document Streams (`---`)**
A single YAML file can contain multiple independent YAML documents, separated by three dashes (`---`). This is used extensively by tools like Kubernetes to bundle multiple resource definitions into one file.

```yaml
# Document 1: A user configuration
name: Gabor Szabo
role: trainer
---
# Document 2: An application configuration
app:
  port: 8080
  database_url: postgres://...
```