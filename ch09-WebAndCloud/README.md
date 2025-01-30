# 9. JavaFX, the Web, and Cloud Infrastructure
## JavaFX Installation
https://gluonhq.com/products/javafx/
## Environment
1. Export these environment variables:
    ```bash
    set JAVA_HOME="C:\Users\<user>\Downloads\jdk-17"
    set PATH_TO_FX="C:\Users\<user>\Downloads\javafx-sdk-17\lib"
    set PATH_TO_FX_MODS="C:\Users\<user>\Downloads\javafx-jmods-17"
    ```
    Note that if you have a different JDK added to the PATH environment variable, this will take precedence. 
## JavaFX References 
* https://openjfx.io/openjfx-docs/#maven
### Maven
```bash
mvn clean javafx:run
```