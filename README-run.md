### Build Neon
```SH
make -j`nproc` -s
```


### Run Neon
```SH
# Create repository in .neon with proper paths to binaries and data
# Later that would be responsibility of a package install script
cargo neon init

# start pageserver, safekeeper, and broker for their intercommunication
cargo neon start
# Starting neon broker at 127.0.0.1:50051.
# storage_broker started and passed status check, pid: 9809
# The files belonging to this database system will be owned by user "nonroot".
# This user must also own the server process.
# 
# The database cluster will be initialized with locale "en_US.utf8".
# The default database encoding has accordingly been set to "UTF8".
# The default text search configuration will be set to "english".
# 
# Data page checksums are disabled.
# 
# creating directory .neon/attachment_service_db ... ok
# creating subdirectories ... ok
# selecting dynamic shared memory implementation ... posix
# selecting default max_connections ... 100
# selecting default shared_buffers ... 128MB
# selecting default time zone ... Etc/UTC
# creating configuration files ... ok
# running bootstrap script ... ok
# performing post-bootstrap initialization ... ok
# syncing data to disk ... ok
# 
# initdb: warning: enabling "trust" authentication for local connections
# initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.
# 
# Success. You can now start the database server using:
# 
#     /workspaces/neon/pg_install/v16/bin/pg_ctl -D .neon/attachment_service_db -l logfile start
# 
# Starting attachment service database...
# localhost:1235 - no response
# .localhost:1235 - accepting connections
# 
# attachment_service_db started and passed status check, pid: 9891
# Running attachment service database setup...
# Creating database: attachment_service
# Running migrations in /workspaces/neon/control_plane/attachment_service/migrations
# Running migration 2024-01-07-211257_create_tenant_shards
# Running migration 2024-01-07-212945_create_nodes
# Migrations complete
# .
# attachment_service started and passed status check, pid: 9907
# Starting pageserver node 1 at '127.0.0.1:64000' in ".neon/pageserver_1".
# pageserver started and passed status check, pid: 9922
# Starting safekeeper at '127.0.0.1:5454' in '.neon/safekeepers/sk1'.
# safekeeper-1 started and passed status check, pid: 9963


# create initial tenant and use it as a default for every future neon_local invocation
cargo neon tenant create --set-default
# tenant 944e391778e03881ac0e64e8f1f949f4 successfully created on the pageserver
# Created an initial timeline '4703cf909f0fded12ff0d34ffc1dd22e' for tenant: 944e391778e03881ac0e64e8f1f949f4
# Setting tenant 944e391778e03881ac0e64e8f1f949f4 as a default one


# create postgres compute node
cargo neon endpoint create main

# start postgres compute node
cargo neon endpoint start main
# Starting existing endpoint main...
# Starting postgres node at 'postgresql://cloud_admin@127.0.0.1:55432/postgres'

# check list of running postgres instances
cargo neon endpoint list
#  ENDPOINT  ADDRESS          TIMELINE                          BRANCH NAME  LSN        STATUS  
#  main      127.0.0.1:55432  4703cf909f0fded12ff0d34ffc1dd22e  main         0/149F4E0  running


# connect pg and run sql
psql -p55432 -h 127.0.0.1 -U cloud_admin postgres

# SQL:
# CREATE TABLE t(key int primary key, value text);
# insert into t values(1,1);
# select * from t;
# exit


# create branch named migration_check
cargo neon timeline branch --branch-name migration_check
# Created timeline '4902d50b731b355eb45fc48a1e804b38' at Lsn 0/14CEE08 for tenant: 944e391778e03881ac0e64e8f1f949f4. Ancestor timeline: 'main'

# check branches tree
cargo neon timeline list
# main [4703cf909f0fded12ff0d34ffc1dd22e]
# ┗━ @0/14CEE08: migration_check [4902d50b731b355eb45fc48a1e804b38]

# create postgres on that branch
cargo neon endpoint create migration_check --branch-name migration_check

# start postgres on that branch
cargo neon endpoint start migration_check

# check the new list of running postgres instances
cargo neon endpoint list
#  ENDPOINT         ADDRESS          TIMELINE                          BRANCH NAME      LSN        STATUS  
#  main             127.0.0.1:55432  4703cf909f0fded12ff0d34ffc1dd22e  main             0/14CEE08  running 
#  migration_check  127.0.0.1:55434  4902d50b731b355eb45fc48a1e804b38  migration_check  0/14CEE40  running


# connect to new endpoint
psql -p55434 -h 127.0.0.1 -U cloud_admin postgres

# SQL:
# select * from t;
# insert into t values(2,2);
# select * from t;
# exit
```


### Connect to attachment_service_db
```SH
psql -p1235 -h 127.0.0.1  postgres

# SQL:
# select * from tenant_shards;
#              tenant_id             | shard_number | shard_count | shard_stripe_size | generation | generation_pageserver | placement_policy |                                                                                                                                                                                                                                                                 config                                                                                                                                                                                                                                                                 
#  ----------------------------------+--------------+-------------+-------------------+------------+-----------------------+------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------#  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------#  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
#   944e391778e03881ac0e64e8f1f949f4 |            0 |           0 |             32768 |          1 |                     1 | "Single"         | {"checkpoint_distance":null,"checkpoint_timeout":null,"compaction_target_size":null,"compaction_period":null,"compaction_threshold":null,"gc_horizon":null,"gc_period":null,"image_creation_threshold":null,"pitr_interval":null,"walreceiver_connect_timeout":null,"lagging_wal_timeout":null,"max_lsn_wal_lag":null,"trace_read_requests":null,"eviction_policy":null,"min_resident_size_override":null,"evictions_low_residence_duration_metric_threshold":null,"gc_feedback":null,"heatmap_period":null,"lazy_slru_download":null}

# select * from nodes;
#   node_id | scheduling_policy | listen_http_addr | listen_http_port | listen_pg_addr | listen_pg_port 
#  ---------+-------------------+------------------+------------------+----------------+----------------
#         1 | filling           | 127.0.0.1        |             9898 | 127.0.0.1      |          64000
```


### Stop ...
```SH
# If you want to run tests afterward (see below), you must stop all the running of the pageserver, safekeeper, and postgres instances
#    you have just started. You can terminate them all with one command:
cargo neon stop
```


### Testing ?
```SH

# deps
poetry update

# run tests
./scripts/pytest

# DEFAULT_PG_VERSION=15 BUILD_TYPE=debug ./scripts/pytest
```