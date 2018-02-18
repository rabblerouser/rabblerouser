# Rabble Rouser

Rabble Rouser consists of many small components, and the code is spread across lots of [repositories](https://github.com/rabblerouser).
This repository uses [Cage](http://cage.faraday.io) to manage all of the pieces and their relationships to each other,
hopefully making it much easier to get started quickly, and to work on features that span across multiple codebases.

It's important to us that the Rabble Rouser developer experience is really great. If you run into problems when trying
to work on the code, it's our fault, not yours! Please get in touch and help us make it better, either by opening an
issue on this repository, or talking to one of the team members.

> Throughout this guide, more in-depth technical information will appear like this. This is here for people who already
know a bit about Docker and want to really understand what's going on under the hood. You can safely ignore these bits
if you want.

## Dependencies

You need to install a couple of things first:

1. [Docker](https://store.docker.com/search?type=edition&offering=community)
2. docker-compose: For most systems this is installed automatically with Docker.
3. [Cage](http://cage.faraday.io/setup) >= v0.2.4 (Windows users [see here](https://github.com/faradayio/cage/blob/master/WINDOWS.md))

## Run the application

First clone this project into a new directory:

```sh
git clone git@github.com:camjackson/rabblerouser.git
```

Then go into the project and start everything up with a single command! (It might take a while the very first time.)

```sh
cd rabblerouser
cage up --init
```

The `--init` flag will write some seed data into the database, which will persist even after you tear everything down.
So you should only ever need it once, unless you manually delete the data files.

You can see a list of all of the services that have been started:

```sh
cage status
```

You should see a bunch of microservices, along with their dependencies, such as mocked AWS services.

> So what did that actually do? Firstly, Cage has pulled down pre-built docker images for each of the services that make
up the whole system. Some of these are our own dockerised applications, which we deploy into production, and some of
them are mocked dependencies, which allow us to have local instances of things like AWS Kinesis or S3. After pulling
down the docker images, cage starts up a bunch of docker containers and connects them all to each other over a local
docker network. When you do `cage status`, you're seeing a list of docker containers, grouped into logical 'pods'.

> If it's reminding you of docker-compose, you'd be correct - Cage is essentially a layer on top of
docker-compose that automates a bunch of common development tasks.

> Note that everything is running from pre-prepared docker images - no need to clone any source code yet!

## Try out Rabble Rouser!

1. Open up your browser and head to http://localhost:8080
2. Register a new member - make sure you see a success message
3. Head to http://localhost:8080/dashboard and log in - you should see your newly registered member
    - Email: `superadmin@rabblerouser.team`
    - Password: `password1234`

## Making your first code change

Let's make a small change to the UI and see the result.

### Check out some source code and mount it

So far we haven't downloaded any Rabble Rouser source code yet. If we want to make a change to, for example, the 'core'
application, we'll need to download its source, install its library dependencies, and restart Rabble Rouser:

```sh
cage source mount core
cage run core yarn install
cage up
```

Now open http://localhost:8080 again in your browser, and make sure the registration form still loads. If it doesn't
load, wait a minute and try again, as the application may still be compiling or starting up.

> The first command tells Cage that we want to run that particular application from local source code. If you don't
already have the code, then cage will clone it for you automatically! Next we need to install the app's dependencies -
we can use `cage run <app_name> <command>` for one-off commands like this. The installation will run inside a docker
container, but through the magic of volume-mounting, the dependencies will be written to your regular hard drive outside
the container.

> Mounting/unmounting does not modify containers that are already running, so you need to apply your changes by running
the final command, restarting the container. Now the app will be running inside a docker container just like before, but
this time it's running from the source code on your hard drive, so we can start making changes!

### Make a change

The source code has been downloaded to `rabblerouser/src/core`. Inside that directory, open up `frontend/src/signup/components/DetailsForm.js`
in your favourite text editor or IDE, so we can make a change to the member registration form. You should see some code
that looks something like `label="Full name"`; change it to say `label="What shall we call you?"`. Once you save the
file, the frontend will automatically rebuild itself, the browser page will refresh, and you should see your changes!

## Run the tests

Whenever we make a change, we should make sure that the tests still pass as well. First we need to start a new terminal
session (or 'shell') *inside* the application, and then we can run the tests from there. Once you're done, exit the
shell to get back to where you started.

```sh
cage shell core
yarn test
exit
```

> Here we're getting a shell inside the running container. Something equivalent to `docker exec -it core sh`. Inside the
container we have a fully isolated development environment which we can use for all kinds of development tasks, such
as running tests, linting or formatting code, managing application dependencies, etc. By doing all of this inside
containers, we don't have to worry about e.g. making sure the right version of Node.js is installed on everyone's
development machine.

## Putting things back

Earlier we used `cage source mount core` to download the source code, and also to tell Cage to switch the 'core'
application into source mode. If we're not working on core any more, then it's a good idea to switch off source mode,
putting our development environment back how it was at the start. We should also pull the latest published version of
the application. Then we need to restart the application again for those commands to take effect:

```sh
cage source unmount core
cage pull core
cage up
```

> `unmount` is just the reverse of `mount`, telling cage not to mount the source code inside the container when the
container next starts up. And `pull` simply pulls the latest version of the docker image from the docker hub. If you run
`cage pull` without any application name, it will pull the latest images for everything.

## Cleaning up

Once we're all done, we can shut down the applications, and clean up any resources they used:

```sh
cage stop
cage rm
```

This does *not* clear out the database, so next time you can start Rabble Rouser with just `cage up` (no `--init`), and
all your data from last time will still be there. For more information on where this data is, and how to inspect or
remove it, see the section below on the local AWS mocks.

> The above commands are basically the same as the `stop` and `rm` commands of both Docker and docker-compose. See their
docs for more information. One significant difference is that while `docker rm -f` will delete a running container in
one step, so you don't have to stop it first, `cage rm -f` simply means "don't ask for confirmation". As of cage v0.2.4,
the `rm` command will always ignore any containers that are still running, so you have to `stop` them first. You might
like to create a shell alias for something like: `docker ps -aq | xargs docker rm -f`.

## What's next?

The above workflow should give you most of the commands needed to start contributing to Rabble Rouser! You're now able
to run things locally, check out source code, make changes, and test them both manually and automatically. From here you
might want to dig a bit deeper into one of the exsting applications, you could learn more about the development
environment, or you could even try creating a new one! The next few subsections will cover each of these options.

### Contributing to an application

The first step would be to find something to work on. Head to https://huboard.com/rabblerouser/core to find our backlog,
and if you see something you want to tackle, leave a comment on the issue letting us know. Please ask for help if you
don't know where to get started!

Once you find an application to work on, look at its own docs to see how to contribute. There should be instructions for
installing its dependencies, running its tests, etc. If there aren't, then please raise an issue on GitHub so we can fix
it.

Once you've made your changes, you can open a pull request to have them merged into the main repository. GitHub itself
has more detailed guides for how to do this, but in short you need to:

- Fork the github repository you want to make changes to
- Point your local repository at your fork on github
- Push your commits up to your fork on github
- Open a pull request from your fork to the original repository.

If that all sounds terribly confusing, please reach out to the team and we'll help you out.

### Learning more about the development environment

While you've already learned the basics, there's still a lot more you can do.

1. If a service is misbehaving, a great way to debug it is to watch its logs:

  ```sh
  cage logs -f <service-or-pod-name>
  ```

2. For a fairly thorough end-to-end test of the system, after bringing up all the services, try running this:

  ```sh
  cage run group-mail-receiver email-from-john
  ```

  The group-mail-receiver lambda is integrated here as a "task pod" which means that rather than running in the background,
  it's a task that we can run manually whenever we need it. In this case, we're running a lambda which triggers a series
  of events ultimately leading to emails being sent (local only, no *actual* emails get sent here). If everything worked,
  you should be able to look inside the `sent-email` directory and see that an email was just sent successfully.

3. Using the local AWS mocks:

  You can use the local AWS mocks similarly to real AWS services, as long as you remember to set the endpoint. For
  example, to list the buckets that exist locally:

  ```sh
  aws --endpoint-url='http://localhost:4569' s3api list-buckets
  ```

  For convenience, some of the AWS mocks expose their data directly as files on your host machine. This is partly so
  that their data persists even after you stop the services, and is also useful in other ways:

  - `s3-data`: the data store of the local S3 mock. Useful for checking or tweaking bucket contents during development.
  - `kinesis-data`: the data store of the local kinesis mock.
  - `sent-mail`: successful requests to the local SES mock are logged here. Useful for manual end-to-end testing.

  If you want to clear these out and start fresh, then just delete the directories.

4. For more cage commands, run `cage --help`.

### Adding a new application

All of the applications and surrounding services are configured in this repository under `pods`. To add a new
application, your best bet is to pick whichever pod file is most similar to what you're trying to add, and just copy and
paste it. Then go through each of the fields and update the names, paths, URLs, commands, etc to match your new app.

If you're building a new Node.js application and you're using the standard [base image](https://hub.docker.com/r/rabblerouser/node-base/),
then Cage's source mounting feature should just work. If you're using something else, you'll need to make sure that the
directories and commands of your source code and Docker image all line up. To learn more, read the [Cage docs](http://cage.faraday.io/),
or try to reverse engineer how it works from our existing Node.js applications and images, or ask the team for help.
