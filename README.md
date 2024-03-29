![PadoGrid](https://github.com/padogrid/padogrid/raw/develop/images/padogrid-3d-16x16.png) [*PadoGrid*](https://github.com/padogrid) | [*Catalogs*](https://github.com/padogrid/catalog-bundles/blob/master/all-catalog.md) | [*Manual*](https://github.com/padogrid/padogrid/wiki) | [*FAQ*](https://github.com/padogrid/padogrid/wiki/faq) | [*Releases*](https://github.com/padogrid/padogrid/releases) | [*Templates*](https://github.com/padogrid/padogrid/wiki/Using-Bundle-Templates) | [*Pods*](https://github.com/padogrid/padogrid/wiki/Understanding-Padogrid-Pods) | [*Kubernetes*](https://github.com/padogrid/padogrid/wiki/Kubernetes) | [*Docker*](https://github.com/padogrid/padogrid/wiki/Docker) | [*Apps*](https://github.com/padogrid/padogrid/wiki/Apps) | [*Quick Start*](https://github.com/padogrid/padogrid/wiki/Quick-Start)

---

<!-- Platforms -->
[![PadoGrid 1.x](https://github.com/padogrid/padogrid/wiki/images/padogrid-padogrid-1.x.drawio.svg)](https://github.com/padogrid/padogrid/wiki/Platform-PadoGrid-1.x) [![Host OS](https://github.com/padogrid/padogrid/wiki/images/padogrid-host-os.drawio.svg)](https://github.com/padogrid/padogrid/wiki/Platform-Host-OS) [![VM](https://github.com/padogrid/padogrid/wiki/images/padogrid-vm.drawio.svg)](https://github.com/padogrid/padogrid/wiki/Platform-VM) [![Docker](https://github.com/padogrid/padogrid/wiki/images/padogrid-docker.drawio.svg)](https://github.com/padogrid/padogrid/wiki/Platform-Docker) [![Kubernetes](https://github.com/padogrid/padogrid/wiki/images/padogrid-kubernetes.drawio.svg)](https://github.com/padogrid/padogrid/wiki/Platform-Kubernetes)

# Geode/GemFire WAN

The `wan` bundle includes two (2) local clusters configured with bidirectional WAN gateways. You can test the bundle immediately after installation. No configurations required.

To understand how the clusters are configured, please see the following WAN example. It provides step-by-step instructions for creating and running WAN enabled clusters from scratch.

[Geode/GemFire WAN Example](https://github.com/padogrid/padogrid/wiki/Geode-WAN-Example)

## Table of Contents

- [Installing Bundle](#installing-bundle)
- [Use Case](#use-case)
- [Required Software](#required-software)
- [Bundle Contents](#bundle-contents)
- [Configuring Bundle Environment](#configuring-bundle-environment)
- [Startup Sequence](#startup-sequence)
  - [1. Switch Workspace](#1-switch-workspace)
  - [2. Start Clusters](#2-start-clusters)
  - [3. Monitor Clusters](#3-monitor-clusters)
- [Test Cases](#test-cases)
  - [Test Case 1. Parallel Gateway - Region `summary`](#test-case-1-parallel-gateway---region-summary)
  - [Test Case 2. Parallel Gateway - Region `map1`](#test-case-2-parallel-gateway---region-map1)
  - [Test Case 3. Serial Gateway - Region `map2`](#test-case-3-serial-gateway---region-map2)
  - [Test Case 4. Parallel Gateway - Co-located Regions `/nw/customers` and `/nw/orders`](#test-case-4-parallel-gateway---co-located-regions-nwcustomers-and-nworders)
    - [REST API](#rest-api)
    - [Swagger UI](#swagger-ui)
    - [`run_query` Command](#run_query-command)
 - [Useful Scripts](#useful-scripts)
   - [run_clear](#run_clear)
   - [run_query](#run_query)
 - [Teardown](#teardown)
 - [References](#references)

## Installing Bundle

Install the bundle as a workspace by executing one of the following:

```bash
# To run
install_bundle -download -workspace bundle-geode-1-app-perf_test_wan-cluster-ln-ny

# To run and/or check in
install_bundle -checkout bundle-geode-1-app-perf_test_wan-cluster-ln-ny
```

## Use Case

In this use case, two (2) local clusters are configured to replicate select  Geode/GemFire regions across the WAN. With the included `perf_test_wan` app you can ingest data into the `ny` cluster and monitor the WAN replication taking place into the `ln` cluster using Geode/GemFire Pulse in the browser.

![WAN Diagram](images/wan-ny-ln.png)

## Required Software

- Maven 3.x
- Geode 1.x or GemFire 9.x/10.x

## Bundle Contents

```console
apps
└── perf_test_wan

clusters
├── ln
└── ny
```

## Configuring Bundle Environment

Make sure you have all the required products installed.

```bash
# To use Geode:
install_padogrid -product geode
update_padogrid -product geode

# To use GemFire:
# GemFire must be downloaded manually from the their website.
# The install_padogrid usage provides the download link.
install_padogrid -?
# Upon download, install it in $PADOGRID_ENV_BASE_PATH/products directory.
# For example, the following installs GemFire 10.1.0.
tar -C $PADOGRID_ENV_BASE_PATH/products -xzf ~/Downloads/vmware-gemfire-10.1.0.tgz
# Update the workspace environment with the installed GemFire version.
update_padogrid -product gemfire
```

## Startup Sequence

### 1. Switch Workspace

Switch into the bundle workspace to set the current workspace context.

```bash
switch_workspace bundle-geode-1-app-perf_test_wan-cluster-ln-ny
```

### 2. Start Clusters

Add a locator and two (2) members to both clusters. You can add more members if you have enough memory. Each member is configured with the maxiumu heap size of 1GB.

```bash
# ny
switch_cluster ny
add_locator; add_member -count 2

# ln
switch_cluster ln
add_locator; add_member -count 2
```

Run the clusters by executing one of the following options.

```bash
# Start both (all) clusters in the workspace
start_workspace -quiet

# You can also start clusters individually as follows
start_cluster -cluster ny
start_cluster -cluster ln
```

You can also optionally group the clusters to manage them with a single command. The following example creates a group named `wan` and adds both clusters to it.

```bash
create_group -group wan -product geode -count 0
add_cluster -group wan -cluster ny
add_cluster -group wan -cluster ln
```

The group you created can be managed using the `*_grroup` commands as follows.

```bash
# Switch to the wan group
switch_group wan

# Start all clusters belonging to the current group
start_group
```

### 3. Monitor Clusters

The clusters can be monitored in a number of ways, i.e., by Pulse, gfsh, VSD, JMX, Grafana, geode-addon, etc. Let's use Pulse for our example. You can view the gateway sender activities from Pulse. To get the Pulse URLs for both clusters run the following:

```bash
# View all clusters in the workspace
show_cluster -all

# View clusters individually
show_cluster -long -cluster ny
show_cluster -long -cluster ln

# If you created group, then 'show_group' also displays both clusters
show_group -long
```

You should see the following URLs from the command outputs.

- **ny:** [http://localhost:7070/pulse](http://localhost:7070/pulse)
- **ln:** [http://localhost:7080/pulse](http://localhost:7080/pulse)

✏️  Note that only one instance of Pulse can be viewed per browser. To view both clusters, you need to open two different browsers, e.g., view `ny` from Chrome and `ln` from Firefox.

## Test Cases

- [Test Case 1. Parallel Gateway - Region `summary`](#test-case-1-parallel-gateway---region-summary)
- [Test Case 2. Parallel Gateway - Region `map1`](#test-case-2-parallel-gateway---region-map1)
- [Test Case 3. Serial Gateway - Region `map2`](#test-case-3-serial-gateway---region-map2)
- [Test Case 4. Parallel Gateway - Co-located Regions `/nw/customers` and `/nw/orders`](#test-case-4-parallel-gateway---co-located-regions-nwcustomers-and-nworders)

### Test Case 1. Parallel Gateway - Region `summary`

In this test case, we replicate data from the `ny` cluster to the `ln` cluster using **parallel** gatway senders. Both clusters have been configured with a parallel gateway named, `ny-to-ln-summary`. You can view the configuration file as follows.

```bash
cd_cluster ny
cat etc/cache.xml
```

Content of `cache.xml`:

```xml
  ...
  <!-- Parallel event queue for summary -->
  <gateway-sender id="ny-to-ln-summary" remote-distributed-system-id="2"
      parallel="true" dispatcher-threads="5" maximum-queue-memory="150">
  </gateway-sender>
  ...
```
      
There are three (3) regions configured to store co-located data: `eligibility`, `profile`, and `summary`. The `eligibility` and `profile` regions store member group benefit information. The `summary` region stores member group summary information which is aggregated by invoking [`EligFunction`](https://github.com/padogrid/padogrid/blob/develop/geode-addon-core/src/test/java/org/apache/geode/addon/test/perf/EligFunction.java) via `FunctionService.onRegion()`. `EligFunction` aggregates data from the co-located regions, `eligibility` and `profile`.

✏️  The included `perf_test_wan` app is the same `perf_test` app that you can also install by running the `create_app` command. There are no configuration differences between the two. Either one will properly populate data into both clusters.

The `perf_test_wan` app connects to the `ny` cluster so the `ny` cluster is the sender and `ln` is the receiver.

Change directory into the `perf_test_wan`'s `bin_sh` directory.

```bash
cd_app perf_test_wan/bin_sh
```

First, ingest data into the `eligibility` and `profile` regions. 

```bash
./test_ingestion -run
```

Next, generate transactional data into the `summary` region which is configured to replicate between the clusters. The following command invokes [`EligFunction`](https://github.com/padogrid/padogrid/blob/develop/geode-addon-core/src/test/java/org/apache/geode/addon/test/perf/EligFunction.java) for each member group ID.

```bash
./test_tx -run
```

You can monitor the Pulse instances to see the `summary` region being populated with [`GroupSummary`](https://github.com/padogrid/padogrid/blob/develop/geode-addon-core/src/test/java/org/apache/geode/addon/test/perf/data/GroupSummary.java) data.

### Replicating from `ln` to `ny`

If you want to send from the `ln` cluster to the `ny` cluster then change the locator port in the `client-cache.xml` file.

```bash
cd_app perf_test_wan
vi etc/client-cache.xml
```

Change the locator port number from 10333 (ny) to 10344 (ln).

```xml
<pool name="serverPool">
   <locator host="localhost" port="10344" />
</pool>
```

Run `test_tx` as before.

```bash
cd_app perf_test_wan/bin_sh
./test_tx -run
```

### Test Case 2. Parallel Gateway - Region `map1`

In this test case, we use another parallel gateway to replicate data from `ny` to `ln`. The `map` region has been configured with a parallel gateway named, `ny-to-ln-parallel`. Note that we cannot use the previous test case's `ny-to-ln-summary` gateway since in this test case, we have not co-located data.

```bash
cd_cluser nyc
cat etc/cache.xml
```

Content of `cache.xml`:

```xml
  ...
  <!-- Parallel event queue -->
  <gateway-sender id="ny-to-ln-parallel" remote-distributed-system-id="2"
      parallel="true" dispatcher-threads="5" maximum-queue-memory="150" order-policy="partition">
  </gateway-sender>
  ...
```

Execute the following to ingest data into the `map1` region.

```bash
cd_app perf_test_wan/bin_sh
./test_group -run -prop ../etc/group-map1.properties
```

Monitor the Pulse instances to view data getting replicated from `ny` to `ln`.

### Test Case 3. Serial Gateway - Region `map2`

A serial gateway sender funnels region events through a single Geode server in the local cluster to a gateway receiver in the remote Geode cluster. All other servers configured with the same gateway replicate the region events but do not send them to the gateway receiver. They remain standby until a failover occurs in which time one of them assumes the sender responsibility. For this reson, we should limit the number of serial gateway servers to a small number.

For our test case, only the first two (2) servers are preconfigured with a serial gateway. You can change the number of serial gateway servers by editing the `bin_sh/setenv.sh` file as follows.

```bash
# ny
cd_cluster ny/bin_sh
vi setenv.sh

# ln
cd_cluster ln/bin_sh
vi setenv.sh
```

Set `SERIAL_GATEWAY_SERVER_COUNT` in both cluster's `setenv.sh`:

```bash
SERIAL_GATEWAY_SERVER_COUNT=2
```

Both clusters have been configured with a serial gateway named, `ny-to-ln-serial`. You can view the configuration file as follows.

```bash
cd_cluster ny
cat etc/cache.xml
```

Content of `cache.xml`:

```xml
  ...
  <!-- Serial event queue -->
  <gateway-sender id="ny-to-ln-serial" remote-distributed-system-id="2"
      parallel="false" dispatcher-threads="5" maximum-queue-memory="150"
      order-policy="key">
  </gateway-sender>
  ...
```

The `map2` region has been configured with the `ny-to-ln-serial` gateway. You can use `perf_test` to ingest data in the `map2` region as follows.

```bash
cd_app perf_test_wan/bin_sh
./test_group -run -prop ../etc/group-map2.properties
```

Monitor the Pulse instances to view data getting replicated from `ny` to `ln`.

### Test Case 4. Parallel Gateway - Co-located Regions `/nw/customers` and `/nw/orders`

In this test case, we WAN-replicate two (2) regions that have been co-located. The `/nw/customers` and `/nw/orders` regions have been preconfigured to co-locate data using the PadoGrid's [`IdentityKeyPartitionResolver`](https://github.com/padogrid/padogrid/blob/develop/geode-addon-core/src/main/java/org/apache/geode/addon/cluster/cache/IdentityKeyPartitionResolver.java). You can view the configuration file as follows.

```bash
cd_cluster ny/etc
vi cache.xml
```

Content of `cache.xml`

```xml
...
   <region-attributes id="customerIdentityKey">
      <partition-attributes colocated-with="/nw/customers">
         <partition-resolver>
            <class-name>org.apache.geode.addon.cluster.cache.IdentityKeyPartitionResolver</class-name>
            <parameter name="indexes">
               <string>1</string>
            </parameter>
         </partition-resolver>
      </partition-attributes>
   </region-attributes>
   ...
   <region name="nw">
      ...
      <region name="customers" refid="PARTITION">
         <region-attributes gateway-sender-ids="ny-to-ln-customers" />
      </region>
      ...
      <region name="orders" refid="PARTITION">
         <region-attributes refid="customerIdentityKey" gateway-sender-ids="ny-to-ln-customers" />
     </region >
   ...
   </region>
...
```

`IdentityKeyPartitionResolver` is a generic partition resolver that works with any string keys conforming to the following default format.

```console
<key>.<routing-key>
```

For our test case, we supply the customer ID as the routing key as follows.

```console
<key>.<customerId>
```

For example, if the customer's key is `000000-0017` in the `/nw/customers` region, then the orders belonging to that customer in the `/nw/orders` region would have keys similar to the following.

```console
k0000009651.000000-0017
k0000000869.000000-0017
k0000002724.000000-0017
k0000003384.000000-0017
...
```


✏️ *`IdentityKeyPartitionResolver` supports keys with delimiters other than '.' (period),  multiple delimiters, and custom delimiter sequences.*

Note that the `/nw/customers` region is not configured with the `customerIdentityKey` region attributes. This is because the routing keys are customer IDs, and customer IDs are actual keys for the `/nw/customers` region. If we have configured the `/nw/customers` with the same `customerIdentityKey` region attributes, then we will see the following error message in the member log files.

```console
java.lang.IllegalStateException: Region specified in 'colocated-with' (/nw/customers) for region /nw/customers does not exist. It should be created before setting 'colocated-with' attribute for this region.
```

Before we can ingest mock data into the `/nw/customers` and `/nw/orders` regions, we need to run the `build_app` script to download the `javafaker` package required by `perf_test` for generating mock data.

```bash
cd_app perf_test_wan/bin_sh
./build_app
```

Let's now ingest mock data into the `/nw/customers` and `/nw/orders` regions.

```bash
cd_app perf_test_wan/bin_sh

# PadoGrid v0.9.17 or above - provides more quantized entity relationships
./test_group -prop ../etc/group-factory-er.properties -run

# PadoGrid v0.9.16 or less
./test_group -prop ../etc/group-factory.properties -run
```

Monitor both Pulse instances to see the regions getting populated.

### REST API

One of the benefits of co-located data is that you can equi-join regions together using OQL. This can only be done in functions, however. PadoGrid provides another generic plugin that we can use for this purpose. The `addon.QueryFunction` plugin has already been deployed in the PadoGrid workspace and we can simply run the following OQL statement using the `curl` command to equi-join `/nw/customers` and `/nw/orders`.

OQL:

```sql
select distinct * from /nw/customers c, /nw/orders o where c.customerId=o.customerId order by c.customerId limit 100
```

Execute the following `curl` command to invoke the `addon.QueryFunction`.

```bash
curl -X POST "http://localhost:7080/geode/v1/functions/addon.QueryFunction?onRegion=%2Fnw%2Forders" \
     -H "accept: application/json" -H "Content-Type: application/json" \
     -d "[ { \"@type\": \"String\",\"@value\": \"select distinct * from /nw/customers c, /nw/orders o where c.customerId=o.customerId order by c.customerId limit 100\"}]"
```

You should see outputs similar to the following.

```console
[ [ {
  "c" : {
    "createdOn" : "06/01/2022",
    "updatedOn" : "06/01/2022",
    "customerId" : "000000-0017",
    "companyName" : "Russel-Barrows",
    "contactName" : "McDermott",
    "contactTitle" : "Central Manager",
    "address" : "Apt. 892 5479 Dewitt Bypass, Eulahborough, NH 90390",
    "city" : "South Zackaryview",
    "region" : "MD",
    "postalCode" : "19374-0549",
    "country" : "Uruguay",
    "phone" : "1-423-878-7980",
    "fax" : "258-478-9848"
  },
  "o" : {
    "createdOn" : "06/01/2022",
    "updatedOn" : "06/01/2022",
    "orderId" : "k0000007416",
    "customerId" : "000000-0017",
    "employeeId" : "705328+5656",
    "orderDate" : "05/27/2022",
    "requiredDate" : "06/02/2022",
    "shippedDate" : "06/01/2022",
    "shipVia" : "4",
    "freight" : 95.39,
    "shipName" : "Cassin LLC",
    "shipAddress" : "Suite 215 1566 Sipes Meadows, West Terrance, WY 31508-9261",
    "shipCity" : "New Genaroview",
    "shipRegion" : "ND",
    "shipPostalCode" : "78439",
    "shipCountry" : "Germany"
  }
}, {
...
```

### Swagger UI

You can also execute the query using the Swagger UI. The Swagger UI URLs can be obtained by running `show_bundle -long` as follows.

```bash
show_cluster -long -cluster ny
show_cluster -long -cluster ln
```

Output:

```
...
01        Member: ln-padomac.local-01
           STATE: Running
             PID: 81065
     MEMBER_PORT: 40414
 MEMBER_HTTP_URL: http://localhost:7090/geode/swagger-ui.html
...
02        Member: ln-padomac.local-02
           STATE: Running
             PID: 81066
     MEMBER_PORT: 40415
 MEMBER_HTTP_URL: http://localhost:7091/geode/swagger-ui.html
...
```

Go to one of the Swagger UI URLs and select **POST /v1/functions/{fuctionId}**.

Enter the following and click on the **Execute** button.

- argsInBody or Request body:

```json
[{"@type": "String",
"@value": "select distinct * from /nw/customers c, /nw/orders o where c.customerId=o.customerId order by c.customerId limit 100" }]
```

- functionId:

```console
addon.QueryFunction
```

- onRegion:

```console
/nw/customers
```

![Swagger Screenshopt](images/swagger-addon-query-function.png)

### `run_query`

You can also execute queries in a simpler way by running the `run_query` script found in the `bin_sh` directory. The following executes the default equi-join query statement we used in the previous examples.

```bash
cd_app perf_test_wan/bin_sh
./run_query
```

You can use `run_query` to target clusters and members. Please see the next section for the usage.

## Useful Scripts

This bundle includes the following additional scripts for your convenience.

### `run_clear`

```bash
cd_app perf_test_wan/bin_sh
./run_clear -?
```

Output:

```console
NAME
   run_clear - Execute 'addon.ClearFunction' via the REST API to clear the specified region

SYNOPSIS
   run_clear [-cluster cluster_name] [-num member_number] [-query "oql_statement"] [-region target_region_path] [-?]

DESCRIPTION
   Executes 'addon.ClearFunction' via the REST API to clear the specified region.

   By default, the REST API is invoked on the first member of the current cluster. You can target
   a specific cluster and member using the '-cluster' and '-num' options, respective.

   'addon.ClearFunction' is executed on all members using 'onMembers'. Each member is responsible
   for clearing their own primary set of partitioned data. If the specified region is a replicated region,
   then the same effect takes place except that each member clears the entire region. We can do better
   by having only one member to clear replicated regions. This is an improvement that can be added in the future.

   Note that unlike 'addon.QueryFunction', 'addon.ClearFunction' does not require a target region.

OPTIONS
   -region region_path
             The region to clear. Required.

   -cluster cluster_name
             Cluster name. Default: ny

   -num member_number
             The number of member to execute the REST API on. Default: 1

DEFAULT
   ./run_clear -cluster ny -num 1 -region region_path
```

### run_query

```bash
cd_app perf_test_wan/bin_sh
run_query -?
```

```console
NAME
   run_query - Execute 'addon.QueryFunction' via the REST API to execute ad-hoc queries

SYNOPSIS
   run_query [-cluster cluster_name] [-num member_number] [-region target_region_path] [-query "oql_statement"] [-?]

DESCRIPTION
   Executes 'addon.QueryFunction' via the REST API to execute ad-hoc queries. If the '-query' option
   is not specified then it executes the following equi-join query statement.

   select distinct * from /nw/customers c, /nw/orders o where c.customerId=o.customerId order by c.customerId limit 100

   By default, the REST API is invoked on the first member of the current cluster. You can target
   a specific cluster and member using the '-cluster' and '-num' options, respective.

   Note that 'addon.QueryFunction' is executed using 'onRegion'. This means the target region to
   execute the function on must be a partitioned region. This script assigns the target region
   to the first region path found in the 'from' clause. Optionally, you can specify the target
   region using the '-region' option.

OPTIONS
   -query "oql_statement"
             OQL query statement in double quotes. If this option is not specified then it defaults
             to the equi-join statement shown above.

   -cluster cluster_name
             Cluster name. Default: ny

   -num member_number
             The number of member to execute the REST API on. Default: 1

   -region target_region_path
             The target region to execute the query, i.e., onRegion. If this option is not specified
             then this script assigns the first region extracted from the 'from' clause as the target
             region.

DEFAULT
   ./run_query -cluster ny -num 1 -query "select distinct * from /nw/customers c, /nw/orders o where c.customerId=o.customerId order by c.customerId limit 100"
```

## Teardown

```bash
# Stop all custers in the workspace - '-all' stops both members and locators
stop_workspace -all
```

## References

1. Geode/GemFire WAN Example, https://github.com/padogrid/padogrid/wiki/Geode-WAN-Example

---

![PadoGrid](https://github.com/padogrid/padogrid/raw/develop/images/padogrid-3d-16x16.png) [*PadoGrid*](https://github.com/padogrid) | [*Catalogs*](https://github.com/padogrid/catalog-bundles/blob/master/all-catalog.md) | [*Manual*](https://github.com/padogrid/padogrid/wiki) | [*FAQ*](https://github.com/padogrid/padogrid/wiki/faq) | [*Releases*](https://github.com/padogrid/padogrid/releases) | [*Templates*](https://github.com/padogrid/padogrid/wiki/Using-Bundle-Templates) | [*Pods*](https://github.com/padogrid/padogrid/wiki/Understanding-Padogrid-Pods) | [*Kubernetes*](https://github.com/padogrid/padogrid/wiki/Kubernetes) | [*Docker*](https://github.com/padogrid/padogrid/wiki/Docker) | [*Apps*](https://github.com/padogrid/padogrid/wiki/Apps) | [*Quick Start*](https://github.com/padogrid/padogrid/wiki/Quick-Start)
