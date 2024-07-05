# Deploy an Application

To deploy an application, first make sure you have a valid application definition according to the section above. Then, verify that the port and address of the shell backend or your controller are reachable from the device you plan to run the icos-shell from (VPN might be needed).
Next, download the latest ICOS shell binary and create a configuration.yaml file for your environment as follows:

```yaml
controller: CONTROLLER_ADDRESS:PORT
keycloak:
	pass: KEYCLOAK_PASSWORD
	user: KEYCLOAK_USER
lighthouse: LIGHTHOUSE_ADDRESS:PORT

```

??? note
	you do not have to define the controller address in the config file, 
	you can also just rely on the lighthouse.

First of all, make sure to log in to retrieve an authentication token (saved by the shell):

```bash
./icos-shell --config=configuration.yml auth login
```

You will have to repeat this every couple of minutes if your requests are declined with error 500.
Deploy application like this:

```bash
./icos-shell --config=configuration.yml create deployment --file app_descriptor.yaml
```
Check if you application was deployed successfully:

```bash
./icos-shell --config=configuration.yml get deployment
```
For more details, refer to the README.md file of the repository.

