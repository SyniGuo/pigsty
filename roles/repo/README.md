# Repo (ansible role)

This role will bootstrap a local yum repo and download rpm packages


Tasks:

```yaml
tasks:
  repo : Create local repo directory		TAGS: [repo, repo_dir]
  repo : Backup & remove existing repos		TAGS: [repo, repo_upstream]
  repo : Add required upstream repos		TAGS: [repo, repo_upstream]
  repo : Check repo pkgs cache exists		TAGS: [repo, repo_prepare]
  repo : Set fact whether repo_exists		TAGS: [repo, repo_prepare]
  repo : Move upstream repo to backup		TAGS: [repo, repo_prepare]
  repo : Add local file system repos		TAGS: [repo, repo_prepare]
  repo : Remake yum cache if not exists		TAGS: [repo, repo_prepare]
  repo : Install repo bootstrap packages	TAGS: [repo, repo_boot]
  repo : Render repo nginx server files		TAGS: [repo, repo_nginx]
  repo : Disable selinux for repo server	TAGS: [repo, repo_nginx]
  repo : Launch repo nginx server			TAGS: [repo, repo_nginx]
  repo : Waits repo server online			TAGS: [repo, repo_nginx]
  repo : Download web url packages			TAGS: [repo, repo_download]
  repo : Download repo packages				TAGS: [repo, repo_download]
  repo : Download repo pkg deps				TAGS: [repo, repo_download]
  repo : Create local repo index			TAGS: [repo, repo_download]
  repo : Mark repo cache as valid			TAGS: [repo, repo_download]
```

Related variables:

