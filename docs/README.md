***
# Installing the xsede-oauth-mapfile tool
***

This package generates a file called **xsede-oauth-mapfile** containing entries like:

    {xsede_username}@xsede.org {local_username}

Each entry maps an XSEDE OAuth identity in the form {xsede_username}@xsede.org to the
corresponding local username on a specific XSEDE resource. An XSEDE OAuth identity
may have multiple lines mapping it to multiple local usernames. These mappings come
from the XSEDE Central Database (XCDB) and are accessed by this tool through an API.

XSEDE's Globus Connect Server (GCS) v5.4+ and other tools use these mappings to
access local resources as the authenticated user.

## Install

Choose either RPM or TAR install.

#### RPM Install

Setup XSEDE RPM repository trust:

  * For production software use [Production Repo Trust Instructions](https://software.xsede.org/production/repo/repoconfig.txt)
  * For development/testing software use [Development Repo Trust Instructions](https://software.xsede.org/development/repo/repoconfig.txt)

Install package:

     $ yum install xsede-oauth-mapfile

The RPM installs files under /usr/local/share/utils/xsede_oauth_mapfile/ by default.

#### TAR Install

Download the latest Production or Development/Testing tar from either:
  * [https://software.xsede.org/production/xsede-oauth-mapfile/latest](https://software.xsede.org/production/xsede-oauth-mapfile/latest)
  * [https://software.xsede.org/development/xsede-oauth-mapfile/latest](https://software.xsede.org/development/xsede-oauth-mapfile/latest)

Execute:

    $ mkdir {arbitrary_path}/xsede-oauth-mapfile-<version>
    $ cd {arbitrary_path}/xsede-oauth-mapfile-<version>
    $ tar -xzf {downloaded_file}

Edit the untarred bin/xsede-oauth-mapfile.sh and set MAP_FILE_BASE to the directory that you
just untarred.


## Request API Access Key

If you don't have an XDCDB access API-KEY from a previous xsede-oauth-mapfile install,
request one by following the instructions at https://xsede-xdcdb-api.xsede.org/, Clicking
on the "Generate APIKEY" link, and following the displayed instructions. In the e-mail to
help@xsede.org specify <AGENT> "spacct". If you are accessing the API for an XSEDE allocated
resource, provide the official XDCDB Resource Name, like "expanse.sdsc.xsede.org". If you are
are testing the xsede-oauth-mapfile tool, or accessing mapping information for some other
reason, provide the fully qualified hostname that will access the API as the XDCDB Resource
Name. The XDCDB Resource Name is the API Client ID and does not limit which resources mappings
can be looked up with API calls. XSEDE's active XDCDB Resource Names are listed at:
* https://info.xsede.org/wh1/warehouse-views/v1/resources-xdcdb-active/?format=html

The email request will register your xsede-oauth-mapfile deployment in XDCDB and authorize it
to access the XDCDB API using your API-KEY.

### Configure xsede-oauth-mapfile

If you have an etc/xsede-oauth-mapfile-config.json from a previous install, copy it to
this install etc/ directory, otherwise create it by copying the newly RPM/TAR installed
etc/xsede-oauth-mapfile-config-template.json.

Edit etc/xsede-oauth-mapfile-config.json and set:

    "XA-AGENT": "spacct",
    "XA-RESOURCE": "<XDCDB resource name> or <other fully qualified API client hostname>",
    "MAP-RESOURCE": "<XDCDB resource name>",
    "XA-API-KEY": "<your API-KEY>"

Where "XA-RESOURCE" identifies the Client ID (XDCDB Resource Name above) that is retrieving
the mappings, and "MAP-RESOURCE" is which XSEDE resource mappings you are retrieving. On a
production resource both of these are normally the same, but in a testing or other situation
a testing server / XA-RESOURCE may be accessing mappings for a production MAP-RESOURCE.

The API-KEY may be from a previous xsede-oauth-mapfile configuration or a new one
from the previous step.

Set the permissions for the config for read-only by root to keep the API-KEY private:

    $ chmod 0600 etc/xsede-oauth-mapfile-config.json

If you have additional local mappings that don't come from XDCDB, place them in
/etc/grid-security/xsede-oauth-mapfile.local, or customize the LOCAL_FILE variable
in bin/xsede-oauth-mapfile.sh to point to your local mappings.

XDCDB generated mappings and local mappings will be combined into the MAP_FILE.

If you change the default path of the mapfiles from /etc/grid-security/ to another
directory, also update the "mapfile = " paths in the bin/gcs-mapfile-lookup.py script. 

### Setup to generate using cron

Execute as root:

    $ cd /etc/cron.hourly
    $ ln -s /usr/local/share/utils/xsede_oauth_mapfile/bin/xsede-oauth-mapfile.sh

### Getting Support

Report xsede-oauth-mapfile bugs, questions, and suggestions to help@xsede.org.

### Advanced Usage

xsede-oauth-mapfile can be configured to create and maintain multiple different mapfiles, including ones based on the sets of authorized users for multiple different resources.

To create both a mapfile for resource A and one for resource B on the same machine, do the following:
1. Copy /usr/local/share/utils/xsede-oauth-mapfile/bin/xsede-oauth-mapfile.sh as /etc/cron.hourly/A.sh
2. Copy /usr/local/share/utils/xsede-oauth-mapfile/bin/xsede-oauth-mapfile.sh as /etc/cron.hourly/B.sh
3. Edit /etc/cron.hourly/A.sh, setting the following values:
  * MAP_FILE=/etc/grid-security/xsede-oauth-mapfile-A
  * CONFIG_FILE=etc/xsede-oauth-mapfile-config-A.json
  * GEN_FILE=xsede-oauth-mapfile-A
  * LOCAL_FILE=/etc/grid-security/xsede-oauth-mapfile-A.local
4. Edit /etc/cron.hourly/B.sh, setting the following values:
  * MAP_FILE=/etc/grid-security/xsede-oauth-mapfile-B
  * CONFIG_FILE=etc/xsede-oauth-mapfile-config-B.json
  * GEN_FILE=xsede-oauth-mapfile-B
  * LOCAL_FILE=/etc/grid-security/xsede-oauth-mapfile-B.local
5. copy /usr/local/share/utils/xsede-oauth-mapfile/etc/xsede-oauth-mapfile-config-template to /usr/local/share/utils/xsede-oauth-mapfile/etc/xsed-oauth-mapfile-config-A.json
6. edit /usr/local/share/utils/xsede-oauth-mapfile/etc/xsed-oauth-mapfile-config-A.json, setting XA-RESOURCE and XA-API-KEY to the values used when requesting your API key.  Set MAP_RESOURCE to be the XSEDE resource you wish to map oauth identity information for your "A" resource
7. copy /usr/local/share/utils/xsede-oauth-mapfile/etc/xsede-oauth-mapfile-config-template to /usr/local/share/utils/xsede-oauth-mapfile/etc/xsed-oauth-mapfile-config-B.json
8. edit /usr/local/share/utils/xsede-oauth-mapfile/etc/xsed-oauth-mapfile-config-A.json, setting XA-RESOURCE and XA-API-KEY to the values used when requesting your API key.  Set MAP_RESOURCE to be the XSEDE resource you wish to map oauth identity information for your "B" resource
9. Define any additional mappings for your "A" resource in/etc/grid-security/xsede-oauth-mapfile-A.local
10. Define any additional mappings for your "B" resource in/etc/grid-security/xsede-oauth-mapfile-B.local
11. Your mapfile for your "A" resource will be updated hourly by cron to the file /etc/grid-security/xsede-oauth-mapfile-A
12. Your mapfile for your "B" resource will be updated hourly by cron to the file /etc/grid-security/xsede-oauth-mapfile-B

## NOTES

**bin/xsede-oauth-mapfile.py** - Python map file generator.
It should run with default python3 and included modules.

**bin/gcs-mapfile-lookup** - Script used by GCS v5.4 to lookup values in
the **xsede-oauth-mapfile**. Instructions for doing so are in the GCS v5.4 Installation Guide.
