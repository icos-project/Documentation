# Lomos-API2

Lomos-API2 is REST API to get summary about processed logs.
It returns number of log lines with anomaly_score above threashold in given time window.

Why lomos-api2 (and not just api) - because lomos-controller already includes api.

# Usage


Sample API call:

```shell
curl "http://127.0.0.1:5000/api/top_anomaly?min_anomaly_score=0.7&from_timestamp=2024-01-08T00:00:01.200000Z&to_timestamp=2024-01-18T00:00:01.200000Z" -v
```

Sample response:

```json
{
  "aggregate": {
    "count": 1,
    "max_anomaly_score": 0.7692307978868484,
    "min_anomaly_score": 0.7692307978868484
  },
  "messages": [
    {
      "_index": "some_app_backend",
      "_type": "_doc",
      "_id": "D01aa40BjZ7m1DvJ_-y0",
      "_score": 1,
      "_source": {
        "timestamp": "2024-01-17T10:06:03.000014",
        "logs_full": "2024-01-17 10:06:03.000014 DEBUG BaseJdbcLogger:159 - <==    Updates: 1",
        "anomaly_score": 0.7692307978868484
      }
    }
  ]
}
```

# Development

```shell
python3.9 -m venv .venv
source .venv/bin/activate

pip install -e .
cp .env.sample .env
nano .env

flask --app lomos_api2/lomos_api2 --debug run

curl http://127.0.0.1:5000/
curl "http://127.0.0.1:5000/api/top_anomaly?min_anomaly_score=0.7&from_timestamp=2024-01-08T00:00:01.200000Z&to_timestamp=2024-01-18T00:00:01.200000Z" -v
```

# Build

```shell
LOMOS_API2_VERSION=0.0.1  # TODO use git tag
docker build --build-arg "LOMOS_API2_VERSION=$LOMOS_API2_VERSION" -t lomos-api2:$LOMOS_API2_VERSION .
```

# Run

```shell
docker run -it --name lomos-api2 --rm -p25001:25001 --env-file=.env lomos-api2:$LOMOS_API2_VERSION
# TODO container IP
curl "http://172.17.0.2:25001/api/top_anomaly?min_anomaly_score=0.7&from_timestamp=2024-01-08T00:00:01.200000Z&to_timestamp=2024-01-18T00:00:01.200000Z"
```

or

```shell
kubectl apply -f k8s/lomos-api2.yml
```
