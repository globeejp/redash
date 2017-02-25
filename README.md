# abceed-publisher-console

Administration screen for publisher.
customize redash and use it.

Log in as an administrator and check the settings.
https://publisher.abceed.com (admin / Globee612)


## DB

redash parses s3's log via redshift.

So load s3 log into redshift.　(It is recommended to automate)
Connect to redshift with postgresql client and do the following.

host: cloudfront-log-analyze.czzjrseua4dw.ap-northeast-1.redshift.amazonaws.com
database: cloudfrontlog
password: Globee612
port: 5439

```
COPY tmp_cf_accesslog
  FROM 's3://log.cdn.abceed.com/E3U2OV1NEL2I0C.2016'
    CREDENTIALS 'aws_access_key_id=AKIAI5XC7ZQ4NBQC2BWA;aws_secret_access_key=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'
    DELIMITER '\t'
    IGNOREHEADER 2 TRUNCATECOLUMNS TRIMBLANKS ACCEPTINVCHARS MAXERROR AS 1000 gzip
    REGION 'ap-northeast-1';

INSERT into sound_download_log
SELECT
(tmp_cf_accesslog.request_date || ' ' || tmp_cf_accesslog.request_time )::TIMESTAMP as request_timestamp,
sound_download_log.request_timestamp as request_timestamp2
FROM tmp_cf_accesslog
LEFT JOIN sound_download_log
   ON sound_download_log.x_edge_request_id = tmp_cf_accesslog.x_edge_request_id
WHERE tmp_cf_accesslog.cs_uri_stem LIKE '%zip' AND sound_download_log.x_edge_request_id IS NULL;

```

If more publishers are added, we will add a view that only shows the publisher's data.
Then add the user to the DB so that the author can see only the view.
By connecting Redash with the user as a new data source, we realize multi-tenancy.

```
create view yadokari_sound_download_log as
select
   *
from sound_download_log
LEFT JOIN book ON book.id_book = sound_download_log.book_id
where publisher='やどかり出版';
```

The following users are already registered. (user id / pass)

* asahi / Ks6nBiQc
* yadokari / nW29qcGU

Please check DB's pg_shadow table for details.


## Deployed on AWS EC2

instance name : redash-a-02
instance id : i-0a3b59da101d1970a
ssh-key : ./redash-a-02.pem

> if you login, user root dir [~/] has [/redash] directory.
> Please pull the source latest code.


/opt/redash is runnning code. The following is set.

```
$ cat /opt/redash/.env 
#export REDASH_STATIC_ASSETS_PATH="../rd_ui/dist/"
export REDASH_LOG_LEVEL="INFO"
export REDASH_REDIS_URL=redis://localhost:6379/0
export REDASH_DATABASE_URL="postgresql://redash"
export REDASH_COOKIE_SECRET=aew1Wah3wahquahBeex6Ohfi7tailee5

export REDASH_NAME="abceed for publisher"
export REDASH_MAIL_SERVER="smtp.gmail.com"
export REDASH_MAIL_PORT="465"
export REDASH_MAIL_USE_TLS="false" # default: false
export REDASH_MAIL_USE_SSL="true" # default: false
export REDASH_MAIL_USERNAME="info@globeejp.com" # default: None
export REDASH_MAIL_PASSWORD="globee1955" # default: None
export REDASH_MAIL_DEFAULT_SENDER="info@globeejp.com" # Email address to send from

export REDASH_DATE_FORMAT="YYYY/MM/DD"
export REDASH_HOST="http://publisher.abceed.com/"


export REDASH_STATIC_ASSETS_PATH="/home/ubuntu/redash/rd_ui/app/"
#export REDASH_STATIC_ASSETS_PATH="/opt/redash/redash.1.0.0.b2521/rd_ui_old/dist/"

#export REDASH_CORS_ACCESS_CONTROL_ALLOW_ORIGIN="*"
#export REDASH_CORS_ACCESS_CONTROL_ALLOW_CREDENTIALS="true"
```

# Original README

<p align="center">
  <img title="Redash" src='http://redash.io/static/old_img/redash_logo.png' width="200px"/>
</p>
<p align="center">
    <img title="Build Status" src='https://circleci.com/gh/getredash/redash.png?circle-token=8a695aa5ec2cbfa89b48c275aea298318016f040'/>
</p>

[![Join the chat at https://gitter.im/getredash/redash](https://badges.gitter.im/getredash/redash.svg)](https://gitter.im/getredash/redash?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
[![Documentation](https://img.shields.io/badge/docs-redash.io/help-brightgreen.svg)](https://redash.io/help/)

**_Redash_** is our take on freeing the data within our company in a way that will better fit our culture and usage patterns.

Prior to **_Redash_**, we tried to use traditional BI suites and discovered a set of bloated, technically challenged and slow tools/flows. What we were looking for was a more hacker'ish way to look at data, so we built one.

**_Redash_** was built to allow fast and easy access to billions of records, that we process and collect using Amazon Redshift ("petabyte scale data warehouse" that "speaks" PostgreSQL).
Today **_Redash_** has support for querying multiple databases, including: Redshift, Google BigQuery, PostgreSQL, MySQL, Graphite,
Presto, Google Spreadsheets, Cloudera Impala, Hive and custom scripts.

**_Redash_** consists of two parts:

1. **Query Editor**: think of [JS Fiddle](http://jsfiddle.net) for SQL queries. It's your way to share data in the organization in an open way, by sharing both the dataset and the query that generated it. This way everyone can peer review not only the resulting dataset but also the process that generated it. Also it's possible to fork it and generate new datasets and reach new insights.
2. **Dashboards/Visualizations**: once you have a dataset, you can create different visualizations out of it, and then combine several visualizations into a single dashboard. Currently it supports charts, pivot table and cohorts.

## Demo

<img src="https://cloud.githubusercontent.com/assets/71468/17391289/8e83878e-5a1d-11e6-8938-af9054a33b19.gif" width="60%"/>

You can try out the demo instance: http://demo.redash.io/ (login with any Google account).

## Getting Started

* [Setting up Redash instance](https://redash.io/help-onpremise/setup/setting-up-redash-instance.html) (includes links to ready made AWS/GCE images).
* [Documentation](https://redash.io/help/).


## Getting Help

* Issues: https://github.com/getredash/redash/issues
* Discussion Forum: https://discuss.redash.io/
* Slack: http://slack.redash.io/
* Gitter (chat): https://gitter.im/getredash/redash

## Reporting Bugs and Contributing Code

* Want to report a bug or request a feature? Please open [an issue](https://github.com/getredash/redash/issues/new).
* Want to help us build **_Redash_**? Fork the project, edit in a [dev environment](https://redash.io/help-onpremise/setup/setting-up-development-environment-using-vagrant.html), and make a pull request. We need all the help we can get!

## License

BSD-2-Clause.
