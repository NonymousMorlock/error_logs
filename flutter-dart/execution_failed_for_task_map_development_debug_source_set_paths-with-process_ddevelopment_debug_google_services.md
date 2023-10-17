
## PROBLEM

```commandline
Launching lib/main_development.dart on SM A125F in debug mode...
Running Gradle task 'assembleDevelopmentDebug'...

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:mapDevelopmentDebugSourceSetPaths'.
> Error while evaluating property 'extraGeneratedResDir' of task ':app:mapDevelopmentDebugSourceSetPaths'
   > Failed to calculate the value of task ':app:mapDevelopmentDebugSourceSetPaths' property 'extraGeneratedResDir'.
      > Querying the mapped value of provider(java.util.Set) before task ':app:processDevelopmentDebugGoogleServices' has completed is not supported

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 21s
Exception: Gradle task assembleDevelopmentDebug failed with exit code 1

```

This happened when I tried to run the project for android, but was totally okay when I ran it for web.


## SOLUTION
I simply went to the firebase console, and went to project overview >> settings.
Next I clicked on the android config and went to `See SDK Instructions`, then I checked the 
`google-services` version, which was 4.4.0, meanwhile, the one in my project-level `build.gradle` file
was 4.3.10, so, I simply changed the one in my project-level `build.gradle` to 4.4.0 and that solved it.

