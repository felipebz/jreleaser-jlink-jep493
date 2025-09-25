# JEP 493 Jlink Cross-Platform Reproducer

This project demonstrates a subtle issue when using Jlink with JDK 24+ distributions that implement [JEP 493: Linking Run-Time Images without JMODs](https://openjdk.org/jeps/493) with JReleaser 1.20.0.

## Background

* Starting with [Eclipse Temurin 24](https://adoptium.net/pt-BR/news/2025/08/eclipse-temurin-jdk24-JEP493-enabled), JEP 493 is enabled.
* The `jmods/` directory is no longer shipped with the JDK.
* `jlink` is able to create **same-platform** custom runtimes directly from the runtime image.
* **Cross-platform linking requires external JMODs** (downloaded separately).

### The Problem

* When `<targetJdk>/jmods` does not exist, `jlink` does **not** fail.
* Instead, it silently falls back to the **host** JDK's runtime image.
* This makes a "cross-platform" runtime appear to succeed, but the output is actually for the host platform.

This project reproduces that situation.

## How It Works

* A small Maven project configured with JReleaser's `jlink` assembler.
* The GitHub Actions workflow builds on Ubuntu with Temurin JDK 25.
* JReleaser is instructed to produce both **Linux x64** and **Windows x64** runtimes.

### Expected

* The Windows image should contain `.exe` binaries.
* The Linux image should contain ELF binaries.

### Actual

* Both images contain **Linux binaries only**.
* The Windows runtime directory is created, but its `bin/` folder has the same Linux executables as the Linux build.

## Reproducing Locally

1. Install JDK 24 or newer (Temurin recommended).

2. Run:

   ```bash
   ./mvnw clean package
   ```

3. Inspect the generated runtimes:

   ```bash
   ls -la target/jreleaser/assemble/jlinktest/jlink/work-linux-x86_64/jlinktest-linux-x86_64/bin
   ls -la target/jreleaser/assemble/jlinktest/jlink/work-windows-x86_64/jlinktest-windows-x86_64/bin
   ```

   Both directories will show the same binaries, confirming the issue.

## GitHub Actions

The included [workflow](.github/workflows/build.yml) shows the `bin/` contents of both Linux and Windows runtime images.
