<<<<<<< HEAD
# Strangler Fig Pattern Demo

## Build applications

Before being able to spin up the docker-compose based demo environment please make sure to successfully **build all 3 projects - the petclinic monolith based on Spring, the Apache Kafka Streams application and the owner microservice powered by Quarkus:**

```
./build-applications.sh
```

## Run with Docker

Spin up the demo environment by means of Docker compose:

```
docker-compose up
```

## Strangler Fig Proxy

This demo uses nginx and depending on the request, routes either towards petclinic monolith or the owner microservice.

1. The proxy serves a static index page at `http://localhost` just to verify it's up and running.

2. The initial proxy configuration - see `docker/.config/nginx/nginx_initial.conf` - routes all requests starting with `http://localhost/petclinic` to the monolithic application.

3. As a first adaption, the proxy is reconfigured - see `docker/.config/nginx/nginx_read.conf` - to route a specific read request i.e. the owner search from the monolith to the corresponding microservice.

_NOTE: In order for this to work the CDC setup (from monolith -> microservice) needs to be configured properly based on Apache Kafka Connect (see below)_

4. As a second adaption, the proxy is reconfigured - see `docker/.config/nginx/nginx_initial.conf` - to route owner edit requests to the owner microservice as well.

_NOTE: In order for this to work the CDC setup (from microservice -> monolith) needs to be configured properly based on Apache Kafka Connect (see below)_

## Proxy Reconfiguration

With the docker compose stack up and running, the proxy can be reconfigured by running the following simple script with one of the 3 support parameters:

```
./proxy_config.sh initial | read | read_write
```

## Apache Kafka Connect Setup for CDC Pipelines

### CDC from monolith (MySQL) -> microservice (MongoDB)

1. Create MySQL source connector for owners + pets tables:

```
http POST http://localhost:8083/connectors/ < register-mysql-source-owners-pets.json
```

2. Create MongoDB sink connector for pre-joined owner with pets aggregates:

```
http POST http://localhost:8083/connectors/ < register-mongodb-sink-owners-pets.json
```

## Apache Kafka Connect Setup for CDC Pipelines

### CDC from microservice (MongoDB) -> monolith (MySQL)

1. Before processing writes for owner data at the microservice in MongoDB update the MySQL source connector from the monolith database to ignore changes happening in the owners table. Otherwise we would try to do CDC for the same data from both sides which would lead to a propagation cycle.

```
http PUT http://localhost:8083/connectors/petclinic-owners-pets-mysql-src-001/config < update-mysql-source-owners-pets.json
```

2. Create MongoDB source connector  to capture data changes in the microservice

```
http POST http://localhost:8083/connectors/ < register-mongodb-source-owners.json
```

3. Create MySQL JDBC sink connector  to propagate changes from the microservice into the monolith's database

```
http POST http://localhost:8083/connectors/ < register-jdbc-mysql-sink-owners.json
```

## Consume messages from CDC-related Apache Kafka topics

```
docker run --tty --rm \
    --network mdblive21-demo_default \
    debezium/tooling:1.1 \
    kafkacat -b kafka:9092 -C -t mysql1.petclinic.owners -o beginning -q | jq .
```

```
docker run --tty --rm \
    --network mdblive21-demo_default \
    debezium/tooling:1.1 \
    kafkacat -b kafka:9092 -C -t mysql1.petclinic.pets -o beginning -q | jq .
```

```
docker run --tty --rm \
    --network mdblive21-demo_default \
    debezium/tooling:1.1 \
    kafkacat -b kafka:9092 -C -t kstreams.owners-with-pets -o beginning -q | jq .
 ```

 ```
docker run --tty --rm \
    --network mdblive21-demo_default \
    debezium/tooling:1.1 \
    kafkacat -b kafka:9092 -C -t mongodb.petclinic.kstreams.owners-with-pets -o beginning -q | jq .
```
=======
# Strangler Fig Pattern



## Getting started

To make it easy for you to get started with GitLab, here's a list of recommended next steps.

