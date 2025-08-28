# WebSMS deployment guide

To properly deploy WebSMS app, please follow above steps:

- Rename `compose.example.yml` to `compose.yml` and update services name if needed

```shell
mv compose.example.yml compose.yml
```

- Rename each `.env.*` to corresponding `.env` and provide correct values. See instructions related files.

```shell
mv .env.websms .env
```

- Run compose.yml and connect to database, then run these queries to create required views

```sql
-- Ongoing campaigns view
-- CREATE OR REPLACE VIEW "ongoing_campaigns" AS
SELECT
    c.id,
    CASE
        WHEN COUNT(m.id) = SUM(CASE WHEN m.status IN ('Sent', 'Delivered', 'Failed') THEN 1 ELSE 0 END)
        THEN TRUE
        ELSE FALSE
    END AS done
FROM
    campaigns c
JOIN
    messages m ON c.id = m."campaignId"
WHERE
    c.status = 'Ongoing'
GROUP BY
    c.id;
```

- Create Minio access key

Access Minio dashboard at `<HOSTNAME>:9001` and connect with `MINIO_ROOT_USER`and `MINIO_ROOT_PASSWORD`.
See .env file.

Then, create an access key from `Access Keys` menu.

Once created, set the value of `MINIO_ACCESS_KEY` and `MINIO_SECRET_KEY`.

- [Optional] Setup files auto delete rules in Minio

You can setup lifecyle rules to auto delete old files to optimize files storage space.

See `Buckets` menu and bucket `Lifecycle Rules`.

- Restart `websms-app` service
