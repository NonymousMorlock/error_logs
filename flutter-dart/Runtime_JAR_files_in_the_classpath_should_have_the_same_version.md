## PROBLEM

```commandline
w: Runtime JAR files in the classpath should have the same version. These files were found in the classpath:
    /home/akundadababalei/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.5.30/5fd47535cc85f9e24996f939c2de6583991481b0/kotlin-stdlib-jdk8-1.5.30.jar (version 1.5)
    /home/akundadababalei/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.6.10/e1c380673654a089c4f0c9f83d0ddfdc1efdb498/kotlin-stdlib-jdk7-1.6.10.jar (version 1.6)
    /home/akundadababalei/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.6.10/b8af3fe6f1ca88526914929add63cf5e7c5049af/kotlin-stdlib-1.6.10.jar (version 1.6)
    /home/akundadababalei/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.6.10/c118700e3a33c8a0d9adc920e9dec0831171925/kotlin-stdlib-common-1.6.10.jar (version 1.6)
w: Some runtime JAR files in the classpath have an incompatible version. Consider removing them from the classpath
```

## SOLUTION

```bash
cd android
```

```bash
./gradlew clean
```