Already a pro? Just edit this README.md and make it your own. Want to make it easy? [Use the template at the bottom](#editing-this-readme)!

## Add your files

- [ ] [Create](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#create-a-file) or [upload](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#upload-a-file) files
- [ ] [Add files using the command line](https://docs.gitlab.com/ee/gitlab-basics/add-file.html#add-a-file-using-the-command-line) or push an existing Git repository with the following command:

```
cd existing_repo
git remote add origin https://git.clever-age.net/oxiane/gitlab-oxiane/contenu/strangler-fig-pattern.git
git branch -M main
git push -uf origin main
```

## Integrate with your tools

- [ ] [Set up project integrations](https://git.clever-age.net/oxiane/gitlab-oxiane/contenu/strangler-fig-pattern/-/settings/integrations)

## Collaborate with your team

- [ ] [Invite team members and collaborators](https://docs.gitlab.com/ee/user/project/members/)
- [ ] [Create a new merge request](https://docs.gitlab.com/ee/user/project/merge_requests/creating_merge_requests.html)
- [ ] [Automatically close issues from merge requests](https://docs.gitlab.com/ee/user/project/issues/managing_issues.html#closing-issues-automatically)
- [ ] [Enable merge request approvals](https://docs.gitlab.com/ee/user/project/merge_requests/approvals/)
- [ ] [Set auto-merge](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html)

## Test and Deploy

Use the built-in continuous integration in GitLab.

- [ ] [Get started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/index.html)
- [ ] [Analyze your code for known vulnerabilities with Static Application Security Testing(SAST)](https://docs.gitlab.com/ee/user/application_security/sast/)
- [ ] [Deploy to Kubernetes, Amazon EC2, or Amazon ECS using Auto Deploy](https://docs.gitlab.com/ee/topics/autodevops/requirements.html)
- [ ] [Use pull-based deployments for improved Kubernetes management](https://docs.gitlab.com/ee/user/clusters/agent/)
- [ ] [Set up protected environments](https://docs.gitlab.com/ee/ci/environments/protected_environments.html)

***

# Editing this README

When you're ready to make this README your own, just edit this file and use the handy template below (or feel free to structure it however you want - this is just a starting point!). Thank you to [makeareadme.com](https://www.makeareadme.com/) for this template.

## Suggestions for a good README
Every project is different, so consider which of these sections apply to yours. The sections used in the template are suggestions for most open source projects. Also keep in mind that while a README can be too long and detailed, too long is better than too short. If you think your README is too long, consider utilizing another form of documentation rather than cutting out information.

## Name
Choose a self-explaining name for your project.

## Description
Let people know what your project can do specifically. Provide context and add a link to any reference visitors might be unfamiliar with. A list of Features or a Background subsection can also be added here. If there are alternatives to your project, this is a good place to list differentiating factors.

## Badges
On some READMEs, you may see small images that convey metadata, such as whether or not all the tests are passing for the project. You can use Shields to add some to your README. Many services also have instructions for adding a badge.

## Visuals
Depending on what you are making, it can be a good idea to include screenshots or even a video (you'll frequently see GIFs rather than actual videos). Tools like ttygif can help, but check out Asciinema for a more sophisticated method.

## Installation
Within a particular ecosystem, there may be a common way of installing things, such as using Yarn, NuGet, or Homebrew. However, consider the possibility that whoever is reading your README is a novice and would like more guidance. Listing specific steps helps remove ambiguity and gets people to using your project as quickly as possible. If it only runs in a specific context like a particular programming language version or operating system or has dependencies that have to be installed manually, also add a Requirements subsection.

## Usage
Use examples liberally, and show the expected output if you can. It's helpful to have inline the smallest example of usage that you can demonstrate, while providing links to more sophisticated examples if they are too long to reasonably include in the README.

## Support
Tell people where they can go to for help. It can be any combination of an issue tracker, a chat room, an email address, etc.

## Roadmap
If you have ideas for releases in the future, it is a good idea to list them in the README.

## Contributing
State if you are open to contributions and what your requirements are for accepting them.

For people who want to make changes to your project, it's helpful to have some documentation on how to get started. Perhaps there is a script that they should run or some environment variables that they need to set. Make these steps explicit. These instructions could also be useful to your future self.

You can also document commands to lint the code or run tests. These steps help to ensure high code quality and reduce the likelihood that the changes inadvertently break something. Having instructions for running tests is especially helpful if it requires external setup, such as starting a Selenium server for testing in a browser.

## Authors and acknowledgment
Show your appreciation to those who have contributed to the project.

## License
For open source projects, say how it is licensed.

## Project status
If you have run out of energy or time for your project, put a note at the top of the README saying that development has slowed down or stopped completely. Someone may choose to fork your project or volunteer to step in as a maintainer or owner, allowing your project to keep going. You can also make an explicit request for maintainers.
>>>>>>> refs/remotes/origin/main
