# SonarQube + Docker + M1

[<img src="https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white">](https://hub.docker.com/r/davealdon/sonarqube-with-docker-and-m1-macs)

[![](https://img.shields.io/badge/SonarQube-v9.3.0.51899-blue)](https://img.shields.io/badge/SonarQube-v9.3.0.51899-blue)
##### TL;DR:

Pull down the docker build and run it:
```
docker pull davealdon/sonarqube-with-docker-and-m1-macs
```

Specify a port number so that you can go to the Sonar URL: http://localhost:9000/

#### OR

Clone the repo on your M1 Mac, and run the following command:
```
docker build -f arm64.Dockerfile -t sonarqubem1 .
docker run -p9000:9000 sonarqubem1
```
Then go to the URL: http://localhost:9000/

### Tell me more!

![](https://c.tenor.com/Z1o2HxFnHy4AAAAC/tell-me-more-michael-scott.gif)

<details>
<summary>What is SonarQube?</summary>
SonarQube is a great static code analysis tool. A lot of the time, you'll encounter SonarCloud, which is a cloud-based version of SonarQube. It's usually added to a CI/CD pipeline, which means you might have to be patient to get the analysis done on your code, and at that point you've already committed your work. What if you wanted to run SonarQube locally, and get instant results before committing? This is where Docker comes in.
</details>
<details>
<summary>What is Docker?</summary>
Docker is a tool that simplifies the process of building and running software in containerized environments. It does this by virtualizing an operating system for specialized tasks/applications you want to run. You could have three different versions of some program that you need to run, and with Docker you could have three different containers that have each of your special setups ready to go.
</details>
<details>
<summary>What is M1?</summary>
Apple has started creating their own processors, called M1. They're built using the arm architecture, instead of previously Intel, which uses x86. This change requires native rewrites of applications to work, or translation using Rosetta 2.0 to virtualize the x86 architecture for M1 arm based processors.
</details>
<br>
Combining all three of these technologies is the reason for this repository. But it wasn't that easy.

#### SonarQube has a Docker guide that doesn't work on M1 Macs

You can find their nice, 2 minute guide [here](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/). But, on an M1 you can only run it locally using the source zip file. I _really_ wanted to use Docker, which is their preferred method anyway.

It didn't work out of the box because you immediately get this error when building:
```
no matching manifest for linux/arm64/v8 in the manifest list entries
```

So, they don't have the right manifest for this architecture. You can force it to run in x86_64 mode:
```
docker run --platform linux/x86_64 sonarqube 
```

But surprise, surprise, of course that doesn't work. You'll get a big error dump talking about Elasticsearch failing and other stuff:
```
unable to install syscall filter
java.lang.UnsupportedOperationException: seccomp unavailable: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
etc. etc.
```
After further investigation, it turns out that this is something SonarQube isn't really working on, but there's [definitely requests](https://jira.sonarsource.com/browse/CPP-2882) for it.

After SonarQube gave the simple response that "we don't support M1" another developer [created a solution](https://github.com/SonarSource/docker-sonarqube/issues/475). This was great, but after trying to pull down *mwizner*'s Docker, I got this error:
```
Error response from daemon: manifest for mwizner/sonarqube:latest not found: manifest unknown: manifest unknown
```
Also, [mwizner's solution](https://hub.docker.com/r/mwizner/sonarqube) was for v8 of SonarQube, from almost a year ago, and I'm greedy and wanted the latest v9 to work (which matches the SonarQube version my team is using).

### The Working Solution

A big thanks to my colleague [Chris Watts](https://github.com/cj-watts) for helping me with this, because I'm a beginner at Docker and was going down the wrong path of trying to containerize SonarQube's source web app manually. I knew the source code worked without Docker for sure, but I didn't understand how to get Docker to work with specific architectures.

[This file](https://github.com/sonar-scala/docker-sonarqube/blob/arm64/8/community/arm64.Dockerfile) was the starting point, provided by mwizner's original solution. Chris provided changes to this in some key places:

1. The version of the architecture the Docker build will use:
```
FROM arm64v8/alpine:3.14.3
and
ZLIB_URL="https://mirrors.dotsrc.org/archlinuxarm/arm/core/zlib-1%3A1.2.11-5-arm.pkg.tar.xz";
```

2. The latest version of SonarQube:
```
ARG SONARQUBE_VERSION=9.3.0.51899
```

3. Many frameworks expect the app to be stopped using SIGINT, or Control-C. When you run the command `Docker stop`, you're telling Docker to send a signal to the container to stop. By default, this command is `SIGTERM`, but some apps may be configured to listen to something else like `SIGUSR1`. Therefore, this command was added to manually make sure that `SIGINT` is sent to the container upon stopping. This is more of a QoL update, and is [explained more here](https://docs.docker.com/engine/reference/commandline/stop/).
```
STOPSIGNAL SIGINT
```

4. Added updated entrypoint and command paths to the Dockerfile:
```
ENTRYPOINT ["/opt/sonarqube/bin/run.sh"]
CMD ["/opt/sonarqube/bin/sonar.sh"]
```

5. Our new run command uses the -p flag that tells Docker to bind the container's port to your computer's port, allowing us access to the web app:
```
docker run -p9000:9000 sonarqubem1
```

### Credits
Once again, thank you to [Chris Watts](https://github.com/cj-watts) for helping me with this! If you want to talk with us about Docker, check out Bravo LT's Discord server [here](https://discord.gg/84eWHK26CU)!

### Download links
[Docker (Choose Mac with Apple Chip for M1)](https://docs.docker.com/get-started/#download-and-install-docker)

[SonarQube](https://www.sonarqube.org/downloads/)
