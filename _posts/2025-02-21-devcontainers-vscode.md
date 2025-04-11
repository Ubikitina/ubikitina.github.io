---
layout: post
title: "Boosting Data Engineering Workflows with Dev Containers in Visual Studio Code"
permalink: /2025-02-21-devcontainers-vscode.html 
date: 2025-04-11
---

As a data engineer, maintaining a consistent development environment across machines and projects can be challenging. That's where **dev containers** come in. By using [Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers), I’ve managed to create a reproducible, portable, and fully configured environment tailored specifically for my data projects.

In this post, I’ll walk you through how I use devcontainers in my day-to-day workflow, and how my current setup—with Python, Spark, AWS CLI, and Terraform—helps me be more productive and avoid the classic “it works on my machine” syndrome.

---

## 🔧 What’s Inside My Dev Container?

My `.devcontainer` folder contains three key files that bring everything together:


### 1. `Dockerfile` – Custom Python Environment for Data Engineering

This file defines the base image and installs everything I need for my workflow:

- **Python (specific version)** – Ensures version consistency across team members.
- **System dependencies:**  
  `openjdk` (necessary because Spark runs on the JVM), `curl`, `unzip`, `build-essential`, `libpq-dev`
- **Environment variables:**  
  `JAVA_HOME` and custom `PATH`
- **Python libraries:**  such as
  - `pyspark` – for distributed data processing  
  - `pandas` – for data wrangling  
  - `psycopg2-binary` – PostgreSQL interaction  
- **Tools:**  
  - `awscli` – for interacting with AWS services  
  - `terraform` – for infrastructure as code  

This setup gives me everything from Spark jobs to deploying infrastructure in one container.


### 2. `devcontainer.json` – VS Code Extensions and Workspace Configuration

This JSON file brings the VS Code experience to life by adding:

- **VS Code's integrated terminal** to use Bash as the default shell, and adjusts SSL settings to make it easier to work in environments with proxies or non-standard certificates.
- **VS Code extensions for productivity:**  
  
| Extension                             | Description |
|---------------------------------------|-------------|
| `AmazonWebServices.aws-toolkit-vscode` | Integrates AWS services into VS Code, allowing exploration of cloud resources, invocation of Lambda functions, and management of credentials. |
| `andyyaldoo.vscode-json`              | Improves the experience of working with JSON files by providing validation, syntax highlighting, and formatting support. |
| `editorconfig.editorconfig`           | Applies consistent coding style rules across different editors and environments using `.editorconfig` files. |
| `hashicorp.terraform`                 | Offers full support for Terraform configuration files, including syntax highlighting, auto-completion, linting, and validation. |
| `mark-tucker.aws-cli-configure`       | Assists in the interactive setup of AWS CLI credentials and profiles within the VS Code interface. |
| `ms-azuretools.vscode-docker`         | Enables the management of Docker containers, images, volumes, and networks through the Visual Studio Code user interface. |
| `oderwat.indent-rainbow`              | Uses color coding to visually distinguish levels of indentation, which enhances readability of nested structures in formats like YAML, JSON, and Python. |
| `redhat.vscode-yaml`                  | Provides schema-based validation, auto-completion, and formatting for YAML files, essential for configuration-driven development. |
| `ryu1kn.annotator`                    | Supports the addition of custom annotations or visual markers (e.g., highlights and notes) directly in the code. |
| `tuxtina.json2yaml`                   | Facilitates the conversion between JSON and YAML formats, useful when working with configuration files in different ecosystems. |
| `wayou.vscode-todo-highlight`         | Highlights tags such as `TODO`, `FIXME`, and other keywords, improving the visibility of in-code reminders and pending tasks. |
| `ms-python.python`                    | The official extension for Python development, providing script execution, environment management, linting, and Jupyter notebook integration. |
| `ms-python.debugpy`                   | A Python debugging engine that supports breakpoints, variable inspection, and step-through code execution. |
| `ms-python.vscode-pylance`            | Delivers fast and precise type checking, intelligent auto-completion, and advanced static code analysis using the Pyright engine. |


- **Mount points:** Useful for attaching local directories or secrets into the container, making it flexible while still isolated.


### 3. `.bashrc` – Interactive and Smart Terminal

I created a custom `.bashrc` to supercharge the shell:

- **Persistent command history** – Never lose track of useful commands  
- **Custom prompt** – Shows user, host, working directory, and current Git branch  
- **Aliases and autocomplete** – Faster access to commonly used commands, including AWS CLI  

The result is a terminal that’s not just functional, but a joy to use.

---

## 🚀 How This Helps Me Day to Day

- **Consistency**: Whether I switch machines or onboard a new collaborator, they get the exact same environment.
- **Isolation**: I don’t clutter my local machine with conflicting versions or global packages.
- **Productivity**: Everything is pre-configured—no more wasting time setting up libraries or tools.
- **Portability**: I can run the same dev environment on my laptop, in the cloud, or in a CI/CD pipeline.

---

## ✅ Final Thoughts

Using devcontainers in VS Code has completely changed how I approach project setup. For data engineering work that spans multiple tools, languages, and cloud services, having a well-defined environment is invaluable.

If you're a data engineer looking for a scalable way to manage your dev environment, I highly recommend giving devcontainers a try.

---

*Got questions or want to share your setup? Reach out to me on my [GitHub profile](https://github.com/Ubikitina).* 📩😊
