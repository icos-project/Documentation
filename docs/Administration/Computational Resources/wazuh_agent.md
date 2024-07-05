# **Wazuh Agent Installation** 
The agent installation instructions are available on [Wazuh documentation](https://documentation.wazuh.com/4.7/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)

The instructions are only for installing Wazuh agent on [Ubuntu](https://ubuntu.com).
The steps are:

1. Add the Wazuh repository

Run the following commands:

* Install the GPG key:

```console
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```

* Add the repository:

```console
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
* Update the package information:

```console
apt-get update
```

2. Deploy a Wazuh agent

Run the following command and update the WAZUH_MANAGER IP with the correct IP.
The command supports multiple deployment options like setting the agent name and registration
password.  For more information read the official [Installation Guide](https://documentation.wazuh.com/current/index.html).

```console
WAZUH_MANAGER="<manager-ip>" apt-get install wazuh-agent
```

3. Step3: Enable and start the Wazuh agent service
Run the following commands:

```console
systemctl daemon-reload

systemctl enable wazuh-agent

systemctl start wazuh-agent
```