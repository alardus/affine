# AFFiNE Cooking Book

## Building

Download the docker compose file and create .env:

```
cp env.example .env
```

Edit the .env file with your own values.

Build the docker container:

```bash
docker compose -f affine-compose.yaml up -d
```

## Tune limitations on the selfhosted version
By default every user (neither on self hosted) has the "free plan" which has some limitations -- no more than 3 members for each workspace and 10GB of storage for every member.

For everyone wanted to change limitations on a self-hosted instance until there's an update in the future to do this:

1. Enter Docker PostgreSQL Server
    ```bash
    docker exec -it affine_postgres psql -U affine
    ```

2. List possible feature id's
    ```sql
    select id, feature, configs from features;
    ```

3. List users with id's
    ```sql
    select * from users;
    ```

4. List features assigned to users
    ```sql
    select * from user_features;
    ```

5. Assign a feature to a user
    * feature_id = YOUR FEATURE ID YOU WANT TO ASSIGN (get it from 'List possible feature id's')
    * user_id = YOUR USER ID YOU WANT TO CHANGE (get it from 'List users with id's')

    ```sql
    update user_features set feature_id = <feature id> where user_id = '<user id>';
    ```

6. It is also possible to update the feature configs to change limitations like members for e.x.

    ```sql
    update features set configs = '{"name":"Pro","blobLimit":104857600,"storageQuota":107374182400,"historyPeriod":2592000000,"memberLimit":100,"copilotActionLimit":10}' where id = 14;
    ```

Just restart the docker container afterwards. Now the limit of the members should be 100.

Link to the issue: https://github.com/toeverything/AFFiNE/issues/6641#issuecomment-2119875179

## Setup HTTPS (for internal use)
By defaults some features are not working without HTTPS (like copy/paste from other apps). To fix this you can use a reverse proxy like Caddy.


### Caddyfile

```
your.domain.com {

        # to use existing ssl certs
        # tls /data/ssl/your_fullchain.pem /data/ssl/your_privkey.pem

        # or to use internal certs made by Caddy
        tls internal
        
        reverse_proxy destination_host:destination_port {
                header_up X-Real-IP {http.request.remote}
                transport http {
                        read_timeout 3m
                        write_timeout 3m
                }
        }
}
```

## Websockets
AFFiNE uses websocket for real-time sync. Is you use a reverse proxy you need to make sure the websocket connections are also proxied. As for example if you use Nginx-Proxy-Manager you need to enable "Websocket" in the proxy settings for this host.