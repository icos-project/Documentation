# Example: Zenoh multi-storages demo for ICOS 3rd F2F, Athens

This demo shows zenoh network that publish data from a python publisher, there is a subscriber who consumes the data, there are also two storages technologies, in-memory storage and influxDB storage which saves timeseries data.

In this demo you will see Zenoh deployed using docker-compose and InfluxDB docker image.

![Zenoh Demo](../../../../assets/icons/zenoh_demo.png)


# How to run the demo.

Step by step guide on reproducing the demo.

1. Clone the `zenoh-getting-started` repository, from the ICOS GitLab repository. 

```bash
git clone https://production.eng.it/gitlab/icos/data-management/examples/zenoh-getting-started.git
```

2. Go to the project directoy and start the `docker-compose.yml` as follows:

```bash
cd zenoh-getting-started
docker compose up -d
```

3. Open the Docker Desktop app, and see that the `pub` service is publishing data.

4. Check that the subscriber service `sub` is receiving the data.

5. Run the `get` service, and observe the latest data being recover.

6. Run the `get_ts` time-series get() with a time filter of the last 10 seconds of data i.e. (now - 10sec). and observe that the last 10 records are recovered.

