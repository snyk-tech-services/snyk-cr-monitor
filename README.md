# snyk-cr-monitor

### Basic Flow

- Query the Artifactory API to gather Docker repos to test
- Iterate through results, running `snyk monitor` against each `*repo*` for the `*latest*` image
- Provides mechanism to run multiple jobs simultaneously that is configurable

Note that `snyk monitor` will run a `docker pull` behind-the-scenes.  

Please use the `SNYK_CR_MONITOR_MAX_JOBS` and `SNYK_CR_MONITOR_JOB_SPACING` variables, as describe below, to control the throughput to your docker repos, as pulling too many images at once may be taxing on the system.

### set environment
required:
```
# Your Snyk Api Key (General Settings -> API Token, or Settings -> Service Accounts)
export SNYK_TOKEN=<snyk_api_key>

# the Snyk Org ID to post results to (Settings -> General -> Organization ID)
export SNYK_ORG=<snyk_org_id> 

# The hostname (and optional port) of the on-premise artifactory instance, where the API will be accessible
# example: onpremartifactory.example.com, or onpremartifactory.example.com:8443
export ARTIFACTORY_API_HOST=<api_hostname> 

# The hostname (and optional port) of the host used with docker login
# This is separate from the API host because they may be different
export ARTIFACTORY_CLI_HOST=<cli_endpoint_hostname>

# The username for which the api key being used is created for
# This is needed in the docker pull command.
export ARTIFACTORY_USER=<artifactory_user_for_key>

# Artficatory API Key
export ARTIFACTORY_KEY=<artifactory_api_key>
```

optional: 
```
# Number of container scanning jobs that may be simultaneously running
# if unspecified, default is 10
export SNYK_CR_MONITOR_MAX_JOBS=<num-jobs>

# Number of milliseconds between job launches to space requests out
# If unspecified, default is 1500 (1.5 seconds)
export SNYK_CR_MONITOR_JOB_SPACING=<milliseconds-between-job-launches>
```

If connecting to an on-prem Artifactory instance over HTTPS with a self-signed certificate, you will need to set
```
export NODE_TLS_REJECT_UNAUTHORIZED=0
```
or
```
export NODE_EXTRA_CA_CERTS=[your CA certificate file path]
```


### Running as a container
```
docker build -t snyk-cr-monitor .
```
```
docker run -v /var/run/docker.sock:/var/run/docker.sock \
           -e SNYK_TOKEN=$SNYK_TOKEN \
           -e SNYK_ORG=$SNYK_ORG \
           -e ARTIFACTORY_USER=$ARTIFACTORY_USER \
           -e ARTIFACTORY_KEY=$ARTIFACTORY_KEY \
           -e ARTIFACTORY_API_HOST=$ARTIFACTORY_API_HOST \
           -e ARTIFACTORY_CLI_HOST=$ARTIFACTORY_CLI_HOST \
           -e SNYK_CR_MONITOR_MAX_JOBS=$SNYK_CR_MONITOR_MAX_JOBS \
           -e SNYK_CR_MONITOR_JOB_SPACING=$SNYK_CR_MONITOR_JOB_SPACING
       snyk-cr-monitor
```

* If extra environment variables are needed for self-signed certs, add them to the command above or the script below.

### Use bash wrapper script
modify arguments as needed for your environment
```
$ chmod +x snyk-cr-monitor
$ ./snyk-cr-monitor
```
### Running natively 
  requires node, docker, snyk
```
$ npm install -g 
$ snyk-cr-monitor
```

### References

**API** **Documentation** [Artificatory APIs](https://www.jfrog.com/confluence/display/RTF6X/Artifactory+REST+API#ArtifactoryRESTAPI-Authentication)

### TODO
- add support for AQL file option for customer image tag searches
- add tests
