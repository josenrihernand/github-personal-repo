## Overview:

This script is supposed to replace the hostname and, eventually, other components of the current webhook URL of the [orgs]/[repositories] located in the Enterprise Github Account code.espn.com

The script takes user arguments from the command line. 

Next the detailed usage information:

```
------------------------------------------------------------------------------
usage: update_webhooks.py [-h] [--d] [--o] [--c] [--u] [--k] [--t]
Arguments:

-h, --help  show this help message and exit
--d        * Please specify True | False for the DRYRUN mode
--o        * Please specify a valid organization whose repositories will be updated
--c        * Please specify the current hostname or IP address whose webhook you want to replace
--u        * Please specify a valid hostname/IP address to be updated
--k        * Please specify the hook type service to be processed (webhook for normal Hooks or servicehook for Integration & Services)
--t        Please specify the token in plain text, and enclosed in quotation marks. If it isn't passed, the token will be read from the s3 bucket

(*) required arguments
------------------------------------------------------------------------------
```

## Technologies needed:

- Python 2.7

The script uses the following python modules, so, all of them must be installed before execute the script:
- json
- requests
- boto3
- sys
- os
- github3
- argparse
- urlparse

For debugging purposes, it is also recommended to have the following modules installed
(but no required for normal execution):
- logging
- httplib


## Suggested approach to execute the script and description of some key arguments:

- Dry Run mode:

The first and most important suggestion to consider is the dry run mode. It is because as we already know from the command line arguments above, when we enable this mode (--d True) we can mitigate the possible effects of the script execution in a particular Github [repo]/[organization].
The central idea is that dry run produces an output of the potential changes, so that the user can verify them before performing a real (--d False) run.



- During a real execution, save the output for future reference:

Once we are sure about what it will be exactly changed (using dry run mode), we can proceed to perform a real run.
In this step it's suggested to save the execution. It could be simply by redirect the standard output to a text file (e.g: python update_webhooks.py arguments > MY_OUTPUT.out).
In that way we will have an text file which contains the output for the real execution.



- Info about the arguments --u and --c

The --u argument allow us to specify the hostname/IP address (or hostname/IP + path) that eventually will replace the current webhook URL.
With the value specified in the --c argument, the script searches the current IP address (or hostname) that contains the webhook URL to be replaced.

For example, suppose we have a webhook URL like this:
http://10.10.110.110:8080/pr/espn-pr/v1/teamcity?buildType=ConnectedDevices_Imaginary_01PullRequest

If we want to replace the IP address 10.10.110.110 by 20.20.120.120, we would specify --c 10.10.110.110 and --u 20.20.120.120, as below:

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "xxxxx" --c 10.10.110.110 --u 20.20.120.120 --k webhook
```

In this example, we are specifying the value to replace only the IP address.

The output (new webhook URL) would be:
http://20.20.120.120:8080/pr/espn-pr/v1/teamcity?buildType=ConnectedDevices_Imaginary_01PullRequest

Continuing with the same example, if we wanted to specify the hostname+path in --u argument, we could run the script as below:

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "xxxxx" --c 10.10.110.110 --u http://pr.aws.hosted.espn.com/espn-pr/v1/teamcity --k webhook
```

The output (new webhook URL) would be:
http://pr.aws.hosted.espn.com/espn-pr/v1/teamcity?buildType=ConnectedDevices_Imaginary_01PullRequest




## Common Issues / Error messages:

In this section, the description of some messages that could appear during the execution of the script will be shown:

Message:	` **The organization [ORG_NAME] doesn't have any repo created. Please specify a valid org.** `
Description: 	The org specified with the argument --o doesn't have any repo created. A common cause of this message might be a typo in the name of the org.
	
Message:	` ***This repo doesn't have webhook URL `
Description:	The current repo (for the org specified) doesn't have any webhook URL. 
		(It is possible that other repositories for the current organization have created a webhook URL).

Message:	**\*** Warning: The protocol schema passed in --u argument is different for this webhook: [PROTOCOL_NAME], No action was taken **
Description:	Obvious. The script validates that the protocol schema specified with --u argument matches the current webhook URL.
		Example: The protocol schema of the webhook URL is "http", and the one that was passed with argument --u is "https" (or other than "http").
	
