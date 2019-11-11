# MobSF CI

This repo contains all the is required to run [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) in the CI.
MobSF is a security tool that can scan APK/IPA and report various security issues.
By running it in the CI, you can find those issues earlier, and fix them. To learn more about what it MobSF and what it can detect, checkout the [blog post](https://medium.com/@omerlh/how-to-continuously-hacking-your-app-c8b32d1633ad).

## Architecture

There are two docker containers and one docker app used:

1. `omerl/mobsf-ci` : This container image is created using Dockerfile in the `scan` folder. This container image is built only once. The container holds the scanning logic: wait for the mobsf server container to start at port 8000, upload the apk to the server after it starts and initiate the scan. After the scan is done, report is saved at `output/report.json`.
2. `opensecurity/mobile-security-framework-mobsf` : This is the official container image of MobSF. This acts as the MobSF server and opens the web app at port 8000.
3. `omerl/mobsf-ci.dockerapp` : This is a [docker app](https://github.com/docker/app). The source is stored at `docker-app` directory. This is used to create MobSF environment from scratch for every scan. This pulls both the above container images, sets up the scanning environment, initiates the scan for the apk file you provide and returns the result in json file.

## Docker App

The easiest way to use this repo is by using [docker app](https://github.com/docker/app). Simply run:
```
docker-app render omerl/mobsf-ci:0.3.0 --set target_folder=<path to the folder that contains the APK> --set target_apk=<apk name> --set output_folder=<path to folder where the report will be written> | docker-compose -f - up --exit-code-from scan
```

To parse the report, use Glue - see in the next section how.

## Usage

* Clone the repo
* Create a folder named `target` in the root folder, and place the target there (e.g. `target/my_app.apk`).
* Run the tests using:
```
TARGET_PATH='target/<name of the target>' docker-compose up --build --exit-code-from scan
```
* Wait for the command to complete, it will take some time. When the command will be completed, checkout the report under `output/report.json`.
* Use [OWASP Glue](https://github.com/OWASP/glue) to process the report by running:
```
docker run -it -v $(pwd)/output:/app owasp/glue:raw-latest ruby bin/glue -t Dynamic -T /app/report.json --mapping-file mobsf --finding-file-path /app/android.json -z 2
```