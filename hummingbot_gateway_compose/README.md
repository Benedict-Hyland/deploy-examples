# Deploy Hummingbot and Gateway Instances

This installs a [Hummingbot](https://github.com/hummingbot/hummingbot) instance linked to a [Hummingbot Gateway](https://github.com/hummingbot/gateway) instance.

## Prerequisites

This configuration requires [Docker Compose](https://docs.docker.com/compose/), a tool for defining and running multi-container Docker applications. The recommended way to get Docker Compose is to install [Docker Desktop](https://www.docker.com/products/docker-desktop/), which includes Docker Compose along with Docker Engine and Docker CLI which are Compose prerequisites.

Docker Desktop is available on:

* [Linux](https://docs.docker.com/desktop/install/linux-install/)
* [Mac](https://docs.docker.com/desktop/install/mac-install/)
* [Windows](https://docs.docker.com/desktop/install/windows-install/)


## Apple M1/M2 and other ARM machines

If you have a recent Mac that uses Apple Silicon (M1/M2) chipset or another ARM-based machine, you need to change the image tag to ensure that you pull a container that is optimized for your chip architecture. 

Use an IDE like [VSCode](https://code.visualstudio.com/) to edit the `docker-compose.yml` file. Change the tag for **both** the Hummingbot and Gateway images from `latest` to `latest-arm` to pull the images built for ARM-based machines. 

You can also comment out the each line that contains `latest` and uncomment each line that contains `latest-arm`:
```
# image: hummingbot/hummingbot:latest
image: hummingbot/hummingbot:latest-arm

# image: hummingbot/gateway:latest
image: hummingbot/gateway:latest-arm
```

Afterwards, save the file and proceed to the next step.

If you are using a Mac with an Intel (x86) chipset, Windows or any other Intel-based machine, you don't need to make any changes before deploying a container.

## Getting Started

In Terminal/Bash, run the following command to check that you have installed Docker Compose successfully:
```
docker compose
```

You should see a response that start with:
```
Usage:  docker compose [OPTIONS] COMMAND
```

Clone this repo or copy the `docker-compose.yml` file to a directory on your machine where you want to store your Hummingbot files. This is where your encrypted keys, scripts, trades, configs, logs, and other files related to your bots will be saved.

To link the Hummingbot and Gateway instances, you first have to generate certificates within Hummingbot and set the `GATEWAY_PASSPHRASE` variable in the YAML file.


### 1. Launch network

Pull the latest Hummingbot and Gateway images and start instances with the following command:
```
docker compose up -d
```

After the images have been downloaded, you should see the following output:
```
[+] Running 3/3
 ⠿ Network hummingbot_gateway_compose_default     Created
 ⠿ Container hummingbot                           Started
 ⠿ Container gateway                              Started       
```

### 2. Set permissions

Run this command in root directory to grant read/write permission to the `hummingbot_files` and `gateway_files` folders:
```
sudo chmod -R a+rw ./hummingbot_files ./gateway_files
```

### 3. Generate certificates

Now, attach to the Hummingbot `bot` instance:
```
docker attach hummingbot_gateway_compose-bot-1
```

You should see the Hummingbot welcome screen:

![welcome screen](../welcome.png)

Set your Hummingbot [password](https://docs.hummingbot.org/operation/password/) and write it down. This is the `CONFIG_PASSWORD` environment variable in your `docker-compose.yml` file.

Afterwards, run the following command to generate Gateway certificates:
```
gateway generate-certs
```

You'll be prompted for a passphrase used to generate the certificates. This is the `GATEWAY_PASSPHRASE` environment variable in your `docker-compose.yml` file.

Afterwards, Hummingbot will use the passphrase to generate the certificates and save them in the `hummingbot_files/certs` folder, where the Gateway instance will look for the certificates it needs.

Now, run `exit` to exit the client. 

### 4. Remove network

Once you're back in Bash/Terminal, run the following command to remove the Compose project:
```
docker compose down
```

You should see the following output:
```
[+] Running 3/3 
 ⠿ Container gateway                            Removed
 ⠿ Container hummingbot                         Removed
 ⠿ Network hummingbot_gateway_compose_default   Removed
```  

### 5. Modify YAML file

Now, use an IDE like [VSCode](https://code.visualstudio.com/) to edit the `docker-compose.yml` file.

Edit the section that defines the `CONFIG_PASSWORD` and `CONFIG_FILE_NAME` environment variables:
```yaml
  bot:
    # environment:
      #  - CONFIG_PASSWORD=[password]
  gateway:
    # environment:
      #  - GATEWAY_PASSPHRASE=[passphrase]
```

Uncomment out:
 * The `environment:` lines
 * The `CONFIG_PASSWORD` lines: add your Hummingbot password
 * The `GATEWAY_PASSPHRASE` line: add the passphrase you used to generate the certificates

The final `environment` section of the YAML file should look like this:
```yaml
  bot:
    environment:
      - CONFIG_PASSWORD=[password]
  gateway:
    environment:
      - GATEWAY_PASSPHRASE=[passphrase]
```

Afterwards, save the file.

### 6. Recreate network

Now, recreate the Compose project:
```
docker compose up -d
```

Attach to the `hummingbot` instance:
```
docker attach hummingbot
```
After you enter your password, you should now see `GATEWAY:ONLINE` in the upper-right hand corner.

Open a new Terminal/Bash window. In it, attach to the `gateway` instance to see its logs:
```
docker attach gateway
```
See [Gateway](https://docs.hummingbot.org/gateway/) for more details on how to configure it for use with Hummingbot.


## Useful Docker Commands

Use the commands below or use the Docker Desktop application to manage your containers:

### Create/Launch Compose project
```
docker compose up -d
```

### Remove the Compose project
```
docker compose down
```

### Update the Compose project for the latest images
```
docker compose up --force-recreate --build -d
```

### Give all users read/write permissions to local files
```
sudo chmod 666 *.*
```

### Attach to a container
```
docker attach <container-name>
```

### Detach from a container and return to command line

* Press keys <kbd>Ctrl</kbd> + <kbd>P</kbd> then <kbd>Ctrl</kbd> + <kbd>Q</kbd>

### List all containers
```
docker ps -a
```

### Stop a container
```
docker stop <container-name>
```

### Remove a container
```
docker rm <container-name>
```
