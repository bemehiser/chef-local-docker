# chef-local Docker Container
[![Docker Repository on Quay.io](https://quay.io/repository/zuazo/chef-local/status "Docker Repository on Quay.io")](https://quay.io/repository/zuazo/chef-local)

[Docker](https://www.docker.com/) images with [Chef](https://www.chef.io/) configured to make it easier to run cookbooks using chef in local mode (with *chef-zero*).

The images come with Chef `12.3.0` installed, also include Berkshelf and git.

## Installation

    $ docker pull zuazo/chef-local:debian-7

## Supported Tags

* `debian-7`: A Debian Jessie image.
* `ubuntu-12.04`: An Ubuntu Precise Pangolin LTS image.
* `ubuntu-14.04`: An Ubuntu Trusty Tahr LTS image.
* `centos-6`: A CentOS 6 image.

## Usage

### Running a Cookbook from the Current Directory

You can include the following *Dockerfile* in your cookbooks to run them inside a Docker container:

```Dockerfile
FROM zuazo/chef-local:debian-7

# Copy the cookbook from the current working directory:
COPY . /tmp/mycookbook
# Download and install the cookbook and its dependencies in the cookbook path:
RUN berks vendor -b /tmp/mycookbook/Berksfile $COOKBOOK_PATH
# Run Chef Client, runs in local mode by default:
RUN chef-client -r "recipe[apt],recipe[mycookbook]"

# CMD to run you application
```

Now you can create a Docker image and run your application:

    $ docker build -t mycookbook .
    $ docker run -d mycookbook bash

The cookbook must have a *Berksfile* for this to work. You can use `$ berks init .` to generate a *Berksfile*. See the [Berkshelf](http://berkshelf.com/) documentation for more information.

### Running a Cookbook from GitHub

For example, you can user the following *Dockerfile* to install Nginx:

```Dockerfile
FROM zuazo/chef-local:debian-7

RUN git clone https://github.com/miketheman/nginx.git /tmp/nginx
RUN berks vendor -b /tmp/nginx/Berksfile $COOKBOOK_PATH
RUN chef-client -r "recipe[apt],recipe[nginx]"

CMD ["nginx", "-g", "daemon off;"]
```

Then you can build your image and start your Nginx server:

    $ docker build -t chef-nginx .
    $ docker run -d -p 8080:80 chef-nginx

Now you can go to [http://localhost:8080](http://localhost:8080) to see your gorgeous web server.

### Running Cookbooks from a Berksfile

You can use a *Berksfile* to run cookbooks if you prefer.

For example, using the following *Berksfile*:

```ruby
source 'https://supermarket.chef.io'

cookbook 'apache2'
# cookbook ...
```

With the following *Dockerfile*:

```Dockerfile
FROM zuazo/chef-local:debian-7

COPY Berksfile /tmp/Berksfile
RUN berks vendor -b /tmp/Berksfile $COOKBOOK_PATH
RUN chef-client -r "recipe[apache2]"
```

Then you can build your image and start your apache2 server installed with Chef:

    $ docker build -t chef-apache2 .
    $ docker run -d -p 8088:80 chef-apache2

Now you can go to [http://localhost:8088](http://localhost:8088) to see your web server.

See the [*examples/*](https://github.com/zuazo/chef-local-docker/tree/master/examples) directory.

## Build from Sources

Instead of installing the image from Docker Hub, you can build the images from sources if you prefer:

    $ git clone https://github.com/zuazo/chef-local-docker chef-local
    $ cd chef-local/debian-7
    $ docker build -t zuazo/chef-local:debian-7 .

## Defined Environment Variables

* `COOKBOOK_PATH`: Directory where the cookbooks should be copied
  (`/tmp/chef/cookbooks`).
* `CHEF_VERSION`: Chef version to install (`12.3.0`).
* `CHEF_REPO_PATH`: Chef main repository path (`/tmp/chef`).

# License and Author

|                      |                                          |
|:---------------------|:-----------------------------------------|
| **Author:**          | [Xabier de Zuazo](https://github.com/zuazo) (<xabier@zuazo.org>)
| **Copyright:**       | Copyright (c) 2015
| **License:**         | Apache License, Version 2.0

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.