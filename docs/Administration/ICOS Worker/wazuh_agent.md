---
weight: 100
---
# Wazuh

From the Wazuh [documentation](https://documentation.wazuh.com/current/index.html):


> Wazuh is a security platform that provides unified XDR and SIEM protection for endpoints and cloud workloads. The solution is composed of a single universal agent and three central components: the Wazuh server, the Wazuh indexer, and the Wazuh dashboard.

# Installation guide

## Wazuh agent

Install the GPG key:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```

### Add the repository:

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```

### Update the package information:

```bash
apt-get update
```

### Deploy a Wazuh agent

Run the following command and update the WAZUH_MANAGER IP with the correct IP.
The command supports multiple deployment options like setting the agent name and registration
password.  For more information read the official installation guide.

```bash
WAZUH_MANAGER="<manager-ip>" apt-get install wazuh-agent
```

Since we are running Wazuh Server in the Kubernetes cluster, the services are expoed via LoadBalancer and can be accessed with the NodePort.

So it is importat to edit the ossec.conf on the 

```bash
sudo vim /var/ossec/etc/ossec.conf
```

Add authd.pass

```bash
sudo touch /var/ossec/etc/authd.pass && sudo echo "password" >> authd.pass
```

Edit client - server part to look like this:

```
  <client>
    <server>
      <address>10.160.3.20</address>
      <port>32486</port>
      <protocol>tcp</protocol>
    </server>
    <config-profile>ubuntu, ubuntu20, ubuntu20.04</config-profile>
    <notify_time>10</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
    <crypto_method>aes</crypto_method>
    <enrollment>
      <enabled>yes</enabled>
      <port>32092</port>
      <agent_name>ADD AGENT HOSTNAME</agent_name>
      <groups>icos</groups>
      <authorization_pass_path>etc/authd.pass</authorization_pass_path>
    </enrollment>
  </client>
```

### Enable and start the Wazuh agent service

Run the following commands:

```bash
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

Watch the logs:

```bash
sudo tail -f /var/ossec/logs/ossec/logs
```

## Wazuh server

Wazuh server will be installed as a part of the [Controller suite](https://production.eng.it/gitlab/icos/suites/icos-controller).
