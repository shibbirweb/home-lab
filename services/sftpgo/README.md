# Instruction

Download the `docker-compose.yaml` file from git by using this command.

```bash
curl -O https://raw.githubusercontent.com/shibbirweb/home-lab/refs/heads/master/services/sftpgo/docker-compose.yaml
```

Create volumes directories where the `docker-compose.yaml` file exists.

```bash
mkdir config files
```

Run docker compose by using this command

```bash
HOST_UID=$(id -u) HOST_GID=$(id -g) docker compose up -d
```

After successful docker compose run, visit the dashboard site using this URL

`http://<your-server-ip>:8080`