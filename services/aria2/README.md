# Instruction

Download the `docker-compose.yaml` file from git by using this command.

```bash
curl -O https://raw.githubusercontent.com/shibbirweb/home-lab/refs/heads/master/services/aria2/docker-compose.yaml
```

Create volumes directories where the `docker-compose.yaml` file exists.

```bash
mkdir config downloads
```

Run docker compose by using this command

```bash
docker compose up -d
```

After successful docker compose run, visit the dashboard site using this URL

`http://<your-server-ip>:6880`