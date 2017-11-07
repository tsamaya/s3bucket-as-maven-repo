# s3bucket-as-maven-repo

How to use S3 Bucket as a gradle/maven repository

## prerequisites

- java (1.8)
- gradle (3.5.1)
- an AWS account with an S3 Bucket to use

## get started

- clone the repository
- `$ cd /path/to/repo`

- or create a new gradle project with:
  * `$ gradle init --type java-library`
  * then edit the gradle file

- assuming that your S3 Bucket is not public, run once  `$ aws configure` to create your `credentials` file (see AWS [documentation](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html))

## editing gradle file
Edit `build.gradle` file assuming below you started from a brand new project

  * add `maven-publish` plugin to be able to publish into the S3 Bucket

  ```groovy
  apply plugin: 'maven-publish'
  ```

  * specify your project properties

  ```groovy
  version = '0.1.0-SNAPSHOT'
  sourceCompatibility = 1.8

  def mvnGroupId = 'net.tsamaya'
  ```

  * specify your S3 Butcket (long) URL

  ```groovy
  def mvnReleasesURL = "s3://my-artifacts.s3.eu-west-2.amazonaws.com/m2/releases"
  def mvnSnapshotsURL = "s3://my-artifacts.s3.eu-west-2.amazonaws.com/m2/snapshots"
  ```

  NB: split releases and snapshots at your convenience (here: m2/releases and m2/snapshots)

  * if you want to publish the source files, add :

  ```groovy
  task sourceJar(type: Jar) {
      from sourceSets.main.allJava
  }
  ```

  * Edit the `repositories` section, to add the maven repositories for snapshots and releases:

  ```groovy
  // In this section you declare where to find the dependencies of your project
  repositories {
      // Use jcenter for resolving your dependencies.
      // You can declare any Maven/Ivy/file repository here.
      maven {
          url mvnReleasesURL
          credentials(AwsCredentials) {
              accessKey AWS_ACCESS_KEY
              secretKey AWS_SECRET_KEY
          }
      }

      maven {
          url mvnSnapshotsURL
          credentials(AwsCredentials) {
              accessKey AWS_ACCESS_KEY
              secretKey AWS_SECRET_KEY
          }
      }

      mavenLocal()
      jcenter()
  }
  ```

  * now you can pull your dependencies from your S3 Bucket

  * for publishing add a `publishing` section

  ```groovy
  publishing {
      repositories {
          maven {
              if(project.version.endsWith('-SNAPSHOT')) {
                url mvnSnapshotsURL
              } else {
                url mvnReleasesURL
              }
              credentials(AwsCredentials) {
                  accessKey AWS_ACCESS_KEY
                  secretKey AWS_SECRET_KEY
              }
          }
      }

      publications {
          mavenJava(MavenPublication) {
              groupId mvnGroupId
              artifactId rootProject.name
              version version

              from components.java

              artifact sourceJar {
                  classifier "sources"
              }
          }
      }
  }
  ```

  * now you can publish the project's artifact into the S3 Bucket

  * `$ gradle publish`

## Issues
Find a bug or want to request a new feature?  Please let me know by submitting an issue.

## Licensing
Licensed under the MIT License

A copy of the license is available in the repository's [LICENSE](LICENSE.md) file.
