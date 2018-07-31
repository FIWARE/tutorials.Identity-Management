[![FIWARE Banner](https://fiware.github.io/tutorials.Identity-Management/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Security](https://img.shields.io/badge/FIWARE-Security-ff7059.svg)](https://www.fiware.org/developers/catalogue/)
[![Documentation](https://readthedocs.org/projects/fiware-tutorials/badge/?version=latest)](https://fiware-tutorials.readthedocs.io/en/latest)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This tutorial is an introduction to [FIWARE Keyrock](http://fiware-idm.readthedocs.io/en/latest/) - a generic enabler which introduces
**Identity Management** into FIWARE services. The tutorial explains how to create users and organizations in preparation to assign
roles and permissions to them in a later tutorial.

The tutorial demonstrates examples of interactions using the **Keyrock** GUI, as well [cUrl](https://ec.haxx.se/) commands used
to access the **Keyrock** REST API - [Postman documentation](http://fiware.github.io/tutorials.Identity-Management/) is also available.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/2150531e68299d46f937)

# Contents

- [Introduction to Identity Management](#introduction-to-identity-management)
  * [Standard Concepts of Identity Management](#standard-concepts-of-identity-management)
  * [OAuth2](#oauth2)
- [Prerequisites](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [Architecture](#architecture)
  * [Keyrock Configuration](#keyrock-configuration)
  * [MySQL Configuration](#mysql-configuration)
- [Start Up](#start-up)
- [Identity Management](#identity-management)
    + [Dramatis Personae](#dramatis-personae)
    + [Reading directly from the Keyrock MySQL Database](#reading-directly-from-the-keyrock-mysql-database)
    + [UUIDs within Keyrock](#uuids-within-keyrock)
  * [Logging In](#logging-in)
    + [Create Token with Password](#create-token-with-password)
    + [Get Token Info](#get-token-info)
    + [Refresh Token](#refresh-token)
- [Administrating User Accounts](#administrating-user-accounts)
  * [User CRUD Actions](#user-crud-actions)
    + [Creating Users](#creating-users)
    + [Read Information About a User](#read-information-about-a-user)
    + [List all Users](#list-all-users)
    + [Update a User](#update-a-user)
    + [Delete a User](#delete-a-user)
- [Grouping User Accounts under Organizations](#grouping-user-accounts-under-organizations)
  * [Organization CRUD Actions](#organization-crud-actions)
    + [Create an Organization](#create-an-organization)
    + [Read Organization Details](#read-organization-details)
    + [List all Organizations](#list-all-organizations)
    + [Update an Organization](#update-an-organization)
    + [Delete an Organization](#delete-an-organization)
  * [Users within an Organization](#users-within-an-organization)
    + [Add a User as a Member of an Organization](#add-a-user-as-a-member-of-an-organization)
    + [Add a User as an Owner of an Organization](#add-a-user-as-an-owner-of-an-organization)
    + [List Users within an Organization](#list-users-within-an-organization)
    + [Read User Roles within an Organization](#read-user-roles-within-an-organization)
    + [Remove a User from an Organization](#remove-a-user-from-an-organization)
- [Next Steps](#next-steps)

# Introduction to Identity Management

> "If one meets a powerful person — ask them five questions: ‘What power have you got?
> Where did you get it from? In whose interests do you exercise it? To whom are you
> accountable? And how can we get rid of you?’"
>
> — Anthony Wedgwood Benn (The Five Essential Questions of Democracy)


In computer security terminology, Identity management is the security and business discipline that "enables the right
individuals to access the right resources at the right times and for the right reasons". It addresses the need to
ensure appropriate access to resources across disparate systems.

The FIWARE framework consists of a series of separate components, and the security chapter aims to implement
the common needs of these components regarding who (or what) gets to access which resources within the system,
but before access to resources can be locked down, the identity of the person (or service) making the request
needs to be known. The FIWARE **Keyrock** Generic Enabler sets up all of the common characteristics of an
Identity Management System out-of-the-box, so that other components are able to use standard authentication
mechanisms to  accept or reject requests based on industry standard protocols.

Identity Management therefore covers the issues of how to gain an identity within the system, the protection
of that identity and the surrounding technologies such as passwords and network protocols.

## Standard Concepts of Identity Management

The following common objects are found with the **Keyrock** Identity Management database:

* **User** - Any signed up user able to identify themselves with an eMail and password. Users can be assigned
 rights individually or as a group
* **Application** -  Any securable FIWARE application consisting of a series of microservices
* **Organization** - A group of users who can be assigned a series of rights. Altering the rights of the organization
 effects the access of all users of that organization
* **OrganizationRole** - Users can either be members or admins of an organization - Admins are able to add and remove users
 from their organization, members merely gain the roles and permissions of an organiation. This allows each organization
 to be responisible for their members and removes the need for a super-admin to administer all rights
* **Role** - A role is a descriptive bucket for a set of permissions. A role can be assigned to either a single user
 or an organization. A signed-in user gains all the permissions from all of their own roles plus all of the roles associated
 to their organization
* **Permission** - An ability to do something on a resource within the system

Additionally two further non-human application objects can be secured within a FIWARE application:

* **IoTAgent** - a proxy betwen IoT Sensors and  the Context Broker
* **PEPProxy** - a middleware for use between generic enablers challenging the rights of a user.


 The relationship between the objects can be seen below:

![](https://fiware.github.io/tutorials.Identity-Management/img/entities.png)

## OAuth2

**Keyrock** uses [OAuth2](https://oauth.net/2/) to enable third-party applications
to obtain limited access to services. **OAuth2** is the open standard for access delegation to
grant access rights. It allows notifying a resource provider (e.g. the Knowage Generic Enabler)
that the resource  owner (e.g. you) grants permission to a third-party (e.g. a Knowage Application)
access to their information (e.g. the list of entities).

There are several common OAuth 2.0 grant flows, the details of which can be found below:

* [Authorization Code](https://oauth.net/2/grant-types/authorization-code)
* [Implicit](https://oauth.net/2/grant-types/implicit)
* [Password](https://oauth.net/2/grant-types/password)
* [Client Credentials](https://oauth.net/2/grant-types/client-credentials)
* [Device Code](https://oauth.net/2/grant-types/device-code)
* [Refresh Token](https://oauth.net/2/grant-types/refresh-token)

The primary concept is that both **Users**  and **Applications** must first identify themselves using
a standard OAuth2 Challenge-Response mechanism. Thereafter a user is assigned a token which they
append to every subsequent request. This token identifies the user, the application and the rights the
user is able to exercise.  **Keyrock** can then be used with other enablers can be used to limit and
lock-down access. The details of the access flows are discussed below and in subsequent tutorials.

The reasoning behind OAuth2 is that you never need to expose your own username and password to a
third party to give them  full access - you merely permit the relevant access which can be either Read-Only
or Read-Write and such access can be defined down to a granular level. Furthermore there is provision for
revoking access at any time, leaving the resource owner in control of who can access what.


# Prerequisites

## Docker

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a
container technology which allows to different components isolated into their respective environments.

* To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
* To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
* To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and  Docker for Mac, however Linux users
will need to follow the instructions found  [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/) to provide a
command line functionality similar to a Linux distribution on Windows.

# Architecture

This introduction will only make use of one FIWARE component - the [Keyrock](http://fiware-idm.readthedocs.io/)
Identity Management Generic Enabler. Usage of **Keyrock** alone alone is insufficient for an application to qualify
 as *“Powered by FIWARE”*.  Additionally will be persisting user data in a **MySQL**  database.


The overall architecture will consist of the following elements:

* One **FIWARE Generic Enabler**:
    * FIWARE [Keyrock](http://fiware-idm.readthedocs.io/) offer a complement Identity Management System including:
        * An OAuth2 authentication system for Applications and Users
        * A website graphical front-end for Identity Management Administration
        * An equivalent REST API for Identity Management via HTTP requests

* One [MySQL](https://www.mysql.com/) database :
    * Used to persist user identities, applications, roles and permsissions


Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run from exposed ports.


![](https://fiware.github.io/tutorials.Identity-Management/img/architecture.png)

The specific architecture of each section of the tutorial is discussed below.

## Keyrock Configuration

```yaml
  idm:
    image: fiware-idm
    container_name: idm
    hostname: idm
    depends_on:
      - mysql-db
    ports:
      - "3005:3005"
    networks:
      default:
    environment:
      - DATABASE_HOST=mysql-db
      - IDM_DB_PASS_FILE=/run/secrets/my_secret_data
      - IDM_DB_USER=root
      - IDM_HOST=http://localhost:3005
      - IDM_PORT=3005
    secrets:
      - my_secret_data
```

The `idm` container is a web app server listening on a single port:

* Port `3005` has been  exposed for HTTP traffic so we can display the webpage and interact with the REST API.

> **Note** The HTTP protocols is being used for demonstration purposes only.
> In a production environment, all OAuth2 Authentication should occur over HTTPS, to avoid sending
> any sensitive information using plain-text.

The `idm` container is driven by environment variables as shown:

| Key |Value|Description|
|-----|-----|-----------|
|IDM_DB_PASS|`idm`| Password of the attached MySQL Database - secured by **Docker Secrets** (see below) |
|IDM_DB_USER|`root`|Username of the default MySQL user - left in plain-text |
|IDM_HOST|`http://localhost:3005`| Hostname of the **Keyrock**  App Server - used in activation eMails when signing up users|
|IDM_PORT|`3005`| Port used by the **Keyrock** App Server  - this has been altered from the default 3000 port to avoid clashes |


> :information_source: **Note** that this example has secured the MySQL password using **Docker Secrets**
> By using `IDM_DB_PASS` with the `_FILE` suffix and referring to a secrets file location.
> This avoids exposing the password as an `ENV` variable in plain-text - either in the `Dockerfile` Image or
> as an injected variable which could be read using `docker inspect`.
>
> The following list of variables (where used) should be set via secrets with the  `_FILE` suffix  in a Production System:
>
> * `IDM_SESSION_SECRET`
> * `IDM_ENCRYPTION_KEY`
> * `IDM_DB_PASS`
> * `IDM_DB_USER`
> * `IDM_ADMIN_ID`
> * `IDM_ADMIN_USER`
> * `IDM_ADMIN_EMAIL`
> * `IDM_ADMIN_PASS`
> * `IDM_EX_AUTH_DB_USER`
> * `IDM_EX_AUTH_DB_PASS`



## MySQL Configuration

```yaml
  mysql-db:
    image: mysql:5.7
    hostname: mysql-db
    container_name: db-mysql
    expose:
      - "3306"
    ports:
      - "3306:3306"
    networks:
      default:
    environment:
      - "MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my_secret_data"
      - "MYSQL_ROOT_HOST=172.18.1.5"
    volumes:
      - mysql-db:/var/lib/mysql
    secrets:
      - my_secret_data
```


The `mysql-db` container is listening on a single port:

* Port `3306` is the default port for a MySQL server. It has been exposed so you can also run other database tools to display data if you wish

The `mysql-db` container is driven by environment variables as shown:

| Key               |Value.    |Description                               |
|-------------------|----------|------------------------------------------|
|MYSQL_ROOT_PASSWORD|`123`.    | specifies a password that is set for the MySQL `root` account - secured by **Docker Secrets** (see below)|
|MYSQL_ROOT_HOST    |`root`| By default, MySQL creates the `root'@'localhost` account. This account can only be connected to from inside the container. Setting this environment variable allows root connections from other hosts |

# Start Up

To start the installation, do the following:

```console
git clone git@github.com:Fiware/tutorials.Identity-Management.git
cd tutorials.Identity-Management

./services create
```

>**Note** The initial creation of Docker images can take up to three minutes


Thereafter, all services can be initialized from the command line by running the [services](https://github.com/Fiware/tutorials.Identity-Management/blob/master/services) Bash script provided within the repository:

```console
./services <command>
```

Where `<command>` will vary depending upon the exercise we wish to activate.

>:information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
>```console
>./services stop
>```
>

# Identity Management

As the default super-admin user `admin@test.com` with a password of `1234`, we will set up a series of user accounts and assign them to
relevant organizations within the system.

### Dramatis Personae

The following people legitimately have accounts within the Application

* Alice, she will be the Administrator of the **Keyrock** Application
* Bob, the Regional Manager of the supermarket chain - he has several store managers under him:
  * Manager1
  * Manager2
* Charlie, the Head of Security of the supermarket chain  - he has several store detectives under him:
  * Detective1
  * Detective2



### Reading directly from the Keyrock MySQL Database

All Identify Management records  and releationships are held within the the attached MySQL database. This can be
accessed by entering the running Docker container as shown:

```console
docker exec -it db-mysql bash
```

```console
mysql -u <user> -p<password> idm
```

Where `<user>` and `<password>` match the values defined in the `docker-compose` file for `MYSQL_ROOT_PASSWORD`
and `MYSQL_ROOT_USER`. The default values for the tutorial are usually `root` and `secret`.

SQL commands can then be entered from the command line. e.g.:

```SQL
select id, username, email, password from user;
```


### UUIDs within Keyrock

All ids and tokens within  **Keyrock** are subject to change. The following values will need to be amended when
querying for records .Record ids use Universally Unique Identifiers - UUIDs.

| Key |Description                        | Sample Value |
|-----|-----------------------------------|--------------|
|`keyrock`| URL for the location of the **Keyrock** service|`localhost:3005`|
|`X-Auth-token`| Token received in the Header when logging in as a user - in other words *"Who am I?"* |`51f2e380-c959-4dee-a0af-380f730137c3`|
|`X-Subject-token`|Token added to requests to define *"Who do I want to inquire about?"* - This can also be a repeat the `X-Auth-token` defined above |`51f2e380-c959-4dee-a0af-380f730137c3`|
|`user-id`| id of an existing user, found with the `user`  table |`96154659-cb3b-4d2d-afef-18d6aec0518e`|
|`organization-id`| id of an existing organization, found with the `organization`  table |`e424ed98-c966-46e3-b161-a165fd31bc01`|
|`organization-role-id`| type of role a user has within an organization either `owner` or `member`|`member`|

Tokens are designed to expire after a set period. If the `X-Auth-token` value you are using has expired, log-in again to obtain a new token.

## Logging In

The Log-in Screen allows an existing user to identify themselves and obtain a token for further operations. It is the intial start-up
screen of the **Keyrock** GUI - `http://localhost:3005/idm`


![](https://fiware.github.io/tutorials.Identity-Management/img/log-in.png)

Enter a username and password to enter the application. The default super-user has the values `admin@test.com` and `1234`.

### Create Token with Password

The following example logs in using the Admin Super-User:

#### :one: Request:
```console
curl -iX POST \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "name": "admin@test.com",
  "password": "1234"
}'
```

#### Response:

```
HTTP/1.1 201 Created
X-Subject-Token: d848eb12-889f-433b-9811-6a4fbf0b86ca
Content-Type: application/json; charset=utf-8
Content-Length: 138
ETag: W/"8a-TVwlWNKBsa7cskJw55uE/wZl6L8"
Date: Mon, 30 Jul 2018 12:07:54 GMT
Connection: keep-alive
```
```json
{
    "token": {
        "methods": [
            "password"
        ],
        "expires_at": "2018-07-30T13:02:37.116Z"
    },
    "idm_authorization_config": {
        "level": "basic",
        "authzforce": false
    }
}
```

### Get Token Info

`{{X-Auth-token}}` and `{{X-Subject-token}}` should be taken from the previous request,
in the case of the response above, both variables should be set to `d848eb12-889f-433b-9811-6a4fbf0b86ca` - this indicates that *the
user authorized with the token `{{X-Auth-token}}`  is enquiring about the user holding the token  `{{X-Subject-token}}`* - in this
case we only have one user within the **Keyrock** application, and that user is enquiring about himself.

#### :two: Request:

```console
curl -iX GET \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -H 'X-Subject-token: {{X-Subject-token}}'
```

#### Response:

The response will return the details of the associated user

```json
{
    "access_token": "51f2e380-c959-4dee-a0af-380f730137c3",
    "expires": "2018-07-30T13:02:37.000Z",
    "valid": true,
    "User": {
        "id": "admin",
        "username": "admin",
        "email": "admin@test.com",
        "date_password": "2018-07-30T09:55:38.000Z",
        "enabled": true,
        "admin": true
    }
}
```

### Refresh Token

In order to avoid using an expired token, an existing token can be renewed and swapped for a newer one.

The `token` value, `d848eb12-889f-433b-9811-6a4fbf0b86ca` was acquired when the user logged on for the first time

#### :three: Request:

```console
curl -iX POST \
  'http://localhost:3005/v1/auth/tokens' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "token": "d848eb12-889f-433b-9811-6a4fbf0b86ca"
}'
```

#### Response:

A new token is returned in the `X-Subject-Token` header
```
HTTP/1.1 201 Created
X-Subject-Token: a5b83d68-ebad-4514-9d3a-dd892f6e6174
Content-Type: application/json; charset=utf-8
Content-Length: 135
ETag: W/"87-nPb+4XRSsW5Szsf2JJC6UYab4GM"
Date: Mon, 30 Jul 2018 12:41:47 GMT
Connection: keep-alive
```
```json
{
    "token": {
        "methods": [
            "token"
        ],
        "expires_at": "2018-07-30T13:13:20.567Z"
    },
    "idm_authorization_config": {
        "level": "basic",
        "authzforce": false
    }
}
```

# Administrating User Accounts

Users accounts are at the heart of any identity management system. The essential fields of every account hold a unique user name
and email address to identify the user, along with a password for authentication. The other optional fields
add more information about the user such as a user webiste, description or avatar.

## User CRUD Actions

Users are able to sign-up for themselves using the GUI. The only requirement is an email address and a password.

![](https://fiware.github.io/tutorials.Identity-Management/img/sign-up.png)

Once an account is created, the user is sent an eMail to confirm their existence and activate their account.

![](https://fiware.github.io/tutorials.Identity-Management/img/email.png)

The REST API is also able to create and amend users without their own interaction - this could be useful for
bulk CRUD actions for example.

> **Note** - an eMail server must be configured to send out invites properly, otherwise the invitation
> may be deleted as spam. For testing purposes, it is easier to update the users table directly:
>```update user set enabled = 1;```


All the CRUD actions for Users require an `X-Auth-token` header from a previously logged in administrative user to be able
to read or modify other user accounts. The standard CRUD actions are assigned to the appropriate HTTP verbs (POST, GET, PATCH and DELETE)
under the `/v1/users` endpoint.


### Creating Users

To create a new user, send a POST request to the `/v1/users` endpoint containing the `username`,`email` and `password` along with the `X-Auth-token` header
from a previously logged in administrative user.

#### :four: Request:

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "alice",
    "email": "alice@test.com",
    "password": "test"
  }
}'
```

#### Response:

The response returns the details of the created user

```
{
    "user": {
        "id": "3b3a5ad5-afd3-4baa-a538-25c7fe7cbf6a",
        "image": "default",
        "gravatar": false,
        "enabled": true,
        "admin": false,
        "starters_tour_ended": false,
        "username": "alice",
        "email": "alice@test.com",
        "date_password": "2018-07-30T12:51:26.813Z"
    }
}
```

To grant super-admin power to a newly created user account, the database can be altered directly:

```sql
update user set admin = 1 where username='alice';
```

Additional users can be added by making repeated POST requests.

For example to create additional accounts for Bob, the Regional Manager, Charlie, the Head of Security and their direct reports

```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "bob",
    "email": "bob-the-manager@test.com",
    "password": "test"
  }
}'
```
```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "charlie",
    "email": "charlie-security@test.com",
    "password": "test"
  }
}'
```
```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "manager1",
    "email": "manager1@test.com",
    "password": "test"
  }
}'
```
```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "manager1",
    "email": "manager1@test.com",
    "password": "test"
  }
}'
```
```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "detective1",
    "email": "detective1@test.com",
    "password": "test"
  }
}'
```
```console
curl -iX POST \
  'http://localhost:3005/v1/users' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
    "username": "detective1",
    "email": "detective1@test.com",
    "password": "test"
  }
}'
```




### Read Information About a User

Making a GET request to a resource under the `/v1/users/{{user-id}}` endpoint will return the user listed under that id.
The `X-Auth-token` must be supplied in the headers.

#### :five: Request:

```console
curl -X GET \
  'http://{{keyrock}}/v1/users/{{user-id}}' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:

The response contains basic details of the account in question:

```json
{
    "user": {
        "id": "96154659-cb3b-4d2d-afef-18d6aec0518e",
        "username": "alice",
        "email": "alice-the-admin@test.com",
        "enabled": true,
        "admin": false,
        "image": "default",
        "gravatar": false,
        "date_password": "2018-07-30T09:56:37.000Z",
        "description": null,
        "website": null
    }
}
```



### List all Users

Obtaining a complete list of all users is a super-admin permission requiring the `X-Auth-token`  - most users will only be permitted to return users
within their own organization. Listing users can be done by making a GET request to the  `/v1/users` endpoint

#### :six: Request:

#### Response:

```json
{
    "users": [
        {
            "id": "06a2140f-ccc3-49e5-82a5-76bae48b38ba",
            "username": "alice",
            "email": "alice-the-admin@test.com",
            "enabled": true,
            "gravatar": false,
            "date_password": "2018-07-30T11:41:14.000Z",
            "description": null,
            "website": null
        },
        {
            "id": "27e6ae58-adc1-4aaf-a6a2-f207946ba57e",
            "username": "bob",
            "email": "bob-the-manager@test.com",
            "enabled": true,
            "gravatar": false,
            "date_password": "2018-07-30T10:01:12.000Z",
            "description": null,
            "website": null
        },
        ...etc
    ]
}
```


### Update a User

Within the GUI, users can be updated from the settings page. This can also be done from the command line
by making PATCH request to  `/v1/users/<user-id>` endpoint when the user id is known. The `X-Auth-token`
header must also be set.

#### :seven: Request:

```console
curl -X PATCH \
  'http://localhost:3005/v1/users/{{user-id}}' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 4e76ab55-bad3-4084-929a-d7c8bad5b88a' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "user": {
      "username": "alice",
      "email": "alice-the-admin@test.com",
      "enabled": true,
      "gravatar": false,
      "date_password": "2018-07-26T15:25:14.000Z",
      "description": "Alice works for FIWARE",
      "website": "http://www.fiware.org"
  }
}'
```

#### Response:

The response lists the fields which have been updated:

```json
{
    "values_updated": {
        "description": "Alice works for FIWARE",
        "website": "http://www.fiware.org"
    }
}
```


### Delete a User

Within the GUI, users can delete their account from the settings page, selecting the **Cancel Account** Option, once again a super-admin usr can do
this from the command line by sending a DELETE request to the `/v1/users/{{user-id}}` endpoint. The `X-Auth-token`
header must also be set.

#### :eight: Request:

```console
curl -X DELETE \
  'http://localhost:3005/v1/users/{{user-id}}' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

---

# Grouping User Accounts under Organizations

For any identity management system of a reasonable size, it is useful to be able to assign
roles to groups of users, rather than setting them up individually. Since user administration
is a time consuming business, it is also necessary to be able to delegate the responsibility
of managing these group of users down to other accounts with a lower level of access.

Consider our supermarket chain for example, there could be a group of users (Managers) who
can change the prices of products within the store, and another group of users (Store Detectives)
who can lock and unlock door after closing time. Rather than give access to each individual account,
it would be easier to assign the rights to an organization and then add users to the groups.

Furthermore, Alice, the **Keyrock** admininstrator does not need to explicitly add additional user
accounts to each organization herself  - she could delegate that right to an owner within each organization.
For example Bob the Regional Manager would be made the owner of the *management* organization and could
add and remove addition manager accounts (such as `manager1` and `manager2`) to that organization
whereas Charlie the Head of Security could be handed an ownership  role in the *security* organization and add
additional store detectives to that organization.

Note that Bob does not have the rights to alter the membership list of the *security* organization and
Charlie does not have the rights to alter the membership list of the *management* organization.
Furthermore neither Bob nor Charlie would be able to alter the permissions of the application themselves,
merely add and remove existing user accounts to the organization they control.

Creating an application and setting-up the permissions is not coveredhere as it is the subject of the next tutorial.


## Organization CRUD Actions

The standard CRUD actions are assigned to the appropriate HTTP verbs (POST, GET, PATCH and DELETE) under the `/v1/organizations` endpoint.

### Create an Organization


#### :nine: Request:

```
curl -X POST \
  'http://localhost:3005/v1/organizations' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
  "organization": {
    "name": "Security",
    "description": "This group is for the store detectives"
  }
}'
```

#### Response:

```json
{
    "organization": {
        "id": "18deea43-e12a-4018-a45a-664c3158780d",
        "image": "default",
        "name": "Security",
        "description": "This group is for the store detectives"
    }
}
```

### Read Organization Details

#### :one::zero: Request:

```console
curl -X GET \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:

```json
{
    "organization": {
        "id": "18deea43-e12a-4018-a45a-664c3158780d",
        "name": "Security",
        "description": "This group is for the store detectives",
        "website": null,
        "image": "default"
    }
}
```

### List all Organizations


#### :one::one: Request:

```console
curl -X GET \
  'http://{{keyrock}}/v1/organizations' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:


Note that XXXXXXX .XXXX.XXXXXX

```json
{
    "organizations": [
        {
            "role": "owner",
            "Organization": {
                "id": "18deea43-e12a-4018-a45a-664c3158780d",
                "name": "Security",
                "description": "This group is for the store detectives",
                "image": "default",
                "website": null
            }
        },
        {
            "role": "owner",
            "Organization": {
                "id": "a45f9b5a-dd23-4d0f-a0d4-e97e2d7431a3",
                "name": "Management",
                "description": "This group is for the store manangers",
                "image": "default",
                "website": null
            }
        }
    ]
}
```

### Update an Organization


#### :one::two: Request:

```console
curl -X PATCH \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: ad426eee-0be4-4fbc-99cd-a3d2ff04eaa7' \
  -H 'X-Auth-token: {{X-Auth-token}}' \
  -d '{
    "organization": {
        "name": "FIWARE Security",
        "description": "The FIWARE Foundation is the legal independent body promoting, augmenting open-source FIWARE technologies",
        "website": "http://fiware.org"
    }
}'
```


#### Response:

```json
{
    "values_updated": {
        "name": "FIWARE Security",
        "description": "The FIWARE Foundation is the legal independent body promoting, augmenting open-source FIWARE technologies",
        "website": "http://fiware.org"
    }
}
```

### Delete an Organization

#### :one::three: Request:

```console
curl -X DELETE \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 481affbe-3087-4119-90c0-c55f7d4847f3'
```


## Users within an Organization

Users within an Organization are assigned to one of types - `owner` or `member`.  The members of an organization inherit all of the
roles and permissions assigned to the organization itself. In addition, owners of an organization are able to add an remove other
members and owners.



### Add a User as a Member of an Organization

#### :one::four: Request:

```console
curl -X POST \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/member' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 9cbdb720-8d16-4503-83c5-b350248dc34b' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:

```json
{
    "user_organization_assignments": {
        "role": "member",
        "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
        "user_id": "5e482345-2c48-410e-ae03-203d67a43cea"
    }
}
```


### Add a User as an Owner of an Organization

#### :one::five: Request:

```console
curl -X POST \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/owner' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 4ade9d63-6f40-4191-87c8-912e91b3f1b0' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:

```json
{
    "user_organization_assignments": {
        "role": "owner",
        "user_id": "5e482345-2c48-410e-ae03-203d67a43cea",
        "organization_id": "18deea43-e12a-4018-a45a-664c3158780d"
    }
}
```


### List Users within an Organization


#### :one::six: Request:

```console
curl -X GET \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}/users' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 79920300-1193-45bd-9b7f-8a07ccf5ac1e' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```


#### Response:

```json
{
    "organization_users": [
        {
            "user_id": "admin",
            "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
            "role": "owner"
        },
        {
            "user_id": "5e482345-2c48-410e-ae03-203d67a43cea",
            "organization_id": "18deea43-e12a-4018-a45a-664c3158780d",
            "role": "member"
        }
    ]
}
```

### Read User Roles within an Organization


#### :one::seven: Request:

```console
curl -X GET \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles' \
  -H 'Accept: application/json' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: c99d09d6-d3b4-4b18-9902-2c030421264e' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```

#### Response:

### Remove a User from an Organization

#### :one::eight: Request:

```console
curl -X DELETE \
  'http://{{keyrock}}/v1/organizations/{{organization-id}}/users/{{user-id}}/organization_roles/member' \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'X-Auth-token: {{X-Auth-token}}'
```




# Next Steps

Want to learn how to add more complexity to your application by adding advanced features?
You can find out by reading the other [tutorials in this series](https://fiware-tutorials.readthedocs.io/en/latest)

---

## License

[MIT](LICENSE) © FIWARE Foundation e.V.









