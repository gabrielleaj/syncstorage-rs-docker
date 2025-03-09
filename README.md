Docker Compose configuration to self-host a Firefox sync server based on [this](https://github.com/mozilla-services/syncstorage-rs/tree/master) and forked respository.

Update ./data/config/config.net to add init-file path

Now edit the new `.env` file to add configuration and secrets. Keep in mind the `SYNC_MASTER_SECRET` and `METRICS_HASH_SECRET` require 64 characters.

docker compose up -d --build && docker compose logs -f
```

The first time you run the application, it will do a few things:

1. MariaDB container will be pulled and on first run it will load the `./data/init/init.sql` script that creates the required databases and user permissions. This will only run during the initial setup.

2. Next the Dockerfile will build the syncserver app. This is a Rust app and all of the required dependencies will be loaded into the environment, as well as cloning the Mozilla syncstorage-rs repo. This will take several minutes.

3. Once everything is compiled and configured you should see startup logs begin to appear. Subsequent runs of `docker compose up -d` will happen much faster because the build artifacts are cached. Data is persisted in the database (`./data/config`) between restarts.

### Rebuilding Everything

In the course of setting this up, you may need to tear down and rebuild your instance. To remove persisted data and artifacts, run the following.

```bash
docker compose down
docker image rm app-syncserver
docker builder prune -af
rm -rf ./data/config
```

This will delete the compiled Rust app and any cached layers, and also delete the database data.

### Firefox Setup

Once your app is running, you can configure Firefox by updating the `about:config` settings.

`identity.sync.tokenserver.uri` needs to be set to the `SYNC_URL` configured in your `.env` file followed by `/1.0/sync/1.5`. 

>Example: http://sync.example.com:8000/1.0/sync/1.5

To confirm the sync is working you can enable success logs in `about:config` also. Set `services.sync.log.appender.file.logOnSuccess` to true. Now you should see sync logs in `about:sync-log`

Syncing is usually very quick, and when a sync occurs you can see logs in `docker compose logs -f` also.
