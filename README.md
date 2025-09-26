# JEP 493 Jlink Cross-Platform Reproducer

See the [main README](https://github.com/felipebz/jreleaser-jlink-jep493/blob/main/README.md) for more details.

## Workaround implemented in this branch

To force cross-platform `jlink` to use the correct Windows JMODs with JDKs that ship without a `jmods/` directory (JEP
493), this project applies the following workaround in pom.xml:

1) Download Linux JDK and Windows JMODs with JReleaser jdks-maven-plugin

    - setup-disco downloads the Linux Temurin JDK archive (contains the Linux runtime used to link the Linux image).
    - setup-jdks downloads the Windows Temurin JMODs zip

2) Create a minimal "fake" JAVA_HOME with the Windows JMODs

    - Rename folder `target/jdks/windows/jdk-25+36-jmods` to `target/jdks/windows/jmods`.
    - Create `target/jdks/windows/release` containing `JAVA_VERSION="25"`. JReleaser checks for a release file at the
      JDK root; since we only have JMODs, we generate a minimal one so the directory is accepted as a targetJdk.

3) Point JReleaser jlink assembler to these paths

    - Linux: target/jdks/linux/jdk-25+36
    - Windows: target/jdks/windows

## Why this works

jlink only needs the target platform JMODs for cross-linking. It does not require the full target runtime image. By
feeding the Windows JMODs and a minimal release marker, JReleaser recognizes the directory as a valid target JDK and
passes its jmods to jlink, producing proper Windows images on Linux.
