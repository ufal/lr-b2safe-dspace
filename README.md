# Business to Safe DSpace Module

This DSpace module can be used to automatically replicate each submission after it has been approved. The submission is converted to AIP format which is uploaded to b2safe server.


### Requirements
To use this package you need to first install B2SAFE-repository-package package
```
git clone git@github.com:EUDAT-B2SAFE/B2SAFE-repository-package.git
cd B2SAFE-repository-package
mvn install -Dmaven.test.skip=true
```

### Configuration

In order to add this module to your DSpace, clone this git repository into the sources folder of DSpace.
```
git clone git@github.com:ufal/lr-b2safe-dspace.git
```

Afterwards update the following files.

* Change the parent version in lr-b2safe-dspace/pom.xml to match your DSpace version.
* Add the module in the root pom.xml of DSpace.
```
<profile>
  <id>lr-b2safe-dspace</id>
  <activation>
    <activeByDefault>true</activeByDefault>
  </activation>
  <modules>
    <module>lr-b2safe-dspace</module>
  </modules>
</profile>
```
* Add the dependency to pom.xml in the root directory and dspace-xmlui/dspace-xmlui-api/pom.xml (1.8.x) or dspace-xmlui/pom.xml (4.x+)
```
<dependency>
  <groupId>cz.cuni.mff.ufal.dspace</groupId>
  <artifactId>b2safe-dspace</artifactId>
  <version>${project.version}</version>
</dependency>
```
* Add module lr-b2safe-dspace to dspace-xmlui profile in pom.xml similar to
```
    <profile>
        <id>dspace-xmlui</id>
        <activation>
            <file>
                <exists>dspace-xmlui/pom.xml</exists>
            </file>
        </activation>
        <modules>
            <module>dspace-xmlui</module>
            <module>lr-b2safe-dspace</module>
        </modules>
    </profile>
```
* Add a new configuration module "lr.cfg" in dspace/config/modules with following properties.
```
lr.replication.on=true
lr.replication.protocol=irods
lr.replication.host=
lr.replication.port=
lr.replication.username=
lr.replication.password=
lr.replication.id=
# must end with a /
lr.replication.homedirectory=
lr.replication.zone=
lr.replication.defaultstorage=
```
* Finally add the event listener to dspace.cfg, e.g. named replication
```
# list of event listeners
event.dispatcher.default.consumers = search, browse, discovery, eperson, harvester, replication 
# consumer to maintain the browse index
event.consumer.replication.class = cz.cuni.mff.ufal.dspace.b2safe.ItemModifyConsumer
event.consumer.replication.filters = Community|Collection|Item+Create|Modify 
```

### Control Panel

To activate the Replication tab in Control Panel, apply the patch according to dspace version in patch folder.
`git apply -3 ./lr-b2safe-dspace/patch/ControlPanel_dspace5.patch`

Note: in case you see index error while applying the patch, try running `git update-index -q --refresh`.


### What is replicable

Each submission is automatically replicated after it has been approved, provided that the item is PUB (dc.rights.label).
The submission is converted to AIP format that is uploaded to the b2safe server. We use our PID in the name of each AIP e.g.,
```
irods://XXX@irods.server:XX/IRODSZone/home/dspace_1.8.2/11858_00-097C-0000-0001-487A-4-6451959456007568280.zip
irods://XXX@irods.server:XX/IRODSZone/home/dspace_1.8.2/11858_00-097C-0000-0001-487E-B-6544474914476974525.zip
```