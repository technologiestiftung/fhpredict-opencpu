# opencpu with docker


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [opencpu with docker](#opencpu-with-docker)
  - [Prerequisites](#prerequisites)
  - [How To](#how-to)
    - [Run the Container](#run-the-container)
    - [Add R Packages](#add-r-packages)
    - [Test the opencpu API](#test-the-opencpu-api)
    - [Do Requests](#do-requests)
      - [**POST** - /ocpu/library/fhpredict/R/simple](#post---ocpulibraryfhpredictrsimple)
      - [**GET** - /ocpu/tmp/<SESSION R Object>/R/.val](#get---ocputmpsession-r-objectrval)
      - [**GET** - /ocpu/info](#get---ocpuinfo)
      - [**GET** - /ocpu/library/fhpredict/info](#get---ocpulibraryfhpredictinfo)

<!-- /code_chunk_output -->

## Prerequisites

- Docker
  - MacOS `brew cask install docker` or by [download](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
  - Windows by [download](https://hub.docker.com/editions/community/docker-ce-desktop-windows)

## How To

Basic instructions can be found here: [www.opencpu.org/download.html](https://www.opencpu.org/download.html)

> Now simply open [http://localhost/ocpu/](http://localhost/ocpu/) and [http://localhost/rstudio/](http://localhost/rstudio/) in your browser! Login via rstudio with user: opencpu (passwd: opencpu) to build or install apps.

**!Hint:** Since PORT 80 is used we need to use the 8004 PORT:

- opencpu api explorer is at http://localhost:8004/ocpu
- rstudio  is at http://localhost:8004/rstudio (user: opencpu password: opencpu)

The folder `./workspace` is mapped into the working directory of the container at `/home/opencpu` all files in workspace will directly we available in the container.  

### Run the Container

**!Hint:** Don't copy the `$` in the commands. It is for marking the input.

Start it:  

```bash
$ cd path/to/repository
# if you didn't do changes to opencpu/Dockerfile.dev
# You can ommit the --build flag
$ docker-compose up --build
```

Open your browser and start hacking.

Start a bash session within the container run. Be aware that all changes will be lost when the container gets stopped:  

```bash
$ docker ps
> CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                                 NAMES
> 50ad1d0af4c6        opencpu-docker_opencpu   "/bin/sh -c 'service…"   22 minutes ago      Up 22 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:8004->8004/tcp, 443/tcp   opencpu-docker_opencpu_1
```

Use the Container ID or the NAMES property to run the bash session.

```bash
docker exec -it <CONTANER ID | NAME> bash
```

Stop it:

End the terminal session by hitting `CTRL + C` and stop the containers (all changes other the done to the files in `workspace:/home/opencpu` will be lost)

```bash
docker-compose down
```

### Add R Packages

To add additional R packages to the container image you have to edit the file `opencpu/Dockerfile.dev`.  
Add a line containing your desired package like this one. Please add the ref parameter to the install (in this case the `@v0.1.0-beta`):  

```docker
RUN R -e "devtools::install_github(\"technologiestiftung/fhpredict@v0.1.0-beta\")"
```

Then run again a `docker-compose up --build` from the root of the repo.  

### Test the opencpu API

The opencpu API docs can be found [here](https://www.opencpu.org/api.html).  

The code below assumes the `fhpredict` package is installed and the opencpu container is running. 

### Do Requests

#### **POST** - /ocpu/library/fhpredict/R/simple

**Description:** The function `simple(str = '{"foo":"bah"}')` of the package `fhpredict` takes a JSON string as argument. (Needs to be properly escaped.)  

Requests using CURL

```sh
curl -X POST "http://localhost:8004/ocpu/library/fhpredict/R/simple" \
    -H "Content-Type: application/json" \
    --data-raw "$body"
```

**Header Parameters:** **Content-Type** should respect the following schema:

```plain
{
  "type": "string",
  "enum": [
    "application/json"
  ],
  "default": "application/json"
}
```

**Body Parameters:** **body** should respect the following schema:

```plain
{
  "type": "string",
  "default": "{\n  \"str\": \"{\\\"foo\\\":12}\"\n}"
}
```

#### **GET** - /ocpu/tmp/<SESSION R Object>/R/.val

**Description:** The POST request returns something like the example below. These are new endpoints that are generated during runtime and will be discarded on restart. The result of the calculation can be found under the `.val` endpoint. See also [https://www.opencpu.org/api.html#api-session](https://www.opencpu.org/api.html#api-session)

```bash
/ocpu/tmp/x0aa062559b9cf1/R/.val
/ocpu/tmp/x0aa062559b9cf1/R/simple
/ocpu/tmp/x0aa062559b9cf1/stdout
/ocpu/tmp/x0aa062559b9cf1/source
/ocpu/tmp/x0aa062559b9cf1/console
/ocpu/tmp/x0aa062559b9cf1/info
/ocpu/tmp/x0aa062559b9cf1/files/DESCRIPTION
```

Requests using CURL

```sh
curl -X GET "http://localhost:8004/ocpu/tmp/x0aa062559b9cf1/R/.val"
```

#### **GET** - /ocpu/info

**Description:** From the api docs.  
> The `/ocpu/info` endpoint shows the output of sessionInfo() from the main (webserver) process. This can be helpful for debugging. The /ocpu/test URL gives you a handy testing web page to perform server requests.

See also [https://www.opencpu.org/api.html#api-libraries](https://www.opencpu.org/api.html#api-libraries)

Requests using CURL

```sh
curl -X GET "http://localhost:8004/ocpu/info"
```

#### **GET** - /ocpu/library/fhpredict/info

**Description:** Show information about the pacakge fhpredict package. See also [https://www.opencpu.org/api.html#api-libraries](https://www.opencpu.org/api.html#api-libraries)

Requests using CURL

```sh
curl -X GET "http://localhost:8004/ocpu/library/fhpredict/info"
```