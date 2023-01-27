---
title: "Integrating Cloudflare Tunnels with a Linux Server"
date: 2023-01-27T11:27:24+08:00
showToc: true
TocOpen: false

desc: "Building any software product takes a lot of effort. This is no different from building a mobile application. we tried our hands at building one and here are the results."

ShowReadingTime: true
ShowWordCount: false
---

Recently, I started using Cloudflare tunnels to expose my server to the web. It provides security as well as accessibility. I do not have to expose any ports on my server and the links provided to access the services on my server are all SSL protected.

That got me thinking if it was possible to integrate web development into a Linux server. The problem with doing web development remotely is that you cannot test and see the updates you have made.

## Background

Cloud-based development has been around for quite a while. Having such a development environment brings about many benefits.

1. Everyone uses the same dependencies and environment. This reduces the chances of "Oh it worked fine on my local machine".

2. The increased amount of computing power. Cloud servers are likely to provide more computing power than your local machine. It can also be scaled when needed.

3. Cost savings (in a general sense). Having the machine in the cloud, you likely do not need to shell out cash for spec-ed out machines for your developers.

However, development on the cloud does have its drawbacks. For example, if you are doing web development, you cannot just do `localhost:8080` to access the service you created. Instead, you will need to expose port 8080, then access it through your `IP-ADDRESS:8080`. This may seem sound for now. But imagine you are in a team environment. Many ports will need to be exposed just for development. This poses a huge security risk as it is never a good idea to expose ports unless necessary.

