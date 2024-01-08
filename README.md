# Headscale stack

Run headscale with ease.

Subprojects:

- Web interface
  [headscale-management](https://github.com/parallelo3301/headscale-management)
- Integration layer
  [headscale-controller](https://github.com/parallelo3301/headscale-controller)

## Setup

1. Clone this repository and go to the directory

```bash
git clone git@github.com:parallelo3301/headscale-stack.git .
cd headscale-stack
```

2. Create a `.env` file by copying the `.env.example` file and modify it to your
   needs

```bash
cp .env.example .env
nano .env
```

3. Generate a encryption key and set it in `.env` file as `ENCRYPTION_KEY`
   variable

```bash
openssl rand -base64 32
```

4. Create a `config.yaml` in `config` directory by copying the
   `config.yaml.example` file

```bash
cp config/config.yaml.example config/config.yaml
nano config/config.yaml
```

You will most probably want to change the following values:

- `server_url`

5. Run the stack

```bash
docker compose up -d
```

6. Obtain the API key

```bash
# note the expiration set to 1000 days, modify it to your needs
docker compose exec server headscale apikey create --expiration 1000d
```

7. If you run it behind a reverse proxy, you may want to setup it.

8. Set the API key in web interface. You can access it at
   `http://localhost:5000` by default (if you didn't change the
   `MANAGEMENT_PORT` variable in `.env` file), or yours `PUBLIC_SERVER_URL`.

9. Profit

### macOS specific

1. After step 4, you will have to uncomment following section in
   `docker-compose.yaml` file:

```yaml
  # this is still part of controller service
  depends_on:
    - socat
# new service
socat:
  image: alpine/socat
  command: tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  ports:
    - 2375:2375
```

2. (optional) You can comment `- /var/run/docker.sock:/var/run/docker.sock` in
   `volumes` section of `controller` service in `docker-compose.yaml` file.

3. Uncomment line containing `USE_SOCAT=1` in your `.env` file.

## Usage

After the setup, you can use `docker compose exec server headscale` in the
container:

```bash
docker compose exec server headscale <command>

# e.g.
docker compose exec server headscale help
docker compose exec server headscale users list 
docker compose exec server headscale users create bob
```

## TODO

- [ ] Add PostgreSQL to the stack
  - [ ] Use PostgreSQL to save ACLs in more structured way
- [ ] Auth
  - [ ] Basic auth
  - [ ] OIDC
- [ ] Integrate headscale-management once it's ready
