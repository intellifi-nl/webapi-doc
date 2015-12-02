# Brain security
Starting with release 2.0, the Brain graphical interface, web API and socket.io event pusher have several access restrictions by default. This document describes the Brain security measures and how to work with them.

## Security measures

### Graphical user interface authentication
The graphical interface (used to manage smartspots and access apps and tools) requires logging in with an email address and password. Login sessions are valid for thirty days by default; this is configurable through the `services` resource on the API.

Passwords are not stored on the server; only a cryptographically strong fingerprint is retained. If a user loses their password it will have to be reset through one of the interfaces described in [Access management](#access-management).

### Web API authentication
Applications are required to provide an API key with every HTTP request to the web API or new connection with the socket.io event pusher. The API key can be provided using either of the following methods:
* With a query parameter --- `https://brain/api/endpoint?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
* With an HTTP request header --- `X-Api-Key: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

Currently, there are no capability differences between API keys. Every API key can perform any action on any resource, except for the protected resources `users` and `keys`.

Alternatively, being logged into the graphical frontend (i.e. having a valid session cookie) grants full access to the API. This is particularly useful for using the API manually. However, a same-origin policy is enforced for cookie authentication. Therefore browser applications written in Javascript cannot be authenticated in this way.

### Secure connections (TLS)
TLS security (i.e. `https` protocol) is enabled and enforced for Brains on \*.intellifi.nl domains. Brain virtual machine images shipped by Intellifi do not have TLS enabled by default, because this would require a valid certificate for the domain the Brain is going to be deployed on. Customers who have a Brain with their own domain name can contact Intellifi for information on how to install their own certificate so that TLS can be enabled.

## Access management
A graphical utility for managing Brain users and API keys is planned but not yet available. For now security-related administration has to be performed using one of the methods described below.

Users and keys can be viewed and manipulated using the REST API (`/api/users` and `/api/keys` endpoints). Note that this requires being logged in as a user with administrator privileges; just providing an API key is not sufficient to access these resources.

Customers who manage their own Brain and have SSH server access can also utilize a wizard-style command line tool for managing users, keys and low-level security settings (including the possibility to disable API authentication). This tool can be started by logging into SSH and executing the command `accesstool`.

## Default credentials
Every new Brain installation automatically creates one user with administrator privileges and one API key. The user credentials are supplied by Intellifi after purchasing a Brain license. The API key can then be viewed by logging in and navigating to `/api/keys`.
