## Test environments
* local windows 10 environment, R 4.3.1
* ubuntu 16.04 (on r_hub), R-release 3.6.1
* Fedora (on r_hub), R-devel
* win-builder (devel and release)

## R CMD check results
There were no ERRORs, or WARNINGS. There were 2 NOTES: 
1) Non-standard files/directories found at top level:
     'CHANGELOG.md' 'DISCLAIMER.md' 'LICENSE.md' 'code.json'
	 
	 -This package is an official software release of my agency,
	 (the U.S. Geological Survey), and these files are part of the 
	 documentation requirements for all of our software releases.

2) Initiating curl with CURL_SSL_BACKEND: openssl

	-I believe that this is an issue with how I use curl in several functions. 
	At my office I need to have an openssl certificate in order to submit curl
	requests through the firewall. As long as the user is set up to use curl
	this note should not create a problem, and at any rate I don't think it is
	specific to my package.


