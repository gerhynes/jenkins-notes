# Jenkins Notes
Sources
- [What is CI/CD?](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
- [What Is Jenkins?](https://youtu.be/LFDrDnKPOTg)
- [Jenkins Tutorial](https://www.youtube.com/watch?v=f4idgaq2VqA)

## What is CI/CD?
Continuous Integration / Continuous Delivery/Deployment is a method to frequently deliver applications to customers by automating stages of the application development. CI/CD introduces ongoing automation and continuous monitoring throughout the lifecycle of apps, from integraton and testing to delivery and deployment. 

Taken together, these practices form a CI/CD pipeline supported by teams working with either a DevOps or Site Reliability Engineering approach. 

Continuous Integration means that new code changes to an app are regularly built, tested and merged to a shared repository.

Continuous Delivery usually means a developer's changes to an app are automatically tested and uploaded to a repository where they can be deployed to a live production environment by the operations team.

Continuous Deployment refers to automatically releasing a developer's changes from the repository to production. It builds on continuous delivery by automating the next stage in the pipeline.

Continuous integration helps developers merge their code changes back to a shared branch more frequently, sometimes daily. These changes are validated by automatically building and testing the app to ensure they have not broken the app.

Continuous delivery builds on CI by automating the release of the validated code to a repository. The goal of CD is to have a codebase that is always ready for deployment to a production environment.

Continuous deployment automates releasing an app to production. In practice, CD means a developer's changes to a cloud app could go live withi minutes, assuming it passes automated testing.

### CI Tools
Bamboo is a CI tool that runs multiple builds in parallel for faster compilation.

Buildbot is an open-surce framework for automating software build, test and release processes. It supports distrbuted, parallel execution of jobs aross multiple platforms.

Apache Gump is designed to build and test Java projects, nightly. It makes sure that the projects are compatible at both the API level and functionality level.

Travis CI is a hosted, distributed CI service used to build and test projects hosted  on GitHub. It supports over 20 languages and teams of all sizes.

## Jenkins
Jenkins is an open source automation server, which helps automate building, testing and deploying software, facilitating CI/CD.

Built on the JVM, it provides more than 1,800 plugins to support practically any technology that software delivery teams use.

Jenkins is a self-contained Java program, ready to run on different OSes. It can be easily set up and configured via its web interface, which includes built-in error. It is easily extensible via plugins. It can distribute work across multiple machines, helping to speed up building, testing and deployment across multiple platforms.

## Jenkins Pipeline
The first stage in the Jenkins pipeline is to commit your code.

Jenkins will create a build of your code, running a standard set of tests.

If the tests pass, the code is moved to a release-ready environment.

Finally, Jenkins will deploy your code to a production environment.

## Jenkins Architecture
The Jenkins CI server checks the repository at regular intervals and pulls any newly available code. 

Jenkins uses a build-server, such as Maven, to build the code into an executable file. If the build fails, feedback is sent to the developers.

Jenkins then deploys the built application to a test server, for example using Selenium, for testing. If the tests fail, feedback is sent to the developers.

If there are no errors, the tested application is then deployed to the production server.

Jenkins uses a distributed architecture to execute multiple builds.

Jenkins has a "master-slave" architecture. The Jenkins master server pulls the code every time there is a commit. It distributees its workload to all the slaves. On request from the Jenkins master, the slaves carry out builds and tests and produce test reports.    

## Jenkinsfile
Instead of creating jobs via the Jenkins GUI, Jenkinsfile lets you configure a build. It's a scripted pipeline and part of the concept of infrastructure as code. It's stored in your repository with your code.

A Jenkins file can be written as a scripted pipeline or a declarative pipeline.

Scripted - the original syntax, uses Groovy syntax. Has advanced scripting capabilities and is highly flexible.

Declarative - a more recent addition, easier to get started with but not as powerful. It has a pre-defined structure.

A descriptive Jenkinsfile will follow this general format.

```Jenkinsfile
pipeline {

	agent any
	
	stages {
		stage("build") {
			steps {
			}
		}
	}
}
```

"pipeline" must be top-level. 

"agent" speifies where to execute the build.

"stages" is where the work happens. Individual stages, such as "build" and "test", can have multiple "steps". Each step contains a script that executes some command on the Jenkins agent.

In the Jenkins UI, you specify a source and enter credentials. Jenkins will look for a Jenkinsfile and execute it.

At the end of the Jenkinsfile, you can define a "post" attribute which will execute some logic after all stages have been executed.

### post

Inside  "post", you can use different conditions, such as "always" for logic that you want to always be executed. This could be used for example to notify the team if the build succeeded or failed. With the "success" and "failure" conditions, the logic will only be executed if the build completed successfully or failed. 

The "post" block usually handles conditions based on build status or build status change.

### conditionals
You can define conditionals for each stage. For example, you might only want to run the tests on the development branch build. 

The current branch namme in the build is always avalilable through the `BRANCH_NAME` environmental variable which is provided by Jenkins.

```Jenkinsfile
stage("build"){
	when {
		expression {
			BRANCH_NAME == "dev" && CODE_CHANGES == true
		}
	}
	steps {
		echo "building the app..."
	}
}
stage("test"){
	when {
		expression {
			BRANCH_NAME == "dev" || BRANCH_NAME == "master" 
		}
	}
	steps {
		echo "testing the app..."
	}
}
```

### Environmental Variables
Your Jenkins environmental variables are available at `JENKINS_URL/env-vars.html`.

You can also define your own environmental variables using the "environment" attribute.

```Jenkinsfile
pipeline {
	environment {
		NEW_VERSION = "1.3.0"
	}
	stages {
		stage("build"){
			steps {
				echo "building version ${NEW_VERSION}"
			}
		}
	}
}
```

You may need environmental variables for credentials.

You would define credentials in the Jenkins UI.

Once you install the Credentials and Credentials Binding plugin, you can use `credentials("credentialId")` to bind the credentials to an environmental variable.

```Jenkinsfile
pipeline {
	environment {
		SERVER_CREDENTIALS = credentials("server-credentials")
	}
	stages {
		stage("build"){
			steps {
				sh "${SERVER_CREDENTIALS}"
			}
		}
	}
}
```

If you only need credentials for one stage, it makes sense to only read the credentials in that stage.

```Jenkinsfile
stage("deploy"){
	steps {
		echo "deploying the app..."
		withCredentials([
			usernammePassword(credentials: "server-credentials", usernameVariable: USER, passwordVariable: PWD)
		]) {
			sh "some script ${USER} ${PWD}"
		}
	}
}
```

### tools
The "tools" attribute provides you with access to build tools for your project, such as Maven, Gradle, jdk. 

The tool needs to be installed in Jenkins. You can check this under "Global Tool Configuration".

```Jenkinsfile
pipeline {
	agent any
	tools {
		maven "Maven"
	}
	stages {
		stage("build"){
			steps {
				sh "mvn install"
			}
		}
	}
}
```

### Parameters
You might have some external configuration that you want to provide to your build. For example, being able to select which version of the application you want to deploy to a staging server.

You will see a "Build with Parameters" option in the Jenkins UI.

```Jenkinsfile
pipeline {
	agent any
	parameters {
		string(name: "VERSION", defaultValue: "", description: "version to deploy on prod")
		choice(name: "VERSION", choices: ["1.1.0", "1.2.0", "1.3.0"], description: "")
		booleanParam(name: "executeTests", defaultValue: true, description: "")
	}
	stages {
		stage("test"){
			when {
				expression {
					params.executeTests == true
				}
			}
			steps {
				echo "testing the app..."
			}
		}
	}
}
```

### Using external Groovy scripts
Your Jenkinsfile can quickly become overloaded with scripts. It can be a good idea to put these scripts in a seperate file.

Jenkins supports Groovy scripts using "script" blocks. All environmental variables in the Jenkinsfile are also available in the Groovy scripts.

```Groovy
def buildApp() {
	echo "building version ${VERSION}"
}

return this
```

```Jenkinsfile
def gv

stage("init"){
	steps {
		script {
			gv = load "script.groovy"
		}
	}
}
stage("build"){
	steps {
		script {
			gv.buildApp()
		}
	}
}
```