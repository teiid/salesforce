# salesforce
### Using Custom API versions with Salesforce translator in Teiid

The default Salesforce API version used in Teiid translator and resource-adapter is version 22.
Often times, it is required by the user to use a different version of the Salesforce API with Teiid for data 
integration purposes. For Teiid to work correctly, both translator and resource-adapter must be using 
the same Salesforce API for both metadata retrieval and query execution. That means when you build a VDB, if used
version 22 for gathering the metadata, then you MUST use version 22 of the translator and resource-adapter to
execute the queries based on that metadata.

The API referenced in this document means two distinct resources, Teiid uses both for gaining access to the salesforce
 1. The URL used to connecto to Salesforce
 2. The "com.force.api" java libraries that are used to access the Salesforce.

If in case you need to access Salesforce using different URL version than 22, then you need to generate new translator and resource-adapter before you can use it in Teiid

This git repository is provided as a template for generating Teiid translator and resource-adapter based on 
default version of API that is deployed in the Teiid. This template is written for API version 34. However, you can easily update to any version of your choice. The below are some build instructions
 
```
git clone https://github.com/teiid/salesforce.git
cd salesforce
```
Now edit the pom.xml and provide the target Teiid runtime verion in the properties defined and then execute
 
```
mvn clean install
```
This will build the required artifacts as 

```
connector-34/target/connector-salesforce-34-1.0-SNAPSHOT-jboss-as7-dist.zip
translator-salesforce-34/target/translator-salesforce-34-1.0-SNAPSHOT-jboss-as7-dist.zip
````

Unzip these zip files in "<jboss-eap>/modules" directory and then edit "<jboss-eap>/standalone/configuration/standalone-teiid.xml" file
and add following in the respective sections 

Under "teiid" subsystem add
```
<translator name="salesforce-34" module="org.jboss.teiid.translator.salesforce" slot="34"/>
```

under "resource-adapters" subsystem add

```
<resource-adapter id="salesforce-34">
    <module slot="34" id="org.jboss.teiid.resource-adapter.salesforce"/>
</resource-adapter>
```

Save and close the file. Restart the server and start using the new translator and resource-adapter combination

## Example
A Dynamic VDB will look like

sf-vdb.xml
```
<vdb name="sf" version="1">
    <model visible="true" name="source">
        <source name="sf" translator-name="salesforce-34" connection-jndi-name="java:/salesforce34"/>    
    </model>
</vdb>
```

and corresponding data source definition in standalone-teiid.xml will look like

```
<resource-adapter id="salesforce-34">
    <module slot="34" id="org.jboss.teiid.resource-adapter.salesforce"/>
    <connection-definitions>
        <connection-definition class-name="org.teiid.resource.adapter.salesforce.SalesForceManagedConnectionFactory" jndi-name="java:/salesforce34" enabled="true" use-java-context="true" pool-name="sfDS34">
            <config-property name="URL">
                https://www.salesforce.com/services/Soap/u/34.0
            </config-property>
            <config-property name="username">
                user
            </config-property>
            <config-property name="password">
                password
            </config-property>
        </connection-definition>
    </connection-definitions>
</resource-adapter>
```