```yaml
#------------------------------------------------------------------------------
# REPO BUILD
#------------------------------------------------------------------------------
# this section defines how to build a local yum repo including all packages needed
# by this system. it's highly recommended to have a local yum repo on your meta node

# - local yum repo - #
repo_enabled: true                            # build local yum repo on meta nodes?
repo_name: pigsty                             # local repo name
repo_address: yum.pigsty                      # local repo host (ip or hostname, including port if not using 80)
repo_port: 80                                 # repo server listen address, must same as repo_address!
repo_home: /www                               # default repo dir location
repo_rebuild: false                           # force re-download packages
repo_exist: false

# - upstream repo - #
repo_remove: true                             # remove existing repos
repo_upstreams:                               # additional repos to be installed before downloading
  - name: base
    description: CentOS-$releasever - Base - Aliyun Mirror
    baseurl:
      - http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
      - http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
      - http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
    gpgcheck: no
    failovermethod: priority

  - name: updates
    description: CentOS-$releasever - Updates - Aliyun Mirror
    baseurl:
      - http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
      - http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
      - http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
    gpgcheck: no
    failovermethod: priority

  - name: extras
    description: CentOS-$releasever - Extras - Aliyun Mirror
    baseurl:
      - http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
      - http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
      - http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
    gpgcheck: no
    failovermethod: priority

  - name: epel
    description: CentOS $releasever - EPEL - Aliyun Mirror
    baseurl: http://mirrors.aliyun.com/epel/$releasever/$basearch
    gpgcheck: no
    failovermethod: priority

  - name: docker
    description: Docker - Aliyun Mirror
    gpgcheck: no
    baseurl: https://mirrors.aliyun.com/docker-ce/linux/centos/7/$basearch/stable

  - name: kubernetes
    description: Kubernetes - Aliyun Mirror
    gpgcheck: no
    baseurl: https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/

  - name: grafana
    description: Grafana - TsingHua Mirror
    gpgcheck: no
    baseurl: https://mirrors.tuna.tsinghua.edu.cn/grafana/yum/rpm

  - name: prometheus
    description: Prometheus and exporters
    gpgcheck: no
    baseurl: https://packagecloud.io/prometheus-rpm/release/el/$releasever/$basearch

  - name: pgdg-common
    description: PostgreSQL common RPMs for RHEL/CentOS $releasever - $basearch
    gpgcheck: no
    baseurl: https://download.postgresql.org/pub/repos/yum/common/redhat/rhel-$releasever-$basearch

  - name: pgdg12
    description: PostgreSQL 12 for RHEL/CentOS $releasever - $basearch
    gpgcheck: no
    baseurl: https://download.postgresql.org/pub/repos/yum/12/redhat/rhel-$releasever-$basearch

  - name: pgdg13-updates-testing
    description: PostgreSQL 13 for RHEL/CentOS $releasever - $basearch - Updates testing
    gpgcheck: no
    baseurl: https://download.postgresql.org/pub/repos/yum/testing/13/redhat/rhel-$releasever-$basearch

  - name: centos-sclo
    description: CentOS-$releasever - SCLo
    gpgcheck: no
    # baseurl: http://mirror.centos.org/centos/7/sclo/$basearch/sclo/
    mirrorlist: http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-sclo

  - name: centos-sclo-rh
    description: CentOS-$releasever - SCLo rh
    gpgcheck: no
    # baseurl: http://mirror.centos.org/centos/7/sclo/$basearch/rh/
    mirrorlist: http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-rh

  - name: nginx
    description: Nginx Official Yum Repo
    skip_if_unavailable: true
    gpgcheck: no
    baseurl: http://nginx.org/packages/centos/$releasever/$basearch/

  - name: haproxy
    description: Copr repo for haproxy
    skip_if_unavailable: true
    gpgcheck: no
    baseurl: https://download.copr.fedorainfracloud.org/results/roidelapluie/haproxy/epel-$releasever-$basearch/

  - name: harbottle
    description: Copr repo for harbottle
    skip_if_unavailable: true
    gpgcheck: no
    baseurl: https://copr-be.cloud.fedoraproject.org/results/harbottle/main/epel-$releasever-$basearch/

# - repo packages- #
repo_packages:
  - epel-release nginx wget yum-utils yum createrepo                                      # bootstrap packages
  - ntp uuid lz4 nc pv jq vim-enhanced make patch bash lsof wget unzip git tuned          # basic system util
  - readline zlib openssl libyaml libxml2 libxslt perl-ExtUtils-Embed ca-certificates     # basic pg dependency
  - numactl grubby sysstat dstat iotop bind-utils net-tools tcpdump socat ipvsadm telnet  # system utils
  - grafana prometheus2 pushgateway alertmanager                                          # monitor and ui
  - node_exporter postgres_exporter nginx_exporter blackbox_exporter                      # exporter
  - consul consul_exporter consul-template etcd                                           # dcs
  - ansible python python-pip python-psycopg2                                             # ansible & python
  - python3 python3-psycopg2                                                              # python3
  - haproxy keepalived dnsmasq                                                            # proxy and dns
  - docker-ce docker-ce-cli rkt                                                           # container
  - kubelet kubectl kubeadm kubernetes-cni helm                                           # kubernetes
  - postgresql12* postgis30_12* timescaledb_12 citus_12 pglogical_12                      # postgres 12 basic
  - pg_qualstats12 pg_cron_12 pg_top12 pg_repack12 pg_squeeze12 pg_stat_kcache12 wal2json12 pgpool-II-12 pgpool-II-12-extensions python3-psycopg2 python2-psycopg2
  - ddlx_12 bgw_replstatus12 count_distinct12 extra_window_functions_12 geoip12 hll_12 hypopg_12 ip4r12 jsquery_12 multicorn12 osm_fdw12 mysql_fdw_12 ogr_fdw12 mongo_fdw12 hdfs_fdw_12 cstore_fdw_12 wal2mongo12 orafce12 pagila12 pam-pgsql12 passwordcheck_cracklib12 periods_12 pg_auto_failover_12 pg_bulkload12 pg_catcheck12 pg_comparator12 pg_filedump12 pg_fkpart12 pg_jobmon12 pg_partman12 pg_pathman12 pg_track_settings12 pg_wait_sampling_12 pgagent_12 pgaudit14_12 pgauditlogtofile-12 pgbconsole12 pgcryptokey12 pgexportdoc12 pgfincore12 pgimportdoc12 pgmemcache-12 pgmp12 pgq-12 pgrouting_12 pgtap12 plpgsql_check_12 plr12 plsh12 postgresql_anonymizer12 postgresql-unit12 powa_12 prefix12 repmgr12 safeupdate_12 semver12 slony1-12 sqlite_fdw12 sslutils_12 system_stats_12 table_version12 topn_12
  - pgbouncer pg_cli pg_top pgbadger                                                      # postgres common utils
  - pgadmin4                                                                              # pg admin GUI tools
  - patroni patroni-consul patroni-etcd                                                   # these packages in pgdg are not usable for now
  - postgresql13*

repo_url_packages:
  - https://github.com/Vonng/pg_exporter/releases/download/v0.2.0/pg_exporter-0.2.0-1.el7.x86_64.rpm
  - https://github.com/cybertec-postgresql/patroni-packaging/releases/download/1.6.5-1/patroni-1.6.5-1.rhel7.x86_64.rpm
  - http://guichaz.free.fr/polysh/files/polysh-0.4-1.noarch.rpm

```