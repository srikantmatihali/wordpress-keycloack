# SSO with keyclocak

License: MIT

Authors: Srikanth V Mattihalli [github](https://github.com/srikantmatihali) | [Linkedin](https://www.linkedin.com/in/srikanthvmattihalli/) | [Twitter](https://twitter.com/srikantmatihali/) | <a href="mailto:srikantmatihali@gmail.com?"><img src="https://img.shields.io/badge/gmail-%23DD0031.svg?&style=for-the-badge&logo=gmail&logoColor=white"/></a>

## About Project

As part of Demo for Wordpress Community Meetup held in month of January 18 2025, I have built this demo project to showcase the Keycloack login functionality with Wordpress
https://www.meetup.com/bengaluruwordpress/events/305584837/


Slideshare Link: https://www.slideshare.net/slideshow/single-sign-on-in-wordpress-keycloack-pdf/274948229

Please follow the below Instructions for loading Wordpress and Keycloack Applications 


## Wordpress Configurations

### Domain names settings
Open Local domain name as wordpress along with localhost

#### Update permalink
1. Go to Dashboard after login
2. Category Base: wordpress
3. Make sure you have added an entry for wordpress in your /etc/hosts file as below

```
#wordpress
127.0.0.1 wordpress

#keycloack
127.0.0.1 keycloack
```
4. Ensure no firewall or proxy is blocking access to port 81. Temporarily disable your firewall and test:
```
sudo ufw disable
```
5. Exec into the WordPress container and verify the web server is running:
```
docker exec -it wordpress bash
curl http://localhost
```
### Configure Plugin 

1. Download OpenID Connect Client from Wordpress Plugin Repository and Install
2. Login Flow Endpoint
```
http://<keycloak-server>:8080/realms/<realm-name>/protocol/openid-connect/auth
```
3. Token Endpoint
```
http://<keycloak-server>:8080/realms/<realm-name>/protocol/openid-connect/token
```
4. UserInfo Endpoint
```
http://<keycloak-server>:8080/realms/<realm-name>/protocol/openid-connect/userinfo
```
5. Redirect URL
```
http://<wordpress-server>:<port>/wp-admin/admin-ajax.php?action=openid-connect-authorize

```
6. Client Credentials

    Client ID: Must match the Client ID in Keycloak.
    Client Secret: Copy from the Credentials tab in Keycloak.

7. Ensure the scopes include at least:
    openid email profile

## Keycloack Configurations

1. Create a new Realm: my-realm

2. Go to Keycloak Admin Console → Clients → Your Client (e.g., wordpress-client).

2. Client Authentication
   Ensure Client Authentication is enabled.

3. Credentials
    Go to the Credentials tab.
    Verify the Client Secret exists.
    This secret will be used in Wordpress Configuration Site

4. Redirect URIs
```
http://wordpress:81/wp-admin/admin-ajax.php?action=openid-connect-authorize
```

5. HomeURL and Root URL and WebOrigins

```
http://wordpress:81/
```
6. Test Token Endpoint using postman / curl

```
curl -X POST \
  -d "grant_type=authorization_code" \
  -d "client_id=wordpress-client" \
  -d "client_secret=<client-secret>" \
  -d "code=<authorization-code>" \
  -d "redirect_uri=http://localhost:81/wp-admin/admin-ajax.php?action=openid-connect-authorize" \
  http://localhost:8080/realms/my-realm/protocol/openid-connect/token

```
Replace <client-secret> from Credentials Tab 
Replace <authorization-code> with a valid authorization code obtained from the authorization endpoint.

7. In the Client Scopes tab:
   Ensure the default scopes include email, profile, and any other required scopes.
   
   You can also explicitly assign these scopes:
   - Click Add Client Scope.
   - Select the relevant scopes (e.g., email, profile, roles, etc.).

## Docker Settings

Follow the configurations used in docker-compose.yml file

### Create docker network if not started
docker network create keycloak-wordpress-network

### Run docker-compose
docker-compose up --build

