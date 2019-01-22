---
title: Setting Up PostgreSQL
desc: Building a database from scratch.
published: true
date: 2018-12-28
categories: project
tags: postgres database
---
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
list database \l
list table \dt
\dt *.*
\dt public.*

list schema \dn

pg_dump dbname > outfile

# Login with postgres user and:
psql my_database -U postgres
# Enter the postgres password and type in the psql shell:
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION pg_trgm;

https://www.rosette.com/blog/overview-fuzzy-name-matching-techniques/
https://en.wikiversity.org/wiki/Duplicate_record_detection
