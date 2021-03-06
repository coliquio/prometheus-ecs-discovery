# Prometheus Amazon ECS discovery

[Docker Image](https://hub.docker.com/repository/docker/coliquiode/prometheus-ecs-discovery)

Prometheus has native Amazon EC2 discovery capabilities, but it does
not have the capacity to discover ECS instances that can be scraped
by Prometheus.  This program is a Prometheus File Service Discovery
(`file_sd_config`) integration that bridges said gap.

## Help

Run `prometheus-ecs-discovery -?` to get information.

The command line parameters that can be used are:

* -config.scrape-interval (duration): interval at which to scrape
  the AWS API for ECS service discovery information (default 1m0s)
* -config.scrape-times (int): how many times to scrape before
  exiting (0 = infinite)
* -config.write-to (string): path of file to write ECS service
  discovery information to (default "ecs_file_sd.yml")

## Usage

First, build this program using the usual `go get` mechanism.

Then, run it as follows:

* Ensure the program can write to a directory readable by
  your Prometheus master instance(s).
* Export the usual `AWS_REGION`, `AWS_ACCESS_KEY_ID` and
  `AWS_SECRET_ACCESS_KEY` into the environment of the program,
  making sure that the keys have access to the EC2 / ECS APIs
  (IAM policies should include `ECS:ListClusters`,
  `ECS:ListTasks`, `ECS:DescribeTask`, `EC2:DescribeInstances`,
  `ECS:DescribeContainerInstances`, `ECS:DescribeTasks`,
  `ECS:DescribeTaskDefinition`).
* Start the program, using the command line option
  `-config.write-to` to point the program to the specific
  folder that your Prometheus master can read from.
* Add a `file_sd_config` to your Prometheus master:

```
scrape_configs:
- job_name: ecs
  file_sd_configs:
    - files:
      - /path/to/ecs_file_sd.yml
      refresh_interval: 10m
```

That's it.  You should begin seeing the program scraping the
AWS APIs and writing the discovery file (by default it does
that every minute, and by default Prometheus will reload the
file the minute it is written).  After reloading your Prometheus
master configuration, this program will begin informing via
the discovery file of new targets that Prometheus must scrape.
