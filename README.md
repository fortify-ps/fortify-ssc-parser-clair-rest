<x-tag-head>
<x-tag-meta http-equiv="X-UA-Compatible" content="IE=edge"/>

<x-tag-script language="JavaScript"><!--
<X-INCLUDE url="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@10.0.0/build/highlight.min.js"/>
--></x-tag-script>

<x-tag-script language="JavaScript"><!--
<X-INCLUDE url="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js" />
--></x-tag-script>

<x-tag-script language="JavaScript"><!--
<X-INCLUDE url="${gradleHelpersLocation}/spa_readme.js" />
--></x-tag-script>

<x-tag-style><!--
<X-INCLUDE url="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@10.0.0/build/styles/github.min.css" />
--></x-tag-style>

<x-tag-style><!--
<X-INCLUDE url="${gradleHelpersLocation}/spa_readme.css" />
--></x-tag-style>
</x-tag-head>

# Fortify SSC Parser Plugin for Clair / REST API

## Introduction

This Fortify SSC parser plugin allows for importing scan results from Clair (Vulnerability Static Analysis for Containers).

Clair itself doesn't provide any file-based reports; as such this parser plugin parses files containing JSON produced by the
Clair 2.x `/v1/layers/{layerId}?features&vulnerabilities` REST API call. See the [Obtain results](#obtain-results) section for more information.

### Related Links

* **Downloads**: https://github.com/fortify-ps/fortify-ssc-parser-clair-rest/releases
    * _Development releases may be unstable or non-functional. The `*-thirdparty.zip` file is for informational purposes only and does not need to be downloaded._
* **Sample input files**: [sampleData](sampleData)
* **GitHub**: https://github.com/fortify-ps/fortify-ssc-parser-clair-rest
* **Automated builds**: https://github.com/fortify-ps/fortify-ssc-parser-clair-rest/actions
* **Clair resources**:
	* Clair GitHub repository: https://github.com/quay/clair/tree/v2.1.2
	* Legacy Clair documentation: https://coreos.com/clair/docs/latest/
* **Alternatives**:
	* SSC Parser Plugin for Yair (Clair client): https://github.com/fortify-ps/fortify-ssc-parser-clair-yair

## Plugin Installation

These sections describe how to install, upgrade and uninstall the plugin.

### Install & Upgrade

* Obtain the plugin binary jar file
	* Either download from Bintray (see [Related Links](#related-links)) 
	* Or by building yourself (see [Developers](#developers))
* If you already have another version of the plugin installed, first uninstall the previously 
 installed version of the plugin by following the steps under [Uninstall](#uninstall) below
* In Fortify Software Security Center:
	* Navigate to Administration->Plugins->Parsers
	* Click the `NEW` button
	* Accept the warning
	* Upload the plugin jar file
	* Enable the plugin by clicking the `ENABLE` button
  
### Uninstall

* In Fortify Software Security Center:
	* Navigate to Administration->Plugins->Parsers
	* Select the parser plugin that you want to uninstall
	* Click the `DISABLE` button
	* Click the `REMOVE` button 


## Obtain results

* Have Clair perform a scan of your container image
	* For example, using some Clair command line client like Yair
	* Or through container registry integration
* Determine the bottom layer id of the container image that was scanned
	* For example by inspecting the image manifest
* Invoke the Clair `/v1/layers/{layerId}?features&vulnerabilities` REST endpoint
	* Replace `{layerId}` with the bottom layer id identified in the previous step
	* Save the results in a file with the `.json` extension
	* See https://coreos.com/clair/docs/latest/api_v1.html#get-layersname for more information about this API endpoint
	* According to the documentation, this REST endpoint returns all vulnerabilities for both the given layer, and all upper layers
    
The following steps were used to generate the 
[sampleData/node_10.14.2-jessie.clair.rest.json](sampleData/node_10.14.2-jessie.clair.rest.json) 
file:

* Use Yair to scan the `node:10.14.2-jessie` image
	* See https://github.com/fortify-ps/fortify-ssc-parser-clair-yair#obtain-results for an example on how to set-up Clair and run a scan with Yair
* Use the following command to determine the bottom layer id:  
  `layerId=$(docker manifest inspect -v node:10.14.2-jessie | jq -r '.[0]["SchemaV2Manifest"]["layers"][-1]["digest"]')`
	* This command requires Docker experimental mode to be enabled
	* Requires `jq` to be installed
	* Other images may require slightly different approach, depending on manifest version
	* Potentially there are better ways of obtaining this information
* Use the following command to invoke the Clair REST API endpoint and save the results:  
  `curl -X GET "http://localhost:6060/v1/layers/${layerId}?features&vulnerabilities" -o  node_10.14.2-jessie.clair.rest.json`

## Upload results

SSC web interface (manual upload):

* Navigate to the Artifacts tab of your application version
* Click the `UPLOAD` button
* Click the `ADD FILES` button, and select the JSON file to upload
* Enable the `3rd party results` check box
* Select the `CLAIR_REST_V1` type
  
SSC clients (FortifyClient, Maven plugin, ...):

* Generate a scan.info file containing a single line as follows:  
`engineType=CLAIR_REST_V1`
* Generate a zip file containing the following:
	* The scan.info file generated in the previous step
	* The JSON file containing scan results
* Upload the zip file generated in the previous step to SSC
	* Using any SSC client, for example FortifyClient
	* Similar to how you would upload an FPR file



## Developers

The following sections provide information that may be useful for developers of this utility.

### IDE's

This project uses Lombok. In order to have your IDE compile this project without errors, 
you may need to add Lombok support to your IDE. Please see https://projectlombok.org/setup/overview 
for more information.

### Gradle Wrapper

It is strongly recommended to build this project using the included Gradle Wrapper
scripts; using other Gradle versions may result in build errors and other issues.

The Gradle build uses various helper scripts from https://github.com/fortify-ps/gradle-helpers;
please refer to the documentation and comments in included scripts for more information. 

### Common Commands

All commands listed below use Linux/bash notation; adjust accordingly if you
are running on a different platform. All commands are to be executed from
the main project directory.

* `./gradlew tasks --all`: List all available tasks
* Build: (plugin binary will be stored in `build/libs`)
	* `./gradlew clean build`: Clean and build the project
	* `./gradlew build`: Build the project without cleaning
	* `./gradlew dist distThirdParty`: Build distribution zip and third-party information bundle
* `./fortify-scan.sh`: Run a Fortify scan; requires Fortify SCA to be installed

### Automated Builds

This project uses GitHub Actions workflows to perform automated builds for both development and production releases. All pushes to the main branch qualify for building a production release. Commits on the main branch should use [Conventional Commit Messages](https://www.conventionalcommits.org/en/v1.0.0/); it is recommended to also use conventional commit messages on any other branches.

User-facing commits (features or fixes) on the main branch will trigger the [release-please-action](https://github.com/google-github-actions/release-please-action) to automatically create a pull request for publishing a release version. This pull request contains an automatically generated CHANGELOG.md together with a version.txt based on the conventional commit messages on the main branch. Merging such a pull request will automatically publish the production binaries and Docker images to the locations described in the [Related Links](#related-links) section.

Every push to a branch in the GitHub repository will also automatically trigger a development release to be built. By default, development releases are only published as build job artifacts. However, if a tag named `dev_<branch-name>` exists, then development releases are also published to the locations described in the [Related Links](#related-links) section. The `dev_<branch-name>` tag will be automatically updated to the commit that triggered the build.


## License
<x-insert text="<!--"/>

See [LICENSE.TXT](LICENSE.TXT)

<x-insert text="-->"/>

<x-include url="file:LICENSE.TXT"/>

