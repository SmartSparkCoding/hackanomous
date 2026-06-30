# Hackanomous

monorepo for hc's next YSWS: Hackanomous!

## development

### lite

for quick frontend changes, a full setup is overkill. you're probably in codespaces, just use pnpm.

1. clone this repo! (if in a codespace, skip)
    ```
    git clone https://github.com/SmartSparkCoding/hackanomous && cd hackanomous
    ```

2. install frontend deps, incl development ones
    ```
    pnpm install
    ```

3. get to work!
    ```
    pnpm dev
    ```

### full

for anything involving backend! bun and uv strongly recommended.

1. clone this repo!
    ```
    git clone https://github.com/SmartSparkCoding/hackanomous && cd hackanomous
    ```

2. install frontend deps, incl development ones
    ```
    bun install
    ```

3. setup venv using latest python
    ```
    uv venv .venv
    ```

4. activate your venv (depending on platform)
    - windows:
        ```
        .venv/Scripts/activate
        ```
    
    - macos/linux:
        ```
        source .venv/bin/activate
        ```

5. install all backend deps, incl development ones
    ```
    uv sync
    ```

6. if you're a dev on the team, ask `technodot` for a development copy of `config.toml`. otherwise, fill in the blanks as per below:
    ```
    base_url="http://localhost:8080" # change to prod url in prod bc duh

    [database]
    url="postgresql+asyncpg://username:password@point.at.your:5432/database"

    [oauth]

    [oauth.hackclub]
    client_id="hca_client_id"
    secret="hca_client_secret" # probably don't push

    [oauth.hackatime]
    client_id="hackatime_client_id"
    secret="hackatime_client_secret" # probably don't push
    ```

    1. you will need your own postgresql db.
        - your options:
            - ~~use the bundled docker one~~ TODO
            - figure out hosting one yourself (glhf)
            - create a database on some DBaaS like Supabase/Neon (higher latency & less control, but easier)
        
        - most likely, the url you are given will NOT include the `+asyncpg://` part. INSERT IT IN. I PROMISE IT'S IMPORTANT.'
    
    2. create an HCA OAuth app at https://auth.hackclub.com/developer/apps.
        - select ALL the scopes.
        - add your callback url as `http://localhost:8080/oauth/hackclub/callback`.
        - fill in your client id and secret into `config.toml`.
    
    2. create a Hackatime OAuth app at https://hackatime.hackclub.com/oauth/applications.
        - select ALL the scopes.
        - add your callback url as `http://localhost:8080/oauth/hackatime/callback`.
        - fill in your client id and secret into `config.toml`.

7. generate a rsa keypair!
    ```
    ssh-keygen -t rsa -b 4096
    ```

    skip everything with enter, it's not very important. it should save files to `~/.ssh/`.

8. get it up and running!
    - if you're just working on backend only, you can just build the frontend once
        ```
        bun vite build
        ```
    
    - else, if you're working on both at the same time, in a new terminal, run
        ```
        bun vite build --watch
        ```
    
    - finally, run the server itself
        ```
        uv run main.py --config ./config.toml --dev
        ```

9. pull up http://0.0.0.0:8080 and enjoy!

## deployment

docker coming soon, i (don't) promise!

deployment is currently static through @SmartSparkCoding's personal vercel.

dm `technodot` on Slack for any questions!