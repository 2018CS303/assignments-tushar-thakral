# Docker Case Study
# Automate Infra Allocation

## Tasks To Do : 
- Dynamic allocation of Linux systems for users.
- Each user should have independent Linux System.
- Specific training environment should be created in container.
- User should not be allowed to access other containers.
- User should not allowed to access docker command.
- Debug/live demo for the participants if they have any doubts/bug in running applications.
- Automate container creation and deletion.
- Monitor participants' containers.

## Creating Training Environment :
We'll create a new image and use that for each user:

- Run `docker run --name traincont -it ubuntu /bin/bash`. The container's shell should now be available.
- Make changes as required.
- Now type `exit` command to exit from the shell.
- We can create a new image from the `traincont` container by running `docker commit traincont training:v1`.
- The `training` image will be used to create container for each user.

## Allocating Container to each user :

For creating a container for a single user with user id `example` type: `docker create -it --name example training:v1 /bin/bash`. 

We can automate the allocation of containers. Let `userfiles` contain the user ids :

```
user-1
user-2
user-3
```

The script to automate container allocation :

```bash
#!/bin/sh

file=$1

# If filename not passed exit
if [ -z "${file}" ]
then
	echo "Usage: allocate <filename>"
	exit
fi

while read user || [[ -n "$user" ]]
do
	container_id="$(docker create -it --name $user training:v1 /bin/bash)"
	echo "$user: $container_id"
done < "$file"

```

Run the command `allocate userfiles` to allocate a container for every
user id in `userfiles`.

## Using an Allocated Container :

The container for user with id `example` can be used by running the commands:

```bash
docker start example # Starts container example
docker attach example # Attaches local stdin, stdout and stderr to example
```

The user can use the `exit` command to exit from the container. 

## Safety of Containers : 

Since the resulting shell is a process within a container, it is isolated from
the host system and other containers. Any side effects a command or program
might have will only affect the container and not the host system.

## Monitoring Containers :

- We can view the resource usage of container `example` by running the command `docker stats example`. 
- We can also view the logs of container `example` by running `docker logs example`.

## Deletion of Containers :

To remove the container `example` we can use the
`docker rm example` command.

To remove a list of containers we can automate it with the script:

```bash
#!/bin/sh

file=$1

if [ -z "${file}" ]
then
	echo "Usage: delete <file>"
	exit
fi

while read user || [[ -n "$user" ]]
do
	docker rm $user
done < "$file"

```

Run the command `./deallocate userfiles`.
