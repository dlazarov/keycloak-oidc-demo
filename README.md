# Keycloak OIDC demo

Docker compose file that spins up a Keycloak instance and the labs defined in the book **Keycloak – Identity and Access Management for Modern Applications**, 2nd Edition.

The source code for the labs was obtained from the [original GitHub repository](https://github.com/PacktPublishing/Keycloak---Identity-and-Access-Management-for-Modern-Applications-2nd-Edition.git) dedicated to the book.

## Usage

### Spin up containers

This compose file was defined and tested using Podman. Replace `podman` with `docker` in the commands below if you are using Docker Engine.

To allow loading labs from different chapters of the book without having to reconfigure Keycloak, [service profiles](https://docs.docker.com/compose/how-tos/profiles/) are used in the docker compose file to allow the option to specify which set of containers to be deployed. Failing to set a profile will cause no containers to be deployed.

Valid profiles and the containers they deploy:

```text
main:
  caddy
  postgres
  keycloak
ch2:
  backend-ch2
  frontend-ch2
ch4:
  frontend-ch4
ch5:
  backend-ch5
  frontend-ch5
```

To deploy a specific profile, use the `--profile <profile-name>` option. Multiple profiles can be specified for the same command.

```bash
# Spin up Caddy, Keycloak and the chapter 2 lab
podman compose --profile main --profile ch2 up -d
```

Before moving on to the next chapter destroy the containers deployed for the current profile. *While destroying a specifc chapter, the `default` network will fail to be destroyed. The network can only be destroyed when the main profile is also destroyed.*

```bash
# Switch from lab 2 to lab 3
podman compose --profile ch2 down
podman compose --profile ch3 up
```

### Trust CA

When Caddy starts up for the first time, it will generate a self-signed root CA together with an intermediate certificate inside the `.caddy-data` directory. You need to add this CA to your machine or browser trust store in order to use HTTPS for the hostnames defined in the `Caddyfile` for Keycloak and the active lab.

To do this, create a bundle that contains both the intermediate and the root certificates:

```bash
cat .caddy-data/caddy/pki/authorities/local/intermediate.crt \
    .caddy-data/caddy/pki/authorities/local/root.crt > localhost.crt
```

Add the bundle to the trust store in your browser or your machine.

For Firefox:

1. Navigate to the [Privacy & Security settings page](about:preferences#privacy).
2. Look for the `Certificates` heading and click on `View Certificates...`
3. In the `Authorities` tab, click on `Import...` and choose the `localhost.crt` file.
4. If the import was successful, you should see the Authority named `Caddy Local Authority - ECC Intermediate` in the list.

### Access Keycloak and the labs

The Keycloak administration console can be accessed at [https://keycloak.localhost](https://keycloak.localhost).

The frontend for the active lab can be accessed at [https://demo.localhost](https://demo.localhost).
