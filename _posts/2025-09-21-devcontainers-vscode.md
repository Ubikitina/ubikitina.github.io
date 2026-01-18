---
layout: post
title: "Boosting Data Engineering Workflows with Dev Containers in Visual Studio Code"
permalink: /2025-09-21-devcontainers-vscode.html 
date: 2025-09-21
---

As a developer, have you ever encountered the infamous **"it works on my machine"** problem?

This is exactly what **Dev Containers** solve. A development container is a running Docker container that is fully configured with a specific toolset, runtime, and environment settings required for a project.

For Data Engineers, this concept is particularly vital because our work involves complex stacks:
1.  **Multiple Runtimes:** Juggling Python versions alongside the Java Virtual Machine (JVM) needed for Spark.
2.  **Cloud Integration:** Needing various CLIs (`awscli`, `gcloud`, etc.) and Infrastructure-as-Code tools (`terraform`).
3.  **Dependency Conflicts:** Managing project-specific library versions without polluting your local machine.

By using [Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers), I’ve managed to create a reproducible, portable, and fully configured environment tailored specifically for my data projects.

In this post, I’ll walk you through how to set up a devcontainer, sharing an example of a configuration (complete with Python, Spark, AWS CLI, and Terraform) that helps me be more productive and avoid dependency hell.


---

## What’s Inside My Dev Container?

A `.devcontainer` folder can contain multiple files. In this example, I will focus on three key files that bring everything together, ensuring a consistent environment:


### 1. `Dockerfile` - Custom Python Environment for Data Engineering

This file defines the base image and installs everything needed for a comprehensive Data Engineering workflow.

* **Core Runtimes:**
    * **Python (specific version):** Ensures version consistency across team members.
    * **OpenJDK:** Crucial, as distributed processing tools like Spark run on the JVM.
* **System Dependencies:** `curl`, `unzip`, `build-essential`, and database connectors.
* **Spark Integration:** I set up the `Dockerfile` to download and configure Spark, ensuring environment variables like `$JAVA_HOME$`v and `$SPARK_HOME$` are correctly defined and added to the `$PATH$`.
* **Python Libraries:**
    * `pyspark` - for distributed data processing.
    * `pandas` - for efficient data wrangling.
    * Others.
* **Tools:**
    * `awscli` - for interacting with AWS services.
    * `terraform` - for infrastructure as code (IaC).

This complete setup means we have everything from running Spark jobs to deploying infrastructure ready to go in one isolated container.


### 2. `devcontainer.json` - VS Code Extensions and Workspace Configuration

This JSON file brings the VS Code experience to life inside the container.

* **Workspace Settings:** Configures VS Code's integrated terminal to use Bash as the default shell and adjusts SSL settings for working easily with proxies.
* **Mount Points:** Defines local directories or secrets to be attached to the container, providing flexibility while maintaining isolation.
* **Extensions for Productivity:** Visual Studio Extensions are installed directly into the container, ensuring that all developers have the same debugging and development tools. Here is a list of some interesting extensions:

| Category | Extension | Functionality Focus |
| :--- | :--- | :--- |
| **Data/Python** | `ms-python.vscode-pylance` | Fast and precise type checking and advanced *IntelliSense*. |
| **Cloud/AWS** | `AmazonWebServices.aws-toolkit-vscode` | Explore cloud resources, manage credentials, and invoke services from VS Code. |
| **IaC/DevOps** | `hashicorp.terraform` | Full support for Terraform configuration files, including syntax highlighting and validation. |
| **Containers** | `ms-azuretools.vscode-docker` | Manage Docker containers, images, and networks through the VS Code UI. |
| **Configuration** | `redhat.vscode-yaml` | Schema-based validation and formatting for YAML files. |
| **Readability** | `oderwat.indent-rainbow` | Uses color coding to visually distinguish levels of indentation, enhancing readability. |

> **Other Useful Extensions:** While the table above covers the most impactful extensions for core productivity, countless other extensions can be a game-changer depending on your specific needs, such as: `andyyaldoo.vscode-json` (JSON files formatting support), `wayou.vscode-todo-highlight` (hightlight keywords such as `TODO`), and others.



### 3. `.bashrc` - Interactive and Smart Terminal

A custom `.bashrc` file supercharges the terminal experience inside the container:

- **Persistent command history** - Never lose track of useful, project-specific commands.
- **Custom prompt** - Shows user, host, working directory, and current Git branch.
- **Aliases and autocomplete** - Faster access to commonly used commands, including AWS CLI.

The result is a terminal that’s not just functional, but a joy to use.

---

## Final Thoughts

Using devcontainers in VS Code has completely changed how I approach project setup. For data engineering work that spans multiple tools, languages, and cloud services, having a well-defined environment is invaluable.It provides:

* **Consistency:** Every collaborator gets the exact same environment.
* **Isolation:** Your local machine stays clean and free of conflicting packages.
* **Productivity:** Everything is pre-configured, no more time wasted on environment setup.

If you're a data engineer looking for a scalable way to manage your dev environment, I highly recommend giving devcontainers a try.

---

*Got questions or want to share your setup? Reach out to me on my [GitHub profile](https://github.com/Ubikitina).* 📩😊