Message:	**\*** The hostname doesn't match the one that passed as an argument [HOSTNAME] != [CURRENT_HOSTNAME] ***\**
Description:	The hostname or IP address specified with the argument --c doesn't match the current webhook URL.
		The value of --c argument is the search key through which the script will locate the webhook URL that will be updated. If the value specified in this 
		argument is not found in the webhook URL, the script will display the current webhook URL along with this message, for that specific webhook URL.
	
Message:	**\*** Webhook URLs Found, but none of type: [HOOK_TYPE] [HOOK_TYPE_DESCRIPTION] ***\**
Description:	The hook type specified in --k arg (webhook for normal Hooks or servicehook for Integration & Services) doesn't match the hook type of the current webhook URL.
		At this moment the script recognizes the following hook types, as valid ones: 
			- For normal hooks (--k webhook): 
				url. Example: https://hooks.slack.com/services... hooks.slack.com is a good example of a normal webhook.
							
			- For Integration & Services (--k servicehook): 
				This hook type will recognize all of these: base_url (teamcity), url_base (fisheye.corp.dig.com), or jenkins_hook_url (Jenkins). URL of Teamcity and Jenkins are both examples of servicehooks or Integrations & Services.
					
Message:	** Something failed with this repository: [REPO NAME] / Please check: [ERROR_DESCRIPTION] **
Description:	This is an uncommon error message. If a strange situation occurs during the processing of a specific repository, this message will be displayed and, also, the description of the error so that it is possible to take a look.
	
Message:	** The organization [ORG_NAME] doesn't have repos with webhooks URL configured **
Description:	If none of the repositories (for the current org) have webhook URL of the hook type specified with the argument --k, this message will be displayed.
	
Message:	** *** Please check the update of the webhook (URL). There was an error during the requests.patch API command execution *** **
Description: 	The response of the webhook URL updating process was not 200 (response.status_code).
		It is necessary to investigate and figure out what should have happened during the replacement of this specific webhook URL.





## Examples (test cases)
f
In this section, we will demonstrate the execution of the script by means of some examples.
All the tests were executed in a test organization: code test-github-hooks-01 (code.espn.com)


#### `EXAMPLE 01`: Replace the IP address in two webhook URLs, of the regular Hook type (--k webhook). In this example, the name of the repository is repo01. The test consist of replace the IP address _10.10.110.110_ by _20.20.120.120_.

##### 1. With dry run mode True (--d True).

```
python update_webhooks.py --d True --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c 10.10.110.110 --u 20.20.120.120 --k webhook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo01 - Repo ID: 11235 - DRYRUN = True
------------------------------------------
  Webhook: 00001 http://10.10.110.110:8080/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Alpine3jre864_01PullRequest
  -------
  New URL: http://20.20.120.120:8080/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Alpine3jre864_01PullRequest


  Webhook: 00002 http://10.10.110.110/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Centos7jdk864_01PullRequest
  -------
  New URL: http://20.20.120.120/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Centos7jdk864_01PullRequest
```


##### 2. Now with dry run mode False (--d False).

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c 10.10.110.110 --u 20.20.120.120 --k webhook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo01 - Repo ID: 11235 - DRYRUN = False
------------------------------------------
  Webhook: 00001 http://10.10.110.110:8080/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Alpine3jre864_01PullRequest
  -------
  New URL: http://20.20.120.120:8080/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Alpine3jre864_01PullRequest
  *** The webhook was updated successfully ***


  Webhook: 00002 http://10.10.110.110/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Centos7jdk864_01PullRequest
  -------
  New URL: http://20.20.120.120/pr/espn-pr/v1/teamcity?buildType=CseInfrastructure_XxDocker_Centos7jdk864_01PullRequest
  *** The webhook was updated successfully ***
