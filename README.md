# Rabble Rouser

This is the best place to get started developing code for Rabble Rouser. As the project consists of many small components,
the code is spread across lots of [repositories](https://github.com/rabblerouser). Rather than making you clone them all
and install their dependencies individually before you can even get started, this repository contains a [Cage](http://cage.faraday.io)
project to help make things easier. Cage allows you to start by running everything from pre-built Docker images, and
then check out the source code for only the components you want to make changes to.

It is our hope that this will make it fairly painless to get started with Rabble Rouser. If there is any way we can help
make it a better experience for you, please let us know by opening an issue on this repository, or talking to one of the
team members.

## Dependencies

1. [Docker](https://store.docker.com/search?type=edition&offering=community)
2. docker-compose: For most systems this is installed automatically with Docker.
3. [Cage](http://cage.faraday.io/setup) (Windows users [see here](https://github.com/faradayio/cage/blob/master/WINDOWS.md))

## Run the application

From this repository, run:

```sh
cage pull
cage up
cage run seed
cage restart backend #TODO: Make it so this is not required
```

This will pull down the latest Docker images for the project, start up all the various services, and then seed the app
with the minimum data needed to get started.

## Test it out

1. Open up your browser and head to http://localhost:8080
2. Register a new member - make sure you see a success message
3. Head to http://localhost:8080/dashboard and log in - you should see your newly registered member
    - Email: `superadmin@rabblerouser.team`
    - Password: `password1234`

## Developing an application

Let's make a small change to the UI and see the result.

### Check out some source code

Start with this:

```sh
cage source mount core
cage run core npm install
cage up
```

The first command will clone the [`core`](https://github.com/rabblerouser/core) application, so we can work on it. It
also tells cage that you'd like to run this app from source, rather than just using a pre-built Docker image.
The second command installs all of the core app's dependencies, so that we can build and run it locally.
The final one just tells cage to bring everything back up with the changes we made.

### Make a change

You can make any visible change you like here. Perhaps open up the file `frontend/src/signup/components/DetailsForm.js`
and change one of the field labels to something different. Once you save the file, the frontend will automatically
rebuild itself, the page will refresh, and you should see your changes!

Note: You'll find the source code cloned into `./src/core/`.

## Putting things back

If you're no longer working on a service, you might want to unmount the source code and set it back to pre-built Docker
image mode. It's probably also a good idea to pull the latest image:

```sh
cage source unmount core
cage pull core
```

## Running tests
The best way to run the tests of a particular project is to shell into the running container and run the tests from
inside. For example:

```sh
cage source mount core
cage shell core
# You'll now have a new session, inside the docker container, with the source mounted from your host machine
cd frontend
npm test
```

This can be used to run any other kind of development and debugging tasks, keeping everything isolated inside the Docker
container.

## Other useful commands

These cage commands may come in handy while developing:

- `cage status`: see the status of all services in the project
- `cage logs -f <service-name>`: tail the logs of the given application or pod
- `cage`: Run cage with no arguments to see everything it can do!

## TODO
- [x] common
  - [x] kinesis
  - [x] S3
  - [x] SES
  - [x] archiver
- [x] core
  - [x] backend
  - [x] frontend
  - [x] forwarder
  - [x] seeder
- [x] mailer
  - [x] app
  - [x] forwarder
- [ ] group-mailer
  - [ ] app
  - [ ] forwarder
- [ ] group-mail-receiver
  - [ ] lambda
  - [ ] something something SES
- [ ] Run all tests maybe?
