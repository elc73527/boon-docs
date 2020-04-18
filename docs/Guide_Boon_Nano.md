![Logo](../images/BoonLogic.png)
# Guide: BoonNano
<br/>

## General
### [GET /expert/version](../Functions/get_version.md)
>returns the version number of the API that is currently running

<br/>

## Instances
### [POST /expert/v3/nanoInstance/{instanceID}](../Functions/post_nanoInstance.md)
>initializes a nano instance (using the given label)

### [GET /expert/v3/nanoInstance/{instanceID}](../Functions/get_nanoInstance.md)
>checks whether the given instanceID is an instantiated instance

### [DELETE /expert/v3/nanoInstance/{instanceID}](../Functions/delete_nanoInstance.md)
>deletes the specified instance

### [GET /expert/v3/nanoInstances](../Functions/get_nanoInstances.md)
>returns a list of running instance IDs

### [POST /expert/v3/snapshot/{instanceID}](../Functions/post_snapshot.md)
>loads previous nano settings and status to the instantiated instance

### [GET /expert/v3/snapshot/{instanceID}](../Functions/get_snapshot.md)
>saves the nano in its current state as a compressed serialized file

<br/>

## Configuration

### [GET /expert/v3/configTemplate](../Functions/get_configTemplate.md)
>returns a formatted json block from the given parameters

### [POST /expert/v3/clusterConfig/{instanceID}](../Functions/post_clusterConfig.md)
>loads the given configuration to the nano

### [GET /expert/v3/clusterConfig/{instanceID}](../Functions/get_clusterConfig.md)
>returns the json block containing the config parameters: mins, maxes, weights, labels (if applicable), numeric type, percent variation, streaming window size, and accuracy

### [POST /expert/v3/autoTuneConfig/{instanceID}](../Functions/post_autoTuneConfig.md)
>calculates and posts ideal clustering parameters for the given data in the buffer

<br/>

## Cluster
### [POST /expert/v3/data/{instanceID}](../Functions/post_data.md)
>uploads data to the buffer to be clustered

### [GET /expert/v3/bufferStatus/{instanceID}](../Functions/get_bufferStatus.md)
>returns the byte information about the status of the nano

### [POST /expert/v3/nanoRun/{instanceID}](../Functions/post_nanoRun.md)
>clusters the data in the buffer

### [GET /expert/v3/nanoResults/{instanceID}](../Functions/get_nanoResults.md)
>returns status results for each pattern clustered

### [GET /expert/v3/nanoStatus/{instanceID}](../Functions/get_nanoStatus.md)
>returns results for each cluster created

<br/>

[Return to documentation homepage](../UI-docs.md)