```




#### `EXAMPLE 02`: Replace the domain in one webhook URL, of the servicehook Hook type (--k servicehook). In this example, the name of the repository is repo02. The test consist of replace the hostname _teamcityTEST.corp.espn3.com_ by _teamcityNEW_DOMAIN.corp.espn3.com_.

##### 1. With dry run mode True (--d True).

```
python update_webhooks.py --d True --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c teamcityTEST.corp.espn3.com --u teamcityNEW_DOMAIN.corp.espn3.com --k servicehook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo02 - Repo ID: 11236 - DRYRUN = True
------------------------------------------
  Webhook: 00001 http://teamcityTEST.corp.espn3.com/
  -------
  New URL: http://teamcityNEW_DOMAIN.corp.espn3.com/
```

##### 2. Now with dry run mode False (--d False).

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c teamcityTEST.corp.espn3.com --u teamcityNEW_DOMAIN.corp.espn3.com --k servicehook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo02 - Repo ID: 11236 - DRYRUN = False
------------------------------------------
  Webhook: 00001 http://teamcityTEST.corp.espn3.com/
  -------
  New URL: http://teamcityNEW_DOMAIN.corp.espn3.com/
  *** The webhook was updated successfully ***
```



#### `EXAMPLE 03`: Replace the domain in one webhook URL, of the servicehook Hook type (--k servicehook). In this example, the name of the repository is repo02. The test consist of replace the hostname _jenksTEST.ci.cloudbees.com_ by _jenksNEW_DOMAIN.ci.cloudbees.com_.

##### 1. With dry run mode True (--d True).

```
python update_webhooks.py --d True --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c jenksTEST.ci.cloudbees.com --u jenksNEW_DOMAIN.ci.cloudbees.com --k servicehook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo02 - Repo ID: 11236 - DRYRUN = True
------------------------------------------
  Webhook: 00003 https://jenksTEST.ci.cloudbees.com/github-webhook/
  -------
  New URL: https://jenksNEW_DOMAIN.ci.cloudbees.com/github-webhook/
```

##### 2. Now with dry run mode False (--d False).

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c jenksTEST.ci.cloudbees.com --u jenksNEW_DOMAIN.ci.cloudbees.com --k servicehook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo02 - Repo ID: 11236 - DRYRUN = False
------------------------------------------
  Webhook: 00003 https://jenksTEST.ci.cloudbees.com/github-webhook/
  -------
  New URL: https://jenksNEW_DOMAIN.ci.cloudbees.com/github-webhook/
  *** The webhook was updated successfully ***
```





#### `EXAMPLE 04`: Replace the domain in one webhook URL, of the regular Hook type (--k webhook). In this example, the name of the repository is repo03. The test consist of replace the hostname _localcmsTEST.cricinfo.com_ by _localcmsNEW_DOMAIN.cricinfo.com_.

##### 1. With dry run mode True (--d True).

```
python update_webhooks.py --d True --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c localcmsTEST.cricinfo.com --u localcmsNEW_DOMAIN.cricinfo.com --k webhook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo03 - Repo ID: 11237 - DRYRUN = True
------------------------------------------
  Webhook: 00001 http://localcmsTEST.cricinfo.com:9000/cgi-bin/cricinfo-git-build-v1.pl
  -------
  New URL: http://localcmsNEW_DOMAIN.cricinfo.com:9000/cgi-bin/cricinfo-git-build-v1.pl
```


##### 2. Now with dry run mode False (--d False).

```
python update_webhooks.py --d False --o codetest-github-hooks-01 --t "48d4c3ef6bb98f85feaaad045c62f983241ece57" --c localcmsTEST.cricinfo.com --u localcmsNEW_DOMAIN.cricinfo.com --k webhook


Webhooks for repository (with webhook URL): codetest-github-hooks-01/repo03 - Repo ID: 11237 - DRYRUN = False
------------------------------------------
  Webhook: 00001 http://localcmsTEST.cricinfo.com:9000/cgi-bin/cricinfo-git-build-v1.pl
  -------
  New URL: http://localcmsNEW_DOMAIN.cricinfo.com:9000/cgi-bin/cricinfo-git-build-v1.pl
  *** The webhook was updated successfully ***
```
