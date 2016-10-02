Spark Singularity...*in a Private Space
=======================================

This is a fork of https://github.com/heroku/spark-singularity but modified to use a Private-L Dyno Type

Use [Spark](https://spark.apache.org) on Heroku in a single dyno...but within Heroku [Private Spaces](https://devcenter.heroku.com/articles/dyno-runtime#private-spaces-runtime). 

Production-quality Spark clusters may be deployed into [Private Spaces](https://devcenter.heroku.com/articles/dyno-runtime#private-spaces-runtime) using [spark-in-space](https://github.com/heroku/spark-in-space).

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/herokumx/spark-singularity)

This buildpack provides the following three processes, children of the main web process:
  1. Nginx proxy for
    * basic password authentication set via environment variable
      * `SPACE_PROXY_BASIC_AUTH`
      * format `username:{PLAIN}password`
  2. Spark master
    * web UI `https://your-spark-app.herokuapp.com/`
    * [REST API](http://arturmkrtchyan.com/apache-spark-hidden-rest-api) `https://your-spark-app.herokuapp.com/rest`
  3. one [Spark worker](https://spark.apache.org/docs/latest/monitoring.html)
    * `https://your-spark-app.herokuapp.com/worker`

🚨 **This app should not be scaled beyond a single dyno.** (There is no coordination mechanism between multiple instances; implicitly use `127.0.0.1:7077` as Spark Master.)


Submitting & controlling jobs
------------------------------

Because Spark Singularity is contained in a single dyno with only port 80 exposed, there are two options for submitting jobs:

1. [Spark's REST API](http://arturmkrtchyan.com/apache-spark-hidden-rest-api), proxied at `https://your-spark-app.herokuapp.com/rest`
2. Declare the [Spark jobs to submit](http://spark.apache.org/docs/latest/submitting-applications.html) on start-up by adding each classname on an individual line in `Jobfile`:

  ```bash
  org.example.SparkWordCounter
  org.example.SparkWordCluster
  ```

Source deploy
-------------

```bash
heroku create
heroku addons:create bucketeer --as SPARK_S3
heroku buildpacks:add -i 1 https://github.com/heroku/heroku-buildpack-space-proxy.git
heroku buildpacks:add -i 2 heroku/scala
heroku buildpacks:add -i 3 https://github.com/heroku/spark-in-space.git
heroku buildpacks:add -i 4 https://github.com/dpiddy/heroku-buildpack-runit.git
heroku buildpacks:add -i 5 https://github.com/kr/heroku-buildpack-inline.git
```

Sample import & query
---------------------

*These processes will run out of memory without the large 14GB RAM dynos.*

```bash
heroku scale web=0
heroku run bin/spark-local-job spark.in.space.Import -s Private-L
heroku scale web=1:Private-L
heroku logs -t
# Once complete, avoid ongoing PL dyno charges,
heroku scale web=0:Private-L
```
