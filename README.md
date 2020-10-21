# What is this repository for?

This is an automated test framework for crackle mobile using Behave (BDD), Appium (Native Mobile) and Webium for POM related tasks. The framework supports the following view ports
 - iOS (Native)
 - Apple TV
 
 [Learn Markdown](https://bitbucket.org/tutorials/markdowndemo)

# How do I get set up?

## Requirements
1. Python >= 2.7
2. Pipenv
3. Git
4. Xcode (for iOS)
5. Appium Desktop (For Attribute Matchers)
6. Browsermob proxy (running in standalone mode). flag: --use-littleproxy false
7. Carthage

## To setup this project

### OS X, Linux

1. Install Python >= 2.7
2. Install Pipenv
3. Ensure Python >= 2.7 and Pipenv are both visible in your systems path (i.e you can type python and pipenv from your terminal prompt and successfully have them execute without any errors)
4. Pull this repository using git (i.e `git clone https://github.com/CrackleEngineering/crackle_mobile_bdd_tests.git`)
5. Navigate to your preferred terminal (Terminal for Mac Users / cmd for Windows Users) and execute the following commands
6. Install carthage with brew "brew install carthage"

`pipenv install && pipenv shell`

If successfull the above command will install all project related dependencies and configure your terminal environment to be in line with configurations from pipenv (i.e force your python compiler to be >= 2.7 incase you have 2 versions of python installed) hence putting your terminal in shell mode


### Windows

Instructions for Windows are mostly the same as unix however it helps to install the following:
1. Install Python >= 2.7 
2. Install Git for Windows
3. Install mingw64 if you prefer a unix style shell


## Configuration

The execution of this project is controlled by `behave` test runner and as such requires several environment variables to be set in order for device capabilities to be configured during execution

# Running against a local device

Before executing against a local device, you will have to run an appium server taking note of the host ip and port.

If device_host and device_port flags are not set they will be defaulted to below

`device_host=127.0.0.1` and `device_port=4723`

Device execution could happen on a simulator or on a physical device

When executing on a simulator set `simulator=true`

## Running against an iOS device (simulator)

You can fetch a list of iOS simulators present on your Mac host by executing the command below

`instruments -s devices`

Then take note of the device name which will be followed by the platform version, device udid and if it is a simulator or not e.g. `iPhone 6s (9.3) [4B55788A-C353-43BC-8760-C982CF7534DE] (simulator)`

Hence to run against the above simulator assuming we started appium on the default host and port, execute the following command

`simulator=true device={device_name e.g iPhone 6s} device_type=mobile platform=ios platform_version=9.3 behave behave [optional flags as described above]`


### Running against an iOS device (simulator) - install ssl cert on simulators
For proxy tests you may need to install the browsermob-proxy CA root certificate on the simulators, web browsers and machine

Certificate:
https://github.com/lightbody/browsermob-proxy/blob/master/browsermob-core/src/main/resources/sslSupport/ca-certificate-rsa.cer

`$ git clone https://github.com/ADVTOOLS/ADVTrustStore.git
$ cd ADVTrustStore
$ ./iosCertTrustManager.py -a /path/to/ca-certificate-rsa.cer`

The above will install the certifcate on all simulator instances. Now both HTTP and HTTPS requests will get routed to browsermob-proxy. Note that the simulator uses the OS X system proxy settings. The system proxy settings must be set before the simulator launches otherwise the simulator won't be proxied.


## Running against an iOS device (real device)

As with simulators, you can get a list of real devices connected to your host by executing

`instruments -s devices`

Only unlike simulators, real devices will not have any (simulator) at the end e.g. `iPhone 6s (9.3) [4B55788A-C353-43BC-8760-C982CF7534DE]`

Hence to run against the above real device execute the following command

`udid='4B55788A-C353-43BC-8760-C982CF7534DE' device={device_name e.g. iPhone 6s} device_type=[mobile, tablet] platform=ios platform_version=9.3 behave [optional flags as described above]`

*Note*: From the above command, udid of the physical device must be set. Also you will have to set xcode_team_id=<Team ID> which is a unique 10 digit code issued in your apple developers dashboard and identifies your provisioning profile. You cannot execute tests without this on real devices

Here is an example command:
`app_path='/path/to/app.ipa' xcode_team_id='D963FY53C3' udid='YOUR DEVICE UDID' simulator=False device='iPhone 6' device_type=mobile platform=ios platform_version=11.3 environment=production pipenv run behave --no-capture --no-capture-stderr --no-skipped --tags=...`

## Optional execution flags (Environment variables)
* proxy: Required for tested tagged @proxy to run
* har_logging: If true, certain steps will output HAR logs to a subdirectory rather than print to stdout
* country_vpn, state_vpn, region_vpn: string values denoting the names of VPN connections set up in OS X Network settings
* L2TP_shared_secret: string value denoting L2TP vpn connection shared secret which has to be provided for tests that use the vpn functionality

### Setting up real iOS devices
iOS devices need some configuration in order for tests to run:

1. Wifi must be enabled

2. Mobile Data must be enabled (presence of sim card)

3. Wifi and Mobile data buttons must be present in the Control Center

4. Screen Recording button must be added to the Control Center

5. The 'General' link must be present in the iOS Preferences section (default)

### Troubleshooting iOS app deployment to real devices via Appium

1. The device UDID must be registered on the app provisioning profile, therefore the .app or .ipa file must be built for development
2. If Appium does not install Webdriver agent to the device, kill all Webdriver agent server instances with `killall WebDriver` on the host. You can check the status of the server at `http://localhost:8100`. 
3. If the app cannot be installed to the real device with error code -65 in the Appium logs, follow the guide to running Appium on real devies. Open the WebdriverAgent .xcodeproj file (as per the Appium Real Device config link) in XCode and ensure that the project and all its dependecies has a TEAM and PROVISIONING PROFILE assigned.
4. If app deployments are slow it may be deploying over wifi (ios-deploy). Connect the device over usb and disable the wifi connection to force usb deployment. The desired capability `noreset` can also be used. See Appium docs.

# Running against cloud (Saucelabs)

To execute tests against saucelabs, the `saucelabs=true` flag needs to be set and also SAUCE_USERNAME and SAUCE_AUTHKEY variables will have to be set else you will get an exception thrown

Since saucelabs uses capabilities to find the device you are testing against, you will need to use the saucelabs configurator tool [Sauce Labs Configurator Tool](https://wiki.saucelabs.com/display/DOCS/Platform+Configurator#/)

Then you can set the `device= platform= platform_version=`

Note: Saucelabs uses only simulators as real device testing has been moved to [Test Objects](https://testobject.com/)

# Executing based on tags

Note: Tag related expressions apply here as well e.g. ~ indicating except e.t.c. See https://pythonhosted.org/behave/behave.html#command-line-arguments for more information around execution switches

# Writing analytics/proxy tests

Tests can be written that require interception of HTTP requests. There are two generic steps that can be used: 

- request_contains_tag_and_value(data_type, tag, value) - assert true if an HTTP request contains tag, value pair. supported data types are query string, post data

- request_contains_all_tag_values() - assert true if a set of tag, value pairs as defined in context.table are all seen in a single HTTP request, otherwise false

1. To run tests tagged @proxy, Browsermob proxy standalone running in a separate shell with the following flags:
`--use-littleproxy false` and
`-address <machine local ip address>`
2. To run tests tagged @vpn_ (OS X only currently) VPN configurations should be set up in Network Preferences. A subscription to a VPN provider such as Private Internet Access is required. PIA is suitable for our existing tests as you can obtain an IP for a specific country or city, but you may want to review it's coverage
https://www.privateinternetaccess.com/pages/network/

VPN connections need to be manually set up in OS X Network Preferences one time only. We currently use L2TP with Private Internet Access as it is known to be fast and not cause bottlenecks during video playback tests.

Please create connections as per the instructions here:
https://www.privateinternetaccess.com/pages/client-support/osx-l2tp

# Analytics Reporting

Analytics tests are run using the Behave featureset feature. Files analytics.featureset and analytics_defects.featureset.

analytics.featureset is intended to run only tests that are not tagged '@bug'. analytics_defects.featureset is intended to run only tests that are tagged '@bug'.

When the test completes, a 'Single Scenario' report will be created in the after_scenario hook. This report will be added to the result of the test in TestRail.

Also the results of the test will be appended to a more detailed 'Master Report'. The Master Report is executed as a test after all Analytics tests have finished and contains a summary of all Analytics tests executed during the run. Details of this can be seen in analyics_master_report.feature and analytics_master_report_defects.feature

There are two Master Reports, one for the standard regression run (analytics.featureset) and a second Defects Master Report (analytics_defects.featureset).

The expectation is that all tests should have status PASSED in the standard 'Master Report' and for all tests to have status FAILURE in the 'Defects Master Report'. Any results that diverge from this expectation need to be analysed, as they may point to either a regression or bug that has been fixed, respectively. In both situations the tags associated with the Scenario would need to be updated.

`publish_reports=true` - if true, analytics reporting will be enabled for this run (Testrail and HTML type reports - see below). This includes both the 'Single Scenario' report and any Master Reports.

`master_report_testrail=true` - if true, if a master report is generated at the end of an analytics run the report will be embedded in the Testrail comment field of the test result. The test is usually called 'Master Report' or 'Master Report - Defects Run'.

`master_report_html=true` - if true, if a master report is generated at the end of an analytics run the report will be hosted online and a link to the report added to the testrail comment of the run result. The test is usually called 'Master Report' or 'Master Report - Defects Run'

## Notes on HTML Analytics Master Report

In order to generate the HTML analytics master report the following environment variables must be set (refer to the s3 extension and AWS S3 docs). These tell the module which AWS S3 account to point to. This should correspond to an AWS IAM account with read and write access to an S3 bucket.
`AWS_ACCESS_KEY_ID`
`AWS_SECRET_ACCESS_KEY`
`AWS_DEFAULT_REGION`
`AWS_S3_BUCKET_NAME`

# Writing Simulator tests that require both VPN + Proxy

In order to write tests against the simulator that make use of both a software vpn and browsermob-proxy, the order of operation is as follows, as per environment.py. (This also applies to manual testing)

Setup (before_scenario hook):
1. Create the proxy on browsermob-proxy server
2. Set the proxy on the system's primary network interface
3. Set the proxy on the system's VPN network interface
4. Set the HAR on the proxy

5. Test runs

Tear down (after_scenario hook):
6. Unset the proxy on the system's VPN network interface
7. Unset the proxy on the system's primary network interface
8. Close the proxy on browsermob-proxy server


# Troubleshooting

1. You receive the error: `HOOK-ERROR in before_scenario: gaierror: [Errno 8] nodename nor servname provided, or not known`
then run the command: `sudo echo 127.0.0.1 $HOSTNAME >> /etc/hosts`
