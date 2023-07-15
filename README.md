# Telegram message forwarder (client API)

### Functionality
- Live forwarding and updating messages
- Source and target channels/chats (one-to-one, many-to-one, many-to-many) mapping
- Incoming message filters: word replacing, forward formating, skipping by keyword and more

## Prepare

1. [Create Telegram App](https://my.telegram.org/apps)

2. Obtain **API_ID** and **API_HASH**

    ![Telegram API Credentials](/README.md-images/telegramapp.png)

3. Setup Postgres database or use `InMemoryDatabase` with `USE_MEMORY_DB=true` parameter in `.env` file

4. Fill `.env`/`mirror.config.yml` config files with your data

    [.env-example](.env-example) contains the minimum environment configuration to run with an in-memory database.

    **SESSION_STRING** can be obtained by running [login.py](login.py) with provided **API_ID** and **API_HASH** environment variables. **DON'T USE** your own account.
    
    <details>
    <summary><b>.env overview</b></summary>

    ```bash
    ###########################
    #    App configuration    #
    ###########################
    
    # Telegram app ID
    API_ID=test
    # Telegram app hash
    API_HASH=test
    # Telegram session string (telethon session, see login.py in root directory)
    SESSION_STRING=test
    # Use an in-memory database instead of Postgres DB (true or false). Defaults to false
    USE_MEMORY_DB=false
    # Postgres credentials
    DATABASE_URL=postgres://user:pass@host/dbname
    # or
    DB_NAME=test
    DB_USER=test
    DB_HOST=test
    DB_PASS=test
    # Logging level (debug, info, warning, error or critical). Defaults to info
    LOG_LEVEL=info

    ###############################################
    #    Setup directions and filters from env    #
    ###############################################

    # Mapping between source and target channels/chats
    # Channel/chat id can be fetched by using @messageinformationsbot telegram bot
    # Channel id should be prefixed with -100
    # [id1, id2, id3:id4] means send messages from id1, id2, id3 to id4
    # id5:id6 means send messages from id5 to id6
    # [id1, id2, id3:id4];[id5:id6] semicolon means AND
    CHAT_MAPPING=[-100999999,-100999999,-100999999:-1009999999];
    # Remove URLs from incoming messages (true or false). Defaults to false
    REMOVE_URLS=false
    # Comma-separated list of URLs to remove (reddit.com,youtube.com)
    REMOVE_URLS_LIST=google.com,twitter.com
    # Comma-separated list of URLs to exclude from removal (google.com,twitter.com).
    # Will be applied after the REMOVE_URLS_LIST
    REMOVE_URLS_WL=youtube.com,youtu.be,vk.com,twitch.tv,instagram.com
    # Disable mirror message deleting (true or false). Defaults to false
    DISABLE_DELETE=false
    # Disable mirror message editing (true or false). Defaults to false
    DISABLE_EDIT=false
    ```
    </details>

    To setup **directions and filters** section `mirror.config.yml` can be used:

    <details>
    <summary><b>mirror.config.yml</b> overview</summary>

    ```yaml
    # (Optional) Global filters, will be applied in order
    filters:
      - ForwardFormatFilter: # Filter name under telemirror/messagefilters.py
          format: ""           # Filters arguments
      - EmptyMessageFilter
      - UrlMessageFilter:
          blacklist: !!set
            ? t.me
      - SkipUrlFilter:
          skip_mention: false

    # (Optional) Global settings
    disable_edit: true
    disable_delete: true

    # (Required) Mirror directions
    directions:
      - from: [-1001, -1002, -1003]
        to: [-100203]

      - from: [-100226]
        to: [-1006, -1008]
        # Overwrite global settings
        disable_edit: false
        disable_delete: false
        # Overwrite global filters
        filters:
          - UrlMessageFilter:
              blacklist: !!set
                ? t.me
    ```
    </details>

    ❓ Channels ID can be fetched by using [Telegram bot](https://t.me/messageinformationsbot).

    ❗ Note: never push your `.env`/`.yml` files with real crendential to a public repo. Use a separate branch (eg, `heroku-branch`) with `.env`/`.yml` files to push to git-based deployment system like Heroku.

5. Make sure the account has joined source and target channels

6. **Be careful** with forwards from channels with [`RESTRICTED SAVING CONTENT`](https://telegram.org/blog/protected-content-delete-by-date-and-more). It may lead to an account ban

## Deploy
<details>
    <summary><b>Heroku</b></summary>
<br>

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/sureshkim/telemirror)

### or via CLI:

1. Clone project

    ```bash
    git clone https://github.com/khoben/telemirror.git
    ```
2. Create new heroku app within Heroku CLI

    ```bash
    heroku create {your app name}
    ```
3. Add heroku remote

    ```bash
    heroku git:remote -a {your app name}
    ```
4. Set environment variables to your heroku app from .env by running bash script

    ```bash
    ./set_heroku_env.bash
    ```

5. Upload on heroku host

    ```bash
    git push heroku master
    ```

6. Start heroku app

    ```bash
    heroku ps:scale web=1
    ```

#### Keep up-to-date with Heroku

If you deployed manually, move to step 2.

0. Get project to your PC:

    ```bash
    heroku git:clone -a {your app name}
    ```
1. Init upstream repo (this repository or its fork)

    ```bash
    git remote add origin https://github.com/khoben/telemirror
    ```
2. Get latest changes

    ```bash
    git pull origin master
    ```
3. Push latest changes to heroku

    ```bash
    git push heroku master -f
    ```
</details>

### Locally:
1. Create and activate python virtual environment

    ```bash
    python -m venv myvenv
    source myvenv/Scripts/activate # linux
    myvenv/Scripts/activate # windows
    ```
2. Install dependencies

    ```bash
    pip install -r requirements.txt
    ```
3. Run

    ```bash
    python main.py
    ```