So here is where [Cloudflare tunnels](https://www.cloudflare.com/en-gb/products/tunnel/) come in. It provides a solution to access the localhost ports without actually exposing them to the public.

## Use Case

For my use case particularly, I am intending to teach a class using a cloud-based development environment. When a student runs their React app or NodeJs server, they should be able to access the site through a predefined subdomain.

## Requirements

1. Automated - There should be minimal to no manual input from me.

2. Hassle-free - Students should be able to develop their app just as how they can do it on their local machine.

3. Availability - Web link should be available when the student starts running the service. It should also be removed once the student is done using it. In this case, it means when the user has exited the terminal session.

## Constraints

For my use case, the user will be running using the node environment and running the `npm run dev` command to start their service. However, this solution should be easily modifiable to any other commands that start listening on a port.

Another constraint is that each user can only use 1 port (Which is assigned and defined in their environment variable). This is generally done to reduce the chances of port numbers clashing.

## Breaking it up

To solve this problem, we can break the issue down into 2 parts.

1. Detect when the `npm run dev` command is called and when the user exits the session

2. Interacting with Cloudflare:

   - Check if the tunnel configuration for that port already exists

   - Add a new configuration for that port if it does not exist

   - Remove config for a particular port

## Detecting Commands

To detect when a command is being run, we can use the `trap` command available on Linux machines. This command monitors the signals that are produced when commands are run on the machine.

For example, before running every command, the system actually outputs a `DEBUG` signal. We can monitor this signal and check the command that emitted this signal.

### Detect npm run dev

To detect `npm run dev`, we will "trap" every debug signal and check if the command emitting it is `npm run dev`.

```bash
COMMAND_TO_MONITOR='npm run dev'

function trap_command(){
    if [ "$BASH_COMMAND" == "$COMMAND_TO_MONITOR" ]; then
        if [ -z "$PORT" ]; then
            echo "no port defined in env. not accessible to public"
        else
            echo "your port number is $PORT"
        fi
    fi
}

trap 'trap_command' DEBUG
```

The above script snippet creates a variable `COMMAND_TO_MONITOR` which is the command we want to monitor. Following that, we define a function `trap_command` that we want to execute every time we have a `DEBUG` signal sent. This function checks the command ran against the command we want to monitor. If it matches, we then check if a `PORT` is defined in the environment variables. The `-z` flag checks if the string is empty. If no `PORT` is defined, we will just echo a line.

If a port is defined, we would ideally do the checking and updating of tunnel configurations here. These functionalities will be added later. For now, we will just print out the port number that the service is running on.

### Detect Exit

To detect when the user has exited the session, we can use the `EXIT` signal. As the name suggests, this signal is emitted when a user exits the session.

```bash
function exit_cleanup(){
    if [ -z "$PORT" ]; then
        echo "nothing to do"
    else
        echo "cleaning port $PORT"
    fi
}

trap 'exit_cleanup' EXIT
```

In the above snippet, we defined a function `exit_cleanup` that will be doing some housekeeping tasks. One of them is to remove the tunnel configuration of this particular user. We will update that functionality in the future. For now, we check if there is a `PORT` defined in the environment variable. If there is, we will "clean" the port. Else, we will do nothing.

The last line is to trap the `EXIT` signal and run the `exit_cleanup` function when there is an `EXIT` signal detected.

### Side Note

Note that this was not the initial plan. The initial plan was "clean up" after the service is closed. However, upon further consideration, I decided to monitor when the user exits the session instead. This is due to:

1. For some reason, `SIGINT` and `SIGTERM` signals are not detected when I use `CTRL - C` to close the node server. Apart from this, there are other issues as well that I detailed in the appendix.

2. More importantly, it is likely that the user will have to restart the server once in a while to apply changes. Hence it is not wise to keep making API calls to update the tunnel configurations.

## Interacting with Cloudflare

To interact with Cloudflare, we can use the HTTP APIs made available by them. They do have a console tool `cloudflared` available. However, it requires quite a bit of set up. Also for it to be used system-wide, it might pose a security threat as users have access to non-tunnel related configurations.

HTTP APIs use authorization bearer tokens that can be scoped. So we can limit the scope to just tunnel configurations and revoke the token when security issues are detected.

### API Interaction

To interact with the API, we will use a Python script rather than a bash script. Python has a well-supported `requests` library to make HTTP requests. It is also easier to manipulate JSON responses in python than in bash.

Our python script will:

1. Get the hostname from the configuration tagged to the port provided

2. If the provided port does not have a configuration, we create a new configuration

3. Delete configuration tagged to a port

To achieve this, our script will need to take in some flags so it can differentiate `GET` and `DELETE` actions. It also needs a flag to know which port are we interested in.

| flag       | description                              | values            |
| ---------- | ---------------------------------------- | ----------------- |
| --type     | type of action you want the script to do | `GET` \| `DELETE` |
| --port, -p | port you are interested in               | any port number   |

```bash
./cloudflareCall.py --type GET -p 3000
# The script will get the configuration of port 3000 and
# output the hostname tagged to that port.

./cloudflareCall.py --type DELETE -p 3000
# The script will delete the configuration tagged to port 3000
```

### Implementation

In the beginning, we check the flags to make sure they exist and the input values are valid. Once the flags are checked, we run the functions based on what the `type` flag is.

#### type: GET

If the `type` is `GET`, we will get the tunnel configurations from Cloudflare and filter them by the port provided. If there are no configurations available, we will create a new configuration. We build a new configuration using a builder and then append it to the configurations before sending the `PUT` request.

Making update requests does not create a DNS CNAME record. Hence we need to make another call to create a DNS record if there is no DNS record for the hostname. We build the new DNS record and make a `POST` request to create it.

At this point, the link should be available for use.

#### type: DELETE

For `type` `DELETE`, we will make a `GET` request and filter it to make sure configurations exist to be deleted. If there are no configurations linked to that port number, we do not need to do anything.

If there is a configuration, we will remove it and make a `PUT` request to update the configurations on Cloudflare.

Following that, we will also remove the DNS records that are linked to this configuration.

As there are only `PUT` and `GET` methods for the Cloudflare tunnels configuration API, it does raise an issue of race condition. This will be discussed further below.

Full script implementation is located [here](https://github.com/fangpinsern/integrating-cloudflare-tunnels-with-a-linux-server/tree/main).

## Putting it together

After implementing the ability to interact with Cloudflare, we can add the python script into our monitoring script.

### Updated monitoring script

```bash
command_to_monitor='npm run dev'

function trap_command(){
  if [ "$BASH_COMMAND" == "$command_to_monitor" ]; then
    if [ -z "$PORT" ]; then
      echo "no port defined in env. not accessible to public"
    else
      hostname=$(cd /usr/local/bin; ./cloudflareCall.py --type GET --port $PORT)
      echo "your service is deployed on $hostname"
    fi
  fi
}

trap 'trap_command' DEBUG

function exit_command(){
  if [ -z "$PORT" ]; then
    echo "nothing to do"
  else
    echo "cleaning up cloudlflare configs"
    result=$(cd /usr/local/bin; ./cloudflareCall.py --type DELETE --port $PORT)
    echo $result
  fi
  echo "EXIT FUNCTION RAN"
}

trap 'exit_command' EXIT
```

We updated it so it will `GET` the hostname every time `npm run dev` is called. It will `DELETE` the configurations when the user exits. We place the script in `/usr/local/bin` so every user will have access to it.

## Problem (for the future)

The above solutions do bring about some problems. Here I will discuss 2 main issues.

### 1. Race condition

Due to the limitation of the API exposed by Cloudflare, we are only able to make a `PUT` request to update tunnel configurations. This means to add a new configuration, we need to get the latest configuration, add the new configuration in and make the `PUT` request. Likewise for deleting configurations.

Considering a multi-user environment, it might cause some users to experience their configurations being removed even though they did not exit the system.

To resolve this, we can use a mutex lock to only allow one user to run the script at any one time. Other users will need to wait their turn. To implement this, we can add the following snippet to our python script:

```python
import fcntl
import time

lock_file = 'LOCK_FILE_PATH'
wait_time = 1
time_out = 10

def acquire_lock():
    fd = open(lock_file, 'w')
    acquired = False
    count = 0
    while not acquired:
        try:
            fcntl.lockf(fd, fcntl.LOCK_EX | fcntl.LOCK_NB)
            acquired = True
        except IOError:
            if count == time_out:
                break
            count = count + 1
            time.sleep(wait_time)
    return acquired

def release_lock():
    fd = open(lock_file, 'w')
    fcntl.lockf(fd, fcntl.LOCK_UN)
    fd.close()
```

The above snippet defines functions to acquire and release the locks. A timeout is set for 10 seconds. If they cannot acquire the lock after 10 seconds, the program should exit. The function also retries every second.

We can add the `acquire_lock` function to where we update or delete tunnel configurations. All locks need to be released using the `release_lock` function when the operation is completed.

Here is some pseudo code on how we will use it to update the configs.

```
if acquire_lock():
    try:
        get_latest_config()
        add_new_config()
        update_config()
    finally:
        release_lock()
else:
    print("Cannot acquire lock after 10 seconds.")
```

### 2. Inconsistency due to failed requests

In the scenario where the DNS update fails, the tunnel configuration will still be updated. Hence there may be some inconsistency between the tunnel configurations and the DNS records.

This might cause the user's app to be inaccessible at the intended domain name.

To resolve this issue, we may want to use a transactional model instead. So when a request fail is detected anywhere along the pipeline, we will conduct revert operations. This will revert all configurations as if the operation did not happen. This is out of the scope of this post. Maybe I will do it in the future.

Note that due to the use case, we can tolerate some slight inconsistencies. If a student cannot access the site, they can approach their instructors and the instructor will manually resolve the issue for them

## Conclusion

## Appendix (for those interested)

### Why clean up on exit?

Above I mentioned that I wanted to do "clean up" when the user close the service (CTRL + C) but chose against it after further consideration. Here I will detail the other issues faced.

One main issue with it was that the action `CTRL + C` is used everywhere. Not just for closing the service you are developing. How can we know that the `CTRL + C` is for the closing of a node server?

The following are things I tried to make it work:

I tried to resolve this issue by getting the previous command ran before a `CTRL + C` action. This did not work because for some reason when using a script, the script has its own command history. It does not detect the previous command.

Another way was to use the `.bash_history` file which records all commands ran. This sounds possible at first. However, I found out that this command history is only updated when the user exits the session. It is not updated immediately when a command is run. To resolve this, we can actually add a snippet to the `.bashrc` file and it will append to the `.bash_history` file after every command. Assuming the `SIGINT` or `SIGTERM` signal is sent, this would work. But it would only work in a single instance environment. For full-stack development, there is a possibility of running 2 node instances at the same time for manual testing. Hence you might be detecting the wrong service closing.

We could also add certain graceful shutdown functionalities to the services we are running. However, that would violate the "Hassle-free" requirement of this project as students will need to make these changes.
