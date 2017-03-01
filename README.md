# Docker Image for running Android Espresso tests on Fire Base Test Lab
---

This Docker image extends the image [carlospaulino/android-builder](https://github.com/carlospaulino/android-builder) by installing Google Cloud Tools in order to support for Firebase Test Lab.

# Contains

* Google Cloud Tools
* Oracle JDK 8
* Android SDK
* Android NDK
* Android Support Libraries
* Google Play Services
* Latest Build and Platform Tools

See [here](https://github.com/carlospaulino/android-builder/blob/master/android-packages) for the full list of Android dependencies.

# Caveats:

It’s important to highlight that this Docker image doesn’t authorize the Google Cloud Tools. You have to do it manually by following their guide [here](https://cloud.google.com/sdk/docs/authorizing).

# Tips:

If you are using this image on a CI environment such as Travis CI or Bitbucket Pipelines you could setup Google Cloud Tools and run the Espresso tests on Firebase Test Lab using a few custom gradle tasks.

#### For example:

You could expose an environmental variable containing your [service account key]( https://console.cloud.google.com/permissions/serviceaccounts) (GCLOUD_KEY) encoded in base64 and another one with the project id (PROJECT_ID).

```
task(setupFtl) {
    doLast {
        if (!System.getenv('GCLOUD_KEY') || !System.getenv('PROJECT_ID')) {
            throw new IllegalStateException("Missing env vars GCLOUD_KEY or PROJECT_ID")
        }

        new File("$rootProject.rootDir/gcloud-key.json").write(new String(System.getenv('GCLOUD_KEY').decodeBase64()), "UTF-8")

        exec {
            commandLine(['gcloud', 'auth', 'activate-service-account', '--key-file', "$rootProject.rootDir/gcloud-key.json"])
        }
        exec {
            commandLine(['gcloud', 'config', 'set', 'project', System.getenv('PROJECT_ID')])
        }
    }
}
```

Then you can create another task to run the tests.

```
task(runFtl) {
    doLast {
        exec {
            commandLine([
                    'gcloud', 'beta', 'test', 'android', 'run',
                    '--type', 'instrumentation',
                    '--app', "$rootProject.rootDir/app/build/outputs/apk/app-debug.apk",
                    '--test', "$rootProject.rootDir/app/build/outputs/apk/app-debug-androidTest.apk",
                    '--device-ids', 'Nexus5',
                    '--os-version-ids', '23',
                    '--locales', 'en',
                    '--orientations', 'portrait'
            ])
        }
    }
}

runFtl.dependsOn "setupFtl"
runFtl.dependsOn ":app:assembleDebug"
runFtl.dependsOn ":app:assembleAndroidTest"
```

# Notes
* In the previous example the apk path is hardcoded, but you can figure out the path in your project by hooking to the android gradle plugin.
* Read the [Firebase Test Lab documentation](https://firebase.google.com/docs/test-lab/command-line) for more clarity on how to setup and run the test matrix.

# License

Copyright (c) 2017 Carlos Paulino <cpaulino@gmail.com>.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.